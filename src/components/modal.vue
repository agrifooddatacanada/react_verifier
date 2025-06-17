<template>
  <Transition name="modal">
    <div class="modal-overlay" @click.self="$emit('close')">
      <div class="modal-content">
        <div class="modal-header">
          <div class="header-content">
            <h2>{{ fileName }}</h2>
            <span class="file-type">{{ getFileType }}</span>
          </div>
          <button class="close-button" @click="$emit('close')" aria-label="Close modal">
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
        <div class="modal-body">
          <div v-if="isLoading" class="loading-container">
            <div class="loading-spinner"></div>
            <p>Loading validator...</p>
          </div>
          <div class="iframe-container" :class="{ loading: isLoading }">
            <iframe
              ref="iframeRef"
              :src="'http://localhost:3000/oca-data-validator'"
              @load="handleIframeLoad"
            ></iframe>
          </div>
        </div>
      </div>
    </div>
  </Transition>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const props = defineProps({
  filePath: {
    type: String,
    required: true,
  },
})

const emit = defineEmits(['close', 'send-file'])

const iframeRef = ref(null)
const fileContent = ref('')
const isLoading = ref(true)

const fileName = computed(() => {
  return props.filePath.split('/').pop()
})

const getFileType = computed(() => {
  const extension = props.filePath.split('.').pop()?.toUpperCase()
  return extension ? `.${extension}` : ''
})

async function loadAndSendFile() {
  try {
    const response = await fetch(props.filePath)
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    const file = await response.json()
    fileContent.value = file
    // Wait for iframe to load
    iframeRef.value.onload = () => {
      // Send the file content to the iframe
      iframeRef.value.contentWindow.postMessage(
        {
          type: 'JSON_SCHEMA',
          data: file, // Send the JSON data directly
        },
        '*',
      )
    }
  } catch (error) {
    console.error('Error loading file:', error)
  }
}

async function receiveData(event) {
  if (event.origin !== 'http://localhost:3000') {
    return
  }

  if (event.data.type === 'VERIFIED_DATA') {
    let csvData = event.data.data

    emit('send-file', csvData)
    emit('close')

    // console.log('csvData: ', csvData)
    // const blob = new Blob([csvData], { type: 'text/csv' })

    // const url = URL.createObjectURL(blob)

    // const a = document.createElement('a')
    // a.href = url
    // a.download = fileName || 'validatedData.csv'
    // document.body.appendChild(a)
    // a.click()

    // // clean up
    // document.body.removeChild(a)
    // URL.revokeObjectURL(url)
  }
}

function handleIframeLoad() {
  isLoading.value = false
  if (fileContent.value) {
    iframeRef.value.contentWindow.postMessage(
      {
        type: 'JSON_SCHEMA',
        data: fileContent.value,
      },
      '*',
    )
  }
}

// Watch for filePath changes
watch(
  () => props.filePath,
  () => {
    if (props.filePath) {
      loadAndSendFile()
    }
  },
)

onMounted(() => {
  if (props.filePath) {
    loadAndSendFile()
  }
  window.addEventListener('message', receiveData)
})
</script>

<style scoped>
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(4px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  border-radius: 12px;
  width: 95%;
  max-width: 1500px;
  height: 90vh;
  max-height: 90vh;
  overflow: hidden;
  box-shadow:
    0 20px 25px -5px rgb(0 0 0 / 0.1),
    0 8px 10px -6px rgb(0 0 0 / 0.1);
  display: flex;
  flex-direction: column;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.25rem 1.5rem;
  border-bottom: 1px solid #e5e7eb;
  background-color: #f9fafb;
}

.header-content {
  display: flex;
  align-items: baseline;
  gap: 0.75rem;
}

.header-content h2 {
  margin: 0;
  font-size: 1.25rem;
  font-weight: 600;
  color: #111827;
}

.file-type {
  font-size: 0.875rem;
  color: #6b7280;
  font-weight: 500;
}

.close-button {
  background: none;
  border: none;
  padding: 0.5rem;
  cursor: pointer;
  color: #6b7280;
  border-radius: 0.375rem;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.close-button:hover {
  background-color: #f3f4f6;
  color: #111827;
}

.modal-body {
  flex: 1;
  position: relative;
  padding: 1.5rem;
  background-color: white;
}

.iframe-container {
  width: 100%;
  height: 100%;
  border-radius: 8px;
  overflow: hidden;
  opacity: 1;
  transition: opacity 0.3s ease;
}

.iframe-container.loading {
  opacity: 0;
}

iframe {
  width: 100%;
  height: 100%;
  border: none;
  border-radius: 8px;
}

.loading-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #f3f4f6;
  border-top-color: #3b82f6;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@media (max-width: 640px) {
  .modal-content {
    width: 100%;
    height: 100vh;
    max-height: 100vh;
    border-radius: 0;
  }

  .modal-header {
    padding: 1rem;
  }

  .header-content h2 {
    font-size: 1.125rem;
  }

  .modal-body {
    padding: 1rem;
  }
}
</style>
