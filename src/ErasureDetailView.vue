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
const typeOptions = ['Environment Washing', 'Outgoing (Magnets)', 'Thin Model']
const shiftOptions = ['Day', 'Night']

// Sub-panel data structure
const thinModelBatches = ref([
  { id: 1, platingLine: '', erasureLine: '', remarks: '', performer: '' },
  { id: 2, platingLine: '', erasureLine: '', remarks: '', performer: '' },
  { id: 3, platingLine: '', erasureLine: '', remarks: '', performer: '' },
  { id: 4, platingLine: '', erasureLine: '', remarks: '', performer: '' },
  { id: 5, platingLine: '', erasureLine: '', remarks: '', performer: '' },
])

const magnetModelBatches = ref([
  { id: 1, min: '', avg: '', max: '', maxSpec: '', triggerLimit: '', remark: '', swabbedBy: '', erasureBy: '', verifiedBy: '' },
  { id: 2, min: '', avg: '', max: '', maxSpec: '', triggerLimit: '', remark: '', swabbedBy: '', erasureBy: '', verifiedBy: '' },
  { id: 3, min: '', avg: '', max: '', maxSpec: '', triggerLimit: '', remark: '', swabbedBy: '', erasureBy: '', verifiedBy: '' },
  { id: 4, min: '', avg: '', max: '', maxSpec: '', triggerLimit: '', remark: '', swabbedBy: '', erasureBy: '', verifiedBy: '' },
  { id: 5, min: '', avg: '', max: '', maxSpec: '', triggerLimit: '', remark: '', swabbedBy: '', erasureBy: '', verifiedBy: '' },
])
</script>

