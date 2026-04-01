<script setup>
import { ref, computed } from 'vue'
import ProductView from './ProductView.vue'
import ValidationView from './ValidationView.vue'
import InspectionView from './InspectionView.vue'
import SamplingLevelView from './SamplingLevelView.vue'
import ErasureView from './ErasureView.vue'
import ErasureDetailView from './ErasureDetailView.vue'
import MagneticPropertiesView from './MagneticPropertiesView.vue'
import MagneticPropertiesDetailView from './MagneticPropertiesDetailView.vue'
import ShipmentTypeAView from './ShipmentTypeAView.vue'
import ShipmentTypeADetailView from './ShipmentTypeADetailView.vue'
import ShipmentTypeBView from './ShipmentTypeBView.vue'
import ShipmentTypeBDetailView from './ShipmentTypeBDetailView.vue'
import ReliabilityView from './ReliabilityView.vue'
import ReliabilityDetailView from './ReliabilityDetailView.vue'
import ValidationDetailView from './ValidationDetailView.vue'
import InspectionDetailView from './InspectionDetailView.vue'
import SamplingLevelDetailView from './SamplingLevelDetailView.vue'
import EditModal from './EditModal.vue'

const currentModule = ref('product')
const showEditModal = ref(false)
const editingRecord = ref(null)
const editingModule = ref('')

const products = ref([
  { id: 1, code: 'P000001', type: 'G', selected: false },
  { id: 2, code: 'P000002', type: 'Non-G', selected: false },
  { id: 3, code: 'P000003', type: 'G', selected: false },
  { id: 4, code: 'P000004', type: 'Non-G', selected: false },
  { id: 5, code: 'P000005', type: 'G', selected: false },
  { id: 6, code: 'P000006', type: 'Non-G', selected: false },
  { id: 7, code: 'P000007', type: 'G', selected: false },
  { id: 8, code: 'P000008', type: 'Non-G', selected: false },
])

const validationRecords = ref([
  { 
    formNo: 'F-VAL-001', revision: '01', dateIssued: '2026-03-24 10:30:15', title: 'Micrometer Calibration', 
    spec: 'ISO-17025', eq: 'Digital Micrometer', eqSerial: 'MG-99482', freq: 'Monthly',
    date: '2026-03-24 08:00:00', shift: 'Day', type: 'Type 1: Comparator/Micrometer', year: '2026', month: 'March',
    product: 'P000001', serialNo: 'SN-7788', vDate: '2026-03-24', vDue: '2026-04-24',
    selected: false 
  },
  { 
    formNo: 'F-VAL-002', revision: '02', dateIssued: '2026-03-23 14:45:00', title: 'Induction System Check', 
    spec: 'IEC-62305', eq: 'Induction Coil', eqSerial: 'IC-1122', freq: 'Quarterly',
    date: '2026-03-23 20:00:00', shift: 'Night', type: 'Type 2: Induction Check', year: '2026', month: 'March',
    product: 'P000002', serialNo: 'SN-9900', vDate: '2026-03-23', vDue: '2026-06-23',
    selected: false 
  },
  { 
    formNo: 'F-VAL-003', revision: '01', dateIssued: '2026-03-24 09:15:30', title: 'Pin Gauge Verification', 
    spec: 'ANSI/ASME B89', eq: 'Master Pin Set', eqSerial: 'PS-8844', freq: 'Weekly',
    date: '2026-03-22 08:30:00', shift: 'Day', type: 'Type 3: Pin Gauge', year: '2026', month: 'March',
    product: 'P000003', serialNo: 'SN-1122', vDate: '2026-03-22', vDue: '2026-03-29',
    selected: false 
  }
])
const selectedValidationRecord = ref(null)

const inspectionRecords = ref([
  { 
    productType: 'G', product: 'P000001', serialNo: 'SN-7788', revision: '01', 
    dwgNo: 'DWG-99482', lotNo: 'LOT-A123', cDate: '2026-03-24', cLine: 'Line A',
    paNo: 'PA-882', jpnLot: 'JPN-990', mMethod: 'Max Outlier', firstRun: 'Yes',
    remarks: 'Internal Check', creationDate: '2026-03-24 10:30:15', createdBy: 'qa-admin',
    updatedDate: '2026-03-24 10:30:15', updatedBy: 'qa-admin', finalResult: 'OK',
    selected: false 
  },
  { 
    productType: 'Non-G', product: 'P000002', serialNo: 'SN-9900', revision: '02', 
    dwgNo: 'DWG-12345', lotNo: 'LOT-B456', cDate: '2026-03-25', cLine: 'Line B',
    paNo: 'PA-900', jpnLot: 'JPN-1001', mMethod: 'Min Outlier', firstRun: 'No',
    remarks: 'Customer Request', creationDate: '2026-03-25 08:45:00', createdBy: 'qa-admin',
    updatedDate: '2026-03-25 09:15:30', updatedBy: 'qa-admin', finalResult: 'OK',
    selected: false 
  },
  { 
    productType: 'G', product: 'P000003', serialNo: 'SN-1122', revision: '01', 
    dwgNo: 'DWG-88888', lotNo: 'LOT-C789', cDate: '2026-03-26', cLine: 'Line A',
    paNo: 'PA-885', jpnLot: 'JPN-995', mMethod: 'Average', firstRun: 'Yes',
    remarks: 'Re-inspection needed', creationDate: '2026-03-26 14:00:10', createdBy: 'qa-admin',
    updatedDate: '2026-03-26 14:00:10', updatedBy: 'qa-admin', finalResult: 'NG',
    selected: false 
  }
])
const selectedInspectionRecord = ref(null)

