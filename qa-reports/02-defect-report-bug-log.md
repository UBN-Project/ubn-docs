# Defect Report / Bug Log — Project Pegasus

**Project:** Project Pegasus BaaS | **Sprint:** Sprint 3
**Date:** 2026-04-09 | **Prepared By:** QA Team
**Total Defects:** 15 | **Critical:** 2 | **High:** 2 | **Medium:** 7 | **Low:** 4

---

## 1. Defect Summary Dashboard

| Priority | Open | In Progress | Resolved | Closed | Total |
|---|---|---|---|---|---|
| Critical (P0) | 2 | 0 | 0 | 0 | 2 |
| High (P1) | 2 | 0 | 0 | 0 | 2 |
| Medium (P2) | 7 | 0 | 0 | 0 | 7 |
| Low (P3) | 4 | 0 | 0 | 0 | 4 |
| **Total** | **15** | **0** | **0** | **0** | **15** |

## 2. Defect Log

### CRITICAL (P0) — Blocks Testing

| Bug ID | Title | Service | Found Date | Status | Assigned To | Description | Steps to Reproduce | Expected | Actual |
|---|---|---|---|---|---|---|---|---|---|
| PEG-375 | Dev Partner Gateway returning 502 | Ubn-Partner-Gateway | 2026-04-06 | OPEN | DevOps | All partner API traffic blocked in dev environment. Cloudflare returns HTML 502 error page. | 1. curl https://dev-api-partner.onecluster.co/health | 200 "Healthy" | 502 Cloudflare HTML error |
| PEG-376 | All 13 banking microservices unreachable in dev | Infrastructure | 2026-04-06 | OPEN | DevOps | Banking API, Accounts, Payments, Collections, KYC, Wallet, Customer Service, Webhook Dispatcher, 3rd Party Integration all return HTTP 000. | 1. curl https://dev-banking.onecluster.co/health | 200 "Healthy" | HTTP 000 (connection refused) |

### HIGH (P1) — Security Issues

