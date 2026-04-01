<script setup>
import { ref, computed } from 'vue'
import InspectionDetailView from './InspectionDetailView.vue'

const props = defineProps(['records'])
const emit = defineEmits(['open-edit', 'open-create', 'open-detail'])

const currentView = ref('list')
const selectedRecord = ref(null)
const isCreating = ref(false)

const openEdit = (record) => {
  emit('open-edit', record)
}

const internalRecords = computed(() => props.records || [])

const filterOptions = [
  { value: '', label: '--Please Select One--' },
  { value: 'product', label: 'Product' },
  { value: 'lotNo', label: 'Lot Number' },
  { value: 'paNo', label: 'PA Number' },
]

const selectedFilter = ref('')
const searchQuery = ref('')
const selectAll = ref(false)

const filteredRecords = computed(() => {
  if (!searchQuery.value) return internalRecords.value
  return internalRecords.value.filter(r => {
    const field = selectedFilter.value || 'product'
    return String(r[field] || '').toLowerCase().includes(searchQuery.value.toLowerCase())
  })
})

const numSelected = computed(() => internalRecords.value.filter(r => r.selected).length)

const toggleSelectAll = () => {
  filteredRecords.value.forEach(r => r.selected = selectAll.value)
}

const viewRecord = (r) => {
  emit('open-detail', r)
}

const createNewRecord = () => {
  emit('open-create')
}

const handleSave = (updated) => {
  // Logic moved to App.vue via events if needed, but for internal state:
  emit('save', updated)
}
</script>

<template>
  <div v-if="currentView === 'list'" class="top-record-box no-margin">
    <div class="sub-header box-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span style="font-size: 14px;">INSPECTION RECORDS</span>
    </div>

    <div class="box-panel">
      <div class="inner-box">
        <div class="action-row toolbar-row">
          <div class="action-bar no-margin" style="display: flex; gap: 15px; align-items: center;">
            <div class="dropdown-container">
              <span class="action-link">Actions</span>
            </div>
            <span class="action-link" @click.prevent="createNewRecord">Create New</span>
            <span class="selected-text" style="font-size: 11px; margin-left: 5px;">Selected: {{ numSelected }}</span>
          </div>
        </div>

        <div class="search-row toolbar-row">
          <div class="search-bar no-margin" style="display: flex; gap: 10px; align-items: center;">
            <select class="search-select" v-model="selectedFilter" style="font-size: 11px; padding: 2px;">
              <option v-for="opt in filterOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
            </select>
            <input type="text" class="search-input" v-model="searchQuery" style="margin-left: 5px; height: 18px;" />
            <button class="search-btn" style="margin-left: 5px; font-size: 11px; padding: 2px 10px;">SEARCH</button>
          </div>
        </div>

        <div class="table-area" style="overflow-x: auto;">
          <table class="data-table">
            <thead>
              <tr>
                <th class="col-checkbox"><input type="checkbox" v-model="selectAll" @change="toggleSelectAll" /></th>
                <th class="col-icon"></th>
                <th>Product Type</th>
                <th>Product</th>
                <th>Product Serial Number</th>
                <th>Revision</th>
                <th>DWG Number</th>
                <th>Lot Number</th>
                <th>Coating Date</th>
                <th>Coating Line</th>
                <th>PA Number</th>
                <th>JPN Lot No</th>
                <th>Measurement Value Method</th>
                <th>First Run</th>
                <th>Remarks</th>
                <th>Creation Date</th>
                <th>Created By</th>
                <th>Updated Date</th>
                <th>Updated By</th>
                <th>Final Results</th>
              </tr>
            </thead>
            <tbody>
              <tr 
                v-for="(r, idx) in filteredRecords" 
                :key="idx" 
                :class="{ 'selected-row': r.selected }"
              >
                <td class="col-checkbox"><input type="checkbox" v-model="r.selected" /></td>
                <td class="col-icon">
                  <svg @click="openEdit(r)" viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                </td>
                <td>{{ r.productType }}</td>
                <td><a href="#" class="item-link" @click.prevent="viewRecord(r)">{{ r.product }}</a></td>
                <td>{{ r.serialNo }}</td>
                <td>{{ r.revision }}</td>
                <td>{{ r.dwgNo }}</td>
                <td>{{ r.lotNo }}</td>
                <td>{{ r.cDate }}</td>
                <td>{{ r.cLine }}</td>
                <td>{{ r.paNo }}</td>
                <td>{{ r.jpnLot }}</td>
                <td>{{ r.mMethod }}</td>
                <td>{{ r.firstRun }}</td>
                <td>{{ r.remarks }}</td>
                <td>{{ r.creationDate }}</td>
                <td>{{ r.createdBy }}</td>
                <td>{{ r.updatedDate }}</td>
                <td>{{ r.updatedBy }}</td>
                <td>{{ r.finalResult }}</td>
              </tr>
              <tr v-if="filteredRecords.length === 0">
                <td colspan="20" style="text-align: center; padding: 20px; color: #666;">No inspection records found.</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="pagination-bar">
          <div class="page-controls">
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
            <span class="page-text" style="font-size: 11px;">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M16 6v12h2V6zM6 18l8.5-6L6 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></span>
            <span class="display-text" style="font-size: 11px;">Displaying 1 to {{ filteredRecords.length }} of {{ filteredRecords.length }} items</span>
          </div>
        </div>
      </div>
    </div>
  </div>
  <InspectionDetailView v-else :record="selectedRecord" :isCreating="isCreating" @back="currentView = 'list'; selectedRecord = null; isCreating = false" @save="handleSave" />
</template>

<style scoped>
.top-record-box {
  background-color: #fff;
  border: 2px solid #c7c7c7;
  margin: 15px;
  overflow: hidden;
}
.box-header {
  margin: -2px -2px 0 -2px;
}
.toolbar-row {
  padding: 8px 15px;
  border-bottom: 1px solid #ddd;
}
.action-row {
  background-color: #f7f7f7;
}
.action-bar {
  display: flex;
  gap: 15px;
  align-items: center;
}
.action-link {
  color: #333;
  text-decoration: none;
  cursor: pointer;
  font-size: 11px;
}
.action-link:hover {
  text-decoration: underline;
}
.search-row {
  background-color: #fff;
}
.search-bar {
  display: flex;
  gap: 10px;
  align-items: center;
}
.search-select, .page-input {
  border: 1px solid #ccc;
  border-radius: 2px;
  font-size: 11px;
}
.selected-text {
  font-size: 11px;
  color: #444;
}
.table-area {
  overflow-x: auto;
}
.item-link {
  color: blue;
  text-decoration: none;
}
.item-link:hover {
  text-decoration: underline;
}
.selected-row td {
  background-color: #f2dfe1 !important;
}
</style>