const samplingLevelRecords = ref([
 { 
    id: 1, name: 'I, AQL 1% (normal)', type: 'Normal', qty: '500', sSize: '5', r1: '5', r2: '5', r3: '15', r4: '15', r5: '15', selected: false,
    creationDate: '16-March-2026 12:58:05 PM', createdBy: 'qa-admin-p2', updatedDate: '16-March-2026 12:59:47 PM', updatedBy: 'qa-tech-p2' 
 },
 { 
    id: 2, name: 'II, AQL 0.65% (normal)', type: 'Normal', qty: '1200', sSize: '8', r1: '8', r2: '8', r3: '20', r4: '20', r5: '20', selected: false,
    creationDate: '17-March-2026 09:30:00 AM', createdBy: 'qa-admin-p2', updatedDate: '17-March-2026 10:15:22 AM', updatedBy: 'qa-tech-p2' 
 },
 { 
    id: 3, name: 'III, AQL 2.5% (tightened)', type: 'Tightened', qty: '2500', sSize: '13', r1: '13', r2: '13', r3: '32', r4: '32', r5: '32', selected: false,
    creationDate: '18-March-2026 02:00:15 PM', createdBy: 'qa-admin-p2', updatedDate: '18-March-2026 02:45:10 PM', updatedBy: 'qa-tech-p2' 
 },
 { 
    id: 4, name: 'S-3, AQL 4.0% (reduced)', type: 'Reduced', qty: '3500', sSize: '20', r1: '20', r2: '20', r3: '50', r4: '50', r5: '50', selected: false,
    creationDate: '19-March-2026 11:20:45 AM', createdBy: 'qa-admin-p2', updatedDate: '19-March-2026 11:55:30 AM', updatedBy: 'qa-tech-p2' 
 }
])
const selectedSamplingLevelRecord = ref(null)

const filterOptions = [
  { value: '', label: '--Please Select One--' },
  { value: 'code', label: 'Product ID/Code' },
  { value: 'type', label: 'Product Type' },
]

const selectedFilter = ref('')
const searchQuery = ref('')
const actionsDropdownOpen = ref(false)

const filteredProducts = computed(() => {
  if (!searchQuery.value) return products.value
  return products.value.filter(p => {
    if (selectedFilter.value === 'code') {
      return p.code.toLowerCase().includes(searchQuery.value.toLowerCase())
    } else if (selectedFilter.value === 'type') {
      return p.type.toLowerCase().includes(searchQuery.value.toLowerCase())
    } else {
      return p.code.toLowerCase().includes(searchQuery.value.toLowerCase()) || 
             p.type.toLowerCase().includes(searchQuery.value.toLowerCase())
    }
  })
})

const numSelected = computed(() => products.value.filter(p => p.selected).length)

const selectAll = ref(false)
const toggleSelectAll = () => {
  filteredProducts.value.forEach(p => p.selected = selectAll.value)
}

const toggleActionDropdown = () => {
  actionsDropdownOpen.value = !actionsDropdownOpen.value
}

const deleteSelected = () => {
  const selectedCount = products.value.filter(p => p.selected).length
  if (selectedCount === 0) return
  
  if (confirm(`Are you sure you want to delete ${selectedCount} selected product(s)?`)) {
    products.value = products.value.filter(p => !p.selected)
    actionsDropdownOpen.value = false
  }
}

const createNew = () => {
  selectedProductCode.value = ''
  currentView.value = 'create'
}

const currentView = ref('list')
const selectedProductCode = ref('')

const viewProduct = (code) => {
  selectedProductCode.value = code
  currentView.value = 'view'
}

const backToList = () => {
  currentView.value = 'list'
  selectedProductCode.value = ''
  selectedErasureRecord.value = null
}

const editProduct = (code) => {
  alert(`Editing product: ${code}`)
}

