# 05 – Assumptions and Risks

## 1. Key assumptions

### 1.1 Business and workflow assumptions

- The core business workflows remain unchanged: which prescriptions are created, which fulfillment platforms handle them, and which validations/audits are required.  
- The primary focus is on new prescriptions and basic status updates; change/cancel flows and advanced specialty workflows are handled later.  
- External partners (hospitals, pharmacies, HUBs) continue to own their internal workflows, DWHs, and audit trails; the enterprise only manages its own analytics/audit.

### 1.2 Technical and integration assumptions

- There are multiple heterogeneous prescribing systems (e.g., at least one hospital EHR and one ambulatory/telehealth system) using a mix of HL7 v2, NCPDP SCRIPT, and, in some cases, FHIR.  
- There are multiple fulfillment platforms (e.g., internal hospital pharmacy, external retail/mail-order, and possibly a HUB), many of which will continue to expect legacy formats in the near term.  
- Rhapsody can be self-hosted and sized to handle current and near‑term expected volumes and latency targets.  
- Rhapsody provides adapters for HL7 v2, REST/JSON, and other relevant protocols, enabling canonicalization to FHIR Bundles.  
- A baseline set of reference data/value sets (e.g., medication codes, pharmacy IDs, payer IDs) is available or can be established for semantic validation.

### 1.3 Organizational and delivery assumptions

- There is a product owner / business sponsor empowered to prioritize MVP scope and later phases.  
- There is a cross‑functional team (product, architecture, integration, data, compliance) available to design and govern validation rules and routing policies.  
- Existing analytics and audit platforms can consume standardized Rx event feeds from Rhapsody without major re‑platforming in MVP.  
- Partner contracts and regulations allow the introduction of FHIR-based APIs at the enterprise boundary, even if partners initially continue to use legacy formats.  
- Licensing for core terminology assets (clinical codes, drug databases) allows their use within the enterprise integration and analytics platforms; any redistribution to external partners will be reviewed separately.

---

## 2. Design trade‑offs

To keep scope controlled and aligned with MVP, the following trade‑offs are assumed:

- Canonicalizing on FHIR internally even though many systems still speak HL7 v2 or SCRIPT.  
- Centralizing core validation in Rhapsody, while leaving advanced clinical decision support in prescribing systems.  
- Focusing analytics/audit integration on enterprise platforms, not on partner DWHs.  
- Preserving existing fulfillment systems and interfaces in MVP instead of re‑platforming them.

These trade‑offs can be revisited in later phases as adoption and constraints evolve.

---

## 3. Key risks and mitigations

### 3.1 Technical risks

**Risk 1 – Complexity of HL7 v2 / SCRIPT → FHIR mapping**

- Description: Mapping diverse, sometimes inconsistent legacy messages into a clean FHIR Rx Bundle may be more complex than anticipated.  
- Impact: Longer implementation time, inconsistent mappings, difficult troubleshooting.  
- Mitigation:
  - Start with a limited set of high‑volume flows for MVP and iterate.  
  - Define mapping guidelines and test cases early.  
  - Use automated validation and regression tests on mappings.

**Risk 2 – Performance and scalability of the central Rhapsody hub**

- Description: The new architecture introduces a single, central integration hub, which could become a bottleneck if not sized/scaled correctly.  
- Impact: Latency, throughput issues, or downtime affecting prescription routing.  
- Mitigation:
  - Define clear non‑functional requirements (throughput, latency, availability) early.  
  - Design for horizontal scalability and high availability of Rhapsody.  
  - Implement monitoring, alerting, and load testing before go‑live.

**Risk 3 – Limited or inconsistent reference data for semantic validation**

- Description: Medication codes, diagnosis codes, pharmacy IDs, and payer IDs may be inconsistent across systems.  
- Impact: High validation error rates; need to relax checks, reducing value of semantic validation.  
- Mitigation:
  - Start with a small, pragmatic set of enforced value sets.  
  - Align with existing master data / terminology management where available.  
  - Treat semantic validation as configurable, allowing progressive tightening.

---

### 3.2 Business and change‑management risks

**Risk 4 – Partner systems’ readiness for FHIR and REST**

- Description: Some partners may be slow to adopt FHIR or may not support new APIs in the near term.  
- Impact: Longer coexistence with legacy formats; potential friction integrating new flows.  
- Mitigation:
  - Design Rhapsody as a translator at the edge, keeping FHIR internal and supporting legacy formats for key partners.  
  - Use phased onboarding with clear timelines and incentives for partners to move toward FHIR interfaces.

**Risk 5 – Misalignment on validation rules and “golden source”**

- Description: Different stakeholders (IT, compliance, clinical, commercial) may disagree on which validations belong where and what is the golden source of truth.  
- Impact: Conflicting rules, duplicated efforts, or unsafe/over‑lenient validation.  
- Mitigation:
  - Establish a cross‑functional validation/standards working group.  
  - Document validation policies and ownership clearly (e.g., structural vs business vs clinical).  
  - Start with a minimal, agreed baseline for MVP and evolve based on feedback.

**Risk 6 – Underestimating effort to rationalize legacy middleware**

- Description: Turning off or simplifying legacy middleware may require more analysis and migration work than anticipated.  
- Impact: Extended dual‑running of old and new flows, higher operational burden.  
- Mitigation:
  - Plan for coexistence during migration, with clear cutover criteria per flow.  
  - Prioritize decommissioning of highest cost / lowest value middleware components first.  
  - Track migration progress per interface as part of the roadmap.

---

### 3.3 Regulatory, data‑governance, and licensing risks

**Risk 7 – Regulatory requirements not fully captured in early design**

- Description: Some regulatory or contractual requirements (e.g., consent handling, retention, cross‑border data flows) may be missed initially.  
- Impact: Rework, compliance gaps, or project delays.  
- Mitigation:
  - Involve compliance and legal stakeholders early in requirements and validation design.  
  - Treat consent, audit, and retention as explicit requirements in the MVP.  
  - Use configurable policies where regulations differ by market.

**Risk 8 – Data quality and lineage concerns**

- Description: Stakeholders may question the reliability of analytics and audit data if lineage is not clear.  
- Impact: Low trust in the new platform; continued reliance on legacy reports and workarounds.  
- Mitigation:
  - Ensure end‑to‑end correlation IDs and event logs are part of the design.  
  - Provide simple lineage views (e.g., which steps a transaction passed, with timestamps).  
  - Engage data/analytics teams to validate that the new event model meets their needs.

**Risk 9 – Licensing and IP constraints on code systems**

- Description: Some value sets and code systems (e.g., diagnosis codes, clinical terminologies, commercial drug databases) may be subject to licensing and IP restrictions, limiting how they can be replicated or exposed across platforms.  
- Impact: Central semantic validation and shared terminology services may be constrained; additional work may be needed to ensure compliant use and avoid duplication of licenses.  
- Mitigation:
  - Involve legal/procurement when defining terminology sources and shared services.  
  - Prefer using existing, licensed terminology services where possible instead of bulk copying code sets.  
  - Scope semantic validation in MVP to what is legally and contractually allowed, and expand later as agreements permit.

---

## 4. Summary

These assumptions and risks are explicitly documented to make trade‑offs transparent and to guide MVP scope and roadmap decisions. In a real project, they would be refined in early discovery and revisited at each major phase gate.
