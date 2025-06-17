<template>
  <div class="container">
    <Sidebar @select-file="handleFileSelect" @close="handleClose" @open-modal="openModal" />

    <div class="main-content">
      <div class="content-header">
        <div class="header-content">
          <div class="header-text">
            <h2>Data Preview</h2>
            <p class="content-description">View and analyze your OCA bundle data</p>
          </div>
          <!-- <button class="upload-button">Upload Data</button> -->
        </div>
      </div>

      <div class="table-container">
        <DataTable :csvData="csvData" />
      </div>
    </div>

    <Modal
      v-if="isModalOpen"
      :filePath="selectedFile"
      @close="closeModal"
      @send-file="handleVerifiedData"
    ></Modal>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Modal from './components/modal.vue'
import Sidebar from './components/sidebar.vue'
import DataTable from './components/DataTable.vue'

const isModalOpen = ref(false)
const selectedFile = ref('')
const csvData = ref('')

function handleFileSelect() {
  if (selectedFile.value) {
    isModalOpen.value = true
  }
}

function handleVerifiedData(file) {
  console.log('file: ', file)
  if (file) {
    csvData.value = file
  }
}

function closeModal() {
  isModalOpen.value = false
  selectedFile.value = '' // Reset selection when modal is closed
}

function openModal(file) {
  selectedFile.value = file.path
  isModalOpen.value = true
}
</script>

<style scoped>
.container {
  background-color: #f9fafb;
  flex-grow: auto;
  display: flex;
  width: 100%;
  height: 100vh;
}

.main-content {
  flex: 1;
  padding: 2rem;
  overflow-y: auto;
  background-color: white;
  border-radius: 12px 0 0 12px;
  box-shadow: -4px 0 6px -1px rgb(0 0 0 / 0.1);
  margin-left: 0;
}

.content-header {
  margin-bottom: 2rem;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.header-text {
  flex: 1;
}

.upload-button {
  background-color: #2563eb;
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 0.375rem;
  font-size: 0.875rem;
  font-weight: 500;
  border: none;
  cursor: pointer;
  transition: background-color 0.2s;
  white-space: nowrap;
}

.upload-button:hover {
  background-color: #1d4ed8;
}

.upload-button:focus {
  outline: 2px solid #93c5fd;
  outline-offset: 2px;
}

.content-header h2 {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
  color: #111827;
}

.content-description {
  margin: 0.5rem 0 0;
  color: #6b7280;
  font-size: 0.875rem;
}

.table-container {
  background-color: white;
  border-radius: 12px;
  min-height: 400px;
  padding: 1.5rem;
}

.empty-state {
  height: 100%;
  min-height: 400px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #6b7280;
  text-align: center;
  gap: 1rem;
}

.empty-icon {
  color: #9ca3af;
}

/* Keep existing styles for dropdown, file-info, etc. */
.dropdown-container {
  margin-bottom: 0;
}

.select-wrapper {
  position: relative;
}

.select-icon {
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
  color: #6b7280;
}

@media (max-width: 768px) {
  .main-content {
    margin-left: 0;
    padding: 1rem;
    border-radius: 0;
  }

  .content-header h2 {
    font-size: 1.25rem;
  }

  .table-container {
    min-height: 300px;
  }
}
</style>