// ERASURE MODULE STATE
const erasureRecords = ref([
  { 
    id: 'ER-001', 
    date: '2026-03-27', 
    time: '10:00', 
    shift: 'Day', 
    type: 'Environment Washing', 
    area: 'Clean Room', 
    results: 'Pass', 
    performer: 'John Doe', 
    confirmer: 'Jane Smith', 
    onePerDay: 'Completed', 
    twoPerShift: 'Completed', 
    spec: 'Standard', 
    method: 'Chemical', 
    remarks: 'N/A', 
    createdTs: '27-March-2026 09:00:00 AM', 
    createdBy: 'qa-admin', 
    updatedTs: '27-March-2026 11:00:00 AM', 
    updatedBy: 'qa-tech', 
    selected: false 
  },
  { 
    id: 'ER-002', 
    date: '2026-03-28', 
    time: '22:30', 
    shift: 'Night', 
    type: 'Outgoing (Magnets)', 
    area: 'Production Line 1', 
    results: 'Pass', 
    performer: 'Mr. Chen', 
    confirmer: 'Mrs. Lee', 
    onePerDay: 'Completed', 
    twoPerShift: 'Completed', 
    spec: 'Level 2', 
    method: 'Ultrasonic', 
    remarks: 'Routine check', 
    createdTs: '28-March-2026 10:00:00 PM', 
    createdBy: 'qa-admin', 
    updatedTs: '28-March-2026 11:30:00 PM', 
    updatedBy: 'qa-tech', 
    selected: false 
  },
  { 
    id: 'ER-003', 
    date: '2026-03-29', 
    time: '14:15', 
    shift: 'Day', 
    type: 'Thin Model', 
    area: 'Testing Lab', 
    results: 'Pass', 
    performer: 'Alice Brown', 
    confirmer: 'Bob Wilson', 
    onePerDay: 'Completed', 
    twoPerShift: 'Pending', 
    spec: 'Standard', 
    method: 'Magnetic Field', 
    remarks: 'New batch', 
    createdTs: '29-March-2026 02:15:00 PM', 
    createdBy: 'qa-admin', 
    updatedTs: '29-March-2026 03:00:00 PM', 
    updatedBy: 'qa-tech', 
    selected: false 
  },
  { 
    id: 'ER-004', 
    date: '2026-03-30', 
    time: '01:00', 
    shift: 'Night', 
    type: 'Environment Washing', 
    area: 'Storage Room A', 
    results: 'Fail', 
    performer: 'Chris Evans', 
    confirmer: 'Jane Smith', 
    onePerDay: 'Incomplete', 
    twoPerShift: 'Pending', 
    spec: 'High Precision', 
    method: 'Ion Gauge', 
    remarks: 'Re-test required', 
    createdTs: '30-March-2026 01:00:00 AM', 
    createdBy: 'qa-admin', 
    updatedTs: '30-March-2026 01:30:00 AM', 
    updatedBy: 'qa-tech', 
    selected: false 
  }
])
const selectedErasureRecord = ref(null)
const handleOpenErasureDetail = (record) => {
  selectedErasureRecord.value = record
  currentView.value = 'view'
}
const handleOpenErasureCreate = () => {
  selectedErasureRecord.value = { id: 'ER-' + (erasureRecords.value.length + 1).toString().padStart(3, '0') }
  currentView.value = 'create'
}
const handleSaveErasure = (record) => {
  const idx = erasureRecords.value.findIndex(r => r.id === record.id)
  if (idx !== -1) {
    erasureRecords.value[idx] = record
  } else {
    erasureRecords.value.push({ ...record, selected: false, createdBy: 'qa-admin', createdTs: new Date().toLocaleString() })
  }
  currentView.value = 'list'
}
// MAGNETIC PROPERTIES MODULE STATE
const magPropRecords = ref([
  { id: 1, code: 'MAG001', type: 'G', selected: false },
  { id: 2, code: 'MAG002', type: 'Non-G', selected: false },
])
const selectedMagPropRecord = ref(null)

const handleOpenMagPropDetail = (record) => {
  selectedMagPropRecord.value = record
  currentView.value = 'view'
}
const handleOpenMagPropCreate = () => {
  selectedMagPropRecord.value = { code: '', type: 'G' }
  currentView.value = 'create'
}
const handleSaveMagProp = (record) => {
  const idx = magPropRecords.value.findIndex(r => r.id === record.id)
  if (idx !== -1) {
    magPropRecords.value[idx] = record
  } else {
    magPropRecords.value.push({ ...record, id: magPropRecords.value.length + 1, selected: false })
  }
  currentView.value = 'list'
}

