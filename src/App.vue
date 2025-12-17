<template>
  <div class="app-container">
    <header class="header">
      <h1>MetaCubeX Rule Mirror</h1>
      <p class="subtitle">Fast and reliable access to rule files</p>
    </header>

    <main class="main-content">
      <div v-if="loading" class="loading">
        <div class="spinner"></div>
        <p>Loading files...</p>
      </div>

      <div v-else-if="error" class="error">
        <p>{{ error }}</p>
      </div>

      <div v-else class="file-list-container">
        <div class="info-bar">
          <span class="file-count">{{ files.length }} files available</span>
          <span class="update-time" v-if="updatedAt">
            Last updated: {{ formatDate(updatedAt) }}
          </span>
        </div>

        <div class="file-list">
          <div v-for="file in files" :key="file.name" class="file-item-wrapper">
            <div class="file-item">
              <div class="file-info">
                <span class="file-icon">📄</span>
                <div class="file-details">
                  <span class="file-name">{{ file.name }}</span>
                  <div class="file-meta">
                    <span class="file-size">{{ file.size }}</span>
                    <span v-if="file.checksums && file.checksums.length > 0" class="checksum-badges">
                      <button 
                        v-for="checksum in file.checksums" 
                        :key="checksum.name"
                        @click="toggleChecksum(file.name); loadChecksumContent(checksum.name)"
                        class="checksum-badge"
                        :class="{ expanded: isExpanded(file.name) }"
                        :title="`点击查看 ${checksum.name}`"
                      >
                        {{ getChecksumType(checksum.name) }}
                      </button>
                    </span>
                  </div>
                  <span class="file-url">{{ getFileUrl(file.name) }}</span>
                </div>
              </div>
              <div class="file-actions">
                <button @click="copyLink(file.name)" class="copy-button" :class="{ copied: copiedFile === file.name }">
                  {{ copiedFile === file.name ? '已复制' : '复制链接' }}
                </button>
                <a :href="`/downloads/${file.name}`" class="download-button" download>
                  下载
                </a>
              </div>
            </div>

            <!-- 展开的校验信息 -->
            <div v-if="isExpanded(file.name) && file.checksums && file.checksums.length > 0" class="checksum-expanded">
              <div v-for="checksum in file.checksums" :key="checksum.name" class="checksum-row">
                <div class="checksum-info">
                  <span class="checksum-type-label">{{ getChecksumType(checksum.name) }}:</span>
                  <span class="checksum-value">{{ checksumContents[checksum.name] || '加载中...' }}</span>
                </div>
                <div class="checksum-actions">
                  <button 
                    @click="copyChecksumContent(checksum.name)" 
                    class="checksum-action-button"
                    :class="{ copied: copiedFile === `checksum-${checksum.name}` }"
                  >
                    {{ copiedFile === `checksum-${checksum.name}` ? '✓ 已复制' : '复制原文' }}
                  </button>
                  <button 
                    @click="copyLink(checksum.name)" 
                    class="checksum-action-button"
                    :class="{ copied: copiedFile === checksum.name }"
                  >
                    {{ copiedFile === checksum.name ? '✓ 已复制' : '复制链接' }}
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>

    <footer class="footer">
      <p>Powered by <strong>Tencent Cloud EdgeOne Pages</strong></p>
      <p class="source-info">
        Source: <a href="https://github.com/MetaCubeX/meta-rules-dat" target="_blank" rel="noopener">MetaCubeX/meta-rules-dat</a>
      </p>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const files = ref([])
const updatedAt = ref('')
const loading = ref(true)
const error = ref(null)
const copiedFile = ref(null)
const expandedFiles = ref(new Set())
const checksumContents = ref({})

