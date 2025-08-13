<template>
  <div class="container">
    <header class="top-header">
      <img class="logo-left" src="/src/assets/UoG_logo.png" alt="University of Guelph" />
      <img class="logo-right" src="/src/assets/agrifood.png" alt="Agri-Food Data Canada" />
    </header>

    <div class="main-content">
      <div class="upload-card">
        <div class="card-header">
          <!-- Upload icon -->
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="18"
            height="18"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
            class="card-header-icon"
          >
            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
            <polyline points="17 8 12 3 7 8" />
            <line x1="12" y1="3" x2="12" y2="15" />
          </svg>
          <span style="font-weight: bold">Upload</span>
        </div>
        <div class="card-body">
          <p class="disclaimer">Disclaimer placeholder will go here</p>

          <div class="form-field">
            <label class="field-label">Select a Local Schema</label>
            <div class="select-wrapper">
              <select v-model="selectedSchema" class="select">
                <option disabled value="">Select a Schema</option>
                <option v-for="schema in schemas" :key="schema.id" :value="schema.path">
                  {{ schema.name }}
                </option>
              </select>
              <!-- Dropdown arrow -->
              <svg
                class="select-caret"
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
                <polyline points="6 9 12 15 18 9" />
              </svg>
            </div>
          </div>

          <button class="submit-button" :disabled="!selectedSchema" @click="submitSchema">
            Submit
          </button>
        </div>
      </div>

      <ValidatorCard
        v-if="showValidator"
        :filePath="selectedFile"
        @send-file="handleVerifiedData"
        @close="showValidator = false"
      />

      <div class="table-container" v-if="csvData">
        <DataTable :csvData="csvData" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import DataTable from './components/DataTable.vue'
import ValidatorCard from './components/ValidatorCard.vue'
import schemaConfig from './assets/config/schemas.json'

const selectedFile = ref('')
const csvData = ref('')
const showValidator = ref(false)

const schemas = ref([])
const selectedSchema = ref('')

onMounted(() => {
  schemas.value = schemaConfig.schemas || []
})

function submitSchema() {
  if (selectedSchema.value) {
    csvData.value = ''
    selectedFile.value = selectedSchema.value
    showValidator.value = true
  }
}

function handleVerifiedData(file) {
  if (file) {
    csvData.value = file
  }
}
</script>

<style scoped>
.container {
  background-color: #f9fafb;
  min-height: 100vh;
}

.top-header {
  background-color: #f8fafc;
  border-bottom: 1px solid #e5e7eb;
  padding: 1rem 1.5rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.logo-left,
.logo-right {
  height: 84px;
  width: auto;
  object-fit: contain;
}

.main-content {
  width: 100%;
}

.upload-card {
  width: 95%;
  margin: 1.25rem auto;
  padding: 0 1.5rem;
  background-color: white;
  border: 1px solid #e5e7eb;
  border-top: 3px solid #dc2626;
  border-radius: 8px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  overflow: hidden;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid #e5e7eb;
  color: #111827;
  font-weight: 500;
}

.card-body {
  padding: 1rem;
}

.disclaimer {
  margin: 0 0 1rem 0;
  color: #4b5563;
  font-size: 0.875rem;
}

.form-field {
  margin-bottom: 1rem;
}

.field-label {
  display: block;
  margin-bottom: 0.5rem;
  color: #374151;
  font-weight: 600;
  font-size: 0.875rem;
}

.select-wrapper {
  position: relative;
}

.select {
  width: 100%;
  appearance: none;
  padding: 0.625rem 2.5rem 0.625rem 0.75rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  background-color: #ffffff;
  color: #111827;
  font-size: 0.875rem;
}

.select-caret {
  position: absolute;
  right: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  color: #6b7280;
  pointer-events: none;
}

.submit-button {
  background-color: #f3f4f6;
  color: #111827;
  border: 1px solid #d1d5db;
  padding: 0.5rem 1rem;
  border-radius: 0.375rem;
  font-size: 0.875rem;
  cursor: pointer;
}

.submit-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.table-container {
  background-color: white;
  border-radius: 12px;
  min-height: 400px;
  padding: 1.5rem;
  margin-top: 1.25rem;
}

@media (max-width: 768px) {
  .logo-left,
  .logo-right {
    height: 56px;
  }

  .main-content {
    padding: 0 1rem;
  }
}
</style>
