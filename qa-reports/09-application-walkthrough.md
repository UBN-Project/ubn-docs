# Application Walkthrough — Project Pegasus BaaS Platform

**Project:** Pegasus BaaS Platform
**Date:** 2026-04-09
**Environments:** Development
**Partner Portal:** https://dev-ubn-ui.onecluster.co
**Admin Portal:** https://dev-admin.onecluster.co

---

## Part A: Partner Portal Walkthrough

**URL:** https://dev-ubn-ui.onecluster.co

---

### 1. Registration Flow

**Entry Point:** Navigate to `/` — automatically redirects to `/auth/login`

**Step 1: Navigate to Sign Up**
- From the login page, click the **"Sign Up"** link
- Redirects to `/auth/signup`

**Step 2: Fill Registration Form**

| Field | Type | Details |
|-------|------|---------|
| First Name | Text input | Required |
| Last Name | Text input | Required |
| Email | Email input | Required, must be valid format |
| Phone | Phone input | Pre-filled with `+234` country prefix (Nigeria) |
| Company Name | Text input | Required |
| Institution Type | Dropdown | Options: Fintech, Healthcare, Retail, Logistics, IT |
| Terms & Conditions | Checkbox | Must be checked to proceed |

**Step 3: OTP Verification**
- On successful form submission, user is redirected to OTP verification screen
- **6 individual digit input boxes** displayed (auto-focus advances to next box)
- **172-second countdown timer** shown (approximately 3 minutes)
- Timer displays in `MM:SS` format, counting down
- "Resend OTP" link becomes active after timer expires
- User enters the 6-digit code received via email/SMS

**Step 4: Completion**
- After successful OTP verification, user is redirected to `/auth/login`
- Success toast message displayed confirming registration

---

### 2. Login Flow

**URL:** `/auth/login`

**Form Fields:**
- **Email** — text input with email validation
- **Password** — text input with show/hide password toggle (eye icon)

**Authentication Process:**
1. User submits email and password
2. On success, server returns JWT token
3. Frontend decodes JWT to extract:
   - `partnerId` — unique partner identifier
   - `organizationId` — associated organization
   - `isOnboardingComplete` — boolean flag

**Post-Login Routing:**

| Condition | Redirect Target |
|-----------|----------------|
| `isOnboardingComplete === false` | `/dashboard/get-started` (6-step onboarding checklist) |
| `isOnboardingComplete === true` | `/dashboard/overview` (main dashboard) |

---

### 3. Onboarding Wizard

**URL:** `/dashboard/get-started`

The onboarding wizard presents a **6-step checklist** with progress tracking. Each step shows a checkmark when completed.

#### Step 1: Create Account
- **Status:** Auto-completed on registration
- No user action required — marked as done immediately

#### Step 2: Create Organization Profile
- **URL:** `/dashboard` (company details section)
- Form fields for:
  - Company legal name
  - Trading name
  - Registration number (CAC)
  - Tax Identification Number (TIN)
  - Industry sector
  - Company address
  - Company website
  - Company size

#### Step 3: Complete Business KYC
- **URL:** `/dashboard/kyc`
- **4-step sub-wizard:**

| Sub-Step | Description |
|----------|-------------|
| 1. Business Registration | CAC registration number, certificate upload, incorporation date |
| 2. Document Uploads | Certificate of incorporation, memorandum & articles, board resolution, utility bill |
| 3. Business Representatives | Director/shareholder details — name, BVN, ID document, proof of address |
| 4. Review Summary | Read-only summary of all entered data, submit for review button |

- After submission: KYB status changes to "Pending Review" — admin must approve

#### Step 4: Fill Compliance Questionnaire
- **URL:** `/dashboard/assessment`
- **6-step assessment:**

| Step | Section | Content |
|------|---------|---------|
| 1 | Organization Profile | Company structure, regulatory licenses, AML/CFT policies |
| 2 | Data Protection | NDPR compliance, data processing agreements, breach notification procedures |
| 3 | Information Security | ISO 27001 status, encryption standards, access control mechanisms |
| 4 | Risk Management | Risk assessment methodology, business continuity plans |
| 5 | Incident Management | Incident response plan, escalation procedures, SLA commitments |
| 6 | Technical Compliance | API security practices, penetration testing frequency, vulnerability management |

- Each section has a mix of dropdown selects, text areas, and file upload fields
- Progress saved automatically between sections

#### Step 5: Upload Signed SLA
- **URL:** `/dashboard/sla`
- Download SLA template (PDF)
- Upload signed SLA document (PDF/image)
- Admin reviews and countersigns

#### Step 6: Switch to Live
- **URL:** `/dashboard/overview`
- Toggle from Sandbox to Live environment
- Requires all previous steps to be completed and approved
- Live API credentials generated upon activation

---

### 4. Dashboard