// SHIPMENT TYPE A MODULE STATE
const shipmentARecords = ref([
  { 
    reportId: 'REP-A-001', 
    issuedDate: '2026-03-30', 
    customer: 'TOYOTA MOTOR', 
    material: 'Neodymium', 
    codeNo: 'N52-A1', 
    customerPo: 'PO-TYT-9988', 
    customerDwg: 'DWG-9988-X', 
    ourPo: 'OUR-PO-001', 
    quantity: '500 PCS', 
    unitWeight: '12.5g', 
    magThrough: '3.0mm', 
    magnetization: 'MAGNETIZED', 
    marking: 'Yes', 
    dimension: '10x10x5mm', 
    notes: 'Sample check pass', 
    judgement: 'Pass', 
    approvedBy: 'Mr. Tanaka', 
    checkedBy: 'Mr. Sato', 
    selected: false 
  }
])
const selectedShipmentARecord = ref(null)

const handleOpenShipmentADetail = (record) => {
  selectedShipmentARecord.value = record
  currentView.value = 'view'
}
const handleOpenShipmentACreate = () => {
  selectedShipmentARecord.value = { reportId: '', magnetization: 'MAGNETIZED', marking: 'Yes' }
  currentView.value = 'create'
}
const handleSaveShipmentA = (record) => {
  const idx = shipmentARecords.value.findIndex(r => r.reportId === record.reportId)
  if (idx !== -1) {
    shipmentARecords.value[idx] = record
  } else {
    shipmentARecords.value.push({ ...record, selected: false })
  }
}

// SHIPMENT TYPE B MODULE STATE
const shipmentBRecords = ref([
  { 
    reportId: 'REP-B-001', 
    issuedDate: '2026-03-30', 
    customer: 'HONDA MOTOR', 
    material: 'Samarium Cobalt', 
    codeNo: 'SmCo-B1', 
    customerPo: 'PO-HND-1234', 
    customerDwg: 'DWG-1234-Y', 
    ourPo: 'OUR-PO-002', 
    quantity: '1000 PCS', 
    unitWeight: '8.2g', 
    magThrough: 'N/A', 
    magnetization: 'UN-MAGNETIZED', 
    marking: 'No', 
    dimension: '5x5x2mm', 
    notes: 'Bulk order', 
    judgement: 'Pass', 
    approvedBy: 'Mr. Tanaka', 
    checkedBy: 'Mr. Sato', 
    selected: false 
  }
])
const selectedShipmentBRecord = ref(null)

const handleOpenShipmentBDetail = (record) => {
  selectedShipmentBRecord.value = record
  currentView.value = 'view'
}
const handleOpenShipmentBCreate = () => {
  selectedShipmentBRecord.value = { reportId: '', magnetization: 'UN-MAGNETIZED', marking: 'No' }
  currentView.value = 'create'
}
const handleSaveShipmentB = (record) => {
  const idx = shipmentBRecords.value.findIndex(r => r.reportId === record.reportId)
  if (idx !== -1) {
    shipmentBRecords.value[idx] = record
  } else {
    shipmentBRecords.value.push({ ...record, selected: false })
  }
  currentView.value = 'list'
}

// RELIABILITY MODULE STATE
const reliabilityRecords = ref([
  { 
    id: 'REL-0001',
    testType: 'Pressure Cooker',
    testCondition: '120ºc x 2atm x 48hrs',
    samplingFrequency: '5 pcs / line / day / any model',
    criteria: 'No harmful change such as worsened stain, corrosion, rust, or swelling before and after the test',
    platingDate: '2026-03-31',
    preparedBy: 'Admin',
    checkedBy: 'Manager',
    approvedBy: 'Director',
    selected: false
  },
  { 
    id: 'REL-0002',
    testType: 'Pull Test',
    testCondition: 'Adhesive Ratio : 5 (AV138) : 2 (HV998)\nOven Curing : 4 hours (40oC)\nCooling : 15 minutes (room temperature)',
    samplingFrequency: '3 pcs / line / day / any model',
    criteria: '≥ 100 kg.f/cm2',
    platingDate: '2026-03-31',
    preparedBy: 'Admin',
    checkedBy: 'Manager',
    approvedBy: 'Director',
    selected: false
  },
  { 
    id: 'REL-0003',
    testType: 'Quench Test',
    testCondition: 'i) 180ºc x 1hr (HSA)\nii) 250ºc x 1hr (WDA)',
    samplingFrequency: '1 pc / model / day / any line',
    criteria: '*No blister, crack, lifting of plating after quench\n*No Ni peeled off by Tape Test',
    platingDate: '2026-03-31',
    preparedBy: 'Admin',
    checkedBy: 'Manager',
    approvedBy: 'Director',
    selected: false
  },
  { 
    id: 'REL-0004',
    testType: 'Thermal Demagnetisation Test',
    testCondition: '-',
    samplingFrequency: '-',
    criteria: '-',
    platingDate: '2026-03-31',
    preparedBy: 'Admin',
    checkedBy: 'Manager',
    approvedBy: 'Director',
    selected: false
  },
  { 
    id: 'REL-0005',
    testType: 'Routine Reliability Test',
    testCondition: '-',
    samplingFrequency: '-',
    criteria: '-',
    platingDate: '2026-03-31',
    preparedBy: 'Admin',
    checkedBy: 'Manager',
    approvedBy: 'Director',
    selected: false
  }
])
const selectedReliabilityRecord = ref(null)

