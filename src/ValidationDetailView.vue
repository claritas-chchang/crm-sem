<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  record: {
    type: Object,
    required: true
  },
  isCreating: {
    type: Boolean,
    default: false
  },
  isEditing: {
    type: Boolean,
    default: false
  },
  isModal: {
    type: Boolean,
    default: false
  },
  equipmentRecords: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['back', 'save'])

const isEditing = ref(props.isEditing || props.isCreating)
const localRecord = ref({ ...props.record })

/* ---- Link to Equipment module by Serial No. ----
   When Equipment Serial No. exactly matches an Equipment record, auto-fill
   Equipment, Specification, Range and Picture (read-only), and capture the
   Equipment Category that drives the conditional subpanels. */
const linkedEquipment = computed(() =>
  (props.equipmentRecords || []).find(e => e.serialNo && e.serialNo === localRecord.value.eqSerial) || null
)

const applyEquipmentLink = (eqp) => {
  if (!eqp) return
  localRecord.value.eq = eqp.specification || ''
  localRecord.value.spec = eqp.specification || ''
  localRecord.value.range = eqp.type || ''
  localRecord.value.eqPictures = Array.isArray(eqp.pictures) ? eqp.pictures : []
  localRecord.value.category = eqp.category || ''
}

watch(() => localRecord.value.eqSerial, () => {
  if (linkedEquipment.value) applyEquipmentLink(linkedEquipment.value)
}, { immediate: true })

/* ---- Category-driven subpanels ---- */
const okNgRowOptions = ['', 'OK', 'NG']
const shiftDNOptions = ['', 'D', 'N']
const shiftMNOptions = ['', 'Morning', 'Night']

// Comparator / Micrometer: each validation is a "card" with shared header fields plus a
// 3-column gauge table (0mm zero reference + the two gauge blocks from the linked Equipment).
const cardGaugeRows = [
  { key: 'serial', label: 'Block Gauge Serial No.', readonly: true },
  { key: 'nominal', label: 'Block Gauge Nominal Value', readonly: true },
  { key: 'minSpec', label: 'Min Spec', type: 'text' },
  { key: 'maxSpec', label: 'Max Spec', type: 'text' },
  { key: 'r1', label: 'Reading 1', type: 'text' },
  { key: 'r2', label: 'Reading 2', type: 'text' },
  { key: 'r3', label: 'Reading 3', type: 'text' },
  { key: 'judgement', label: 'Judgement', type: 'okng' }
]

// Validation Record columns for Pin Gauge (image 4)
const pinGaugeCols = [
  { key: 'date', label: 'Date', type: 'text' },
  { key: 'shift', label: 'Shift (D/N)', type: 'shiftDN' },
  { key: 'micSerial', label: 'Micrometer Serial No.', type: 'text' },
  { key: 'micZero', label: 'Micrometer "Zero" Check Result', type: 'okng' },
  { key: 'gpNominal', label: 'GP Nominal Value (mm)', type: 'text' },
  { key: 'gpFront1', label: 'GP Front Reading 1', type: 'text' },
  { key: 'gpFront2', label: 'GP Front Reading 2', type: 'text' },
  { key: 'gpEnd1', label: 'GP End Reading 1', type: 'text' },
  { key: 'gpEnd2', label: 'GP End Reading 2', type: 'text' },
  { key: 'gpResult', label: 'GP Result', type: 'okng' },
  { key: 'npNominal', label: 'NP Nominal Value (mm)', type: 'text' },
  { key: 'npFront1', label: 'NP Front Reading 1', type: 'text' },
  { key: 'npFront2', label: 'NP Front Reading 2', type: 'text' },
  { key: 'npResult', label: 'NP Result', type: 'okng' },
  { key: 'checkedBy', label: 'Checked by', type: 'text' },
  { key: 'verifiedBy', label: 'Verified by', type: 'text' }
]

// Induction Check columns (image 5) - shown for all categories
const inductionCols = [
  { key: 'date', label: 'Date', type: 'text' },
  { key: 'shift', label: 'Shift', type: 'shiftMN' },
  { key: 'gpCheck', label: 'Green Paper (GP) Check', type: 'okng' },
  { key: 'gaussReading', label: 'Gauss Meter Reading (mT, if GP NG)', type: 'text' },
  { key: 'checkedBy', label: 'Checked by (ID)', type: 'text' },
  { key: 'gpAfterDemag', label: 'After De-mag GP Check', type: 'okng' },
  { key: 'demagBy', label: 'De-magnetised by (ID)', type: 'text' },
  { key: 'verifiedBy', label: 'Verified by (ID)', type: 'text' }
]

