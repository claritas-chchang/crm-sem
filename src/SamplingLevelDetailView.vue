<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  record: Object,
  isCreating: Boolean,
  isEditing: { type: Boolean, default: false },
  isModal: { type: Boolean, default: false }
})

const emit = defineEmits(['back', 'save'])

const isEditing = ref(props.isEditing || props.isCreating)
const localRecord = ref({ ...props.record })

watch(() => props.record, (newVal) => {
  localRecord.value = JSON.parse(JSON.stringify(newVal))
}, { deep: true })

const startEdit = () => {
  isEditing.value = true
}

const cancelEdit = () => {
  if (props.isCreating) {
    emit('back')
  } else {
    localRecord.value = JSON.parse(JSON.stringify(props.record))
    isEditing.value = false
  }
}

const saveRecord = () => {
  emit('save', { ...localRecord.value })
  isEditing.value = false
}

const goBack = () => {
  emit('back')
}

const samplingNameOptions = [
  '--Please Select One--',
  'General Dim. / Sampling', 'Outer Diameter / Sampling', 'Inner Diameter / Sampling', 'R dimension / Sampling', 
  'R thickness / Sampling', 'Center off O.R / Sampling', 'Center off I.R / Sampling', 'Angularity / Sampling', 
  'Flatness / Sampling', 'Parallelism / Sampling', 'Perpendicularity / Sampling', 'Straightness / Sampling', 
  'Position / Sampling', 'Symmetricity / Sampling', 'Profile / Sampling', 'Coaxiality / Sampling', 
  'Cylindricity / Sampling', 'Roundness / Sampling', 'Runout / Sampling', 'Total Flux / Sampling', 
  'Open Flux / Sampling', 'Appearance / Sampling', 'Marking / Sampling', 'Thickness of Plating / Sampling', 
  'Thickness of HC Inorganic coating / Sampling', 'Thickness of Epoxy coating / Sampling', 'Thickness of EI coating / Sampling', 
  'Adhesion intensity of coating film / Sampling', 'Chamfer / Sampling', 'Weight / Sampling', 'Cleanliness / Sampling', 
  'I, AQL 1% (normal)', 'S-4, AQL 1% (normal)', 'S-3, AQL 1% (normal)', 'I, AQL 1% (reduced)', 'S-4, AQL 1% (reduced)', 
  'S-3, AQL 1% (reduced)', 'I, AQL 0.65%', '40 pcs / lot', '30 pcs / lot', '20 pcs / lot', '13 pcs / lot', 
  '10 pcs / lot', '7 pcs / lot', '5 pcs / lot', '3 pcs / lot', '2 pcs / lot', '1 pcs / lot'
]

const measurementPointOptions = ['--Please Select One--', '1', '2', '3', '4', '5']

/* ---- Sampling Size List sub-panel (same design as ProductView's Dimension List) ---- */
const sampleRows = ref([
  { lotMin: '2', lotMax: '8', sNormal: '5', sReduced: '2', sTightened: '8', r3: '15', r4: '15', r5: '15' },
  { lotMin: '9', lotMax: '15', sNormal: '8', sReduced: '3', sTightened: '13', r3: '20', r4: '20', r5: '20' },
  { lotMin: '16', lotMax: '25', sNormal: '13', sReduced: '5', sTightened: '20', r3: '32', r4: '32', r5: '32' }
])

const isSampleListOpen = ref(false)
const toggleSampleList = () => {
  isSampleListOpen.value = !isSampleListOpen.value
}

const initialNewRow = {
  lotMin: '', lotMax: '', sNormal: '', sReduced: '', sTightened: '', r3: '', r4: '', r5: ''
}
const newRow = ref({ ...initialNewRow })

const showAddModal = ref(false)
const isEditingSub = ref(false)
const editingIdx = ref(-1)

const openAddModal = () => {
  newRow.value = { ...initialNewRow }
  isEditingSub.value = false
  editingIdx.value = -1
  showAddModal.value = true
}

const openEditModal = (idx) => {
  editingIdx.value = idx
  newRow.value = { ...sampleRows.value[idx] }
  isEditingSub.value = true
  showAddModal.value = true
}