const handleOpenReliabilityDetail = (record) => {
  selectedReliabilityRecord.value = record
  currentView.value = 'view'
}
const handleOpenReliabilityCreate = () => {
  selectedReliabilityRecord.value = { id: '', testType: 'Pressure Cooker' }
  currentView.value = 'create'
}
const handleSaveReliability = (record) => {
  const idx = reliabilityRecords.value.findIndex(r => r.id === record.id)
  if (idx !== -1) {
    reliabilityRecords.value[idx] = record
  } else {
    reliabilityRecords.value.push({ ...record, selected: false })
  }
  currentView.value = 'list'
  showEditModal.value = false
}

const handleOpenEdit = (moduleName, record) => {
  editingModule.value = moduleName
  editingRecord.value = { ...record }
  showEditModal.value = true
}

const handleSaveEdit = (updated) => {
  // Find which list to update based on editingModule
  if (editingModule.value === 'erasure') handleSaveErasure(updated)
  else if (editingModule.value === 'magProp') handleSaveMagProp(updated)
  else if (editingModule.value === 'shipmentA') handleSaveShipmentA(updated)
  else if (editingModule.value === 'shipmentB') handleSaveShipmentB(updated)
  else if (editingModule.value === 'reliability') handleSaveReliability(updated)
  else if (editingModule.value === 'product') {
    const idx = products.value.findIndex(p => p.code === updated.code)
    if (idx !== -1) products.value[idx] = updated
  }
  else if (editingModule.value === 'validation') handleSaveValidation(updated)
  else if (editingModule.value === 'inspection') handleSaveInspection(updated)
  else if (editingModule.value === 'samplingLevel') handleSaveSamplingLevel(updated)
  showEditModal.value = false
}

const handleOpenValidationDetail = (record) => {
  selectedValidationRecord.value = record
  currentView.value = 'view'
}
const handleOpenValidationCreate = () => {
  selectedValidationRecord.value = { formNo: 'New', title: '' }
  currentView.value = 'create'
}
const handleSaveValidation = (record) => {
  const idx = validationRecords.value.findIndex(r => r.formNo === record.formNo)
  if (idx !== -1) validationRecords.value[idx] = record
  else validationRecords.value.push({ ...record, selected: false })
  currentView.value = 'list'
}

const handleOpenInspectionDetail = (record) => {
  selectedInspectionRecord.value = record
  currentView.value = 'view'
}
const handleOpenInspectionCreate = () => {
  selectedInspectionRecord.value = { product: 'New', lotNo: '' }
  currentView.value = 'create'
}
const handleSaveInspection = (record) => {
  const idx = inspectionRecords.value.findIndex(r => r.product === record.product && r.lotNo === record.lotNo)
  if (idx !== -1) inspectionRecords.value[idx] = record
  else inspectionRecords.value.push({ ...record, selected: false })
  currentView.value = 'list'
}

const handleOpenSamplingLevelDetail = (record) => {
  selectedSamplingLevelRecord.value = record
  currentView.value = 'view'
}
const handleOpenSamplingLevelCreate = () => {
  selectedSamplingLevelRecord.value = { id: Date.now(), name: 'New' }
  currentView.value = 'create'
}
const handleSaveSamplingLevel = (record) => {
  const idx = samplingLevelRecords.value.findIndex(r => r.id === record.id)
  if (idx !== -1) samplingLevelRecords.value[idx] = record
  else samplingLevelRecords.value.push({ ...record, selected: false })
  currentView.value = 'list'
}

/* Page-level Edit Handlers (New) */
const handleOpenProductEdit = (record) => {
  selectedProductCode.value = record.code
  currentView.value = 'edit'
}
const handleOpenValidationEdit = (record) => {
  selectedValidationRecord.value = record
  currentView.value = 'edit'
}
const handleOpenInspectionEdit = (record) => {
  selectedInspectionRecord.value = record
  currentView.value = 'edit'
}
const handleOpenSamplingLevelEdit = (record) => {
  selectedSamplingLevelRecord.value = record
  currentView.value = 'edit'
}
const handleOpenErasureEdit = (record) => {
  selectedErasureRecord.value = record
  currentView.value = 'edit'
}
const handleOpenReliabilityEdit = (record) => {
  selectedReliabilityRecord.value = record
  currentView.value = 'edit'
}
const handleOpenMagPropEdit = (record) => {
  selectedMagPropRecord.value = record
  currentView.value = 'edit'
}
const handleOpenShipmentAEdit = (record) => {
  selectedShipmentARecord.value = record
  currentView.value = 'edit'
}
const handleOpenShipmentBEdit = (record) => {
  selectedShipmentBRecord.value = record
  currentView.value = 'edit'
}
</script>

