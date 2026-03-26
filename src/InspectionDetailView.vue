<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  record: Object,
  isCreating: Boolean
})

const emit = defineEmits(['back', 'save'])

const localRecord = ref({ ...props.record })
const isEditing = ref(props.isCreating)

watch(() => props.record, (newVal) => {
  localRecord.value = { ...newVal }
})

const startEdit = () => {
  isEditing.value = true
}

const cancelEdit = () => {
  if (props.isCreating) {
    emit('back')
  } else {
    localRecord.value = { ...props.record }
    isEditing.value = false
  }
}

const saveRecord = () => {
  emit('save', { ...localRecord.value })
  isEditing.value = false
}

const mMethodOptions = ['--Please Select One--', 'Max Outlier', 'Max Value']
const firstRunOptions = ['--Please Select One--', 'Yes', 'No']
const resultOptions = ['--Please Select One--', 'OK', 'NG']

const goBack = () => {
  emit('back')
}

// Sub-panel Dimension List logic
const isDimensionListOpen = ref(true)
const toggleDimensionList = () => {
  isDimensionListOpen.value = !isDimensionListOpen.value
}

const dimensions = ref([
  { name: 'Length', min: '24.47', max: '24.55', size: '24.5', uom: 'mm', minTol: '0.03', maxTol: '0.05', minOnly: 'No', maxOnly: 'No', eq: 'Micrometer', sampSize: '-', remarks: '' },
  { name: 'Width', min: '24.45', max: '24.55', size: '24.5', uom: 'mm', minTol: '', maxTol: '0.05', minOnly: 'No', maxOnly: 'No', eq: 'Micrometer', sampSize: '-', remarks: '' },
  { name: 'Thickness', min: '8.85', max: '8.95', size: '8.9', uom: 'mm', minTol: '', maxTol: '0.05', minOnly: 'No', maxOnly: 'No', eq: 'Micrometer', sampSize: '-', remarks: '' },
  { name: 'Flatness', min: '-', max: '0.08', size: '0.08', uom: 'mm', minTol: '-', maxTol: '0', minOnly: 'No', maxOnly: 'Yes', eq: 'Linear gage', sampSize: '-', remarks: '' },
  { name: 'Parallelism', min: '-', max: '0.08', size: '0.08', uom: 'mm', minTol: '-', maxTol: '0', minOnly: 'No', maxOnly: 'Yes', eq: 'Linear gage', sampSize: '-', remarks: '' },
  { name: 'Perpendicularity', min: '-', max: '0.08', size: '0.08', uom: 'mm', minTol: '-', maxTol: '0', minOnly: 'No', maxOnly: 'Yes', eq: 'Square, Thickness gage', sampSize: '-', remarks: '' },
  { name: 'Weight', min: '40.6', max: '-', size: '-', uom: 'g', minTol: '-', maxTol: '0', minOnly: 'Yes', maxOnly: 'No', eq: 'Electronic Balance', sampSize: '-', remarks: '' },
  { name: 'Coating Thickness', min: '5', max: '25', size: '-', uom: 'μm', minTol: '-', maxTol: '0', minOnly: 'No', maxOnly: 'No', eq: 'X-Ray Machine', sampSize: '-', remarks: '' },
  { name: 'Appearance (before magnetize)', min: '-', max: '-', size: '-', uom: 'remarks', minTol: '-', maxTol: '-', minOnly: '-', maxOnly: '-', eq: 'Naked eyes', sampSize: '-', remarks: 'Per visual inspection criteria for magnet' },
  { name: 'Green Paper Check', min: '-', max: '-', size: '-', uom: 'remarks', minTol: '-', maxTol: '-', minOnly: '-', maxOnly: '-', eq: 'Green Paper', sampSize: '-', remarks: 'Per WI No. : W-0604-QA' },
  { name: 'Magnet Sticking Checking', min: '-', max: '-', size: '-', uom: 'remarks', minTol: '-', maxTol: '-', minOnly: '-', maxOnly: '-', eq: '-', sampSize: '-', remarks: 'Per WI No. : W-0604-QA' },
  { name: 'Total Flux', min: '413', max: '430', size: 'x10-4', uom: '[Wb.Ts.]', minTol: '-', maxTol: '-', minOnly: '-', maxOnly: '-', eq: 'Flux meter + Fixture', sampSize: '-', remarks: 'TQ-31' },
  { name: 'Polarity', min: '-', max: '-', size: '-', uom: 'remarks', minTol: '-', maxTol: '-', minOnly: '-', maxOnly: '-', eq: 'Polarity Checker', sampSize: '-', remarks: 'Horizontal RED line, along the top area of W x T surface, with N-pole facing upwards.' },
  { name: 'Appearance (After Magnetizing)', min: '-', max: '-', size: '-', uom: 'remarks', minTol: '-', maxTol: '-', minOnly: '-', maxOnly: '-', eq: 'Naked eyes', sampSize: '-', remarks: 'Per visual inspection criteria for magnet' },
  { name: 'Magnetized Condition', min: '-', max: '-', size: '-', uom: 'remarks', minTol: '-', maxTol: '-', minOnly: '-', maxOnly: '-', eq: 'Green Paper', sampSize: '-', remarks: 'Fully magnetized & no sign of abnormal grain growth' },
  { name: 'Marking Position', min: '-', max: '-', size: '-', uom: 'remarks', minTol: '-', maxTol: '-', minOnly: '-', maxOnly: '-', eq: 'Naked eyes', sampSize: '-', remarks: 'Position: Horizontal RED line, along the top area of W x T surface, with N-pole facing upwards.' },
  { name: 'Marking Color', min: '-', max: '-', size: '-', uom: 'remarks', minTol: '-', maxTol: '-', minOnly: '-', maxOnly: '-', eq: 'Naked eyes', sampSize: '-', remarks: 'Red' }
])

