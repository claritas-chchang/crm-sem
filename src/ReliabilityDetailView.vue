<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'

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

const qaRecords = ref([
  { platingDate: '2026-03-27', line: 'L1', model: 'M-101', lotNo: 'LOT001', loadingDateTime: '2026-03-27 08:00', result: 'OK', judgement: 'Passes', remark: '-', doneBy: 'QA-01', verifiedBy: 'QA-02', confirmedBy: 'MGR-01' }
])

const isQAListOpen = ref(true)

const toggleQAList = () => {
  isQAListOpen.value = !isQAListOpen.value
}

const showAddModal = ref(false)

const initialNewRecord = {
  platingDate: '',
  line: '',
  model: '',
  lotNo: '',
  loadingDateTime: '',
  result: 'OK',
  judgement: 'Passes',
  remark: '',
  doneBy: ''
}

const newRecord = ref({ ...initialNewRecord })

const openAddModal = () => {
  newRecord.value = { ...initialNewRecord }
  showAddModal.value = true
}

const addTestRecord = () => {
  let loadingDateTime = '-'
  if (newRecord.value.loadingDateTime) {
    loadingDateTime = newRecord.value.loadingDateTime.replace('T', ' ')
  }
  
  const formattedRec = {
    platingDate: newRecord.value.platingDate || '-',
    line: newRecord.value.line || '-',
    model: newRecord.value.model || '-',
    lotNo: newRecord.value.lotNo || '-',
    loadingDateTime: loadingDateTime,
    result: newRecord.value.result,
    judgement: newRecord.value.judgement,
    remark: newRecord.value.remark || '-',
    doneBy: newRecord.value.doneBy || '-',
    verifiedBy: '-',
    confirmedBy: '-'
  }
  
  qaRecords.value.push(formattedRec)
  showAddModal.value = false
}

// Inline edit state and methods
const editingCell = ref({ rowIdx: -1, field: '' })
const originalValue = ref('')

const startInlineEdit = (rowIdx, field) => {
  editingCell.value = { rowIdx, field }
  originalValue.value = qaRecords.value[rowIdx][field]
}

const saveInlineEdit = () => {
  editingCell.value = { rowIdx: -1, field: '' }
}

const isEditingSubRecord = ref(false)
const subRecordToEditIdx = ref(-1)

const openEditModal = (idx) => {
  subRecordToEditIdx.value = idx
  newRecord.value = { ...qaRecords.value[idx] }
  // Reverse format loadingDateTime for datetime-local input
  if (newRecord.value.loadingDateTime && newRecord.value.loadingDateTime !== '-') {
    newRecord.value.loadingDateTime = newRecord.value.loadingDateTime.replace(' ', 'T')
  }
  isEditingSubRecord.value = true
  showAddModal.value = true
}

const updateTestRecord = () => {
  let loadingDateTime = '-'
  if (newRecord.value.loadingDateTime) {
    loadingDateTime = newRecord.value.loadingDateTime.replace('T', ' ')
  }
  
  const formattedRec = {
    ...newRecord.value,
    loadingDateTime: loadingDateTime
  }
  
  qaRecords.value[subRecordToEditIdx.value] = formattedRec
  showAddModal.value = false
  isEditingSubRecord.value = false
}

const cancelInlineEdit = () => {
  if (editingCell.value.rowIdx !== -1) {
    qaRecords.value[editingCell.value.rowIdx][editingCell.value.field] = originalValue.value
    editingCell.value = { rowIdx: -1, field: '' }
  }
}

const handleGlobalClick = (e) => {
  if (!e.target.closest('.inline-edit') && !e.target.closest('.editable-text')) {
    if (editingCell.value.rowIdx !== -1) cancelInlineEdit()
    if (inlinePullEditing.value.rowIdx !== -1) cancelPullInlineEdit()
    if (inlineQuenchEditing.value.rowIdx !== -1) cancelQuenchInlineEdit()
    if (inlineThermalEditing.value.rowIdx !== -1) cancelThermalInlineEdit()
    if (inlineRoutineEditing.value.rowIdx !== -1) cancelRoutineInlineEdit()
  }
}

onMounted(() => {
  window.addEventListener('click', handleGlobalClick)
})

onUnmounted(() => {
  window.removeEventListener('click', handleGlobalClick)
})

// --- VCM PULL TEST LIST ---
const pullRecords = ref([
  { platingLine: 'A3', model: 'WDA371', lotNo: 'LOT002', jig: 'Convex', bondingArea: '1.0000', sample: 'S-01', adhesionForce: '100', condition: 'No Peel Off', result: 'OK', substrateColor: 'A', remark: '-', doneBy: 'QA-01', confirmedBy: 'MGR-01' }
])

const isPullListOpen = ref(true)

const togglePullList = () => {
  isPullListOpen.value = !isPullListOpen.value
}

const showAddPullModal = ref(false)

const platingLineOptions = ['A3', 'B1', 'B2', 'B3', 'C1', 'C2', 'D1', 'D2']
const modelOptions = ['WDA371', 'HSA329', 'HSA339', 'HSA346', 'HSA349', 'HSA353', 'HSA355 TOP', 'HSA355 BTM', 'HSA357', 'HSA361 TOP', 'HSA361 BTM', 'HSA363 TOP', 'HSA363 BTM', 'HSA364 TOP', 'HSA364 BTM', 'HSA371 TOP', 'HSA371 BTM', 'STA247', 'STA250', 'STA251', 'STA253', 'STA256', 'STA257', 'STA358', 'STA371 TOP', 'STA371 BTM', 'STA374', 'STA376', 'TOA307', 'TOA311', 'TOA317']
const conditionOptions = ['Peel Off', 'Swelling', 'No Peel Off']
const substrateColorOptions = ['A', 'B', 'C', 'D']

const modelDataMap = {
  'WDA371': { jig: 'Convex', area: '1.0000' },
  'HSA329': { jig: 'Flat', area: '2.6556' },
  'HSA339': { jig: 'Convex', area: '1.0000' },
  'HSA346': { jig: 'Convex', area: '1.0000' },
  'HSA349': { jig: 'Convex', area: '1.0000' },
  'HSA353': { jig: 'Convex', area: '1.0000' },
  'HSA355 TOP': { jig: 'Convex', area: '1.0000' },
  'HSA355 BTM': { jig: 'Convex', area: '1.0000' },
  'HSA357': { jig: 'Convex', area: '1.0000' },
  'HSA361 TOP': { jig: 'Convex', area: '1.0000' },
  'HSA361 BTM': { jig: 'Convex', area: '1.0000' },
  'HSA363 TOP': { jig: 'Convex', area: '1.0000' },
  'HSA363 BTM': { jig: 'Convex', area: '1.0000' },
  'HSA364 TOP': { jig: 'Convex', area: '1.0000' },
  'HSA364 BTM': { jig: 'Convex', area: '1.0000' },
  'HSA371 TOP': { jig: 'Convex', area: '1.0000' },
  'HSA371 BTM': { jig: 'Convex', area: '1.0000' },
  'STA247': { jig: 'Flat', area: '3.0695' },
  'STA250': { jig: 'Flat', area: '2.7999' },
  'STA251': { jig: 'Flat', area: '2.8159' },
  'STA253': { jig: 'Flat', area: '1.8786' },
  'STA256': { jig: 'Flat', area: '1.8983' },
  'STA257': { jig: 'Flat', area: '3.01722' }
}

const initialNewPullRecord = {
  platingLine: platingLineOptions[0],
  model: modelOptions[0],
  lotNo: '',
  jig: 'Convex',
  bondingArea: '1.0000',
  sample: '',
  adhesionForce: '',
  condition: conditionOptions[0],
  result: 'OK',
  substrateColor: substrateColorOptions[0],
  remark: '',
  doneBy: '',
  confirmedBy: ''
}