<template>
  <div class="app-container">
    <header class="top-header">
      <div class="logo" style="font-size: 26px; color: #fff; font-family:'Lobster Two', cursive; letter-spacing:2px;">Calibration Management System</div>
      <div class="user-controls">
        <span class="username">qa-admin-p2</span>
        <div class="toggle-switch">
          <div class="toggle-knob"></div>
        </div>
        <div class="icon bell-icon">
          <svg viewBox="0 0 24 24" width="16" height="16" fill="white"><path d="M12 22c1.1 0 2-.9 2-2h-4c0 1.1.89 2 2 2zm6-6v-5c0-3.07-1.64-5.64-4.5-6.32V4c0-.83-.67-1.5-1.5-1.5s-1.5.67-1.5 1.5v.68C7.63 5.36 6 7.92 6 11v5l-2 2v1h16v-1l-2-2z"/></svg>
        </div>
        <div class="icon user-icon">
          <svg viewBox="0 0 24 24" width="16" height="16" fill="white"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 3c1.66 0 3 1.34 3 3s-1.34 3-3 3-3-1.34-3-3 1.34-3 3-3zm0 14.2c-2.5 0-4.71-1.28-6-3.22.03-1.99 4-3.08 6-3.08 1.99 0 5.97 1.09 6 3.08-1.29 1.94-3.5 3.22-6 3.22z"/></svg>
        </div>
      </div>
    </header>

    <nav class="main-nav">
      <a href="#" :class="{ active: currentModule === 'validation' }" @click.prevent="currentModule = 'validation'; currentView = 'list'">VALIDATION</a>
      <a href="#" :class="{ active: currentModule === 'inspection' }" @click.prevent="currentModule = 'inspection'; currentView = 'list'">INSPECTION</a>
      <a href="#" :class="{ active: currentModule === 'erasure' }" @click.prevent="currentModule = 'erasure'; currentView = 'list'">ERASURE</a>
      <a href="#" :class="{ active: currentModule === 'reliability' }" @click.prevent="currentModule = 'reliability'; currentView = 'list'">RELIABILITY</a>
      <a href="#" :class="{ active: currentModule === 'samplingLevel' }" @click.prevent="currentModule = 'samplingLevel'; currentView = 'list'">SAMPLING LEVEL</a>
      <a href="#" :class="{ active: currentModule === 'product' }" @click.prevent="currentModule = 'product'; currentView = 'list'">PRODUCT RECORDS</a>
      <div class="nav-item-dropdown">
        <a href="#">REPORT</a>
        <div class="dropdown-content">
          <a href="#" class="dropdown-item" @click.prevent="currentModule = 'shipmentA'; currentView = 'list'">Shipment Type A</a>
          <a href="#" class="dropdown-item" @click.prevent="currentModule = 'shipmentB'; currentView = 'list'">Shipment Type B</a>
        </div>
      </div>
      <a href="#" :class="{ active: currentModule === 'magProp' }" @click.prevent="currentModule = 'magProp'; currentView = 'list'">MAGNETIC PROPERTIES</a>
    </nav>
    <main class="content">
      <!-- VALIDATION MODULE -->
      <template v-if="currentModule === 'validation'">
        <ValidationView v-if="currentView === 'list'" :records="validationRecords" @open-create="handleOpenValidationCreate" @open-detail="handleOpenValidationDetail" @open-edit="handleOpenValidationEdit" />
        <ValidationDetailView v-else :record="selectedValidationRecord" :isCreating="currentView === 'create'" :isEditing="currentView === 'edit'" @back="backToList" @save="handleSaveValidation" />
      </template>

      <!-- INSPECTION MODULE -->
      <template v-else-if="currentModule === 'inspection'">
        <InspectionView v-if="currentView === 'list'" :records="inspectionRecords" @open-create="handleOpenInspectionCreate" @open-detail="handleOpenInspectionDetail" @open-edit="handleOpenInspectionEdit" />
        <InspectionDetailView v-else :record="selectedInspectionRecord" :isCreating="currentView === 'create'" :isEditing="currentView === 'edit'" @back="backToList" @save="handleSaveInspection" />
      </template>

      <!-- SAMPLING LEVEL MODULE -->
      <template v-else-if="currentModule === 'samplingLevel'">
        <SamplingLevelView v-if="currentView === 'list'" :records="samplingLevelRecords" @open-create="handleOpenSamplingLevelCreate" @open-detail="handleOpenSamplingLevelDetail" @open-edit="handleOpenSamplingLevelEdit" />
        <SamplingLevelDetailView v-else :record="selectedSamplingLevelRecord" :isCreating="currentView === 'create'" :isEditing="currentView === 'edit'" @back="backToList" @save="handleSaveSamplingLevel" />
      </template>

      <!-- ERASURE MODULE -->
      <template v-else-if="currentModule === 'erasure'">
        <ErasureView v-if="currentView === 'list'" :records="erasureRecords" @open-create="handleOpenErasureCreate" @open-detail="handleOpenErasureDetail" @open-edit="handleOpenErasureEdit" />
        <ErasureDetailView v-else :record="selectedErasureRecord" :isCreating="currentView === 'create'" :isEditing="currentView === 'edit'" @back="backToList" @save="handleSaveErasure" />
      </template>

      <!-- RELIABILITY MODULE -->
      <template v-else-if="currentModule === 'reliability'">
        <ReliabilityView v-if="currentView === 'list'" :records="reliabilityRecords" @open-create="handleOpenReliabilityCreate" @open-detail="handleOpenReliabilityDetail" @open-edit="handleOpenReliabilityEdit" />
        <ReliabilityDetailView v-else :record="selectedReliabilityRecord" :isCreating="currentView === 'create'" :isEditing="currentView === 'edit'" @back="backToList" @save="handleSaveReliability" />
      </template>

      <!-- MAGNETIC PROPERTIES MODULE -->
      <template v-else-if="currentModule === 'magProp'">
        <MagneticPropertiesView v-if="currentView === 'list'" :records="magPropRecords" @open-create="handleOpenMagPropCreate" @open-detail="handleOpenMagPropDetail" @open-edit="handleOpenMagPropEdit" />
        <MagneticPropertiesDetailView v-else :record="selectedMagPropRecord" :isCreating="currentView === 'create'" :isEditing="currentView === 'edit'" @back="backToList" @save="handleSaveMagProp" />
      </template>

      <!-- SHIPMENT TYPE A MODULE -->
      <template v-else-if="currentModule === 'shipmentA'">
        <ShipmentTypeAView v-if="currentView === 'list'" :records="shipmentARecords" @open-create="handleOpenShipmentACreate" @open-detail="handleOpenShipmentADetail" @open-edit="handleOpenShipmentAEdit" />
        <ShipmentTypeADetailView v-else :record="selectedShipmentARecord" :isCreating="currentView === 'create'" :isEditing="currentView === 'edit'" @back="backToList" @save="handleSaveShipmentA" />
      </template>

      <!-- SHIPMENT TYPE B MODULE -->
      <template v-else-if="currentModule === 'shipmentB'">
        <ShipmentTypeBView v-if="currentView === 'list'" :records="shipmentBRecords" @open-create="handleOpenShipmentBCreate" @open-detail="handleOpenShipmentBDetail" @open-edit="handleOpenShipmentBEdit" />
        <ShipmentTypeBDetailView v-else :record="selectedShipmentBRecord" :isCreating="currentView === 'create'" :isEditing="currentView === 'edit'" @back="backToList" @save="handleSaveShipmentB" />
      </template>

      <!-- PRODUCT RECORDS MODULE -->
      <template v-else-if="currentModule === 'product'">
        <div v-if="currentView === 'list'" class="top-record-box">
        <!-- Box Header -->
        <div class="sub-header box-header">
          <svg class="folder-svg" viewBox="0 0 24 24" width="16" height="16" fill="#666"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/></svg>
          <span style="font-size: 14px;">PRODUCT RECORDS</span>
        </div>

        <div class="panel list-panel">
          <div class="list-inner-box">
            <!-- Row 1: Actions -->
            <div class="action-row toolbar-row">
              <div class="action-bar no-margin">
                <div class="dropdown-container">
                  <span class="action-link has-dropdown" @click="toggleActionDropdown">
                    Actions
                  </span>
                  <div class="dropdown-menu" v-if="actionsDropdownOpen">
                    <div class="dropdown-item" @click="deleteSelected">Delete Selected</div>
                  </div>
                </div>
                <span class="action-link" @click="createNew">Create New</span>
                <span class="selected-text">Selected: {{ numSelected }}</span>
              </div>
            </div>

            <!-- Row 2: Search -->
            <div class="search-row toolbar-row">
              <div class="search-bar no-margin">
                <select class="search-select" v-model="selectedFilter">
                  <option v-for="opt in filterOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
                </select>
                <input type="text" class="search-input" v-model="searchQuery" style="margin-left: 5px;" />
                <button class="search-btn" style="margin-left: 5px;">SEARCH</button>
              </div>
            </div>

            <!-- Row 3: Table -->
            <div class="table-area">
              <table class="data-table">
                <thead>
                  <tr>
                    <th class="col-checkbox"><input type="checkbox" v-model="selectAll" @change="toggleSelectAll" /></th>
                    <th class="col-icon"></th>
                    <th>Product ID/Code</th>
                    <th>Product Type</th>
                  </tr>
                </thead>
                <tbody>
                  <tr 
                    v-for="item in filteredProducts" 
                    :key="item.id" 
                    :class="{ 'selected-row': item.selected }"
                  >
                    <td class="col-checkbox"><input type="checkbox" v-model="item.selected" /></td>
                    <td class="col-icon">
                      <svg @click="handleOpenProductEdit(item)" viewBox="0 0 24 24" width="14" height="14" fill="#666" style="cursor: pointer;"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
                    </td>
                    <td><a href="#" class="item-link" @click.prevent="viewProduct(item.code)">{{ item.code }}</a></td>
                    <td>{{ item.type }}</td>
                  </tr>
                  <tr v-if="filteredProducts.length === 0">
                    <td colspan="4" style="text-align: center; padding: 20px; color: #666;">No products found.</td>
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
                <span class="display-text" style="color: #444; margin-left:10px;">Displaying 1 to {{ filteredProducts.length }} of {{ filteredProducts.length }} items</span>
              </div>
            </div>
          </div>
        </div>
      </div>
        <ProductView v-else :productCode="selectedProductCode" :isCreating="currentView === 'create'" :isEditing="currentView === 'edit'" @back="backToList" @save="handleSaveEdit" />
      </template>
    </main>
    
    <!-- GLOBAL EDIT MODAL -->
    <EditModal 
      :show="showEditModal" 
      :title="'EDIT ' + editingModule.toUpperCase() + ' RECORD'"
      @close="showEditModal = false"
    >
      <div class="modal-content-inner">
        <ErasureDetailView v-if="editingModule === 'erasure'" :record="editingRecord" :isCreating="false" :isEditing="true" :isModal="true" @back="showEditModal = false" @save="handleSaveEdit" />
        <ReliabilityDetailView v-else-if="editingModule === 'reliability'" :record="editingRecord" :isCreating="false" :isEditing="true" :isModal="true" @back="showEditModal = false" @save="handleSaveEdit" />
        <MagneticPropertiesDetailView v-else-if="editingModule === 'magProp'" :record="editingRecord" :isCreating="false" :isEditing="true" :isModal="true" @back="showEditModal = false" @save="handleSaveEdit" />
        <ShipmentTypeADetailView v-else-if="editingModule === 'shipmentA'" :record="editingRecord" :isCreating="false" :isEditing="true" :isModal="true" @back="showEditModal = false" @save="handleSaveEdit" />
        <ShipmentTypeBDetailView v-else-if="editingModule === 'shipmentB'" :record="editingRecord" :isCreating="false" :isEditing="true" :isModal="true" @back="showEditModal = false" @save="handleSaveEdit" />
        <ProductView v-else-if="editingModule === 'product'" :productCode="editingRecord.code" :isCreating="false" :isEditing="true" :isModal="true" @back="showEditModal = false" @save="handleSaveEdit" />
        <ValidationDetailView v-else-if="editingModule === 'validation'" :record="editingRecord" :isCreating="false" :isEditing="true" :isModal="true" @back="showEditModal = false" @save="handleSaveEdit" />
        <InspectionDetailView v-else-if="editingModule === 'inspection'" :record="editingRecord" :isCreating="false" :isEditing="true" :isModal="true" @back="showEditModal = false" @save="handleSaveEdit" />
        <SamplingLevelDetailView v-else-if="editingModule === 'samplingLevel'" :record="editingRecord" :isCreating="false" :isEditing="true" :isModal="true" @back="showEditModal = false" @save="handleSaveEdit" />
      </div>
    </EditModal>
  </div>
</template>

<style scoped>
.selected-row td {
  background-color: #f2dfe1 !important;
}

.top-record-box {
  background-color: #fff;
  border: 2px solid #c7c7c7;
  margin-bottom: 20px;
}

.box-header {
  background-color: #c7c7c7;
  border-bottom: 2px solid #c7c7c7;
  margin: -2px -2px 0 -2px;
}

.list-panel {
  padding: 15px;
  border: none;
  min-height: auto;
}

.list-inner-box {
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 2px;
}

.toolbar-row {
  padding: 8px 15px;
  border-bottom: 1px solid #ddd;
}

.action-row {
  background-color: #f7f7f7;
}

.search-row {
  background-color: #ffffff;
}

.action-link:hover {
  color: #8f3235;
  text-decoration: none;
}

.no-margin {
  margin-bottom: 0 !important;
}

/* Pagination Bar Styles */
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
</style>
