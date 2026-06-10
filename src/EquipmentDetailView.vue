<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  record: Object,
  isCreating: Boolean,
  isEditing: { type: Boolean, default: false },
  isModal: { type: Boolean, default: false }
})

const emit = defineEmits(['back', 'save'])

const isEditing = ref(props.isEditing || props.isCreating)

// Parse a "Gauge Block Used" string (e.g. "3.0 & 50.0mm") into nominal values ["3.0mm", "50.0mm"]
const parseGaugeNominals = (gb) => {
  if (!gb) return []
  return gb.split('&').map(s => {
    let v = s.trim()
    if (v && !/mm$/i.test(v)) v += 'mm'
    return v
  }).filter(Boolean)
}

// Ensure a record carries pictures[] and a structured gaugeBlocks[] ({ nominal, serial })
const normalizeRecord = (rec) => {
  const copy = JSON.parse(JSON.stringify(rec || {}))
  if (!Array.isArray(copy.pictures)) copy.pictures = []
  if (!Array.isArray(copy.gaugeBlocks)) {
    copy.gaugeBlocks = parseGaugeNominals(copy.gaugeBlock).map(n => ({ nominal: n, serial: '' }))
  }
  return copy
}

const localRecord = ref(normalizeRecord(props.record))

watch(() => props.record, (newVal) => {
  localRecord.value = normalizeRecord(newVal)
}, { deep: true })

// Rebuild gaugeBlocks from the current gaugeBlock string, preserving serials where the nominal is unchanged
const rebuildGaugeBlocks = () => {
  const noms = parseGaugeNominals(localRecord.value.gaugeBlock)
  const prev = Array.isArray(localRecord.value.gaugeBlocks) ? localRecord.value.gaugeBlocks : []
  localRecord.value.gaugeBlocks = noms.map((n, i) => ({
    nominal: n,
    serial: (prev[i] && prev[i].nominal === n) ? prev[i].serial : ''
  }))
}

const startEdit = () => {
  isEditing.value = true
}

