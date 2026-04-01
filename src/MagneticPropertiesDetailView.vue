<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  record: Object,
  isCreating: Boolean,
  isEditing: { type: Boolean, default: false },
  isModal: { type: Boolean, default: false }
})

const emit = defineEmits(['back', 'save'])

const localRecord = ref({ ...props.record })
const isEditing = ref(props.isEditing || props.isCreating)

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

// Dropdown Options
const typeOptions = ['G', 'Non-G']

// Sub-panel state - Type A
const isTypeAListOpen = ref(true)
const toggleTypeAList = () => {
  isTypeAListOpen.value = !isTypeAListOpen.value
}

// Sub-panel state - Type B
const isTypeBListOpen = ref(true)
const toggleTypeBList = () => {
  isTypeBListOpen.value = !isTypeBListOpen.value
}

// Sample Data for Type A List
const typeARecords = ref([
  { invoiceNo: 'P2-3037B', secPo: '75086522 01 09', codeJpn: '0315640244742899', br: '14181', d4id: '13984', ihc: '14580', bhc: '13687', bhMax: '48.9', d4ik: '0.973', d4is: '14265', ihk: '14385', d4ia: '13861', gcm3: '7.555', zone: '1AEL' },
  { invoiceNo: 'P2-3037B', secPo: '75086522 01 09', codeJpn: '0315640244742899', br: '14174', d4id: '13980', ihc: '14647', bhc: '13687', bhMax: '48.88', d4ik: '0.973', d4is: '14262', ihk: '14327', d4ia: '13847', gcm3: '7.554', zone: '5AM' },
  { invoiceNo: 'P2-3037B', secPo: '75086522 01 09', codeJpn: '0315640244742899', br: '14213', d4id: '14039', ihc: '14463', bhc: '13747', bhMax: '49.29', d4ik: '0.976', d4is: '14294', ihk: '14219', d4ia: '13915', gcm3: '7.552', zone: '9BL' },
  { invoiceNo: 'P2-3037B', secPo: '75086522 01 09', codeJpn: '0315640244742899', br: '14181', d4id: '14000', ihc: '14650', bhc: '13735', bhMax: '49.02', d4ik: '0.975', d4is: '14264', ihk: '14408', d4ia: '13886', gcm3: '7.541', zone: 'J4BER' },
  { invoiceNo: 'P2-3037B', secPo: '75086522 01 09', codeJpn: '0315640244842902', br: '14168', d4id: '13992', ihc: '14609', bhc: '13716', bhMax: '48.96', d4ik: '0.976', d4is: '14252', ihk: '14435', d4ia: '13873', gcm3: '7.541', zone: '1AEL' },
])

// Sample Data for Type B List
const typeBRecords = ref([
  { lotNo: '25061804001', avgBr: '14121', avgHcj: '22849', avgHcb: '13584' },
  { lotNo: '25061804002', avgBr: '14156', avgHcj: '23230', avgHcb: '13599' },
  { lotNo: '25061804003', avgBr: '14217', avgHcj: '22690', avgHcb: '13643' },
  { lotNo: '25061804004', avgBr: '14124', avgHcj: '22488', avgHcb: '13541' },
])
</script>

