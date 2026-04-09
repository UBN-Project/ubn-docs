# Build & Deployment Notes — Project Pegasus Sprint 3

| Field          | Value                          |
|----------------|--------------------------------|
| **Date**       | 2026-04-09                     |
| **Sprint**     | Sprint 3 (Mar 31 — Apr 9)     |
| **Prepared by**| QA Team                        |
| **Status**     | In Progress                    |

---

## 1. Docker Images

A total of **16 Docker images** were built and pushed to `docker.io/jnrjose/` during Sprint 3.

| # | Image Name | Registry |
|---|-----------|----------|
| 1 | `jnrjose/ubn-partner-gateway:latest` | docker.io |
| 2 | `jnrjose/ubn-admin-gateway:latest` | docker.io |
| 3 | `jnrjose/ubn-user-management:latest` | docker.io |
| 4 | `jnrjose/ubn-admin-auth:latest` | docker.io |
| 5 | `jnrjose/ubn-billing:latest` | docker.io |
| 6 | `jnrjose/ubn-wallet:latest` | docker.io |
| 7 | `jnrjose/ubn-customer:latest` | docker.io |
| 8 | `jnrjose/ubn-webhook:latest` | docker.io |
| 9 | `jnrjose/ubn-kyc:latest` | docker.io |
| 10 | `jnrjose/ubn-account:latest` | docker.io |
| 11 | `jnrjose/ubn-payment:latest` | docker.io |
| 12 | `jnrjose/ubn-collection:latest` | docker.io |
| 13 | `jnrjose/ubn-bank-adapter:latest` | docker.io |
| 14 | `jnrjose/ubn-notification:latest` | docker.io |
| 15 | `jnrjose/ubn-partner-portal:latest` | docker.io |
| 16 | `jnrjose/ubn-admin-portal:latest` | docker.io |

---

## 2. Deployment Targets

### Migration: Cloudflare Containers to VPS (EC2)

During Sprint 3 the deployment target was **migrated from Cloudflare Containers to a VPS hosted on AWS EC2**. The primary drivers for this migration were:

- Greater control over networking and port mapping.
- Simplified debugging via SSH access to the host.
- Reduced cold-start latency compared to container-based edge deployments.
- Ability to run long-lived background services (Kafka consumers, scheduled jobs).

The VPS runs Docker Compose for orchestration in the dev environment. Production will use a similar EC2-based setup with managed database backends.

---

## 3. CI/CD Pipeline

**Platform:** GitHub Actions

The pipeline follows a linear stage progression:

```
SAST  -->  Docker Build  -->  Push to Docker Hub  -->  Deploy to VPS  -->  Slack Notification
```

### Pipeline Stages

| Stage | Description |
|-------|-------------|
| **SAST** | Static Application Security Testing — scans source code for known vulnerability patterns and secrets before build. |
| **Docker Build** | Builds the service image using the Dockerfile in each service repository. Multi-stage builds are used to keep final images lean. |
| **Push to Docker Hub** | Tags the image as `latest` and pushes to `docker.io/jnrjose/`. |
| **Deploy to VPS** | SSH into the EC2 instance, pulls the latest image, and restarts the container via Docker Compose. |
| **Slack Notification** | Posts deployment status (success/failure) to the `#pegasus-deployments` Slack channel. |

---

## 4. Startup Sequence

Services must be started in a defined order to satisfy dependency requirements. The startup sequence is organized into six tiers:

| Tier | Category | Services |
|------|----------|----------|
| **Tier 0** | Infrastructure | Redis, SQL Server, Kafka |
| **Tier 1** | Leaf Services | Notification, Webhook |
| **Tier 2** | Core Services | Admin Auth, User Management |
| **Tier 3** | Domain APIs | Account, Payment, Collection, KYC, Billing, Wallet, Customer, Bank Adapter |
| **Tier 4** | Event Consumers | Payment consumer, Collection consumer, Webhook dispatcher |
| **Tier 5** | Frontends & Gateways | Partner Gateway, Admin Gateway, Partner Portal, Admin Portal |

Services at each tier depend on all tiers below them being healthy before they can start successfully.

---

## 5. Changes This Sprint

### Commit Activity

| Contributor | CI/CD Commits |
|-------------|---------------|
| Chijindu | 190 |
| Ezekiel-byte | 55 |
| **Total** | **245** |

### Notable Changes