const saveRow = () => {
  const f = (val) => (val === '' || val === null) ? '-' : val
  const row = {
    lotMin: f(newRow.value.lotMin),
    lotMax: f(newRow.value.lotMax),
    sNormal: f(newRow.value.sNormal),
    sReduced: f(newRow.value.sReduced),
    sTightened: f(newRow.value.sTightened),
    r3: f(newRow.value.r3),
    r4: f(newRow.value.r4),
    r5: f(newRow.value.r5)
  }
  if (isEditingSub.value && editingIdx.value !== -1) {
    sampleRows.value[editingIdx.value] = row
  } else {
    sampleRows.value.push(row)
  }
  showAddModal.value = false
}

const showDeleteConfirm = ref(false)
const rowToDeleteIdx = ref(-1)

const removeRow = (idx) => {
  rowToDeleteIdx.value = idx
  showDeleteConfirm.value = true
}

const confirmRemove = () => {
  if (rowToDeleteIdx.value !== -1) {
    sampleRows.value.splice(rowToDeleteIdx.value, 1)
  }
  showDeleteConfirm.value = false
  rowToDeleteIdx.value = -1
}

const cancelRemove = () => {
  showDeleteConfirm.value = false
  rowToDeleteIdx.value = -1
}
</script>

<template>
  <div class="sampling-detail-page">
  <div :class="['top-record-box custom-sampling-detail', { 'modal-layout': isModal }]">
    <!-- Breadcrumbs -->
    <div v-if="!isModal" class="sub-header box-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span class="breadcrumb">
        <a href="#" class="item-link" @click.prevent="goBack" style="font-weight: bold; color: #0000EE; text-decoration: underline;">SAMPLING LEVEL RECORDS</a> 
        &gt; <span class="current-page">{{ isCreating ? 'Create New' : localRecord.name }}</span>
      </span>
    </div>

    <!-- Actions -->
    <div class="top-actions">
      <template v-if="!isEditing">
        <button class="btn btn-primary" @click="startEdit">EDIT</button>
        <button class="btn btn-secondary" @click="goBack">Cancel</button>
      </template>
      <template v-else>
        <button class="btn btn-primary" @click="saveRecord">SAVE</button>
        <button class="btn btn-secondary" @click="cancelEdit">Cancel</button>
      </template>
    </div>

    <div class="sub-panel-wrapper">
      <fieldset class="fsMargin">
        <legend><b>Sampling Level Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Sampling Name</span></td>
              <td style="width: 34%;"><div v-if="!isEditing" class="field-value">{{ localRecord.name }}</div><select v-else v-model="localRecord.name" class="edit-select"><option v-for="opt in samplingNameOptions" :value="opt" :key="opt">{{ opt }}</option></select></td>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle"></span></td>
              <td style="width: 34%;"></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Measurement Point</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.measurementPoint }}</div><select v-else v-model="localRecord.measurementPoint" class="edit-select"><option v-for="opt in measurementPointOptions" :value="opt" :key="opt">{{ opt }}</option></select></td>
              <td class="labelBack"><span class="labelTitle"></span></td>
              <td></td>
            </tr>
          </tbody>
        </table>
      </fieldset>

      <fieldset v-if="!isEditing" class="fsMargin">
        <legend><b>System Information</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 16%; height: 24px;"><span class="labelTitle">Created Date</span></td>
              <td style="width: 34%;"><div class="field-value">{{ localRecord.creationDate || '16-March-2026 12:58:05 PM' }}</div></td>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Created By</span></td>
              <td style="width: 34%;"><div class="field-value">{{ localRecord.createdBy || 'qa-admin-p2' }}</div></td>
            </tr>
            <tr>
              <td class="labelBack" style="width: 16%; height: 24px;"><span class="labelTitle">Last Updated Date</span></td>
              <td><div class="field-value">{{ localRecord.updatedDate || '16-March-2026 12:59:47 PM' }}</div></td>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Last Updated By</span></td>
              <td><div class="field-value">{{ localRecord.updatedBy || 'qa-tech-p2' }}</div></td>
            </tr>
          </tbody>
        </table>
      </fieldset>
    </div>
  </div>

  <!-- Sampling Size List Section (same design as ProductView's Dimension List) -->
  <div v-if="!isCreating" class="dim-panel-wrapper">
    <div class="dim-panel-header" @click="toggleSampleList">
      <span class="dim-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!isSampleListOpen" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 11px; text-transform: uppercase;">Sampling Size List</span>
    </div>

    <div v-if="isSampleListOpen" class="dim-panel-body">
      <div class="dim-panel-inner-box">
        <div class="sub-actions" style="padding: 5px 15px; border-bottom: 1px solid #ddd;">
          <span class="text-link" @click="openAddModal">Add New Sampling Size</span>
        </div>

        <div class="table-scroll-container">
          <table class="data-table dim-table">
            <thead>
              <tr>
                <th class="col-icon"></th>
                <th>Lot Size (Min)</th>
                <th>Lot Size (Max)</th>
                <th>No of Samples (Normal)</th>
                <th>No of Samples (Reduced)</th>
                <th>No of Samples (Tightened)</th>
                <th>Rank 3</th>
                <th>Rank 4</th>
                <th>Rank 5</th>
                <th>Remove</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, idx) in sampleRows" :key="idx">
                <td class="col-icon">
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;" @click="openEditModal(idx)">
                    <path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/>
                  </svg>
                </td>
                <td>{{ row.lotMin }}</td>
                <td>{{ row.lotMax }}</td>
                <td>{{ row.sNormal }}</td>
                <td>{{ row.sReduced }}</td>
                <td>{{ row.sTightened }}</td>
                <td>{{ row.r3 }}</td>
                <td>{{ row.r4 }}</td>
                <td>{{ row.r5 }}</td>
                <td style="text-align: center;">
                  <button class="btn-remove" @click="removeRow(idx)">
                    <svg viewBox="0 0 24 24" width="12" height="12" fill="#d9534f" style="margin-right: 4px; vertical-align: middle;">
                      <path d="M6 19c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7H6v12zM19 4h-3.5l-1-1h-5l-1 1H5v2h14V4z"/>
                    </svg>
                    Remove
                  </button>
                </td>
              </tr>
              <tr v-if="sampleRows.length === 0">
                <td colspan="10" style="text-align: center; padding: 20px; color: #666;">No sampling size records found.</td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Pagination / Status Bar -->
        <div class="pagination-bar">
          <div class="page-controls">
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
            <span class="page-text">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M16 6v12h2V6zM6 18l8.5-6L6 6z"/></svg></span>
            <span class="display-text" style="color: #444; margin-left:10px;">Displaying 1 to {{ sampleRows.length }} of {{ sampleRows.length }} items</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Add / Edit Sampling Size modal -->
  <div class="modal-overlay" v-if="showAddModal">
    <div class="modal-window">
      <div class="sub-header modal-sub-header">
        <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        <span style="font-weight: bold; color: black;">{{ isEditingSub ? 'EDIT SAMPLING SIZE' : 'SAMPLING SIZE MANAGEMENT' }}</span>
      </div>

      <div class="panel modal-panel">
        <div class="top-actions" style="padding: 15px 0 10px 0; background-color: transparent;">
          <button class="btn btn-primary" @click="saveRow">{{ isEditingSub ? 'SAVE' : 'ADD' }}</button>
          <button class="btn btn-secondary" @click="showAddModal = false">Cancel</button>
        </div>

        <div class="section-container" style="margin: 0; box-shadow: none;">
          <div class="section-title">Sampling Size Details</div>
          <div class="info-grid">

            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">Lot Size (Min)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRow.lotMin" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Lot Size (Max)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRow.lotMax" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">No of Samples (Normal)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRow.sNormal" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">No of Samples (Reduced)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRow.sReduced" /></div>
              </div>
            </div>

            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">No of Samples (Tightened)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRow.sTightened" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Rank 3</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRow.r3" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Rank 4</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRow.r4" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Rank 5</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRow.r5" /></div>
              </div>
            </div>

          </div>
        </div>

        <div class="top-actions" style="padding: 15px 0 0 0; background-color: transparent;">
          <button class="btn btn-primary" @click="saveRow">{{ isEditingSub ? 'SAVE' : 'ADD' }}</button>
          <button class="btn btn-secondary" @click="showAddModal = false">Cancel</button>
        </div>

      </div>
    </div>
  </div>

  <!-- Custom Delete Confirmation Modal -->
  <div class="modal-overlay centered-overlay" v-if="showDeleteConfirm">
    <div class="confirm-banner">
      <div class="confirm-message">Are you sure to remove?</div>
      <div class="confirm-actions">
        <button class="btn-ok" @click="confirmRemove">OK</button>
        <button class="btn-cancel" @click="cancelRemove">Cancel</button>
      </div>
    </div>
  </div>
  </div>
