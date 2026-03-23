# 07 – MVP and Roadmap

## 1. MVP scope

The MVP focuses on delivering a **single, reliable end‑to‑end Rx flow** across a limited but representative set of systems, proving the new architecture and contracts without boiling the ocean.

### 1.1 Systems in scope (MVP)

- **Prescribing systems**
  - 1 hospital EHR (internal inpatient/outpatient prescribing).
  - 1 additional prescribing channel (e.g., ambulatory or telehealth).

- **Fulfillment platforms**
  - 1 internal hospital pharmacy platform.
  - 1 external fulfillment/HUB or external pharmacy platform.

- **Enterprise data platforms**
  - Existing enterprise analytics/data platform (DWH/lake).
  - Existing enterprise compliance/audit store.

### 1.2 Functional scope (MVP)

- **Rx submission and validation**
  - Accept Rx transactions as FHIR Bundles over REST and via adapters for HL7 v2/SCRIPT where needed.
  - Apply structural and core business validation in Rhapsody before routing.
  - Return clear validation errors synchronously to the caller.

- **Routing and delivery**
  - Route validated prescriptions to the in-scope hospital pharmacy and external fulfillment/HUB.
  - Perform necessary format transformations (e.g., FHIR → HL7 v2 or SCRIPT) for those targets.
  - Implement basic retry and dead‑letter handling for downstream delivery failures.

- **Status and visibility**
  - Provide `transactionId` on acceptance and a simple `GET /rx-status/{transactionId}` endpoint.
  - Capture key status changes from downstream systems and surface them via the status API.

- **Analytics and audit**
  - Publish standardized Rx events (submitted, validated, routed, delivered/failed) to enterprise analytics and audit platforms.
  - Ensure end‑to‑end traceability via correlation IDs and timestamps.

### 1.3 Non‑functional scope (MVP)

- Availability and latency targets consistent with current expectations for the in‑scope flows (e.g., near real‑time routing, seconds not minutes).  
- Basic monitoring: key metrics (volumes, error rates, latency) and alerting on failures for in‑scope interfaces.
- Rhapsody high-availability design (active/passive clustering or equivalent) is a hard requirement for MVP — a single Rhapsody instance is not acceptable for production prescription routing.

---

### 1.4 MVP definition of done

The MVP is considered complete when all of the following criteria are met across the in-scope systems (2 prescribing sources, 1 internal hospital pharmacy, 1 external HUB).

**Functional criteria**

- 100% of the 42 Rx Bundle fields from the agreed profile are mapped and exercised in end-to-end test runs for both the HL7 v2 and SCRIPT input paths.
- All three validation layers (structural, business, semantic core) are active and returning structured OperationOutcome-aligned error responses for known invalid inputs.
- Validated prescriptions are routed correctly to both fulfillment targets with the correct format transformation (FHIR → HL7 v2 for the hospital pharmacy; FHIR → SCRIPT or target API for the HUB).
- Every Rx event (submitted, validated, routed, delivered, failed) is traceable end-to-end via correlation ID in the enterprise audit store.
- `GET /rx-status/{transactionId}` reflects accurate, up-to-date status for all in-scope targets within 30 seconds of a downstream status change.

**Non-functional criteria**

- Validation response (`POST /rx-bundles`): p95 latency ≤ 2 seconds under representative load.
- End-to-end routing to fulfillment: p95 ≤ 10 seconds for the happy path.
- Rhapsody availability: ≥ 99.5% measured over a 30-day pilot window.
- Dead-letter queue processing: all delivery failures are surfaced in the status API and audit log within 5 minutes of retry exhaustion.

**Partner and operational criteria**

- Both prescribing system teams and both fulfillment platform teams have signed off on their respective interface test results.
- At least 500 real or realistic synthetic Rx transactions have been processed in the staging environment without a data-loss or silent-failure incident.
- On-call runbook covers the three most critical failure modes: Rhapsody unavailability, downstream delivery failure, and validation spike (abnormal rejection rate).
- Monitoring dashboard is live and alerting is configured for error rate, latency, and dead-letter queue depth.

---

**Go / no-go gate (Phase 0 → Phase 1)**

