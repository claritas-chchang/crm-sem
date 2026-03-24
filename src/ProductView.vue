<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  productCode: String
})

const emit = defineEmits(['back'])

const productType = ref('G')
const typeOptions = ['G', 'Non-G']

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

const openAddModal = () => {
  newDim.value = { ...initialNewDim }
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

</script>

<template>
  <div class="view-panel">
    
    <div class="top-record-box">
      <div class="top-actions">
        <button class="btn btn-primary">EDIT</button>
        <button class="btn btn-secondary" @click="emit('back')">Cancel</button>
      </div>

      <!-- Top Details Wrapper -->
    <div class="form-wrapper">
      <!-- Product Details Section -->
      <div class="section-container inner-section">
        <div class="section-title">Product Details</div>
        <div class="info-grid half">
          <div class="grid-row">
            <div class="grid-label">Product ID/Code</div>
            <div class="grid-value">{{ productCode }}</div>
          </div>
          <div class="grid-row">
            <div class="grid-label">Product Type</div>
            <div class="grid-value">
              <select v-model="productType" class="form-select">
                <option v-for="opt in typeOptions" :key="opt" :value="opt">{{ opt }}</option>
              </select>
            </div>
          </div>
        </div>
      </div>

      <!-- System Information Section -->
      <div class="section-container inner-section">
        <div class="section-title">System Information</div>
        <div class="info-grid">
          <div class="grid-col">
            <div class="grid-row">
              <div class="grid-label">Created Date</div>
              <div class="grid-value">16-March-2026 12:58:05 PM</div>
            </div>
            <div class="grid-row">
              <div class="grid-label">Last Updated Date</div>
              <div class="grid-value">16-March-2026 12:59:47 PM</div>
            </div>
          </div>
          <div class="grid-col">
            <div class="grid-row">
              <div class="grid-label">Created By</div>
              <div class="grid-value">qa-admin-p2</div>
            </div>
            <div class="grid-row">
              <div class="grid-label">Last Updated By</div>
              <div class="grid-value">qa-tech-p2</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    </div>

    <!-- Dimension List Section -->
    <div class="sub-panel-wrapper">
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
          <div class="sub-actions">
            <span class="text-link" @click="openAddModal">Add New Dimension</span>
          </div>

          <div class="table-scroll-container">
            <table class="data-table dim-table">
              <thead>
                <tr>
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
                </tr>
              </thead>
              <tbody>
                <tr v-for="(d, idx) in dimensions" :key="idx">
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
                </tr>
              </tbody>
            </table>
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
          <span style="font-weight: bold; color: black;">DIMENSION MANAGEMENT</span>
        </div>
        
        <div class="panel modal-panel">
          <div class="top-actions" style="padding: 15px 0 10px 0; background-color: transparent;">
            <button class="btn btn-primary" @click="addDimension">ADD</button>
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

  </div>
</template>

<style scoped>
.view-panel {
  background-color: #f7f7f7;
  min-height: calc(100vh - 130px);
  position: relative;
  padding-top: 15px; /* Creates the gap below Sub Header */
}

.top-record-box {
  background-color: #fff;
  border: 1px solid #a0a0a0;
  margin: 0 15px 15px 15px;
}

.top-actions {
  display: flex;
  gap: 10px;
  padding: 15px;
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

.section-container.no-padding {
  margin-top: 15px;
}

.section-title {
  font-weight: bold;
  font-size: 13px;
  margin-bottom: 0;
  padding: 8px 15px;
  background-color: #f0f0f0;
  color: #333;
}

/* Sub-panel styling */
.sub-panel-wrapper {
  margin: 0 15px 15px 15px;
}

.sub-panel-header {
  background-color: #d1d1d1;
  padding: 6px 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  border: 1px solid #a0a0a0;
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
  background-color: #f7f7f7;
  padding: 10px 0;
  border-left: 1px solid #a0a0a0;
  border-right: 1px solid #a0a0a0;
  border-bottom: 1px solid #a0a0a0;
}

.sub-panel-inner-box {
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 2px;
  margin: 0 10px;
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
</style>