<template>
  <div :class="['top-record-box custom-erasure-detail', { 'modal-layout': isModal }]">
    <!-- Breadcrumbs -->
    <div v-if="!isModal" class="sub-header box-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span class="breadcrumb" style="font-size: 14px;">
        <a href="#" class="item-link" @click.prevent="goBack" style="font-weight: bold; color: #0000EE;">ERASURE RECORDS</a> 
        &gt; <span class="current-page">{{ isCreating ? 'Create New' : localRecord.id }}</span>
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

    <div class="sub-panel-wrapper">
      <!-- Section 1: Erasure Details (ORIGINAL 14 FIELDS - UNMODIFIED) -->
      <fieldset class="fsMargin" style="margin: 0 0 15px 0;">
        <legend><b>Erasure Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">ID</span></td>
              <td style="width: 30%;"><div v-if="!isEditing" class="field-value">{{ localRecord.id }}</div><input v-else type="text" v-model="localRecord.id" class="edit-select" disabled /></td>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Shift</span></td>
              <td style="width: 30%;">
                <div v-if="!isEditing" class="field-value">{{ localRecord.shift }}</div>
                <select v-else v-model="localRecord.shift" class="edit-select">
                  <option v-for="opt in shiftOptions" :value="opt" :key="opt">{{ opt }}</option>
                </select>
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Once per Day</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.onePerDay }}</div><input v-else type="text" v-model="localRecord.onePerDay" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Area</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.area }}</div><input v-else type="text" v-model="localRecord.area" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Twice per Shift</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.twoPerShift }}</div><input v-else type="text" v-model="localRecord.twoPerShift" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Time</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.time }}</div><input v-else type="time" v-model="localRecord.time" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Specification</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.spec }}</div><input v-else type="text" v-model="localRecord.spec" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Results</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.results }}</div><input v-else type="text" v-model="localRecord.results" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Method</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.method }}</div><input v-else type="text" v-model="localRecord.method" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Remarks</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.remarks }}</div><input v-else type="text" v-model="localRecord.remarks" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Date</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.date }}</div><input v-else type="datetime-local" v-model="localRecord.date" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Performed By</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.performer }}</div><input v-else type="text" v-model="localRecord.performer" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Type</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.type }}</div>
                <select v-else v-model="localRecord.type" class="edit-select">
                  <option v-for="opt in typeOptions" :value="opt" :key="opt">{{ opt }}</option>
                </select>
              </td>
              <td class="labelBack"><span class="labelTitle">Confirmed By</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.confirmer }}</div><input v-else type="text" v-model="localRecord.confirmer" class="edit-select" /></td>
            </tr>
          </tbody>
        </table>
      </fieldset>

      <!-- Conditional Section 2: Thin Model Details (Show / Hide) -->
      <fieldset class="fsMargin" v-if="localRecord.type === 'Thin Model'" style="margin: 0 0 15px 0;">
        <legend><b>Thin Model Section Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Model</span></td>
              <td style="width: 30%;"><div v-if="!isEditing" class="field-value">{{ localRecord.model || '-' }}</div><input v-else type="text" v-model="localRecord.model" class="edit-select" /></td>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Plant</span></td>
              <td style="width: 30%;"><div v-if="!isEditing" class="field-value">{{ localRecord.plant || '-' }}</div><input v-else type="text" v-model="localRecord.plant" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Specification</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.specThin || '-' }}</div><input v-else type="text" v-model="localRecord.specThin" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Trigger limit</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.triggerThin || '-' }}</div><input v-else type="text" v-model="localRecord.triggerThin" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Plating Date</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.platingDate || '-' }}</div><input v-else type="date" v-model="localRecord.platingDate" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Frequency</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.frequency || '-' }}</div><input v-else type="text" v-model="localRecord.frequency" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Method</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.methodThin || '-' }}</div><input v-else type="text" v-model="localRecord.methodThin" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Action</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.actionThin || '-' }}</div><input v-else type="text" v-model="localRecord.actionThin" class="edit-select" /></td>
            </tr>
          </tbody>
        </table>
        
        <!-- Batch Number Sub Panel inside Thin Model -->
        <div style="margin-top: 15px; border-top: 1px solid #ccc; padding-top: 10px;">
          <div style="background-color: #DADADA; padding: 4px 10px; font-weight: bold; font-size: 13px; margin-bottom: 5px;">Batch Number Sub Panel</div>
          <div style="overflow-x: auto;">
            <table class="grid-table" style="width: 100%; border-collapse: collapse;">
              <thead>
                <tr style="background-color: #f2f2f2; font-size: 11px;">
                  <th style="border: 1px solid #ccc; padding: 4px;">Batch Number</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Plating Line</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Erasure Line</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Remarks</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Performed By</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="b in thinModelBatches" :key="b.id">
                  <td style="border: 1px solid #ccc; padding: 4px; text-align: center; font-size: 11px;">{{ b.id }}</td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.platingLine" class="grid-input" /><span v-else>{{ b.platingLine }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.erasureLine" class="grid-input" /><span v-else>{{ b.erasureLine }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.remarks" class="grid-input" /><span v-else>{{ b.remarks }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.performer" class="grid-input" /><span v-else>{{ b.performer }}</span></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </fieldset>

      <!-- Conditional Section 3: Outgoing Magnet Details (Show / Hide) -->
      <fieldset class="fsMargin" v-if="localRecord.type === 'Outgoing (Magnets)'" style="margin: 0 0 15px 0;">
        <legend><b>Outgoing Magnet Section Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Model</span></td>
              <td style="width: 30%;"><div v-if="!isEditing" class="field-value">{{ localRecord.modelMag || '-' }}</div><input v-else type="text" v-model="localRecord.modelMag" class="edit-select" /></td>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Plant</span></td>
              <td style="width: 30%;"><div v-if="!isEditing" class="field-value">{{ localRecord.plantMag || '-' }}</div><input v-else type="text" v-model="localRecord.plantMag" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Specification</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.specMag || '-' }}</div><input v-else type="text" v-model="localRecord.specMag" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Trigger limit</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.triggerMag || '-' }}</div><input v-else type="text" v-model="localRecord.triggerMag" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">DO Number</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.doNo || '-' }}</div><input v-else type="text" v-model="localRecord.doNo" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Plating Date</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.platDateMag || '-' }}</div><input v-else type="date" v-model="localRecord.platDateMag" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Plating Line</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.platLineMag || '-' }}</div><input v-else type="text" v-model="localRecord.platLineMag" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Magnet Pack Number</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.magPackNo || '-' }}</div><input v-else type="text" v-model="localRecord.magPackNo" class="edit-select" /></td>
            </tr>
          </tbody>
        </table>

        <!-- Batch Number Statistics Sub Panel inside Outgoing Magnet -->
        <div style="margin-top: 15px; border-top: 1px solid #ccc; padding-top: 10px;">
          <div style="background-color: #DADADA; padding: 4px 10px; font-weight: bold; font-size: 13px; margin-bottom: 5px;">Batch Number Statistics</div>
          <div style="overflow-x: auto;">
            <table class="grid-table" style="width: 100%; border-collapse: collapse;">
              <thead>
                <tr style="background-color: #f2f2f2; font-size: 11px;">
                  <th style="border: 1px solid #ccc; padding: 4px;">Batch Number</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Minimum</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Average</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Maximum</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Max Spec</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Trigger Limit</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Remark</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Swabbed by</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Erasure by</th>
                  <th style="border: 1px solid #ccc; padding: 4px;">Verified by</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="b in magnetModelBatches" :key="b.id">
                  <td style="border: 1px solid #ccc; padding: 4px; text-align: center; font-size: 11px;">{{ b.id }}</td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.min" class="grid-input" /><span v-else>{{ b.min }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.avg" class="grid-input" /><span v-else>{{ b.avg }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.max" class="grid-input" /><span v-else>{{ b.max }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.maxSpec" class="grid-input" /><span v-else>{{ b.maxSpec }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.triggerLimit" class="grid-input" /><span v-else>{{ b.triggerLimit }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.remark" class="grid-input" /><span v-else>{{ b.remark }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.swabbedBy" class="grid-input" /><span v-else>{{ b.swabbedBy }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.erasureBy" class="grid-input" /><span v-else>{{ b.erasureBy }}</span></td>
                  <td style="border: 1px solid #ccc; padding: 4px;"><input v-if="isEditing" v-model="b.verifiedBy" class="grid-input" /><span v-else>{{ b.verifiedBy }}</span></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </fieldset>

      <!-- System Information Section -->
      <fieldset v-if="!isEditing" class="fsMargin" style="margin: 0 0 15px 0;">
        <legend><b>System Information</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Creation Date</span></td>
              <td style="width: 30%;"><div class="field-value">{{ localRecord.createdTs || '27-March-2026 10:25:00 AM' }}</div></td>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Created By</span></td>
              <td style="width: 30%;"><div class="field-value">{{ localRecord.createdBy || 'qa-admin' }}</div></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Updated Date</span></td>
              <td><div class="field-value">{{ localRecord.updatedTs || '27-March-2026 11:15:30 AM' }}</div></td>
              <td class="labelBack"><span class="labelTitle">Updated By</span></td>
              <td><div class="field-value">{{ localRecord.updatedBy || 'qa-tech' }}</div></td>
            </tr>
          </tbody>
        </table>
      </fieldset>
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
.box-header {
  background-color: #c7c7c7;
  border-bottom: 2px solid #c7c7c7;
  margin: -2px -2px 0 -2px;
  padding: 10px;
}
.breadcrumb { font-size: 14px; font-weight: bold; }
.item-link { color: #0000EE; text-decoration: none; font-weight: bold; }
.item-link:hover { text-decoration: underline; }
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
.btn-primary { background-color: #8f3235; color: white; }
.btn-secondary { background-color: #a5a5a5; color: #fff; }

.sub-panel-wrapper { padding: 0 15px; }
.fsMargin {
  margin: 10px 0 15px 0;
  border: 1px solid #d1d1d1;
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

.grid-table th { background-color: #f2f2f2; font-weight: bold; text-align: left; }
.grid-table td { font-size: 11px; white-space: nowrap; border: 1px solid #ccc; }
.grid-input { width: 95%; border: 1px solid #eee; padding: 2px; font-size: 11px; }
</style>
