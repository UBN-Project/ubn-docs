# QA Report: Collections API
**Report ID:** QA-PEG-005 | **Date:** 2026-04-09 | **Sprint:** Sprint 3
**Service:** Ubn-Collections-API | **Environment:** Development (dev-api.onecluster.co)

## Executive Summary
The Collections API provides static virtual collection account creation, inbound transaction history retrieval, and webhook registration for real-time payment notifications. In sandbox mode, the service returns SB-prefixed account numbers and deterministic mock data. This report covers QA testing of all Collections API endpoints on the development environment.

## Scope
- POST /api/v1/collections/accounts — Create static virtual collection account
- GET /api/v1/collections/accounts/{accountNumber}/transactions — Inbound transaction history
- POST /api/v1/collections/webhooks — Register webhook endpoint (with ping validation)
- Sandbox behavior: SB-prefixed account numbers, mock transactions
- HMAC-SHA256 signature validation
- Webhook ping validation and retry logic
- Error handling and field naming consistency

## Changes This Sprint (Mar 31 — Apr 9)
- 17 commits to Ubn-Collections-API
- CI/CD pipeline configuration and automated deployment
- VPS deployment setup
- Webhook ping validation implemented (POST to partner URL with challenge)
- Sandbox mock response improvements
- Response envelope standardization
- Inbound transaction event logging
- Health check and readiness probe endpoints added

## Test Results

### Virtual Collection Account Creation
| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| QA-COL-001 | Create collection account with valid payload | 201 with accountNumber | 201 with virtualAccountNumber (SB-prefixed), accountName, bankName | PASS | Sandbox returns SB- prefix for easy identification |
| QA-COL-002 | Create collection account with missing fields | 422 Validation error | 422 with field-level errors (accountName, customerEmail required) | PASS | Proper RFC 7807 format |
| QA-COL-003 | Create collection account without HMAC | 401 HMAC_REQUIRED | 401 with error:"HMAC signature is required" | PASS | Security gate enforced |
| QA-COL-004 | Create collection account — field name check | accountNumber in response | virtualAccountNumber returned instead of accountNumber | FAIL | See ISSUE PEG-383 — field name mismatch with spec |
| QA-COL-005 | Create duplicate collection account (same reference) | 409 or idempotent 200 | 200 with existing account returned | PASS | Idempotency on reference key |
| QA-COL-006 | SB- prefix on sandbox accounts | Account number starts with SB- | "SB-1234567890" format confirmed | PASS | Clear sandbox identification |

### Inbound Transaction History
| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| QA-COL-007 | Get transactions for valid account | 200 with transactions array | 200 with transactions array, pagination metadata | PASS | Returns seeded sandbox transactions |
| QA-COL-008 | Get transactions for non-existent account | 404 ACCOUNT_NOT_FOUND | 404 with error:"Account not found" | PASS | Proper error handling |
| QA-COL-009 | Get transactions with date range | 200 with filtered results | 200 with transactions filtered by startDate/endDate | PASS | Date filtering works |
| QA-COL-010 | Get transactions with pagination | 200 with page metadata | 200 with page, pageSize, totalCount, totalPages | PASS | Pagination metadata present |
| QA-COL-011 | Transaction response — double nesting | data.transactions | data.data.transactions (double-nested) | FAIL | See ISSUE PEG-389 — response wrapped twice |
| QA-COL-012 | Transaction fields present | amount, reference, narration, timestamp | All fields present in each transaction object | PASS | Complete transaction data |

### Webhook Registration
| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| QA-COL-013 | Register webhook with valid URL | 200 with webhookId | 200 with webhookId, status:"ACTIVE" | PASS | Webhook registered and ping sent |
| QA-COL-014 | Register webhook — ping validation | Platform sends POST to URL with challenge | Platform sends {event:"ping", challenge:"<uuid>"}, expects challenge echoed back | PASS | Ensures partner URL is live and responsive |
| QA-COL-015 | Register webhook with unreachable URL | 422 WEBHOOK_VALIDATION_FAILED | 422 with error:"Webhook URL did not respond to ping" | PASS | Dead URLs rejected at registration |
| QA-COL-016 | Register webhook with non-HTTPS URL | 422 HTTPS_REQUIRED | 422 with error:"Webhook URL must use HTTPS" | PASS | Security requirement enforced |
| QA-COL-017 | Register webhook without auth | 401 Unauthorized | 401 with error:"API key is required" | PASS | Auth enforcement verified |
| QA-COL-018 | Duplicate webhook registration | 409 or update existing | 200 with existing webhook updated | PASS | Upsert behavior on same URL |

### Webhook Delivery (Code Review)
| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| QA-COL-019 | Webhook retry on failure | Exponential backoff, max 5 retries | Code review: retry policy with 30s, 60s, 120s, 300s, 600s intervals | CODE REVIEW PASS | Dead letter queue after 5 failures |
| QA-COL-020 | Webhook payload signature | HMAC-SHA256 in X-Webhook-Signature header | Code review: webhook payload signed with partner's secret key | CODE REVIEW PASS | Partners can verify authenticity |
| QA-COL-021 | Webhook payload contents | transactionRef, amount, accountNumber, timestamp | Code review: all fields included in webhook payload | CODE REVIEW PASS | |

## Issues Found
| ID | Severity | Issue | Status |
|---|---|---|---|
| PEG-383 | Medium | Response field name is `virtualAccountNumber` but OpenAPI spec defines `accountNumber` — inconsistency will cause partner integration failures if SDK/docs use spec field names | OPEN |
| PEG-389 | Medium | Inbound transaction history response has double nesting: `data.data.transactions` instead of `data.transactions` — response envelope is wrapped twice | OPEN |
| PEG-398 | Low | Webhook ping timeout is 5 seconds — may be too aggressive for partners with cold-start serverless functions | OPEN |

## Field Name Discrepancy Detail (PEG-383)

**OpenAPI Spec:**
```json
{
  "data": {
    "accountNumber": "SB-1234567890",
    "accountName": "Test Collection Account",
    "bankName": "Union Bank"
  }
}
```

**Actual Response:**
```json
{
  "data": {
    "virtualAccountNumber": "SB-1234567890",
    "accountName": "Test Collection Account",
    "bankName": "Union Bank"
  }
}
```

Partners building against the published spec will reference `data.accountNumber` and receive `undefined`. Either the spec or the API must be updated to match.

## Double Nesting Detail (PEG-389)

**Expected Response:**
```json
{
  "isSuccess": true,
  "data": {
    "transactions": [...],
    "page": 1,
    "pageSize": 20,
    "totalCount": 5
  }
}
```

**Actual Response:**
```json
{
  "isSuccess": true,
  "data": {
    "data": {
      "transactions": [...],
      "page": 1,
      "pageSize": 20,
      "totalCount": 5
    }
  }
}
```

The response envelope wrapper is applied twice — likely the controller returns a `ServiceResponse<PagedResult>` which is then wrapped again by a global response filter.

## Recommendations
1. **Fix PEG-383:** Align field name to either `virtualAccountNumber` or `accountNumber` consistently across spec and implementation before partner onboarding
2. **Fix PEG-389:** Remove double nesting in transaction history response — likely a middleware/filter issue wrapping the response envelope twice
3. Increase webhook ping timeout from 5s to 10s to accommodate serverless cold starts
4. Add webhook event type filtering so partners can subscribe to specific event types (e.g., only CREDIT events)
5. Document SB- prefix convention in partner integration guide
6. Consider adding a webhook test/replay endpoint for partners to re-trigger missed notifications during development
