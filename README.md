# Patient Registration HL7-FHIR Converter

A lightweight Svelte web app to register patient details and generate healthcare data in **FHIR JSON** and **HL7** formats. Ideal for testing and demonstration of healthcare data standards interoperability.

---

## Features

- User-friendly patient registration form with fields for:
  - Patient ID
  - Given Name
  - Family Name
  - Gender
  - Birth Date
  - Address (Street, City, Postal Code)
  - Contact Number
- Generates **FHIR Bundle JSON** compliant with FHIR Patient resource
- Generates **HL7 ADT^A04** message text for patient admission
- Downloadable output files (`.json` and `.txt`)
- Clean responsive UI built with [Shoelace](https://shoelace.style) and [Medblocks UI](https://medblocks.io)

---

## Demo Screenshot

![Hospital Image](https://images.unsplash.com/photo-1503387762-592deb58ef4e?auto=format&fit=crop&w=400&q=80)

---

## Installation & Usage

1. Clone the repository

   ```bash
   git clone https://github.com/Dk1509/patient-registration-hl7-fhir.git
   cd patient-registration-hl7-fhir
