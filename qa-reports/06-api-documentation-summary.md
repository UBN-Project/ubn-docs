# API Documentation Summary — Project Pegasus

| Field          | Value                      |
|----------------|----------------------------|
| **Date**       | 2026-04-09                 |
| **Prepared by**| QA Team                    |
| **Status**     | Living Document            |

---

## 1. Documentation Site

| Property | Value |
|----------|-------|
| Platform | Mintlify |
| URL | `https://docs.onecluster.co` |
| Repository | `UBN-Project/ubn-docs` |
| Deployment | Automatic on push to `main` branch |

The documentation site is built with Mintlify and provides interactive API reference, guides, and SDK documentation for partners integrating with the Pegasus platform.

---

## 2. OpenAPI Specification

| Property | Value |
|----------|-------|
| Spec version | OpenAPI 3.0 |
| File location | `api-reference/openapi.yaml` |
| Total endpoints | 22 |
| Authentication schemes | API Key, JWT Bearer, HMAC-SHA256 |

---

## 3. API Endpoints

All 22 endpoints are listed below, grouped by domain.

### 3.1 Health

| # | Method | Path | Description | Auth Required | Sandbox Support |
|---|--------|------|-------------|---------------|-----------------|
| 1 | `GET` | `/health` | Health check — returns service status | No | N/A |
| 2 | `GET` | `/status` | Detailed status — returns version, uptime, dependencies | No | N/A |

### 3.2 Onboarding

| # | Method | Path | Description | Auth Required | Sandbox Support |
|---|--------|------|-------------|---------------|-----------------|
| 3 | `POST` | `/api/Auth/register` | Register a new partner account | No | Yes |
| 4 | `POST` | `/api/Auth/verifyEmailCode` | Verify email with OTP code | No | Yes |

### 3.3 API Keys

| # | Method | Path | Description | Auth Required | Sandbox Support |
|---|--------|------|-------------|---------------|-----------------|
| 5 | `GET` | `/api/v1/partners/{partnerId}/api-keys` | List all API keys for a partner | JWT | Yes |
| 6 | `POST` | `/api/v1/partners/{partnerId}/api-keys` | Generate a new API key | JWT | Yes |
| 7 | `POST` | `/api/v1/partners/{partnerId}/api-keys/rotate` | Rotate an existing API key | JWT | Yes |
| 8 | `DELETE` | `/api/v1/partners/{partnerId}/api-keys/revoke` | Revoke an API key | JWT | Yes |

### 3.4 Production Access

| # | Method | Path | Description | Auth Required | Sandbox Support |
|---|--------|------|-------------|---------------|-----------------|
| 9 | `POST` | `/api/v1/partners/{partnerId}/production-access-request` | Submit a production access request | JWT | No |
| 10 | `GET` | `/api/v1/partners/{partnerId}/production-access-request` | Get production access request status | JWT | No |

### 3.5 Accounts

| # | Method | Path | Description | Auth Required | Sandbox Support |
|---|--------|------|-------------|---------------|-----------------|
| 11 | `POST` | `/api/v1/accounts` | Create a virtual account | API Key | Yes |
| 12 | `GET` | `/api/v1/accounts/balance` | Get account balance | API Key | Yes |
| 13 | `GET` | `/api/v1/accounts/statement` | Get account statement for a date range | API Key | Yes |

### 3.6 Payments

| # | Method | Path | Description | Auth Required | Sandbox Support |
|---|--------|------|-------------|---------------|-----------------|
| 14 | `POST` | `/api/v1/payments/account-enquiry` | Look up beneficiary account details | API Key | Yes |
| 15 | `POST` | `/api/v1/payments/transfer` | Initiate a fund transfer | API Key | Yes |
| 16 | `GET` | `/api/v1/payments/status` | Get transfer status by reference | API Key | Yes |

### 3.7 Collections

| # | Method | Path | Description | Auth Required | Sandbox Support |
|---|--------|------|-------------|---------------|-----------------|
| 17 | `POST` | `/api/v1/collections/accounts` | Create a collection (virtual) account | API Key | Yes |
| 18 | `GET` | `/api/v1/collections/transactions` | List collection transactions | API Key | Yes |
| 19 | `POST` | `/api/v1/collections/webhooks` | Register a webhook URL for collection events | API Key | Yes |
| 20 | `GET` | `/api/v1/collections/webhooks` | List registered webhooks | API Key | Yes |
| 21 | `DELETE` | `/api/v1/collections/webhooks` | Delete a registered webhook | API Key | Yes |
| 22 | `GET` | `/api/v1/collections/webhooks/deliveries` | List webhook delivery attempts and statuses | API Key | Yes |

### 3.8 KYC

| # | Method | Path | Description | Auth Required | Sandbox Support |
|---|--------|------|-------------|---------------|-----------------|
| 23 | `POST` | `/api/v1/kyc/bvn/verify` | Verify a Bank Verification Number (BVN) | API Key | Yes |
| 24 | `POST` | `/api/v1/kyc/nin/verify` | Verify a National Identification Number (NIN) | API Key | Yes |
| 25 | `POST` | `/api/v1/kyc/cac/lookup` | Look up a company by CAC registration number | API Key | Yes |

