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

const printRecord = () => {
  // Build printable HTML content
  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <title>Shipment Type A Report</title>
      <style>
        @page { size: A4 portrait; margin: 10mm; }
        * { box-sizing: border-box; }
        body { font-family: Arial, Helvetica, sans-serif; margin: 0; padding: 0; font-size: 11px; color: #000; background-color: #fff; }
        .page-container { width: 210mm; min-height: 297mm; margin: 0 auto; padding: 10mm; background-color: #fff; }
        @media screen {
          body { background-color: #525659; display: flex; justify-content: center; padding: 20px 0; }
          .page-container { box-shadow: 0 0 10px rgba(0,0,0,0.5); }
        }
        @media print {
          body { background-color: transparent; padding: 0; display: block; }
          .page-container { width: 100%; min-height: auto; margin: 0; padding: 0; box-shadow: none; }
        }
        .header-table { width: 100%; margin-bottom: 20px; border: none; }
        .header-table td { border: none; padding: 0; vertical-align: bottom; }
        .company-logo { display: flex; align-items: center; margin-left: 20px; }
        .company-logo span.shin { color: #004b87; font-size: 64px; font-weight: 900; font-family: 'Arial Black', Impact, sans-serif; letter-spacing: -2px; font-style: italic; }
        .company-logo span.etsu { color: #009988; font-size: 64px; font-weight: 900; font-family: 'Arial Black', Impact, sans-serif; letter-spacing: -2px; font-style: italic; }
        .company-details { text-align: left; color: #0000ff; font-weight: bold; font-size: 13px; line-height: 1.3; padding-bottom: 15px; padding-left: 40px; }
        h1 { text-align: center; font-size: 20px; margin: 10px 0; padding: 0; font-weight: bold; }
        .title-divider { border-top: 1px solid #000; border-bottom: 2px solid #000; height: 1px; margin-bottom: 15px; }
        table { width: 100%; border-collapse: collapse; margin-bottom: 15px; }
        th, td { border: 1px solid #000; padding: 5px 8px; font-size: 11px; text-align: left; }
        th { font-weight: bold; background-color: transparent; }
        .text-center { text-align: center !important; }
        .font-bold { font-weight: bold; }
        .mt-20 { margin-top: 20px; }
        .underline-title { font-weight: bold; text-decoration: underline; background-color: transparent; border-bottom: 1px solid #000; }
      </style>
    </head>
    <body>
      <div class="page-container">
        <table class="header-table">
          <tr>
            <td style="width: 50%;">
              <div class="company-logo">
                <span class="shin">Shin</span><span class="etsu">Etsu</span>
              </div>
            </td>
            <td style="width: 50%;">
              <div class="company-details">
                SHIN-ETSU (MALAYSIA) SDN BHD (Plant 1)<br/>
                Lot 50, Jalan Serendah 26/17<br/>
                HICOM Industrial Estate<br/>
                40400 Shah Alam<br/>
                Selangor Darul Ehsan
              </div>
            </td>
          </tr>
        </table>
        
        <h1>Shipment Type A Report</h1>
        <div class="title-divider"></div>

        <table>
          <tr>
            <th style="width: 25%;">Report ID :</th>
            <td style="width: 25%;">${localRecord.value.reportId || ''}</td>
            <th style="width: 25%;">Issued Date :</th>
            <td style="width: 25%;">${localRecord.value.issuedDate || ''}</td>
          </tr>
          <tr>
            <th>Customer :</th>
            <td colspan="3">${localRecord.value.customer || ''}</td>
          </tr>
          <tr>
            <th>Material :</th>
            <td>${localRecord.value.material || ''}</td>
            <th>Our Code No :</th>
            <td>${localRecord.value.codeNo || ''}</td>
          </tr>
          <tr>
            <th>Customer's P/O No :</th>
            <td>${localRecord.value.customerPo || ''}</td>
            <th>Our P/O No :</th>
            <td>${localRecord.value.ourPo || ''}</td>
          </tr>
          <tr>
            <th>Customer's Dwg / Part No :</th>
            <td colspan="3">${localRecord.value.customerDwg || ''}</td>
          </tr>
          <tr>
            <th>Quantity :</th>
            <td>${localRecord.value.quantity || ''}</td>
            <th>Unit Weight :</th>
            <td>${localRecord.value.unitWeight || ''}</td>
          </tr>
          <tr>
            <th>Magnetization Through :</th>
            <td>${localRecord.value.magThrough || ''}</td>
            <th>Magnetization :</th>
            <td>${localRecord.value.magnetization || ''}</td>
          </tr>
          <tr>
            <th>Marking :</th>
            <td>${localRecord.value.marking || ''}</td>
            <th>Dimension :</th>
            <td>${localRecord.value.dimension || ''}</td>
          </tr>
          <tr>
            <th>Judgement :</th>
            <td>${localRecord.value.judgement || ''}</td>
            <th>Notes :</th>
            <td>${localRecord.value.notes || ''}</td>
          </tr>

          <tr><td colspan="4" style="border-left: none; border-right: none; height: 10px; background: transparent;"></td></tr>

          <tr><td colspan="4" class="underline-title">Magnetic Properties :</td></tr>
          <tr>
            <th colspan="2" class="text-center font-bold">Name</th>
            <th colspan="2" class="text-center font-bold">AVG</th>
          </tr>
          ${magProps.value.map(row => `<tr><td colspan="2" class="text-center">${row.name}</td><td colspan="2" class="text-center">${row.avg}</td></tr>`).join('')}

          <tr><td colspan="4" style="border-left: none; border-right: none; height: 10px; background: transparent;"></td></tr>

          <tr><td colspan="4" class="underline-title">Product Magnetic Properties :</td></tr>
          <tr>
            <th class="text-center font-bold">Name</th>
            <th class="text-center font-bold">AVG</th>
            <th class="text-center font-bold">MIN</th>
            <th class="text-center font-bold">MAX</th>
          </tr>
          ${prodMagProps.value.map(row => `<tr><td class="text-center">${row.name}</td><td class="text-center">${row.avg}</td><td class="text-center">${row.min}</td><td class="text-center">${row.max}</td></tr>`).join('')}

          <tr><td colspan="4" style="border-left: none; border-right: none; height: 10px; background: transparent;"></td></tr>

          <tr><td colspan="4" class="underline-title">Additional Specification :</td></tr>
          <tr>
            <th class="text-center font-bold">Item</th>
            <th class="text-center font-bold">Judgment</th>
            <th class="text-center font-bold">Instrument</th>
            <th class="text-center font-bold">The Symbol for Instrument</th>
          </tr>
          ${addSpecs.value.map(row => `<tr><td class="text-center">${row.item}</td><td class="text-center">${row.judgment}</td><td class="text-center">${row.instrument}</td><td class="text-center">${row.symbol}</td></tr>`).join('')}
        </table>

        <!-- Signatures Footer -->
        <table class="mt-20">
          <tr>
            <th class="text-center" style="padding-bottom: 5px; width: 25%; text-decoration: underline;">Prepared By :</th>
            <th class="text-center" style="padding-bottom: 5px; width: 25%; text-decoration: underline;">Checked By :</th>
            <th class="text-center" style="padding-bottom: 5px; width: 25%; text-decoration: underline;">Approved By :</th>
            <th style="width: 25%;"></th>
          </tr>
          <tr>
            <td class="text-center" style="padding: 25px 0 5px 0;">${localRecord.value.createdBy || ''}</td>
            <td class="text-center" style="padding: 25px 0 5px 0;">${localRecord.value.checkedBy || ''}</td>
            <td class="text-center" style="padding: 25px 0 5px 0;">${localRecord.value.approvedBy || ''}</td>
            <td></td>
          </tr>
          <tr>
            <td style="padding: 0;">
              <table style="margin: 0; border: none; width: 100%;"><tr><td style="border: none; border-right: 1px solid #000; width: 30%; text-align: center;">Date:</td><td style="border: none; width: 70%; text-align: center;">${localRecord.value.createdTs ? localRecord.value.createdTs.split(' ')[0] : ''}</td></tr></table>
            </td>
            <td style="padding: 0;">
              <table style="margin: 0; border: none; width: 100%;"><tr><td style="border: none; border-right: 1px solid #000; width: 30%; text-align: center;">Date:</td><td style="border: none; width: 70%; text-align: center;">${localRecord.value.updatedTs ? localRecord.value.updatedTs.split(' ')[0] : ''}</td></tr></table>
            </td>
            <td style="padding: 0;">
              <table style="margin: 0; border: none; width: 100%;"><tr><td style="border: none; border-right: 1px solid #000; width: 30%; text-align: center;">Date:</td><td style="border: none; width: 70%; text-align: center;">${localRecord.value.updatedTs ? localRecord.value.updatedTs.split(' ')[0] : ''}</td></tr></table>
            </td>
            <td></td>
          </tr>
        </table>
      </div>
    </body>
    </html>`;
  const printWindow = window.open('', '_blank', 'width=900,height=800');
  printWindow.document.open();
  printWindow.document.write(html);
  printWindow.document.close();
  printWindow.focus();
  // Give the browser a moment to render before printing
  setTimeout(() => { printWindow.print(); }, 300);
};

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

const addSpecs = ref(localRecord.value.addSpecs && localRecord.value.addSpecs.length ? localRecord.value.addSpecs : [
  { item: 'Appearance', judgment: 'OK', instrument: 'A', symbol: 'A : BH tracer' },
  { item: 'Length', judgment: 'NG', instrument: 'B', symbol: 'B : Flux meter' },
  { item: 'Weight', judgment: 'OK', instrument: 'C', symbol: 'C : Gauss meter' }
])
const addAddSpec = () => { addSpecs.value.push({ item: '', judgment: '', instrument: '', symbol: '' }) }
const removeAddSpec = (idx) => { addSpecs.value.splice(idx, 1) }

const itemOpts = [
  'Appearance', 'Length', 'Length 1', 'Length 2', 'Length 3', 'Width', 'Width 1', 'Width 2', 'Width 3',
  'Thickness', 'Thickness 1', 'Thickness 2', 'Thickness 3', 'Height', 'Height 1', 'Height 2', 'Height 3',
  'Outer Diameter', 'Inner Diameter', 'Outer Radius', 'Inner Radius', 'Parallelism', 'Parallelism 1', 'Parallelism 2',
  'Flatness', 'Flatness 1', 'Flatness 2', 'Perpendicularity', 'Perpendicularity 1', 'Perpendicularity 2',
  'Straightness', 'Waviness', 'Angle', 'Circularity', 'Concentricity', 'Center OFF', 'Profile', 'Weight', 'Chamfer',
  'Plating Thickness', 'Coating Thickness', 'Tape Test', 'Marking', 'Total Flux', 'Flux density', 'Magnetic declination',
  'Magnetic Moment', 'Thermal Degradation of magnetic moment', 'Magnet Momentum within 1 delivery', 'Magnet Induction',
  'Magnet Sticking Checking (M.S.C)', 'Weight Loss Test', 'Load test', 'Salt Spray Test (SST)', 'Pressure Cooker Test (PCT)',
  'Humidity test', 'Surface tension', 'Hardness of coating', 'Cleanliness : Weight', 'Cleanliness : Particle size'
]
const judgmentOpts = ['OK', 'NG']
const instrumentOpts = [
  'A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z','1','2','3','4'
]
const symbolOpts = [
  'A : BH tracer','B : Flux meter','C : Gauss meter','D : Micrometer','E : Dial gauge','F : Caliper','G : Limit gauge','H : Special Inspection Jig','I : Projector','J : Measure scope','K : Comparator','L : Std. magnet','M : Pin gauge','N : Magnetic viewer','O : Naked Eyes','P : Plating Thickness Gauge','Q : Contracer','R : Weighing scale','S : Square, Thickness Gauge','T : Tape','U : Linear Gauge','V : Load Test Machine','W : X-Ray Machine','X : Paint Checker Machine','Y : Gloss Ratio Checker','Z : Image Processor','1 : Helmholts Coil','2 : PCT Machine','3 : Hall Element','4 : Magnetic Declination Check Fixture'
]

const goBack = () => {
  emit('back')
}

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
        <a href="#" class="item-link" @click.prevent="goBack" style="color: #0000EE;">SHIPMENT TYPE A RECORDS</a> 
        &gt; <span class="current-page" style="font-weight: normal; color: #333;">{{ isCreating ? 'Create New' : (localRecord.reportId || 'New') }}</span>
      </span>
    </div>

    <!-- Top Action Bar -->
    <div class="top-actions">
      <template v-if="!isEditing">
        <button class="btn btn-primary" @click="startEdit">EDIT</button>
        <button class="btn btn-secondary" @click="goBack">Cancel</button>
        <button class="btn btn-secondary" @click="printRecord">Print</button>
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
.item-link { color: #0000EE; text-decoration: none; }
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
