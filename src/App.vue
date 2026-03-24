<script setup>
import { ref, computed } from 'vue'
import ProductView from './ProductView.vue'

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
  alert('Create New clicked')
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
      <a href="#">CASE</a>
      <a href="#">PL (External Lab)</a>
      <a href="#">PL (Other Plant)</a>
      <a href="#">Equipment</a>
      <a href="#">Vendor</a>
      <a href="#" class="active">Product records</a>
      <a href="#">Department</a>
      <a href="#">Calendar</a>
      <a href="#">Report</a>
      <a href="#">Admin</a>
      <a href="#">Help</a>
    </nav>

    <div class="sub-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span v-if="currentView === 'list'">PRODUCT RECORDS</span>
      <span v-else class="breadcrumb">
        <a href="#" class="item-link" @click.prevent="backToList" style="font-weight: bold; color: blue;">PRODUCT RECORDS</a> 
        > <span style="font-weight: normal;">{{ selectedProductCode }}</span>
      </span>
    </div>

    <main class="content">
      <div class="panel" v-if="currentView === 'list'">
        <div class="action-bar">
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

        <div class="search-bar">
          <select class="search-select" v-model="selectedFilter">
            <option v-for="opt in filterOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
          </select>
          <input type="text" class="search-input" v-model="searchQuery" />
          <button class="search-btn">SEARCH</button>
        </div>

        <div class="table-container">
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
      <ProductView v-else :productCode="selectedProductCode" @back="backToList" />
    </main>
  </div>
</template>

<style scoped>
.selected-row td {
  background-color: #f2dfe1 !important;
}

.selected-row td {
  background-color: #f2dfe1 !important;
}
</style>