const isComparatorMicrometer = computed(() => ['Comparator', 'Micrometer'].includes(localRecord.value.category))
const isPinGauge = computed(() => localRecord.value.category === 'Pin Gauge')
const showValidationRecord = computed(() => isComparatorMicrometer.value || isPinGauge.value)
const showInductionCheck = computed(() => ['Comparator', 'Micrometer', 'Pin Gauge', 'Others'].includes(localRecord.value.category))

const validationRecordTitle = computed(() => isPinGauge.value
  ? 'Validation Record (Pin Gauge)'
  : 'Validation Record (Comparator / Micrometer)')

const showValRecordPanel = ref(true)
const showInductionPanel = ref(true)
const toggleValRecordPanel = () => { showValRecordPanel.value = !showValRecordPanel.value }
const toggleInductionPanel = () => { showInductionPanel.value = !showInductionPanel.value }

function ensureSubpanelArrays () {
  if (!Array.isArray(localRecord.value.validationCards)) localRecord.value.validationCards = []
  if (!Array.isArray(localRecord.value.pinGaugeRows)) localRecord.value.pinGaugeRows = []
  if (!Array.isArray(localRecord.value.inductionRows)) localRecord.value.inductionRows = []
}
ensureSubpanelArrays()

const blankRow = (cols) => {
  const r = {}
  cols.forEach(c => { r[c.key] = '' })
  return r
}

// Build the 3 gauge columns for a new Comparator/Micrometer validation card:
// column 1 = 0mm zero reference, then the gauge blocks (serial + nominal) from the linked Equipment.
const buildCardColumns = () => {
  const eqBlocks = (linkedEquipment.value && Array.isArray(linkedEquipment.value.gaugeBlocks))
    ? linkedEquipment.value.gaugeBlocks : []
  const blank = () => ({ serial: '', nominal: '', minSpec: '', maxSpec: '', r1: '', r2: '', r3: '', judgement: '' })
  const cols = [{ ...blank(), serial: '-', nominal: '0mm' }]
  eqBlocks.forEach(gb => cols.push({ ...blank(), serial: gb.serial || '', nominal: gb.nominal || '' }))
  return cols
}
const addValidationCard = () => {
  localRecord.value.validationCards.push({
    date: '', shift: '', checkedBy: '', verifiedBy: '', remark: '',
    columns: buildCardColumns()
  })
}
const removeValidationCard = (i) => { localRecord.value.validationCards.splice(i, 1) }

const addPinRow = () => { localRecord.value.pinGaugeRows.push(blankRow(pinGaugeCols)) }
const removePinRow = (i) => { localRecord.value.pinGaugeRows.splice(i, 1) }
const addInductionRow = () => { localRecord.value.inductionRows.push(blankRow(inductionCols)) }
const removeInductionRow = (i) => { localRecord.value.inductionRows.splice(i, 1) }

const optionsForType = (type) => {
  if (type === 'okng') return okNgRowOptions
  if (type === 'shiftDN') return shiftDNOptions
  if (type === 'shiftMN') return shiftMNOptions
  return null
}

// Dropdown Options
const frequencyOptions = ['--Please Select One--', 'Daily', 'Weekly', 'Monthly', 'Quarterly', 'Half-yearly', 'Annual']
const shiftOptions = ['--Please Select One--', 'Day', 'Night']

watch(() => props.record, (newVal) => {
  localRecord.value = { ...newVal }
  ensureSubpanelArrays()
}, { deep: true })

const startEdit = () => {
  isEditing.value = true
}

const cancelEdit = () => {
  localRecord.value = { ...props.record }
  isEditing.value = false
}

const saveRecord = () => {
  emit('save', localRecord.value)
  isEditing.value = false
}

const goBack = () => {
  emit('back')
}

const formatDisplayDatetime = (dt) => {
  if (!dt) return '-'
  try {
    const d = new Date(dt.replace(' ', 'T'))
    if (isNaN(d.getTime())) return dt
    
    const options = { 
      day: '2-digit', 
      month: 'long', 
      year: 'numeric', 
      hour: '2-digit', 
      minute: '2-digit', 
      second: '2-digit', 
      hour12: true 
    }
    return d.toLocaleString('en-GB', options).replace(/,/g, '')
  } catch (e) {
    return dt
  }
}