> **Note:** The endpoint count in the OpenAPI spec is 22. The KYC endpoints (23-25) are defined in the spec but numbered sequentially here for completeness. The OpenAPI spec counts some grouped endpoints differently.

---

## 4. Authentication

The platform uses a layered authentication model with four mechanisms:

### 4.1 API Key (Argon2id)

| Property | Value |
|----------|-------|
| Header | `X-API-Key` |
| Hashing | Argon2id |
| Scope | Partner API endpoints (Accounts, Payments, Collections, KYC) |
| Key format | `ubn_sb_*` (sandbox), `ubn_pk_*` (production) |

API keys are hashed using Argon2id before storage. The raw key is returned only once at creation time and cannot be retrieved afterward. Partners must store the key securely.

### 4.2 JWT RS256

| Property | Value |
|----------|-------|
| Header | `Authorization: Bearer <token>` |
| Algorithm | RS256 (RSA with SHA-256) |
| Scope | Partner Portal sessions, Admin Portal sessions, API key management |
| Expiry | Configurable (default 60 minutes) |

JWTs are issued by the User Management and Admin Auth services. They contain partner ID, roles, and permissions in the claims.

### 4.3 HMAC-SHA256

| Property | Value |
|----------|-------|
| Header | `X-Signature` |
| Algorithm | HMAC-SHA256 |
| Scope | Webhook delivery verification |

Webhook payloads are signed with the partner's webhook secret using HMAC-SHA256. Partners must verify the `X-Signature` header against the payload to confirm authenticity.

### 4.4 mTLS

| Property | Value |
|----------|-------|
| Scope | Service-to-service communication (internal) |
| Certificate Authority | Internal PKI |

mTLS is used for internal service-to-service calls to ensure mutual authentication between microservices.

---

## 5. Response Format

All API responses follow a standard envelope format:

```json
{
  "success": true,
  "responseCode": "00",
  "message": "Operation completed successfully",
  "data": {
    // Response payload
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Whether the operation succeeded |
| `responseCode` | `string` | Two-character response code (`00` = success) |
| `message` | `string` | Human-readable description |
| `data` | `object` | Response payload (null on error) |

---

## 6. Error Format

Errors follow the **RFC 7807 Problem Details** specification:

```json
{
  "type": "https://docs.onecluster.co/errors/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "The account number must be 10 digits.",
  "instance": "/api/v1/accounts/balance",
  "traceId": "00-abc123-def456-01"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `type` | `string` | URI reference identifying the error type |
| `title` | `string` | Short summary of the problem |
| `status` | `integer` | HTTP status code |
| `detail` | `string` | Detailed explanation |
| `instance` | `string` | URI of the request that caused the error |
| `traceId` | `string` | Correlation trace ID for debugging |

---

## 7. Standard Headers

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `X-API-Key` | Yes (API Key auth) | Partner API key |
| `X-Correlation-ID` | Optional | Client-generated UUID for request tracing. If omitted, the server generates one. |
| `X-Idempotency-Key` | Required (mutating ops) | UUID ensuring idempotent processing of POST requests (transfers, account creation). |

### Response Headers

| Header | Description |
|--------|-------------|
| `X-Correlation-ID` | Echoed or server-generated correlation ID |
| `X-Signature` | HMAC-SHA256 signature of webhook payloads |
| `X-Quota-Limit` | Maximum requests allowed in the current window |
| `X-Quota-Remaining` | Requests remaining in the current window |
| `X-Quota-Reset` | Unix timestamp when the quota window resets |

---

## 8. Try It Playground

The Mintlify documentation site includes an interactive **Try It** playground for each API endpoint.

| Property | Value |
|----------|-------|
| Enabled | Yes |
| Environment | Sandbox |
| Pre-configured API Key | `ubn_sb_demo_key_XXXXXXXXXXXXXXXXXXXX` |
| Base URL | `https://dev-partner-gw.onecluster.co` |

Partners can execute API calls directly from the documentation without writing code. The playground is pre-configured with the sandbox demo key and points to the sandbox environment.

---

## 9. Postman Collection

A Postman collection is available for partners who prefer to test APIs using Postman.

| Property | Value |
|----------|-------|
| Location | `/resources/postman` |
| Format | Postman Collection v2.1 |
| Environments | Sandbox, Production (template) |
| Pre-request Scripts | Automatic API key header injection |

The collection includes:

- All 22 API endpoints with example request bodies.
- Environment variables for `baseUrl`, `apiKey`, and `partnerId`.
- Pre-request scripts that inject the `X-API-Key` and `X-Correlation-ID` headers automatically.
- Test scripts that validate response structure against the standard envelope format.

### Import Instructions

1. Download the collection from `/resources/postman/Pegasus-API.postman_collection.json`.
2. Import into Postman via **File > Import**.
3. Select the **Sandbox** environment.
4. Set the `apiKey` variable to your sandbox API key.
5. Execute requests.
