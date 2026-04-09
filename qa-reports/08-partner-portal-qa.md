# QA Report 08: Partner Portal (Ubn-API-Portal-Frontend)

**Project:** OneCluster UBN Platform
**Report ID:** QA-RPT-008
**Sprint Period:** March 31 - April 9, 2026
**Prepared By:** QA Team
**Date:** April 9, 2026
**Status:** PASS

---

## 1. Executive Summary

This report covers quality assurance testing for the Partner Portal (Ubn-API-Portal-Frontend), the external-facing web application used by partner organizations to register, complete onboarding, manage API credentials, and monitor their API usage on the UBN platform.

This sprint saw the **largest change set across all repositories** with 76 commits merged. Major changes include a complete KYC page overhaul (266 lines changed), reworked assessment questionnaire steps, a redesigned dashboard overview, a new SLA page, and a new profile underlay component. All 58 Playwright UI test cases passed against the dev-ubn-ui.onecluster.co environment. All authentication pages render correctly, navigation works end-to-end, and responsive layouts function properly on mobile and tablet viewports. No blocking issues were identified.

---

## 2. Scope

### Components Under Test

| Component | Description |
|---|---|
| Registration | New partner organization sign-up |
| Login | Partner user authentication |
| OTP Verification | One-time password email verification |
| Forgot Password | Password reset request flow |
| Reset Password | Password reset completion flow |
| KYC Onboarding Wizard | 4-step Know-Your-Customer onboarding process |
| Assessment Questionnaire | 6-step partner readiness assessment |
| Dashboard Overview | Partner home dashboard with key metrics |
| API Dashboard | API usage analytics and monitoring |
| Product Catalog | Available API products browsing |
| API Credentials | API key and secret management |
| Settings | Partner organization settings and preferences |
| SLA Page | Service Level Agreement monitoring (NEW) |
| Profile Underlay | Profile management slide-out panel (NEW) |

### KYC Onboarding Wizard Steps

| Step | Content |
|---|---|
| Step 1 — Business Registration | Company name, RC number, incorporation date, business type |
| Step 2 — Documents | CAC certificate, memorandum of association, utility bill uploads |
| Step 3 — Representatives | Director/shareholder details, BVN verification, ID uploads |
| Step 4 — Review & Submit | Summary review of all entered data before submission |

### Assessment Questionnaire Steps

| Step | Content |
|---|---|
| Step 1 — Business Overview | Industry sector, annual revenue, customer base size |
| Step 2 — Technical Readiness | Development team size, infrastructure, security practices |
| Step 3 — Compliance | Data protection policies, regulatory compliance status |
| Step 4 — Use Case | Intended API products, expected volumes, integration timeline |
| Step 5 — Security Assessment | Penetration testing, vulnerability management, incident response |
| Step 6 — Review & Submit | Final review and submission |

### Test Environment

| Property | Value |
|---|---|
| URL | https://dev-ubn-ui.onecluster.co |
| Test Framework | Playwright |
| Browser Matrix | Chromium (primary), Firefox, WebKit |
| Viewport Testing | Desktop (1920x1080), Tablet (768x1024), Mobile (375x812) |

---

## 3. Changes This Sprint

**Total Commits:** 76 (largest change set this sprint)

| Change | Description | Impact |
|---|---|---|
| KYC page overhaul | Complete redesign and restructuring of the KYC onboarding wizard (266 lines changed) | High — core onboarding flow rewritten |
| Assessment steps reworked | Questionnaire steps reorganized with improved validation and progress tracking | High — affects partner onboarding journey |
| Dashboard overview redesigned | New layout with updated metric cards, charts, and activity feed | Medium — visual and UX improvements |
| SLA page added | New page displaying SLA metrics, uptime targets, and compliance status | Medium — new feature for partners |
| Profile underlay component | Slide-out panel for profile viewing and editing from any page | Low — UX enhancement |
| Navigation restructuring | Sidebar updated to accommodate SLA page and new sections | Low — layout change |
| Responsive layout fixes | Mobile and tablet viewport adjustments across multiple pages | Medium — cross-device compatibility |

---

## 4. Test Results

### 4.1 Playwright UI Tests

**Environment:** dev-ubn-ui.onecluster.co
**Result:** 58/58 PASS

#### Authentication Tests

| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| PP-001 | Registration page renders | Form with org name, email, password | Form rendered correctly | PASS | All fields present with validation |
| PP-002 | Valid registration submission | Success message, redirect to OTP | Registration created, OTP page shown | PASS | Confirmation email triggered |
| PP-003 | Duplicate email registration | Error: email already registered | Error displayed | PASS | No information leakage about existing accounts |
| PP-004 | Login page renders | Email and password fields | Form rendered | PASS | — |
| PP-005 | Valid login | Redirect to dashboard | Dashboard loaded | PASS | Token stored securely |
| PP-006 | Invalid login credentials | Error message | "Invalid credentials" shown | PASS | Generic error, no leakage |
| PP-007 | OTP verification page renders | 6-digit code input | OTP input displayed | PASS | Auto-focus on first digit |
| PP-008 | Valid OTP submission | Account verified, redirect to login | Verification successful | PASS | — |
| PP-009 | Invalid OTP rejection | Error, retry allowed | Error shown | PASS | Rate limited after 5 attempts |
| PP-010 | OTP resend functionality | New OTP sent, cooldown timer | OTP resent, 60s cooldown shown | PASS | — |
| PP-011 | Forgot password page renders | Email input field | Form rendered | PASS | — |
| PP-012 | Forgot password submission | Success message (regardless of email existence) | Generic success shown | PASS | No email enumeration |
| PP-013 | Reset password page renders | New password + confirm fields | Form rendered | PASS | Via token in URL |
| PP-014 | Valid password reset | Password updated, redirect to login | Reset successful | PASS | Old sessions invalidated |
| PP-015 | Password reset — weak password | Validation error | Strength requirements shown | PASS | Min 8 chars, uppercase, number, special |

#### KYC Onboarding Wizard Tests

| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| PP-016 | KYC wizard — Step 1 renders | Business registration form | Form displayed with all fields | PASS | Overhauled layout this sprint |
| PP-017 | KYC wizard — Step 1 validation | Required fields enforced | Validation errors on empty submit | PASS | RC number format validated |
| PP-018 | KYC wizard — Step 1 completion | Advance to Step 2 | Step 2 loaded | PASS | Progress bar updated |
| PP-019 | KYC wizard — Step 2 renders | Document upload interface | Upload zones displayed | PASS | Drag-and-drop supported |
| PP-020 | KYC wizard — Step 2 file upload | File accepted, preview shown | Upload successful with preview | PASS | PDF and image formats |
| PP-021 | KYC wizard — Step 2 file size limit | Reject files over 5MB | Error for oversized files | PASS | Clear error message |
| PP-022 | KYC wizard — Step 3 renders | Representative details form | Form with director/shareholder fields | PASS | — |
| PP-023 | KYC wizard — Step 3 BVN field | 11-digit BVN input | BVN field with format validation | PASS | Numeric only, exact length |
| PP-024 | KYC wizard — Step 4 review | Summary of all entered data | All steps data displayed | PASS | Editable via "Back" navigation |
| PP-025 | KYC wizard — submission | Application submitted, confirmation shown | Submission successful | PASS | Status set to "Pending Review" |
| PP-026 | KYC wizard — progress persistence | Partially completed wizard saved | Data retained on page reload | PASS | Auto-save on step transition |
| PP-027 | KYC wizard — back navigation | Previous step loaded with data | Data preserved | PASS | No data loss on back navigation |

#### Assessment Questionnaire Tests

| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| PP-028 | Assessment — Step 1 renders | Business overview form | Form displayed | PASS | Reworked layout this sprint |
| PP-029 | Assessment — Step 2 renders | Technical readiness form | Form displayed | PASS | — |
| PP-030 | Assessment — Step 3 renders | Compliance questions | Form displayed | PASS | — |
| PP-031 | Assessment — Step 4 renders | Use case details | Form displayed | PASS | — |
| PP-032 | Assessment — Step 5 renders | Security assessment | Form displayed | PASS | — |
| PP-033 | Assessment — Step 6 review | Full summary before submission | All data summarized | PASS | — |
| PP-034 | Assessment — submission | Questionnaire submitted | Submission confirmed | PASS | — |
| PP-035 | Assessment — progress tracking | Step indicator shows current position | Progress bar accurate | PASS | Reworked progress component |
| PP-036 | Assessment — validation per step | Required fields enforced on each step | Validation errors shown | PASS | Cannot skip steps |

#### Dashboard & Feature Tests

| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| PP-037 | Dashboard overview renders | Metric cards, charts, activity feed | Dashboard displayed | PASS | Redesigned layout this sprint |
| PP-038 | Dashboard — API call volume chart | Usage chart with date range | Chart rendered with data | PASS | Interactive date picker |
| PP-039 | Dashboard — quick action cards | Links to common actions | Cards clickable, navigate correctly | PASS | — |
| PP-040 | API Dashboard renders | Detailed API analytics | Analytics page loaded | PASS | — |
| PP-041 | API Dashboard — per-product metrics | Breakdown by API product | Metrics per product shown | PASS | — |
| PP-042 | Product Catalog renders | Available API products listing | Products displayed | PASS | Cards with descriptions |
| PP-043 | Product Catalog — product detail | Product documentation and specs | Detail page loaded | PASS | — |
| PP-044 | API Credentials page renders | API key and secret display | Credentials shown (masked) | PASS | Copy-to-clipboard working |
| PP-045 | API Credentials — regenerate key | New key generated, old invalidated | Key regenerated | PASS | Confirmation dialog shown |
| PP-046 | Settings page renders | Organization settings form | Settings loaded | PASS | — |
| PP-047 | Settings — update organization name | Name updated | Change saved | PASS | — |
| PP-048 | SLA page renders | SLA metrics and targets | Page displayed with metrics | PASS | NEW feature this sprint |
| PP-049 | SLA page — uptime display | Uptime percentage shown | Percentage rendered | PASS | — |
| PP-050 | Profile underlay opens | Slide-out panel from avatar click | Panel slides in from right | PASS | NEW component this sprint |
| PP-051 | Profile underlay — edit profile | Profile fields editable | Changes saved | PASS | — |

#### Responsive & Navigation Tests

| Test ID | Test Case | Expected | Actual | Status | Notes |
|---|---|---|---|---|---|
| PP-052 | Mobile viewport — login page | Responsive layout, no horizontal scroll | Properly formatted | PASS | 375x812 viewport |
| PP-053 | Mobile viewport — dashboard | Cards stack vertically | Responsive grid applied | PASS | — |
| PP-054 | Mobile viewport — KYC wizard | Form fields full-width | Wizard usable on mobile | PASS | Step navigation accessible |
| PP-055 | Tablet viewport — dashboard | 2-column grid layout | Layout correct | PASS | 768x1024 viewport |
| PP-056 | Tablet viewport — product catalog | Cards in 2-column grid | Layout correct | PASS | — |
| PP-057 | Navigation — sidebar links | All links navigate to correct pages | All routes working | PASS | No broken links |
| PP-058 | Navigation — breadcrumbs | Breadcrumb trail accurate | Breadcrumbs correct | PASS | Clickable parent crumbs |

---

## 5. Issues Found

No blocking or critical issues were identified during this testing cycle. The Partner Portal is in a healthy state with all 58 test cases passing.

### Minor Observations (Not Tracked as Issues)

| Observation | Description | Recommendation |
|---|---|---|
| KYC wizard load time | Step transitions take 800-1200ms due to auto-save API calls | Consider optimistic UI updates with background saves |
| Assessment form labels | Some labels truncated on smaller mobile viewports (320px) | Not a supported viewport, but consider min-width CSS |
| Dashboard chart rendering | Chart library (Recharts) briefly shows empty state before data loads | Add skeleton loader to prevent layout shift |

---

## 6. Recommendations

1. **KYC Onboarding Performance:** The overhauled KYC wizard is functionally correct, but step transitions could be faster. Implement optimistic updates where form data is immediately displayed in the next step while the save request completes in the background.

2. **End-to-End Onboarding Flow:** Create a Playwright test that covers the full journey: Registration -> OTP -> Login -> KYC Wizard (all 4 steps) -> Assessment (all 6 steps) -> Dashboard. This would validate the complete partner onboarding experience as a single workflow.

3. **Accessibility Testing:** With 76 commits and significant UI changes, conduct an accessibility audit (WCAG 2.1 AA) focusing on the new KYC wizard, assessment questionnaire, and dashboard overview. Check keyboard navigation, screen reader compatibility, and color contrast.

4. **Performance Baseline:** Establish Lighthouse performance baselines for key pages (Dashboard, KYC Wizard, Product Catalog) now that the major redesign is complete. Track scores across future sprints.

5. **SLA Page Data Source:** Verify that the new SLA page pulls data from actual monitoring metrics rather than static values. Ensure the page accurately reflects real uptime data once production monitoring is connected.

---

*End of Report*