const formatDisplayDate = (dstr) => {
  if (!dstr) return '-'
  try {
    const d = new Date(dstr)
    if (isNaN(d.getTime())) return dstr
    const options = { day: '2-digit', month: 'long', year: 'numeric' }
    return d.toLocaleDateString('en-GB', options)
  } catch (e) {
    return dstr
  }
}

const fromInputDatetime = (val) => {
  if (!val) return ''
  return val.replace('T', ' ') + ':00'
}

// Custom Picker State
const showPicker = ref(null) // 'dateIssued', 'date', 'vDate', 'vDue'
const pickerDate = ref(new Date())
const pickerHour = ref(12)
const pickerMinute = ref(0)
const pickerIsAm = ref(true)

const openPicker = (field) => {
  showPicker.value = field
  const currentVal = localRecord.value[field]
  if (currentVal) {
    pickerDate.value = new Date(currentVal.replace(' ', 'T'))
    const h = pickerDate.value.getHours()
    pickerHour.value = h === 0 ? 12 : (h > 12 ? h - 12 : h)
    pickerIsAm.value = h < 12
    pickerMinute.value = pickerDate.value.getMinutes()
  } else {
    pickerDate.value = new Date()
    pickerHour.value = 12
    pickerMinute.value = 0
    pickerIsAm.value = true
  }
}

const closePicker = () => {
  showPicker.value = null
}

const selectDay = (day) => {
  pickerDate.value = new Date(pickerDate.value.getFullYear(), pickerDate.value.getMonth(), day)
}

const selectNow = () => {
  const now = new Date()
  pickerDate.value = now
  const h = now.getHours()
  pickerHour.value = h === 0 ? 12 : (h > 12 ? h - 12 : h)
  pickerIsAm.value = h < 12
  pickerMinute.value = now.getMinutes()
}

const confirmPicker = () => {
  const finalDate = new Date(pickerDate.value)
  let h = parseInt(pickerHour.value)
  if (pickerIsAm.value && h === 12) h = 0
  if (!pickerIsAm.value && h < 12) h += 12
  finalDate.setHours(h, pickerMinute.value, 0)
  
  const YYYY = finalDate.getFullYear()
  const MM = String(finalDate.getMonth() + 1).padStart(2, '0')
  const DD = String(finalDate.getDate()).padStart(2, '0')
  const hh = String(finalDate.getHours()).padStart(2, '0')
  const mm = String(finalDate.getMinutes()).padStart(2, '0')
  const ss = '00'

  if (['vDate', 'vDue', 'calDate', 'calDue'].includes(showPicker.value)) {
    localRecord.value[showPicker.value] = `${YYYY}-${MM}-${DD}`
  } else {
    localRecord.value[showPicker.value] = `${YYYY}-${MM}-${DD} ${hh}:${mm}:${ss}`
  }
  showPicker.value = null
}

const daysInMonth = computed(() => {
  const year = pickerDate.value.getFullYear()
  const month = pickerDate.value.getMonth()
  return new Date(year, month + 1, 0).getDate()
})

const firstDayOfMonth = computed(() => {
  const year = pickerDate.value.getFullYear()
  const month = pickerDate.value.getMonth()
  return new Date(year, month, 1).getDay()
})

const calendarMonths = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
const calendarYears = Array.from({ length: 21 }, (_, i) => 2020 + i)

const pickerMonth = computed({
  get: () => pickerDate.value.getMonth(),
  set: (val) => pickerDate.value = new Date(pickerDate.value.getFullYear(), val, 1)
})

const pickerYear = computed({
  get: () => pickerDate.value.getFullYear(),
  set: (val) => pickerDate.value = new Date(val, pickerDate.value.getMonth(), 1)
})

const prevMonth = () => {
  pickerDate.value = new Date(pickerDate.value.getFullYear(), pickerDate.value.getMonth() - 1, 1)
}
const nextMonth = () => {
  pickerDate.value = new Date(pickerDate.value.getFullYear(), pickerDate.value.getMonth() + 1, 1)
}
</script>