const uomOptions = ['mm', 'μm', 'g', 'degree', 'N/m', 'remarks']
const minMaxOnlyOptions = ['No', 'Yes']

const editingCell = ref({ rowIdx: -1, field: '' })
const originalGridValue = ref('')

const startGridEdit = (rowIdx, field) => {
  editingCell.value = { rowIdx, field }
  originalGridValue.value = dimensions.value[rowIdx][field]
}

const saveGridEdit = () => {
  editingCell.value = { rowIdx: -1, field: '' }
}

const cancelGridEdit = () => {
  if (editingCell.value.rowIdx !== -1) {
    dimensions.value[editingCell.value.rowIdx][editingCell.value.field] = originalGridValue.value
    editingCell.value = { rowIdx: -1, field: '' }
  }
}

const handleGlobalClick = (e) => {
  if (editingCell.value.rowIdx === -1) return
  // If click is outside both the edit box and the triggering text, cancel
  if (!e.target.closest('.inline-edit') && !e.target.closest('.editable-text')) {
    cancelGridEdit()
  }
}

onMounted(() => {
  window.addEventListener('click', handleGlobalClick)
})

onUnmounted(() => {
  window.removeEventListener('click', handleGlobalClick)
})
</script>

<template>
  <div class="top-record-box">
    <!-- Breadcrumbs -->
    <div class="sub-header box-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span class="breadcrumb" style="font-size: 11px;">
        <a href="#" class="item-link" @click.prevent="goBack" style="font-weight: bold; color: blue;">INSPECTION RECORDS</a> 
        > <span style="font-weight: normal; color: #333;">{{ isCreating ? 'Create New' : localRecord.product }}</span>
      </span>
    </div>

    <!-- Top Action Bar -->
    <div class="top-actions" style="padding: 10px 15px; margin-bottom: 5px;">
      <template v-if="!isEditing">
        <button class="btn btn-primary" @click="startEdit">EDIT</button>
        <button class="btn btn-secondary" @click="goBack">Cancel</button>
      </template>
      <template v-else>
        <button class="btn btn-primary" @click="saveRecord">SAVE</button>
        <button class="btn btn-secondary" @click="cancelEdit">Cancel</button>
      </template>
    </div>

    <div class="form-wrapper">
      <!-- Section 1: Inspection Details -->
      <fieldset class="fsMargin">
        <legend><b>Inspection Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Product Type</span></td>
              <td style="width: 30%;"><div v-if="!isEditing" class="field-value">{{ localRecord.productType }}</div><input v-else type="text" v-model="localRecord.productType" class="edit-select" /></td>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Coating Line</span></td>
              <td style="width: 30%;"><div v-if="!isEditing" class="field-value">{{ localRecord.cLine }}</div><input v-else type="text" v-model="localRecord.cLine" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Product</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.product }}</div><input v-else type="text" v-model="localRecord.product" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">PA Number</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.paNo }}</div><input v-else type="text" v-model="localRecord.paNo" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Product Serial Number</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.serialNo }}</div><input v-else type="text" v-model="localRecord.serialNo" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">JPN Lot No</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.jpnLot }}</div><input v-else type="text" v-model="localRecord.jpnLot" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Revision</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.revision }}</div><input v-else type="text" v-model="localRecord.revision" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Measurement Value Method</span></td>
              <td>
                 <div v-if="!isEditing" class="field-value">{{ localRecord.mMethod }}</div>
                 <select v-else v-model="localRecord.mMethod" class="edit-select">
                   <option v-for="opt in mMethodOptions" :key="opt" :value="opt">{{ opt }}</option>
                 </select>
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">DWG Number</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.dwgNo }}</div><input v-else type="text" v-model="localRecord.dwgNo" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">First Run</span></td>
              <td>
                 <div v-if="!isEditing" class="field-value">{{ localRecord.firstRun }}</div>
                 <select v-else v-model="localRecord.firstRun" class="edit-select">
                   <option v-for="opt in firstRunOptions" :key="opt" :value="opt">{{ opt }}</option>
                 </select>
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Lot Number</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.lotNo }}</div><input v-else type="text" v-model="localRecord.lotNo" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Remarks</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.remarks }}</div><input v-else type="text" v-model="localRecord.remarks" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Coating Date</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.cDate }}</div><input v-else type="text" v-model="localRecord.cDate" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Final Results</span></td>
              <td>
                 <div v-if="!isEditing" class="field-value">{{ localRecord.finalResult }}</div>
                 <select v-else v-model="localRecord.finalResult" class="edit-select">
                   <option v-for="opt in resultOptions" :key="opt" :value="opt">{{ opt }}</option>
                 </select>
              </td>
            </tr>
          </tbody>
        </table>
      </fieldset>

      <!-- System Information Section -->
      <fieldset v-if="!isEditing" class="fsMargin">
        <legend><b>System Information</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%; height: 20px;">
                <span class="labelTitle">Creation Date</span>
              </td>
              <td style="width: 30%; height: 21px;">
                <div class="field-value">{{ localRecord.creationDate || '16-March-2026 12:58:05 PM' }}</div>
              </td>
              <td class="labelBack" style="width: 20%; height: 20px;">
                <span class="labelTitle">Created By</span>
              </td>
              <td style="width: 30%; height: 21px;">
                <div class="field-value">{{ localRecord.createdBy || 'qa-admin-p2' }}</div>
              </td>
            </tr>
            <tr>
              <td class="labelBack" style="width: 20%; height: 20px;">
                <span class="labelTitle">Updated Date</span>
              </td>
              <td style="width: 30%; height: 21px;">
                <div class="field-value">{{ localRecord.updatedDate || '16-March-2026 12:59:47 PM' }}</div>
              </td>
              <td class="labelBack" style="width: 20%; height: 20px;">
                <span class="labelTitle">Updated By</span>
              </td>
              <td style="width: 30%; height: 21px;">
                <div class="field-value">{{ localRecord.updatedBy || 'qa-tech-p2' }}</div>
              </td>
            </tr>
          </tbody>
        </table>
      </fieldset>
    </div>
  </div>

  <!-- Product Dimension List Section (Moved OUTSIDE the main box) -->
  <div v-if="!isCreating" class="sub-panel-wrapper">
    <div class="sub-panel-header" @click="toggleDimensionList">
      <span class="sub-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!isDimensionListOpen" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 11px; text-transform: uppercase;">Product Dimension List</span>
    </div>
    
    <div v-if="isDimensionListOpen" class="sub-panel-body">
      <div class="sub-panel-inner-box">
        <div class="table-scroll-container">
          <table class="data-table dim-table">
            <thead>
              <tr>
                <th class="col-icon"></th>
                <th>Name</th>
                <th>Min Spec</th>
                <th>Max Spec</th>
                <th>Spec Size</th>
                <th>UOM</th>
                <th>Min Tolerance V (+-)</th>
                <th>Max Tolerance V (+-)</th>
                <th>Min Only</th>
                <th>Max Only</th>
                <th>Equipment</th>
                <th>Sampling Size</th>
                <th>Remarks</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(d, idx) in dimensions" :key="idx">
                <td class="col-icon">
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                </td>
                <td>{{ d.name }}</td>
                <td>{{ d.min }}</td>
                <td>{{ d.max }}</td>
                <td>{{ d.size }}</td>
                <td>
                  <div v-if="editingCell.rowIdx === idx && editingCell.field === 'uom'" class="inline-edit">
                    <select v-model="d.uom" class="grid-select" @click.stop>
                      <option v-for="opt in uomOptions" :value="opt" :key="opt">{{ opt }}</option>
                    </select>
                    <button class="btn-grid-save" @click.stop="saveGridEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startGridEdit(idx, 'uom')">{{ d.uom }}</span>
                </td>
                <td>{{ d.minTol }}</td>
                <td>{{ d.maxTol }}</td>
                <td>
                  <div v-if="editingCell.rowIdx === idx && editingCell.field === 'minOnly'" class="inline-edit">
                    <select v-model="d.minOnly" class="grid-select" @click.stop>
                      <option v-for="opt in minMaxOnlyOptions" :value="opt" :key="opt">{{ opt }}</option>
                    </select>
                    <button class="btn-grid-save" @click.stop="saveGridEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startGridEdit(idx, 'minOnly')">{{ d.minOnly }}</span>
                </td>
                <td>
                  <div v-if="editingCell.rowIdx === idx && editingCell.field === 'maxOnly'" class="inline-edit">
                    <select v-model="d.maxOnly" class="grid-select" @click.stop>
                      <option v-for="opt in minMaxOnlyOptions" :value="opt" :key="opt">{{ opt }}</option>
                    </select>
                    <button class="btn-grid-save" @click.stop="saveGridEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startGridEdit(idx, 'maxOnly')">{{ d.maxOnly }}</span>
                </td>
                <td>{{ d.eq }}</td>
                <td>{{ d.sampSize }}</td>
                <td>{{ d.remarks }}</td>
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
            <span class="page-text" style="font-size: 11px;">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M16 6v12h2V6zM6 18l8.5-6L6 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></span>
            <span class="display-text" style="font-size: 11px; margin-left: 10px;">Displaying 1 to {{ dimensions.length }} of {{ dimensions.length }} items</span>
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
  margin: 15px;
  overflow: hidden;
}
.box-header {
  background-color: #c7c7c7;
  border-bottom: 2px solid #c7c7c7;
  margin: -2px -2px 0 -2px;
}
.top-actions {
  display: flex;
  gap: 10px;
  padding: 10px 15px;
  background-color: transparent;
}
.btn {
  padding: 8px 18px;
  font-size: 13px;
  font-weight: bold;
  border: none;
  border-radius: 2px;
  cursor: pointer;
}
.btn-primary { background-color: #8f3235; color: white; }
.btn-secondary { background-color: #a5a5a5; color: #fff; }
.form-wrapper {
  margin: 0;
  padding: 0 15px;
}
.fsMargin {
  margin: 10px 0 15px 0;
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
.labelTitle { font-size: 13px; color: black; font-family: Arial, Helvetica, sans-serif; }
.field-value { padding-left: 12px; font-size: 13px; color: #333; }
.edit-select { width: 100%; padding: 4px; border: 1px solid #ccc; font-size: 13px; box-sizing: border-box; }
.item-link { color: blue; text-decoration: none; font-weight: bold; }
.item-link:hover { text-decoration: underline; }

/* Sub-panel styling (Product style) */
.sub-panel-wrapper { margin: 0 15px 15px 15px; }
.sub-panel-header {
  background-color: #c7c7c7;
  padding: 6px 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  border: 2px solid #c7c7c7;
}
.sub-panel-body {
  background-color: #ffffff;
  padding: 10px 0;
  border-left: 2px solid #c7c7c7;
  border-right: 2px solid #c7c7c7;
  border-bottom: 2px solid #c7c7c7;
}
.sub-panel-inner-box {
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 2px;
  margin: 0 15px 10px 15px;
}
.table-scroll-container { overflow-x: auto; border-top: 1px solid #eee; }
.dim-table th, .dim-table td { padding: 4px 8px; font-size: 11px; border: 1px solid #eee; white-space: nowrap; }
.dim-table th { background: #f2f2f2; font-weight: bold; border-bottom: 2px solid #ddd; text-align: left; }
.dim-table tbody tr:nth-child(odd) { background-color: #ffffff; }
.dim-table tbody tr:nth-child(even) { background-color: #f8f8f8; }

.pagination-bar { background-color: #f7f7f7; border-top: 1px solid #ddd; padding: 4px 10px; }
.page-controls { display: flex; align-items: center; gap: 8px; font-size: 11px; }
.p-btn { display: flex; align-items: center; cursor: pointer; }
.page-input { width: 35px; height: 18px; border: 1px solid #ccc; text-align: center; font-size: 11px; }

.inline-edit { display: flex; align-items: center; gap: 4px; }
.grid-select { border: 1px solid #ccc; font-size: 11px; }
.btn-grid-save { padding: 1px 4px; font-size: 10px; cursor: pointer; }
.editable-text { cursor: pointer; text-decoration: underline dotted #aaa; }
</style>
