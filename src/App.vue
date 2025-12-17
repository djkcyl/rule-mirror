<template>
  <div class="min-h-screen flex flex-col bg-gray-50 dark:bg-gray-950">
    <!-- Header -->
    <header class="bg-blue-600 dark:bg-blue-700 text-white shadow-lg">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12 sm:py-16">
        <div class="text-center">
          <h1 class="text-3xl sm:text-4xl lg:text-5xl font-bold mb-3 tracking-tight">Rule Files Mirror</h1>
          <p class="text-base sm:text-lg text-blue-100 max-w-2xl mx-auto">Fast and reliable rule file downloads with automatic updates</p>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="flex-1 w-full px-4 sm:px-6 lg:px-8 py-6 sm:py-8 lg:py-12">
      <div class="max-w-7xl mx-auto">
        <!-- Loading State -->
        <div v-if="loading" class="flex flex-col items-center justify-center py-20">
          <div class="relative">
            <div class="w-16 h-16 border-4 border-blue-200 dark:border-blue-800 border-t-blue-600 dark:border-t-blue-400 rounded-full animate-spin"></div>
            <div class="absolute inset-0 w-16 h-16 border-4 border-transparent border-b-blue-400 dark:border-b-blue-600 rounded-full animate-spin animation-delay-150"></div>
          </div>
          <p class="mt-6 text-gray-600 dark:text-gray-400 font-medium animate-pulse">Loading files...</p>
        </div>

        <!-- Error State -->
        <div v-else-if="error" class="max-w-2xl mx-auto">
          <div class="p-6 bg-red-50 dark:bg-red-900/20 text-red-800 dark:text-red-200 rounded-xl border-l-4 border-red-500 shadow-sm">
            <div class="flex items-start gap-3">
              <svg class="w-6 h-6 shrink-0 mt-0.5" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
              </svg>
              <p class="flex-1">{{ error }}</p>
            </div>
          </div>
        </div>

        <!-- Content -->
        <div v-else class="space-y-6 sm:space-y-8">
          <!-- Global Info Bar -->
          <div class="bg-white dark:bg-gray-900 rounded-2xl shadow-md border border-gray-200 dark:border-gray-700 overflow-hidden">
            <div class="px-6 py-4 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
              <div class="flex flex-wrap items-center gap-4">
                <div class="flex items-center gap-2 text-gray-700 dark:text-gray-100">
                  <svg class="w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"/>
                  </svg>
                  <span class="font-medium">{{ projects.length }} Projects</span>
                </div>
                <div class="flex items-center gap-2 text-gray-700 dark:text-gray-100">
                  <svg class="w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                  </svg>
                  <span class="font-medium">{{ totalFiles }} Files</span>
                </div>
                <div v-if="totalSize" class="flex items-center gap-2 text-gray-700 dark:text-gray-100">
                  <svg class="w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 7v10c0 2.21 3.582 4 8 4s8-1.79 8-4V7M4 7c0 2.21 3.582 4 8 4s8-1.79 8-4M4 7c0-2.21 3.582-4 8-4s8 1.79 8 4"/>
                  </svg>
                  <span class="font-medium">{{ totalSize }}</span>
                </div>
              </div>
              <div v-if="updatedAt" class="flex items-center gap-2 text-sm text-gray-600 dark:text-gray-300">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
                </svg>
                <span>Last updated: {{ formatDate(updatedAt) }}</span>
              </div>
            </div>
          </div>

          <!-- Projects -->
          <div v-for="(project, index) in projects" :key="project.name" 
               class="bg-white dark:bg-gray-900 rounded-2xl shadow-md border border-gray-200 dark:border-gray-700 overflow-hidden transition-all duration-300 hover:shadow-lg hover:border-blue-300 dark:hover:border-blue-600"
               :class="{ 'animate-fade-in-up': true }"
               :style="{ animationDelay: `${index * 100}ms` }">
            <!-- Project Header -->
            <div class="px-6 py-6 bg-gray-50 dark:bg-gray-800 border-b-2 border-blue-500">
              <div class="flex flex-col sm:flex-row sm:items-start sm:justify-between gap-4">
                <div class="flex-1">
                  <div class="flex items-center gap-3 mb-2">
                    <h2 class="text-xl sm:text-2xl font-bold text-gray-900 dark:text-white flex items-center gap-2">
                      {{ project.name }}
                    </h2>
                    <a :href="getProjectUrl(project.name)" 
                       class="group flex items-center justify-center p-2 text-gray-500 dark:text-gray-400 hover:text-blue-600 dark:hover:text-blue-400 hover:bg-blue-50 dark:hover:bg-blue-900/30 rounded-lg transition-all duration-200"
                       target="_blank" 
                       rel="noopener"
                       title="View on GitHub">
                      <svg viewBox="0 0 16 16" class="w-5 h-5 transition-transform group-hover:scale-110" fill="currentColor">
                        <path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"></path>
                      </svg>
                    </a>
                  </div>
                  <p class="text-gray-600 dark:text-gray-200 mb-3 text-sm sm:text-base">{{ project.description }}</p>
                  <div class="flex items-center gap-3 flex-wrap">
                    <span class="inline-flex items-center gap-1.5 text-sm font-semibold text-blue-700 dark:text-blue-300 bg-blue-50 dark:bg-blue-900/40 px-3 py-1.5 rounded-full border border-blue-200 dark:border-blue-800">
                      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                      </svg>
                      {{ project.files.length }} files
                    </span>
                    <span v-if="project.total_size" class="inline-flex items-center gap-1.5 text-sm font-semibold text-green-700 dark:text-green-300 bg-green-50 dark:bg-green-900/40 px-3 py-1.5 rounded-full border border-green-200 dark:border-green-800">
                      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 7v10c0 2.21 3.582 4 8 4s8-1.79 8-4V7M4 7c0 2.21 3.582 4 8 4s8-1.79 8-4M4 7c0-2.21 3.582-4 8-4s8 1.79 8 4"/>
                      </svg>
                      {{ formatSize(project.total_size) }}
                    </span>
                    <span v-if="project.license" class="inline-flex items-center gap-1.5 text-sm font-semibold text-purple-700 dark:text-purple-300 bg-purple-50 dark:bg-purple-900/40 px-3 py-1.5 rounded-full border border-purple-200 dark:border-purple-800">
                      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"/>
                      </svg>
                      {{ project.license }}
                    </span>
                  </div>
                </div>
              </div>
            </div>

            <!-- File List -->
            <div class="divide-y divide-gray-200 dark:divide-gray-700">
              <div v-for="file in project.files" :key="file.name" class="transition-colors hover:bg-gray-50 dark:hover:bg-gray-800">
                <!-- File Item -->
                <div class="px-4 sm:px-6 py-4">
                  <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between gap-4">
                    <!-- File Info -->
                    <div class="flex items-start gap-3 sm:gap-4 flex-1 min-w-0">
                      <span class="text-2xl shrink-0 mt-0.5">📄</span>
                      <div class="flex flex-col gap-2 flex-1 min-w-0">
                        <div class="flex items-center gap-2 flex-wrap">
                          <span class="font-semibold text-gray-900 dark:text-white break-all">{{ file.name }}</span>
                          <span class="inline-flex items-center text-xs font-medium text-gray-600 dark:text-gray-200 bg-gray-100 dark:bg-gray-700 px-2 py-0.5 rounded">
                            {{ formatSize(file.size) }}
                          </span>
                        </div>
                        <div v-if="file.checksums && file.checksums.length > 0" class="flex gap-2 flex-wrap">
                          <button 
                            v-for="checksum in file.checksums" 
                            :key="checksum.name"
                            @click="toggleChecksum(file.name); loadChecksumContent(checksum.name, project.path)"
                            :class="['inline-flex items-center gap-1 text-xs font-medium px-2.5 py-1 rounded-lg border transition-all duration-200',
                                     isExpanded(file.name) ? 'bg-blue-600 text-white border-blue-600 shadow-md' : 'bg-white dark:bg-gray-800 text-gray-700 dark:text-gray-200 border-gray-300 dark:border-gray-600 hover:border-blue-500 dark:hover:border-blue-500 hover:bg-blue-50 dark:hover:bg-gray-700']"
                            :title="`Click to view ${checksum.name}`">
                            <svg v-if="!isExpanded(file.name)" class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"/>
                            </svg>
                            <svg v-else class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 12H4"/>
                            </svg>
                            {{ getChecksumType(checksum.name) }}
                          </button>
                        </div>
                        <code class="text-xs text-gray-500 dark:text-gray-400 font-mono break-all bg-gray-100 dark:bg-gray-800 px-2 py-1 rounded">{{ getFileUrl(file.name, project.path) }}</code>
                      </div>
                    </div>

                    <!-- File Actions -->
                    <div class="flex gap-2 shrink-0 self-end lg:self-center">
                      <button 
                        @click="copyLink(file.name, project.path)" 
                        :class="['inline-flex items-center justify-center gap-2 px-4 py-2 text-sm font-medium rounded-lg transition-all duration-200 min-w-[110px]',
                                 copiedFile === file.name ? 'bg-green-500 text-white shadow-md' : 'bg-white dark:bg-gray-800 text-blue-600 dark:text-blue-300 border border-blue-600 dark:border-blue-500 hover:bg-blue-50 dark:hover:bg-gray-700 shadow-sm hover:shadow']">
                        <svg v-if="copiedFile === file.name" class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/>
                        </svg>
                        <svg v-else class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"/>
                        </svg>
                        {{ copiedFile === file.name ? 'Copied!' : 'Copy Link' }}
                      </button>
                      <a 
                        :href="`/downloads/${project.path}/${file.name}`" 
                        class="inline-flex items-center justify-center gap-2 px-4 py-2 text-sm font-medium bg-blue-600 dark:bg-blue-600 text-white rounded-lg hover:bg-blue-700 dark:hover:bg-blue-700 transition-all duration-200 shadow-sm hover:shadow min-w-[110px]"
                        download>
                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"/>
                        </svg>
                        Download
                      </a>
                    </div>
                  </div>

                  <!-- Expanded Checksum Info -->
                  <transition
                    enter-active-class="transition ease-out duration-200"
                    enter-from-class="opacity-0 -translate-y-2"
                    enter-to-class="opacity-100 translate-y-0"
                    leave-active-class="transition ease-in duration-150"
                    leave-from-class="opacity-100 translate-y-0"
                    leave-to-class="opacity-0 -translate-y-2">
                    <div v-if="isExpanded(file.name) && file.checksums && file.checksums.length > 0" 
                         class="mt-4 ml-10 sm:ml-14 space-y-2">
                      <div v-for="checksum in file.checksums" :key="checksum.name" 
                           class="flex flex-col lg:flex-row lg:items-center gap-3 p-4 bg-blue-50 dark:bg-gray-800 rounded-lg border border-blue-200 dark:border-gray-600 shadow-sm">
                        <div class="flex items-center gap-3 flex-1 min-w-0">
                          <span class="inline-flex items-center justify-center text-xs font-bold text-white bg-blue-600 dark:bg-blue-700 px-3 py-1.5 rounded-md shadow-sm shrink-0 min-w-[70px]">
                            {{ getChecksumType(checksum.name) }}
                          </span>
                          <code class="text-xs text-gray-700 dark:text-gray-200 font-mono overflow-hidden text-ellipsis whitespace-nowrap select-all cursor-text hover:text-gray-900 dark:hover:text-white transition-colors flex-1 bg-white dark:bg-gray-900 px-3 py-2 rounded border border-gray-200 dark:border-gray-500">
                            {{ checksumContents[checksum.name] || 'Loading...' }}
                          </code>
                        </div>
                        <div class="flex gap-2 shrink-0">
                          <button 
                            @click="copyChecksumContent(checksum.name)" 
                            :class="['inline-flex items-center gap-1.5 px-3 py-2 text-xs font-medium rounded-lg transition-all duration-200 whitespace-nowrap',
                                     copiedFile === `checksum-${checksum.name}` ? 'bg-green-500 text-white shadow-sm' : 'bg-white dark:bg-gray-700 text-gray-700 dark:text-gray-100 border border-gray-300 dark:border-gray-500 hover:bg-gray-100 dark:hover:bg-gray-600 shadow-sm']">
                            <svg v-if="copiedFile === `checksum-${checksum.name}`" class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/>
                            </svg>
                            {{ copiedFile === `checksum-${checksum.name}` ? '✓ Copied' : 'Copy Hash' }}
                          </button>
                          <button 
                            @click="copyLink(checksum.name, project.path)" 
                            :class="['inline-flex items-center gap-1.5 px-3 py-2 text-xs font-medium rounded-lg transition-all duration-200 whitespace-nowrap',
                                     copiedFile === checksum.name ? 'bg-green-500 text-white shadow-sm' : 'bg-white dark:bg-gray-700 text-gray-700 dark:text-gray-100 border border-gray-300 dark:border-gray-500 hover:bg-gray-100 dark:hover:bg-gray-600 shadow-sm']">
                            <svg v-if="copiedFile === checksum.name" class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/>
                            </svg>
                            {{ copiedFile === checksum.name ? '✓ Copied' : 'Copy Link' }}
                          </button>
                        </div>
                      </div>
                    </div>
                  </transition>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- Footer -->
    <footer class="mt-auto px-4 sm:px-6 lg:px-8 py-8 bg-white dark:bg-gray-900 border-t border-gray-200 dark:border-gray-600 shadow-inner">
      <div class="max-w-7xl mx-auto">
        <div class="text-center space-y-3">
          <p class="text-gray-700 dark:text-gray-100 font-medium">
            Powered by <strong class="text-blue-600 dark:text-blue-400">Tencent Cloud EdgeOne Pages</strong>
          </p>
          <p class="text-sm text-gray-600 dark:text-gray-300">
            Mirrored: 
            <a href="https://github.com/MetaCubeX/meta-rules-dat" class="text-blue-600 dark:text-blue-400 hover:underline hover:text-blue-700 dark:hover:text-blue-300 transition-colors" target="_blank" rel="noopener">MetaCubeX/meta-rules-dat</a>
            <span class="mx-2">·</span>
            <a href="https://github.com/Loyalsoldier/clash-rules" class="text-blue-600 dark:text-blue-400 hover:underline hover:text-blue-700 dark:hover:text-blue-300 transition-colors" target="_blank" rel="noopener">Loyalsoldier/clash-rules</a>
            <span class="mx-2">·</span>
            <a href="https://github.com/mnixry/direct-android-ruleset" class="text-blue-600 dark:text-blue-400 hover:underline hover:text-blue-700 dark:hover:text-blue-300 transition-colors" target="_blank" rel="noopener">mnixry/direct-android-ruleset</a>
            <span class="mx-2">·</span>
            <a href="https://github.com/MetaCubeX/metacubexd" class="text-blue-600 dark:text-blue-400 hover:underline hover:text-blue-700 dark:hover:text-blue-300 transition-colors" target="_blank" rel="noopener">MetaCubeX/metacubexd</a>
          </p>
          <p class="text-sm text-gray-600 dark:text-gray-300">
            This Project: 
            <a href="https://github.com/djkcyl/rule-mirror" class="text-blue-600 dark:text-blue-400 hover:underline hover:text-blue-700 dark:hover:text-blue-300 transition-colors" target="_blank" rel="noopener">djkcyl/rule-mirror</a>
          </p>
        </div>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const projects = ref([])
