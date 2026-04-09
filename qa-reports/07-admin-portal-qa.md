# QA Report 07: Admin Portal (Ubn-API-Portal-Admin)

**Project:** OneCluster UBN Platform
**Report ID:** QA-RPT-007
**Sprint Period:** March 31 - April 9, 2026
**Prepared By:** QA Team
**Date:** April 9, 2026
**Status:** PASS (with noted issues)

---

## 1. Executive Summary

This report covers quality assurance testing for the Admin Portal (Ubn-API-Portal-Admin), the internal-facing management interface for the UBN platform. The Admin Portal enables bank operations staff to manage partner onboarding (KYB review), vendor lifecycle management, role-based access control, system health monitoring, and API catalog administration.

During this sprint, 43 commits were merged with significant feature additions including completed billing screens, the renaming of Operations SIAs to SLAs, token/logo fixes, and KYB detail modal improvements. All 25 Playwright UI test cases passed against the dev-admin.onecluster.co environment. Three issues were identified: missing security headers (PEG-377), incorrect page title showing "Create Next App" (PEG-385), and the API catalog being publicly accessible without authentication (PEG-378).

---

## 2. Scope

### Components Under Test

| Component | Description |
|---|---|
| Login + MFA Flow | Admin user authentication with multi-factor authentication |
| KYB Review Queue | Partner Know-Your-Business application review list |
| KYB Case Detail Modal | Detailed view of individual KYB submissions |
| Vendor Management | Partner suspend/unsuspend lifecycle controls |
| Roles & Permissions | RBAC matrix for admin user access control |
| System Health Monitoring | Service status dashboard for platform components |
| API Catalog Management | CRUD operations for API product definitions |
| Billing Screens | Partner billing and invoice management (NEW) |
| SLA Operations | Service Level Agreement tracking (renamed from SIA) |

### Test Environment

| Property | Value |
|---|---|
| URL | https://dev-admin.onecluster.co |
| Test Framework | Playwright |
| Browser Matrix | Chromium (primary), Firefox, WebKit |
| Test Data | Seeded admin accounts with various RBAC roles |

---

## 3. Changes This Sprint

**Total Commits:** 43

| Change | Description | Impact |
|---|---|---|
| Billing screens completed | Full billing management UI with invoice listing, payment history, and partner billing configuration | High — new feature area |
| Operations SIAs renamed to SLAs | Terminology alignment across the platform from "SIA" to "SLA" | Low — cosmetic, but affects navigation and labels |
| Token/logo fix | Corrected authentication token handling and logo rendering issues | Medium — visual and auth impact |
| KYB detail modal improvements | Enhanced layout, document preview, and approval/rejection workflow in the KYB case detail modal | Medium — improves reviewer workflow |
| Navigation updates | Sidebar menu restructuring to accommodate new billing and SLA sections | Low — UI layout change |

---

## 4. Test Results

### 4.1 Playwright UI Tests

**Environment:** dev-admin.onecluster.co
**Result:** 25/25 PASS

| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| ADM-001 | Login page renders | Login form with email and password fields | Form rendered correctly | PASS | — |
| ADM-002 | Valid admin login | Redirect to dashboard | Dashboard loaded | PASS | JWT stored in httpOnly cookie |
| ADM-003 | Invalid credentials rejection | Error message displayed | "Invalid credentials" shown | PASS | No credential leakage |
| ADM-004 | MFA prompt after login | MFA code input displayed | MFA input shown | PASS | 6-digit OTP field |
| ADM-005 | Valid MFA code submission | Access granted to dashboard | Dashboard accessible | PASS | — |
| ADM-006 | Invalid MFA code rejection | Error message, retry allowed | Error shown, retry available | PASS | Max 3 attempts enforced |
| ADM-007 | KYB Review Queue loads | List of pending KYB applications | Queue populated | PASS | Pagination working |
| ADM-008 | KYB Queue filtering | Filter by status (pending/approved/rejected) | Filters applied correctly | PASS | — |
| ADM-009 | KYB Case Detail Modal opens | Modal with partner details, documents | Modal rendered with all sections | PASS | Document preview loads |
| ADM-010 | KYB Approve action | Status updated to Approved | Status changed, notification sent | PASS | Audit log entry created |
| ADM-011 | KYB Reject action with reason | Status updated to Rejected, reason stored | Rejection recorded | PASS | Reason field required |
| ADM-012 | Vendor list loads | Table of registered partners | Partners listed with status indicators | PASS | — |
| ADM-013 | Vendor suspend action | Partner status set to Suspended | API access revoked immediately | PASS | Confirmation dialog shown |
| ADM-014 | Vendor unsuspend action | Partner status set to Active | API access restored | PASS | — |
| ADM-015 | Roles & Permissions page loads | RBAC matrix displayed | Matrix rendered correctly | PASS | All roles visible |
| ADM-016 | Role assignment | Assign role to admin user | Role updated in user record | PASS | Permission changes immediate |
| ADM-017 | Permission enforcement — unauthorized action | Action blocked with 403 | Access denied message shown | PASS | RBAC enforced client and server side |
| ADM-018 | System Health dashboard | Service status indicators | All monitored services shown | PASS | Color-coded status (green/red/yellow) |
| ADM-019 | API Catalog listing | List of available API products | Products displayed with metadata | PASS | — |
| ADM-020 | API Catalog — create product | New product appears in catalog | Product created successfully | PASS | Form validation working |
| ADM-021 | API Catalog — edit product | Product details updated | Changes saved and reflected | PASS | — |
| ADM-022 | Billing screen — invoice list | Partner invoices displayed | Invoices listed with amounts and dates | PASS | NEW feature this sprint |
| ADM-023 | Billing screen — payment history | Payment records with status | History loaded correctly | PASS | NEW feature this sprint |
| ADM-024 | SLA Operations page | SLA metrics and tracking (renamed from SIA) | Page loads with correct "SLA" labeling | PASS | Terminology updated throughout |
| ADM-025 | Session timeout handling | Redirect to login after inactivity | Auto-redirect after 30 min idle | PASS | Warning shown at 25 min |

---

## 5. Issues Found

| Issue ID | Severity | Title | Status | Description |
|---|---|---|---|---|
| PEG-377 | Medium | Missing security headers on Admin Portal responses | Open | The following HTTP security headers are absent from responses: `X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy`, `Strict-Transport-Security`. This exposes the admin portal to clickjacking, MIME-type sniffing, and other client-side attacks. Should be configured at the Next.js middleware or reverse proxy level. |
| PEG-385 | Low | Page title shows "Create Next App" instead of portal name | Open | The HTML `<title>` tag on all pages displays the default Next.js boilerplate title "Create Next App" instead of "OneCluster Admin Portal" or a page-specific title. This affects browser tab identification and SEO (though SEO is not a concern for an internal tool, it is unprofessional in appearance). |
| PEG-378 | High | API Catalog pages accessible without authentication | Open | The `/catalog` and `/catalog/[productId]` routes are accessible without a valid admin session. An unauthenticated user can view the full API product catalog including endpoint specifications, pricing tiers, and integration documentation. This should be behind the authentication middleware. |

---

## 6. Recommendations

1. **PEG-378 — Authentication on Catalog (High Priority):** Add authentication middleware to all `/catalog` routes immediately. Even if catalog information is not classified, unauthenticated access to the admin portal sets a dangerous precedent and may expose internal pricing or unreleased API product details.

2. **PEG-377 — Security Headers:** Implement security headers via Next.js middleware (`next.config.js` headers configuration) or at the reverse proxy level. Recommended headers:
   - `X-Content-Type-Options: nosniff`
   - `X-Frame-Options: DENY`
   - `Content-Security-Policy: default-src 'self'`
   - `Strict-Transport-Security: max-age=31536000; includeSubDomains`
   - `X-XSS-Protection: 1; mode=block`

3. **PEG-385 — Page Titles:** Update the Next.js layout or individual page metadata to reflect proper page titles. This is a quick fix in the root `layout.tsx` or via the `metadata` export in each page component.

4. **RBAC Audit:** Conduct a manual review of the RBAC permission matrix to ensure the principle of least privilege is applied. Verify that roles like "Viewer" cannot inadvertently trigger state-changing operations.

5. **Billing Screen Testing:** The new billing screens should receive additional testing with edge cases: empty invoice lists, large payment histories (pagination), and currency formatting for NGN values.

---

*End of Report*
