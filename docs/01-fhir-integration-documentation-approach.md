# 01 – FHIR Integration Documentation Approach

The FHIR integration documentation follows a **layered, scalable model** designed to support both migration from legacy HL7 v2/SCRIPT systems and long-term FHIR adoption by new partners.

This structure minimizes cognitive load for different audiences (business stakeholders, developers, ops) while enabling governance and onboarding at scale.

## 1. Layered Documentation Model

### Business & Workflow Layer
- Rx lifecycle diagrams and narratives
- System actors and responsibilities
- High-level sequence diagrams (happy path + error flows) — see [`rx-sequence-flow.puml`](../diagrams/rx-sequence-flow.puml) and [`rx-sequence-flow.png`](../diagrams/rx-sequence-flow.png)
- As-is and to-be architecture diagrams — see [`as-is-prescription-flow.puml`](../diagrams/as-is-prescription-flow.puml) and [`to-be-rx-flow-rhapsody-fhir.puml`](../diagrams/to-be-rx-flow-rhapsody-fhir.puml)


### Canonical Transaction Layer
- Rx FHIR Bundle definition (type=transaction)
- Required resources, extensions, cardinalities
- Business identifier and atomicity guarantees

### FHIR Profile & Conformance Layer
- StructureDefinition profiles for Bundle and child resources
- Extension definitions
- ValueSet/CodeSystem bindings and terminology catalog

### API Specification Layer
- REST endpoints (e.g., POST /rx-bundles, GET /rx-status/{id})
- Authentication (OAuth2/JWT or mTLS)
- Request/response schemas and examples
- Error model (OperationOutcome with structured codes)

### Mapping Specifications Layer
- HL7 v2 → FHIR Rx Bundle transformations
- NCPDP SCRIPT → FHIR mappings
- FHIR → downstream fulfillment formats (legacy + modern)

### Validation Rules Catalog
- Structural validation (FHIR schema + Schematron)
- Business rules (e.g., consent required, valid pharmacy ID)
- Semantic rules (terminology validation)

### Examples & Test Cases
- Valid Rx Bundles (happy path)
- Invalid Bundles + expected OperationOutcome errors
- Test harness / Postman collections

### Operational Documentation
- Monitoring and alerting (SLA thresholds)
- Retry and idempotency policies
- Circuit breaker patterns
- Support and escalation procedures

### Versioning & Governance
- API versioning strategy (header-based)
- Backward compatibility rules
- Deprecation timeline
- Change approval process

### Onboarding Guide
- Step-by-step integration checklist
- Sandbox access and testing
- Certification criteria
- Partner portal with self-service docs