const newPullRecord = ref({ ...initialNewPullRecord })

const handlePullModelChange = () => {
  const data = modelDataMap[newPullRecord.value.model]
  if (data) {
    newPullRecord.value.jig = data.jig || ''
    newPullRecord.value.bondingArea = data.area || ''
  }
}

const openAddPullModal = () => {
  newPullRecord.value = { ...initialNewPullRecord }
  showAddPullModal.value = true
}

const openEditPullModal = (idx) => {
  subRecordToEditIdx.value = idx
  newPullRecord.value = { ...pullRecords.value[idx] }
  isEditingSubRecord.value = true
  showAddPullModal.value = true
}

const updatePullRecord = () => {
  pullRecords.value[subRecordToEditIdx.value] = { ...newPullRecord.value }
  showAddPullModal.value = false
  isEditingSubRecord.value = false
}

const addPullRecord = () => {
  pullRecords.value.push({ ...newPullRecord.value })
  showAddPullModal.value = false
}

const inlinePullEditing = ref({ rowIdx: -1, field: '' })
const pullOriginalValue = ref('')

const startPullInlineEdit = (rowIdx, field) => {
  inlinePullEditing.value = { rowIdx, field }
  pullOriginalValue.value = pullRecords.value[rowIdx][field]
}

const savePullInlineEdit = () => {
  inlinePullEditing.value = { rowIdx: -1, field: '' }
}

const cancelPullInlineEdit = () => {
  if (inlinePullEditing.value.rowIdx !== -1) {
    pullRecords.value[inlinePullEditing.value.rowIdx][inlinePullEditing.value.field] = pullOriginalValue.value
    inlinePullEditing.value = { rowIdx: -1, field: '' }
  }
}

// --- QUENCH TEST LIST ---
const quenchRecords = ref([
  { month: 'March', year: '2026', model: 'WDA371', date: '2026-03-28', lotNo: 'LOT003', line: 'L1', testingDateTime: '2026-03-28 10:00', result: 'OK', judgement: 'Passes', remark: '-', doneBy: 'QA-01', verifiedBy: 'QA-02', confirmedBy: 'MGR-01' }
])

const isQuenchListOpen = ref(true)

const toggleQuenchList = () => {
  isQuenchListOpen.value = !isQuenchListOpen.value
}

const showAddQuenchModal = ref(false)

const initialNewQuenchRecord = {
  month: '',
  year: '',
  model: '',
  date: '',
  lotNo: '',
  line: '',
  testingDateTime: '',
  result: 'OK',
  judgement: 'Passes',
  remark: '',
  doneBy: '',
  verifiedBy: '',
  confirmedBy: ''
}

const newQuenchRecord = ref({ ...initialNewQuenchRecord })

const openAddQuenchModal = () => {
  newQuenchRecord.value = { ...initialNewQuenchRecord }
  showAddQuenchModal.value = true
}

const openEditQuenchModal = (idx) => {
  subRecordToEditIdx.value = idx
  newQuenchRecord.value = { ...quenchRecords.value[idx] }
  if (newQuenchRecord.value.testingDateTime && newQuenchRecord.value.testingDateTime !== '-') {
    newQuenchRecord.value.testingDateTime = newQuenchRecord.value.testingDateTime.replace(' ', 'T')
  }
  isEditingSubRecord.value = true
  showAddQuenchModal.value = true
}

const updateQuenchRecord = () => {
  const formattedRec = { ...newQuenchRecord.value }
  if (formattedRec.testingDateTime) {
    formattedRec.testingDateTime = formattedRec.testingDateTime.replace('T', ' ')
  }
  quenchRecords.value[subRecordToEditIdx.value] = formattedRec
  showAddQuenchModal.value = false
  isEditingSubRecord.value = false
}

const addQuenchRecord = () => {
  const formattedRec = { ...newQuenchRecord.value }
  if (formattedRec.testingDateTime) {
    formattedRec.testingDateTime = formattedRec.testingDateTime.replace('T', ' ')
  }
  quenchRecords.value.push(formattedRec)
  showAddQuenchModal.value = false
}

const inlineQuenchEditing = ref({ rowIdx: -1, field: '' })
const quenchOriginalValue = ref('')

const startQuenchInlineEdit = (rowIdx, field) => {
  inlineQuenchEditing.value = { rowIdx, field }
  quenchOriginalValue.value = quenchRecords.value[rowIdx][field]
}

const saveQuenchInlineEdit = () => {
  inlineQuenchEditing.value = { rowIdx: -1, field: '' }
}

const cancelQuenchInlineEdit = () => {
  if (inlineQuenchEditing.value.rowIdx !== -1) {
    quenchRecords.value[inlineQuenchEditing.value.rowIdx][inlineQuenchEditing.value.field] = quenchOriginalValue.value
    inlineQuenchEditing.value = { rowIdx: -1, field: '' }
  }
}

// --- THERMAL DEMAGNETISATION TEST LIST ---
const thermalRecords = ref([
  { month: 'Oct', year: '2026', platingLine: 'A4', platingDate: '2026-10-01', time: '10:00AM', model: '', lotNo: 'LINE NOT RUNNING', paNumber: '', formingNo: '', result: '', judgement: '', oven: '', remark: '', 
    bf: ['', '', '', '', '', '', '', '', '', ''],
    af: ['', '', '', '', '', '', '', '', '', ''],
    diff: ['0.00', '0.00', '0.00', '0.00', '0.00', '0.00', '0.00', '0.00', '0.00', '0.00'],
    act: ['0', '0', '0', '0', '0', '0', '0', '0', '0', '0'],
    sfBefore: '', sfAfter: '', sfFactor: '#DIV/0!',
    gsStandard: '', gsMeasured: '', gsFactor: '0.000'
  },
  { month: 'Oct', year: '2026', platingLine: 'A4', platingDate: '2026-10-01', time: '6:00PM', model: 'OLP 0785N', lotNo: '2N3 25081201-001 (19/24)', paNumber: '2852B', formingNo: '0048', result: 'OK', judgement: 'Passes', oven: '990155', remark: '', 
    bf: ['584', '587', '583', '587', '579', '580', '578', '583', '582', '584'],
    af: ['583', '586', '582', '586', '578', '579', '577', '582', '581', '583'],
    diff: ['0.17', '0.17', '0.17', '0.17', '0.17', '0.17', '0.17', '0.17', '0.17', '0.17'],
    act: ['0', '0', '0', '0', '0', '0', '0', '0', '0', '0'],
    sfBefore: '581', sfAfter: '581', sfFactor: '1.000',
    gsStandard: '', gsMeasured: '', gsFactor: '0.000'
  },
  { month: 'Oct', year: '2026', platingLine: 'A4', platingDate: '2026-10-01', time: '1:00AM', model: '', lotNo: '-', paNumber: '', formingNo: '', result: '', judgement: '', oven: '', remark: '', 
    bf: ['', '', '', '', '', '', '', '', '', ''],
    af: ['', '', '', '', '', '', '', '', '', ''],
    diff: ['0.00', '0.00', '0.00', '0.00', '0.00', '0.00', '0.00', '0.00', '0.00', '0.00'],
    act: ['0', '0', '0', '0', '0', '0', '0', '0', '0', '0'],
    sfBefore: '', sfAfter: '', sfFactor: '#DIV/0!',
    gsStandard: '', gsMeasured: '', gsFactor: '0.000'
  },
  { month: 'Oct', year: '2026', platingLine: 'A4', platingDate: '2026-10-03', time: '6:00PM', model: 'ACT 0836N', lotNo: '2SB 25082616-019 (6/6)', paNumber: '2946A', formingNo: '0342', result: 'OK', judgement: 'Passes', oven: '990155', remark: '', 
    bf: ['532', '535', '533', '536', '532', '534', '533', '536', '533', '532'],
    af: ['531', '534', '532', '535', '531', '533', '532', '535', '532', '531'],
    diff: ['0.19', '0.19', '0.19', '0.19', '0.19', '0.19', '0.19', '0.19', '0.19', '0.19'],
    act: ['252', '253', '252', '254', '252', '253', '252', '254', '252', '252'],
    sfBefore: '535', sfAfter: '535', sfFactor: '1.000',
    gsStandard: '248', gsMeasured: '523', gsFactor: '0.474'
  }
])

