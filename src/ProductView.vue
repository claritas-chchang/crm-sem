<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'

const props = defineProps({
  productCode: String,
  product: Object,
  isCreating: Boolean,
  isEditing: { type: Boolean, default: false },
  isModal: { type: Boolean, default: false }
})

const emit = defineEmits(['back', 'save'])

const productType = ref('G')
const typeOptions = ['G', 'Non-G']

// New fields requested by user
const modelName = ref('M-456')
const materialCode = ref('RM-0012')
const materialPowderType = ref('Type A')
const materialGrade = ref('Grade N52')
const magneticDirection = ref('Axial')
const marking = ref('Laser')
const bendingStrengthMin = ref('280')
const customerWeightMin = ref('40.5')
const customerWeightMinSpec = ref('40.6')
const magnetization = ref('MAGNETIZED')
const productDimension = ref('24.5')
const tolerance = ref('0.05')
const productWeight = ref('40.7')
const customerDwgNo = ref('DWG-99482')
const totalFluxMin = ref('413')
const totalFluxMax = ref('438')

// Magnetic Properties (CGS)
const brMinCgs = ref('11.0')
const ihcMinCgs = ref('800')
const bhcMinCgs = ref('750')
const bhMaxMinCgs = ref('30')
const brMaxCgs = ref('14.5')
const ihcMaxCgs = ref('1200')
const bhcMaxCgs = ref('900')
const bhMaxMaxCgs = ref('45')

// Magnetic Properties (SI)
const brMinSi = ref('1.1')
const ihcMinSi = ref('64')
const bhcMinSi = ref('60')
const bhMaxMinSi = ref('239')
const brMaxSi = ref('1.45')
const ihcMaxSi = ref('96')
const bhcMaxSi = ref('72')
const bhMaxMaxSi = ref('358')

const magnetizationOptions = ['MAGNETIZED', 'UN-MAGNETIZED']

const isEditing = ref(props.isEditing || props.isCreating)
const tempProductType = ref(props.isCreating ? '' : productType.value)
const localProductCode = ref('')

const tempModelName = ref('')
const tempMaterialCode = ref('')
const tempMaterialPowderType = ref('')
const tempMaterialGrade = ref('')
const tempMagneticDirection = ref('')
const tempMarking = ref('')
const tempBendingStrengthMin = ref('')
const tempCustomerWeightMin = ref('')
const tempCustomerWeightMinSpec = ref('')
const tempMagnetization = ref('')
const tempProductDimension = ref('')
const tempTolerance = ref('')
const tempProductWeight = ref('')
const tempCustomerDwgNo = ref('')
const tempTotalFluxMin = ref('')
const tempTotalFluxMax = ref('')

const tempBrMinCgs = ref('')
const tempIhcMinCgs = ref('')
const tempBhcMinCgs = ref('')
const tempBhMaxMinCgs = ref('')
const tempBrMaxCgs = ref('')
const tempIhcMaxCgs = ref('')
const tempBhcMaxCgs = ref('')
const tempBhMaxMaxCgs = ref('')

const tempBrMinSi = ref('')
const tempIhcMinSi = ref('')
const tempBhcMinSi = ref('')
const tempBhMaxMinSi = ref('')
const tempBrMaxSi = ref('')
const tempIhcMaxSi = ref('')
const tempBhcMaxSi = ref('')
const tempBhMaxMaxSi = ref('')

watch(() => props.product, (newVal) => {
  if (newVal) {
    productType.value = newVal.type || 'G'
    modelName.value = newVal.modelName || 'M-456'
    materialCode.value = newVal.materialCode || 'RM-0012'
    materialPowderType.value = newVal.materialPowderType || 'Type A'
    materialGrade.value = newVal.materialGrade || 'Grade N52'
    magneticDirection.value = newVal.magneticDirection || 'Axial'
    marking.value = newVal.marking || 'Laser'
    bendingStrengthMin.value = newVal.bendingStrengthMin || '280'
    customerWeightMin.value = newVal.customerWeightMin || '40.5'
    customerWeightMinSpec.value = newVal.customerWeightMinSpec || '40.6'
    magnetization.value = newVal.magnetization || 'MAGNETIZED'
    productDimension.value = newVal.productDimension || '24.5'
    tolerance.value = newVal.tolerance || '0.05'
    productWeight.value = newVal.productWeight || '40.7'
    customerDwgNo.value = newVal.customerDwgNo || 'DWG-99482'
    totalFluxMin.value = newVal.totalFluxMin || '413'
    totalFluxMax.value = newVal.totalFluxMax || '438'
    
    brMinCgs.value = newVal.brMinCgs || '11.0'
    ihcMinCgs.value = newVal.ihcMinCgs || '800'
    bhcMinCgs.value = newVal.bhcMinCgs || '750'
    bhMaxMinCgs.value = newVal.bhMaxMinCgs || '30'
    brMaxCgs.value = newVal.brMaxCgs || '14.5'
    ihcMaxCgs.value = newVal.ihcMaxCgs || '1200'
    bhcMaxCgs.value = newVal.bhcMaxCgs || '900'
    bhMaxMaxCgs.value = newVal.bhMaxMaxCgs || '45'

    brMinSi.value = newVal.brMinSi || '1.1'
    ihcMinSi.value = newVal.ihcMinSi || '64'
    bhcMinSi.value = newVal.bhcMinSi || '60'
    bhMaxMinSi.value = newVal.bhMaxMinSi || '239'
    brMaxSi.value = newVal.brMaxSi || '1.45'
    ihcMaxSi.value = newVal.ihcMaxSi || '96'
    bhcMaxSi.value = newVal.bhcMaxSi || '72'
    bhMaxMaxSi.value = newVal.bhMaxMaxSi || '358'
  }
}, { immediate: true })