const processFiles = (rawFiles) => {
  const checksumExtensions = ['.sha256sum', '.md5sum', '.sha512sum']
  const fileMap = new Map()
  const checksumMap = new Map()

  // 分类文件：原始文件 vs 校验文件
  rawFiles.forEach(file => {
    const isChecksum = checksumExtensions.some(ext => file.name.endsWith(ext))
    
    if (isChecksum) {
      // 找到对应的原始文件名
      let originalName = file.name
      for (const ext of checksumExtensions) {
        if (file.name.endsWith(ext)) {
          originalName = file.name.slice(0, -ext.length)
          break
        }
      }
      
      if (!checksumMap.has(originalName)) {
        checksumMap.set(originalName, [])
      }
      checksumMap.get(originalName).push(file)
    } else {
      fileMap.set(file.name, { ...file, checksums: [] })
    }
  })

  // 关联校验文件到原始文件
  checksumMap.forEach((checksums, originalName) => {
    if (fileMap.has(originalName)) {
      fileMap.get(originalName).checksums = checksums
    }
  })

  return Array.from(fileMap.values())
}

const fetchFiles = async () => {
  try {
    loading.value = true
    error.value = null
    
    const response = await fetch('/files.json')
    if (!response.ok) {
      throw new Error('Failed to fetch file list')
    }
    
    const data = await response.json()
    files.value = processFiles(data.files || [])
    updatedAt.value = data.updated_at || ''
  } catch (err) {
    error.value = err.message || 'Failed to load files'
    console.error('Error fetching files:', err)
  } finally {
    loading.value = false
  }
}

const getFileUrl = (filename) => {
  const baseUrl = window.location.origin
  return `${baseUrl}/downloads/${filename}`
}

const getChecksumType = (filename) => {
  if (filename.endsWith('.sha256sum')) return 'SHA256'
  if (filename.endsWith('.md5sum')) return 'MD5'
  if (filename.endsWith('.sha512sum')) return 'SHA512'
  return 'CHECKSUM'
}

const toggleChecksum = (filename) => {
  if (expandedFiles.value.has(filename)) {
    expandedFiles.value.delete(filename)
  } else {
    expandedFiles.value.add(filename)
  }
  expandedFiles.value = new Set(expandedFiles.value)
}

const isExpanded = (filename) => {
  return expandedFiles.value.has(filename)
}

const loadChecksumContent = async (filename) => {
  if (checksumContents.value[filename]) return
  
  try {
    const response = await fetch(`/downloads/${filename}`)
    if (response.ok) {
      const content = await response.text()
      // 提取纯 hash 值（去掉文件名部分）
      const trimmedContent = content.trim()
      // 校验文件格式通常是: <hash>  <filename> 或 <hash> <filename>
      const hashOnly = trimmedContent.split(/\s+/)[0]
      checksumContents.value[filename] = hashOnly || trimmedContent
    } else {
      checksumContents.value[filename] = '无法加载'
    }
  } catch (err) {
    console.error('Failed to load checksum:', err)
    checksumContents.value[filename] = '加载失败'
  }
}

const copyChecksumContent = async (filename) => {
  try {
    if (!checksumContents.value[filename]) {
      await loadChecksumContent(filename)
    }
    
    const content = checksumContents.value[filename]
    if (content && content !== '无法加载' && content !== '加载失败') {
      await navigator.clipboard.writeText(content)
      copiedFile.value = `checksum-${filename}`
      setTimeout(() => {
        copiedFile.value = null
      }, 2000)
    } else {
      alert('无法复制，校验文件内容未加载')
    }
  } catch (err) {
    console.error('Failed to copy checksum:', err)
    alert('复制失败，请重试')
  }
}

const copyLink = async (filename) => {
  try {
    const url = getFileUrl(filename)
    await navigator.clipboard.writeText(url)
    copiedFile.value = filename
    setTimeout(() => {
      copiedFile.value = null
    }, 2000)
  } catch (err) {
    console.error('Failed to copy:', err)
    alert('复制失败，请手动复制链接')
  }
}

const formatDate = (dateString) => {
  try {
    const date = new Date(dateString)
    return date.toLocaleString('en-US', {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit',
      timeZoneName: 'short'
    })
  } catch {
    return dateString
  }
}

onMounted(() => {
  fetchFiles()
})
</script>

<style scoped>
.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  padding: 3rem 2rem;
  background: #2563eb;
  color: white;
  border-bottom: 1px solid #e5e7eb;
}

.header h1 {
  font-size: 2.5rem;
  margin-bottom: 0.5rem;
  font-weight: 700;
}