</template>

<style scoped>
.custom-sampling-detail {
  --header-red: #a51c22;
  --border-color: #d1d1d1;
  --link-color: #0055cc;
  --font-family: Arial, Helvetica, sans-serif;
  
  font-family: var(--font-family);
  font-size: 13px;
  color: #333;
  box-sizing: border-box;
  margin: 0 15px 15px 15px;
  background-color: #fff;
  border: 2px solid var(--border-color);
  overflow: hidden;
}
.modal-layout {
  border: none !important;
  margin: 0 !important;
  box-shadow: none !important;
}

.box-header {
  margin: -1px -1px 0 -1px;
  background-color: #c7c7c7;
  padding: 8px 15px;
  border-bottom: 3px solid #fff;
  display: flex;
  align-items: center;
  gap: 8px;
}
.breadcrumb { font-size: 14px; font-weight: bold; }
.breadcrumb .item-link { color: var(--link-color); text-decoration: none; }
.current-page { font-weight: normal; color: #333; }

.top-actions {
  display: flex;
  gap: 10px;
  padding: 10px 15px;
  background-color: #fff;
}
.btn {
  padding: 8px 18px;
  font-size: 13px;
  font-weight: bold;
  border: none;
  border-radius: 2px;
  cursor: pointer;
}
.btn-primary { background-color: var(--header-red); color: white; }
.btn-secondary { background-color: #a5a5a5; color: white; }

.sub-panel-wrapper {
  margin: 15px;
}
.fsMargin {
  margin: 0px 0px 15px;
  border: 1px solid #E9E8E6;
  padding: 10px;
  background-color: #f1f1f1;
}
.fsMargin legend {
  padding: 0 10px;
  font-size: 13px;
}
.labelBack {
  background-color: #DADADA;
  padding: 4px 12px;
  vertical-align: middle;
}
.labelTitle { font-size: 13px; font-weight: normal; color: black; font-family: var(--font-family); } /* font-weight removed/set to normal */
.field-value { padding: 4px 12px; font-size: 13px; color: #333; font-family: var(--font-family); }
.edit-select { width: 80%; padding: 4px; border: 1px solid #ccc; font-size: 13px; box-sizing: border-box; }
.item-link { color: #0000EE; text-decoration: none; font-weight: bold; }
.item-link:hover { text-decoration: underline; }

/* ---- Sampling Size List sub-panel (mirrors ProductView's Dimension List) ---- */
.dim-panel-wrapper {
  margin: 0 15px 15px 15px;
}
.dim-panel-header {
  background-color: #c7c7c7;
  padding: 6px 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  border: 2px solid #c7c7c7;
}
.dim-panel-header:hover {
  background-color: #c9c9c9;
}
.dim-panel-icon-btn {
  display: flex;
  align-items: center;
  margin-right: -4px;
}
.dim-panel-body {
  background-color: #ffffff;
  padding: 10px 0;
  border-left: 2px solid #c7c7c7;
  border-right: 2px solid #c7c7c7;
  border-bottom: 2px solid #c7c7c7;
}
.dim-panel-inner-box {
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 2px;
  margin: 0 15px 10px 15px;
}

.sub-actions {
  padding: 8px 15px;
  background-color: #fff;
}
.text-link {
  font-size: 13px;
  color: #333;
  cursor: pointer;
}
.text-link:hover {
  color: #8f3235;
}

.table-scroll-container {
  overflow-x: auto;
  border-top: 1px solid #eee;
}
.data-table {
  width: 100%;
  border-collapse: collapse;
}
.dim-table th, .dim-table td {
  padding: 4px 8px;
  font-size: 11px;
  border: 1px solid #eee;
  white-space: nowrap;
}
.dim-table th {
  background: #f2f2f2;
  font-weight: bold;
  border-bottom: 2px solid #ddd;
}
.dim-table tbody tr:nth-child(odd) {
  background-color: #ffffff;
}
.dim-table tbody tr:nth-child(odd) td:first-child {
  background-color: #f0f0f0;
}
.dim-table tbody tr:nth-child(even) {
  background-color: #f8f8f8;
}
.dim-table tbody tr:nth-child(even) td:first-child {
  background-color: #e6e6e6;
}
.col-icon {
  width: 28px;
  text-align: center;
}

.btn-remove {
  background-color: white;
  border: 1px solid #ccc;
  border-radius: 2px;
  padding: 1px 6px;
  font-size: 11px;
  cursor: pointer;
  color: #333;
  display: inline-flex;
  align-items: center;
}
.btn-remove:hover {
  background-color: #f9f9f9;
  border-color: #bbb;
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
.page-text { color: #333; }
.page-input {
  width: 35px;
  height: 18px;
  border: 1px solid #ccc;
  text-align: center;
  font-size: 11px;
}

/* Modal */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding-top: 50px;
  z-index: 1000;
}
.modal-window {
  width: 900px;
  background-color: #fcfcfc;
  border: 1px solid #ddd;
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
}
.modal-sub-header {
  background: linear-gradient(to bottom, #eeeeee, #cccccc);
  padding: 8px 15px;
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  border-bottom: 2px solid #fff;
}
.sub-header {
  display: flex;
  align-items: center;
  gap: 8px;
}
.modal-panel {
  padding: 0 15px 15px 15px;
}
.section-container {
  background-color: #fff;
  border: 1px solid #ddd;
}
.section-title {
  background-color: transparent;
  padding: 8px 0;
  font-weight: bold;
  font-size: 13px;
}
.info-grid {
  display: flex;
  padding: 0;
  background-color: #f6f6f6;
}
.grid-col {
  flex: 1;
  display: flex;
  flex-direction: column;
}
.grid-row {
  display: flex;
  min-height: 28px;
  border-bottom: 2px solid #fff;
}
.grid-label {
  width: 150px;
  background-color: #e6e6e6;
  padding: 6px 15px;
  color: #333;
  font-size: 12px;
}
.grid-value {
  flex: 1;
  background-color: #f6f6f6;
  padding: 6px 15px;
  color: #333;
}
.modal-label {
  background-color: #e8e8e8;
  border-right: 1px solid #fff;
  border-bottom: 1px solid #fff;
  font-family: Arial, sans-serif;
  padding: 6px 12px;
  width: 200px;
  font-weight: normal;
}
.modal-value {
  background-color: white;
  border-bottom: 1px solid #e8e8e8;
  padding: 2px;
  display: flex;
  align-items: center;
}
.form-input {
  width: 100%;
  height: 24px;
  border: 1px solid #ccc;
  padding: 2px 5px;
  font-size: 12px;
  box-sizing: border-box;
}

/* Delete confirmation modal */
.centered-overlay {
  align-items: center;
  padding-top: 0;
}
.confirm-banner {
  background-color: #fff;
  width: 100%;
  max-width: 1200px;
  padding: 30px;
  text-align: center;
  box-shadow: 0 5px 25px rgba(0,0,0,0.5);
}
.confirm-message {
  font-size: 18px;
  color: #333;
  margin-bottom: 25px;
}
.confirm-actions {
  display: flex;
  justify-content: center;
  gap: 15px;
}
.btn-ok {
  background-color: #69d17d;
  color: white;
  border: none;
  padding: 8px 45px;
  font-size: 14px;
  cursor: pointer;
  border-radius: 2px;
}
.btn-ok:hover { background-color: #5cb85c; }
.btn-cancel {
  background-color: #a5a5a5;
  color: white;
  border: none;
  padding: 8px 45px;
  font-size: 14px;
  cursor: pointer;
  border-radius: 2px;
}
.btn-cancel:hover { background-color: #888; }
</style>