const startEditing = () => {
  tempProductType.value = productType.value
  tempModelName.value = modelName.value
  tempMaterialCode.value = materialCode.value
  tempMaterialPowderType.value = materialPowderType.value
  tempMaterialGrade.value = materialGrade.value
  tempMagneticDirection.value = magneticDirection.value
  tempMarking.value = marking.value
  tempBendingStrengthMin.value = bendingStrengthMin.value
  tempCustomerWeightMin.value = customerWeightMin.value
  tempCustomerWeightMinSpec.value = customerWeightMinSpec.value
  tempMagnetization.value = magnetization.value
  tempProductDimension.value = productDimension.value
  tempTolerance.value = tolerance.value
  tempProductWeight.value = productWeight.value
  tempCustomerDwgNo.value = customerDwgNo.value
  tempTotalFluxMin.value = totalFluxMin.value
  tempTotalFluxMax.value = totalFluxMax.value
  
  tempBrMinCgs.value = brMinCgs.value
  tempIhcMinCgs.value = ihcMinCgs.value
  tempBhcMinCgs.value = bhcMinCgs.value
  tempBhMaxMinCgs.value = bhMaxMinCgs.value
  tempBrMaxCgs.value = brMaxCgs.value
  tempIhcMaxCgs.value = ihcMaxCgs.value
  tempBhcMaxCgs.value = bhcMaxCgs.value
  tempBhMaxMaxCgs.value = bhMaxMaxCgs.value

  tempBrMinSi.value = brMinSi.value
  tempIhcMinSi.value = ihcMinSi.value
  tempBhcMinSi.value = bhcMinSi.value
  tempBhMaxMinSi.value = bhMaxMinSi.value
  tempBrMaxSi.value = brMaxSi.value
  tempIhcMaxSi.value = ihcMaxSi.value
  tempBhcMaxSi.value = bhcMaxSi.value
  tempBhMaxMaxSi.value = bhMaxMaxSi.value

  isEditing.value = true
}

const saveProduct = () => {
  productType.value = tempProductType.value
  modelName.value = tempModelName.value
  materialCode.value = tempMaterialCode.value
  materialPowderType.value = tempMaterialPowderType.value
  materialGrade.value = tempMaterialGrade.value
  magneticDirection.value = tempMagneticDirection.value
  marking.value = tempMarking.value
  bendingStrengthMin.value = tempBendingStrengthMin.value
  customerWeightMin.value = tempCustomerWeightMin.value
  customerWeightMinSpec.value = tempCustomerWeightMinSpec.value
  magnetization.value = tempMagnetization.value
  productDimension.value = tempProductDimension.value
  tolerance.value = tempTolerance.value
  productWeight.value = tempProductWeight.value
  customerDwgNo.value = tempCustomerDwgNo.value
  totalFluxMin.value = tempTotalFluxMin.value
  totalFluxMax.value = tempTotalFluxMax.value
  
  brMinCgs.value = tempBrMinCgs.value
  ihcMinCgs.value = tempIhcMinCgs.value
  bhcMinCgs.value = tempBhcMinCgs.value
  bhMaxMinCgs.value = tempBhMaxMinCgs.value
  brMaxCgs.value = tempBrMaxCgs.value
  ihcMaxCgs.value = tempIhcMaxCgs.value
  bhcMaxCgs.value = tempBhcMaxCgs.value
  bhMaxMaxCgs.value = tempBhMaxMaxCgs.value

  brMinSi.value = tempBrMinSi.value
  ihcMinSi.value = tempIhcMinSi.value
  bhcMinSi.value = tempBhcMinSi.value
  bhMaxMinSi.value = tempBhMaxMinSi.value
  brMaxSi.value = tempBrMaxSi.value
  ihcMaxSi.value = tempIhcMaxSi.value
  bhcMaxSi.value = tempBhcMaxSi.value
  bhMaxMaxSi.value = tempBhMaxMaxSi.value

  isEditing.value = false

  emit('save', {
    ...(props.product || {}),
    code: props.isCreating ? localProductCode.value : props.productCode,
    type: productType.value,
    modelName: modelName.value,
    materialCode: materialCode.value,
    materialPowderType: materialPowderType.value,
    materialGrade: materialGrade.value,
    magneticDirection: magneticDirection.value,
    marking: marking.value,
    bendingStrengthMin: bendingStrengthMin.value,
    customerWeightMin: customerWeightMin.value,
    customerWeightMinSpec: customerWeightMinSpec.value,
    magnetization: magnetization.value,
    productDimension: productDimension.value,
    tolerance: tolerance.value,
    productWeight: productWeight.value,
    customerDwgNo: customerDwgNo.value,
    totalFluxMin: totalFluxMin.value,
    totalFluxMax: totalFluxMax.value,
    brMinCgs: brMinCgs.value,
    ihcMinCgs: ihcMinCgs.value,
    bhcMinCgs: bhcMinCgs.value,
    bhMaxMinCgs: bhMaxMinCgs.value,
    brMaxCgs: brMaxCgs.value,
    ihcMaxCgs: ihcMaxCgs.value,
    bhcMaxCgs: bhcMaxCgs.value,
    bhMaxMaxCgs: bhMaxMaxCgs.value,
    brMinSi: brMinSi.value,
    ihcMinSi: ihcMinSi.value,
    bhcMinSi: bhcMinSi.value,
    bhMaxMinSi: bhMaxMinSi.value,
    brMaxSi: brMaxSi.value,
    ihcMaxSi: ihcMaxSi.value,
    bhcMaxSi: bhcMaxSi.value,
    bhMaxMaxSi: bhMaxMaxSi.value
  })
}

