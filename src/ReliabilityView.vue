<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  records: { type: Array, default: () => [] }
})

const emit = defineEmits(['open-detail', 'open-create', 'open-edit'])

const searchQuery = ref('')
const selectedFilter = ref('testType')
const selectAll = ref(false)

const filterOptions = [
  { label: 'Reliability Test', value: 'testType' },
  { label: 'Prepared By', value: 'preparedBy' }
]

const filteredRecords = computed(() => {
  if (!searchQuery.value) return props.records
  const q = searchQuery.value.toLowerCase()
  return props.records.filter(r => {
    if (selectedFilter.value === 'testType') return r.testType?.toLowerCase().includes(q)
    if (selectedFilter.value === 'preparedBy') return r.preparedBy?.toLowerCase().includes(q)
    return false
  })
})

const numSelected = computed(() => props.records.filter(r => r.selected).length)

const toggleSelectAll = () => {
  props.records.forEach(r => r.selected = selectAll.value)
}

const openDetail = (record) => {
  emit('open-detail', record)
}

const openCreate = () => {
  emit('open-create')
}

const openEdit = (record) => {
  emit('open-edit', record)
}
</script>

<template>
  <div class="top-record-box no-margin custom-reliability-view">
    <div class="sub-header box-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span style="font-size: 14px; font-weight: bold; color: #333;">RELIABILITY RECORDS</span>
    </div>

    <div class="box-panel list-panel">
      <div class="list-inner-box">
        <!-- Action Toolbar -->
        <div class="action-row toolbar-row">
          <div class="action-bar no-margin" style="display: flex; gap: 15px; align-items: center;">
            <div class="dropdown-container">
              <span class="action-link">Actions</span>
            </div>
            <span class="action-link" @click.prevent="openCreate">Create New</span>
            <span class="selected-text" style="font-size: 11px; margin-left: 5px;">Selected: {{ numSelected }}</span>
          </div>
        </div>

        <!-- Search Toolbar -->
        <div class="search-row toolbar-row">
          <div class="search-bar no-margin" style="display: flex; gap: 10px; align-items: center;">
            <select class="search-select" v-model="selectedFilter" style="font-size: 11px; padding: 2px;">
              <option v-for="opt in filterOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
            </select>
            <input type="text" class="search-input" v-model="searchQuery" style="font-size: 11px; padding: 2px; height: 18px;" />
            <button class="search-btn" style="margin-left: 5px; font-size: 11px; padding: 2px 10px;">SEARCH</button>
          </div>
        </div>

        <!-- Data Table -->
        <div class="table-area" style="overflow-x: auto;">
          <table class="data-table">
            <thead>
              <tr>
                <th class="col-checkbox"><input type="checkbox" v-model="selectAll" @change="toggleSelectAll" /></th>
                <th class="col-icon"></th>
                <th>Reliability Test</th>
                <th>Plating Date</th>
                <th>Test Condition</th>
                <th>Prepared By</th>
                <th>Criteria</th>
              </tr>
            </thead>
            <tbody>
              <tr 
                v-for="record in filteredRecords" 
                :key="record.id" 
                :class="{ 'selected-row': record.selected }"
              >
                <td class="col-checkbox"><input type="checkbox" v-model="record.selected" /></td>
                <td class="col-icon">
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;" @click="openEdit(record)">
                    <path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/>
                  </svg>
                </td>
                <td><a href="#" class="item-link" @click.prevent="openDetail(record)">{{ record.testType }}</a></td>
                <td>{{ record.platingDate }}</td>
                <td style="white-space: pre-wrap;">{{ record.testCondition }}</td>
                <td>{{ record.preparedBy }}</td>
                <td style="white-space: pre-wrap;">{{ record.criteria }}</td>
              </tr>
              <tr v-if="filteredRecords.length === 0">
                <td colspan="7" style="text-align: center; padding: 20px; color: #666;">No reliability records found.</td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Pagination -->
        <div class="pagination-bar">
          <div class="page-controls">
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
            <span class="page-text">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M16 6v12h2V6zM6 18l8.5-6L6 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></span>
            <span class="display-text" style="color: #444; margin-left:10px;">Displaying 1 to {{ filteredRecords.length }} of {{ filteredRecords.length }} items</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.top-record-box {
  background-color: #fff;
  border: 2px solid #c7c7c7;
  margin-bottom: 20px;
}

.box-header {
  background-color: #c7c7c7;
  border-bottom: 2px solid #c7c7c7;
  margin: -2px -2px 0 -2px;
  padding: 10px;
}

.list-panel {
  padding: 15px;
  border: none;
}

.list-inner-box {
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 2px;
}

.toolbar-row {
  padding: 8px 15px;
  border-bottom: 1px solid #ddd;
}

.action-row {
  background-color: #f7f7f7;
}

.search-row {
  background-color: #ffffff;
}

.action-link:hover {
  color: #8f3235;
  text-decoration: none;
}

.selected-row td {
  background-color: #f2dfe1 !important;
}

.pagination-bar {
  background-color: #f7f7f7;
  border-top: 1px solid #ddd;
  padding: 4px 10px;
}

.page-controls {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 11px;
}

.p-btn {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.page-input {
  width: 35px;
  height: 18px;
  border: 1px solid #ccc;
  text-align: center;
  font-size: 11px;
}

.item-link {
  color: #0000EE;
  text-decoration: none;
}
.item-link:hover {
  text-decoration: underline;
}
</style>
