<template>
  <div class="validator-card">
    <div class="card-body">
      <div v-if="isLoading" class="loading-container">
        <div class="loading-spinner"></div>
        <p>Loading validator...</p>
      </div>
      <div class="iframe-container" :class="{ loading: isLoading }">
        <iframe ref="iframeRef" :src="validatorUrl" @load="handleIframeLoad"></iframe>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const props = defineProps({
  filePath: {
    type: String,
    required: true,
  },
  validatorUrl: {
    type: String,
    default: 'http://localhost:3000/oca-data-validator',
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
  } catch (error) {
    console.error('Error loading file:', error)
  }
}

function receiveData(event) {
  if (event.origin !== 'http://localhost:3000') {
    return
  }

  if (event.data.type === 'VERIFIED_DATA') {
    const csvData = event.data.data
    emit('send-file', csvData)
    emit('close')
  }
}

function handleIframeLoad() {
  isLoading.value = false
  if (fileContent.value && iframeRef.value?.contentWindow) {
    iframeRef.value.contentWindow.postMessage(
      {
        type: 'JSON_SCHEMA',
        data: JSON.parse(JSON.stringify(fileContent.value)),
      },
      '*',
    )
  }
}

watch(
  () => props.filePath,
  () => {
    if (props.filePath) {
      loadAndSendFile()
    }
  },
  { immediate: true },
)

onMounted(() => {
  if (props.filePath) {
    loadAndSendFile()
  }
  window.addEventListener('message', receiveData)
})
</script>

<style scoped>
.validator-card {
  width: 95%;
  height: 100vh;
  margin: 1.25rem auto;
  padding: 0 1.5rem;
  background-color: white;
  border: 1px solid #e5e7eb;
  border-top: 3px solid #dc2626;
  border-radius: 8px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  overflow: hidden;
}

.close-button {
  background: none;
  border: 1px solid #e5e7eb;
  border-radius: 0.375rem;
  padding: 0.375rem;
  cursor: pointer;
  color: #6b7280;
}

.card-body {
  padding: 1rem;
  height: 100%;
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
  display: flex;
  align-items: center;
  gap: 0.75rem;
  color: #6b7280;
  margin-bottom: 0.75rem;
}

.loading-spinner {
  width: 20px;
  height: 20px;
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
</style>
