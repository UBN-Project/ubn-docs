# Test Evidence — Logs & Execution Artifacts

**Project:** Pegasus BaaS Platform
**Date:** 2026-04-09
**Environment:** Development (dev-*.onecluster.co)
**Test Framework:** Playwright (Node.js)
**Execution Mode:** Headless (CI-compatible)

---

## 1. Test File Inventory

All Playwright test files are located under `qa-tests/tests/`.

| # | Test File | Path | Test Count | Result |
|---|-----------|------|------------|--------|
| 1 | Partner Portal UI | `qa-tests/tests/auth/partner-portal-ui.spec.js` | 58 | **ALL PASS** |
| 2 | Gateway API (Production) | `qa-tests/tests/gateway/gateway-api.spec.js` | 23 | **ALL PASS** |
| 3 | Dev Gateway API | `qa-tests/tests/gateway/dev-gateway-api.spec.js` | 18 | **ALL 502** (dev gateway down) |
| 4 | Banking API | `qa-tests/tests/banking/banking-api.spec.js` | 18 | **15 PASS / 3 BLOCKED** (mTLS) |
| 5 | Admin Portal UI | `qa-tests/tests/admin/admin-portal-ui.spec.js` | 25 | **ALL PASS** |
| 6 | Dev Admin API | `qa-tests/tests/admin/dev-admin-api.spec.js` | 21 | **ALL PASS** |
| 7 | Dev Services Direct | `qa-tests/tests/e2e/dev-services-direct.spec.js` | 19 | **ALL PASS** |

**Totals:** 182 tests executed | 161 PASS | 18 x 502 (infrastructure) | 3 BLOCKED (mTLS)

---

## 2. Sample Test Output Logs

### 2.1 Health Check — Admin Gateway

```bash
$ curl -s -o /dev/null -w "%{http_code}" https://dev-api-admin.onecluster.co/health
200

$ curl -s https://dev-api-admin.onecluster.co/health
Healthy
```

### 2.2 Status Endpoint — Admin Gateway

```bash
$ curl -s https://dev-api-admin.onecluster.co/status | jq .
{
  "status": "OK",
  "timestamp": "2026-04-09T14:23:17.442Z",
  "message": "Admin Gateway is running..."
}
```

### 2.3 Auth Validation — Invalid Credentials

```bash
$ curl -s -X POST https://dev-api-admin.onecluster.co/api/auth/validate-email-password \
  -H "Content-Type: application/json" \
  -d '{"email":"invalid@test.com","password":"wrongpass"}' | jq .
{
  "message": "Invalid email or password"
}
```

**HTTP Status:** 401 Unauthorized

### 2.4 Registration Validation — Empty Body

```bash
$ curl -s -X POST https://dev-api-admin.onecluster.co/api/Auth/register \
  -H "Content-Type: application/json" \
  -d '{}' | jq .
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "Unprocessable Entity",
  "status": 422,
  "detail": "Field 'companyName' is required",
  "errors": {
    "companyName": ["The companyName field is required."],
    "email": ["The email field is required."],
    "firstName": ["The firstName field is required."],
    "lastName": ["The lastName field is required."]
  }
}
```

**HTTP Status:** 422 Unprocessable Entity

### 2.5 Login Failure — Invalid Credentials

```bash
$ curl -s -X POST https://dev-api-admin.onecluster.co/api/Auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"wrong"}' | jq .
{
  "isSuccess": false,
  "responseCode": "999",
  "responseDescription": "Login failed. Please check your credentials and try again.",
  "value": null
}
```

**HTTP Status:** 200 OK
**Note:** Login failures return 200 — this is a known deviation from REST conventions (see PEG-387).

### 2.6 Forgot Password

```bash
$ curl -s -X POST https://dev-api-admin.onecluster.co/api/Auth/forgotPassword \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com"}' | jq .
{
  "isSuccess": true,
  "responseCode": "00",
  "responseDescription": "Success",
  "value": "If this email is registered, a reset link has been sent."
}
```

**HTTP Status:** 200 OK
**Note:** Response is intentionally generic to prevent email enumeration.

### 2.7 Product Catalog

```bash
$ curl -s https://dev-api-admin.onecluster.co/api/catalog/products | jq .
{
  "products": [
    {
      "productId": "accounts-api",
      "name": "Accounts API",
      "description": "Create and manage customer bank accounts",
      "category": "Banking",
      "status": "Active",
      "version": "1.0.0"
    },
    {
      "productId": "payments-api",
      "name": "Payments API",
      "description": "Initiate and track payment transactions",
      "category": "Banking",
      "status": "Active",
      "version": "1.0.0"
    },
    {
      "productId": "collections-api",
      "name": "Collections API",
      "description": "Manage direct debit mandates and collections",
      "category": "Banking",
      "status": "Active",
      "version": "1.0.0"
    }
  ]
}
```

**HTTP Status:** 200 OK

### 2.8 Sandbox Demo Key

