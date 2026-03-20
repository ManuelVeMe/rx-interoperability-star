# 00 – Executive Summary

## Context

An enterprise pharma company relies on an outdated, supply‑and‑demand integration landscape for its drug prescription (Rx) workflows. A single prescription passes through multiple legacy middleware components for business logic, analytics/audit mirroring, compliance/validation, and routing before reaching downstream fulfillment platforms (hospital, retail, mail‑order, HUB, etc.). The company wants to preserve critical business behavior while modernizing the technical realization using FHIR, RESTful APIs, clearer interface contracts, and a single self‑hosted Rhapsody integration engine as the main platform.

## Proposed direction

This solution treats prescriptions as **FHIR-based Rx transaction Bundles** exchanged via RESTful APIs with a **Rhapsody‑centric integration hub** at the core.

- Rhapsody exposes FHIR REST endpoints for multiple prescribing systems (EHRs, telehealth, clinical apps), accepting HL7 v2 / NCPDP SCRIPT / FHIR inputs through adapters but canonicalizing to a single internal FHIR Rx Bundle model.  
- Within Rhapsody, we centralize structural, business, and basic semantic validations, apply routing and enrichment rules, and emit standardized events into the enterprise’s own analytics and audit platforms, removing the need for separate opaque middleware.  
- Rhapsody routes to downstream fulfillment platforms (hospital pharmacy, retail/mail‑order, HUB/specialty) in the formats they support today, while providing a path to progressively introduce FHIR-based APIs for new or modernized systems.

This preserves existing routing and validation intent while simplifying the architecture and creating a clear, future‑proof API and contract surface.

## Key design choices and trade‑offs

- **Canonical FHIR model vs. legacy diversity**  
  - Decision: define a canonical FHIR Rx Bundle as the internal transaction, even though many current systems speak HL7 v2 or SCRIPT.  
  - Trade‑off: up‑front mapping effort in Rhapsody in exchange for a cleaner, future‑proof model and simpler downstream logic.

- **Centralized validation vs. distributed checks**  
  - Decision: perform structural and core business/compliance validation centrally in Rhapsody before routing, while keeping advanced clinical decision support in prescribing systems and existing CDS services.  
  - Trade‑off: avoids duplicating validation across multiple middleware components, at the cost of initial consolidation work and tighter governance of the central rule set.

- **Enterprise‑centric analytics/audit vs. partner DWH coupling**  
  - Decision: have Rhapsody publish standardized Rx events into the pharma company’s own audit and analytics platforms only, while hospitals, pharmacies, and HUBs maintain their own local logging and warehouses.  
  - Trade‑off: simpler scope and clearer ownership of data, at the cost of partners needing to consume enterprise event feeds or reports rather than being directly updated.

- **Preserve current fulfillment behaviors vs. aggressive re‑platforming**  
  - Decision: keep current fulfillment platforms and their interfaces in place for MVP, integrating via Rhapsody instead of re‑platforming them immediately.  
  - Trade‑off: faster time‑to‑value and lower migration risk, with a clear roadmap for gradual adoption of FHIR APIs and modern data platforms.

## MVP scope

The MVP focuses on delivering a single, well‑governed end‑to‑end Rx flow across a limited but representative set of systems:

- Sources: 1–2 prescribing systems (e.g., a hospital EHR and one ambulatory/telehealth system).  
- Fulfillment: 1 internal hospital pharmacy platform and 1 external fulfillment/HUB platform.  
- Capabilities:  
  - FHIR-based Rx transaction API (Bundle) into Rhapsody.  
  - Canonical mapping from HL7 v2 / SCRIPT to the Rx Bundle for selected flows.  
  - Central structural and business validation plus routing before fulfillment.  
  - Enterprise‑centric analytics/audit export for each Rx event into central audit and data platforms.

This MVP preserves current business behavior for selected flows while de‑risking the new architecture and creating reusable patterns for later phases.

## Later phases (high level)

Subsequent phases build on the same foundation to:

- Onboard additional prescribing and fulfillment systems via the same FHIR/Rhapsody pattern.  
- Expand validation (terminology services, richer compliance rules) and operational monitoring.  
- Modernize downstream integrations toward native FHIR APIs where feasible.  
- Evolve the analytics and audit landscape toward a unified enterprise Rx event model supporting supply‑and‑demand planning and regulatory reporting.
