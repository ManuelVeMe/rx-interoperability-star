# 04 – FHIR Rx Bundle Profile

## 1. Overview

The Rx transaction is represented as a **FHIR `Bundle` of type `transaction`** that groups all resources needed to process a prescription end‑to‑end.

- The **Bundle** carries technical metadata (transaction ID, timestamp, sender/receiver IDs, correlation ID).
- Core clinical and business content is modeled using standard resources: `Patient`, `Practitioner`, `Organization`, `MedicationRequest`, `Coverage`, and an `Organization` for the dispensing pharmacy.
- Additional flags and legacy fields (consent, legacy status/error codes) are included via extensions or dedicated elements where appropriate.

This profile is intended to act as a **clear interface contract** between prescribing systems and the integration platform. It defines the minimum set of resources and fields that must be present for a valid Rx transaction, plus optional fields that are supported when available.

The goal is a pragmatic, readable profile, not a fully formal HL7 Implementation Guide.

---

## 2. Bundle structure

- **Resource**: `Bundle`
- **Type**: `transaction`
- **Contains** (minimum set for this use case):
  - 1 `Patient`
  - 1 `Practitioner` (prescriber)
  - 1 `Organization` (prescriber organization)
  - 1 `MedicationRequest`
  - 0–1 `Coverage`
  - 1 `Organization` (pharmacy) and/or `Location` for dispensing site

### Bundle-level fields (technical mapping)

- Transaction ID → `Bundle.id`
- Message timestamp → `Bundle.timestamp`
- Sending system ID → `Bundle.meta.source` (or extension)
- Receiving system ID → extension on `Bundle`
- Correlation ID → extension on `Bundle` (e.g., `correlation-id`)

Legacy status/error codes can be captured as:
- Extensions on `Bundle`, or
- Separate `OperationOutcome` resources linked to the Bundle (for error scenarios).

---

## 3. Patient mapping and contract

**Resource**: `Patient`

Sample field mapping:

- Patient internal ID → `Patient.identifier` (type = internal, system = enterprise or local ID)
- Patient MRN → `Patient.identifier` (type = MRN, system = hospital ID domain)
- First name → `Patient.name.given`
- Last name → `Patient.name.family`
- Date of birth → `Patient.birthDate`
- Sex → `Patient.gender`
- Phone number → `Patient.telecom` (system = phone)
- Address → `Patient.address` (line, city, postalCode, country)

`MedicationRequest.subject` references this `Patient`.

### Patient – key elements (MVP)

| Element                    | Description             | Requirement |
|---------------------------|-------------------------|------------|
| Patient.identifier (MRN/internal) | Patient IDs      | Required   |
| Patient.name              | First/last name         | Required   |
| Patient.birthDate         | Date of birth           | Required   |
| Patient.gender            | Administrative gender   | Required   |
| Patient.telecom (phone)   | Contact phone           | Optional   |
| Patient.address           | Postal address          | Optional   |

---

## 4. Prescriber and organization mapping and contract

**Resource**: `Practitioner` (prescriber)

- Prescriber ID → `Practitioner.identifier` (internal/system-specific ID)
- Prescriber NPI → `Practitioner.identifier` (type = NPI, where applicable)
- Prescriber name → `Practitioner.name` (given/family)

**Resource**: `Organization` (prescriber organization)

- Prescriber organization → `Organization.name` and `Organization.identifier` as appropriate.

In `MedicationRequest`:

- `MedicationRequest.requester` → reference to `Practitioner`
- `MedicationRequest.requester.onBehalfOf` → reference to prescriber `Organization` (if needed)

### Practitioner – key elements (MVP)

| Element                          | Description             | Requirement |
|----------------------------------|-------------------------|------------|
| Practitioner.identifier (ID/NPI) | Prescriber identifiers  | Required   |
| Practitioner.name                | Prescriber name         | Required   |

### Prescriber Organization – key elements (MVP)

| Element                 | Description               | Requirement |
|-------------------------|---------------------------|------------|
| Organization.name       | Prescriber organization   | Required   |
| Organization.identifier | Org identifier (if used)  | Optional   |

---

## 5. MedicationRequest mapping and contract

**Resource**: `MedicationRequest`  
Core clinical order for the prescription.

Key fields:

- Prescription number → `MedicationRequest.identifier` (type = prescription number)
- Prescription status → `MedicationRequest.status` (e.g., active, completed, cancelled)
- Order priority → `MedicationRequest.priority` (e.g., routine, urgent)

Medication details:

- Medication code → `MedicationRequest.medicationCodeableConcept.coding.code`
- Medication display name → `MedicationRequest.medicationCodeableConcept.text` (and/or `coding.display`)
- Dose → `MedicationRequest.dosageInstruction.doseAndRate.doseQuantity.value`
- Dose unit → `MedicationRequest.dosageInstruction.doseAndRate.doseQuantity.unit`
- Route → `MedicationRequest.dosageInstruction.route`
- Frequency → `MedicationRequest.dosageInstruction.timing`
- Duration → `MedicationRequest.dispenseRequest.expectedSupplyDuration`
- Quantity → `MedicationRequest.dispenseRequest.quantity.value`
- Quantity unit → `MedicationRequest.dispenseRequest.quantity.unit`
- Number of refills → `MedicationRequest.dispenseRequest.numberOfRepeatsAllowed`
- Substitution allowed flag → `MedicationRequest.substitution.allowedBoolean`