const cancelProductEdit = () => {
  if (props.isCreating) {
    emit('back')
  } else {
    isEditing.value = false
  }
}

const dimensions = ref([
  { name: 'Length', min: '24.45', max: '24.55', size: '24.5', uom: 'mm', tol: '0.05', minOnly: 'No', maxOnly: 'No', eq: 'Micrometer', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: '-' },
  { name: 'Width', min: '24.45', max: '24.55', size: '24.5', uom: 'mm', tol: '0.05', minOnly: 'No', maxOnly: 'No', eq: 'Micrometer', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: '-' },
  { name: 'Thickness', min: '8.85', max: '8.95', size: '8.9', uom: 'mm', tol: '0.05', minOnly: 'No', maxOnly: 'No', eq: 'Micrometer', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: '-' },
  { name: 'Flatness', min: '-', max: '0.08', size: '0.08', uom: 'mm', tol: '0', minOnly: 'No', maxOnly: 'Yes', eq: 'Linear gage', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: '-' },
  { name: 'Parallelism', min: '-', max: '0.08', size: '0.08', uom: 'mm', tol: '0', minOnly: 'No', maxOnly: 'Yes', eq: 'Linear gage', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: '-' },
  { name: 'Perpendicularity', min: '-', max: '0.08', size: '0.08', uom: 'mm', tol: '0', minOnly: 'No', maxOnly: 'Yes', eq: 'Square, Thickness gage', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: '-' },
  { name: 'Weight', min: '40.6', max: '-', size: '-', uom: 'g', tol: '0', minOnly: 'Yes', maxOnly: 'No', eq: 'Electronic Balance', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: '-' },
  { name: 'Coating Thickness', min: '5', max: '25', size: '-', uom: 'µm', tol: '0', minOnly: 'No', maxOnly: 'No', eq: 'X-Ray Machine', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: '-' },
  { name: 'Appearance (before magnetize)', min: '-', max: '-', size: '-', uom: 'remarks', tol: '-', minOnly: '-', maxOnly: '-', eq: 'Naked eyes', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: 'Per visual inspection criteria for magnet' },
  { name: 'Green Paper Check', min: '-', max: '-', size: '-', uom: 'remarks', tol: '-', minOnly: '-', maxOnly: '-', eq: 'Green Paper', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: 'Per WI No. : W-0604-QA' },
  { name: 'Magnet Sticking Checking', min: '-', max: '-', size: '-', uom: 'remarks', tol: '-', minOnly: '-', maxOnly: '-', eq: '-', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: 'Per WI No. : W-0604-QA' },
  { name: 'Total Flux', min: '413', max: '438', size: 'x10-4', uom: '[Wb.Ts.]', tol: '-', minOnly: '-', maxOnly: '-', eq: 'Flux meter + Fixture', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: 'TG-31' },
  { name: 'Polarity', min: '-', max: '-', size: '-', uom: 'remarks', tol: '-', minOnly: '-', maxOnly: '-', eq: 'Polarity Checker', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: 'Horizontal RED line, along the top area of W x T surface, with N-pole facing upwards.' },
  { name: 'Appearance (After Magnetizing)', min: '-', max: '-', size: '-', uom: 'remarks', tol: '-', minOnly: '-', maxOnly: '-', eq: 'Naked eyes', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: 'Per visual inspection criteria for magnet' },
  { name: 'Magnetized Condition', min: '-', max: '-', size: '-', uom: 'remarks', tol: '-', minOnly: '-', maxOnly: '-', eq: 'Green Paper', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: 'Fully magnetized & no sign of abnormal grain growth' },
  { name: 'Marking Position', min: '-', max: '-', size: '-', uom: 'remarks', tol: '-', minOnly: '-', maxOnly: '-', eq: 'Naked eyes', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: 'Position. Horizontal RED line...' },
  { name: 'Marking Color', min: '-', max: '-', size: '-', uom: 'remarks', tol: '-', minOnly: '-', maxOnly: '-', eq: 'Naked eyes', samp: 'Level I, AQL 1%', type: 'Reduced', sampSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', remarks: 'Red' }
])

const showAddModal = ref(false)

const nameOptions = ['Length', 'Length 1', 'Length 2', 'Width', 'Width 1', 'Width 2', 'Thickness', 'Thickness 1', 'Thickness 2', 'Height', 'Height 1', 'Height 2', 'Outer Radius', 'Outer Radius 1', 'Outer Radius 2', 'Inner Radius', 'Inner Radius 1', 'Inner Radius 2', 'Outer Diameter', 'Outer Diameter 1', 'Outer Diameter 2', 'Inner Diameter', 'Inner Diameter 1', 'Inner Diameter 2', 'Center Off', 'Center Off 1', 'Center Off 2', 'Profile', 'Profile 1', 'Profile 2', 'Cylindricity', 'Cylindricity 1', 'Cylindricity 2', 'Flatness', 'Flatness 1', 'Flatness 2', 'Parallelism', 'Parallelism 1', 'Parallelism 2', 'Perpendicularity', 'Perpendicularity 1', 'Perpendicularity 2', 'Perpendicularity 3', 'Perpendicularity 4', 'Weight', 'Weight 1', 'Weight 2', 'Coating Thickness', 'Coating Thickness 1', 'Coating Thickness 2', 'Plating Thickness', 'Plating Thickness 1', 'Plating Thickness 2', 'Total Flux', 'Appearance', 'Green paper', 'MSC']

const uomOptions = ['mm', 'g', 'µm', 'remarks', '[Wb.Ts.]']

const minMaxOnlyOptions = ['Yes', 'No', '-']

const samplingNameOptions = ['General Dim. / Sampling', 'Outer Diameter / Sampling', 'Inner Diameter / Sampling', 'R dimension / Sampling', 'R thickness / Sampling', 'Center off O.R / Sampling', 'Center off I.R / Sampling', 'Angularity / Sampling', 'Flatness / Sampling', 'Parallelism / Sampling', 'Perpendicularity / Sampling', 'Straightness / Sampling', 'Position / Sampling', 'Symmetricity / Sampling', 'Profile / Sampling', 'Coaxiality / Sampling', 'Cylindricity / Sampling', 'Roundness / Sampling', 'Runout / Sampling', 'Total Flux / Sampling', 'Open Flux / Sampling', 'Appearance / Sampling', 'Marking / Sampling', 'Thickness of Plating / Sampling', 'Thickness of HC Inorganic coating / Sampling', 'Thickness of Epoxy coating / Sampling', 'Thickness of EI coating / Sampling', 'Adhesion intensity of coating film / Sampling', 'Chamfer / Sampling', 'Weight / Sampling', 'Cleanliness / Sampling', 'I, AQL 1% (normal)', 'S-4, AQL 1% (normal)', 'S-3, AQL 1% (normal)', 'I, AQL 1% (reduced)', 'S-4, AQL 1% (reduced)', 'S-3, AQL 1% (reduced)', 'I, AQL 0.65%', '40 pcs / lot', '30 pcs / lot', '20 pcs / lot', '13 pcs / lot', '10 pcs / lot', '7 pcs / lot', '5 pcs / lot', '3 pcs / lot', '2 pcs / lot', '1 pcs / lot']

const initialNewDim = {
  name: 'Length',
  min: '',
  max: '',
  size: '',
  uom: 'mm',
  tol: '',
  minOnly: 'No',
  maxOnly: 'No',
  eq: '',
  samp: 'I, AQL 1% (reduced)',
  type: '',
  sampSize: '',
  r1: '',
  r2: '',
  r3: '',
  r4: '',
  r5: '',
  remarks: ''
}

const newDim = ref({ ...initialNewDim })

const isEditingSub = ref(false)
const editingIdx = ref(-1)

const openAddModal = () => {
  newDim.value = { ...initialNewDim }
  isEditingSub.value = false
  showAddModal.value = true
}

const openEditModal = (idx) => {
  editingIdx.value = idx
  newDim.value = { ...dimensions.value[idx] }
  isEditingSub.value = true
  showAddModal.value = true
}

const addDimension = () => {
  // Check empty decimal values and convert to "-"
  const formatField = (val) => (val === '' || val === null) ? '-' : val;

  const formattedDim = {
    name: newDim.value.name,
    min: formatField(newDim.value.min),
    max: formatField(newDim.value.max),
    size: formatField(newDim.value.size),
    uom: newDim.value.uom,
    tol: formatField(newDim.value.tol),
    minOnly: newDim.value.minOnly,
    maxOnly: newDim.value.maxOnly,
    eq: formatField(newDim.value.eq),
    samp: newDim.value.samp,
    type: formatField(newDim.value.type),
    sampSize: formatField(newDim.value.sampSize),
    r1: formatField(newDim.value.r1),
    r2: formatField(newDim.value.r2),
    r3: formatField(newDim.value.r3),
    r4: formatField(newDim.value.r4),
    r5: formatField(newDim.value.r5),
    remarks: formatField(newDim.value.remarks)
  }

  dimensions.value.push(formattedDim)
  showAddModal.value = false
}

const isDimensionListOpen = ref(false)

const toggleDimensionList = () => {
  isDimensionListOpen.value = !isDimensionListOpen.value
}

const editingCell = ref({ rowIdx: -1, field: '' })
const originalValue = ref('')

const startEdit = (rowIdx, field) => {
  editingCell.value = { rowIdx, field }
  originalValue.value = dimensions.value[rowIdx][field]
}

const saveEdit = () => {
  editingCell.value = { rowIdx: -1, field: '' }
}

const cancelEdit = () => {
  if (editingCell.value.rowIdx !== -1) {
    dimensions.value[editingCell.value.rowIdx][editingCell.value.field] = originalValue.value
    editingCell.value = { rowIdx: -1, field: '' }
  }
}

const handleGlobalClick = (e) => {
  if (editingCell.value.rowIdx === -1) return
  // If click is outside both the edit box and the triggering text, cancel
  if (!e.target.closest('.inline-edit') && !e.target.closest('.editable-text')) {
    cancelEdit()
  }
}

onMounted(() => {
  window.addEventListener('click', handleGlobalClick)
})

onUnmounted(() => {
  window.removeEventListener('click', handleGlobalClick)
})

const showDeleteConfirm = ref(false)
const rowToDeleteIdx = ref(-1)

const removeDimension = (rowIdx) => {
  rowToDeleteIdx.value = rowIdx
  showDeleteConfirm.value = true
}

const confirmRemove = () => {
  if (rowToDeleteIdx.value !== -1) {
    dimensions.value.splice(rowToDeleteIdx.value, 1)
  }
  showDeleteConfirm.value = false
  rowToDeleteIdx.value = -1
}

const cancelRemove = () => {
  showDeleteConfirm.value = false
  rowToDeleteIdx.value = -1
}
const goBack = () => {
  emit('back')
}
</script>

<template>
  <div class="view-panel">
    
    <div :class="['top-record-box', { 'modal-layout': isModal }]">
      <!-- Box 1 Header: Breadcrumbs -->
      <div v-if="!isModal" class="sub-header box-header">
        <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        <span class="breadcrumb" style="font-size: 14px;">
          <a href="#" class="item-link" @click.prevent="goBack" style="font-weight: bold; color: #0000EE;">PRODUCT RECORDS</a> 
          > <span style="font-weight: normal; color: #333;">{{ isCreating ? 'Create New' : productCode }}</span>
        </span>
      </div>

      <div class="top-actions">
        <template v-if="!isEditing">
          <button class="btn btn-primary" @click="startEditing">EDIT</button>
          <button class="btn btn-secondary" @click="emit('back')">Cancel</button>
        </template>
        <template v-else>
          <button class="btn btn-primary" @click="saveProduct">SAVE</button>
          <button class="btn btn-secondary" @click="cancelProductEdit">Cancel</button>
        </template>
      </div>

      <div class="form-wrapper">
        <!-- Product Details Section -->
        <fieldset class="fsMargin">
          <legend><b>Product Details</b></legend>
          <table border="0" style="width: 100%; table-layout: fixed;">
            <tbody>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Product Code</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isCreating" class="field-value">{{ productCode }}</div>
                  <input v-else type="text" v-model="localProductCode" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Product Type</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ productType }}</div>
                  <select v-else v-model="tempProductType" class="form-select" style="width: 80.4%; height: 22px;">
                    <option v-for="opt in typeOptions" :key="opt" :value="opt">{{ opt }}</option>
                  </select>
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Model Name</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ modelName }}</div>
                  <input v-else type="text" v-model="tempModelName" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Magnetization</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ magnetization }}</div>
                  <select v-else v-model="tempMagnetization" class="form-select" style="width: 80.4%; height: 22px;">
                    <option v-for="opt in magnetizationOptions" :key="opt" :value="opt">{{ opt }}</option>
                  </select>
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Material Code/ RM</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ materialCode }}</div>
                  <input v-else type="text" v-model="tempMaterialCode" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Product Dimension</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ productDimension }}</div>
                  <input v-else type="text" v-model="tempProductDimension" class="edit-select" />
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Material Powder Type</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ materialPowderType }}</div>
                  <input v-else type="text" v-model="tempMaterialPowderType" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Tolerance</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ tolerance }}</div>
                  <input v-else type="text" v-model="tempTolerance" class="edit-select" />
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Material Grade</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ materialGrade }}</div>
                  <input v-else type="text" v-model="tempMaterialGrade" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Product Weight</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ productWeight }}</div>
                  <input v-else type="text" v-model="tempProductWeight" class="edit-select" />
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Magnetic Direction</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ magneticDirection }}</div>
                  <input v-else type="text" v-model="tempMagneticDirection" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Customer Weight (Min Spec)</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ customerWeightMinSpec }}</div>
                  <input v-else type="text" v-model="tempCustomerWeightMinSpec" class="edit-select" />
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Marking</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ marking }}</div>
                  <input v-else type="text" v-model="tempMarking" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Customer's Dwg No./ Part No.</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ customerDwgNo }}</div>
                  <input v-else type="text" v-model="tempCustomerDwgNo" class="edit-select" />
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Bending Strength (Min)</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ bendingStrengthMin }}</div>
                  <input v-else type="text" v-model="tempBendingStrengthMin" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Total Flux (Min)</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ totalFluxMin }}</div>
                  <input v-else type="text" v-model="tempTotalFluxMin" class="edit-select" />
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Customer Weight (Min)</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ customerWeightMin }}</div>
                  <input v-else type="text" v-model="tempCustomerWeightMin" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Total Flux (Max)</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ totalFluxMax }}</div>
                  <input v-else type="text" v-model="tempTotalFluxMax" class="edit-select" />
                </td>
              </tr>
            </tbody>
          </table>
        </fieldset>

        <!-- Magnetic Properties (CGS) Section -->
        <fieldset class="fsMargin">
          <legend><b>Magnetic Properties (CGS)</b></legend>
          <table border="0" style="width: 100%; table-layout: fixed;">
            <tbody>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BR (Min)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ brMinCgs }}</div>
                  <input v-else type="text" v-model="tempBrMinCgs" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BR (Max)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ brMaxCgs }}</div>
                  <input v-else type="text" v-model="tempBrMaxCgs" class="edit-select" />
                </td>
                
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">IHC (Min)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ ihcMinCgs }}</div>
                  <input v-else type="text" v-model="tempIhcMinCgs" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">IHC (Max)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ ihcMaxCgs }}</div>
                  <input v-else type="text" v-model="tempIhcMaxCgs" class="edit-select" />
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BHC (Min)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ bhcMinCgs }}</div>
                  <input v-else type="text" v-model="tempBhcMinCgs" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BHC (Max)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ bhcMaxCgs }}</div>
                  <input v-else type="text" v-model="tempBhcMaxCgs" class="edit-select" />
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BHMax (Min)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ bhMaxMinCgs }}</div>
                  <input v-else type="text" v-model="tempBhMaxMinCgs" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BHMax (Max)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ bhMaxMaxCgs }}</div>
                  <input v-else type="text" v-model="tempBhMaxMaxCgs" class="edit-select" />
                </td>
              </tr>
            </tbody>
          </table>
        </fieldset>

        <!-- Magnetic Properties (SI) Section -->
        <fieldset class="fsMargin">
          <legend><b>Magnetic Properties (SI)</b></legend>
          <table border="0" style="width: 100%; table-layout: fixed;">
            <tbody>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BR (Min)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ brMinSi }}</div>
                  <input v-else type="text" v-model="tempBrMinSi" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BR (Max)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ brMaxSi }}</div>
                  <input v-else type="text" v-model="tempBrMaxSi" class="edit-select" />
                </td>
                
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">IHC (Min)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ ihcMinSi }}</div>
                  <input v-else type="text" v-model="tempIhcMinSi" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">IHC (Max)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ ihcMaxSi }}</div>
                  <input v-else type="text" v-model="tempIhcMaxSi" class="edit-select" />
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BHC (Min)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ bhcMinSi }}</div>
                  <input v-else type="text" v-model="tempBhcMinSi" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BHC (Max)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ bhcMaxSi }}</div>
                  <input v-else type="text" v-model="tempBhcMaxSi" class="edit-select" />
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BHMax (Min)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ bhMaxMinSi }}</div>
                  <input v-else type="text" v-model="tempBhMaxMinSi" class="edit-select" />
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;"><span class="labelTitle">BHMax (Max)</span></td>
                <td style="width: 33.3333%; height: 21px;">
                  <div v-if="!isEditing" class="field-value">{{ bhMaxMaxSi }}</div>
                  <input v-else type="text" v-model="tempBhMaxMaxSi" class="edit-select" />
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
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Created Date</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div class="field-value">16-March-2026 12:58:05 PM</div>
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Created By</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div class="field-value">qa-admin-p2</div>
                </td>
              </tr>
              <tr>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Last Updated Date</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div class="field-value">16-March-2026 12:59:47 PM</div>
                </td>
                <td class="labelBack" style="width: 16.6667%; height: 20px;">
                  <span class="labelTitle">Last Updated By</span>
                </td>
                <td style="width: 33.3333%; height: 21px;">
                  <div class="field-value">qa-tech-p2</div>
                </td>
              </tr>
            </tbody>
          </table>
        </fieldset>
      </div>
    </div>

    <!-- Dimension List Section -->
    <div v-if="!isCreating" class="sub-panel-wrapper">
      <div class="sub-panel-header" @click="toggleDimensionList">
        <span class="sub-panel-icon-btn">
          <svg style="vertical-align: middle; margin-right: 5px;" viewBox="0 0 24 24" width="14" height="14" fill="#333">
            <path v-if="!isDimensionListOpen" d="M10 17l5-5-5-5v10z"/>
            <path v-else d="M7 10l5 5 5-5H7z"/>
          </svg>
          <svg style="vertical-align: middle;" viewBox="0 0 24 24" width="14" height="14" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
        </span>
        <span style="font-weight: bold; font-size: 11px; text-transform: uppercase;">Dimension List</span>
      </div>
      
      <div v-if="isDimensionListOpen" class="sub-panel-body">
        <div class="sub-panel-inner-box">
          <div class="sub-actions" style="padding: 5px 15px; border-bottom: 1px solid #ddd;">
            <span class="text-link" @click="openAddModal">Add New Dimension</span>
          </div>

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
                  <th>Min & Max Tolerance Value (+-)</th>
                  <th>Min Only</th>
                  <th>Max Only</th>
                  <th>Equipment (1 to many)</th>
                  <th>Sampling Name</th>
                  <th>Type</th>
                  <th>Sampling Size</th>
                  <th>Rank 1 Size</th>
                  <th>Rank 2 Size</th>
                  <th>Rank 3 Size</th>
                  <th>Rank 4 Size</th>
                  <th>Rank 5 Size</th>
                  <th>Remarks</th>
                  <th>Remove</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(d, idx) in dimensions" :key="idx">
                  <td class="col-icon">
                    <svg viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;" @click="openEditModal(idx)">
                      <path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/>
                    </svg>
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
                      <button class="btn-save" @click.stop="saveEdit">Save</button>
                    </div>
                    <span v-else class="editable-text" @click="startEdit(idx, 'uom')">{{ d.uom }}</span>
                  </td>
                  <td>{{ d.tol }}</td>
                  <td>
                    <div v-if="editingCell.rowIdx === idx && editingCell.field === 'minOnly'" class="inline-edit">
                      <select v-model="d.minOnly" class="grid-select" @click.stop>
                        <option v-for="opt in minMaxOnlyOptions" :value="opt" :key="opt">{{ opt }}</option>
                      </select>
                      <button class="btn-save" @click.stop="saveEdit">Save</button>
                    </div>
                    <span v-else class="editable-text" @click="startEdit(idx, 'minOnly')">{{ d.minOnly }}</span>
                  </td>
                  <td>
                    <div v-if="editingCell.rowIdx === idx && editingCell.field === 'maxOnly'" class="inline-edit">
                      <select v-model="d.maxOnly" class="grid-select" @click.stop>
                        <option v-for="opt in minMaxOnlyOptions" :value="opt" :key="opt">{{ opt }}</option>
                      </select>
                      <button class="btn-save" @click.stop="saveEdit">Save</button>
                    </div>
                    <span v-else class="editable-text" @click="startEdit(idx, 'maxOnly')">{{ d.maxOnly }}</span>
                  </td>
                  <td>{{ d.eq }}</td>
                  <td>
                    <div v-if="editingCell.rowIdx === idx && editingCell.field === 'samp'" class="inline-edit">
                      <select v-model="d.samp" class="grid-select" style="max-width: 150px;" @click.stop>
                        <option v-for="opt in samplingNameOptions" :value="opt" :key="opt">{{ opt }}</option>
                      </select>
                      <button class="btn-save" @click.stop="saveEdit">Save</button>
                    </div>
                    <span v-else class="editable-text" @click="startEdit(idx, 'samp')">{{ d.samp }}</span>
                  </td>
                  <td>{{ d.type }}</td>
                  <td>{{ d.sampSize }}</td>
                  <td>{{ d.r1 }}</td>
                  <td>{{ d.r2 }}</td>
                  <td>{{ d.r3 }}</td>
                  <td>{{ d.r4 }}</td>
                  <td>{{ d.r5 }}</td>
                  <td>{{ d.remarks }}</td>
                  <td style="text-align: center;">
                    <button class="btn-remove" @click="removeDimension(idx)">
                      <svg viewBox="0 0 24 24" width="12" height="12" fill="#d9534f" style="margin-right: 4px; vertical-align: middle;">
                        <path d="M6 19c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7H6v12zM19 4h-3.5l-1-1h-5l-1 1H5v2h14V4z"/>
                      </svg>
                      Remove
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- Pagination / Status Bar -->
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
              <span class="display-text" style="color: #444; margin-left:10px;">Displaying 1 to {{ dimensions.length }} of {{ dimensions.length }} items</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- pop up modal matching the example photo -->
    <div class="modal-overlay" v-if="showAddModal">
      <div class="modal-window">
        <!-- Sub Header identical to app style -->
        <div class="sub-header modal-sub-header">
          <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
          <span style="font-weight: bold; color: black;">{{ isEditingSub ? 'EDIT DIMENSION' : 'DIMENSION MANAGEMENT' }}</span>
        </div>
        
        <div class="panel modal-panel">
          <div class="top-actions" style="padding: 15px 0 10px 0; background-color: transparent;">
            <button class="btn btn-primary" @click="addDimension">{{ isEditingSub ? 'SAVE' : 'ADD' }}</button>
            <button class="btn btn-secondary" @click="showAddModal = false">Cancel</button>
          </div>

          <div class="section-container" style="margin: 0; box-shadow: none;">
            <div class="section-title" style="background-color: transparent; padding-left: 0;">Dimension Details</div>
            <div class="info-grid">
              
              <div class="grid-col">
                <div class="grid-row">
                  <div class="grid-label modal-label">Name</div>
                  <div class="grid-value modal-value">
                    <select class="form-input" v-model="newDim.name">
                      <option v-for="opt in nameOptions" :value="opt" :key="opt">{{opt}}</option>
                    </select>
                  </div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Min Spec</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.min" /></div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Max Spec</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.max" /></div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Spec Size</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.size" /></div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">UOM</div>
                  <div class="grid-value modal-value">
                    <select class="form-input" v-model="newDim.uom">
                      <option v-for="opt in uomOptions" :value="opt" :key="opt">{{opt}}</option>
                    </select>
                  </div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Min & Max Tolerance Value (+-)</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.tol" /></div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Min Only</div>
                  <div class="grid-value modal-value">
                    <select class="form-input" v-model="newDim.minOnly">
                      <option v-for="opt in minMaxOnlyOptions" :value="opt" :key="opt">{{opt}}</option>
                    </select>
                  </div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Max Only</div>
                  <div class="grid-value modal-value">
                    <select class="form-input" v-model="newDim.maxOnly">
                      <option v-for="opt in minMaxOnlyOptions" :value="opt" :key="opt">{{opt}}</option>
                    </select>
                  </div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Equipment (1 to many)</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.eq" /></div>
                </div>
              </div>

              <div class="grid-col">
                <div class="grid-row">
                  <div class="grid-label modal-label">Sampling Name</div>
                  <div class="grid-value modal-value">
                    <select class="form-input" v-model="newDim.samp">
                      <option v-for="opt in samplingNameOptions" :value="opt" :key="opt">{{opt}}</option>
                    </select>
                  </div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Type</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.type" /></div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Sampling Size</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.sampSize" /></div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Rank 1 Size</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.r1" /></div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Rank 2 Size</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.r2" /></div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Rank 3 Size</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.r3" /></div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Rank 4 Size</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.r4" /></div>
                </div>
                <div class="grid-row">
                  <div class="grid-label modal-label">Rank 5 Size</div>
                  <div class="grid-value modal-value"><input type="text" class="form-input" v-model="newDim.r5" /></div>
                </div>
                <div class="grid-row" style="flex:1;">
                  <div class="grid-label modal-label">Remarks</div>
                  <div class="grid-value modal-value" style="height: 100%;">
                    <textarea class="form-input" style="height: 100%; min-height: 40px; resize: none;" v-model="newDim.remarks"></textarea>
                  </div>
                </div>
              </div>

            </div>
          </div>
          
          <div class="top-actions" style="padding: 15px 0 0 0; background-color: transparent;">
            <button class="btn btn-primary" @click="addDimension">ADD</button>
            <button class="btn btn-secondary" @click="showAddModal = false">Cancel</button>
          </div>

        </div>
      </div>
    </div>

    <!-- Custom Delete Confirmation Modal -->
    <div class="modal-overlay centered-overlay" v-if="showDeleteConfirm">
      <div class="confirm-banner">
        <div class="confirm-message">Are you sure to remove?</div>
        <div class="confirm-actions">
          <button class="btn-ok" @click="confirmRemove">OK</button>
          <button class="btn-cancel" @click="cancelRemove">Cancel</button>
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

