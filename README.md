# HealthTech Rx Interoperability – Star Case Study

This repository contains my solution to a HealthTech Interoperability Product Manager case for Star. It proposes a practical target architecture for a drug prescription (Rx) workflow using FHIR, RESTful APIs, and a single Rhapsody integration engine as the main interoperability platform.

The goal is to preserve the critical business behavior of the current legacy landscape while simplifying middleware, clarifying interface contracts, and making future onboarding of systems easier.

---

## Repository structure

- `docs/`
  - `00-executive-summary.md` – One-page overview of the problem, goals, key design decisions, and MVP scope.
  - `01-fhir-integration-documentation-approach.md` – Proposed structure for FHIR integration documentation to support migration, governance, onboarding, and future adoption.
  - `02-as-is-overview.md` – Description of the current 6-step prescription workflow, assumptions, and main pain points.
  - `03-target-architecture-and-flow.md` – To-be architecture with Rhapsody as the central integration hub, including Rx transaction flow and validation approach.
  - `04-fhir-rx-bundle-profile.md` – Proposed FHIR Bundle profile for the prescription transaction, mapping sample fields to FHIR resources.
  - `05-api-spec-and-error-handling.md` – RESTful interaction style, endpoints, validation gates, and negative/error flow behavior.
  - `06-assumptions-and-risks.md` – Key assumptions, constraints, and project risks highlighted for stakeholders.
  - `07-mvp-and-roadmap.md` – MVP vs later-phase scope and a simple phased rollout view.
  - `08-notes-and-future-work.md` – Ideas for how this would evolve into a full implementation guide and delivery plan.

- `diagrams/`
  - `as-is-prescription-flow.puml` / `.png` – As-is system diagram of the current Rx message path (multiple sources → legacy middleware → integration engine → downstream platforms).
  - `to-be-rx-flow-rhapsody-fhir.puml` / `.png` – To-be architecture with a FHIR/Rhapsody-centric integration layer, validation, routing, and analytics/audit exports.
  - `rx-sequence-flow.png` (optional) – High-level sequence diagram for a single Rx from prescribing system to fulfillment and back.

- `fhir/`
  - `rx-bundle-example-1.json` – Example “happy path” Rx FHIR transaction Bundle with the key resources (Patient, Practitioner, MedicationRequest, Coverage, Pharmacy, etc.).
  - `rx-bundle-example-error.json` – Example invalid submission and corresponding error representation.
  - `value-sets-and-codes.md` – Assumed code systems/value sets (medication codes, diagnosis, payer/coverage, pharmacy IDs, statuses).

- `misc/`
  - `references.md` – References to relevant standards and materials (FHIR, Rhapsody, interoperability notes).

---

## How to read this case

If you only have a few minutes:

1. Start with `docs/00-executive-summary.md` for the high-level problem framing, goals, and proposed direction.
2. Look at the diagrams:
   - `diagrams/as-is-prescription-flow.png`
   - `diagrams/to-be-rx-flow-rhapsody-fhir.png`
3. Skim `docs/02-target-architecture-and-flow.md` for the core architecture and Rx API flow.

If you want more detail:

- See `docs/03-fhir-rx-bundle-profile.md` for how the sample fields are mapped into a FHIR Bundle.
- See `docs/04-api-spec-and-error-handling.md` for REST interaction style, validation layers, and error handling.
- See `docs/05-assumptions-and-risks.md` and `docs/06-mvp-and-roadmap.md` for delivery considerations.

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