<template>
  <div :class="['top-record-box custom-mag-prop-detail', { 'modal-layout': isModal }]">
    <!-- Breadcrumbs -->
    <div v-if="!isModal" class="sub-header box-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span class="breadcrumb" style="font-size: 14px;">
        <a href="#" class="item-link" @click.prevent="goBack" style="font-weight: bold; color: #0000EE;">MAGNETIC PROPERTIES RECORDS</a> 
        &gt; <span class="current-page" style="font-weight: normal; color: #333;">{{ isCreating ? 'Create New' : (localRecord.code || 'New') }}</span>
      </span>
    </div>

    <!-- Top Action Bar -->
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

    <div class="form-wrapper">
      <!-- Product Details Section -->
      <fieldset class="fsMargin">
        <legend><b>Magnetic Properties Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%; height: 20px;"><span class="labelTitle">Product ID/Code</span></td>
              <td style="width: 30%; height: 21px;"><div v-if="!isCreating" class="field-value">{{ localRecord.code }}</div><input v-else type="text" v-model="localRecord.code" class="edit-select" /></td>
              <td class="labelBack" style="width: 20%; height: 20px;"><span class="labelTitle">Product Type</span></td>
              <td style="width: 30%; height: 21px;">
                <div v-if="!isEditing" class="field-value">{{ localRecord.type }}</div>
                <select v-else v-model="localRecord.type" class="edit-select">
                  <option v-for="opt in typeOptions" :value="opt" :key="opt">{{ opt }}</option>
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
              <td class="labelBack" style="width: 20%; height: 20px;"><span class="labelTitle">Creation Date</span></td>
              <td style="width: 30%;"><div class="field-value">27-March-2026 12:58:05 PM</div></td>
              <td class="labelBack" style="width: 20%; height: 20px;"><span class="labelTitle">Created By</span></td>
              <td style="width: 30%;"><div class="field-value">qa-admin</div></td>
            </tr>
            <tr>
              <td class="labelBack" style="width: 20%; height: 20px;"><span class="labelTitle">Updated Date</span></td>
              <td><div class="field-value">27-March-2026 01:15:30 PM</div></td>
              <td class="labelBack" style="width: 20%; height: 20px;"><span class="labelTitle">Updated By</span></td>
              <td><div class="field-value">qa-tech</div></td>
            </tr>
          </tbody>
        </table>
      </fieldset>
    </div>
  </div>

  <!-- Sub-panel: Type A Section (Matching Inspection page Product Dimension List EXACTLY) -->
  <div v-if="!isCreating && localRecord.type === 'G'" class="sub-panel-wrapper">
    <div class="sub-panel-header" @click="toggleTypeAList">
      <span class="sub-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!isTypeAListOpen" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 14px; text-transform: uppercase;">Type A</span>
    </div>

    <div v-if="isTypeAListOpen" class="sub-panel-body">
      <div class="sub-panel-inner-box">
        <div class="table-scroll-container">
          <table class="data-table dim-table">
            <thead>
              <tr>
                <th class="col-icon" style="width: 30px;"></th>
                <th>Invoice Number</th>
                <th>Sec PO</th>
                <th>Code-JPN-Furnance Lot No</th>
                <th>BR (G)</th>
                <th>4ID (G)</th>
                <th>IHC (Oe)</th>
                <th>BHC (Oe)</th>
                <th>BH(MAX) (MGOe)</th>
                <th>4IK (Oe)</th>
                <th>4IS (Oe)</th>
                <th>IHK (Oe)</th>
                <th>4IA (Oe)</th>
                <th>g/cm3</th>
                <th>Zone</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(r, idx) in typeARecords" :key="idx">
                <td class="col-icon">
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                </td>
                <td>{{ r.invoiceNo }}</td>
                <td>{{ r.secPo }}</td>
                <td>{{ r.codeJpn }}</td>
                <td style="text-align: right;">{{ r.br }}</td>
                <td style="text-align: right;">{{ r.d4id }}</td>
                <td style="text-align: right;">{{ r.ihc }}</td>
                <td style="text-align: right;">{{ r.bhc }}</td>
                <td style="text-align: right;">{{ r.bhMax }}</td>
                <td style="text-align: right;">{{ r.d4ik }}</td>
                <td style="text-align: right;">{{ r.d4is }}</td>
                <td style="text-align: right;">{{ r.ihk }}</td>
                <td style="text-align: right;">{{ r.d4ia }}</td>
                <td style="text-align: right;">{{ r.gcm3 }}</td>
                <td>{{ r.zone }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="pagination-bar">
          <div class="page-controls">
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
            <span class="page-text" style="font-size: 11px;">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="display-text" style="font-size: 11px; margin-left: 10px;">Displaying 1 to {{ typeARecords.length }} of {{ typeARecords.length }} items</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Sub-panel: Type B Section -->
  <div v-if="!isCreating && localRecord.type === 'Non-G'" class="sub-panel-wrapper">
    <div class="sub-panel-header" @click="toggleTypeBList">
      <span class="sub-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!isTypeBListOpen" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 14px; text-transform: uppercase;">Type B</span>
    </div>

    <div v-if="isTypeBListOpen" class="sub-panel-body">
      <div class="sub-panel-inner-box">
        <div class="table-scroll-container">
          <table class="data-table dim-table">
            <thead>
              <tr>
                <th class="col-icon" style="width: 30px;"></th>
                <th>Lot No</th>
                <th>Average Br</th>
                <th>Average Hcj</th>
                <th>Average Hcb</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(r, idx) in typeBRecords" :key="idx">
                <td class="col-icon">
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                </td>
                <td>{{ r.lotNo }}</td>
                <td style="text-align: right;">{{ r.avgBr }}</td>
                <td style="text-align: right;">{{ r.avgHcj }}</td>
                <td style="text-align: right;">{{ r.avgHcb }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="pagination-bar">
          <div class="page-controls">
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
            <span class="page-text" style="font-size: 11px;">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="display-text" style="font-size: 11px; margin-left: 10px;">Displaying 1 to {{ typeBRecords.length }} of {{ typeBRecords.length }} items</span>
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
.modal-layout {
  border: none !important;
  margin: 0 !important;
  box-shadow: none !important;
}
.breadcrumb { font-size: 14px; font-weight: bold; }
.item-link { color: #0000EE; text-decoration: none; font-weight: bold; }
.item-link:hover { text-decoration: underline; }

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

.form-wrapper { padding: 0 15px; }
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
.edit-select { width: 80%; padding: 4px; border: 1px solid #ccc; font-size: 13px; box-sizing: border-box; }

/* Sub-panel styling (EXACT Replication of Inspection page) */
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
</style>
