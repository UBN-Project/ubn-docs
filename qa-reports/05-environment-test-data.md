# Environment & Test Data Details — Project Pegasus

| Field          | Value                      |
|----------------|----------------------------|
| **Date**       | 2026-04-09                 |
| **Prepared by**| QA Team                    |
| **Status**     | Living Document            |

---

## 1. Dev Environment URLs

All dev services are deployed on a VPS (EC2) with Docker Compose orchestration.

| # | Service | URL | Status |
|---|---------|-----|--------|
| 1 | Partner Gateway | `https://dev-partner-gw.onecluster.co` | DOWN |
| 2 | Admin Gateway | `https://dev-admin-gw.onecluster.co` | UP |
| 3 | User Management | `https://dev-user-mgmt.onecluster.co` | UP |
| 4 | Admin Auth | `https://dev-admin-auth.onecluster.co` | UP |
| 5 | Billing | `https://dev-billing.onecluster.co` | DOWN |
| 6 | Wallet | `https://dev-wallet.onecluster.co` | DOWN |
| 7 | Customer | `https://dev-customer.onecluster.co` | DOWN |
| 8 | Webhook | `https://dev-webhook.onecluster.co` | DOWN |
| 9 | KYC | `https://dev-kyc.onecluster.co` | DOWN |
| 10 | Account | `https://dev-account.onecluster.co` | DOWN |
| 11 | Payment | `https://dev-payment.onecluster.co` | DOWN |
| 12 | Collection | `https://dev-collection.onecluster.co` | DOWN |
| 13 | Bank Adapter | `https://dev-bank-adapter.onecluster.co` | DOWN |
| 14 | Notification | `https://dev-notification.onecluster.co` | DOWN |
| 15 | Partner Portal | `https://dev-partner.onecluster.co` | UP |
| 16 | Admin Portal | `https://dev-admin.onecluster.co` | UP |

**Summary:** 7 services UP, 9 services DOWN (all banking domain services unreachable).

---

## 2. Production Environment URLs

| # | Service | URL |
|---|---------|-----|
| 1 | Partner Gateway | `https://api.onecluster.co` |
| 2 | Admin Gateway | `https://admin-api.onecluster.co` |
| 3 | Partner Portal | `https://partner.onecluster.co` |
| 4 | Admin Portal | `https://admin.onecluster.co` |
| 5 | Documentation | `https://docs.onecluster.co` |

> **Note:** Production is not yet live. These URLs are reserved and will be configured once staging validation is complete.

---

## 3. Infrastructure

### 3.1 Redis

| Property | Value |
|----------|-------|
| Version | Redis 7.2 |
| Port | 6379 |
| Usage | Session caching, rate limiting, idempotency key storage, distributed locking |
| Connection | `redis://localhost:6379` (dev), managed Redis in production |

### 3.2 SQL Server

| Property | Value |
|----------|-------|
| Version | SQL Server 2022 / Azure SQL Managed Instance |
| Host | `ubn-db-33351.database.windows.net` |
| Port | 1433 |
| Authentication | SQL Authentication (credentials in environment variables) |
| TLS | Required (`Encrypt=True;TrustServerCertificate=False`) |

### 3.3 Kafka

| Property | Value |
|----------|-------|
| Port | 9092 |
| Usage | Event-driven communication between services (payment events, webhook dispatch, collection notifications) |
| Topics | `payment-events`, `collection-events`, `webhook-dispatch`, `notification-events` |

---

## 4. Database Names Per Service

Each service owns its own database to enforce bounded context separation.

| # | Service | Database Name |
|---|---------|---------------|
| 1 | User Management | `PegasusUserMgmtDb` |
| 2 | Admin Auth | `PegasusAdminAuthDb` |
| 3 | Webhook | `PegasusWebhookDb` |
| 4 | Billing | `PegasusBillingDb` |
| 5 | Wallet | `PegasusWalletDb` |
| 6 | Customer | `PegasusCustomerDb` |

> **Note:** Services such as Account, Payment, Collection, KYC, Bank Adapter, and Notification may share databases or use the above databases depending on the domain model. Refer to each service's `appsettings.json` for the exact connection string.

---

## 5. Test Data

### 5.1 Sandbox Demo API Key

```
ubn_sb_demo_key_XXXXXXXXXXXXXXXXXXXX
```

This is the pre-provisioned sandbox API key used for testing. It is bound to the demo partner account and has access to all sandbox endpoints. The `XX` portion represents the actual key value, which is stored securely and provided to testers on request.

### 5.2 Sandbox Test Accounts

| Account Number | Type | Balance | Status |
|---------------|------|---------|--------|
| `0100000001` | Savings | ₦1,000,000.00 | Active |
| `0100000002` | Current | ₦500,000.00 | Active |
| `0199999999` | — | ₦0.00 | Closed |

These accounts are pre-seeded in the sandbox environment and reset on each deployment. They are designed to exercise various account states:

- **0100000001** — Standard active savings account with sufficient balance for transfer tests.
- **0100000002** — Standard active current account for receiving transfers and testing account enquiry.
- **0199999999** — Closed account used to test error handling (expected error: `ACCOUNT_CLOSED`).

### 5.3 Payment Test Bank Codes

| Bank Code | Bank Name | Type |
|-----------|-----------|------|
| `044` | Union Bank of Nigeria | Live (sandbox mock) |
| `058` | GTBank | Mock |

In sandbox mode, transfers to bank code `044` simulate Union Bank processing. Transfers to bank code `058` simulate GTBank and return mock success responses.

---

## 6. Sandbox Mock Responses

The sandbox environment returns deterministic mock responses for third-party integrations.

### 6.1 Account Balance

```json
{
  "success": true,
  "responseCode": "00",
  "message": "Balance retrieved successfully",
  "data": {
    "accountNumber": "0100000001",
    "availableBalance": 500000,
    "currency": "NGN",
    "ledgerBalance": 500000
  }
}
```

### 6.2 BVN Verification

```json
{
  "success": true,
  "responseCode": "00",
  "message": "BVN verification successful",
  "data": {
    "bvn": "22200000000",
    "matchStatus": "FULL_MATCH",
    "firstName": "DEMO",
    "lastName": "USER"
  }
}
```

### 6.3 CAC Lookup

```json
{
  "success": true,
  "responseCode": "00",
  "message": "CAC lookup successful",
  "data": {
    "rcNumber": "RC000001",
    "companyName": "SANDBOX TEST COMPANY LTD",
    "status": "ACTIVE",
    "registrationDate": "2020-01-15"
  }
}
```

---

## 7. Admin Test Accounts

### 7.1 DemoAdAuthClient

A pre-configured OAuth client for the Admin Auth service used in testing.

| Property | Value |
|----------|-------|
| Client ID | `DemoAdAuthClient` |
| Grant Type | `client_credentials` |
| Scope | `admin.read admin.write` |

### 7.2 Seed Data — ad-users.json

Admin user accounts are seeded from the `ad-users.json` file located in the Admin Auth service's data directory. This file contains pre-configured admin users with the following roles:

| Username | Role | Purpose |
|----------|------|---------|
| `admin@onecluster.co` | Owner | Full platform access |
| `reviewer@onecluster.co` | Reviewer | Production access request reviews |
| `support@onecluster.co` | Support | Read-only access for support operations |

> **Note:** Passwords for seed accounts are set via environment variables and are not stored in the JSON file.

---

## 8. Environment Variables Reference

The following table lists key environment variables required by each service category. Actual values are stored in GitHub Secrets and injected at deployment time.

### 8.1 Common (All Services)

| Variable | Description |
|----------|-------------|
| `ASPNETCORE_ENVIRONMENT` | Runtime environment (`Development`, `Staging`, `Production`) |
| `ConnectionStrings__DefaultConnection` | SQL Server connection string |
| `Redis__ConnectionString` | Redis connection string (`localhost:6379`) |
| `Kafka__BootstrapServers` | Kafka broker address (`localhost:9092`) |

### 8.2 Authentication Services (Admin Auth, User Management)

| Variable | Description |
|----------|-------------|
| `Jwt__Issuer` | JWT token issuer URI |
| `Jwt__Audience` | JWT token audience |
| `Jwt__SigningKey` | RSA private key for RS256 signing (base64-encoded) |
| `Jwt__ExpiryMinutes` | Token expiry duration |

### 8.3 Gateway Services (Partner Gateway, Admin Gateway)

| Variable | Description |
|----------|-------------|
| `ReverseProxy__Clusters__*` | YARP reverse proxy cluster destinations |
| `RateLimiting__PermitLimit` | Requests per window |
| `RateLimiting__WindowSeconds` | Rate limit window duration |

### 8.4 Domain Services (Account, Payment, Collection, KYC)

| Variable | Description |
|----------|-------------|
| `BankAdapter__BaseUrl` | Bank Adapter service URL |
| `Webhook__BaseUrl` | Webhook service URL |
| `Notification__BaseUrl` | Notification service URL |

### 8.5 Frontend Services (Partner Portal, Admin Portal)

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_API_URL` | Backend API base URL |
| `NEXT_PUBLIC_ENVIRONMENT` | Frontend environment label |
| `NEXTAUTH_SECRET` | NextAuth.js session secret |

---

## 9. Data Cleanup

**Sandbox data is ephemeral.** No manual cleanup is required.

- Test accounts (`0100000001`, `0100000002`, `0199999999`) are re-seeded on each deployment.
- Transaction history in sandbox mode is not persisted across container restarts.
- The sandbox demo API key remains stable across deployments (it is seeded idempotently).
- Admin seed users are upserted on startup — existing records are updated, not duplicated.

For manual data reset during testing, restart the relevant service container:

```bash
docker compose restart <service-name>
```

This triggers the database migration and seed data pipeline on startup.
