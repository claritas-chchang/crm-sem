<script setup>
import { ref, computed } from 'vue'
import ProductView from './ProductView.vue'
import ValidationView from './ValidationView.vue'
import InspectionView from './InspectionView.vue'
import SamplingLevelView from './SamplingLevelView.vue'

const currentModule = ref('product')

const products = ref([
  { id: 1, code: 'PNO000496', type: 'Internal Ver Lab (Other Plant)', selected: false },
  { id: 2, code: 'PNO000495', type: 'Internal Ver Lab (Other Plant)', selected: false },
  { id: 3, code: 'PNO000494', type: 'Internal Ver Lab (Other Plant)', selected: false },
  { id: 4, code: 'PNO000493', type: 'Internal Ver Lab (Other Plant)', selected: false },
  { id: 5, code: 'PNO000491', type: 'Internal Ver Lab (Other Plant)', selected: false },
  { id: 6, code: 'PNO000489', type: 'Internal Ver Lab (Other Plant)', selected: false },
  { id: 7, code: 'PNO000486', type: 'Internal Ver Lab (Other Plant)', selected: false },
  { id: 8, code: 'PNO000482', type: 'Internal Ver Lab (Other Plant)', selected: false },
])

const filterOptions = [
  { value: '', label: '--Please Select One--' },
  { value: 'code', label: 'Product ID/Code' },
  { value: 'type', label: 'Product Type' },
]

const selectedFilter = ref('')
const searchQuery = ref('')
const actionsDropdownOpen = ref(false)

const filteredProducts = computed(() => {
  if (!searchQuery.value) return products.value
  return products.value.filter(p => {
    if (selectedFilter.value === 'code') {
      return p.code.toLowerCase().includes(searchQuery.value.toLowerCase())
    } else if (selectedFilter.value === 'type') {
      return p.type.toLowerCase().includes(searchQuery.value.toLowerCase())
    } else {
      return p.code.toLowerCase().includes(searchQuery.value.toLowerCase()) || 
             p.type.toLowerCase().includes(searchQuery.value.toLowerCase())
    }
  })
})

const numSelected = computed(() => products.value.filter(p => p.selected).length)

const selectAll = ref(false)
const toggleSelectAll = () => {
  filteredProducts.value.forEach(p => p.selected = selectAll.value)
}

const toggleActionDropdown = () => {
  actionsDropdownOpen.value = !actionsDropdownOpen.value
}

const deleteSelected = () => {
  const selectedCount = products.value.filter(p => p.selected).length
  if (selectedCount === 0) return
  
  if (confirm(`Are you sure you want to delete ${selectedCount} selected product(s)?`)) {
    products.value = products.value.filter(p => !p.selected)
    actionsDropdownOpen.value = false
  }
}

const createNew = () => {
  selectedProductCode.value = ''
  currentView.value = 'create'
}

const currentView = ref('list')
const selectedProductCode = ref('')

const viewProduct = (code) => {
  selectedProductCode.value = code
  currentView.value = 'view'
}

const backToList = () => {
  currentView.value = 'list'
  selectedProductCode.value = ''
}

const editProduct = (code) => {
  alert(`Editing product: ${code}`)
}

const toggleRowSelection = (item) => {
  item.selected = !item.selected
}
</script>