const isThermalListOpen = ref(true)

const toggleThermalList = () => {
  isThermalListOpen.value = !isThermalListOpen.value
}

const showAddThermalModal = ref(false)

const thermalPlatingLineOptions = ['A4', 'B1', 'B2', 'C1', 'C2', 'D1', 'D2 (Ni-Ni)', 'D2 (Cu-Ni)']
const thermalTimeOptions = ['10:00AM', '6:00PM', '1:00AM']

const initialNewThermalRecord = {
  month: '', year: '', platingLine: thermalPlatingLineOptions[0], platingDate: '', time: thermalTimeOptions[0], model: '', lotNo: '', paNumber: '', formingNo: '', 
  bf: ['', '', '', '', '', '', '', '', '', ''],
  af: ['', '', '', '', '', '', '', '', '', ''],
  diff: ['', '', '', '', '', '', '', '', '', ''],
  act: ['', '', '', '', '', '', '', '', '', ''],
  sfBefore: '', sfAfter: '', sfFactor: '',
  gsStandard: '', gsMeasured: '', gsFactor: '',
  result: 'OK', judgement: 'Passes', oven: '', remark: ''
}

const newThermalRecord = ref({ ...initialNewThermalRecord })

const openAddThermalModal = () => {
  newThermalRecord.value = JSON.parse(JSON.stringify(initialNewThermalRecord))
  showAddThermalModal.value = true
}

const openEditThermalModal = (idx) => {
  subRecordToEditIdx.value = idx
  newThermalRecord.value = JSON.parse(JSON.stringify(thermalRecords.value[idx]))
  isEditingSubRecord.value = true
  showAddThermalModal.value = true
}

const updateThermalRecord = () => {
  thermalRecords.value[subRecordToEditIdx.value] = JSON.parse(JSON.stringify(newThermalRecord.value))
  showAddThermalModal.value = false
  isEditingSubRecord.value = false
}

const addThermalRecord = () => {
  thermalRecords.value.push(JSON.parse(JSON.stringify(newThermalRecord.value)))
  showAddThermalModal.value = false
}

const inlineThermalEditing = ref({ rowIdx: -1, field: '' })
const thermalOriginalValue = ref('')

const startThermalInlineEdit = (rowIdx, field) => {
  inlineThermalEditing.value = { rowIdx, field }
  thermalOriginalValue.value = thermalRecords.value[rowIdx][field]
}

const saveThermalInlineEdit = () => {
  inlineThermalEditing.value = { rowIdx: -1, field: '' }
}

const cancelThermalInlineEdit = () => {
  if (inlineThermalEditing.value.rowIdx !== -1) {
    thermalRecords.value[inlineThermalEditing.value.rowIdx][inlineThermalEditing.value.field] = thermalOriginalValue.value
    inlineThermalEditing.value = { rowIdx: -1, field: '' }
  }
}

// --- ROUTINE RELIABILITY TEST LIST ---
const routineRecords = ref([
  { 
    month: 'March', year: '2026', platingLine: 'A4', platingDate: '2026-03-31', time: '10:00AM', model: 'WDA371', lotNo: 'LOT100', paNumber: 'PA100', formingNo: 'F100',
    cookerTest: 'OK', colorSub: 'A', resultColorSub: 'OK', 
    shear1: '1.2', shear2: '1.3', shear3: '1.4', shear4: '1.5', shear5: '1.6', resultShear: '6.0',
    pull1: '0.8', pull2: '0.9', pull3: '0.9', pull4: '0.8', pull5: '1.0', resultPull: '4.4',
    oven: 'OVEN-001', tdtOk: '25', tdtNg: '0', remark: '-'
  }
])

const isRoutineListOpen = ref(true)
const toggleRoutineList = () => { isRoutineListOpen.value = !isRoutineListOpen.value }

const showAddRoutineModal = ref(false)
const routinePlatingLineOptions = ['A4', 'B1', 'B2', 'C1', 'C2', 'D1', 'D2 (Ni-Ni)', 'D2 (Cu-Ni)']
const routineTimeOptions = ['10:00AM', '6:00PM', '1:00AM']
const cookerOptions = ['OK', 'NG', 'Peding For Result']
const colorSubOptions = ['A', 'B', 'C', 'D']
const okNgOptions = ['OK', 'NG']

const initialNewRoutineRecord = {
  month: '', year: '', platingLine: routinePlatingLineOptions[0], platingDate: '', time: routineTimeOptions[0], model: '', lotNo: '', paNumber: '', formingNo: '',
  cookerTest: cookerOptions[0], colorSub: colorSubOptions[0], resultColorSub: okNgOptions[0],
  shear1: '', shear2: '', shear3: '', shear4: '', shear5: '', resultShear: '',
  pull1: '', pull2: '', pull3: '', pull4: '', pull5: '', resultPull: '',
  oven: '', tdtOk: '', tdtNg: '', remark: ''
}

const newRoutineRecord = ref({ ...initialNewRoutineRecord })

const openAddRoutineModal = () => {
  newRoutineRecord.value = JSON.parse(JSON.stringify(initialNewRoutineRecord))
  showAddRoutineModal.value = true
}

const openEditRoutineModal = (idx) => {
  subRecordToEditIdx.value = idx
  newRoutineRecord.value = JSON.parse(JSON.stringify(routineRecords.value[idx]))
  isEditingSubRecord.value = true
  showAddRoutineModal.value = true
}

const updateRoutineRecord = () => {
  routineRecords.value[subRecordToEditIdx.value] = JSON.parse(JSON.stringify(newRoutineRecord.value))
  showAddRoutineModal.value = false
  isEditingSubRecord.value = false
}

const addRoutineRecord = () => {
  routineRecords.value.push(JSON.parse(JSON.stringify(newRoutineRecord.value)))
  showAddRoutineModal.value = false
}

const inlineRoutineEditing = ref({ rowIdx: -1, field: '' })
const routineOriginalValue = ref('')

const startRoutineInlineEdit = (rowIdx, field) => {
  inlineRoutineEditing.value = { rowIdx, field }
  routineOriginalValue.value = routineRecords.value[rowIdx][field]
}

const saveRoutineInlineEdit = () => {
  inlineRoutineEditing.value = { rowIdx: -1, field: '' }
}

const cancelRoutineInlineEdit = () => {
  if (inlineRoutineEditing.value.rowIdx !== -1) {
    routineRecords.value[inlineRoutineEditing.value.rowIdx][inlineRoutineEditing.value.field] = routineOriginalValue.value
    inlineRoutineEditing.value = { rowIdx: -1, field: '' }
  }
}
</script>

