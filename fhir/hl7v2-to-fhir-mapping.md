# HL7 v2 → FHIR Mapping (Rx Flow)

## 1. Purpose and scope

This document describes **key mappings from typical HL7 v2 pharmacy/order messages** (e.g., ORM/RDE) into the **canonical Rx FHIR transaction Bundle** defined in [`03-fhir-rx-bundle-profile.md`](./03-fhir-rx-bundle-profile.md).

- Scope: only the main fields needed for the MVP Rx flow (patient, prescriber, medication, quantity, refills, route, timing, pharmacy).  
- Non‑goals: exhaustive segment/field coverage, all trigger events, or all local/custom Z‑segments.

The intent is to give integration and mapping teams a **clear starting point**, not a full HL7 implementation guide.

---

## 2. Message types in scope (MVP)

For the MVP, we assume:

- **ORM^O01 / RDE^O11** (or equivalent) are used for medication orders/prescriptions between the hospital EHR and internal pharmacy.  
- Standard segments in scope: `MSH`, `PID`, `PV1` (optional), `ORC`, `RXE`, `RXR`.  
- Local/custom segments (e.g., `ZPH`, `ZPV`) may exist but are not modeled here; they would be handled case-by-case.

All such messages are mapped into a single FHIR `Bundle` of type `transaction` containing `Patient`, `Practitioner`, `Organization`, `MedicationRequest`, and Pharmacy `Organization`.

---

## 3. MSH → Bundle metadata

### Mapping

| HL7 v2 field | FHIR element                        | Notes                               |
|--------------|-------------------------------------|-------------------------------------|
| MSH-7        | Bundle.timestamp                    | Message creation time               |
| MSH-3        | Bundle.meta.source (or extension)   | Sending application/system ID       |
| MSH-5        | Extension on Bundle (receiver ID)   | Intended primary receiving system   |
| MSH-10       | Bundle.id (or correlation extension)| Message control ID / transaction ID |

- If a separate enterprise transaction ID exists, it can be placed in a dedicated `Bundle` extension and reused as `transactionId` in APIs and analytics.

---

## 4. PID → Patient

### Mapping

| HL7 v2 field        | FHIR element               | Notes                               |
|---------------------|----------------------------|-------------------------------------|
| PID-3 (Patient ID)  | Patient.identifier         | MRN and/or internal IDs             |
| PID-5 (Name)        | Patient.name               | family / given                      |
| PID-7 (DOB)         | Patient.birthDate          |                                     |
| PID-8 (Sex)         | Patient.gender             |                                     |
| PID-11 (Address)    | Patient.address            | line, city, postalCode, country     |
| PID-13 (Phone)      | Patient.telecom            | system = phone                      |

### Contract (MVP)

- Required for mapping:
  - At least one `PID-3` identifier that can be used as the **patient’s MRN or internal ID**.
  - `PID-5`, `PID-7`, `PID-8` to populate name, birth date, and gender.
- Optional:
  - Phone and address fields; if missing, FHIR `Patient` is still valid, but some use cases may be impacted.

---

## 5. ORC / RXE / RXR → MedicationRequest

The medication order is mapped primarily from `ORC`, `RXE`, and `RXR` segments into `MedicationRequest`.

### 5.1 ORC – order control and identifiers