#### 4.1 Overview (`/dashboard/overview`)

**Stats Cards (top row):**

| Card | Sample Value | Description |
|------|-------------|-------------|
| Total Transactions | ₦25.9M | Monetary value of all transactions |
| Transaction Volume | 234k | Count of transactions processed |
| Avg Latency | — | Average API response time |

**Charts:**
- **Transaction Chart** — Line/bar chart showing transaction trends over time
- **Channel Breakdown** — Donut chart showing distribution across payment channels (e.g., NIP, USSD, Cards)

#### 4.2 API Dashboard

- **Time Range Selector:** 7 days / 30 days / 90 days toggle
- **Request Volume Chart:** Bar chart showing daily API request counts
- **Per-Product Breakdown Table:**

| Product | Requests | Success Rate | Avg Latency |
|---------|----------|-------------|-------------|
| Accounts API | — | — | — |
| Payments API | — | — | — |
| Collections API | — | — | — |
| KYC API | — | — | — |

#### 4.3 Product Catalog

**Tabs:**
- **ALL PRODUCTS** — Full list of available API products
- **SUBSCRIBED** — Products the partner has access to

**Request Access Flow:**
1. Click "Request Access" on any unsubscribed product
2. Modal dialog appears with:
   - Product name and description (read-only)
   - Justification text area (**minimum 20 characters** required)
   - Submit button
3. Request goes to admin approval queue

#### 4.4 Settings

| Section | URL | Features |
|---------|-----|----------|
| User Profile | `/dashboard/settings/profile` | Edit name, email, phone |
| Organization Profile | `/dashboard/settings/organization` | Edit company details |
| API Credentials | `/dashboard/settings/credentials` | Generate new API keys, revoke existing keys, view key history |
| Security | `/dashboard/settings/security` | Change password with **strength meter** (weak/fair/strong/very strong), 2FA setup |
| Team Members | `/dashboard/settings/team` | Invite team members, assign roles, remove members |
| Audit Trail | `/dashboard/settings/audit` | Chronological log of all actions — login, key generation, settings changes |

---

## Part B: Admin Portal Walkthrough

**URL:** https://dev-admin.onecluster.co

---

### 1. Login

**Entry Point:** Navigate to `/` — automatically redirects to `/Auth/Login`

**Branding:**
- Page title: **"Pegasus Vendor Management Admin"**
- UBN logo displayed prominently

**Login Form:**
- **Email** — text input, required
- **Password** — text input, minimum 6 characters, show/hide toggle (eye icon)
- "Forgot Password?" link

**MFA Flow:**
1. Submit email + password
2. On success → redirected to `/Auth/Token`
3. **MFA token entry** — single text input field (6 characters)
4. Token sent to admin's registered email/authenticator
5. On token success → redirected to `/Dashboard`

---

### 2. Dashboard

**URL:** `/Dashboard`

**System Metrics (top cards):**

| Metric | Description |
|--------|-------------|
| System Uptime | Percentage uptime for the platform |
| Avg Latency | Average response time across all APIs |
| Traffic | Total API requests in current period |

**KYB Notification List:**
- Displays **top 5 pending** KYB review requests
- Each item shows: business name, submission date, priority
- Click any item to navigate to KYB review queue

**Charts:**
- **Pie Chart** — Distribution of vendors by status (Active, Pending, Suspended)
- **Line Chart** — API traffic trends over time

**Time Period Selector:**
- Dropdown to filter dashboard data by time range (Today, 7 days, 30 days, 90 days)

---

### 3. KYB Review Queue

**URL:** `/Dashboard/Compliance&Approvals/KYB&GoLive`

**Queue List View:**

| Column | Description |
|--------|-------------|
| Business Name | Legal name of the applying business |
| Reference ID | Unique KYB application reference |
| Entity Type | Company type (Limited, PLC, etc.) |
| Risk Level | Badge: Low (green), Medium (amber), High (red) |
| SLA | Time remaining for review completion |
| Assessment Score | Compliance questionnaire score |

**Review Process:**

1. Click **"Start Review"** on any queue item
2. **KYB Detail Modal** opens with sections:
   - **Organization Info** — company name, registration number, address, industry
   - **Business Documents** — uploaded documents with **download** buttons (certificate of incorporation, memorandum, board resolution)
   - **Bank Account** — settlement account details
   - **Business Representatives** — director/shareholder details with ID documents
   - **KYB Status** — current review status and history

**Review Actions:**

| Action | Button Color | Requirements |
|--------|-------------|-------------|
| **Approve** | Green | One-click approval, confirms KYB completion |
| **Flag Request** | Amber | Must select **reason from dropdown** + provide **minimum 20 characters** of instructions for the partner |
| **Reject** | Red | Must provide **reason in textarea** explaining the rejection |

---

### 4. Vendor Management

**URL:** `/Dashboard/Vendors&Partners/AllVendors`

