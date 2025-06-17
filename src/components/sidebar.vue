<template>
  <aside class="sidebar" :class="{ 'mobile-open': isMobileOpen }">
    <div class="sidebar-header">
      <div class="header-content">
        <h1>OCA Data Verifier</h1>
        <p class="subtitle">Verify and process your OCA bundle files</p>
      </div>
      <button class="mobile-close" @click="$emit('close')" v-if="isMobile">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <line x1="18" y1="6" x2="6" y2="18"></line>
          <line x1="6" y1="6" x2="18" y2="18"></line>
        </svg>
      </button>
    </div>

    <div class="sidebar-content">
      <div class="search-container">
        <div class="search-input-wrapper">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="16"
            height="16"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
            class="search-icon"
          >
            <circle cx="11" cy="11" r="8"></circle>
            <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
          </svg>
          <input
            type="text"
            v-model="searchQuery"
            placeholder="Search files..."
            class="search-input"
            @input="filterFiles"
          />
        </div>
      </div>

      <div class="file-section">
        <h2 class="section-title">Available Files</h2>
        <div class="file-list">
          <div
            v-for="file in filteredFiles"
            :key="file.id"
            class="file-item"
            :class="{ active: selectedFile?.id === file.id }"
            @click="handleFileClick(file)"
          >
            <div class="file-icon">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="20"
                height="20"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
              >
                <path
                  d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2z"
                ></path>
                <polyline points="14 2 14 8 20 8"></polyline>
              </svg>
            </div>
            <div class="file-info">
              <span class="file-name">{{ file.name }}</span>
              <span class="file-type">{{ getFileType(file.name) }}</span>
            </div>
            <div class="file-status" v-if="selectedFile?.id === file.id">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="16"
                height="16"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
              >
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
          </div>
        </div>
      </div>

      <div class="selected-file-info" v-if="selectedFile">
        <div class="info-header">
          <h3>Selected File</h3>
        </div>
        <div class="info-content">
          <div class="info-item">
            <span class="info-label">Name:</span>
            <span class="info-value">{{ selectedFile.name }}</span>
          </div>
          <div class="info-item">
            <span class="info-label">Type:</span>
            <span class="info-value">{{ getFileType(selectedFile.name) }}</span>
          </div>
          <button class="validate-button" @click="handleValidate">
            Verify Data
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <polyline points="9 11 12 14 22 4"></polyline>
              <path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"></path>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </aside>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  isMobile: {
    type: Boolean,
    default: false,
  },
  isMobileOpen: {
    type: Boolean,
    default: false,
  },
})

const emit = defineEmits(['close', 'select-file', 'open-modal'])

const selectedFile = ref(null)
const searchQuery = ref('')
const filteredFiles = ref([])

const files = ref([
  { id: 'file3', name: 'Carcass_Schema.json', path: 'src/assets/files/CSV_OCA_package.json' },
  { id: 'file4', name: 'Soil_Schema.json', path: 'src/assets/files/soil_package_1.json' },
])

// Initialize filtered files
filteredFiles.value = [...files.value]

function getFileType(fileName) {
  const extension = fileName.split('.').pop()?.toUpperCase()
  return extension ? `.${extension}` : ''
}

function filterFiles() {
  if (!searchQuery.value) {
    filteredFiles.value = [...files.value]
    return
  }

  const query = searchQuery.value.toLowerCase()
  filteredFiles.value = files.value.filter((file) => file.name.toLowerCase().includes(query))
}

function handleFileClick(file) {
  selectedFile.value = file
  emit('select-file', file)
}

function handleValidate() {
  if (selectedFile.value) {
    emit('open-modal', selectedFile.value)
  }
}
</script>

<style scoped>
.sidebar {
  background-color: white;
  border-right: 1px solid #e5e7eb;
  height: 100%;
  left: 0;
  top: 0;
  bottom: 0;
  display: flex;
  flex-direction: column;
  transition: transform 0.3s ease;
  z-index: 50;
  padding: 1rem;
}

.sidebar-header {
  padding: 1.25rem;
  border-bottom: 1px solid #e5e7eb;
  background-color: #f8fafc;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.header-content {
  flex: 1;
}

.header-content h1 {
  margin: 0;
  font-size: 1.25rem;
  font-weight: 600;
  color: #111827;
  line-height: 1.2;
}

.subtitle {
  margin: 0.25rem 0 0;
  color: #6b7280;
  font-size: 0.875rem;
}

.mobile-close {
  background: none;
  border: none;
  padding: 0.5rem;
  cursor: pointer;
  color: #6b7280;
  border-radius: 0.375rem;
  display: none;
}

.mobile-close:hover {
  background-color: #f3f4f6;
  color: #111827;
}

.sidebar-content {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.search-container {
  padding: 0 0.5rem;
}

.search-input-wrapper {
  position: relative;
}

.search-icon {
  position: absolute;
  left: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  color: #9ca3af;
}

.search-input {
  width: 80%;
  padding: 0.625rem 0.625rem 0.625rem 2.25rem;
  border: 1px solid #e5e7eb;
  border-radius: 0.5rem;
  font-size: 0.875rem;
  color: #374151;
  background-color: #f9fafb;
  transition: all 0.2s ease;
}

.search-input:focus {
  outline: none;
  border-color: #3b82f6;
  background-color: white;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.file-section {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.section-title {
  font-size: 0.875rem;
  font-weight: 600;
  color: #4b5563;
  padding: 0 0.5rem;
}

.file-list {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.file-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem;
  border-radius: 0.5rem;
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
}

.file-item:hover {
  background-color: #f9fafb;
}

.file-item.active {
  background-color: #eff6ff;
}

.file-icon {
  color: #6b7280;
  display: flex;
  align-items: center;
}

.file-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.file-name {
  font-size: 0.875rem;
  font-weight: 500;
  color: #111827;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.file-type {
  font-size: 0.75rem;
  color: #6b7280;
}

.file-status {
  color: #3b82f6;
  display: flex;
  align-items: center;
}

.selected-file-info {
  background-color: #f8fafc;
  border: 1px solid #e5e7eb;
  border-radius: 0.5rem;
  overflow: hidden;
}

.info-header {
  padding: 0.75rem 1rem;
  background-color: #f1f5f9;
  border-bottom: 1px solid #e5e7eb;
}

.info-header h3 {
  margin: 0;
  font-size: 0.875rem;
  font-weight: 600;
  color: #4b5563;
}

.info-content {
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.info-item {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.info-label {
  font-size: 0.75rem;
  color: #6b7280;
}

.info-value {
  font-size: 0.875rem;
  color: #111827;
  font-weight: 500;
}

.validate-button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  width: 100%;
  padding: 0.625rem;
  background-color: #3b82f6;
  color: white;
  border: none;
  border-radius: 0.375rem;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.validate-button:hover {
  background-color: #2563eb;
}

.validate-button:active {
  background-color: #1d4ed8;
}

@media (max-width: 768px) {
  .sidebar {
    transform: translateX(-100%);
  }

  .sidebar.mobile-open {
    transform: translateX(0);
  }

  .mobile-close {
    display: block;
  }
}
</style>
