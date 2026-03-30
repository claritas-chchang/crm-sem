<script setup>
import { ref, watch } from 'vue'

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
    localRecord.value = JSON.parse(JSON.stringify(props.record))
    isEditing.value = false
  }
}

const goBack = () => {
  emit('back')
}

// Dropdown Options
const magnetizationOptions = ['UN-MAGNETIZED', 'MAGNETIZED']
const markingOptions = ['Yes', 'No']

const magProps = ref(localRecord.value.magProps || [
  { name: 'Br [ T ]', avg: '' },
  { name: 'HcJ [ kA/m ]', avg: '' },
  { name: 'HcB [ kA/m ]', avg: '' },
  { name: '(BH)max [kJ/m3]', avg: '' }
])

const prodMagProps = ref(localRecord.value.prodMagProps || [
  { name: 'Flux Meter (10-5 Wb.Ts)', avg: '', min: '', max: '' }
])

const addSpecs = ref(localRecord.value.addSpecs || [])
const addAddSpec = () => { addSpecs.value.push({ item: '', judgment: '', instrument: '', symbol: '' }) }
const removeAddSpec = (idx) => { addSpecs.value.splice(idx, 1) }

const itemOpts = ['Size', 'Visual', 'Weight', 'Dimension']
const judgmentOpts = ['OK', 'NG']
const instrumentOpts = ['Vernier Caliper', 'Micrometer', 'Scale']
const symbolOpts = ['VC-01', 'VC-02', 'MC-01', 'SC-01']

// Save override
const saveRecord = () => {
  emit('save', {
    ...localRecord.value,
    magProps: JSON.parse(JSON.stringify(magProps.value)),
    prodMagProps: JSON.parse(JSON.stringify(prodMagProps.value)),
    addSpecs: JSON.parse(JSON.stringify(addSpecs.value))
  })
  isEditing.value = false
}
</script>