- **Slack Notifications** — Added post-deployment Slack notifications to `#pegasus-deployments` for every service. Notifications include service name, image tag, environment, and deployment status.
- **CODEOWNERS Files** — Added `CODEOWNERS` files to all service repositories to enforce required code reviews. Each service is owned by its respective team lead.
- **Secrets Stripped from appsettings.json** — All connection strings, API keys, and credentials were removed from checked-in `appsettings.json` files and migrated to environment variables / GitHub Secrets. This addresses a security finding from Sprint 2.
- **Swagger Enabled in Staging** — Swagger UI is now enabled for all API services when running in the `Staging` environment. Previously it was only available in `Development`.

---

## 6. Dev Environment URLs

| # | Service | URL | Port |
|---|---------|-----|------|
| 1 | Partner Gateway | `https://dev-partner-gw.onecluster.co` | 5100 |
| 2 | Admin Gateway | `https://dev-admin-gw.onecluster.co` | 5200 |
| 3 | User Management | `https://dev-user-mgmt.onecluster.co` | 5010 |
| 4 | Admin Auth | `https://dev-admin-auth.onecluster.co` | 5020 |
| 5 | Billing | `https://dev-billing.onecluster.co` | 5030 |
| 6 | Wallet | `https://dev-wallet.onecluster.co` | 5040 |
| 7 | Customer | `https://dev-customer.onecluster.co` | 5050 |
| 8 | Webhook | `https://dev-webhook.onecluster.co` | 5060 |
| 9 | KYC | `https://dev-kyc.onecluster.co` | 5070 |
| 10 | Account | `https://dev-account.onecluster.co` | 5080 |
| 11 | Payment | `https://dev-payment.onecluster.co` | 5090 |
| 12 | Collection | `https://dev-collection.onecluster.co` | 5110 |
| 13 | Bank Adapter | `https://dev-bank-adapter.onecluster.co` | 5120 |
| 14 | Notification | `https://dev-notification.onecluster.co` | 5130 |
| 15 | Partner Portal | `https://dev-partner.onecluster.co` | 3000 |
| 16 | Admin Portal | `https://dev-admin.onecluster.co` | 3001 |

---

## 7. Known Deployment Issues

### 7.1 Partner Gateway — HTTP 502 Bad Gateway

- **Symptom:** Requests to the Partner Gateway return `502 Bad Gateway` intermittently.
- **Impact:** Partner API consumers cannot reach backend services through the gateway.
- **Root Cause (suspected):** The gateway is unable to route to downstream services that have not yet started or are unreachable on the internal Docker network.
- **Status:** Under investigation.

### 7.2 Nine Banking Services Unreachable

- **Symptom:** Nine of the backend services behind the Partner Gateway are not responding to health checks.
- **Services Affected:** Account, Payment, Collection, KYC, Billing, Wallet, Customer, Bank Adapter, Notification.
- **Impact:** All domain API functionality is unavailable in the dev environment.
- **Root Cause (suspected):** Services are either not starting due to missing environment variables or database connectivity issues on the VPS.
- **Status:** Under investigation. Requires SSH access to the VPS to inspect container logs.

---

## 8. Rollback Procedure

### Kubernetes (if applicable)

```bash
kubectl rollout undo deployment/<service-name> -n pegasus
```

### Docker Compose (current dev setup)

```bash
# Stop the current container
docker stop <container-name>

# Pull the previous image tag
docker pull jnrjose/<service-name>:<previous-tag>

# Restart with the previous tag
docker compose up -d <service-name>
```

Rollback should be performed within **15 minutes** of a failed deployment. If the previous tag is unknown, check the Docker Hub repository tags or the GitHub Actions workflow run history.

---

## 9. Database Migrations Applied

The following Entity Framework Core migrations were applied during Sprint 3:

| Migration Name | Service | Description |
|---------------|---------|-------------|
| `RenameAdminPermissionToOwner` | Admin Auth | Renames the `Admin` permission/role to `Owner` to better reflect the access level semantics across the platform. |
| `AddKycCompleted` | User Management / Customer | Adds a `KycCompleted` boolean column to the partner/customer record, used to gate access to production API keys. |

### Applying Migrations

Migrations are applied automatically on service startup via `Database.Migrate()` in the `Program.cs` startup pipeline. Manual application can be done with:

```bash
dotnet ef database update --project <ServiceProject> --startup-project <ServiceProject>
```

---

## 10. Summary

Sprint 3 focused heavily on CI/CD maturity and infrastructure migration. The pipeline is functional end-to-end (SAST through Slack notification), and all 16 Docker images build and push successfully. The primary blockers are the VPS networking issues causing 9 services to be unreachable and the Partner Gateway 502 errors. These must be resolved before Sprint 4 testing can proceed effectively.