| HL7 v2 field            | FHIR element                                      | Notes                              |
|-------------------------|---------------------------------------------------|------------------------------------|
| ORC-2 (Placer Order #)  | MedicationRequest.identifier                      | Prescription / order number        |
| ORC-1 (Order Control)   | MedicationRequest.status                          | Map to FHIR: e.g., NW → active     |
| ORC-7 (Quantity/Timing) | MedicationRequest.dosageInstruction.timing        | May be simplified for MVP          |
| ORC-9 (Date/Time)       | MedicationRequest.authoredOn                      | Order creation date/time           |
| ORC-12 (Ordering Provider) | MedicationRequest.requester (ref to Practitioner) | Prescriber                         |

- `ORC-1` values (e.g., NW, CA) are mapped to FHIR `status` (e.g., active, cancelled) using a simple mapping table per site.

### 5.2 RXE – dispense/give details

| HL7 v2 field        | FHIR element                                           | Notes                                 |
|---------------------|--------------------------------------------------------|---------------------------------------|
| RXE-2 (Give Code)   | MedicationRequest.medicationCodeableConcept           | Medication code + display             |
| RXE-3 (Give Amount) | MedicationRequest.dosageInstruction.doseAndRate.doseQuantity.value | Dose                 |
| RXE-4 (Give Units)  | MedicationRequest.dosageInstruction.doseAndRate.doseQuantity.unit  | Dose unit            |
| RXE-10 (Quantity)   | MedicationRequest.dispenseRequest.quantity.value      | Dispense quantity                     |
| RXE-11 (Number of Refills) | MedicationRequest.dispenseRequest.numberOfRepeatsAllowed | Refills                 |

### 5.3 RXR – route

| HL7 v2 field      | FHIR element                                  | Notes                       |
|-------------------|-----------------------------------------------|-----------------------------|
| RXR-1 (Route)     | MedicationRequest.dosageInstruction.route     | CodeableConcept for route   |

### 5.4 Additional fields

- Order priority (if present, e.g., in ORC or in a site-specific field) → `MedicationRequest.priority`.  
- Diagnosis/indication:
  - If present in a DG1 segment or Z‑segment, mapped to `MedicationRequest.reasonCode`.  
- Free‑text instructions / notes:
  - Rx comments in pharmacy segments → `MedicationRequest.note`.

---

## 6. Prescriber and organization mapping

Prescriber details are typically contained in `ORC-12` (Ordering Provider) and sometimes in repeating provider segments.

### 6.1 ORC-12 → Practitioner

| HL7 v2 field (from ORC-12) | FHIR element             | Notes                            |
|----------------------------|--------------------------|----------------------------------|
| ID number                  | Practitioner.identifier  | Internal prescriber ID           |
| Family name                | Practitioner.name.family |                                  |
| Given name(s)              | Practitioner.name.given  |                                  |

- If NPI or national IDs are available (e.g., via specific components or XCN subfields), they can be added as additional `identifier` entries.

### 6.2 Prescriber organization

- If the prescriber’s organization is available (e.g., in ORC-21 or a site-specific segment), map to a prescriber `Organization`:
  - `Organization.identifier`
  - `Organization.name`

In `MedicationRequest`:

- `MedicationRequest.requester` → reference to `Practitioner`.  
- `MedicationRequest.requester.onBehalfOf` → reference to prescriber `Organization` (if populated).

---

## 7. Pharmacy / fulfillment mapping

Pharmacy information may appear in one or more of:

- ORC shipping fields.  
- Specific pharmacy-related segments (site‑specific).  
- Z‑segments.

For MVP, assume there is a field that identifies the **destination pharmacy** (e.g., an internal ID).

### Mapping

| HL7 v2 concept             | FHIR element                            | Notes                         |
|----------------------------|-----------------------------------------|-------------------------------|
| Destination Pharmacy ID    | Pharmacy Organization.identifier        | Internal or partner ID        |
| Destination Pharmacy Name  | Pharmacy Organization.name              |                               |

In `MedicationRequest`:

- `MedicationRequest.dispenseRequest.performer` → reference to the pharmacy `Organization`.

---

## 8. Error and edge-case handling (mapping level)

A few mapping rules worth stating explicitly:

- **Missing required fields**  
  - If key fields required by the FHIR Rx profile (e.g., medication code, patient ID, prescriber ID) are missing from the v2 message and cannot be derived, the transaction should **fail validation** with a clear error rather than attempting to create a partial `MedicationRequest`.

- **Unknown codes**  
  - If medication or diagnosis codes are not recognized, they may be passed through as‑is, flagged as warnings, or rejected, depending on policy; this is controlled by semantic validation rules.

- **Multiple orders in one message**  
  - If an HL7 message contains multiple ORC/RXE groups, each may map to a separate `MedicationRequest` within the same Bundle, referencing the same Patient and Practitioner.

- **Local/custom Z‑segments**  
  - Handling of Z‑segments is site‑specific and should be documented separately if used for business‑critical data (e.g., consent flags, special routing hints).

---

## 9. Reverse mapping (FHIR → HL7 v2)

For downstream systems that still expect HL7 v2 (e.g., internal pharmacy interfaces), Rhapsody maps from the canonical FHIR Rx Bundle back to v2:

- `Patient` → PID segment.  
- `MedicationRequest` → ORC/RXE/RXR segments.  
- `Practitioner` → ORC-12 (ordering provider).  
- Pharmacy `Organization` → destination pharmacy fields.

The reverse mapping is conceptually the inverse of the tables above, with additional target-specific formatting, code conversions, and trigger event handling as required.

---

## 10. Next steps in a real project

In a real implementation, this high‑level mapping would be extended with:

- Detailed per‑field mapping specs (including data types, code systems, and example values).  
- Site‑specific mapping rules for local/custom segments.  
- Automated tests to validate that HL7 v2 messages map to FHIR Bundles (and back) as expected across a representative sample of messages.

