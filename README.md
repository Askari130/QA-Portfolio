# 👋 Muqaddis Olopade — QA Portfolio

**Software Quality Assurance Engineer** | Manual & Automated Testing | Fintech & Marketplace Platforms

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Muqaddis%20Olopade-blue)](https://www.linkedin.com/in/muqaddis-olopade-76798425b/) [![GitHub](https://img.shields.io/badge/GitHub-Askari130-black)](https://github.com/Askari130) [![Email](https://img.shields.io/badge/Email-muqaddiso%40gmail.com-red)](mailto:muqaddiso@gmail.com)

---

## 📁 What's in This Repo

| Path | What it is |
| :--- | :--- |
| [`test-cases/RideSeat_MVP_Test_Cases.xlsx`](test-cases/RideSeat_MVP_Test_Cases.xlsx) | Full RideSeat MVP test suite (147 cases, 11 modules) with coverage summary sheet |
| [`test-cases/RideSeat_MVP_Test_Cases.csv`](test-cases/RideSeat_MVP_Test_Cases.csv) | Same test suite as CSV, viewable directly on GitHub |
| [`test-cases/RideSeat_MVP_Coverage_Summary.csv`](test-cases/RideSeat_MVP_Coverage_Summary.csv) | Coverage by module and priority |
| [`docs/QA_Mission_Scope_and_Guardrails.md`](docs/QA_Mission_Scope_and_Guardrails.md) | QA strategy: scope, test plan, entry/exit criteria, smoke & release checklists, 10-week timeline, and Requirements Traceability Matrix (RTM) |

---

## 🧪 About Me

Quality Assurance Engineer with **3+ years of experience** in manual and automated testing across fintech and marketplace platforms. I bring a developer's mindset to QA — having built projects with HTML, CSS, JavaScript, and React, I understand how software is built, which helps me find defects that surface-level testers miss.

I specialize in:

- Building structured test suites from scratch (147 test cases on a single MVP)
- Security testing: XSS, brute-force, user enumeration, data exposure
- API validation with Postman across RESTful endpoints
- UI automation with Selenium WebDriver
- Fintech-specific testing: Stripe payments, OAuth flows, escrow logic

---

## 📊 At a Glance

| Metric | Value |
| :--- | :--- |
| Years of QA Experience | 3+ |
| Test Cases Written (RideSeat MVP) | 147 |
| Modules Covered | 11 |
| Security Test Cases | 10 |
| Testing Types | Functional, Negative, Edge Case, Security, UI, Non-Functional |
| Tools | Selenium, Postman, TestRail, Jira, Chrome DevTools |

---

## 💼 Work Experience

### Quality Assurance Engineer — RideWay *(Feb 2026 – September 2026)*

> RideSeat is a web-based carpooling platform enabling drivers to publish trips and travelers to book affordable rides.

- Designed and executed **147 test cases** across 11 modules from scratch
- Authored **10 dedicated security test cases** (XSS, brute-force, user enumeration, IDOR, access control, data exposure)
- Validated **Stripe escrow-like payment integration** and **Google OAuth** authentication
- Tested full booking lifecycle: Pending → Confirmed → Cancelled → Completed
- Managed bug lifecycle end-to-end using **Jira**, collaborating with dev team

### Quality Assurance Engineer — Gadapay *(Apr 2025 – Jan 2026)*

> Gadapay is a fintech company building secure payment solutions for the African market.

- Performed manual and automated testing for web-based fintech applications
- Wrote and managed test cycles using **TestRail**
- Implemented **Selenium UI automation** and **Postman API testing** for payment flows
- Tracked and resolved bugs end-to-end via Jira

---

## 🗂️ Case Study: RideWay MVP Test Suite

### Module Coverage

| Module | Test Cases | High | Medium | Low |
| :--- | :--- | :--- | :--- | :--- |
| Authentication & Accounts | 28 | 12 | 13 | 3 |
| Trip Creation (Driver) | 27 | 7 | 16 | 4 |
| Trip Discovery (Search) | 13 | 5 | 7 | 1 |
| Payments | 13 | 7 | 5 | 1 |
| User Profiles | 12 | 1 | 8 | 3 |
| Booking Flow | 12 | 9 | 3 | 0 |
| Non-Functional / Cross-Cutting | 11 | 4 | 5 | 2 |
| Messaging | 9 | 4 | 3 | 2 |
| Ratings & Reviews | 8 | 3 | 4 | 1 |
| Trip Detail Page | 7 | 5 | 2 | 0 |
| Admin | 7 | 4 | 3 | 0 |
| **Total** | **147** | **61** | **69** | **17** |

### Test Type Distribution

| Type | Count |
| :--- | :--- |
| Functional | 64 |
| Negative | 37 |
| Edge Case | 20 |
| Security | 10 |
| Non-Functional | 9 |
| UI/UX | 7 |

> **Note:** The PRD did not explicitly specify a booking-approval model (auto-confirm vs. driver-approval), so several test cases (BOOK-005, BOOK-008, BOOK-009) are written to be re-verified against whichever model is implemented. UI-specific assertions (exact copy, disabled-state styling) should be cross-checked manually against the Figma designs.

---

## 🔒 Security Testing Highlight

| ID | Scenario | Priority |
| :--- | :--- | :--- |
| AUTH-014 | Login with non-existent email — no user enumeration | High |
| AUTH-015 | Brute force login — rate limiting / lockout triggered | High |
| AUTH-017 | Password reset with unregistered email — no enumeration | Medium |
| PROF-008 | Private fields hidden on other users' public profiles | High |
| SRCH-012 | Search fields sanitize special characters / SQL injection | High |
| DET-007 | Notes field renders as plain text (no XSS execution) | High |
| MSG-007 | Message input sanitized against script injection | High |
| ADMIN-001 | Non-admin and logged-out users blocked from admin routes | High |
| NFR-009 | Users cannot open other users' bookings or chats by changing IDs (IDOR) | High |
| NFR-010 | Passwords and sensitive data never exposed in responses, URLs or logs | High |

---

## 🧩 Sample Test Cases

### Authentication

**ID:** AUTH-001 | **Priority:** High | **Type:** Functional  
**Scenario:** Sign up with valid email & password  
**Steps:** 1. Go to Sign Up → 2. Enter credentials → 3. Tap Sign Up  
**Expected:** Account created; verification email sent; success screen shown

**ID:** AUTH-007 | **Priority:** Medium | **Type:** Edge Case  
**Scenario:** Google OAuth with email that already has a password account  
**Expected:** System links accounts or shows clear conflict message — no silent duplicate

### Payments

**ID:** PAY-001 | **Priority:** High | **Type:** Functional  
**Scenario:** Successful payment with valid card via Stripe  
**Expected:** Payment processed; booking moves to Confirmed; receipt generated

**ID:** PAY-002 | **Priority:** High | **Type:** Negative  
**Scenario:** Payment fails — card declined  
**Expected:** Clear error shown; booking stays Pending; no charge applied

---

## 🛠️ Tech Stack

**Testing:** Selenium WebDriver · Postman · TestRail · Jira · Chrome DevTools  
**Languages:** Python · JavaScript · HTML · CSS · React  
**Databases:** MySQL · MongoDB  
**CI/CD & VCS:** Git · GitHub · GitLab

---

## 🎓 Education

- **BSc Technology Education** — University of Lagos (2023 – 2027)
- **Python & Software Engineering** — Cisco Networking Academy (2021)

---

## 📬 Contact

📧 [muqaddiso@gmail.com](mailto:muqaddiso@gmail.com) | 🔗 [LinkedIn](https://www.linkedin.com/in/muqaddis-olopade-76798425b/) | 🐙 [GitHub](https://github.com/Askari130) | 🌐 [Linktree](https://linktr.ee/Askari_123)