<template>
  <div class="validation-detail-page">
  <div :class="['top-record-box', { 'modal-layout': isModal }]">
    <!-- Breadcrumb Header (Matches ProductView exactly) -->
    <div v-if="!isModal" class="sub-header box-header">
      <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      <span class="breadcrumb" style="font-size: 14px;">
        <a href="#" class="item-link" @click.prevent="goBack" style="font-weight: bold; color: #0000EE;">VALIDATION RECORDS</a> 
        > <span style="font-weight: normal; color: #333;">{{ isCreating ? 'Create New' : (record.formNo || 'New') }}</span>
      </span>
    </div>

    <!-- Top Action Bar (Matches ProductView exactly) -->
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
      <!-- Section 1: Validation Details -->
      <fieldset class="fsMargin" style="margin: 0 0 15px 0;">
        <legend><b>Validation Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Form Number</span></td>
              <td style="width: 34%;"><div class="field-value">{{ record.formNo }}</div></td>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Revision</span></td>
              <td style="width: 34%;"><div v-if="!isEditing" class="field-value">{{ localRecord.revision }}</div><input v-else type="text" v-model="localRecord.revision" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Date Issued</span></td>
              <td style="position: relative;">
                <div v-if="!isEditing" class="field-value">{{ formatDisplayDatetime(localRecord.dateIssued) }}</div>
                <input v-else type="text" readonly :value="localRecord.dateIssued ? formatDisplayDatetime(localRecord.dateIssued) : ''" @click="openPicker('dateIssued')" class="edit-select date-input-text" />
              </td>
              <td class="labelBack"><span class="labelTitle">Title</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.title }}</div><input v-else type="text" v-model="localRecord.title" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Specification</span></td>
              <td><div class="field-value linked-value">{{ localRecord.spec || '-' }}</div></td>
              <td class="labelBack"><span class="labelTitle">Equipment</span></td>
              <td><div class="field-value linked-value">{{ localRecord.eq || '-' }}</div></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Equipment Serial No.</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.eqSerial }}</div>
                <template v-else>
                  <input type="text" v-model="localRecord.eqSerial" class="edit-select" placeholder="Enter equipment serial no." />
                  <div class="link-hint" :class="{ matched: linkedEquipment }">
                    {{ linkedEquipment ? '✓ Linked to ' + linkedEquipment.formNo + ' (' + linkedEquipment.category + ')' : 'No matching equipment found' }}
                  </div>
                </template>
              </td>
              <td class="labelBack"><span class="labelTitle">Frequency of Check</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.freq }}</div>
                <select v-else v-model="localRecord.freq" class="edit-select">
                  <option v-for="opt in frequencyOptions" :key="opt" :value="opt">{{ opt }}</option>
                </select>
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Date</span></td>
              <td style="position: relative;">
                <div v-if="!isEditing" class="field-value">{{ formatDisplayDatetime(localRecord.date) }}</div>
                <input v-else type="text" readonly :value="localRecord.date ? formatDisplayDatetime(localRecord.date) : ''" @click="openPicker('date')" class="edit-select date-input-text" />
              </td>
              <td class="labelBack"><span class="labelTitle">Shift</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.shift }}</div>
                <select v-else v-model="localRecord.shift" class="edit-select">
                  <option v-for="opt in shiftOptions" :key="opt" :value="opt">{{ opt }}</option>
                </select>
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Range</span></td>
              <td><div class="field-value linked-value">{{ localRecord.range || '-' }}</div></td>
              <td class="labelBack"><span class="labelTitle">Calibration Date</span></td>
              <td style="position: relative;">
                <div v-if="!isEditing" class="field-value">{{ formatDisplayDate(localRecord.calDate) }}</div>
                <input v-else type="text" readonly :value="localRecord.calDate ? formatDisplayDate(localRecord.calDate) : ''" @click="openPicker('calDate')" class="edit-select date-input-text" />
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Picture</span></td>
              <td>
                <div class="field-value linked-value" style="display: flex; gap: 6px; flex-wrap: wrap; align-items: center;">
                  <template v-if="localRecord.eqPictures && localRecord.eqPictures.length">
                    <span v-for="(p, i) in localRecord.eqPictures" :key="i" class="val-thumb"><img :src="p.url" :alt="p.name" /></span>
                  </template>
                  <template v-else>-</template>
                </div>
              </td>
              <td class="labelBack"><span class="labelTitle">Calibration Due Date</span></td>
              <td style="position: relative;">
                <div v-if="!isEditing" class="field-value">{{ formatDisplayDate(localRecord.calDue) }}</div>
                <input v-else type="text" readonly :value="localRecord.calDue ? formatDisplayDate(localRecord.calDue) : ''" @click="openPicker('calDue')" class="edit-select date-input-text" />
              </td>
            </tr>
          </tbody>
        </table>
      </fieldset>

      <!-- Section 3: System Information (Only visible in View mode) -->
      <fieldset v-if="!isEditing" class="fsMargin" style="margin: 0 0 15px 0;">
        <legend><b>System Information</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Created Date</span></td>
              <td style="width: 34%;"><div class="field-value">16-March-2026 12:58:05 PM</div></td>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Created By</span></td>
              <td style="width: 34%;"><div class="field-value">qa-admin-p2</div></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Last Updated Date</span></td>
              <td><div class="field-value">16-March-2026 12:59:47 PM</div></td>
              <td class="labelBack"><span class="labelTitle">Last Updated By</span></td>
              <td><div class="field-value">qa-tech-p2</div></td>
            </tr>
          </tbody>
        </table>
      </fieldset>
    </div>

    <!-- Custom Popup Calendar Picker -->
    <div v-if="showPicker" class="custom-calendar-popup">
      <div class="cal-header">
         <div class="cal-nav-btn" @click="prevMonth">
           <svg viewBox="0 0 24 24" width="16" height="16" fill="white"><path d="M15.41 7.41L14 6l-6 6 6 6 1.41-1.41L10.83 12z"/></svg>
         </div>
         <select v-model="pickerMonth" class="cal-select">
           <option v-for="(m, i) in calendarMonths" :key="m" :value="i">{{ m }}</option>
         </select>
         <select v-model="pickerYear" class="cal-select">
           <option v-for="y in calendarYears" :key="y" :value="y">{{ y }}</option>
         </select>
         <div class="cal-nav-btn" @click="nextMonth">
           <svg viewBox="0 0 24 24" width="16" height="16" fill="white"><path d="M10 6L8.59 7.41 13.17 12l-4.58 4.59L10 18l6-6z"/></svg>
         </div>
      </div>
      <div class="cal-body">
         <div class="cal-weekdays">
            <span>Su</span><span>Mo</span><span>Tu</span><span>We</span><span>Th</span><span>Fr</span><span>Sa</span>
         </div>
         <div class="cal-days">
            <span v-for="empty in firstDayOfMonth" :key="'e'+empty"></span>
            <span 
              v-for="d in daysInMonth" 
              :key="d" 
              class="cal-day" 
              :class="{ 'cal-today': d === pickerDate.getDate() }"
              @click="selectDay(d)"
            >{{ d }}</span>
         </div>
         
         <div class="cal-time-section" v-if="!['vDate', 'vDue', 'calDate', 'calDue'].includes(showPicker)">
           <div class="time-display">
             Time: {{ pickerHour }}:{{ String(pickerMinute).padStart(2, '0') }}:00 {{ pickerIsAm ? 'AM' : 'PM' }}
           </div>
           <div class="slider-group">
             <label>Hour</label>
             <input type="range" v-model="pickerHour" min="1" max="12" class="cal-slider" />
           </div>
           <div class="slider-group">
             <label>Minute</label>
             <input type="range" v-model="pickerMinute" min="0" max="59" class="cal-slider" />
           </div>
           <div style="margin-top:5px;">
              <button @click="pickerIsAm = true" :style="{ backgroundColor: pickerIsAm ? '#8f3235' : '#eee', color: pickerIsAm ? 'white' : '#333' }">AM</button>
              <button @click="pickerIsAm = false" :style="{ backgroundColor: !pickerIsAm ? '#8f3235' : '#eee', color: !pickerIsAm ? 'white' : '#333' }">PM</button>
           </div>
         </div>
      </div>
      <div class="cal-footer">
         <button class="footer-btn" @click="selectNow">Now</button>
         <button class="footer-btn done-btn" @click="confirmPicker">Done</button>
         <button class="footer-btn" @click="closePicker">Cancel</button>
      </div>
    </div>
  </div>

  <!-- Validation Record subpanel (outside the record box, like other modules) -->
  <div v-if="showValidationRecord" class="dim-panel-wrapper">
    <div class="dim-panel-header" @click="toggleValRecordPanel">
      <span class="dim-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!showValRecordPanel" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 11px; text-transform: uppercase;">{{ validationRecordTitle }}</span>
    </div>
    <div v-if="showValRecordPanel" class="dim-panel-body">
      <div class="dim-panel-inner-box">
        <div v-if="isEditing" class="sub-actions" style="padding: 5px 15px; border-bottom: 1px solid #ddd;">
          <span class="text-link" @click="isComparatorMicrometer ? addValidationCard() : addPinRow()">{{ isComparatorMicrometer ? 'Add New Validation' : 'Add New Row' }}</span>
        </div>

        <!-- Comparator / Micrometer: one card per validation, each with a 3-column gauge table -->
        <template v-if="isComparatorMicrometer">
          <div v-if="localRecord.validationCards.length === 0" class="val-empty">No validations yet.</div>
          <div v-for="(v, vi) in localRecord.validationCards" :key="vi" class="val-card">
            <div class="val-card-head">
              <span class="val-card-title">Validation {{ vi + 1 }}</span>
              <button v-if="isEditing" class="btn-remove-row" @click="removeValidationCard(vi)">✕ Remove</button>
            </div>
            <div class="val-card-shared">
              <div class="val-shared-field">
                <label>Date</label>
                <span v-if="!isEditing" class="field-value">{{ v.date || '-' }}</span>
                <input v-else type="text" v-model="v.date" class="val-cell-input" placeholder="e.g. 24 Mar 2026" />
              </div>
              <div class="val-shared-field">
                <label>Shift (D/N)</label>
                <span v-if="!isEditing" class="field-value">{{ v.shift || '-' }}</span>
                <select v-else v-model="v.shift" class="val-cell-input">
                  <option v-for="opt in shiftDNOptions" :key="opt" :value="opt">{{ opt || '--' }}</option>
                </select>
              </div>
            </div>
            <div class="table-scroll-container">
              <table class="data-table val-grid val-card-grid">
                <thead>
                  <tr>
                    <th class="val-card-rowlabel">Measurement</th>
                    <th v-for="(col, ci) in v.columns" :key="ci">Column {{ ci + 1 }}</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="r in cardGaugeRows" :key="r.key">
                    <td class="val-card-rowlabel">{{ r.label }}</td>
                    <td v-for="(col, ci) in v.columns" :key="ci">
                      <template v-if="r.readonly || !isEditing">{{ col[r.key] || '-' }}</template>
                      <template v-else>
                        <select v-if="optionsForType(r.type)" v-model="col[r.key]" class="val-cell-input">
                          <option v-for="opt in optionsForType(r.type)" :key="opt" :value="opt">{{ opt || '--' }}</option>
                        </select>
                        <input v-else type="text" v-model="col[r.key]" class="val-cell-input" />
                      </template>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
            <div class="val-card-shared">
              <div class="val-shared-field">
                <label>Checked by</label>
                <span v-if="!isEditing" class="field-value">{{ v.checkedBy || '-' }}</span>
                <input v-else type="text" v-model="v.checkedBy" class="val-cell-input" />
              </div>
              <div class="val-shared-field">
                <label>Verified by</label>
                <span v-if="!isEditing" class="field-value">{{ v.verifiedBy || '-' }}</span>
                <input v-else type="text" v-model="v.verifiedBy" class="val-cell-input" />
              </div>
              <div class="val-shared-field val-shared-wide">
                <label>Remark</label>
                <span v-if="!isEditing" class="field-value">{{ v.remark || '-' }}</span>
                <input v-else type="text" v-model="v.remark" class="val-cell-input" />
              </div>
            </div>
          </div>
        </template>

        <!-- Pin Gauge: flat add-row table (image 4) -->
        <template v-else>
          <div class="table-scroll-container">
            <table class="data-table val-grid">
              <thead>
                <tr>
                  <th v-for="col in pinGaugeCols" :key="col.key">{{ col.label }}</th>
                  <th v-if="isEditing" class="val-col-action">Remove</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, ri) in localRecord.pinGaugeRows" :key="ri">
                  <td v-for="col in pinGaugeCols" :key="col.key">
                    <template v-if="!isEditing">{{ row[col.key] || '-' }}</template>
                    <template v-else>
                      <select v-if="optionsForType(col.type)" v-model="row[col.key]" class="val-cell-input">
                        <option v-for="opt in optionsForType(col.type)" :key="opt" :value="opt">{{ opt || '--' }}</option>
                      </select>
                      <input v-else type="text" v-model="row[col.key]" class="val-cell-input" />
                    </template>
                  </td>
                  <td v-if="isEditing" class="val-col-action">
                    <button class="btn-remove-row" @click="removePinRow(ri)">✕</button>
                  </td>
                </tr>
                <tr v-if="localRecord.pinGaugeRows.length === 0">
                  <td :colspan="pinGaugeCols.length + (isEditing ? 1 : 0)" class="val-empty">No records yet.</td>
                </tr>
              </tbody>
            </table>
          </div>
        </template>
      </div>
    </div>
  </div>

  <!-- Induction Check subpanel (all categories) -->
  <div v-if="showInductionCheck" class="dim-panel-wrapper">
    <div class="dim-panel-header" @click="toggleInductionPanel">
      <span class="dim-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!showInductionPanel" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 11px; text-transform: uppercase;">Induction Check</span>
    </div>
    <div v-if="showInductionPanel" class="dim-panel-body">
      <div class="dim-panel-inner-box">
        <div v-if="isEditing" class="sub-actions" style="padding: 5px 15px; border-bottom: 1px solid #ddd;">
          <span class="text-link" @click="addInductionRow">Add New Row</span>
        </div>
        <div class="table-scroll-container">
          <table class="data-table val-grid">
            <thead>
              <tr>
                <th v-for="col in inductionCols" :key="col.key">{{ col.label }}</th>
                <th v-if="isEditing" class="val-col-action">Remove</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, ri) in localRecord.inductionRows" :key="ri">
                <td v-for="col in inductionCols" :key="col.key">
                  <template v-if="!isEditing">{{ row[col.key] || '-' }}</template>
                  <template v-else>
                    <select v-if="optionsForType(col.type)" v-model="row[col.key]" class="val-cell-input">
                      <option v-for="opt in optionsForType(col.type)" :key="opt" :value="opt">{{ opt || '--' }}</option>
                    </select>
                    <input v-else type="text" v-model="row[col.key]" class="val-cell-input" />
                  </template>
                </td>
                <td v-if="isEditing" class="val-col-action">
                  <button class="btn-remove-row" @click="removeInductionRow(ri)">✕</button>
                </td>
              </tr>
              <tr v-if="localRecord.inductionRows.length === 0">
                <td :colspan="inductionCols.length + (isEditing ? 1 : 0)" class="val-empty">No records yet.</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="pagination-bar">
          <span class="display-text">Displaying 1 to {{ localRecord.inductionRows.length }} of {{ localRecord.inductionRows.length }} items</span>
        </div>
      </div>
    </div>
  </div>

  <!-- Shown when no equipment is linked yet -->
  <div v-if="!showInductionCheck" class="dim-panel-wrapper">
    <div class="val-no-link-note">Link an Equipment by entering a matching <b>Equipment Serial No.</b> above to load the relevant validation subpanels.</div>
  </div>
  </div>
