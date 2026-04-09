# Performance Test Summary — Project Pegasus Sprint 3

| Field          | Value                          |
|----------------|--------------------------------|
| **Date**       | 2026-04-09                     |
| **Sprint**     | Sprint 3 (Mar 31 — Apr 9)     |
| **Prepared by**| QA Team                        |
| **Status**     | Partial — k6 tests not yet executed |

---

## 1. Test Tooling

| Tool | Purpose | Status |
|------|---------|--------|
| **Playwright** | API timing — measures response latency during end-to-end test execution | Executed |
| **curl** | Endpoint latency — manual spot checks on individual endpoints | Executed |
| **k6** | Load testing suite — synthetic load generation with configurable virtual users and scenarios | Written, NOT executed this sprint |

---

## 2. k6 Test Suite

### Location

```
UBN/load-tests/k6/
```

### Configured Scenarios

| Scenario | Target | Virtual Users | Duration |
|----------|--------|---------------|----------|
| Aggregate throughput | 500 TPS across all endpoints | 100 | 5 minutes |
| P99 latency | < 200ms per request | 50 | 3 minutes |
| Sustained load | Steady-state 200 TPS | 50 | 15 minutes |
| Spike test | 0 to 1000 TPS ramp | 200 | 2 minutes ramp, 3 minutes hold |

### Why Not Executed

The k6 test suite was written during Sprint 3 but could not be executed because 9 of the 16 services in the dev environment are currently unreachable. Load testing against an incomplete environment would produce invalid results. Execution is deferred until the dev environment is fully operational.

---

## 3. Measured Latencies (Playwright)

The following latencies were captured from Playwright test execution against the dev environment. Each measurement represents the wall-clock time from request initiation to response receipt.

### 3.1 Health Endpoints

| Service | Endpoint | Latency | Verdict |
|---------|----------|---------|---------|
| Admin Gateway | `GET /health` | ~200ms | PASS |
| Admin Auth | `GET /health` | ~200ms | PASS |
| User Management | `GET /health` | ~200ms | PASS |

Health endpoints are lightweight and respond within acceptable limits. The ~200ms latency includes network round-trip time to the VPS.

### 3.2 Sandbox Mock Endpoints

| Service | Endpoint | Latency | Verdict |
|---------|----------|---------|---------|
| Partner Gateway | `POST /api/v1/payments/account-enquiry` (sandbox) | 27-30 seconds | **FAIL** |
| Partner Gateway | `POST /api/v1/kyc/bvn/verify` (sandbox) | 27-30 seconds | **FAIL** |
| Partner Gateway | `POST /api/v1/accounts/balance` (sandbox) | 27-30 seconds | **FAIL** |

**ISSUE: PEG-381 — Severe Performance Degradation on Sandbox Mock Endpoints**

All sandbox mock endpoints are exhibiting response times of 27 to 30 seconds, far exceeding the 200ms target. This is a critical performance issue that renders the sandbox environment unusable for partner integration testing.

**Suspected root causes:**
- Database connection pool exhaustion on the Bank Adapter service.
- Synchronous blocking on the mock HTTP client used for sandbox responses.
- DNS resolution delays on the VPS for outbound mock service calls.

### 3.3 Partner Portal

| Page | Metric | Value | Threshold | Verdict |
|------|--------|-------|-----------|---------|
| Login page | Full page load | 2.2 seconds | < 5 seconds | PASS |
| Dashboard | Full page load | 2.6 seconds | < 5 seconds | PASS |
| API Keys page | Full page load | 2.4 seconds | < 5 seconds | PASS |

The Partner Portal is a Next.js application served behind Cloudflare CDN. Page load times are within the 5-second acceptance threshold.

### 3.4 Admin Portal

| Page | Metric | Value | Threshold | Verdict |
|------|--------|-------|-----------|---------|
| Login page | DOM interactive | ~966ms | < 3 seconds | PASS |
| Login page | Server response (TTFB) | 375ms | < 1 second | PASS |
| Dashboard | Full page load | ~1.2 seconds | < 5 seconds | PASS |

The Admin Portal demonstrates good performance with sub-second server response times.

### 3.5 Admin API Endpoints

