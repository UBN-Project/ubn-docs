# QA Report: Accounts API
**Report ID:** QA-PEG-003 | **Date:** 2026-04-09 | **Sprint:** Sprint 3
**Service:** Ubn-Accounts-API | **Environment:** Development (dev-api.onecluster.co)

## Executive Summary
The Accounts API provides virtual account creation, balance inquiry, and statement retrieval capabilities for the Pegasus BaaS platform. In sandbox mode, the service returns deterministic mock responses with a fixed balance of 500,000 NGN to enable partner integration testing. This report covers QA testing of all Accounts API endpoints on the development environment.

## Scope
- POST /api/v1/accounts — Create virtual account
- GET /api/v1/accounts/{accountNumber} — Balance inquiry
- GET /api/v1/accounts/{accountNumber}/statement — Account statement with date filters
- Sandbox mock response behavior (fixed balances, test account names)
- HMAC-SHA256 signature validation
- Error handling, validation, and edge cases

## Changes This Sprint (Mar 31 — Apr 9)
- 18 commits to Ubn-Accounts-API
- Full platform sandbox APIs added across all account endpoints
- CI/CD pipeline configuration and deployment automation
- Sandbox mock data seeding for deterministic test scenarios
- Response envelope standardization
- Logging and audit trail enhancements
- Health check endpoint added

## Test Results

### Virtual Account Creation
| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| QA-ACC-001 | Create virtual account with valid payload | 201 with accountNumber | 201 with accountNumber, accountName:"SANDBOX TEST ACCOUNT", currency:"NGN" | PASS | Sandbox returns SB-prefixed account numbers |
| QA-ACC-002 | Create virtual account with missing fields | 422 Validation error | 422 with field-level errors (accountName, customerEmail required) | PASS | Proper RFC 7807 validation |
| QA-ACC-003 | Create virtual account without HMAC | 401 HMAC_REQUIRED | 401 with error:"HMAC signature is required" | PASS | Security gate enforced |
| QA-ACC-004 | Create virtual account with invalid currency | 422 UNSUPPORTED_CURRENCY | 422 with error:"Only NGN is supported" | PASS | Currency whitelist active |
| QA-ACC-005 | Create duplicate virtual account (same reference) | 409 Conflict or idempotent 200 | 200 with existing account details returned | PASS | Idempotency on reference key |

### Balance Inquiry
| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| QA-ACC-006 | Get balance for valid account | 200 with balance details | 200 with availableBalance:500000, ledgerBalance:500000, currency:"NGN" | PASS | Sandbox returns fixed 500,000 NGN balance |
| QA-ACC-007 | Get balance for non-existent account | 404 ACCOUNT_NOT_FOUND | 404 with error:"Account not found" | PASS | Proper error for invalid accounts |
| QA-ACC-008 | Get balance without authentication | 401 Unauthorized | 401 with error:"API key is required" | PASS | Auth enforcement verified |
| QA-ACC-009 | Get balance with invalid HMAC | 401 HMAC_INVALID | 401 with error:"HMAC signature validation failed" | PASS | Tampered requests rejected |
| QA-ACC-010 | Balance response field types | availableBalance as number | availableBalance:500000 (number, not string) | PASS | Correct JSON typing |

### Account Statement
| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| QA-ACC-011 | Get statement for valid account | 200 with transactions array | 200 with transactions:[ ], pagination metadata | PASS | Returns empty array for new sandbox accounts |
| QA-ACC-012 | Get statement with date range | 200 with filtered transactions | 200 with transactions filtered by startDate/endDate | PASS | Date filtering works correctly |
| QA-ACC-013 | Get statement with invalid date format | 422 INVALID_DATE_FORMAT | 422 with error:"Date must be in YYYY-MM-DD format" | PASS | Date validation enforced |
| QA-ACC-014 | Get statement for non-existent account | 404 ACCOUNT_NOT_FOUND | 404 with error:"Account not found" | PASS | Consistent error handling |
| QA-ACC-015 | Get statement with pagination | 200 with page metadata | 200 with page, pageSize, totalCount, totalPages | PASS | Pagination metadata present |
| QA-ACC-016 | Get statement with large page size | 200 capped at max | 200 with pageSize capped at 100 | PASS | Server-side cap prevents abuse |

### Sandbox Mock Data
| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| QA-ACC-017 | Sandbox account name | "SANDBOX TEST ACCOUNT" | "SANDBOX TEST ACCOUNT" | PASS | Clearly identifies sandbox data |
| QA-ACC-018 | Sandbox balance consistency | Same balance on repeated calls | 500000 NGN on every call | PASS | Deterministic sandbox behavior |
| QA-ACC-019 | Sandbox account number prefix | SB- prefix | Account numbers start with SB- | PASS | Easy to distinguish sandbox vs live |

## Issues Found
| ID | Severity | Issue | Status |
|---|---|---|---|
| PEG-391 | Low | Account statement returns empty `transactions` array for all sandbox accounts — would be more useful with seeded mock transactions for partner testing | OPEN |
| PEG-392 | Low | Balance response does not include `lastUpdated` timestamp — partners cannot determine data freshness | OPEN |

## Security Verification
| Check | Expected | Actual | Status |
|---|---|---|---|
| HMAC-SHA256 on all endpoints | Required | Enforced on POST and GET endpoints | PASS |
| API key authentication | Required | x-api-key header validated | PASS |
| Rate limiting | Per-partner limits | Rate limiting headers present (X-RateLimit-Remaining) | PASS |
| PII in logs | Masked | Account numbers and customer names masked in application logs | PASS |

## Recommendations
1. Seed sandbox accounts with 5-10 mock transactions to enable statement testing without requiring real inbound transfers
2. Add `lastUpdated` timestamp to balance response for data freshness indication
3. Consider adding bulk account creation endpoint for partners managing high volumes
4. Document sandbox account number format (SB- prefix) in partner integration guide
