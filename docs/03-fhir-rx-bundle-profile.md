# 03 – FHIR Rx Bundle Profile

## 1. Overview

The Rx transaction is represented as a **FHIR `Bundle` of type `transaction`** that groups all resources needed to process a prescription end‑to‑end:

- The **Bundle** itself carries technical metadata (transaction ID, timestamp, sender/receiver IDs, correlation ID).  
- Core clinical and business content is modeled using standard resources: `Patient`, `Practitioner`, `Organization`, `MedicationRequest`, `Coverage`, and `Organization`/`Location` for the dispensing pharmacy.  
- Additional flags and legacy fields (consent, legacy status/error codes) are included via extensions or dedicated elements where appropriate.

The goal is a **pragmatic, readable profile**, not a fully formal HL7 IG.

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

### Bundle-level fields (mapping key technical fields)

- Transaction ID → `Bundle.id` (or a dedicated extension)  
- Message timestamp → `Bundle.timestamp`  
- Sending system ID → `Bundle.meta.source` (or extension)  
- Receiving system ID → extension on `Bundle` or target field in routing metadata  
- Correlation ID → extension on `Bundle` (e.g., `extension.url = "http://example.org/fhir/StructureDefinition/correlation-id"`)

Legacy status/error codes can be captured as:
- Extensions on `Bundle`, or  
- Separate `OperationOutcome` resources linked to the Bundle (for error scenarios).

---

## 3. Patient mapping

**Resource**: `Patient`

Sample field mapping:

- Patient internal ID → `Patient.identifier` (type = internal, system = enterprise MRN or internal ID)  
- Patient MRN → `Patient.identifier` (type = MRN, system = hospital ID domain)  
- First name → `Patient.name.given`  
- Last name → `Patient.name.family`  
- Date of birth → `Patient.birthDate`  
- Sex → `Patient.gender`  
- Phone number → `Patient.telecom` (system = phone, use = mobile/home)  
- Address → `Patient.address` (line, city, postalCode, country)

The Bundle’s `MedicationRequest.subject` will reference this `Patient`.

---

## 4. Prescriber and organization mapping

**Resource**: `Practitioner` (prescriber)

- Prescriber ID → `Practitioner.identifier` (internal/system-specific ID)  
- Prescriber NPI → `Practitioner.identifier` (type = NPI, system = national provider registry, where applicable)  
- Prescriber name → `Practitioner.name` (given/family)

**Resource**: `Organization` (prescriber organization)

- Prescriber organization → `Organization.name` and `Organization.identifier` as appropriate.

In `MedicationRequest`:

- `MedicationRequest.requester` → reference to `Practitioner`  
- `MedicationRequest.requester.onBehalfOf` → reference to prescriber `Organization` (if needed)

---

## 5. MedicationRequest mapping

**Resource**: `MedicationRequest`  
This is the core clinical order for the prescription.

Key fields:

- Prescription number → `MedicationRequest.identifier` (type = prescription number)  
- Prescription status → `MedicationRequest.status` (e.g., active, completed, cancelled)  
- Order priority → `MedicationRequest.priority` (e.g., routine, urgent)

Medication details:

- Medication code → `MedicationRequest.medicationCodeableConcept.coding.code`  
- Medication display name → `MedicationRequest.medicationCodeableConcept.text` (and/or `coding.display`)  
- Dose → `MedicationRequest.dosageInstruction.doseAndRate.doseQuantity.value`  
- Dose unit → `MedicationRequest.dosageInstruction.doseAndRate.doseQuantity.unit`  
- Route → `MedicationRequest.dosageInstruction.route` (codeable concept)  
- Frequency → `MedicationRequest.dosageInstruction.timing` (e.g., repeat.interval, repeat.frequency)  
- Duration → `MedicationRequest.dispenseRequest.expectedSupplyDuration`  
- Quantity → `MedicationRequest.dispenseRequest.quantity.value`  
- Quantity unit → `MedicationRequest.dispenseRequest.quantity.unit`  
- Number of refills → `MedicationRequest.dispenseRequest.numberOfRepeatsAllowed`  
- Substitution allowed flag → `MedicationRequest.substitution.allowedBoolean` (or CodeableConcept if needed)

Clinical context:

- Diagnosis code → `MedicationRequest.reasonCode` (CodeableConcept)  
- Allergy summary text → could be included as:
  - `MedicationRequest.note.text`, or  
  - Reference to `AllergyIntolerance` resources in a richer scenario (not modeled in detail here).  
- Clinical notes → `MedicationRequest.note.text`

Timing:

- Fulfillment requested date → `MedicationRequest.dispenseRequest.validityPeriod.start` (or `authoredOn` if appropriate)

Consent:

- Consent flag → extension on `MedicationRequest` or a related `Consent` resource reference, depending on implementation maturity.

Legacy fields:

- Legacy status code → extension on `MedicationRequest` (or Bundle)  
- Error / exception code → typically represented by an `OperationOutcome` when returning errors, rather than inside the successful order. For tracking legacy values, an extension can be used where needed.

---

## 6. Coverage and payer mapping

**Resource**: `Coverage`

- Payer / insurance ID → `Coverage.identifier` or `Coverage.subscriberId`  
- Coverage plan name → `Coverage.class.name` or `Coverage.type.text` (depending on data structure)  
- Payer organization (if available) → `Coverage.payor` referencing an `Organization`

`MedicationRequest.insurance` can reference the `Coverage` resource to link the order to payer information.

---

## 7. Pharmacy / fulfillment mapping

**Resource**: `Organization` (pharmacy)

- Pharmacy ID → `Organization.identifier` (pharmacy ID domain)  
- Pharmacy name → `Organization.name`

`MedicationRequest.dispenseRequest.performer` can reference the pharmacy `Organization` to indicate the intended dispensing site.

If physical site detail is important, a `Location` resource can be added and referenced from the pharmacy `Organization` or directly from `MedicationRequest.dispenseRequest.performer`.

---

## 8. Technical and correlation fields

The remaining technical fields are handled as follows:

- Transaction ID → `Bundle.id` (canonical transaction identifier)  
- Message timestamp → `Bundle.timestamp`  
- Sending system ID → `Bundle.meta.source` (or a dedicated extension)  
- Receiving system ID → extension on `Bundle` to indicate intended primary receiver; routing rules can interpret this.  
- Correlation ID → dedicated extension on `Bundle` (for tying together related messages/events across systems)

These identifiers are reused in analytics/audit events emitted by Rhapsody so that every stage of processing can be traced back to the original Rx transaction.

---

## 9. Profile scope and simplifications

This Bundle profile is intentionally **minimal but complete** for the test:

- It focuses on mapping the provided fields into widely used FHIR resources without defining every possible element or national constraint.  
- It does not attempt to formalize all value sets, code systems, or cardinalities as a full HL7 Implementation Guide would.  
- In a real project, this profile would be:
  - Expressed as a formal FHIR profile set (StructureDefinitions, ValueSets, etc.).  
  - Accompanied by conformance statements, examples, and automated validation rules.

Example JSON instances for a “happy path” Rx and an error scenario are provided separately in the `fhir/` directory.