</template>

<style scoped>
.custom-calendar-popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 280px;
  background: #f1f1f1;
  border: 1px solid #ccc;
  border-top: 4px solid #8f3235;
  box-shadow: 0 10px 25px rgba(0,0,0,0.2);
  z-index: 10001;
  font-family: Arial, sans-serif;
  border-radius: 4px;
}

.cal-header {
  background: #8f3235;
  padding: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.cal-select {
  background: white;
  border: none;
  font-size: 13px;
  padding: 2px 4px;
  border-radius: 2px;
}

.cal-nav-btn {
  cursor: pointer;
  display: flex;
  align-items: center;
}

.cal-body {
  padding: 10px;
}

.cal-weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  font-weight: bold;
  font-size: 13px;
  text-align: center;
  margin-bottom: 5px;
  color: #333;
}

.cal-days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  text-align: center;
}

.cal-day {
  padding: 5px 0;
  cursor: pointer;
  font-size: 13px;
}

.cal-day:hover {
  background: #ddd;
}

.cal-today {
  background: #fff9c4 !important;
  border: 1px solid #ffd600;
  font-weight: bold;
}

.cal-time-section {
  border-top: 1px solid #ddd;
  margin-top: 10px;
  padding-top: 10px;
}

.time-display {
  font-size: 14px;
  margin-bottom: 10px;
  text-align: center;
  color: #333;
}

