<script lang="ts">
  import { setBasePath } from '@shoelace-style/shoelace/dist/utilities/base-path';
  import '@shoelace-style/shoelace/dist/themes/light.css';

  import '@shoelace-style/shoelace/dist/components/input/input.js';
  import '@shoelace-style/shoelace/dist/components/select/select.js';
  import '@shoelace-style/shoelace/dist/components/option/option.js';
  import '@shoelace-style/shoelace/dist/components/button/button.js';
  import '@shoelace-style/shoelace/dist/components/textarea/textarea.js';

  // Set Shoelace base path before usage
  setBasePath('https://cdn.jsdelivr.net/npm/@shoelace-style/shoelace@2.20.1/cdn/');


  let patient = {
    id: '',
    givenName: '',
    familyName: '',
    gender: '',
    birthDate: '',
    street: '',
    city: '',
    postalCode: '',
    contactNumber: ''
  };

  let fhirJson = '';
  let hl7Text = '';

  const hospitalImage = 'https://images.unsplash.com/photo-1503387762-592deb58ef4e?auto=format&fit=crop&w=400&q=80';

  function bindField(obj, field) {
    return (e) => obj[field] = e.target.value;
  }

  function formatDateHL7(dateStr) {
    if (!dateStr) return '';
    return dateStr.replace(/-/g, '');
  }

  function getCurrentHL7Timestamp() {
    const now = new Date();
    const pad = (n) => n.toString().padStart(2, '0');
    return `${now.getFullYear()}${pad(now.getMonth() + 1)}${pad(now.getDate())}${pad(now.getHours())}${pad(now.getMinutes())}${pad(now.getSeconds())}`;
  }

  const generateData = () => {
    const fhirBundle = {
      resourceType: "Bundle",
      type: "collection",
      entry: [
        {
          fullUrl: `urn:uuid:${patient.id || 'no-id'}`,
          resource: {
            resourceType: "Patient",
            identifier: [{ system: "http://hospital.org/mrn", value: patient.id || '' }],
            name: [{ family: patient.familyName || '', given: [patient.givenName || ''] }],
            gender: (patient.gender || '').toLowerCase(),
            birthDate: patient.birthDate || '',
            telecom: [{ system: "phone", value: patient.contactNumber || '', use: "home" }],
            address: [{
              line: [patient.street || ''],
              city: patient.city || '',
              postalCode: patient.postalCode || ''
            }]
          }
        }
      ]
    };
    fhirJson = JSON.stringify(fhirBundle, null, 2);

    const msh = [
      'MSH',
      '^~\\&',
      'REGISTRATION_APP',
      'HOSPITAL',
      'RECEIVING_APP',
      'RECEIVING_FAC',
      getCurrentHL7Timestamp(),
      '',
      'ADT^A04',
      '123456',
      'P',
      '2.5'
    ].join('|');

    const pid = [
      'PID',
      '1',
      '',
      `${patient.id || ''}^^^Hospital^MR`,
      '',
      `${patient.familyName || ''}^${patient.givenName || ''}`,
      '',
      formatDateHL7(patient.birthDate),
      (patient.gender || '').toUpperCase(),
      '',
      '',
      `${patient.street || ''}^^${patient.city || ''}^^${patient.postalCode || ''}^USA`,
      '',
      patient.contactNumber || ''
    ].join('|');

    hl7Text = [msh, pid].join('\n');
  };

  const download = (content, filename, type) => {
    const blob = new Blob([content], { type });
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = filename;
    a.click();
  };
</script>

<style>
  :global(body) {
    margin: 0;
    min-height: 100vh;
    background: linear-gradient(135deg, #2a5d84 0%, #1c8adb 50%, #6cb2eb 100%);
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }

  .main-flex {
    display: flex;
    justify-content: center;
    align-items: flex-start;
    gap: 2rem;
    padding: 2rem;
    flex-wrap: wrap;
  }

  .section {
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    padding: 2rem;
    border-radius: 16px;
    box-shadow: 0 12px 30px rgba(0, 0, 0, 0.25);
    color: #222;
    min-width: 320px;
    flex: 1;
  }

  h1, h2 {
    color: #14375a;
    text-align: center;
  }

  sl-input, sl-select {
    margin-bottom: 1rem;
    width: 100%;
  }

  sl-button {
    margin-top: 1rem;
    width: 100%;
  }

  .hospital-image-container {
    text-align: center;
    margin-bottom: 24px;
  }

  .hospital-image-container img {
    max-width: 320px;
    width: 100%;
    height: auto;
    border-radius: 16px;
    box-shadow: 0 6px 20px rgba(0,0,0,0.4);
  }
</style>

<!-- ✅ FLEX ROW: FHIR | FORM | HL7 -->
<div class="main-flex">
  <!-- FHIR output -->
  <div class="section">
    <h2>FHIR JSON</h2>
    <sl-textarea readonly rows="20" value={fhirJson}></sl-textarea>
    <sl-button variant="success" on:click={() => download(fhirJson, 'patient.fhir.json', 'application/json')}>
      Download FHIR
    </sl-button>
  </div>

  <!-- Patient Form in Center -->
  <div class="section">
    <div style="text-align:center; font-size: 0.9rem; font-weight: 600; color: red; margin-bottom: 0.5rem;">
      Created by - Divyanshu Khattar
    </div>
    <h1>Patient Registration</h1>
    <div class="hospital-image-container">
      <img src={hospitalImage} alt="Hospital" />
    </div>
    <sl-input label="Patient ID" on:sl-input={bindField(patient, 'id')} value={patient.id}></sl-input>
    <sl-input label="Given Name" on:sl-input={bindField(patient, 'givenName')} value={patient.givenName}></sl-input>
    <sl-input label="Family Name" on:sl-input={bindField(patient, 'familyName')} value={patient.familyName}></sl-input>
    <sl-select label="Gender" value={patient.gender} on:sl-change={e => patient.gender = e.target.value}>
      <sl-option value="">Select Gender</sl-option>
      <sl-option value="male">Male</sl-option>
      <sl-option value="female">Female</sl-option>
      <sl-option value="other">Other</sl-option>
      <sl-option value="unknown">Unknown</sl-option>
    </sl-select>
    <sl-input type="date" label="Birth Date" on:sl-input={bindField(patient, 'birthDate')} value={patient.birthDate}></sl-input>
    <sl-input label="Street" on:sl-input={bindField(patient, 'street')} value={patient.street}></sl-input>
    <sl-input label="City" on:sl-input={bindField(patient, 'city')} value={patient.city}></sl-input>
    <sl-input label="Postal Code" on:sl-input={bindField(patient, 'postalCode')} value={patient.postalCode}></sl-input>
    <sl-input label="Contact Number" on:sl-input={bindField(patient, 'contactNumber')} value={patient.contactNumber}></sl-input>
    <sl-button variant="primary" on:click={generateData}>Generate</sl-button>
  </div>

  <!-- HL7 Output -->
  <div class="section">
    <h2>HL7 Text</h2>
    <sl-textarea readonly rows="20" value={hl7Text}></sl-textarea>
    <sl-button variant="success" on:click={() => download(hl7Text, 'patient.hl7.txt', 'text/plain')}>
      Download HL7
    </sl-button>
  </div>
</div>