const updatedAt = ref('')
const loading = ref(true)
const error = ref(null)
const copiedFile = ref(null)
const expandedFiles = ref(new Set())
const checksumContents = ref({})
const totalSize = ref('')
const totalFiles = ref(0)

const processFiles = (rawFiles) => {
  const checksumExtensions = ['.sha256sum', '.md5sum', '.sha512sum']
  const fileMap = new Map()
  const checksumMap = new Map()

  rawFiles.forEach(file => {
    const isChecksum = checksumExtensions.some(ext => file.name.endsWith(ext))
    
    if (isChecksum) {
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

  checksumMap.forEach((checksums, originalName) => {
    if (fileMap.has(originalName)) {
      fileMap.get(originalName).checksums = checksums
    }
  })

  return Array.from(fileMap.values())
}

const formatSize = (bytes) => {
  if (!bytes || bytes === 0) return '0 B'
  
  const units = ['B', 'KB', 'MB', 'GB', 'TB']
  const k = 1024
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  
  return `${(bytes / Math.pow(k, i)).toFixed(2)} ${units[i]}`
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
    
    projects.value = data.projects.map(project => ({
      ...project,
      files: processFiles(project.files || [])
    }))
    
    updatedAt.value = data.updated_at || ''
    
    // Calculate total files
    totalFiles.value = projects.value.reduce((sum, project) => sum + project.files.length, 0)
    
    // Calculate total size (sum of all project sizes in bytes)
    const totalBytes = projects.value.reduce((sum, project) => {
      return sum + (parseInt(project.total_size) || 0)
    }, 0)
    
    totalSize.value = formatSize(totalBytes)
  } catch (err) {
    error.value = err.message || 'Failed to load files'
    console.error('Error fetching files:', err)
  } finally {
    loading.value = false
  }
}

const getFileUrl = (filename, projectPath = '') => {
  const baseUrl = window.location.origin
  const path = projectPath ? `${projectPath}/` : ''
  return `${baseUrl}/downloads/${path}${filename}`
}

const getProjectUrl = (projectName) => {
  return `https://github.com/${projectName}`
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

const loadChecksumContent = async (filename, projectPath = '') => {
  if (checksumContents.value[filename]) return
  
  try {
    const path = projectPath ? `${projectPath}/` : ''
    const response = await fetch(`/downloads/${path}${filename}`)
    if (response.ok) {
      const content = await response.text()
      const trimmedContent = content.trim()
      const hashOnly = trimmedContent.split(/\s+/)[0]
      checksumContents.value[filename] = hashOnly || trimmedContent
    } else {
      checksumContents.value[filename] = 'Failed to load'
    }
  } catch (err) {
    console.error('Failed to load checksum:', err)
    checksumContents.value[filename] = 'Load error'
  }
}

const copyChecksumContent = async (filename) => {
  try {
    if (!checksumContents.value[filename]) {
      await loadChecksumContent(filename)
    }
    
    const content = checksumContents.value[filename]
    if (content && content !== 'Failed to load' && content !== 'Load error') {
      await navigator.clipboard.writeText(content)
      copiedFile.value = `checksum-${filename}`
      setTimeout(() => {
        copiedFile.value = null
      }, 2000)
    } else {
      alert('Unable to copy, checksum content not loaded')
    }
  } catch (err) {
    console.error('Failed to copy checksum:', err)
    alert('Copy failed, please try again')
  }
}

const copyLink = async (filename, projectPath = '') => {
  try {
    const url = getFileUrl(filename, projectPath)
    await navigator.clipboard.writeText(url)
    copiedFile.value = filename
    setTimeout(() => {
      copiedFile.value = null
    }, 2000)
  } catch (err) {
    console.error('Failed to copy:', err)
    alert('Copy failed, please try again')
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
@keyframes fade-in-up {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fade-in-up {
  animation: fade-in-up 0.6s ease-out forwards;
  opacity: 0;
}

.animation-delay-150 {
  animation-delay: 150ms;
}
</style>