.subtitle {
  font-size: 1.1rem;
  opacity: 0.95;
}

.main-content {
  flex: 1;
  padding: 2rem;
  max-width: 1200px;
  width: 100%;
  margin: 0 auto;
}

.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 4rem 2rem;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 4px solid rgba(37, 99, 235, 0.2);
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.error {
  padding: 2rem;
  background: #fee;
  color: #c33;
  border-radius: 8px;
  border: 1px solid #fcc;
}

.file-list-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

@media (prefers-color-scheme: dark) {
  .file-list-container {
    background: #2a2a2a;
  }
}

.info-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem 2rem;
  background: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
  flex-wrap: wrap;
  gap: 1rem;
}

@media (prefers-color-scheme: dark) {
  .info-bar {
    background: #333;
    border-bottom-color: #444;
  }
}

.file-count {
  font-weight: 600;
  color: #2563eb;
}

.update-time {
  color: #6c757d;
  font-size: 0.9rem;
}

.file-list {
  padding: 1rem;
}

.file-item-wrapper {
  border-bottom: 1px solid #e9ecef;
}

.file-item-wrapper:last-child {
  border-bottom: none;
}

@media (prefers-color-scheme: dark) {
  .file-item-wrapper {
    border-bottom-color: #444;
  }
}

.file-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.25rem 1.5rem;
  transition: background-color 0.2s ease;
  gap: 1.5rem;
}

.file-item:hover {
  background-color: #f8f9fa;
}

@media (prefers-color-scheme: dark) {
  .file-item:hover {
    background-color: #333;
  }
}

.file-info {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex: 1;
  min-width: 0;
}

.file-icon {
  font-size: 1.5rem;
  flex-shrink: 0;
}

.file-details {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 0.4rem;
  flex: 1;
  min-width: 0;
}

.file-name {
  font-weight: 600;
  font-size: 0.95rem;
  word-break: break-all;
  line-height: 1.4;
}

.file-meta {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.file-size {
  font-size: 0.85rem;
  color: #6c757d;
}

.checksum-badges {
  display: flex;
  gap: 0.5rem;
}

.checksum-badge {
  display: inline-block;
  padding: 0.15rem 0.5rem;
  background: #f3f4f6;
  color: #6b7280;
  font-size: 0.75rem;
  font-weight: 500;
  border-radius: 4px;
  transition: all 0.2s ease;
  border: 1px solid #e5e7eb;
  cursor: pointer;
}

.checksum-badge:hover {
  background: #e5e7eb;
  color: #374151;
  border-color: #d1d5db;
}

.checksum-badge.expanded {
  background: #2563eb;
  color: white;
  border-color: #2563eb;
}

@media (prefers-color-scheme: dark) {
  .checksum-badge {
    background: #374151;
    color: #9ca3af;
    border-color: #4b5563;
  }
  
  .checksum-badge:hover {
    background: #4b5563;
    color: #d1d5db;
    border-color: #6b7280;
  }

  .checksum-badge.expanded {
    background: #2563eb;
    color: white;
    border-color: #2563eb;
  }
}

.checksum-expanded {
  padding: 0.75rem 1.5rem 1rem 4.5rem;
  background: #f8f9fa;
  border-top: 1px solid #e9ecef;
}

@media (prefers-color-scheme: dark) {
  .checksum-expanded {
    background: #1f2937;
    border-top-color: #374151;
  }
}

.checksum-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  gap: 1rem;
  background: white;
  border-radius: 6px;
  margin-bottom: 0.5rem;
  border: 1px solid #e5e7eb;
}

.checksum-row:last-child {
  margin-bottom: 0;
}

@media (prefers-color-scheme: dark) {
  .checksum-row {
    background: #374151;
    border-color: #4b5563;
  }
}

.checksum-info {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex: 1;
  min-width: 0;
}

.checksum-type-label {
  font-size: 0.75rem;
  font-weight: 600;
  color: white;
  background: #2563eb;
  padding: 0.25rem 0.6rem;
  border-radius: 4px;
  white-space: nowrap;
  min-width: 70px;
  text-align: center;
}