<template>
  <div class="app-container">
    <header class="top-header">
      <div class="logo" style="font-size: 26px; color: #fff; font-family:'Lobster Two', cursive; letter-spacing:2px;">Calibration Management System</div>
      <div class="user-controls">
        <span class="username">qa-admin-p2</span>
        <div class="toggle-switch">
          <div class="toggle-knob"></div>
        </div>
        <div class="icon bell-icon">
          <svg viewBox="0 0 24 24" width="16" height="16" fill="white"><path d="M12 22c1.1 0 2-.9 2-2h-4c0 1.1.89 2 2 2zm6-6v-5c0-3.07-1.64-5.64-4.5-6.32V4c0-.83-.67-1.5-1.5-1.5s-1.5.67-1.5 1.5v.68C7.63 5.36 6 7.92 6 11v5l-2 2v1h16v-1l-2-2z"/></svg>
        </div>
        <div class="icon user-icon">
          <svg viewBox="0 0 24 24" width="16" height="16" fill="white"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 3c1.66 0 3 1.34 3 3s-1.34 3-3 3-3-1.34-3-3 1.34-3 3-3zm0 14.2c-2.5 0-4.71-1.28-6-3.22.03-1.99 4-3.08 6-3.08 1.99 0 5.97 1.09 6 3.08-1.29 1.94-3.5 3.22-6 3.22z"/></svg>
        </div>
      </div>
    </header>

    <nav class="main-nav">
      <a href="#" :class="{ active: currentModule === 'validation' }" @click.prevent="currentModule = 'validation'; currentView = 'list'">VALIDATION</a>
      <a href="#" :class="{ active: currentModule === 'inspection' }" @click.prevent="currentModule = 'inspection'; currentView = 'list'">INSPECTION</a>
      <a href="#">ERASURE</a>
      <a href="#">RELIABILITY</a>
      <a href="#" :class="{ active: currentModule === 'samplingLevel' }" @click.prevent="currentModule = 'samplingLevel'; currentView = 'list'">SAMPLING LEVEL</a>
      <a href="#" :class="{ active: currentModule === 'product' }" @click.prevent="currentModule = 'product'; currentView = 'list'">PRODUCT RECORDS</a>
      <div class="nav-item-dropdown">
        <a href="#">REPORT</a>
        <div class="dropdown-content">
          <a href="#" class="dropdown-item">Shipment Type A</a>
          <a href="#" class="dropdown-item">Shipment Type B</a>
        </div>
      </div>
      <a href="#">MAGNETIC PROPERTIES</a>
      <a href="#">CALIBRATION</a>
    </nav>

    <main class="content">
      <!-- VALIDATION MODULE -->
      <ValidationView v-if="currentModule === 'validation'" />

      <!-- INSPECTION MODULE -->
      <InspectionView v-else-if="currentModule === 'inspection'" />

      <!-- SAMPLING LEVEL MODULE -->
      <SamplingLevelView v-else-if="currentModule === 'samplingLevel'" />

      <!-- PRODUCT RECORDS MODULE -->
      <template v-else-if="currentModule === 'product'">
        <div v-if="currentView === 'list'" class="top-record-box">
        <!-- Box Header -->
        <div class="sub-header box-header">
          <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
          <span style="font-size: 14px;">PRODUCT RECORDS</span>
        </div>

        <div class="panel list-panel">
          <div class="list-inner-box">
            <!-- Row 1: Actions -->
            <div class="action-row toolbar-row">
              <div class="action-bar no-margin">
                <div class="dropdown-container">
                  <span class="action-link has-dropdown" @click="toggleActionDropdown">
                    Actions
                  </span>
                  <div class="dropdown-menu" v-if="actionsDropdownOpen">
                    <div class="dropdown-item" @click="deleteSelected">Delete Selected</div>
                  </div>
                </div>
                <span class="action-link" @click="createNew">Create New</span>
                <span class="selected-text">Selected: {{ numSelected }}</span>
              </div>
            </div>

            <!-- Row 2: Search -->
            <div class="search-row toolbar-row">
              <div class="search-bar no-margin">
                <select class="search-select" v-model="selectedFilter">
                  <option v-for="opt in filterOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
                </select>
                <input type="text" class="search-input" v-model="searchQuery" style="margin-left: 5px;" />
                <button class="search-btn" style="margin-left: 5px;">SEARCH</button>
              </div>
            </div>

            <!-- Row 3: Table -->
            <div class="table-area">
              <table class="data-table">
                <thead>
                  <tr>
                    <th class="col-checkbox"><input type="checkbox" v-model="selectAll" @change="toggleSelectAll" /></th>
                    <th class="col-icon"></th>
                    <th>Product ID/Code</th>
                    <th>Product Type</th>
                  </tr>
                </thead>
                <tbody>
                  <tr 
                    v-for="item in filteredProducts" 
                    :key="item.id" 
                    :class="{ 'selected-row': item.selected }"
                  >
                    <td class="col-checkbox"><input type="checkbox" v-model="item.selected" /></td>
                    <td class="col-icon">
                      <svg @click="editProduct(item.code)" viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                    </td>
                    <td><a href="#" class="item-link" @click.prevent="viewProduct(item.code)">{{ item.code }}</a></td>
                    <td>{{ item.type }}</td>
                  </tr>
                  <tr v-if="filteredProducts.length === 0">
                    <td colspan="4" style="text-align: center; padding: 20px; color: #666;">No products found.</td>
                  </tr>
                </tbody>
              </table>
            </div>

            <!-- Pagination / Status Bar -->
            <div class="pagination-bar">
              <div class="page-controls">
                <span class="pi pi-search">
                   <svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg>
                </span>
                <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/></svg></span>
                <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
                <span class="page-text">Page <input type="text" value="1" class="page-input" /> of 1</span>
                <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
                <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M16 6v12h2V6zM6 18l8.5-6L6 6z"/></svg></span>
                <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></span>
                <span class="display-text" style="color: #444; margin-left:10px;">Displaying 1 to {{ filteredProducts.length }} of {{ filteredProducts.length }} items</span>
              </div>
            </div>
          </div>
        </div>
      </div>
      <ProductView v-else :productCode="selectedProductCode" :isCreating="currentView === 'create'" @back="backToList" />
    </template>
    </main>
  </div>
</template>

<style scoped>
.selected-row td {
  background-color: #f2dfe1 !important;
}

.top-record-box {
  background-color: #fff;
  border: 2px solid #c7c7c7; /* Updated from 1px solid #a0a0a0 */
  margin-bottom: 20px;
}

.box-header {
  background-color: #c7c7c7; /* Lightened from #a0a0a0 */
  border-bottom: 2px solid #c7c7c7; /* Sync header border with box border */
  margin: -2px -2px 0 -2px; /* Adjust margin for 2px thick border */
}

.list-panel {
  padding: 15px;
  border: none;
  min-height: auto;
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
  background-color: #f7f7f7; /* Changed to lighter grey matching table stripes */
}

.search-row {
  background-color: #ffffff;
}

.action-link:hover {
  color: #8f3235; /* Dark red matching photo hover colors */
  text-decoration: none;
}

.no-margin {
  margin-bottom: 0 !important;
}

/* Pagination Bar Styles */
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

.page-text {
  color: #333;
}

.page-input {
  width: 35px;
  height: 18px;
  border: 1px solid #ccc;
  text-align: center;
  font-size: 11px;
}
</style>