Clinical context:

- Diagnosis code → `MedicationRequest.reasonCode`
- Allergy summary text → `MedicationRequest.note.text` (or separate resources in future)
- Clinical notes → `MedicationRequest.note.text` (can be distinct from allergy summary)

Timing:

- Fulfillment requested date → `MedicationRequest.dispenseRequest.validityPeriod.start` (or `authoredOn` if more appropriate)

Consent:

- Consent flag → extension on `MedicationRequest` or reference to a `Consent` resource.

Legacy fields:

- Legacy status code → extension on `MedicationRequest` (or `Bundle`)
- Error / exception code → returned as `OperationOutcome` for error responses; extensions only if legacy tracking is needed on successful flows.

### MedicationRequest – key elements (MVP)

| Element                                      | Description                 | Requirement |
|---------------------------------------------|-----------------------------|------------|
| MedicationRequest.identifier (Rx number)    | Prescription number         | Required   |
| MedicationRequest.status                    | Current Rx status           | Required   |
| MedicationRequest.priority                  | Order priority              | Optional   |
| MedicationRequest.medicationCodeableConcept | Medication code + display   | Required   |
| MedicationRequest.subject                   | Reference to Patient        | Required   |
| MedicationRequest.requester                 | Reference to Practitioner   | Required   |
| MedicationRequest.dispenseRequest.quantity  | Quantity + unit             | Required   |
| MedicationRequest.dispenseRequest.numberOfRepeatsAllowed | Refills     | Optional   |
| MedicationRequest.substitution.allowedBoolean | Substitution allowed      | Optional   |
| MedicationRequest.reasonCode                | Diagnosis/indication        | Optional   |
| MedicationRequest.note                      | Notes / allergy summary     | Optional   |
| MedicationRequest.dispenseRequest.validityPeriod.start | Requested date | Optional   |

> **Note on allergy summary (field 32):** For MVP, allergy summary text is captured as free text in `MedicationRequest.note`. The semantically correct FHIR resource for structured allergy data is `AllergyIntolerance`, referenced from the Bundle. This is intentionally deferred to Phase 2, when structured allergy data is reliably available from prescribing systems and the added complexity of an additional resource type is justified.
---

## 6. Coverage and payer mapping and contract

**Resource**: `Coverage`

- Payer / insurance ID → `Coverage.identifier` or `Coverage.subscriberId`
- Coverage plan name → `Coverage.class.name` or `Coverage.type.text`
- Payer organization → `Coverage.payor` referencing an `Organization`

`MedicationRequest.insurance` references the `Coverage` resource to link the order to payer information.

### Coverage – key elements (MVP)

| Element                 | Description              | Requirement |
|-------------------------|--------------------------|------------|
| Coverage.identifier     | Payer/insurance ID       | Optional   |
| Coverage.subscriberId   | Subscriber/member ID     | Optional   |
| Coverage.type / class   | Plan / product name      | Optional   |
| Coverage.payor          | Reference to payer org   | Optional   |

(For MVP, coverage is useful but not strictly required for all flows; requirements may vary by market.)

---

## 7. Pharmacy / fulfillment mapping and contract

**Resource**: `Organization` (pharmacy)

- Pharmacy ID → `Organization.identifier`
- Pharmacy name → `Organization.name`

`MedicationRequest.dispenseRequest.performer` references the pharmacy `Organization`.

Optionally, a `Location` resource can capture the physical dispensing site.

### Pharmacy Organization – key elements (MVP)

| Element                 | Description          | Requirement |
|-------------------------|----------------------|------------|
| Organization.identifier | Pharmacy identifier | Required   |
| Organization.name       | Pharmacy name       | Required   |

---

## 8. Technical and correlation fields

The remaining technical fields are handled as follows:

- Transaction ID → `Bundle.id`
- Message timestamp → `Bundle.timestamp`
- Sending system ID → `Bundle.meta.source` (or extension)
- Receiving system ID → extension on `Bundle`
- Correlation ID → extension on `Bundle`

These identifiers are reused in analytics and audit events emitted by Rhapsody so that every processing step can be traced back to the original Rx transaction.

---

## 9. Profile scope and simplifications

This Bundle profile is intentionally **minimal but complete** for the case study:

- It focuses on mapping the provided fields into widely used FHIR resources and clearly marks **which elements are required for MVP vs optional**.
- It does not attempt to formalize all value sets, code systems, or cardinalities as a full HL7 Implementation Guide.
- In a real project, this profile would be:
  - Expressed as a set of formal FHIR profiles (StructureDefinitions, ValueSets, etc.).
  - Supported by conformance statements, examples, and automated validation rules.

Example JSON instances for a “happy path” Rx and an error scenario are provided separately in the `fhir/` directory.