| Endpoint | Method | Latency | Threshold | Verdict |
|----------|--------|---------|-----------|---------|
| `/api/admin/partners` | GET | ~800ms | < 2 seconds | PASS |
| `/api/admin/partners/{id}` | GET | ~450ms | < 2 seconds | PASS |
| `/api/admin/production-access-requests` | GET | ~1.1 seconds | < 2 seconds | PASS |
| `/api/admin/production-access-requests/{id}/approve` | POST | ~1.5 seconds | < 2 seconds | PASS |

All Admin API endpoints respond within the 2-second threshold.

### 3.6 User Management Validation

| Endpoint | Method | Latency | Threshold | Verdict |
|----------|--------|---------|-----------|---------|
| `/api/Auth/register` (validation error) | POST | ~240ms | < 1 second | PASS |
| `/api/Auth/verifyEmailCode` (invalid code) | POST | ~220ms | < 1 second | PASS |

Validation responses are fast, indicating the request pipeline short-circuits correctly on invalid input without hitting the database.

---

## 4. Performance Issues Found

### PEG-381: Sandbox Mock Endpoints — 27-30 Second Latency

| Property | Value |
|----------|-------|
| Jira ID | PEG-381 |
| Severity | Critical |
| Affected endpoints | All sandbox mock endpoints via Partner Gateway |
| Expected latency | < 200ms |
| Actual latency | 27,000 — 30,000ms |
| Degradation factor | ~135x slower than target |
| Status | Open |

**Impact:** Partners cannot test integrations against the sandbox environment. The 30-second response time will cause client-side timeouts in most HTTP clients (default timeout is typically 30 seconds).

**Recommended investigation steps:**
1. Check Bank Adapter service container logs for timeout or connection errors.
2. Inspect database connection pool metrics on the Bank Adapter.
3. Verify DNS resolution time from the VPS to external mock endpoints.
4. Check for synchronous blocking calls in the sandbox mock handler.

---

## 5. Non-Functional Requirement (NFR) Targets

| NFR | Target | Status |
|-----|--------|--------|
| Throughput | 500 TPS aggregate | NOT YET VALIDATED |
| Latency (P99) | < 200ms | NOT YET VALIDATED |
| Uptime | 99.9% | NOT YET VALIDATED |
| Page load (Partner Portal) | < 5 seconds | PASS (measured 2.2-2.6s) |
| Page load (Admin Portal) | < 5 seconds | PASS (measured ~1.2s) |
| Health endpoint response | < 1 second | PASS (measured ~200ms) |

The primary NFR targets (500 TPS, P99 < 200ms, 99.9% uptime) have **not yet been validated** because the k6 load tests were not executed during this sprint. These require a fully operational dev environment with all 16 services running.

---

## 6. Recommendation

**Execute k6 load tests once the dev environment is fully operational.**

Specific prerequisites:
1. All 9 unreachable banking services must be brought online.
2. The Partner Gateway 502 issue must be resolved.
3. PEG-381 (sandbox mock latency) must be investigated and fixed.
4. Database connection pools should be configured with appropriate limits before load testing.

Once these prerequisites are met, the k6 test suite in `UBN/load-tests/k6/` can be executed against the dev environment to validate the 500 TPS, P99 < 200ms, and 99.9% uptime targets.

---

## 7. Cloudflare CDN Caching Observations

During testing, the following CDN caching behavior was observed on frontend services:

| Header | Value | Meaning |
|--------|-------|---------|
| `cf-cache-status` | `DYNAMIC` | Cloudflare is not caching the response (dynamic content) |
| `x-nextjs-cache` | `HIT` | Next.js internal cache served the page from its cache layer |

**Interpretation:** Cloudflare is treating all responses as dynamic (no edge caching). However, Next.js is caching pages at the application layer, which explains the good page load times despite the lack of CDN caching. For production, enabling Cloudflare edge caching for static assets and ISR pages would further improve performance.

---

## 8. Summary

| Area | Result |
|------|--------|
| Health endpoints | PASS (~200ms) |
| Sandbox mock endpoints | FAIL (27-30s — PEG-381) |
| Partner Portal pages | PASS (2.2-2.6s) |
| Admin Portal pages | PASS (~966ms DOM, 375ms TTFB) |
| Admin API endpoints | PASS (all < 2s) |
| User Management validation | PASS (~240ms) |
| k6 load tests (500 TPS, P99) | NOT EXECUTED |
| NFR validation | INCOMPLETE |

Sprint 3 performance testing is partial. The endpoints that are reachable perform well (with the critical exception of sandbox mocks). Full NFR validation requires the k6 suite to be executed in Sprint 4 once the environment is stable.
