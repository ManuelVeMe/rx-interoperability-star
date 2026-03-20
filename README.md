# HealthTech Rx Interoperability – Star Case Study

This repository contains my solution to a HealthTech Interoperability Product Manager case for Star. It proposes a practical target architecture for a drug prescription (Rx) workflow using FHIR, RESTful APIs, and a single Rhapsody integration engine as the main interoperability platform.

The goal is to preserve the critical business behavior of the current legacy landscape while simplifying middleware, clarifying interface contracts, and making future onboarding of systems easier.

---

## Mapping to Test Tasks

This repository is structured to explicitly address each part of the test:

1. **Simplified architecture and middleware responsibilities**  
   → `03-target-architecture-and-flow.md`

2. **FHIR integration documentation approach**  
   → `01-fhir-integration-documentation-approach.md`

3. **Rx FHIR Bundle profiling**  
   → `04-fhir-rx-bundle-profile.md`

4. **Target API and transaction flow (sync/async, validation, negative flows)**  
   → `03-target-architecture-and-flow.md`, `05-api-spec-and-error-handling.md`

5. **Assumptions and risks**  
   → `06-assumptions-and-risks.md`

6. **MVP scope and future roadmap**  
   → `07-mvp-and-roadmap.md`
   
## Repository structure

- `docs/`
  - `00-executive-summary.md` – High-level overview and key decisions.
  - `01-fhir-integration-documentation-approach.md` – Documentation structure for FHIR integration, governance, and onboarding.
  - `02-as-is-overview.md` – Current state and legacy middleware landscape.
  - `03-target-architecture-and-flow.md` – Target architecture, validation, routing, and end-to-end Rx flow.
  - `04-fhir-rx-bundle-profile.md` – Canonical FHIR Rx transaction definition.
  - `05-api-spec-and-error-handling.md` – API design, validation behavior, and error handling.
  - `06-assumptions-and-risks.md` – Key assumptions and identified risks.
  - `07-mvp-and-roadmap.md` – MVP scope and future evolution.
  - `08-notes-and-future-work.md` – Additional considerations and extensions.

- `diagrams/`
  - `as-is-prescription-flow.puml` / `.png` – As-is system diagram of the current Rx message path (multiple sources → legacy middleware → integration engine → downstream platforms).
  - `to-be-rx-flow-rhapsody-fhir.puml` / `.png` – To-be architecture with a FHIR/Rhapsody-centric integration layer, validation, routing, and analytics/audit exports.
  - `rx-sequence-flow.puml` / `.png` – High-level sequence diagram for a single Rx from prescribing system to fulfillment and back.

- `fhir/`
  - `rx-bundle-example-1.json` – Example “happy path” Rx FHIR transaction Bundle with the key resources (Patient, Practitioner, MedicationRequest, Coverage, Pharmacy, etc.).
  - `rx-bundle-example-error.json` – Example invalid submission and corresponding error representation.
  - `value-sets-and-codes.md` – Assumed code systems/value sets (medication codes, diagnosis, payer/coverage, pharmacy IDs, statuses).

- `misc/`
  - `references.md` – References to relevant standards and materials (FHIR, Rhapsody, interoperability notes).

---

## How to read this case

If you want a quick overview:
- Start with [`00-executive-summary.md`](./00-executive-summary.md)
- Then read [`03-target-architecture-and-flow.md`](./03-target-architecture-and-flow.md) for the end-to-end design

If you want to understand the interoperability model:
- See [`01-fhir-integration-documentation-approach.md`](./01-fhir-integration-documentation-approach.md)
- Then [`04-fhir-rx-bundle-profile.md`](./04-fhir-rx-bundle-profile.md)

If you want implementation details:
- See [`05-api-spec-and-error-handling.md`](./05-api-spec-and-error-handling.md)

For product and delivery considerations:
- See [`06-assumptions-and-risks.md`](./06-assumptions-and-risks.md)
- See [`07-mvp-and-roadmap.md`](./07-mvp-and-roadmap.md)

---

## Design highlights

- **Simplified middleware landscape**  
  Consolidates legacy middleware responsibilities into a single Rhapsody-based integration layer with clearly separated concerns (validation, routing, analytics/audit export).

- **FHIR-first Rx transaction**  
  Uses a transaction `Bundle` as the primary prescription message, combining Patient, Prescriber, MedicationRequest, Coverage, and Pharmacy context to support both clinical workflow and supply/demand analytics.

- **Clear validation and error model**  
  Distinguishes structural (FHIR), business, and semantic/terminology validations with explicit failure behaviors before routing to downstream fulfillment platforms.

- **MVP-focused**  
  Prioritizes a minimal but complete end-to-end Rx flow between a small set of prescribing systems and fulfillment platforms, with analytics/audit feeds, while deferring more advanced scenarios to later phases.

---

## Notes

- FHIR examples are intentionally simplified and focus on structure and responsibilities rather than perfect conformance to any specific national implementation guide.
- This is an individual case-study deliverable, not an official Star project, but it is structured so that a real delivery team could extend it into a full implementation guide and backlog.