<template>
  <div class="top-record-box custom-shipment-a-detail">
    <!-- Breadcrumbs -->
    <div class="sub-header box-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span class="breadcrumb" style="font-size: 14px;">
        <a href="#" class="item-link" @click.prevent="goBack" style="font-weight: bold; color: #0000EE;">SHIPMENT TYPE A RECORDS</a> 
        &gt; <span class="current-page" style="font-weight: normal; color: #333;">{{ isCreating ? 'Create New' : (localRecord.reportId || 'New') }}</span>
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
      <!-- Report Header Information -->
      <fieldset class="fsMargin">
        <legend><b>Report Information</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Report ID</span></td>
              <td style="width: 30%;"><div v-if="!isCreating" class="field-value">{{ localRecord.reportId }}</div><input v-else type="text" v-model="localRecord.reportId" class="edit-select" /></td>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Issued Date</span></td>
              <td style="width: 30%;">
                <div v-if="!isEditing" class="field-value">{{ localRecord.issuedDate }}</div>
                <input v-else type="date" v-model="localRecord.issuedDate" class="edit-select" />
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Customer</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.customer }}</div><input v-else type="text" v-model="localRecord.customer" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Material</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.material }}</div><input v-else type="text" v-model="localRecord.material" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Our Code No</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.codeNo }}</div><input v-else type="text" v-model="localRecord.codeNo" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Customer's P/O No</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.customerPo }}</div><input v-else type="text" v-model="localRecord.customerPo" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Customer's Dwg / Part No</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.customerDwg }}</div><input v-else type="text" v-model="localRecord.customerDwg" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Our P/O No</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.ourPo }}</div><input v-else type="text" v-model="localRecord.ourPo" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Quantity</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.quantity }}</div><input v-else type="text" v-model="localRecord.quantity" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Unit Weight</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.unitWeight }}</div><input v-else type="text" v-model="localRecord.unitWeight" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Magnetization Through</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.magThrough }}</div><input v-else type="text" v-model="localRecord.magThrough" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Magnetization</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.magnetization }}</div>
                <select v-else v-model="localRecord.magnetization" class="edit-select">
                  <option v-for="opt in magnetizationOptions" :key="opt" :value="opt">{{ opt }}</option>
                </select>
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Marking</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.marking }}</div>
                <select v-else v-model="localRecord.marking" class="edit-select">
                  <option v-for="opt in markingOptions" :key="opt" :value="opt">{{ opt }}</option>
                </select>
              </td>
              <td class="labelBack"><span class="labelTitle">Dimension</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.dimension }}</div><input v-else type="text" v-model="localRecord.dimension" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Notes</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.notes }}</div><input v-else type="text" v-model="localRecord.notes" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Judgement</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.judgement }}</div><input v-else type="text" v-model="localRecord.judgement" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Approved by</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.approvedBy }}</div><input v-else type="text" v-model="localRecord.approvedBy" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Checked by</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.checkedBy }}</div><input v-else type="text" v-model="localRecord.checkedBy" class="edit-select" /></td>
            </tr>
          </tbody>
        </table>
      </fieldset>

      <!-- System Information -->
      <fieldset v-if="!isEditing" class="fsMargin">
        <legend><b>System Information</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Created Date</span></td>
              <td style="width: 30%;"><div class="field-value">{{ localRecord.createdTs || '30-March-2026 09:25:05 AM' }}</div></td>
              <td class="labelBack" style="width: 20%;"><span class="labelTitle">Created By</span></td>
              <td style="width: 30%;"><div class="field-value">{{ localRecord.createdBy || 'qa-admin' }}</div></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Updated Date</span></td>
              <td><div class="field-value">{{ localRecord.updatedTs || '30-March-2026 10:15:30 AM' }}</div></td>
              <td class="labelBack"><span class="labelTitle">Updated By</span></td>
              <td><div class="field-value">{{ localRecord.updatedBy || 'qa-tech' }}</div></td>
            </tr>
          </tbody>
        </table>
      </fieldset>
    </div>
  </div>

  <!-- Magnetic Properties Sub-panel -->
  <div class="sub-panel-wrapper">
      <div class="sub-panel-header">
        <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        <span style="font-size: 14px; font-weight: bold; color: #333; text-transform: uppercase;">Magnetic Properties</span>
      </div>
      <div class="sub-panel-body">
        <div class="sub-panel-inner-box">
          <div class="table-scroll-container">
            <table class="dim-table" style="width: 100%; border-collapse: collapse;">
              <thead>
                <tr>
                  <th>Name</th>
                  <th>AVG</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, idx) in magProps" :key="idx">
                  <td>{{ row.name }}</td>
                  <td>
                    <span v-if="!isEditing">{{ row.avg }}</span>
                    <input v-else type="text" v-model="row.avg" class="edit-select" style="width: 100%;" />
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <!-- Product Magnetic Properties Sub-panel -->
    <div class="sub-panel-wrapper">
      <div class="sub-panel-header">
        <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        <span style="font-size: 14px; font-weight: bold; color: #333; text-transform: uppercase;">Product Magnetic Properties</span>
      </div>
      <div class="sub-panel-body">
        <div class="sub-panel-inner-box">
          <div class="table-scroll-container">
            <table class="dim-table" style="width: 100%; border-collapse: collapse;">
              <thead>
                <tr>
                  <th>Name</th>
                  <th>AVG</th>
                  <th>MIN</th>
                  <th>MAX</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, idx) in prodMagProps" :key="idx">
                  <td>{{ row.name }}</td>
                  <td>
                    <span v-if="!isEditing">{{ row.avg }}</span>
                    <input v-else type="text" v-model="row.avg" class="edit-select" style="width: 100%;" />
                  </td>
                  <td>
                    <span v-if="!isEditing">{{ row.min }}</span>
                    <input v-else type="text" v-model="row.min" class="edit-select" style="width: 100%;" />
                  </td>
                  <td>
                    <span v-if="!isEditing">{{ row.max }}</span>
                    <input v-else type="text" v-model="row.max" class="edit-select" style="width: 100%;" />
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <!-- Additional Specification Sub-panel -->
    <div class="sub-panel-wrapper">
      <div class="sub-panel-header">
        <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        <span style="font-size: 14px; font-weight: bold; color: #333; text-transform: uppercase;">Additional Specification</span>
      </div>
      <div class="sub-panel-body">
        <div class="sub-panel-inner-box">
          
          <div v-if="isEditing" class="action-row toolbar-row" style="border-bottom: 1px solid #ddd; padding: 6px 10px; background: #f7f7f7;">
            <button class="btn btn-secondary" style="font-size: 11px; padding: 4px 10px;" @click.prevent="addAddSpec">Add Record</button>
          </div>

          <div class="table-scroll-container">
            <table class="dim-table" style="width: 100%; border-collapse: collapse;">
              <thead>
                <tr>
                  <th v-if="isEditing" style="width: 40px; text-align: center;">Action</th>
                  <th>Item</th>
                  <th>Judgment</th>
                  <th>Instrument</th>
                  <th>The Symbol for Instrument</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, idx) in addSpecs" :key="idx">
                  <td v-if="isEditing" style="text-align: center;">
                    <button class="btn btn-secondary" style="font-size: 10px; padding: 2px 6px;" @click.prevent="removeAddSpec(idx)">X</button>
                  </td>
                  <td>
                    <span v-if="!isEditing">{{ row.item }}</span>
                    <select v-else v-model="row.item" class="edit-select" style="width: 100%;">
                      <option v-for="opt in itemOpts" :key="opt" :value="opt">{{ opt }}</option>
                    </select>
                  </td>
                  <td>
                    <span v-if="!isEditing">{{ row.judgment }}</span>
                    <select v-else v-model="row.judgment" class="edit-select" style="width: 100%;">
                      <option v-for="opt in judgmentOpts" :key="opt" :value="opt">{{ opt }}</option>
                    </select>
                  </td>
                  <td>
                    <span v-if="!isEditing">{{ row.instrument }}</span>
                    <select v-else v-model="row.instrument" class="edit-select" style="width: 100%;">
                      <option v-for="opt in instrumentOpts" :key="opt" :value="opt">{{ opt }}</option>
                    </select>
                  </td>
                  <td>
                    <span v-if="!isEditing">{{ row.symbol }}</span>
                    <select v-else v-model="row.symbol" class="edit-select" style="width: 100%;">
                      <option v-for="opt in symbolOpts" :key="opt" :value="opt">{{ opt }}</option>
                    </select>
                  </td>
                </tr>
                <tr v-if="addSpecs.length === 0">
                  <td :colspan="isEditing ? 5 : 4" style="text-align: center; color: #999; padding: 15px;">No records added</td>
                </tr>
              </tbody>
            </table>
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
.breadcrumb { font-size: 14px; font-weight: bold; }
.item-link { color: #0000EE; text-decoration: none; font-weight: bold; }
.item-link:hover { text-decoration: underline; }
.current-page { font-weight: normal; color: #333; }

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

/* Sub-panel styling */
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
.dim-table th, .dim-table td { padding: 5px 8px; font-size: 11px; border: 1px solid #eee; white-space: nowrap; }
.dim-table th { background: #f2f2f2; font-weight: bold; border-bottom: 2px solid #ddd; text-align: left; }
.dim-table tbody tr:nth-child(odd) { background-color: #ffffff; }
.dim-table tbody tr:nth-child(even) { background-color: #f8f8f8; }
</style>
