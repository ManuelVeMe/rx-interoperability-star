# 05 – API Spec and Error Handling

## 1. Goals and scope

This API specification describes how prescribing systems interact with the Rhapsody-based integration platform for Rx transactions, and how errors and negative flows are handled.

- Scope: submission of Rx transactions, retrieval of status, and basic error reporting.
- Style: pragmatic RESTful APIs using FHIR Bundles as the canonical transaction payload.
- Non-goals (for this case): full FHIR server behavior, advanced search, and full national eRx IG coverage.

---

## 2. Core REST endpoints

### 2.1 Submit prescription transaction

**Endpoint**

- POST /rx-bundles

**Description**

Submit a prescription as a FHIR Bundle of type transaction that conforms to the agreed Rx Bundle profile.

**Request**

- Headers:
  - Content-Type: application/fhir+json
  - Authorization: Bearer <token> (or equivalent)
- Body:
  - FHIR Bundle (Rx transaction Bundle)

**Successful response**

- Status: 201 Created
- Headers:
  - Location: /rx-status/{transactionId}
- Body (JSON example):

```json
{
  "transactionId": "123e4567-e89b-12d3-a456-426614174000",
  "status": "accepted",
  "createdAt": "2026-03-20T10:15:00Z"
}
```

Notes:

- transactionId is the canonical ID used for tracking and status retrieval.
- Structural and core business validation are executed before returning 201.

**Validation error response**

- Status: 400 Bad Request
- Body (simplified, OperationOutcome-like):

```json
{
  "transactionId": "123e4567-e89b-12d3-a456-426614174000",
  "status": "rejected",
  "errorType": "validation",
  "errors": [
    {
      "code": "REQUIRED_FIELD_MISSING",
      "field": "MedicationRequest.medicationCodeableConcept",
      "message": "Medication code is required."
    }
  ]
}
```

Other possible codes:

- 401 Unauthorized – authentication or authorization failure.
- 429 Too Many Requests – throttling limit exceeded.
- 500 Internal Server Error – unexpected platform error.

---

### 2.2 Retrieve prescription status

**Endpoint**

- GET /rx-status/{transactionId}

**Description**

Retrieve the current processing status and key timestamps for a previously submitted transaction.

**Response**

- Status: 200 OK
- Body example:

```json
{
  "transactionId": "123e4567-e89b-12d3-a456-426614174000",
  "overallStatus": "in-progress",
  "validationStatus": "passed",
  "routingStatus": "delivered",
  "fulfillmentStatuses": [
    {
      "targetSystemId": "HOSPITAL_PHARMACY_01",
      "status": "dispensed",
      "lastUpdatedAt": "2026-03-20T10:18:45Z"
    },
    {
      "targetSystemId": "ANALYTICS",
      "status": "delivered",
      "lastUpdatedAt": "2026-03-20T10:15:30Z"
    }
  ],
  "createdAt": "2026-03-20T10:15:00Z",
  "lastUpdatedAt": "2026-03-20T10:18:45Z",
  "lastError": null
}
```

Error codes:

- 404 Not Found – unknown transactionId or outside retention window.
- 401 Unauthorized – invalid credentials.

---

### 2.3 Optional: status callback (webhook)

For partners that want asynchronous notifications:

- Endpoint (on client side): POST {clientCallbackUrl}

Rhapsody sends:

```json
{
  "transactionId": "123e4567-e89b-12d3-a456-426614174000",
  "eventType": "fulfillment-status-changed",
  "overallStatus": "completed",
  "fulfillmentStatus": {
    "targetSystemId": "HOSPITAL_PHARMACY_01",
    "status": "dispensed",
    "lastUpdatedAt": "2026-03-20T10:18:45Z"
  }
}
```

Configuration of clientCallbackUrl is out of scope for the test but would normally be managed per client/tenant.

---

## 3. Validation behavior

### 3.1 Validation layers

Validation runs before routing and is split into layers:

1. **Structural validation**
   - FHIR Bundle is syntactically valid JSON.
   - Bundle.type = "transaction".
   - Required resources (Patient, Practitioner, MedicationRequest, Pharmacy Organization) are present and correctly referenced.

2. **Business validation**
   - Required elements per the Bundle profile are present (e.g., medication code, patient ID, prescriber ID, prescription number, status).
   - Logical consistency (e.g., dates, refills, statuses).

3. **Semantic/value-set validation (core subset)**
   - Key codes (e.g., medication, diagnosis, pharmacy ID) belong to agreed value sets or formats where enforced.
   - Policies decide whether unknown codes are blocking or warning-only.

