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
          <div v-for="file in files" :key="file.name" class="file-item">
            <div class="file-info">
              <span class="file-icon">📄</span>
              <div class="file-details">
                <span class="file-name">{{ file.name }}</span>
                <span class="file-size">{{ file.size }}</span>
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

const fetchFiles = async () => {
  try {
    loading.value = true
    error.value = null
    
    const response = await fetch('/files.json')
    if (!response.ok) {
      throw new Error('Failed to fetch file list')
    }
    
    const data = await response.json()
    files.value = data.files || []
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

.file-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem 1.5rem;
  border-bottom: 1px solid #e9ecef;
  transition: background-color 0.2s ease;
  gap: 1rem;
}

@media (prefers-color-scheme: dark) {
  .file-item {
    border-bottom-color: #444;
  }
}

.file-item:last-child {
  border-bottom: none;
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
}

.file-icon {
  font-size: 1.5rem;
}

.file-details {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 0.35rem;
}

.file-name {
  font-weight: 600;
  font-size: 1rem;
  word-break: break-all;
}

.file-size {
  font-size: 0.85rem;
  color: #6c757d;
}

.file-url {
  font-size: 0.8rem;
  color: #9ca3af;
  font-family: 'Monaco', 'Menlo', 'Consolas', monospace;
  word-break: break-all;
}

.file-actions {
  display: flex;
  gap: 0.75rem;
  flex-shrink: 0;
}

.copy-button {
  padding: 0.6rem 1.2rem;
  background: white;
  color: #2563eb;
  border: 1px solid #2563eb;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
  font-size: 0.9rem;
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
  padding: 0.6rem 1.5rem;
  background: #2563eb;
  color: white;
  text-decoration: none;
  border-radius: 6px;
  font-weight: 500;
  transition: all 0.2s ease;
  white-space: nowrap;
  display: inline-block;
  font-size: 0.9rem;
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

  .info-bar {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>