| Bug ID | Title | Service | Found Date | Status | Description | Impact | Recommendation |
|---|---|---|---|---|---|---|---|
| PEG-377 | Missing security headers on Admin Portal | Ubn-API-Portal-Admin | 2026-04-06 | OPEN | All 6 standard security headers missing (X-Frame-Options, X-Content-Type-Options, Strict-Transport-Security, CSP, X-XSS-Protection, Referrer-Policy) | Vulnerable to XSS, clickjacking, MIME sniffing attacks. Admin portal handles KYB approvals and user management. | Add headers in next.config.js or Cloudflare Page Rules |
| PEG-378 | API catalog publicly accessible without authentication | Ubn-Admin-Gateway | 2026-04-06 | OPEN | GET /api/catalog/products returns full product list (names, descriptions, subscriber counts, pricing models) without any auth token. | Business-sensitive data exposed. Competitive intelligence risk. | Add auth middleware to /api/catalog/* route in ocelot.json |

### MEDIUM (P2) — Functional Issues

| Bug ID | Title | Service | Found Date | Status | Description | Steps to Reproduce | Expected | Actual |
|---|---|---|---|---|---|---|---|---|
| PEG-379 | Login returns HTTP 200 for failed authentication | Ubn-User-Management | 2026-04-06 | OPEN | POST /api/Auth/login with invalid credentials returns 200 with isSuccess:false instead of 401 | POST dev-api.onecluster.co/api/Auth/login with wrong password | 401 Unauthorized | 200 {isSuccess:false, responseCode:"999"} |
| PEG-380 | mTLS enforced on payment routes in sandbox | Ubn-Partner-Gateway | 2026-04-06 | OPEN | All /api/v1/payments/* routes require mTLS client certificate even with sandbox key (ubn_sb_*). Prevents sandbox payment testing. | POST /api/v1/payments/transfer with demo key | 202 Accepted (sandbox mock) | 401 MTLS_CERTIFICATE_REQUIRED |
| PEG-381 | Sandbox mock responses take 27-30 seconds | Ubn-Partner-Gateway | 2026-04-06 | OPEN | Static sandbox mock responses should return in <100ms but take 27-30s. Suspected cause: rate limiting middleware hitting slow/unreachable Redis before sandbox intercept. | GET /api/v1/accounts/balance with demo key | Response in <1s | Response in 27-30s |
| PEG-382 | /api/Profile/GetAllRoles returns 404 | Ubn-User-Management | 2026-04-06 | OPEN | Endpoint returns 404 on both direct UMS access and via Admin Gateway. Admin Portal RBAC page depends on this endpoint. | GET dev-api.onecluster.co/api/Profile/GetAllRoles | 200 with roles list | 404 empty body |
| PEG-383 | Collections API field name mismatch | Ubn-Collections-API | 2026-04-06 | OPEN | POST /api/v1/collections/virtual-account returns `virtualAccountNumber` but spec says `accountNumber`. Prefix is `SB` not `SB-`. | POST /api/v1/collections/virtual-account | {data:{accountNumber:"SB-..."}} | {data:{virtualAccountNumber:"SB..."}} |
| PEG-387 | Inconsistent error formats between services | Multiple | 2026-04-06 | OPEN | Admin Auth uses simple {"message":"..."}, User Management uses RFC 7807 {type,title,status,detail,errors}. Platform standard is RFC 7807. | POST invalid request to both services | Consistent RFC 7807 format | Two different formats |
| PEG-384 | KYC BVN response field name mismatch | Ubn-KYC-API | 2026-04-06 | OPEN | POST /api/v1/kyc/verify/bvn returns `verificationStatus` but spec says `matchResult`. | POST /api/v1/kyc/verify/bvn | {data:{matchResult:"FULL_MATCH"}} | {data:{verificationStatus:"FULL_MATCH"}} |

### LOW (P3) — Cosmetic / Minor

| Bug ID | Title | Service | Found Date | Status | Description |
|---|---|---|---|---|---|
| PEG-385 | Admin Portal page title "Create Next App" | Ubn-API-Portal-Admin | 2026-04-06 | OPEN | Default Next.js title not customized. Should be "Pegasus Admin Portal". |
| PEG-386 | 404 sandbox endpoint returns empty body | Ubn-Partner-Gateway | 2026-04-06 | OPEN | Unknown sandbox endpoints return 404 with empty body instead of RFC 7807 JSON. |
| PEG-388 | sendOtp aggressively rate-limited | Ubn-User-Management | 2026-04-06 | OPEN | POST /api/Auth/sendOtp returns 429 even on first call. May be Cloudflare WAF or overly aggressive app-level rate limiting. |
| PEG-389 | Collections transactions double-nested | Ubn-Collections-API | 2026-04-06 | OPEN | Response nested at data.transactions instead of directly under data. Minor inconsistency with other APIs. |

## 3. Defect Trend

| Week | New | Resolved | Open |
|---|---|---|---|
| Apr 1–6 | 15 | 0 | 15 |
| Apr 7–9 | 0 | 0 | 15 |

## 4. Defect Distribution by Service

| Service | Critical | High | Medium | Low | Total |
|---|---|---|---|---|---|
| Infrastructure | 1 | 0 | 0 | 0 | 1 |
| Ubn-Partner-Gateway | 1 | 0 | 2 | 1 | 4 |
| Ubn-API-Portal-Admin | 0 | 1 | 0 | 1 | 2 |
| Ubn-Admin-Gateway | 0 | 1 | 0 | 0 | 1 |
| Ubn-User-Management | 0 | 0 | 2 | 1 | 3 |
| Ubn-Collections-API | 0 | 0 | 1 | 1 | 2 |
| Ubn-KYC-API | 0 | 0 | 1 | 0 | 1 |
| Multiple | 0 | 0 | 1 | 0 | 1 |
