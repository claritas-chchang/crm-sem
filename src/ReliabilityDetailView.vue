<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  record: { type: Object, default: () => null },
  isCreating: { type: Boolean, default: false }
})

const emit = defineEmits(['back', 'save'])

const testTypeOptions = [
  'Pressure Cooker',
  'Pull Test',
  'Quench Test',
  'Thermal Demagnetisation Test',
  'Routine Reliability Test'
]

const testTypeData = {
  'Pressure Cooker': {
    condition: '120ºc x 2atm x 48hrs',
    frequency: '5 pcs / line / day / any model',
    criteria: 'No harmful change such as worsened stain, corrosion, rust, or swelling before and after the test'
  },
  'Pull Test': {
    condition: 'Adhesive Ratio :  5 (AV138) : 2 (HV998)\nOven Curing :  4 hours (40oC)\nCooling :  15 minutes (room temperature)',
    frequency: '3 pcs / line / day / any model',
    criteria: '≥ 100 kg.f/cm2'
  },
  'Quench Test': {
    condition: 'i) 180ºc x 1hr (HSA)\nii) 250ºc x 1hr (WDA)',
    frequency: '1 pc / model / day / any line',
    criteria: '*No blister, crack, lifting of plating after quench\n*No Ni peeled off by Tape Test'
  },
  'Thermal Demagnetisation Test': {
    condition: '',
    frequency: '',
    criteria: ''
  },
  'Routine Reliability Test': {
    condition: '',
    frequency: '',
    criteria: ''
  }
}

const localRecord = ref({})
const isEditing = ref(props.isCreating)

watch(() => props.record, (newVal) => {
  if (newVal) {
    localRecord.value = { ...newVal }
  } else {
    localRecord.value = {
      id: '',
      testType: 'Pressure Cooker',
      testCondition: testTypeData['Pressure Cooker'].condition,
      samplingFrequency: testTypeData['Pressure Cooker'].frequency,
      criteria: testTypeData['Pressure Cooker'].criteria,
      platingDate: '',
      preparedBy: '',
      checkedBy: '',
      approvedBy: ''
    }
  }
}, { immediate: true, deep: true })

const handleTestTypeChange = () => {
  const data = testTypeData[localRecord.value.testType]
  if (data) {
    localRecord.value.testCondition = data.condition
    localRecord.value.samplingFrequency = data.frequency
    localRecord.value.criteria = data.criteria
  }
}

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
  if (!localRecord.value.id) {
    localRecord.value.id = 'REL-' + Math.floor(Math.random() * 10000).toString().padStart(4, '0')
  }
  emit('save', { ...localRecord.value })
  isEditing.value = false
}

const goBack = () => {
  emit('back')
}
</script>

<template>
  <div class="top-record-box custom-reliability-detail">
    <!-- Breadcrumbs -->
    <div class="sub-header box-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span class="breadcrumb" style="font-size: 14px;">
        <a href="#" class="item-link" @click.prevent="goBack" style="font-weight: bold; color: #0000EE;">RELIABILITY RECORDS</a> 
        &gt; <span class="current-page" style="font-weight: normal; color: #333;">{{ isCreating ? 'Create New' : localRecord.id }}</span>
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
      <fieldset class="fsMargin" style="margin: 0 0 15px 0;">
        <legend><b>Reliability Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Reliability Test</span></td>
              <td style="width: 30%;">
                <div v-if="!isEditing" class="field-value">{{ localRecord.testType }}</div>
                <select v-else v-model="localRecord.testType" class="edit-select" @change="handleTestTypeChange">
                  <option v-for="opt in testTypeOptions" :value="opt" :key="opt">{{ opt }}</option>
                </select>
              </td>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Plating Date</span></td>
              <td style="width: 30%;">
                <div v-if="!isEditing" class="field-value">{{ localRecord.platingDate || '-' }}</div>
                <input v-else type="date" v-model="localRecord.platingDate" class="edit-select" />
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Test Condition</span></td>
              <td colspan="3">
                <div v-if="!isEditing" class="field-value" style="white-space: pre-wrap; word-break: break-word; width: 30%;">{{ localRecord.testCondition || '-' }}</div>
                <textarea v-else v-model="localRecord.testCondition" class="edit-select" rows="3" style="width: 30%; resize: vertical;"></textarea>
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Sampling Frequency</span></td>
              <td colspan="3">
                <div v-if="!isEditing" class="field-value" style="white-space: pre-wrap; word-break: break-word; width: 30%;">{{ localRecord.samplingFrequency || '-' }}</div>
                <input v-else type="text" v-model="localRecord.samplingFrequency" class="edit-select" style="width: 30%;" />
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Criteria</span></td>
              <td colspan="3">
                <div v-if="!isEditing" class="field-value" style="white-space: pre-wrap; word-break: break-word; width: 30%;">{{ localRecord.criteria || '-' }}</div>
                <textarea v-else v-model="localRecord.criteria" class="edit-select" rows="3" style="width: 30%; resize: vertical;"></textarea>
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Prepared By</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.preparedBy || '-' }}</div>
                <input v-else type="text" v-model="localRecord.preparedBy" class="edit-select" />
              </td>
              <td class="labelBack"><span class="labelTitle">Checked By</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.checkedBy || '-' }}</div>
                <input v-else type="text" v-model="localRecord.checkedBy" class="edit-select" />
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Approved By</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.approvedBy || '-' }}</div>
                <input v-else type="text" v-model="localRecord.approvedBy" class="edit-select" />
              </td>
              <td colspan="2"></td>
            </tr>
          </tbody>
        </table>
      </fieldset>

      <!-- System Information Section -->
      <fieldset v-if="!isEditing" class="fsMargin" style="margin: 0 0 15px 0;">
        <legend><b>System Information</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Creation Date</span></td>
              <td style="width: 30%;"><div class="field-value">27-March-2026 10:25:00 AM</div></td>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Created By</span></td>
              <td style="width: 30%;"><div class="field-value">qa-admin</div></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Updated Date</span></td>
              <td><div class="field-value">27-March-2026 11:15:30 AM</div></td>
              <td class="labelBack"><span class="labelTitle">Updated By</span></td>
              <td><div class="field-value">qa-tech</div></td>
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
.box-header {
  background-color: #c7c7c7;
  border-bottom: 2px solid #c7c7c7;
  margin: -2px -2px 0 -2px;
  padding: 10px;
}
.folder-svg {
  margin-right: 5px;
  vertical-align: middle;
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

.form-wrapper { padding: 0 15px; }
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
</style>