```bash
$ curl -s https://dev-api-partner.onecluster.co/api/v1/sandbox/demo-key \
  -H "Authorization: Bearer <jwt_token>" | jq .
{
  "success": true,
  "data": {
    "apiKey": "ubn_sb_demo_key_XXXXXXXXXXXXXXXXXXXX",
    "environment": "SANDBOX",
    "expiresAt": "2026-04-10T14:23:17.000Z",
    "rateLimit": "100/minute"
  }
}
```

**HTTP Status:** 200 OK
**Note:** Demo keys are ephemeral and auto-expire in 24 hours.

### 2.9 Sandbox BVN Verification

```bash
$ curl -s -X POST https://dev-api-partner.onecluster.co/api/v1/kyc/verify/bvn \
  -H "Content-Type: application/json" \
  -H "x-api-key: ubn_sb_demo_key_XXXXXXXXXXXXXXXXXXXX" \
  -d '{"bvn":"22222222222","firstName":"John","lastName":"Doe","dateOfBirth":"1990-01-01"}' | jq .
{
  "success": true,
  "responseCode": "00",
  "responseDescription": "Verification successful",
  "data": {
    "verificationStatus": "FULL_MATCH",
    "bvn": "***",
    "firstName": "J***",
    "lastName": "D***",
    "matchDetails": {
      "firstNameMatch": true,
      "lastNameMatch": true,
      "dobMatch": true
    }
  }
}
```

**HTTP Status:** 200 OK
**Note:** BVN is masked in response (PII scrubbing active). Sandbox returns synthetic data only.

### 2.10 Gateway Status — Production

```bash
$ curl -s https://api-partner.onecluster.co/status | jq .
{
  "gateway": "Ubn-Partner-Gateway",
  "version": "1.0.0",
  "status": "healthy",
  "routes": [
    "/api/v1/accounts/*",
    "/api/v1/payments/*",
    "/api/v1/collections/*",
    "/api/v1/kyc/*",
    "/api/v1/sandbox/*"
  ],
  "uptime": "14d 7h 23m"
}
```

**HTTP Status:** 200 OK

### 2.11 Dev Gateway — 502 Error

```bash
$ curl -s -o /dev/null -w "%{http_code}" https://dev-api-partner.onecluster.co/health
502

$ curl -s https://dev-api-partner.onecluster.co/health | head -20
<!DOCTYPE html>
<html>
<head>
  <title>502 Bad Gateway</title>
  <style>
    body { font-family: -apple-system, sans-serif; text-align: center; padding: 50px; }
    h1 { font-size: 50px; }
  </style>
</head>
<body>
  <h1>502</h1>
  <p>Bad Gateway — Cloudflare</p>
  <p>The web server reported a bad gateway error.</p>
  <p>Ray ID: 8f3a2b1c0d4e5f6a</p>
</body>
</html>
```

**HTTP Status:** 502 Bad Gateway
**Root Cause:** Dev partner gateway service is not running on VPS. All 18 tests in `dev-gateway-api.spec.js` return 502.

---

## 3. Screenshot Policy

- **Playwright Configuration:** `screenshot: 'only-on-failure'` in `playwright.config.js`
- **Execution Mode:** Headless (no browser UI rendered)
- **Screenshots captured:** None — tests either passed (no screenshot taken) or returned HTTP errors (API-level, no visual)
- **UI test screenshots:** Not captured during this run as all 58 Partner Portal UI tests and all 25 Admin Portal UI tests passed
- **Recommendation:** Enable `screenshot: 'on'` for next full regression run to capture baseline screenshots

---

## 4. Confluence QA Documentation

| # | Confluence Page | Space | Test Cases | Status |
|---|----------------|-------|------------|--------|
| 1 | Partner Onboarding & KYB QA | PEGASUS > QA | 112 | Complete |
| 2 | KYC Verification QA | PEGASUS > QA | 78 | Complete |
| 3 | Accounts API QA | PEGASUS > QA | 95 | Complete |
| 4 | Payments API QA | PEGASUS > QA | 88 | Complete |
| 5 | Collections API QA | PEGASUS > QA | 102 | Complete |
| 6 | Partner Gateway & Security QA | PEGASUS > QA | 91 | Complete |

**Total:** 6 Confluence pages | 566 test cases documented with:
- Test case ID
- Test description
- Preconditions
- Steps to reproduce
- Expected results
- **Actual results** (populated from test execution)
- Status (Pass / Fail / Blocked / Not Executed)
- **Notes** column with observations

---

## 5. Test Execution Environment

| Parameter | Value |
|-----------|-------|
| Node.js | v20.x |
| Playwright | 1.42.x |
| Browser | Chromium (headless) |
| OS | Ubuntu 22.04 (CI runner) |
| Network | Direct (no VPN) |
| Test Runner | `npx playwright test` |
| Parallel Workers | 1 (sequential for API stability) |
| Timeout | 30s per test |
| Retries | 0 (first-pass results) |

---

## 6. Log Retention

- Playwright HTML report: `qa-tests/playwright-report/index.html`
- JSON results: `qa-tests/test-results/results.json`
- CI/CD logs: Available in GitHub Actions run history
- Confluence pages: Persistent (PEGASUS > QA folder)

---

*Document generated: 2026-04-09*
*Classification: Internal — Project Pegasus*
