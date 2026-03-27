<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  record: Object,
  isCreating: Boolean
})

const emit = defineEmits(['back', 'save'])

const isEditing = ref(props.isCreating)
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

const typeOptions = ['--Please Select One--', 'Reduced', 'Normal']
</script>

<template>
  <div class="top-record-box custom-sampling-detail">
    <!-- Breadcrumbs -->
    <div class="sub-header box-header">
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
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Qty</span></td>
              <td style="width: 34%;"><div v-if="!isEditing" class="field-value">{{ localRecord.qty }}</div><input v-else type="text" v-model="localRecord.qty" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Type</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.type }}</div><select v-else v-model="localRecord.type" class="edit-select"><option v-for="opt in typeOptions" :value="opt" :key="opt">{{ opt }}</option></select></td>
              <td class="labelBack"><span class="labelTitle">Sampling Size</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.sSize }}</div><input v-else type="text" v-model="localRecord.sSize" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Rank 1 Sampling Size</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.r1 }}</div><input v-else type="text" v-model="localRecord.r1" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Rank 2 Sampling Size</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.r2 }}</div><input v-else type="text" v-model="localRecord.r2" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Rank 3 Sampling Size</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.r3 }}</div><input v-else type="text" v-model="localRecord.r3" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Rank 4 Sampling Size</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.r4 }}</div><input v-else type="text" v-model="localRecord.r4" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Rank 5 Sampling Size</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.r5 }}</div><input v-else type="text" v-model="localRecord.r5" class="edit-select" /></td>
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
</style>
