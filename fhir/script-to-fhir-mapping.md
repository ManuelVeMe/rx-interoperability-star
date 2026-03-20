# NCPDP SCRIPT → FHIR Mapping (Rx Flow)

## 1. Purpose and scope

This document describes **key mappings from NCPDP SCRIPT NewRx transactions** into the **canonical Rx FHIR transaction Bundle** defined in [`03-fhir-rx-bundle-profile.md`](./03-fhir-rx-bundle-profile.md).

- Scope: main fields needed for the MVP Rx flow (patient, prescriber, medication, SIG, quantity, refills, pharmacy, payer).  
- Non‑goals: full SCRIPT coverage, all message types, or jurisdiction-specific constraints.

The intent is to give integration and mapping teams a **clear starting point**, not a full NCPDP implementation guide.

---

## 2. Message types in scope (MVP)

For MVP we assume:

- **NewRx** is the primary SCRIPT transaction used to send new prescriptions from ambulatory EHRs to pharmacies or intermediaries.  
- Other transactions (e.g., RxChange, CancelRx, RxFill) are out of scope for MVP and would be mapped later.

Each NewRx message is mapped into a single FHIR `Bundle` of type `transaction` containing:

- `Patient`  
- `Practitioner` (prescriber)  
- Prescriber `Organization`  
- `MedicationRequest`  
- Pharmacy `Organization`  
- Optional `Coverage`

---

## 3. NewRx Patient → Patient

### Mapping

(Concept names rather than exact XML paths; actual paths depend on the SCRIPT flavor used.)

| SCRIPT concept          | FHIR element        | Notes                      |
|-------------------------|---------------------|----------------------------|
| Patient Identifier      | Patient.identifier  | MRN / internal / external  |
| Patient Name (First/Last) | Patient.name      | family / given             |
| Date of Birth           | Patient.birthDate   |                            |
| Sex/Gender              | Patient.gender      |                            |
| Address                 | Patient.address     | line, city, postalCode, etc. |
| Phone                   | Patient.telecom     | system = phone             |

### Contract (MVP)

- Required:
  - At least one patient identifier
  - Name, birth date, and gender
- Optional:
  - Address and phone; useful for some pharmacy workflows but not blocking for all flows.

The `MedicationRequest.subject` references this `Patient`.

---

## 4. NewRx Prescriber → Practitioner / Organization

### 4.1 Prescriber → Practitioner

| SCRIPT concept              | FHIR element             | Notes                         |
|-----------------------------|--------------------------|-------------------------------|
| Prescriber Identifier       | Practitioner.identifier  | Local or system-specific ID   |
| Prescriber NPI              | Practitioner.identifier  | Identifier.type = NPI         |
| Prescriber Name (First/Last)| Practitioner.name        | family / given                |
| Prescriber Phone/Contact    | Practitioner.telecom     | Optional for MVP              |

### 4.2 Prescriber Organization

| SCRIPT concept            | FHIR element        | Notes                       |
|---------------------------|---------------------|-----------------------------|
| Clinic/Practice Name      | Organization.name   | Prescriber organization     |
| Clinic/Practice Identifier| Organization.identifier | Optional, if provided   |
| Clinic/Address            | Organization.address | Optional                    |

In `MedicationRequest`:

- `MedicationRequest.requester` → reference to `Practitioner`.  
- `MedicationRequest.requester.onBehalfOf` → reference to prescriber `Organization`, if populated.

---

## 5. NewRx Medication + SIG → MedicationRequest

The core prescription content (drug, SIG, quantity, refills) is mapped into `MedicationRequest`.

### 5.1 Identification and status

| SCRIPT concept                 | FHIR element                           | Notes                          |
|--------------------------------|----------------------------------------|--------------------------------|
| Prescription/Order Number      | MedicationRequest.identifier           | Primary Rx identifier          |
| NewRx message context          | MedicationRequest.status               | Typically mapped to `active`   |

### 5.2 Drug / medication

| SCRIPT concept              | FHIR element                                   | Notes                          |
|-----------------------------|-----------------------------------------------|--------------------------------|
| Drug Code (e.g., NDC)       | MedicationRequest.medicationCodeableConcept.coding.code | Code               |
| Drug Code System            | MedicationRequest.medicationCodeableConcept.coding.system | System URI        |
| Drug Description / Name     | MedicationRequest.medicationCodeableConcept.text          | Human-readable name|

If multiple codes are present (e.g., NDC + internal code), they can be represented as multiple `coding` entries.

### 5.3 SIG: dose, route, frequency

SCRIPT SIG content is often complex; for MVP we map primary elements:

| SCRIPT concept                       | FHIR element                                   | Notes                               |
|--------------------------------------|-----------------------------------------------|-------------------------------------|
| Dose amount                          | MedicationRequest.dosageInstruction.doseAndRate.doseQuantity.value | Dose        |
| Dose unit                            | MedicationRequest.dosageInstruction.doseAndRate.doseQuantity.unit  | Dose unit   |
| Route                                | MedicationRequest.dosageInstruction.route      | CodeableConcept (e.g., oral)        |
| Frequency / administration timing    | MedicationRequest.dosageInstruction.timing     | May be simplified for MVP           |
| Free-text SIG                        | MedicationRequest.dosageInstruction.text       | Optional; preserves original text   |

MVP keeps SIG mapping pragmatic: we map key numeric/structural elements where available and keep the free-text SIG as a fallback.

### 5.4 Quantity, duration, refills

| SCRIPT concept                 | FHIR element                                         | Notes                     |
|--------------------------------|------------------------------------------------------|---------------------------|
| Quantity to Dispense           | MedicationRequest.dispenseRequest.quantity.value     | Quantity                  |
| Quantity Unit                  | MedicationRequest.dispenseRequest.quantity.unit      | Unit                      |
| Days Supply / Duration         | MedicationRequest.dispenseRequest.expectedSupplyDuration | Optional             |
| Number of Refills              | MedicationRequest.dispenseRequest.numberOfRepeatsAllowed | Refills             |

### 5.5 Substitution and indication

| SCRIPT concept                    | FHIR element                                | Notes                       |
|-----------------------------------|---------------------------------------------|-----------------------------|
| Substitution Permitted?           | MedicationRequest.substitution.allowedBoolean | true/false                |
| Diagnosis / Indication            | MedicationRequest.reasonCode                | CodeableConcept            |
| Clinical Notes                    | MedicationRequest.note                      | Free-text notes            |

---

## 6. NewRx Pharmacy & Payer → Organization / Coverage

### 6.1 Pharmacy → Organization

| SCRIPT concept          | FHIR element                    | Notes                            |
|-------------------------|---------------------------------|----------------------------------|
| Destination Pharmacy ID | Pharmacy Organization.identifier| ID used for routing              |
| Destination Pharmacy    | Pharmacy Organization.name      | Human-readable name              |
| Pharmacy Address        | Pharmacy Organization.address   | Optional                         |

`MedicationRequest.dispenseRequest.performer` references the pharmacy `Organization`.

### 6.2 Payer / Plan → Coverage

| SCRIPT concept        | FHIR element                     | Notes                            |
|-----------------------|----------------------------------|----------------------------------|
| Payer / Plan ID       | Coverage.identifier / type       | Plan identifier                  |
| Member / Subscriber ID| Coverage.subscriberId            | Patient member ID                |
| Plan Name             | Coverage.class.name or Coverage.type.text | Human-readable name      |

`MedicationRequest.insurance` references the `Coverage` resource to link the order to payer information.

---

## 7. Error and edge-case handling (mapping level)

A few mapping rules worth stating explicitly:

- **Missing required fields**  
  - If key fields required by the FHIR Rx profile (e.g., medication code, patient ID, prescriber ID) are missing from the SCRIPT message and cannot be derived, the transaction should **fail validation** with a clear error rather than creating a partial `MedicationRequest`.

- **Free-text only SIG**  
  - When structured SIG details are incomplete, the FHIR `dosageInstruction.text` can carry the original SIG; structured fields are populated only when reliable data exists.

- **Unknown codes**  
  - Medication or diagnosis codes unknown to the enterprise code sets may be passed through with warnings or flagged as errors based on policy.

- **Multiple medications**  
  - A single NewRx typically represents one medication; if multiple medications are present (e.g., multiple item groups), each would map to a separate `MedicationRequest` within the same Bundle.

---

## 8. Reverse mapping (FHIR → SCRIPT)

For downstream systems that still expect SCRIPT (e.g., retail/mail-order pharmacies), Rhapsody maps from the canonical FHIR Rx Bundle back to SCRIPT:

- `Patient` → Patient section of NewRx.  
- `Practitioner` and prescriber `Organization` → Prescriber section.  
- `MedicationRequest` → Drug, SIG, Quantity/Refill, Substitution, and Indication sections.  
- Pharmacy `Organization` → Destination pharmacy.  
- `Coverage` → Insurance/Payer sections where supported.

The reverse mapping is conceptually the inverse of the tables above, with additional target-specific formatting, code conversions, and optional fields populated as required by each partner.

---

## 9. Next steps in a real project

In a real implementation, this high-level mapping would be extended with:

- Exact XML/field paths for the SCRIPT version in use.  
- Detailed handling rules for all mandatory SCRIPT elements.  
- Tests to validate that representative NewRx messages map to FHIR Bundles (and back) correctly across partners.
