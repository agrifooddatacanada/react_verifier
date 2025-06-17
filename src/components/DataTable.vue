<template>
  <div class="data-table-container">
    <table class="data-table" v-if="tableData.length">
      <thead>
        <tr>
          <th v-for="header in headers" :key="header">{{ header }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in tableData" :key="index">
          <td v-for="header in headers" :key="header">{{ row[header] }}</td>
        </tr>
      </tbody>
    </table>
    <div v-else class="empty-state">
      <p>No data available to display</p>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  csvData: {
    type: String,
    default: '',
  },
})

const tableData = ref([])
const headers = ref([])

watch(
  () => props.csvData,
  (newData) => {
    if (newData) {
      parseCSV(newData)
    }
  },
  { immediate: true },
)

function parseCSV(csvString) {
  // Split the CSV string into lines
  const lines = csvString.split('\n')

  // Get headers from the first line
  headers.value = lines[0].split(',').map((header) => header.trim())

  // Parse the data rows
  tableData.value = lines.slice(1).map((line) => {
    const values = line.split(',').map((value) => value.trim())
    const row = {}
    headers.value.forEach((header, index) => {
      row[header] = values[index] || ''
    })
    return row
  })
}
</script>

<style scoped>
.data-table-container {
  width: 100%;
  overflow-x: auto;
  background: white;
  border-radius: 8px;
  box-shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1);
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.875rem;
}

.data-table th,
.data-table td {
  padding: 0.75rem 1rem;
  text-align: left;
  border-bottom: 1px solid #e5e7eb;
}

.data-table th {
  background-color: #f9fafb;
  font-weight: 600;
  color: #374151;
  white-space: nowrap;
}

.data-table td {
  color: #1f2937;
}

.data-table tbody tr:hover {
  background-color: #f9fafb;
}

.empty-state {
  padding: 2rem;
  text-align: center;
  color: #6b7280;
}

@media (max-width: 768px) {
  .data-table th,
  .data-table td {
    padding: 0.5rem;
    font-size: 0.75rem;
  }
}
</style>