### 3.2 Validation outcomes

- **Passed**:
  - POST /rx-bundles returns 201 Created and routing proceeds.
- **Failed**:
  - POST /rx-bundles returns 400 Bad Request with details.
  - No downstream routing is performed.
  - Failure is logged in the audit store with reason, fields, and correlation ID.

---

## 4. Routing and downstream interactions

Routing is internal to Rhapsody but influences how errors are surfaced.

### 4.1 Routing decisions

Routing rules consider:

- Target pharmacy ID.
- Drug type (e.g., specialty vs retail).
- Payer or region (for HUB or specialty routes).
- Source system or tenant.

Multiple targets are possible (e.g., primary fulfillment plus analytics).

### 4.2 Downstream protocol handling

Rhapsody may:

- Call downstream REST/FHIR APIs.
- Send HL7 v2 messages.
- Send SCRIPT messages or other formats where required.

From the client’s perspective, this complexity is hidden behind the /rx-bundles interface and status APIs.

---

## 5. Error and negative flows

### 5.1 Validation errors (pre-routing)

Handled synchronously at submission:

- Triggered when Bundle fails structural, business, or semantic checks.
- Response is 400 Bad Request with a structured error body listing:
  - Error codes (e.g., REQUIRED_FIELD_MISSING, INVALID_CODE).
  - Affected fields.
  - Human-readable messages.

Example:

```json
{
  "transactionId": "123e4567-e89b-12d3-a456-426614174000",
  "status": "rejected",
  "errorType": "validation",
  "errors": [
    {
      "code": "INVALID_CODE",
      "field": "MedicationRequest.reasonCode",
      "message": "Diagnosis code is not in the allowed value set."
    }
  ]
}
```

### 5.2 Routing/delivery errors (post-validation)

If routing or delivery to downstream platforms fails:

- Rhapsody retries delivery as per configured policies.
- If retries are exhausted:
  - The message is placed in a dead‑letter queue.
  - A routing/delivery error event is logged to audit/analytics.
  - GET /rx-status/{transactionId} reflects a failure in routingStatus or specific fulfillmentStatuses.
- Optionally, a notification can be sent via webhook to the source system.

Example status when downstream delivery failed:

```json
{
  "transactionId": "123e4567-e89b-12d3-a456-426614174000",
  "overallStatus": "failed",
  "validationStatus": "passed",
  "routingStatus": "failed",
  "fulfillmentStatuses": [
    {
      "targetSystemId": "HOSPITAL_PHARMACY_01",
      "status": "delivery-failed",
      "lastUpdatedAt": "2026-03-20T10:17:10Z"
    }
  ],
  "lastError": {
    "code": "DOWNSTREAM_UNAVAILABLE",
    "message": "Target system did not respond after 3 attempts."
  }
}
```

### 5.3 Partial failures in multi-target routes

In multi-target scenarios (e.g., fulfillment plus analytics):

- Each target has its own status entry.
- Overall status reflects the most critical failure:
  - If fulfillment fails but analytics succeed, overall status is still "failed".
  - If fulfillment succeeds and analytics fail, it may be reported as "completed-with-warnings" depending on policy.

---

## 6. Security and access (high-level)

Security considerations for the test (high-level only):

- **Authentication/Authorization**:
  - OAuth2/OpenID Connect or equivalent per client/partner.
  - Each client is only allowed to submit and query its own Rx transactions.

- **Data protection**:
  - TLS for all external API calls.
  - Logging and analytics limited to necessary fields; sensitive data protected according to policy.

Detailed security design is out of scope but would be specified in a real implementation.

---

## 7. Relation to other documents

- The structure and required elements of the FHIR Rx Bundle are defined in 03-fhir-rx-bundle-profile.md.
- The overall architecture and the positioning of these APIs within the Rhapsody platform are described in 02-target-architecture-and-flow.md.

## 8. Out-of-scope (future) API actions

The following actions are **intentionally out of scope for the MVP** but are expected in a full Rx API surface:

- Change / modify prescription  
  - E.g., dose change, pharmacy change, or updated instructions via a follow-up transaction or PATCH-like operation.
- Cancel prescription  
  - Explicit cancellation messages to downstream fulfillment platforms and corresponding status updates.
- Search / query across prescriptions  
  - Querying by patient, prescriber, time window, or status for operational use cases and reporting.
- Access to original / normalized Bundles  
  - Retrieval of the original or canonical Rx Bundle for troubleshooting and audit.

These capabilities would be added in later phases, building on the same FHIR-based Rx transaction model and Rhapsody integration patterns.