**Vendor List View:**

| Column | Description |
|--------|-------------|
| Vendor Name | Business name |
| Health Status | Service health indicator |
| KYB Status | Approved / Pending / Rejected |
| Creation Date | Date vendor was onboarded |

**Vendor Detail View (click row to expand):**

| Tab | Content |
|-----|---------|
| **Overview** | Company details, contact info, account status, subscription tier |
| **API & Apps** | API keys issued, active applications, rate limits, subscription products |
| **Analytics** | Transaction volumes, API usage charts, error rates, latency percentiles |
| **Consents** | Data processing consents, terms acceptance history |
| **Audit** | Complete activity log for this vendor — logins, API calls, config changes |

**Actions:**
- **Suspend** button — immediately revokes API access, changes status to Suspended
- **Unsuspend** button — restores API access for previously suspended vendors

---

### 5. Other Admin Screens

#### 5.1 User Management

| Screen | URL | Features |
|--------|-----|----------|
| Internal Users | `/Dashboard/Users/InternalUsers` | Admin team members, roles, last login, status |
| API Portal Users | `/Dashboard/Users/ApiPortalUsers` | Partner users registered through the portal |

#### 5.2 Roles & Permissions

- **RBAC Matrix** displayed as a grid
- Rows: roles (Super Admin, Admin, Reviewer, Viewer, etc.)
- Columns: permissions (View KYB, Approve KYB, Manage Users, View Analytics, etc.)
- **Checkboxes** at each intersection to grant/revoke permissions
- Changes saved immediately on toggle

#### 5.3 API Catalog

**URL:** `/Dashboard/ApiCatalog`

- **Product Cards** displaying:
  - Product name and description
  - Usage metrics (requests, active subscribers)
  - Status badge (Active, Beta, Deprecated)
- **Category Filters:** Banking, KYC, Payments, Collections, Sandbox
- **"Create API Service"** wizard:
  - Step 1: Basic info (name, description, category)
  - Step 2: Endpoint configuration
  - Step 3: Rate limiting and access controls
  - Step 4: Review and publish

#### 5.4 System Health

- **7 service cards** displayed in a grid layout
- Each card shows:
  - Service name
  - **Status badge**: Healthy (green), Degraded (amber), Down (red)
  - Last health check timestamp
  - Uptime percentage

#### 5.5 Analytics & Compliance

| Screen | Description |
|--------|-------------|
| Commercial Analytics | Revenue metrics, transaction fees, partner billing summaries |
| SLAs & Breaches | SLA compliance tracking, breach notifications, penalty calculations |
| Audit Logs | Platform-wide audit trail with filters (user, action, date range, resource) |

#### 5.6 Settings

**All Settings** — displayed as a grid of setting categories:

| Setting | Description |
|---------|-------------|
| System Configuration | Platform-wide settings, feature flags, environment toggles |
| Security & Integrations | Third-party service credentials, webhook URLs, security policies |
| Manual Provisioning | Manually create partner accounts, assign products, generate keys |
| Developer Portal Settings | Customize developer portal branding, documentation links, API playground |
| Billing | Manage subscription plans and pricing |

**Billing Plans:**

| Plan | Price | Features |
|------|-------|----------|
| **Starter** | Free | Basic API access, sandbox only, 1,000 requests/month |
| **Growth** | $49/month | Production access, 50,000 requests/month, email support |
| **Enterprise** | $299/month | Unlimited requests, dedicated support, custom SLAs, priority queue |

---

## Navigation Summary

### Partner Portal Site Map

```
/auth/login
/auth/signup
/auth/otp-verify
/dashboard/get-started
/dashboard/overview
/dashboard/api
/dashboard/products
/dashboard/kyc
/dashboard/assessment
/dashboard/sla
/dashboard/settings/profile
/dashboard/settings/organization
/dashboard/settings/credentials
/dashboard/settings/security
/dashboard/settings/team
/dashboard/settings/audit
```

### Admin Portal Site Map

```
/Auth/Login
/Auth/Token
/Dashboard
/Dashboard/Compliance&Approvals/KYB&GoLive
/Dashboard/Vendors&Partners/AllVendors
/Dashboard/Users/InternalUsers
/Dashboard/Users/ApiPortalUsers
/Dashboard/Roles&Permissions
/Dashboard/ApiCatalog
/Dashboard/SystemHealth
/Dashboard/Analytics/Commercial
/Dashboard/Analytics/SLAs
/Dashboard/AuditLogs
/Dashboard/Settings
/Dashboard/Settings/SystemConfiguration
/Dashboard/Settings/Security
/Dashboard/Settings/ManualProvisioning
/Dashboard/Settings/DeveloperPortal
/Dashboard/Settings/Billing
```

---

*Document generated: 2026-04-09*
*Classification: Internal — Project Pegasus*