Before build starts, the following must be confirmed in writing:

| Gate item | Owner |
|---|---|
| Rx Bundle profile v1.0 agreed by clinical, IT, and compliance | Product owner |
| Minimum validation rule set approved | Cross-functional working group |
| Both prescribing system teams committed to integration timeline | Programme manager |
| Both fulfillment platforms have confirmed test environment access | Integration lead |
| Non-functional requirements signed off (latency, availability) | Architecture + ops |
| Regulatory / consent requirements confirmed for in-scope markets | Compliance lead |

If any gate item is unresolved at Phase 0 close, Phase 1 start date moves — scope does not shrink to compensate.

---

## 2. Out‑of‑scope for MVP (later phases)

The following capabilities are explicitly deferred beyond MVP to keep scope manageable:

- **Broader system coverage**
  - Additional prescribing systems and regions.
  - Additional pharmacies, including full retail/mail‑order network coverage.
  - Additional HUBs and specialty pharmacies beyond the initial partners.

- **Extended Rx lifecycle operations**
  - Change/modify prescription flows (dose changes, pharmacy change, etc.).
  - Cancel prescription flows and corresponding downstream notifications.
  - Advanced refill/renewal workflows beyond basic status handling.

- **Richer API surface**
  - Search/query APIs across prescriptions (by patient, prescriber, date range, status).
  - Retrieval of original/canonical Bundles for troubleshooting via dedicated endpoints.
  - More granular FHIR resource-level APIs beyond the transaction Bundle.

- **Advanced validation and decision support**
  - Deep semantic validation using comprehensive terminology services across all codes.
  - Embedded clinical decision support; these remain in prescribing systems and CDS services initially.

- **Partner DWH integration**
  - Direct updates to hospital/pharmacy DWHs; partners remain responsible for their own data platforms.
  - Standardized outbound data products for partners (beyond existing reporting channels).

- **Platform modernization beyond integration**
  - Re‑platforming of existing analytics, audit, or fulfillment systems.
  - Major changes to upstream clinical workflows.

---

## 3. Roadmap (phased view)

### Phase 0 – Discovery and alignment

- Validate assumptions about in‑scope systems, standards, and current flows.  
- Agree on the Rx Bundle profile and minimum validation rules.  
- Confirm non‑functional requirements and regulatory constraints.

### Phase 1 – MVP implementation (as defined above)

- Implement Rhapsody-based FHIR/REST ingress and adapters for in‑scope legacy formats.  
- Build validation, routing, and analytics/audit export for the selected flows.  
- Integrate with one hospital EHR, one additional prescribing channel, one internal pharmacy, and one external fulfillment/HUB.  
- Conduct integration and end‑to‑end testing, including negative/error flows.

### Phase 2 – Expansion of coverage

- Onboard additional prescribing systems and pharmacies using the same patterns and contracts.  
- Extend routing logic and mappings for new targets and use cases.  
- Add more detailed semantic validation where reference data and licensing allow.

### Phase 3 – Lifecycle and API enrichment

- Introduce change/cancel Rx flows and richer status models.  
- Add search/query capabilities and controlled access to original/canonical Bundles.  
- Standardize and publish a more complete Rx API surface and documentation (mini implementation guide).

### Phase 4 – Optimization and modernization

- Gradually transition more partners to native FHIR APIs where feasible.  
- Enhance analytics/audit event models and explore unified Rx data products for supply‑and‑demand planning.  
- Decommission remaining legacy middleware components as flows are migrated.

---

## 4. Value drivers per phase

- **MVP (Phase 1)** – Reduce architectural complexity for high‑value flows, improve validation consistency, and gain end‑to‑end visibility while preserving existing business behavior.  
- **Expansion (Phase 2)** – Scale benefits across more systems and channels, reusing the same contracts and patterns.  
- **Enrichment (Phase 3)** – Improve clinical and operational flexibility with richer lifecycle support and APIs.  
- **Optimization (Phase 4)** – Lower long‑term integration cost, improve data quality for supply/demand analytics, and simplify the overall integration landscape.

This phased roadmap keeps the initial delivery focused while making it clear how the design supports longer‑term evolution.