const cancelEdit = () => {
  if (props.isCreating) {
    emit('back')
  } else {
    localRecord.value = normalizeRecord(props.record)
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

/* ---- Dropdown options & cascading logic ---- */
const categoryOptions = ['--Please Select One--', 'Comparator', 'Micrometer', 'Pin Gauge', 'Others']

const typeOptionsByCategory = {
  Micrometer: ['0-25mm', '25-50mm', '50-75mm', '75-100mm'],
  Comparator: ['Digimatic Indicator (Comparator)']
}

const gaugeBlockByType = {
  '0-25mm': '3mm & 25mm',
  '25-50mm': '25mm & 50mm',
  '50-75mm': '50mm & 75mm',
  '75-100mm': '75mm & 100mm',
  'Digimatic Indicator (Comparator)': '3.0 & 50.0mm'
}

// Type field only shows for Micrometer & Comparator
const showTypeField = computed(() => Object.keys(typeOptionsByCategory).includes(localRecord.value.category))
const typeOptions = computed(() => ['--Please Select One--', ...(typeOptionsByCategory[localRecord.value.category] || [])])

// Gauge Block Used only shows when the selected Type has a mapping
const showGaugeField = computed(() => showTypeField.value && !!gaugeBlockByType[localRecord.value.type])
const gaugeOptions = computed(() => {
  const mapped = gaugeBlockByType[localRecord.value.type]
  return mapped ? ['--Please Select One--', mapped] : ['--Please Select One--']
})

const onCategoryChange = () => {
  // Reset dependent fields when category changes
  localRecord.value.type = ''
  localRecord.value.gaugeBlock = ''
  localRecord.value.gaugeBlocks = []
}

const onTypeChange = () => {
  // Auto-fill the only valid gauge block for the chosen type, then rebuild the structured list
  localRecord.value.gaugeBlock = gaugeBlockByType[localRecord.value.type] || ''
  rebuildGaugeBlocks()
}

/* ---- Pictures (max 5) ---- */
const MAX_PICTURES = 5
const pictureError = ref('')

const onPickFiles = (e) => {
  pictureError.value = ''
  const files = Array.from(e.target.files || [])
  const remaining = MAX_PICTURES - localRecord.value.pictures.length
  if (files.length > remaining) {
    pictureError.value = `You can upload a maximum of ${MAX_PICTURES} images.`
  }
  files.slice(0, Math.max(0, remaining)).forEach(file => {
    const reader = new FileReader()
    reader.onload = (ev) => {
      localRecord.value.pictures.push({ name: file.name, url: ev.target.result })
    }
    reader.readAsDataURL(file)
  })
  e.target.value = '' // allow re-selecting the same file
}

const removePicture = (idx) => {
  localRecord.value.pictures.splice(idx, 1)
}
</script>

<template>
  <div :class="['top-record-box custom-equipment-detail', { 'modal-layout': isModal }]">
    <!-- Breadcrumbs -->
    <div v-if="!isModal" class="sub-header box-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span class="breadcrumb">
        <a href="#" class="item-link" @click.prevent="goBack" style="font-weight: bold; color: #0000EE; text-decoration: underline;">EQUIPMENT RECORDS</a>
        &gt; <span class="current-page">{{ isCreating ? 'Create New' : localRecord.formNo }}</span>
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
        <legend><b>Equipment Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Form No.</span></td>
              <td style="width: 34%;"><div v-if="!isEditing" class="field-value">{{ localRecord.formNo }}</div><input v-else type="text" v-model="localRecord.formNo" class="edit-select" /></td>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Serial No.</span></td>
              <td style="width: 34%;"><div v-if="!isEditing" class="field-value">{{ localRecord.serialNo }}</div><input v-else type="text" v-model="localRecord.serialNo" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Sub No.</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.subNo }}</div><input v-else type="number" v-model="localRecord.subNo" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Specification</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.specification }}</div><input v-else type="text" v-model="localRecord.specification" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Revision</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.revision }}</div><input v-else type="number" v-model="localRecord.revision" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Equipment Category</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.category }}</div>
                <select v-else v-model="localRecord.category" class="edit-select" @change="onCategoryChange">
                  <option v-for="opt in categoryOptions" :value="opt === '--Please Select One--' ? '' : opt" :key="opt">{{ opt }}</option>
                </select>
              </td>
            </tr>
            <tr v-if="showTypeField">
              <td class="labelBack"><span class="labelTitle">Type</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.type }}</div>
                <select v-else v-model="localRecord.type" class="edit-select" @change="onTypeChange">
                  <option v-for="opt in typeOptions" :value="opt === '--Please Select One--' ? '' : opt" :key="opt">{{ opt }}</option>
                </select>
              </td>
              <template v-if="showGaugeField">
                <td class="labelBack"><span class="labelTitle">Gauge Block Used</span></td>
                <td>
                  <div v-if="!isEditing" class="field-value">{{ localRecord.gaugeBlock }}</div>
                  <select v-else v-model="localRecord.gaugeBlock" class="edit-select">
                    <option v-for="opt in gaugeOptions" :value="opt === '--Please Select One--' ? '' : opt" :key="opt">{{ opt }}</option>
                  </select>
                </td>
              </template>
              <template v-else>
                <td class="labelBack"><span class="labelTitle"></span></td>
                <td></td>
              </template>
            </tr>
            <tr v-if="showGaugeField && localRecord.gaugeBlocks && localRecord.gaugeBlocks.length">
              <td class="labelBack" style="vertical-align: top; padding-top: 8px;"><span class="labelTitle">Gauge Blocks</span></td>
              <td colspan="3">
                <table class="gauge-block-table">
                  <thead>
                    <tr>
                      <th>Nominal Value</th>
                      <th>Serial No.</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(gb, gi) in localRecord.gaugeBlocks" :key="gi">
                      <td>{{ gb.nominal }}</td>
                      <td>
                        <div v-if="!isEditing" class="field-value" style="padding-left: 0;">{{ gb.serial || '-' }}</div>
                        <input v-else type="text" v-model="gb.serial" class="edit-select" placeholder="Enter serial no." />
                      </td>
                    </tr>
                  </tbody>
                </table>
              </td>
            </tr>
            <tr>
              <td class="labelBack" style="vertical-align: top; padding-top: 8px;"><span class="labelTitle">Pictures</span></td>
              <td colspan="3">
                <div class="picture-area">
                  <div v-if="localRecord.pictures && localRecord.pictures.length" class="picture-grid">
                    <div v-for="(pic, idx) in localRecord.pictures" :key="idx" class="picture-thumb">
                      <img :src="pic.url" :alt="pic.name" />
                      <button v-if="isEditing" type="button" class="pic-remove" @click="removePicture(idx)" title="Remove">&times;</button>
                    </div>
                  </div>
                  <div v-else class="field-value" style="padding-left: 0;">No images uploaded.</div>

                  <div v-if="isEditing" class="picture-upload">
                    <label class="upload-btn">
                      Upload Images
                      <input type="file" accept="image/*" multiple @change="onPickFiles" :disabled="localRecord.pictures.length >= MAX_PICTURES" hidden />
                    </label>
                    <span class="upload-hint">{{ localRecord.pictures.length }}/{{ MAX_PICTURES }} images (max {{ MAX_PICTURES }})</span>
                    <span v-if="pictureError" class="upload-error">{{ pictureError }}</span>
                  </div>
                </div>
              </td>
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
.custom-equipment-detail {
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
.labelTitle { font-size: 13px; font-weight: normal; color: black; font-family: var(--font-family); }
.field-value { padding: 4px 12px; font-size: 13px; color: #333; font-family: var(--font-family); }
.edit-select { width: 80%; padding: 4px; border: 1px solid #ccc; font-size: 13px; box-sizing: border-box; }
.item-link { color: #0000EE; text-decoration: none; font-weight: bold; }
.item-link:hover { text-decoration: underline; }

/* Pictures */
.picture-area { padding: 6px 12px; }
.picture-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 8px;
}
.picture-thumb {
  position: relative;
  width: 90px;
  height: 90px;
  border: 1px solid #ccc;
  background-color: #fff;
  overflow: hidden;
}
.picture-thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.pic-remove {
  position: absolute;
  top: 1px;
  right: 1px;
  width: 18px;
  height: 18px;
  line-height: 16px;
  text-align: center;
  border: none;
  border-radius: 2px;
  background-color: rgba(165,28,34,0.85);
  color: #fff;
  font-size: 14px;
  cursor: pointer;
}
.pic-remove:hover { background-color: #a51c22; }
.picture-upload {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}
.upload-btn {
  display: inline-block;
  background-color: #a5a5a5;
  color: #fff;
  font-size: 12px;
  font-weight: bold;
  padding: 6px 14px;
  border-radius: 2px;
  cursor: pointer;
}
.upload-btn:hover { background-color: #888; }
.upload-hint { font-size: 12px; color: #666; }
.upload-error { font-size: 12px; color: #a51c22; }

/* Gauge Blocks table */
.gauge-block-table {
  border-collapse: collapse;
  margin: 4px 0 4px 12px;
}
.gauge-block-table th,
.gauge-block-table td {
  border: 1px solid #ccc;
  padding: 4px 10px;
  font-size: 12px;
  text-align: left;
}
.gauge-block-table th {
  background-color: #ececec;
  font-weight: bold;
}
.gauge-block-table .edit-select { width: 160px; }
</style>