@media (prefers-color-scheme: dark) {
  .checksum-type-label {
    background: #1e40af;
  }
}

.checksum-value {
  font-family: 'Monaco', 'Menlo', 'Consolas', monospace;
  font-size: 0.75rem;
  color: #4b5563;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  user-select: all;
  flex: 1;
  cursor: text;
  padding: 0.25rem 0;
}

.checksum-value:hover {
  color: #1f2937;
}

@media (prefers-color-scheme: dark) {
  .checksum-value {
    color: #9ca3af;
  }
  
  .checksum-value:hover {
    color: #e5e7eb;
  }
}

.checksum-actions {
  display: flex;
  gap: 0.5rem;
  flex-shrink: 0;
}

.checksum-action-button {
  padding: 0.5rem 0.9rem;
  background: #f3f4f6;
  color: #4b5563;
  border: 1px solid #e5e7eb;
  border-radius: 4px;
  font-size: 0.8rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.checksum-action-button:hover {
  background: #e5e7eb;
  border-color: #d1d5db;
  color: #1f2937;
}

.checksum-action-button.copied {
  background: #10b981;
  color: white;
  border-color: #10b981;
}

@media (prefers-color-scheme: dark) {
  .checksum-action-button {
    background: #1f2937;
    color: #d1d5db;
    border-color: #4b5563;
  }

  .checksum-action-button:hover {
    background: #374151;
    border-color: #6b7280;
    color: #f3f4f6;
  }
}

.file-url {
  font-size: 0.75rem;
  color: #9ca3af;
  font-family: 'Monaco', 'Menlo', 'Consolas', monospace;
  word-break: break-all;
  line-height: 1.5;
}

.file-actions {
  display: flex;
  gap: 0.5rem;
  flex-shrink: 0;
  align-items: center;
}

.copy-button {
  padding: 0.55rem 1.1rem;
  background: white;
  color: #2563eb;
  border: 1px solid #2563eb;
  border-radius: 5px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
  font-size: 0.85rem;
}

.copy-button:hover {
  background: #eff6ff;
}

.copy-button.copied {
  background: #10b981;
  color: white;
  border-color: #10b981;
}

.download-button {
  padding: 0.55rem 1.3rem;
  background: #2563eb;
  color: white;
  text-decoration: none;
  border-radius: 5px;
  font-weight: 500;
  transition: all 0.2s ease;
  white-space: nowrap;
  display: inline-block;
  font-size: 0.85rem;
}

.download-button:hover {
  background: #1d4ed8;
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.download-button:active {
  transform: translateY(0);
}

.footer {
  padding: 2rem;
  background: #f8f9fa;
  border-top: 1px solid #e9ecef;
  margin-top: auto;
}

@media (prefers-color-scheme: dark) {
  .footer {
    background: #2a2a2a;
    border-top-color: #444;
  }
}

.footer p {
  margin: 0.5rem 0;
}

.source-info {
  font-size: 0.9rem;
  color: #6c757d;
}

.source-info a {
  color: #2563eb;
  text-decoration: none;
}

.source-info a:hover {
  text-decoration: underline;
}

@media (max-width: 768px) {
  .header h1 {
    font-size: 1.8rem;
  }

  .subtitle {
    font-size: 1rem;
  }

  .file-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 1rem;
  }

  .file-actions {
    width: 100%;
    flex-direction: column;
  }

  .copy-button,
  .download-button {
    width: 100%;
    text-align: center;
  }

  .checksum-expanded {
    padding-left: 1.5rem;
    padding-right: 1.5rem;
  }

  .checksum-row {
    flex-direction: column;
    align-items: stretch;
    gap: 0.75rem;
    padding: 0.75rem;
  }

  .checksum-info {
    width: 100%;
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
  }

  .checksum-type-label {
    align-self: flex-start;
  }

  .checksum-value {
    width: 100%;
    white-space: normal;
    word-break: break-all;
    font-size: 0.7rem;
  }

  .checksum-actions {
    width: 100%;
  }

  .checksum-action-button {
    flex: 1;
    font-size: 0.75rem;
    padding: 0.5rem 0.75rem;
  }

  .info-bar {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>