.sub-header {
  background-color: #c7c7c7; /* Changed from gradient to solid light grey */
  padding: 6px 15px;
  font-weight: bold;
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 8px;
  border-bottom: 2px solid #fff;
  color: #333;
}

.box-header {
  background-color: #c7c7c7; /* Matched color to main list */
  border-bottom: 2px solid #c7c7c7;
  margin: -2px -2px 0 -2px;
}

.top-record-box {
  background-color: #fff;
  border: 2px solid #c7c7c7; /* Updated to match main list */
  margin: 0 15px 15px 15px;
  overflow: hidden;
}
.modal-layout {
  border: none !important;
  margin: 0 !important;
  box-shadow: none !important;
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

.btn-primary {
  background-color: #a43033;
  color: white;
}

.btn-primary:hover {
  background-color: #962c2e;
}

.btn-secondary {
  background-color: #a5a5a5;
  color: #fff;
}

.btn-secondary:hover {
  background-color: #888;
}

.btn-outline {
  background-color: transparent;
  color: #333;
  border: 1px solid #ccc;
  padding: 4px 10px;
}

.section-container {
  background-color: #fff;
  border: 1px solid #ddd;
  margin: 0 15px 15px 15px;
}

.form-wrapper {
  margin: 0;
  border: none;
  background-color: #fff;
}

.inner-section {
  border: none;
  margin: 0;
}

.inner-section + .inner-section {
  border-top: 10px solid #fff;
}

.fsMargin {
  margin: 10px 15px 12px 15px;
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
.labelTitle {
  font-size: 13px;
  color: black;
  font-family: Arial, Helvetica, sans-serif;
}
.field-value {
  padding-left: 12px;
  font-size: 13px;
  color: #333;
}

.inner-section {
  border: none;
  margin: 0;
}

.inner-section + .inner-section {
  border-top: 10px solid #fff;
}

/* Sub-panel styling */
.sub-panel-wrapper {
  margin: 0 15px 15px 15px;
}

.sub-panel-header {
  background-color: #c7c7c7; /* Changed to match central style */
  padding: 6px 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  border: 2px solid #c7c7c7; /* Thicker and softer color */
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
  margin: 0 15px 10px 15px; /* Matched to fieldset's horizontal margin */
}

.info-grid {
  display: flex;
  padding: 0;
  background-color: #f6f6f6;
}

.info-grid.half .grid-row {
  width: 50%;
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

.form-select {
  padding: 2px 4px;
  font-size: 12px;
  border: 1px solid #ccc;
}

.sub-actions {
  padding: 8px 15px;
  background-color: #fff;
}

.table-scroll-container {
  overflow-x: auto;
  border-top: 1px solid #eee;
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

.text-link {
  font-size: 13px;
  color: #333;
  cursor: pointer;
}

.text-link:hover {
  color: #8f3235;
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

.faint-col {
  color: #aaa;
}

/* Modal styles styling it exactly like the image given "CASE MANAGEMENT" page */
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
}

.form-input {
  width: 100%;
  height: 24px;
  border: 1px solid #ccc;
  padding: 2px 5px;
  font-size: 12px;
  box-sizing: border-box;
}

input.form-input[type="text"] {
  /* Let text fields take full width organically or specify max-width if desired */
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

.btn-remove {
  background-color: white;
  border: 1px solid #ccc;
  border-radius: 2px;
  padding: 1px 6px;
  font-size: 11px;
  cursor: pointer;
  color: #333;
  display: inline-flex;
  align-items: center;
}

.btn-remove:hover {
  background-color: #f9f9f9;
  border-color: #bbb;
}

/* Custom confirmation modal specific styles */
.centered-overlay {
  align-items: center;
  padding-top: 0;
}

.confirm-banner {
  background-color: #fff;
  width: 100%;
  max-width: 1200px;
  padding: 30px;
  text-align: center;
  box-shadow: 0 5px 25px rgba(0,0,0,0.5);
}

.confirm-message {
  font-size: 18px;
  color: #333;
  margin-bottom: 25px;
}

.confirm-actions {
  display: flex;
  justify-content: center;
  gap: 15px;
}

.btn-ok {
  background-color: #69d17d; /* Green matching photo */
  color: white;
  border: none;
  padding: 8px 45px;
  font-size: 14px;
  cursor: pointer;
  border-radius: 2px;
}

.btn-cancel {
  background-color: #a5a5a5; /* Grey matching photo */
  color: white;
  border: none;
  padding: 8px 45px;
  font-size: 14px;
  cursor: pointer;
  border-radius: 2px;
}

.btn-ok:hover {
  background-color: #5cb85c;
}

.btn-cancel:hover {
  background-color: #888;
}
</style>
