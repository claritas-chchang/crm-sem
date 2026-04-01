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
  }
})

const emit = defineEmits(['back', 'save'])

const isEditing = ref(props.isEditing || props.isCreating)
const localRecord = ref({ ...props.record })

// Dropdown Options
const frequencyOptions = ['--Please Select One--', 'Daily', 'Weekly', 'Monthly', 'Quarterly', 'Half-yearly', 'Annual']
const shiftOptions = ['--Please Select One--', 'Day', 'Night']
const typeOptions = ['--Please Select One--', 'Type 1: Comparator/Micrometer', 'Type 2: Induction Check', 'Type 3: Pin Gauge']
const okNgOptions = ['--Please Select One--', 'OK', 'NG']

watch(() => props.record, (newVal) => {
  localRecord.value = { ...newVal }
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

  if (showPicker.value === 'vDate' || showPicker.value === 'vDue') {
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
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.spec }}</div><input v-else type="text" v-model="localRecord.spec" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Equipment</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.eq }}</div><input v-else type="text" v-model="localRecord.eq" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Equipment Serial No.</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.eqSerial }}</div><input v-else type="text" v-model="localRecord.eqSerial" class="edit-select" /></td>
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
              <td class="labelBack"><span class="labelTitle">Type</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.type }}</div>
                <select v-else v-model="localRecord.type" class="edit-select">
                  <option v-for="opt in typeOptions" :key="opt" :value="opt">{{ opt }}</option>
                </select>
              </td>
              <td class="labelBack"><span class="labelTitle">Year</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.year }}</div><input v-else type="text" v-model="localRecord.year" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Month</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.month }}</div><input v-else type="text" v-model="localRecord.month" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Product</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.product }}</div><input v-else type="text" v-model="localRecord.product" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Serial No.</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.serialNo }}</div><input v-else type="text" v-model="localRecord.serialNo" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Verification Date</span></td>
              <td style="position: relative;">
                <div v-if="!isEditing" class="field-value">{{ formatDisplayDate(localRecord.vDate) }}</div>
                <input v-else type="text" readonly :value="localRecord.vDate ? formatDisplayDate(localRecord.vDate) : ''" @click="openPicker('vDate')" class="edit-select date-input-text" />
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Verification Due Date</span></td>
              <td colspan="3" style="position: relative;">
                <div v-if="!isEditing" class="field-value">{{ formatDisplayDate(localRecord.vDue) }}</div>
                <input v-else type="text" readonly :value="localRecord.vDue ? formatDisplayDate(localRecord.vDue) : ''" @click="openPicker('vDue')" class="edit-select date-input-text" style="max-width:34%;" />
              </td>
            </tr>
          </tbody>
        </table>
      </fieldset>

      <!-- Conditional Section - Type 1: Comparator/Micrometer -->
      <fieldset class="fsMargin" v-if="localRecord.type === 'Type 1: Comparator/Micrometer'" style="margin: 0 0 15px 0;">
        <legend><b>Type 1: Comparator/Micrometer Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Block Gauge S/N</span></td>
              <td style="width: 34%;"><div v-if="!isEditing" class="field-value">{{ localRecord.bgSerial || '-' }}</div><input v-else type="text" v-model="localRecord.bgSerial" class="edit-select" /></td>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Block Gauge Norminal</span></td>
              <td style="width: 34%;"><div v-if="!isEditing" class="field-value">{{ localRecord.bgNorminal || '-' }}</div><input v-else type="text" v-model="localRecord.bgNorminal" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Min Spec</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.minSpec || '-' }}</div><input v-else type="text" v-model="localRecord.minSpec" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Max Spec</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.maxSpec || '-' }}</div><input v-else type="text" v-model="localRecord.maxSpec" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Reading 1</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.read1 || '-' }}</div><input v-else type="text" v-model="localRecord.read1" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Reading 2</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.read2 || '-' }}</div><input v-else type="text" v-model="localRecord.read2" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Reading 3</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.read3 || '-' }}</div><input v-else type="text" v-model="localRecord.read3" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Judgement</span></td>
              <td>
                <div v-if="!isEditing" class="field-value">{{ localRecord.judgement || '-' }}</div>
                <select v-else v-model="localRecord.judgement" class="edit-select">
                  <option v-for="opt in okNgOptions" :key="opt" :value="opt">{{ opt }}</option>
                </select>
              </td>
            </tr>
          </tbody>
        </table>
      </fieldset>

      <!-- Conditional Section - Type 2: Induction Check -->
      <fieldset class="fsMargin" v-if="localRecord.type === 'Type 2: Induction Check'" style="margin: 0 0 15px 0;">
        <legend><b>Type 2: Induction Check Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Green Paper Check</span></td>
              <td style="width: 34%;">
                <div v-if="!isEditing" class="field-value">{{ localRecord.gpCheck || '-' }}</div>
                <select v-else v-model="localRecord.gpCheck" class="edit-select">
                   <option v-for="opt in okNgOptions" :key="opt" :value="opt">{{ opt }}</option>
                </select>
              </td>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Gauss Meter if GP NG</span></td>
              <td style="width: 34%;"><div v-if="!isEditing" class="field-value">{{ localRecord.gaussReading || '-' }}</div><input v-else type="text" v-model="localRecord.gaussReading" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">GP After De-mag</span></td>
              <td>
                 <div v-if="!isEditing" class="field-value">{{ localRecord.gpAfterDemag || '-' }}</div>
                 <select v-else v-model="localRecord.gpAfterDemag" class="edit-select">
                   <option v-for="opt in okNgOptions" :key="opt" :value="opt">{{ opt }}</option>
                 </select>
              </td>
              <td class="labelBack"><span class="labelTitle">De-magnetised By</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.demagBy || '-' }}</div><input v-else type="text" v-model="localRecord.demagBy" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Checked By</span></td>
              <td colspan="3"><div v-if="!isEditing" class="field-value">{{ localRecord.checkedBy || '-' }}</div><input v-else type="text" v-model="localRecord.checkedBy" class="edit-select" style="max-width:34%;" /></td>
            </tr>
          </tbody>
        </table>
      </fieldset>

      <!-- Conditional Section - Type 3: Pin Gauge -->
      <fieldset class="fsMargin" v-if="localRecord.type === 'Type 3: Pin Gauge'" style="margin: 0 0 15px 0;">
        <legend><b>Type 3: Pin Gauge Details</b></legend>
        <table border="0" style="width: 100%; table-layout: fixed;">
          <tbody>
            <tr>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">Permission Error/Tolerance</span></td>
              <td style="width: 34%;"><div v-if="!isEditing" class="field-value">{{ localRecord.permError || '-' }}</div><input v-else type="text" v-model="localRecord.permError" class="edit-select" /></td>
              <td class="labelBack" style="width: 16%;"><span class="labelTitle">GP</span></td>
              <td style="width: 34%;"><div v-if="!isEditing" class="field-value">{{ localRecord.gpVal || '-0.002mm' }}</div><input v-else type="text" v-model="localRecord.gpVal" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">NP</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.npVal || '0.002mm' }}</div><input v-else type="text" v-model="localRecord.npVal" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Micrometer Zero Check Result</span></td>
              <td>
                 <div v-if="!isEditing" class="field-value">{{ localRecord.microZero || '-' }}</div>
                 <select v-else v-model="localRecord.microZero" class="edit-select">
                   <option v-for="opt in okNgOptions" :key="opt" :value="opt">{{ opt }}</option>
                 </select>
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">GP Norminal Value</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.gpNorminal || '-' }}</div><input v-else type="text" v-model="localRecord.gpNorminal" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">GP Front</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.gpFront || '-' }}</div><input v-else type="text" v-model="localRecord.gpFront" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">   Reading 1</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.gpFR1 || '-' }}</div><input v-else type="text" v-model="localRecord.gpFR1" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">   Reading 2</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.gpFR2 || '-' }}</div><input v-else type="text" v-model="localRecord.gpFR2" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">GP End</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.gpEnd || '-' }}</div><input v-else type="text" v-model="localRecord.gpEnd" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">   Reading 1</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.gpER1 || '-' }}</div><input v-else type="text" v-model="localRecord.gpER1" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">   Reading 2</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.gpER2 || '-' }}</div><input v-else type="text" v-model="localRecord.gpER2" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">Result</span></td>
              <td>
                 <div v-if="!isEditing" class="field-value">{{ localRecord.gpResult || '-' }}</div>
                 <select v-else v-model="localRecord.gpResult" class="edit-select">
                   <option v-for="opt in okNgOptions" :key="opt" :value="opt">{{ opt }}</option>
                 </select>
              </td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">NP Norminal Value</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.npNorminal || '-' }}</div><input v-else type="text" v-model="localRecord.npNorminal" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">NP Front</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.npFront || '-' }}</div><input v-else type="text" v-model="localRecord.npFront" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">   Reading 1</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.npFR1 || '-' }}</div><input v-else type="text" v-model="localRecord.npFR1" class="edit-select" /></td>
              <td class="labelBack"><span class="labelTitle">   Reading 2</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.npFR2 || '-' }}</div><input v-else type="text" v-model="localRecord.npFR2" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Result</span></td>
              <td>
                 <div v-if="!isEditing" class="field-value">{{ localRecord.npResult || '-' }}</div>
                 <select v-else v-model="localRecord.npResult" class="edit-select">
                   <option v-for="opt in okNgOptions" :key="opt" :value="opt">{{ opt }}</option>
                 </select>
              </td>
              <td class="labelBack"><span class="labelTitle">Checked By</span></td>
              <td><div v-if="!isEditing" class="field-value">{{ localRecord.checkedBy || '-' }}</div><input v-else type="text" v-model="localRecord.checkedBy" class="edit-select" /></td>
            </tr>
            <tr>
              <td class="labelBack"><span class="labelTitle">Verified By</span></td>
              <td colspan="3"><div v-if="!isEditing" class="field-value">{{ localRecord.verifiedBy || '-' }}</div><input v-else type="text" v-model="localRecord.verifiedBy" class="edit-select" style="max-width:34%;" /></td>
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
         
         <div class="cal-time-section" v-if="showPicker !== 'vDate' && showPicker !== 'vDue'">
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
</style>
