# 01 – FHIR Integration Documentation Approach

The FHIR integration documentation follows a **layered, scalable model** designed to support both migration from legacy HL7 v2/SCRIPT systems and long-term FHIR adoption by new partners.

This structure minimizes cognitive load for different audiences (business stakeholders, developers, ops) while enabling governance and onboarding at scale.

## 1. Layered Documentation Model

### Business & Workflow Layer
*Primary audience: clinical leads, product managers, business stakeholders*
- Rx lifecycle diagrams and narratives
- System actors and responsibilities
- High-level sequence diagrams (happy path + error flows) — see [`rx-sequence-flow.puml`](../diagrams/rx-sequence-flow.puml) and [`rx-sequence-flow.png`](../diagrams/rx-sequence-flow.png)
- As-is and to-be architecture diagrams — see [`as-is-prescription-flow.puml`](../diagrams/as-is-prescription-flow.puml) and [`to-be-rx-flow-rhapsody-fhir.puml`](../diagrams/to-be-rx-flow-rhapsody-fhir.puml)

### Canonical Transaction Layer
*Primary audience: product managers, integration architects*
- Rx FHIR Bundle definition (type=transaction)
- Required resources, extensions, cardinalities
- Business identifier and atomicity guarantees

### FHIR Profile & Conformance Layer
*Primary audience: integration architects, FHIR specialists*
- StructureDefinition profiles for Bundle and child resources
- Extension definitions
- ValueSet/CodeSystem bindings and terminology catalog

### API Specification Layer
*Primary audience: integration developers, partner technical teams*
- REST endpoints (e.g., POST /rx-bundles, GET /rx-status/{id})
- Authentication (OAuth2/JWT or mTLS)
- Request/response schemas and examples
- Error model (OperationOutcome with structured codes)

### Mapping Specifications Layer
*Primary audience: integration developers, mapping engineers*
- HL7 v2 → FHIR Rx Bundle transformations
- NCPDP SCRIPT → FHIR mappings
- FHIR → downstream fulfillment formats (legacy + modern)

### Validation Rules Catalog
*Primary audience: clinical leads, compliance, integration architects*
- Structural validation (FHIR schema + Schematron)
- Business rules (e.g., consent required, valid pharmacy ID)
- Semantic rules (terminology validation)

### Examples & Test Cases
*Primary audience: integration developers, QA engineers, partner technical teams*
- Valid Rx Bundles (happy path)
- Invalid Bundles + expected OperationOutcome errors
- Test harness / Postman collections

### Operational Documentation
*Primary audience: SRE, ops teams, on-call engineers*
- Monitoring and alerting (SLA thresholds)
- Retry and idempotency policies
- Circuit breaker patterns
- Support and escalation procedures

### Versioning & Governance
*Primary audience: product managers, integration architects, partner leads*
- API versioning strategy (header-based)
- Backward compatibility rules
- Deprecation timeline and partner notification process
  - Breaking changes: formal notice ≥ 60 days before cutover, distributed via partner portal, including nature of change, affected endpoints, migration steps, and sandbox access
  - Non-breaking changes (additive fields, new optional elements): changelog entry only, no mandatory notice period
- Change approval process

### Onboarding Guide
*Primary audience: partner technical teams, integration developers*
- Step-by-step integration checklist
- Sandbox access and testing
- Certification criteria
- Partner portal with self-service docs
