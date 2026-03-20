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
