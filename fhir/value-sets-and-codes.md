# Value Sets and Codes (Rx Flow)

## 1. Purpose and scope

This document lists **key value sets and code systems** assumed in the Rx Bundle and validation rules for the MVP.

- Scope: only the most important domains (medication, diagnosis, pharmacy ID, payer/coverage, statuses).  
- Non‑goals: exhaustive terminology design or binding; full licensing/legal details.

In a real project, these would be formalized as FHIR `CodeSystem` and `ValueSet` resources and governed by terminology services.

---

## 2. Medication codes

### 2.1 Code systems

Depending on market and existing systems, medication codes may include:

- National or regional drug codes (e.g., NDC in US, AEMPS codes in Spain, other national catalogs).  
- Internal enterprise drug catalog codes.  

For this case, we assume:

- A primary **enterprise medication code system**, e.g.:  
  - `system = "urn:drug:code"`  
- Optionally, additional codings for national codes (e.g., `urn:ndc` if relevant).

### 2.2 Validation (MVP)

- For MVP, structural validation checks that:
  - `MedicationRequest.medicationCodeableConcept.coding.code` is present.  
  - The `system` is one of the allowed medication code systems.  
- Semantic validation may start with:
  - Ensuring codes are **syntactically valid** (format checks).  
  - Optionally checking against an internal drug catalog if available.

---

## 3. Diagnosis / indication codes

### 3.1 Code systems

Typical code systems for diagnosis / indication:

- **ICD‑10 / ICD‑11** (depending on region).  
- SNOMED CT or other clinical terminologies where licensed.

In this case, we assume:

- Diagnosis codes in `MedicationRequest.reasonCode` primarily use an ICD‑10‑like system, e.g.:  
  - `system = "urn:icd10"`

### 3.2 Validation (MVP)

- MVP enforces:
  - Presence of a diagnosis code only where business rules require it (e.g., for certain drug classes).  
  - Basic format checks for codes (e.g., allowed pattern).  
- Full semantic validation (e.g., verifying codes against a terminology service) is a **later-phase enhancement**, subject to licensing/IP constraints.

---

## 4. Pharmacy identifiers

### 4.1 Code system

Pharmacy IDs must be consistent for routing purposes.

- Assume an enterprise pharmacy ID domain:  
  - `system = "urn:pharmacy:id"`  
  - `Organization.identifier.system = "urn:pharmacy:id"`  

These IDs may be mapped to:

- Internal hospital pharmacy systems.  
- External retail/mail-order or HUB systems via lookup tables.

### 4.2 Validation (MVP)

- MVP checks that:
  - `Organization.identifier` for the pharmacy is present and non-empty.  
  - The ID is recognized in a **simple reference table** of known pharmacies used by routing rules.

Unrecognized IDs can either:

- Cause a validation error (blocking), or  
- Be flagged for manual review, depending on policy.

---

## 5. Payer / coverage codes

### 5.1 Code systems

Coverage and payer information can be coded in multiple ways:

- Payer IDs (internal or external)  
- Plan/product codes  
- Member/subscriber IDs

For MVP we assume:

- Coverage identifiers use an enterprise payer ID domain, e.g.:  
  - `Coverage.identifier.system = "urn:payer:id"`  
- Plan types may be free text (`Coverage.type.text`) or simple internal codes.

### 5.2 Validation (MVP)

- MVP checks:
  - Structural presence of coverage only when required for specific flows or partners.  
  - Basic format/lookup of payer IDs if a reference list is available.

Deeper eligibility checks remain with payer/HUB systems in MVP.

---

## 6. Rx status and priority

### 6.1 MedicationRequest.status

Proposal for MVP mapping:

- Allowed FHIR statuses for this flow:
  - `active` – active prescription  
  - `completed` – completed/fulfilled or expired prescriptions (depending on business rules)  
  - `cancelled` – cancelled prescriptions  
  - `entered-in-error` – erroneous entries

Mapping from legacy statuses (HL7 v2 ORC-1, SCRIPT status codes) should be defined in the integration layer.

### 6.2 MedicationRequest.priority

- Suggested priority codes:
  - `routine`  
  - `urgent`  
  - `stat`  

In MVP, this field is **optional** and primarily used where existing systems already express priority.

---

## 7. Consent and flags

### 7.1 Consent flag

- Modeled as an extension on `MedicationRequest` (or as a reference to a `Consent` resource in later phases).  
- For MVP:
  - `extension.url = "http://example.org/fhir/StructureDefinition/consent-flag"`  
  - `valueBoolean = true/false`

Validation:

- For certain flows or regions, consent may be **required**; if missing, the transaction fails validation.

### 7.2 Legacy status/error codes

- Legacy codes can be preserved as extensions on `Bundle` or `MedicationRequest` if needed for traceability.  
- There is no attempt to standardize these; they are treated as **opaque values** for analytics/ops.

---

## 8. Licensing and terminology services (note)

Some code systems and drug databases (e.g., ICD, SNOMED CT, commercial drug catalogs) are subject to **licensing and IP restrictions**. For the MVP:

- The design assumes **licensed use within the enterprise integration and analytics platforms**.  
- Redistribution or exposure of full code sets to external partners is out of scope and would require legal review.  
- Where possible, the enterprise should rely on existing, licensed terminology services rather than bulk copying value sets.

---

## 9. Next steps in a real project

In a real implementation, this high-level view would be extended by:

- Creating formal FHIR `CodeSystem` and `ValueSet` definitions for each domain.  
- Integrating with a terminology service or master data hub.  
- Defining enforcement levels (required / warning / none) per element and per use case.  
- Aligning value sets and code systems with regulatory requirements in each target market.