<template>
  <div class="view-panel">
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

  <!-- QA Pressure Cooker List Sub-Panel -->
  <div v-if="!isCreating && localRecord.testType === 'Pressure Cooker'" class="sub-panel-wrapper">
    <div class="sub-panel-header" @click="toggleQAList">
      <span class="sub-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!isQAListOpen" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 11px; text-transform: uppercase;">QA Pressure Cooker List</span>
    </div>

    <div v-if="isQAListOpen" class="sub-panel-body">
      <div class="sub-panel-inner-box">
        <div class="sub-actions" style="padding: 5px 15px; border-bottom: 1px solid #ddd;">
          <span class="text-link" @click="openAddModal">Add New Test Record</span>
        </div>
        <div class="table-scroll-container">
          <table class="data-table dim-table">
            <thead>
              <tr>
                <th class="col-icon"></th>
                <th>Plating Date</th>
                <th>Line</th>
                <th>Model</th>
                <th>Lot No</th>
                <th>Loading Date Time</th>
                <th>Result</th>
                <th>Judgement</th>
                <th>Remark</th>
                <th>Done By (ID Number)</th>
                <th>Verified By (ID Number)</th>
                <th>Confirmed By (ID Number)</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(rec, idx) in qaRecords" :key="idx">
                <td class="col-icon">
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;" @click="openEditModal(idx)"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                </td>
                <td>{{ rec.platingDate }}</td>
                <td>{{ rec.line }}</td>
                <td>{{ rec.model }}</td>
                <td>{{ rec.lotNo }}</td>
                <td>{{ rec.loadingDateTime }}</td>
                <td>
                  <div v-if="editingCell.rowIdx === idx && editingCell.field === 'result'" class="inline-edit">
                    <select v-model="rec.result" class="grid-select" @click.stop>
                      <option value="OK">OK</option>
                      <option value="NG">NG</option>
                    </select>
                    <button class="btn-save" @click.stop="saveInlineEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startInlineEdit(idx, 'result')">{{ rec.result }}</span>
                </td>
                <td>
                  <div v-if="editingCell.rowIdx === idx && editingCell.field === 'judgement'" class="inline-edit">
                    <select v-model="rec.judgement" class="grid-select" @click.stop>
                      <option value="Passes">Passes</option>
                      <option value="Failed">Failed</option>
                    </select>
                    <button class="btn-save" @click.stop="saveInlineEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startInlineEdit(idx, 'judgement')">{{ rec.judgement }}</span>
                </td>
                <td>{{ rec.remark }}</td>
                <td>{{ rec.doneBy }}</td>
                <td>{{ rec.verifiedBy }}</td>
                <td>{{ rec.confirmedBy }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="pagination-bar">
          <div class="page-controls">
            <span class="pi pi-search">
               <svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg>
            </span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
            <span class="page-text">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M16 6v12h2V6zM6 18l8.5-6L6 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></span>
            <span class="display-text" style="color: #444; margin-left:10px;">Displaying 1 to {{ qaRecords.length }} of {{ qaRecords.length }} items</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- VCM PULL TEST LIST Sub-Panel -->
  <div v-if="!isCreating && localRecord.testType === 'Pull Test'" class="sub-panel-wrapper">
    <div class="sub-panel-header" @click="togglePullList">
      <span class="sub-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!isPullListOpen" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 11px; text-transform: uppercase;">VCM PULL TEST LIST</span>
    </div>

    <div v-if="isPullListOpen" class="sub-panel-body">
      <div class="sub-panel-inner-box">
        <div class="sub-actions" style="padding: 5px 15px; border-bottom: 1px solid #ddd;">
          <span class="text-link" @click="openAddPullModal">Add New Test Record</span>
        </div>
        <div class="table-scroll-container">
          <table class="data-table dim-table">
            <thead>
              <tr>
                <th class="col-icon"></th>
                <th>Plating Line</th>
                <th>Model</th>
                <th>Lot No.</th>
                <th>Bonding Jig Used</th>
                <th>Bonding Area (cm2)</th>
                <th>Sample</th>
                <th>Adhesion Force (kg.f/cm2)</th>
                <th>Sample Condition</th>
                <th>Result</th>
                <th>Substrate Color</th>
                <th>Remark</th>
                <th>Done By (ID No.)</th>
                <th>Confirmed By (ID No.)</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(rec, idx) in pullRecords" :key="idx">
                <td class="col-icon">
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;" @click="openEditPullModal(idx)"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                </td>
                <td>{{ rec.platingLine }}</td>
                <td>{{ rec.model }}</td>
                <td>{{ rec.lotNo }}</td>
                <td>{{ rec.jig }}</td>
                <td>{{ rec.bondingArea }}</td>
                <td>{{ rec.sample }}</td>
                <td>{{ rec.adhesionForce }}</td>
                <td>{{ rec.condition }}</td>
                <td>
                  <div v-if="inlinePullEditing.rowIdx === idx && inlinePullEditing.field === 'result'" class="inline-edit">
                    <select v-model="rec.result" class="grid-select" @click.stop>
                      <option value="OK">OK</option>
                      <option value="NG">NG</option>
                    </select>
                    <button class="btn-save" @click.stop="savePullInlineEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startPullInlineEdit(idx, 'result')">{{ rec.result }}</span>
                </td>
                <td>{{ rec.substrateColor }}</td>
                <td>{{ rec.remark }}</td>
                <td>{{ rec.doneBy }}</td>
                <td>{{ rec.confirmedBy }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="pagination-bar">
          <div class="page-controls">
            <span class="pi pi-search">
               <svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg>
            </span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
            <span class="page-text">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M16 6v12h2V6zM6 18l8.5-6L6 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></span>
            <span class="display-text" style="color: #444; margin-left:10px;">Displaying 1 to {{ pullRecords.length }} of {{ pullRecords.length }} items</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- QUENCH TEST LIST Sub-Panel -->
  <div v-if="!isCreating && localRecord.testType === 'Quench Test'" class="sub-panel-wrapper">
    <div class="sub-panel-header" @click="toggleQuenchList">
      <span class="sub-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!isQuenchListOpen" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 11px; text-transform: uppercase;">QUENCH TEST LIST</span>
    </div>

    <div v-if="isQuenchListOpen" class="sub-panel-body">
      <div class="sub-panel-inner-box">
        <div class="sub-actions" style="padding: 5px 15px; border-bottom: 1px solid #ddd;">
          <span class="text-link" @click="openAddQuenchModal">Add New Test Record</span>
        </div>
        <div class="table-scroll-container">
          <table class="data-table dim-table">
            <thead>
              <tr>
                <th class="col-icon"></th>
                <th>Month</th>
                <th>Year</th>
                <th>Model</th>
                <th>Date</th>
                <th>Lot No</th>
                <th>Line</th>
                <th>Testing Date Time</th>
                <th>Result</th>
                <th>Judgement</th>
                <th>Remark</th>
                <th>Done By (ID Number)</th>
                <th>Verified By (ID Number)</th>
                <th>Confirmed By (ID Number)</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(rec, idx) in quenchRecords" :key="idx">
                <td class="col-icon">
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;" @click="openEditQuenchModal(idx)"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                </td>
                <td>{{ rec.month }}</td>
                <td>{{ rec.year }}</td>
                <td>{{ rec.model }}</td>
                <td>{{ rec.date }}</td>
                <td>{{ rec.lotNo }}</td>
                <td>{{ rec.line }}</td>
                <td>{{ rec.testingDateTime }}</td>
                <td>
                  <div v-if="inlineQuenchEditing.rowIdx === idx && inlineQuenchEditing.field === 'result'" class="inline-edit">
                    <select v-model="rec.result" class="grid-select" @click.stop>
                      <option value="OK">OK</option>
                      <option value="NG">NG</option>
                    </select>
                    <button class="btn-save" @click.stop="saveQuenchInlineEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startQuenchInlineEdit(idx, 'result')">{{ rec.result }}</span>
                </td>
                <td>
                  <div v-if="inlineQuenchEditing.rowIdx === idx && inlineQuenchEditing.field === 'judgement'" class="inline-edit">
                    <select v-model="rec.judgement" class="grid-select" @click.stop>
                      <option value="Passes">Passes</option>
                      <option value="Failed">Failed</option>
                    </select>
                    <button class="btn-save" @click.stop="saveQuenchInlineEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startQuenchInlineEdit(idx, 'judgement')">{{ rec.judgement }}</span>
                </td>
                <td>{{ rec.remark }}</td>
                <td>{{ rec.doneBy }}</td>
                <td>{{ rec.verifiedBy }}</td>
                <td>{{ rec.confirmedBy }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="pagination-bar">
          <div class="page-controls">
            <span class="pi pi-search">
               <svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg>
            </span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
            <span class="page-text">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M16 6v12h2V6zM6 18l8.5-6L6 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></span>
            <span class="display-text" style="color: #444; margin-left:10px;">Displaying 1 to {{ quenchRecords.length }} of {{ quenchRecords.length }} items</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- THERMAL DEMAGNETISATION TEST LIST Sub-Panel -->
  <div v-if="!isCreating && localRecord.testType === 'Thermal Demagnetisation Test'" class="sub-panel-wrapper">
    <div class="sub-panel-header" @click="toggleThermalList">
      <span class="sub-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!isThermalListOpen" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 11px; text-transform: uppercase;">THERMAL DEMAGNETISATION TEST LIST</span>
    </div>

    <div v-if="isThermalListOpen" class="sub-panel-body">
      <div class="sub-panel-inner-box">
        <div class="sub-actions" style="padding: 5px 15px; border-bottom: 1px solid #ddd;">
          <span class="text-link" @click="openAddThermalModal">Add New Test Record</span>
        </div>
        <div class="table-scroll-container">
          <table class="data-table dim-table">
            <thead>
              <tr>
                <th class="col-icon"></th>
                <th>Month</th>
                <th>Year</th>
                <th>Plating Line</th>
                <th>Plating Date</th>
                <th>Time</th>
                <th>Model</th>
                <th>Lot No</th>
                <th>PA Number</th>
                <th>Forming No</th>
                <th v-for="i in 10" :key="'bfh'+i">Flux Before ({{i}})</th>
                <th v-for="i in 10" :key="'afh'+i">Flux After ({{i}})</th>
                <th>SF Before</th>
                <th>SF After</th>
                <th>SF Factor</th>
                <th>GS Standard</th>
                <th>GS Measured</th>
                <th>GS Factor</th>
                <th v-for="i in 10" :key="'dfh'+i">Flux Diff % ({{i}})</th>
                <th v-for="i in 10" :key="'acth'+i">Actual Flux After ({{i}})</th>
                <th>Result</th>
                <th>Judgement</th>
                <th>S/N Oven</th>
                <th>Remark</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(rec, idx) in thermalRecords" :key="idx">
                <td class="col-icon">
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;" @click="openEditThermalModal(idx)"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                </td>
                <td>{{ rec.month }}</td>
                <td>{{ rec.year }}</td>
                <td>{{ rec.platingLine }}</td>
                <td>{{ rec.platingDate }}</td>
                <td>{{ rec.time }}</td>
                <td>{{ rec.model }}</td>
                <td>{{ rec.lotNo }}</td>
                <td>{{ rec.paNumber }}</td>
                <td>{{ rec.formingNo }}</td>
                <td v-for="(val, i) in rec.bf" :key="'bfv'+i">{{ val }}</td>
                <td v-for="(val, i) in rec.af" :key="'afv'+i">{{ val }}</td>
                <td>{{ rec.sfBefore }}</td>
                <td>{{ rec.sfAfter }}</td>
                <td>{{ rec.sfFactor }}</td>
                <td>{{ rec.gsStandard }}</td>
                <td>{{ rec.gsMeasured }}</td>
                <td>{{ rec.gsFactor }}</td>
                <td v-for="(val, i) in rec.diff" :key="'dfv'+i">{{ val }}</td>
                <td v-for="(val, i) in rec.act" :key="'actv'+i">{{ val }}</td>
                <td>
                  <div v-if="inlineThermalEditing.rowIdx === idx && inlineThermalEditing.field === 'result'" class="inline-edit">
                    <select v-model="rec.result" class="grid-select" @click.stop>
                      <option value="OK">OK</option>
                      <option value="NG">NG</option>
                    </select>
                    <button class="btn-save" @click.stop="saveThermalInlineEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startThermalInlineEdit(idx, 'result')">{{ rec.result }}</span>
                </td>
                <td>
                  <div v-if="inlineThermalEditing.rowIdx === idx && inlineThermalEditing.field === 'judgement'" class="inline-edit">
                    <select v-model="rec.judgement" class="grid-select" @click.stop>
                      <option value="Passes">Passes</option>
                      <option value="Failed">Failed</option>
                    </select>
                    <button class="btn-save" @click.stop="saveThermalInlineEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startThermalInlineEdit(idx, 'judgement')">{{ rec.judgement }}</span>
                </td>
                <td>{{ rec.oven }}</td>
                <td>{{ rec.remark }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="pagination-bar">
          <div class="page-controls">
            <span class="pi pi-search">
               <svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg>
            </span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
            <span class="page-text">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M16 6v12h2V6zM6 18l8.5-6L6 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></span>
            <span class="display-text" style="color: #444; margin-left:10px;">Displaying 1 to {{ thermalRecords.length }} of {{ thermalRecords.length }} items</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- ROUTINE RELIABILITY TEST LIST Sub-Panel -->
  <div v-if="!isCreating && localRecord.testType === 'Routine Reliability Test'" class="sub-panel-wrapper">
    <div class="sub-panel-header" @click="toggleRoutineList">
      <span class="sub-panel-icon-btn">
        <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
          <path v-if="!isRoutineListOpen" d="M10 17l5-5-5-5v10z"/>
          <path v-else d="M7 10l5 5 5-5H7z"/>
        </svg>
        <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
      </span>
      <span style="font-weight: bold; font-size: 11px; text-transform: uppercase;">ROUTINE RELIABILITY TEST LIST</span>
    </div>

    <div v-if="isRoutineListOpen" class="sub-panel-body">
      <div class="sub-panel-inner-box">
        <div class="sub-actions" style="padding: 5px 15px; border-bottom: 1px solid #ddd;">
          <span class="text-link" @click="openAddRoutineModal">Add New Test Record</span>
        </div>
        <div class="table-scroll-container">
          <table class="data-table dim-table">
            <thead>
              <tr>
                <th class="col-icon"></th>
                <th>Month</th>
                <th>Year</th>
                <th>Plating Line</th>
                <th>Plating Date</th>
                <th>Time</th>
                <th>Model</th>
                <th>Lot No</th>
                <th>PA Number</th>
                <th>Forming No</th>
                <th>Pressure Cooker Test</th>
                <th>Colour of Subtratum</th>
                <th>Result Colour of Subtratum</th>
                <th>Shear Test (1)</th>
                <th>Shear Test (2)</th>
                <th>Shear Test (3)</th>
                <th>Shear Test (4)</th>
                <th>Shear Test (5)</th>
                <th>Result of Shear Test</th>
                <th>Pull Test (1)</th>
                <th>Pull Test (2)</th>
                <th>Pull Test (3)</th>
                <th>Pull Test (4)</th>
                <th>Pull Test (5)</th>
                <th>Result of Pull Test</th>
                <th>S/N Oven</th>
                <th>TDT - OK</th>
                <th>TDT - NG</th>
                <th>Remark</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(rec, idx) in routineRecords" :key="idx">
                <td class="col-icon">
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;" @click="openEditRoutineModal(idx)"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                </td>
                <td>{{ rec.month }}</td>
                <td>{{ rec.year }}</td>
                <td>{{ rec.platingLine }}</td>
                <td>{{ rec.platingDate }}</td>
                <td>{{ rec.time }}</td>
                <td>{{ rec.model }}</td>
                <td>{{ rec.lotNo }}</td>
                <td>{{ rec.paNumber }}</td>
                <td>{{ rec.formingNo }}</td>
                <td>
                  <div v-if="inlineRoutineEditing.rowIdx === idx && inlineRoutineEditing.field === 'cookerTest'" class="inline-edit">
                    <select v-model="rec.cookerTest" class="grid-select" @click.stop>
                      <option v-for="opt in cookerOptions" :key="opt" :value="opt">{{opt}}</option>
                    </select>
                    <button class="btn-save" @click.stop="saveRoutineInlineEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startRoutineInlineEdit(idx, 'cookerTest')">{{ rec.cookerTest }}</span>
                </td>
                <td>
                  <div v-if="inlineRoutineEditing.rowIdx === idx && inlineRoutineEditing.field === 'colorSub'" class="inline-edit">
                    <select v-model="rec.colorSub" class="grid-select" @click.stop>
                      <option v-for="opt in colorSubOptions" :key="opt" :value="opt">{{opt}}</option>
                    </select>
                    <button class="btn-save" @click.stop="saveRoutineInlineEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startRoutineInlineEdit(idx, 'colorSub')">{{ rec.colorSub }}</span>
                </td>
                <td>
                  <div v-if="inlineRoutineEditing.rowIdx === idx && inlineRoutineEditing.field === 'resultColorSub'" class="inline-edit">
                    <select v-model="rec.resultColorSub" class="grid-select" @click.stop>
                      <option value="OK">OK</option>
                      <option value="NG">NG</option>
                    </select>
                    <button class="btn-save" @click.stop="saveRoutineInlineEdit">Save</button>
                  </div>
                  <span v-else class="editable-text" @click="startRoutineInlineEdit(idx, 'resultColorSub')">{{ rec.resultColorSub }}</span>
                </td>
                <td>{{ rec.shear1 }}</td>
                <td>{{ rec.shear2 }}</td>
                <td>{{ rec.shear3 }}</td>
                <td>{{ rec.shear4 }}</td>
                <td>{{ rec.shear5 }}</td>
                <td>{{ rec.resultShear }}</td>
                <td>{{ rec.pull1 }}</td>
                <td>{{ rec.pull2 }}</td>
                <td>{{ rec.pull3 }}</td>
                <td>{{ rec.pull4 }}</td>
                <td>{{ rec.pull5 }}</td>
                <td>{{ rec.resultPull }}</td>
                <td>{{ rec.oven }}</td>
                <td>{{ rec.tdtOk }}</td>
                <td>{{ rec.tdtNg }}</td>
                <td>{{ rec.remark }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="pagination-bar">
          <div class="page-controls">
            <span class="pi pi-search">
               <svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg>
            </span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg></span>
            <span class="page-text">Page <input type="text" value="1" class="page-input" /> of 1</span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M16 6v12h2V6zM6 18l8.5-6L6 6z"/></svg></span>
            <span class="p-btn"><svg viewBox="0 0 24 24" width="14" height="14" fill="#333"><path d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></span>
            <span class="display-text" style="color: #444; margin-left:10px;">Displaying 1 to {{ routineRecords.length }} of {{ routineRecords.length }} items</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Add New Test Record Modal -->
  <div class="modal-overlay" v-if="showAddModal">
    <div class="modal-window">
      <!-- Sub Header -->
      <div class="sub-header modal-sub-header">
        <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        <span style="font-weight: bold; color: black;">TEST RECORD MANAGEMENT</span>
      </div>
      
      <div class="panel modal-panel">
        <div class="top-actions" style="padding: 15px 0 10px 0; background-color: transparent;">
          <button class="btn btn-primary" @click="isEditingSubRecord ? updateTestRecord() : addTestRecord()">{{ isEditingSubRecord ? 'UPDATE' : 'ADD' }}</button>
          <button class="btn btn-secondary" @click="showAddModal = false; isEditingSubRecord = false">Cancel</button>
        </div>

        <div class="section-container" style="margin: 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Record Details</div>
          <div class="info-grid">
            
            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">Plating Date</div>
                <div class="grid-value modal-value"><input type="date" class="form-input" v-model="newRecord.platingDate" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Line</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRecord.line" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Model</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRecord.model" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Lot No</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRecord.lotNo" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Loading Date Time</div>
                <div class="grid-value modal-value"><input type="datetime-local" class="form-input" v-model="newRecord.loadingDateTime" /></div>
              </div>
            </div>

            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">Result</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newRecord.result">
                    <option value="OK">OK</option>
                    <option value="NG">NG</option>
                  </select>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Judgement</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newRecord.judgement">
                    <option value="Passes">Passes</option>
                    <option value="Failed">Failed</option>
                  </select>
                </div>
              </div>
              <div class="grid-row" style="flex:1;">
                <div class="grid-label modal-label">Remark</div>
                <div class="grid-value modal-value" style="height: 100%;">
                  <textarea class="form-input" style="height: 100%; min-height: 40px; resize: none;" v-model="newRecord.remark"></textarea>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Done By (ID Number)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRecord.doneBy" /></div>
              </div>
            </div>

          </div>
        </div>
        
        <div class="top-actions" style="padding: 15px 0 0 0; background-color: transparent;">
          <button class="btn btn-primary" @click="addTestRecord">ADD</button>
          <button class="btn btn-secondary" @click="showAddModal = false">Cancel</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Add VCM Pull Test Modal -->
  <div class="modal-overlay" v-if="showAddPullModal">
    <div class="modal-window">
      <!-- Sub Header -->
      <div class="sub-header modal-sub-header">
        <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        <span style="font-weight: bold; color: black;">VCM PULL TEST RECORD MANAGEMENT</span>
      </div>
      
      <div class="panel modal-panel">
        <div class="top-actions" style="padding: 15px 0 10px 0; background-color: transparent;">
          <button class="btn btn-primary" @click="addPullRecord">ADD</button>
          <button class="btn btn-secondary" @click="showAddPullModal = false">Cancel</button>
        </div>

        <div class="section-container" style="margin: 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Record Details</div>
          <div class="info-grid">
            
            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">Plating Line</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newPullRecord.platingLine">
                    <option v-for="opt in platingLineOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Model</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newPullRecord.model" @change="handlePullModelChange">
                    <option v-for="opt in modelOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Lot No.</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newPullRecord.lotNo" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Bonding Jig Used</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newPullRecord.jig" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Bonding Area (cm2)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newPullRecord.bondingArea" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Sample</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newPullRecord.sample" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Adhesion Force (kg.f/cm2)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newPullRecord.adhesionForce" /></div>
              </div>
            </div>

            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">Sample Condition</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newPullRecord.condition">
                    <option v-for="opt in conditionOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Result</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newPullRecord.result">
                    <option value="OK">OK</option>
                    <option value="NG">NG</option>
                  </select>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Substrate Color</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newPullRecord.substrateColor">
                    <option v-for="opt in substrateColorOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
              <div class="grid-row" style="flex:1;">
                <div class="grid-label modal-label">Remark</div>
                <div class="grid-value modal-value" style="height: 100%;">
                  <textarea class="form-input" style="height: 100%; min-height: 40px; resize: none;" v-model="newPullRecord.remark"></textarea>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Done By (ID No.)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newPullRecord.doneBy" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Confirmed By (ID No.)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newPullRecord.confirmedBy" /></div>
              </div>
            </div>

          </div>
        </div>
        
        <div class="top-actions" style="padding: 15px 0 0 0; background-color: transparent;">
          <button class="btn btn-primary" @click="isEditingSubRecord ? updatePullRecord() : addPullRecord()">{{ isEditingSubRecord ? 'UPDATE' : 'ADD' }}</button>
          <button class="btn btn-secondary" @click="showAddPullModal = false; isEditingSubRecord = false">Cancel</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Add QUENCH TEST RECORD Modal -->
  <div class="modal-overlay" v-if="showAddQuenchModal">
    <div class="modal-window">
      <div class="sub-header modal-sub-header">
        <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        <span style="font-weight: bold; color: black;">QUENCH TEST RECORD MANAGEMENT</span>
      </div>
      
      <div class="panel modal-panel">
        <div class="top-actions" style="padding: 15px 0 10px 0; background-color: transparent;">
          <button class="btn btn-primary" @click="addQuenchRecord">ADD</button>
          <button class="btn btn-secondary" @click="showAddQuenchModal = false">Cancel</button>
        </div>

        <div class="section-container" style="margin: 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Record Details</div>
          <div class="info-grid">
            
            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">Month</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newQuenchRecord.month" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Year</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newQuenchRecord.year" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Model</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newQuenchRecord.model" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Date</div>
                <div class="grid-value modal-value"><input type="date" class="form-input" v-model="newQuenchRecord.date" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Lot No</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newQuenchRecord.lotNo" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Line</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newQuenchRecord.line" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Testing Date Time</div>
                <div class="grid-value modal-value"><input type="datetime-local" class="form-input" v-model="newQuenchRecord.testingDateTime" /></div>
              </div>
            </div>

            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">Result</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newQuenchRecord.result">
                    <option value="OK">OK</option>
                    <option value="NG">NG</option>
                  </select>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Judgement</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newQuenchRecord.judgement">
                    <option value="Passes">Passes</option>
                    <option value="Failed">Failed</option>
                  </select>
                </div>
              </div>
              <div class="grid-row" style="flex:1;">
                <div class="grid-label modal-label">Remark</div>
                <div class="grid-value modal-value" style="height: 100%;">
                  <textarea class="form-input" style="height: 100%; min-height: 40px; resize: none;" v-model="newQuenchRecord.remark"></textarea>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Done By (ID Number)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newQuenchRecord.doneBy" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Verified By (ID Number)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newQuenchRecord.verifiedBy" /></div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Confirmed By (ID Number)</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newQuenchRecord.confirmedBy" /></div>
              </div>
            </div>

          </div>
        </div>
        
        <div class="top-actions" style="padding: 15px 0 0 0; background-color: transparent;">
          <button class="btn btn-primary" @click="isEditingSubRecord ? updateQuenchRecord() : addQuenchRecord()">{{ isEditingSubRecord ? 'UPDATE' : 'ADD' }}</button>
          <button class="btn btn-secondary" @click="showAddQuenchModal = false; isEditingSubRecord = false">Cancel</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Add THERMAL DEMAGNETISATION TEST RECORD Modal -->
  <div class="modal-overlay" v-if="showAddThermalModal">
    <div class="modal-window" style="width: 90%; max-width: 1000px; max-height: 90vh; overflow-y: auto;">
      <div class="sub-header modal-sub-header">
        <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        <span style="font-weight: bold; color: black;">THERMAL DEMAGNETISATION TEST RECORD MANAGEMENT</span>
      </div>
      
      <div class="panel modal-panel">
        <div class="top-actions" style="padding: 15px 0 10px 0; background-color: transparent;">
          <button class="btn btn-primary" @click="isEditingSubRecord ? updateThermalRecord() : addThermalRecord()">{{ isEditingSubRecord ? 'UPDATE' : 'ADD' }}</button>
          <button class="btn btn-secondary" @click="showAddThermalModal = false; isEditingSubRecord = false">Cancel</button>
        </div>

        <!-- Basic Info Section -->
        <div class="section-container" style="margin: 0 0 15px 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Basic Information</div>
          <div class="info-grid">
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">Month</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.month" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Year</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.year" /></div></div>
              <div class="grid-row">
                <div class="grid-label modal-label">Plating Line</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newThermalRecord.platingLine">
                    <option v-for="opt in thermalPlatingLineOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
              <div class="grid-row"><div class="grid-label modal-label">Plating Date</div><div class="grid-value modal-value"><input type="date" class="form-input" v-model="newThermalRecord.platingDate" /></div></div>
              <div class="grid-row">
                <div class="grid-label modal-label">Time</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newThermalRecord.time">
                    <option v-for="opt in thermalTimeOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
            </div>
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">Model</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.model" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Lot No</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.lotNo" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">PA Number</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.paNumber" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Forming No</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.formingNo" /></div></div>
            </div>
          </div>
        </div>

        <!-- Flux Measurements Section -->
        <div class="section-container" style="margin: 0 0 15px 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Flux Measurements (Before / After)</div>
          <div class="info-grid">
            <div class="grid-col">
              <div class="grid-row" v-for="i in 10" :key="'bfm'+i">
                <div class="grid-label modal-label">Flux Value - Before ({{i}})</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.bf[i-1]" /></div>
              </div>
            </div>
            <div class="grid-col">
              <div class="grid-row" v-for="i in 10" :key="'afm'+i">
                <div class="grid-label modal-label">Flux Value - After ({{i}})</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.af[i-1]" /></div>
              </div>
            </div>
          </div>
        </div>

        <!-- Factors Section -->
        <div class="section-container" style="margin: 0 0 15px 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Factors & Standards</div>
          <div class="info-grid">
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">Sample Factor - Before</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.sfBefore" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Sample Factor - After</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.sfAfter" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Sample Factor - Factor</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.sfFactor" /></div></div>
            </div>
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">Gold Standard - Standard</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.gsStandard" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Gold Standard - Measured</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.gsMeasured" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Gold Standard - Factor</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.gsFactor" /></div></div>
            </div>
          </div>
        </div>

        <!-- Calculations Section -->
        <div class="section-container" style="margin: 0 0 15px 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Calculated Flux (Diff / Actual After)</div>
          <div class="info-grid">
            <div class="grid-col">
              <div class="grid-row" v-for="i in 10" :key="'dfm'+i">
                <div class="grid-label modal-label">Flux Difference % ({{i}})</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.diff[i-1]" /></div>
              </div>
            </div>
            <div class="grid-col">
              <div class="grid-row" v-for="i in 10" :key="'actm'+i">
                <div class="grid-label modal-label">Actual Flux - After ({{i}})</div>
                <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.act[i-1]" /></div>
              </div>
            </div>
          </div>
        </div>

        <!-- Result Section -->
        <div class="section-container" style="margin: 0 0 15px 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Final Judgement</div>
          <div class="info-grid">
            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">Result</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newThermalRecord.result">
                    <option value="OK">OK</option>
                    <option value="NG">NG</option>
                  </select>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Judgement</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newThermalRecord.judgement">
                    <option value="Passes">Passes</option>
                    <option value="Failed">Failed</option>
                  </select>
                </div>
              </div>
            </div>
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">S/N Oven</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newThermalRecord.oven" /></div></div>
              <div class="grid-row" style="flex:1;">
                <div class="grid-label modal-label">Remark</div>
                <div class="grid-value modal-value" style="height: 100%;">
                  <textarea class="form-input" style="height: 100%; min-height: 40px; resize: none;" v-model="newThermalRecord.remark"></textarea>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        <div class="top-actions" style="padding: 15px 0 0 0; background-color: transparent;">
          <button class="btn btn-primary" @click="addThermalRecord">ADD</button>
          <button class="btn btn-secondary" @click="showAddThermalModal = false">Cancel</button>
        </div>
      </div>
    </div>
    </div>

  <!-- Add ROUTINE RELIABILITY TEST RECORD Modal -->
  <div class="modal-overlay" v-if="showAddRoutineModal">
    <div class="modal-window" style="width: 90%; max-width: 1000px; max-height: 90vh; overflow-y: auto;">
      <div class="sub-header modal-sub-header">
        <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        <span style="font-weight: bold; color: black;">ROUTINE RELIABILITY TEST RECORD MANAGEMENT</span>
      </div>
      
      <div class="panel modal-panel">
        <div class="top-actions" style="padding: 15px 0 10px 0; background-color: transparent;">
          <button class="btn btn-primary" @click="addRoutineRecord">ADD</button>
          <button class="btn btn-secondary" @click="showAddRoutineModal = false">Cancel</button>
        </div>

        <!-- Basic Info Section -->
        <div class="section-container" style="margin: 0 0 15px 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Basic Information</div>
          <div class="info-grid">
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">Month</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.month" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Year</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.year" /></div></div>
              <div class="grid-row">
                <div class="grid-label modal-label">Plating Line</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newRoutineRecord.platingLine">
                    <option v-for="opt in routinePlatingLineOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
              <div class="grid-row"><div class="grid-label modal-label">Plating Date</div><div class="grid-value modal-value"><input type="date" class="form-input" v-model="newRoutineRecord.platingDate" /></div></div>
              <div class="grid-row">
                <div class="grid-label modal-label">Time</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newRoutineRecord.time">
                    <option v-for="opt in routineTimeOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
            </div>
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">Model</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.model" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Lot No</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.lotNo" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">PA Number</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.paNumber" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Forming No</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.formingNo" /></div></div>
            </div>
          </div>
        </div>

        <!-- Pressure Cooker & Color Section -->
        <div class="section-container" style="margin: 0 0 15px 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Pressure Cooker & Substratum Color</div>
          <div class="info-grid">
            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">Pressure Cooker Test</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newRoutineRecord.cookerTest">
                    <option v-for="opt in cookerOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
              <div class="grid-row">
                <div class="grid-label modal-label">Colour of Subtratum</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newRoutineRecord.colorSub">
                    <option v-for="opt in colorSubOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
            </div>
            <div class="grid-col">
              <div class="grid-row">
                <div class="grid-label modal-label">Result Colour of Subtratum</div>
                <div class="grid-value modal-value">
                  <select class="form-input" v-model="newRoutineRecord.resultColorSub">
                    <option v-for="opt in okNgOptions" :key="opt" :value="opt">{{opt}}</option>
                  </select>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Shear Test Section -->
        <div class="section-container" style="margin: 0 0 15px 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Shear Test Results</div>
          <div class="info-grid">
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">Shear Test (1)</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.shear1" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Shear Test (2)</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.shear2" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Shear Test (3)</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.shear3" /></div></div>
            </div>
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">Shear Test (4)</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.shear4" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Shear Test (5)</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.shear5" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Result of Shear Test</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.resultShear" /></div></div>
            </div>
          </div>
        </div>

        <!-- Pull Test Section -->
        <div class="section-container" style="margin: 0 0 15px 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Pull Test Results</div>
          <div class="info-grid">
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">Pull Test (1)</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.pull1" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Pull Test (2)</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.pull2" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Pull Test (3)</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.pull3" /></div></div>
            </div>
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">Pull Test (4)</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.pull4" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Pull Test (5)</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.pull5" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">Result of Pull Test</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.resultPull" /></div></div>
            </div>
          </div>
        </div>

        <!-- Finalization Section -->
        <div class="section-container" style="margin: 0; box-shadow: none;">
          <div class="section-title" style="background-color: transparent; padding-left: 0; font-weight: bold; padding-bottom: 10px;">Finalization</div>
          <div class="info-grid">
            <div class="grid-col">
              <div class="grid-row"><div class="grid-label modal-label">S/N Oven</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.oven" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">TDT - OK</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.tdtOk" /></div></div>
              <div class="grid-row"><div class="grid-label modal-label">TDT - NG</div><div class="grid-value modal-value"><input type="text" class="form-input" v-model="newRoutineRecord.tdtNg" /></div></div>
            </div>
            <div class="grid-col">
              <div class="grid-row" style="flex:1;">
                <div class="grid-label modal-label">Remark</div>
                <div class="grid-value modal-value" style="height: 100%;">
                  <textarea class="form-input" style="height: 100%; min-height: 40px; resize: none;" v-model="newRoutineRecord.remark"></textarea>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        <div class="top-actions" style="padding: 15px 0 0 0; background-color: transparent;">
          <button class="btn btn-primary" @click="isEditingSubRecord ? updateRoutineRecord() : addRoutineRecord()">{{ isEditingSubRecord ? 'UPDATE' : 'ADD' }}</button>
          <button class="btn btn-secondary" @click="showAddRoutineModal = false; isEditingSubRecord = false">Cancel</button>
        </div>
      </div>
    </div>
    </div>
  </div>
</template>

<style scoped>
.view-panel {
  background-color: #ffffff;
  min-height: calc(100vh - 130px);
  position: relative;
  padding-top: 10px;
}
.top-record-box {
  background-color: #fff;
  border: 2px solid #c7c7c7;
  margin: 0 15px 15px 15px;
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
.btn-primary:hover { background-color: #7a2a2c; }
.btn-secondary { background-color: #a5a5a5; color: #fff; }
.btn-secondary:hover { background-color: #888; }

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

/* Sub-panel styling */
.sub-panel-wrapper {
  margin: 0 15px 15px 15px;
}

.sub-panel-header {
  background-color: #c7c7c7;
  padding: 6px 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  border: 2px solid #c7c7c7;
}

.sub-panel-header:hover {
  background-color: #c9c9c9;
}

.sub-panel-icon-btn {
  display: flex;
  align-items: center;
  margin-right: -4px;
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

.dim-table {
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

.page-text {
  color: #333;
}

.page-input {
  width: 35px;
  height: 18px;
  border: 1px solid #ccc;
  text-align: center;
  font-size: 11px;
}

/* Inline edit styling */
.inline-edit {
  display: flex;
  align-items: center;
  gap: 4px;
}

.btn-save {
  padding: 2px 6px;
  font-size: 11px;
  border: 1px solid #999;
  background-color: #f6f6f6;
  color: #333;
  cursor: pointer;
  border-radius: 2px;
}

.btn-save:hover {
  background-color: #e5e5e5;
}

.editable-text {
  cursor: pointer;
  display: inline-block;
  min-height: 16px;
  min-width: 20px;
  color: #333;
}
.editable-text:hover {
  outline: 1px dotted #a0a0a0;
  background-color: #fafafa;
}

.grid-select {
  background-color: white;
  border: 1px solid #ccc;
  padding: 1px 4px;
  border-radius: 2px;
  font-size: 11px;
}

.grid-select option {
  background-color: #fff;
}

/* Modal styles styling it exactly like the image given */
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

.modal-panel {
  padding: 0 15px 15px 15px;
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
  width: 100%;
}

.form-input {
  width: 100%;
  height: 24px;
  border: 1px solid #ccc;
  padding: 2px 5px;
  font-size: 12px;
  box-sizing: border-box;
}

.section-container {
  background-color: #fff;
  border: 1px solid #ddd;
  margin: 0 15px 15px 15px;
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
</style>