.slider-group {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 5px;
}

.slider-group label {
  width: 50px;
  font-size: 11px;
}

.cal-slider {
  flex: 1;
  height: 4px;
  accent-color: #8f3235;
}

.cal-footer {
  padding: 8px;
  background: #eee;
  border-top: 1px solid #ddd;
  display: flex;
  justify-content: space-between;
}

.footer-btn {
  border: 1px solid #ccc;
  background: white;
  padding: 3px 10px;
  font-size: 12px;
  cursor: pointer;
}

.done-btn {
  font-weight: bold;
  background: #eee;
}

.date-input-text {
  cursor: pointer;
  background-color: #fff !important;
}


.top-record-box {
  background-color: #fff;
  border: 2px solid #c7c7c7;
  margin: 15px 15px 15px 15px;
  overflow: hidden;
}
.modal-layout {
  border: none !important;
  margin: 0 !important;
  box-shadow: none !important;
}
.sub-panel-wrapper {
  margin: 0 15px 15px 15px;
}
.top-actions {
  display: flex;
  gap: 10px;
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
.btn-secondary { background-color: #a5a5a5; color: white; }

/* Equipment-linked (auto-filled, read-only) fields */
.linked-value { color: #333; }
.link-hint {
  font-size: 11px;
  margin-top: 3px;
  color: #b00;
}
.link-hint.matched { color: #2e7d32; }
.val-thumb {
  display: inline-block;
  width: 34px;
  height: 34px;
  border: 1px solid #ccc;
  overflow: hidden;
  background: #fff;
}
.val-thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Subpanels outside the record box (matches other modules) */
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
.dim-panel-header:hover { background-color: #c9c9c9; }
.dim-panel-icon-btn { display: flex; align-items: center; margin-right: -4px; }
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
.sub-actions { padding: 8px 15px; background-color: #fff; }
.text-link { font-size: 13px; color: #8f3235; cursor: pointer; font-weight: bold; }
.text-link:hover { text-decoration: underline; }
.table-scroll-container { overflow-x: auto; border-top: 1px solid #eee; }
.data-table { width: 100%; border-collapse: collapse; }
.val-grid th, .val-grid td {
  border: 1px solid #e0e0e0;
  padding: 4px 6px;
  font-size: 11px;
  white-space: nowrap;
}
.val-grid th { background: #f2f2f2; font-weight: bold; border-bottom: 2px solid #ddd; }
.val-cell-input {
  width: 100%;
  min-width: 90px;
  padding: 2px 4px;
  border: 1px solid #ccc;
  font-size: 11px;
  box-sizing: border-box;
}
.val-col-action { text-align: center; width: 60px; }
.btn-remove-row {
  background: #fff;
  border: 1px solid #ccc;
  color: #a51c22;
  cursor: pointer;
  border-radius: 2px;
  padding: 1px 7px;
  font-size: 12px;
}
.btn-remove-row:hover { background: #f9f9f9; }
.val-empty { text-align: center; color: #666; padding: 14px; }

/* Validation card (Comparator / Micrometer) */
.val-card {
  border: 1px solid #ccc;
  margin: 10px 15px;
  background: #fff;
}
.val-card-head {
  background: #ececec;
  padding: 5px 10px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #ddd;
}
.val-card-title { font-weight: bold; font-size: 12px; color: #333; }
.val-card-shared {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  padding: 8px 10px;
}
.val-shared-field {
  display: flex;
  align-items: center;
  gap: 6px;
}
.val-shared-field label {
  font-size: 11px;
  font-weight: bold;
  color: #555;
  white-space: nowrap;
}
.val-shared-field .val-cell-input { min-width: 120px; }
.val-shared-wide { flex: 1; }
.val-shared-wide .val-cell-input { width: 100%; min-width: 200px; }
.val-card-grid { margin: 0; }
.val-card-rowlabel {
  background: #f2f2f2;
  font-weight: bold;
  text-align: left;
  white-space: nowrap;
  min-width: 170px;
}

.pagination-bar {
  background-color: #f7f7f7;
  border-top: 1px solid #ddd;
  padding: 4px 15px;
}
.display-text { font-size: 11px; color: #444; }
.val-no-link-note {
  color: #666;
  font-size: 12px;
  padding: 12px;
  border: 1px solid #E9E8E6;
  background-color: #f1f1f1;
}
</style>
