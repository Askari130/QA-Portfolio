# RideSeat QA Mission, Scope, and Guardrails

**Contents:** [QA Mission](#qa-mission) · [My Daily Rhythm](#my-daily-rhythm) · [10-Week Timeline](#10-week-timeline-day-by-day-focus-monfri) · [RTM](#rtm)

---

# QA Mission

## 1) QA Mission, Scope, and Guardrails

### Product scope (MVP)

Web app where:

- **Drivers** post planned inter/intra-city trips with available seats.
- **Passengers** search, book, and pay for seats.
- Includes **basic trust** (profiles, ratings) and **messaging** between matched users.

### Explicit MVP non-goals (don't waste cycles here)

No: native apps, real-time GPS tracking, surge/dynamic pricing, corporate fleets, multi-stop routing, insurance/claims.

---

## 2) What You Need To Do as QA (End-to-End)

### A. Requirements & Risk Pass (before writing test cases)

1. Convert PRD into a **Requirements Traceability Matrix (RTM)**:
   - Map each PRD feature + acceptance criteria (US-1 → US-6) to test cases.
2. Identify **ambiguities** (document them as "Open QA Questions"):
   - Example: the booking approval model can be **Pending or Confirmed depending on driver approval**. Testing must cover both states.

### B. Test Planning (per sprint)

- Maintain:
  - **Smoke suite** (5–10 mins) for every build
  - **Sprint functional suite** (new work)
  - **Regression suite** (critical paths)
  - **Release checklist** (go/no-go)

### C. Test Environments & Data

**Environments**

- Dev (fast iteration), Staging (release candidate), Prod (only smoke).

**Must-have test integrations**

- Email service in test mode (verification + reset)
- Stripe **test keys** for Payment Intents + Connect (escrow-like hold + payout on completion)

**Test data sets**

- Users: Driver-only, Passenger-only, Both
- Trips: past date, today, future; 1 seat; 6 seats; luggage yes/no; long notes; different cities
- Payments: success, failure, cancel/abandon, duplicate submit
- Bookings: pending, confirmed, cancelled, completed

### D. Test Types You Should Run

1. **Functional UI** (happy path + negative)
2. **API tests** (auth, trips, bookings, payments webhooks, chat, reviews)
3. **Integration** (Stripe + email + DB consistency)
4. **Access control / Security basics** (OWASP top checks: authz, IDOR, rate-limits on auth, CSRF if applicable)
5. **Compatibility**: Chrome/Edge/Firefox + mobile web (iOS Safari, Android Chrome)
6. **Responsive UX**: key pages on small/medium/large viewports (MVP optimized for desktop/mobile web)
7. **Accessibility quick pass** (keyboard nav, focus states, form labels, contrast)
8. **Performance sanity** (search results load time; chat send latency; payment callback handling)

### E. Automation Strategy (recommended)

- **Playwright/Cypress** for critical UI flows:
  - Register → verify → login
  - Driver post trip → passenger search → book → pay
  - Messaging after booking
  - Rating after completion
- **Postman/Newman** or **contract tests** for core APIs
- CI runs: smoke on PRs; regression nightly

### F. Entry/Exit Criteria (simple and enforceable)

**Entry (start testing a feature)**

- Acceptance criteria written and understood for the story (US-1..US-6)
- Feature deployed to test env + logging enabled

**Exit (feature sign-off)**

- 0 open **blocker/critical**
- All "P0" tests passed for that module
- No data integrity issues (seats, payments, booking states)

---

## 3) Core Test Suites (High-Level)

### P0 Critical Paths (must pass for release)

1. Signup + email verification + login
2. Driver creates trip (validations) and appears in search
3. Passenger searches and views trip detail
4. Passenger books seats + payment success → booking status updates
5. Seat count decrements correctly (no oversell)
6. Messaging available after booking request; order preserved
7. Trip completion → payout eligibility; rating allowed only after completion
8. Admin can view users/trips/bookings and resolve disputes

---

## 4) Detailed Test Cases (Catalog)

> Format: **ID — Title (Priority)** → Preconditions | Steps | Expected
>
> The full, executable suite (147 cases) lives in [`../test-cases/`](../test-cases/).

### A) Authentication & Accounts (Email/Password, Google OAuth optional, verification, reset)

**AUTH-001 — Signup with valid email/password (P0)**  
Pre: none | Steps: open signup → enter valid data → submit | Expected: account created; verification email sent.

**AUTH-002 — Login blocked before email verification (P0)**  
Pre: unverified account | Steps: attempt login | Expected: user blocked; clear message prompting verification.

**AUTH-003 — Verify email with valid token/link (P0)**  
Pre: verification email | Steps: open link | Expected: account becomes verified; user can log in.

**AUTH-004 — Verify email with expired/invalid token (P1)**  
Expected: error shown; option to resend verification.

**AUTH-005 — Password reset request (P0)**  
Steps: "Forgot password" → submit registered email | Expected: reset email sent.

**AUTH-006 — Password reset with valid token (P0)**  
Steps: open reset link → set new password | Expected: password updated; login works with new password.

**AUTH-007 — Password reset with invalid token (P1)**  
Expected: error + re-request reset.

**AUTH-008 — Signup with existing email (P0)**  
Expected: blocked; informative error.

**AUTH-009 — Weak password policy enforced (P1)**  
Expected: client/server validation consistent.

**AUTH-010 — Optional Google OAuth sign-in works (P1)**  
Expected: account created/logged in; role selection handled.

**AUTH-011 — Logout ends session (P0)**  
Expected: protected pages require re-auth.

**AUTH-012 — Session persistence across refresh (P1)**  
Expected: user remains logged in until expiry/logout.

---

### B) Roles & Profiles (Driver/Passenger/Both; public vs private fields)

**PROF-001 — Select role: Driver / Passenger / Both (P0)**  
Expected: features gated correctly (driver can post trips; passenger can book).

**PROF-002 — Public profile shows required visible fields (P0)**  
Expected: avatar (if any), first name, rating+count, bio, trips completed visible.

**PROF-003 — Private fields not exposed to other users (P0)**  
Expected: email/phone/payment details hidden to others.

**PROF-004 — Bio max 300 chars enforced (P1)**  
Expected: cannot exceed; server rejects over-limit.

**PROF-005 — Profile photo upload optional (P2)**  
Expected: can skip; can add later; file-type/size validated.

**PROF-006 — Rating count and average display correctly (P1)**  
Expected: rounding rules consistent; count increments after reviews.

---

### C) Trip Creation (Driver) — Inputs + rules

**TRIP-001 — Create trip with valid required fields (P0)**  
Steps: fill origin/destination/date/time/seats/price/vehicle/luggage/notes → submit | Expected: trip created.

**TRIP-002 — Origin/Destination autocomplete behaves (P2)**  
Expected: suggestions shown; value saved correctly.

**TRIP-003 — Cannot create trip in the past (P0)**  
Expected: blocked client+server; clear error.

**TRIP-004 — Available seats range enforced (1–6) (P0)**  
Expected: cannot set 0 or 7+; server validation matches.

**TRIP-005 — Price per seat validation (P0)**  
Expected: rejects zero/negative; handles decimals properly.

**TRIP-006 — Notes max 500 chars enforced (P1)**

**TRIP-007 — Vehicle details required (make/model/colour) (P1)**

**TRIP-008 — Luggage allowed toggle saved (P2)**

**TRIP-009 — "Price × seats = total potential earnings" computed correctly (P2)**

**TRIP-010 — Edit trip: cannot reduce seats below confirmed bookings (P0)**  
Pre: trip with confirmed bookings | Steps: attempt reduce seats below confirmed | Expected: blocked.

**TRIP-011 — Edit trip: reduce seats to exactly confirmed count (P1)**  
Expected: allowed; seats remaining becomes 0.

**TRIP-012 — Delete/cancel trip behavior (P1)**  
Expected: if supported, state changes + booked passengers handled (document actual behavior).

---

### D) Trip Discovery (Search + results sorted)

**SEARCH-001 — Search by origin+destination+date (P0)**  
Expected: returns relevant trips only.

**SEARCH-002 — Filter by number of passengers (P0)**  
Expected: trips returned must have seats remaining >= requested passengers.

**SEARCH-003 — Results sorted by departure time (P0)**

**SEARCH-004 — Empty state when no trips (P1)**  
Expected: clear message; suggest changing filters.

**SEARCH-005 — Each trip card shows required fields (P1)**  
Driver name+rating, departure time, price/seat, seats remaining, vehicle summary.

**SEARCH-006 — Pagination / load more (if exists) (P2)**  
Expected: stable ordering; no duplicates.

---

### E) Trip Detail Page (display + CTA)

**DETAIL-001 — Trip details show full required info (P0)**  
Expected: route summary, date/time, price/seat, seats remaining, driver snippet, vehicle details, luggage, notes.

**DETAIL-002 — CTA text matches state ("Request Seat"/"Book Seat") (P1)**

**DETAIL-003 — No booking allowed when seats remaining = 0 (P0)**  
Expected: CTA disabled or blocked.

**DETAIL-004 — Passenger cannot see private driver info (P0)**  
Expected: email/phone/payment not shown (unless specifically allowed by product).

---

### F) Booking Flow + Booking States

**BOOK-001 — Select number of seats within availability (P0)**  
Expected: can't exceed seats remaining.

**BOOK-002 — Booking request confirmation step shown (P1)**  
Expected: user must confirm before payment.

**BOOK-003 — Payment success → booking status updates (P0)**  
Expected: status = Pending or Confirmed per approval model; booking ID generated.

**BOOK-004 — Seat decrement after booking (P0)**  
Expected: seats remaining reduces accurately; search + detail reflect new count.

**BOOK-005 — Prevent oversell under concurrency (P0)**  
Pre: 1 seat left | Steps: two users attempt booking simultaneously | Expected: only one succeeds; other gets "sold out".

**BOOK-006 — Driver receives booking notification (P1)**

**BOOK-007 — Driver accept booking → status Confirmed (P0)**  
Expected: passenger notified; chat rules apply.

**BOOK-008 — Driver reject booking → status Cancelled (P0)**  
Expected: passenger notified; seat count restored (if that's the intended behavior).

**BOOK-009 — Passenger cancels booking (P1)**  
Expected: status Cancelled; seat count behavior consistent; payment/refund behavior matches rules.

**BOOK-010 — Booking state transitions valid only (P0)**  
Expected: cannot go Cancelled → Confirmed; cannot rate before Completed; etc.

**BOOK-011 — Completed state set after trip completion (P0)**  
Expected: triggers payout eligibility + review availability.

---

### G) Payments (Stripe Payment Intents + Connect; escrow-like hold; platform fee; payout)

**PAY-001 — Successful card payment (P0)**  
Expected: payment status success; booking created/updated; receipt shown.

**PAY-002 — Payment fails (insufficient funds / declined) (P0)**  
Expected: booking not confirmed; clear error; user can retry.

**PAY-003 — User abandons payment (P1)**  
Expected: booking remains not-paid; no seat decrement (or auto-expire).

**PAY-004 — Double-submit protection (P0)**  
Expected: no duplicate charges/bookings on rapid clicks/refresh.

**PAY-005 — Webhook handling idempotency (P0)**  
Expected: repeated Stripe events don't duplicate bookings/transactions.

**PAY-006 — Platform fee calculated and stored (P1)**  
Expected: gross amount, fee (10–15%), net payout recorded.

**PAY-007 — Funds held until trip completion (P0)**  
Expected: driver payout not triggered before completion.

**PAY-008 — Payout triggered after completion (P0)**  
Expected: payout record created; driver receives net payout.

**PAY-009 — Cancellation handling (refund vs partial vs none) (P1)**  
Expected: behavior matches implemented policy; ensure data consistency.

---

### H) Messaging (1:1, text-only, timestamps, opens after booking request)

**CHAT-001 — Chat opens after booking request (P0)**  
Pre: booking requested | Steps: open chat | Expected: chat available.

**CHAT-002 — Chat blocked without booking request (P0)**  
Expected: no access to message a driver from discovery/detail.

**CHAT-003 — Only matched parties can access chat (P0)**  
Expected: other users cannot open chat thread (authorization).

**CHAT-004 — Messages stored with timestamps (P1)**

**CHAT-005 — Messages delivered in order (P0)**

**CHAT-006 — Empty state + first message send (P2)**  
Expected: good UX; send button states correct.

**CHAT-007 — After booking cancelled, chat access behavior (P1)**  
Expected: matches product decision (lock/read-only/allow).

---

### I) Ratings & Reviews (post-completion only; aggregation)

**RATE-001 — Passenger can rate driver only after Completed (P0)**

**RATE-002 — Attempt to rate before completion blocked (P0)**  
Expected: blocked at UI + API.

**RATE-003 — Rating range 1–5 enforced (P0)**  
Expected: cannot submit 0/6.

**RATE-004 — Optional comment saves (P2)**

**RATE-005 — Average rating updates correctly (P0)**

**RATE-006 — Rating count increments (P1)**

**RATE-007 — Driver can rate passenger (optional MVP) (P2)**  
Expected: only if implemented.

---

### J) Admin (basic view + dispute resolution)

**ADMIN-001 — Admin login/access control (P0)**  
Expected: non-admin blocked; admin-only routes protected.

**ADMIN-002 — View users list (P1)**

**ADMIN-003 — View trips list (P1)**

**ADMIN-004 — View bookings list (P1)**

**ADMIN-005 — Manually resolve dispute updates correct records (P0)**  
Expected: audit trail (who/when/what) and state/payment consistency.

**ADMIN-006 — Admin cannot break invariants (P0)**  
Expected: cannot set seats negative; cannot force payout without completion unless explicit override.

---

## 5) Smoke Test Checklist (Run on Every Build)

1. Signup → verification flow works
2. Login/logout works
3. Driver creates a future trip; appears in search
4. The passenger searches and opens the trip detail
5. Passenger books 1 seat + payment success updates booking
6. Seats decrement everywhere (search + detail)
7. Chat opens after the booking request

---

## 6) Release (Pre-Deploy) Go/No-Go Checklist

- P0 suite passes (no critical bugs)
- No payment inconsistencies (no double charge; webhook idempotency verified)
- Seat inventory integrity verified (no oversell)
- Ratings gated by completion (cannot rate early)
- Basic security checks: authz on trips/bookings/chat/admin; rate-limit login/reset
- Cross-browser sanity: Chrome + iOS Safari responsive critical pages
- Logs/monitoring enabled for auth, booking, and payment webhooks

---

## 7) Open QA Questions (Log these as Risks, not blockers)

These are areas the PRD leaves flexible; you still test what's implemented, but you should **track** the chosen behavior:

- Does booking become **Pending** until the driver approves, or instantly **Confirmed** after payment? (PRD says it depends on the approval model.)
- Cancellation policy: refund rules, payout reversal rules (not fully specified).
- When/how "Trip Completed" is set (driver action, time-based, admin action).

---

# My Daily Rhythm

## My daily rhythm (use this every weekday)

**09:00–09:20 — Build + smoke**

- Pull latest staging/dev build
- Run **P0 smoke** (login, search, create trip, book/pay test mode, chat open)
- If smoke fails → block merges/release candidate until fixed

**09:20–09:40 — Standup + priorities**

- Share: what passed/failed, top risks (payments, seat oversell, auth), what you're testing today

**09:40–11:30 — Feature testing (new stories)**

- Validate acceptance criteria for stories "done" yesterday
- Do happy path + edge/negative + authz checks (IDOR-ish: can user access others' booking/chat?)

**11:30–12:30 — Bug writing + triage**

- File bugs with: steps, expected vs actual, env, logs, screenshots, severity
- Update bug board (Blocker/Critical/Minor)

**13:30–15:00 — Regression + automation**

- Add tests to **regression pack** for anything that just shipped
- Automate at least **1–3 P0 flows per week** (UI + API)

**15:00–16:30 — Dev sync + re-tests**

- Pair with devs on reproductions
- Re-test fixes + close tickets

**16:30–17:00 — QA report**

- Daily report: what tested, what passed, new bugs, remaining risks, tomorrow plan

> Optional weekend (30–45 mins): quick smoke on staging if they deploy frequently.

---

## 10-week timeline (day-by-day focus, Mon–Fri)

### Week 1 — QA setup + requirements lock

- **Day 1:** Read PRD end-to-end; extract user stories + acceptance criteria into **RTM**
- **Day 2:** Define test strategy (P0 smoke, regression, release criteria); create test data plan (drivers/passengers/trips)
- **Day 3:** Set up environments (dev/staging), logging access, Stripe test mode, email test mode
- **Day 4:** Draft core test cases for **Auth + Profiles**; review with PM/dev for gaps
- **Day 5:** Build initial **Smoke Suite v1** checklist + bug template + QA report template

### Week 2 — Auth + Profiles deep test + start automation

- **Day 6:** Test signup/login/email verification/reset (happy + negative)
- **Day 7:** Test role gating (Driver/Passenger/Both), public vs private fields
- **Day 8:** Security sanity: authz on profile endpoints, session handling, rate-limit expectations
- **Day 9:** Automate: Auth smoke (signup/login/logout) + API checks
- **Day 10:** Mini-regression + stabilize "ready for trips module" baseline

### Week 3 — Driver Trip Creation

- **Day 11:** Test trip create validations (date not past, seats 1–6, notes limits, vehicle fields)
- **Day 12:** Test edit trip rules (reduce seats vs existing bookings), cancel/delete behavior (as implemented)
- **Day 13:** Cross-device responsive pass on trip create page (mobile web + desktop)
- **Day 14:** Automate: create trip flow + API contract (create/list)
- **Day 15:** Regression sweep: auth + profiles + trip creation

### Week 4 — Search + Trip Detail

- **Day 16:** Test search by origin/destination/date; sort by departure time
- **Day 17:** Test filters (passengers <= seats remaining), empty states, pagination (if any)
- **Day 18:** Test trip detail page: all fields displayed; CTA state; no booking if sold out
- **Day 19:** Automate: search → detail → validate card fields
- **Day 20:** Regression: Trips + Search + Auth

### Week 5 — Booking (no payments yet or stub payments)

- **Day 21:** Booking request flow; seat selection rules; booking states (pending/confirmed model)
- **Day 22:** Seat inventory logic (decrement/restoration on cancel/reject)
- **Day 23:** Concurrency test: last seat, two users booking; ensure no oversell
- **Day 24:** Automate: booking flow + inventory check
- **Day 25:** Regression: Search/Detail/Booking + bug-bash with team

### Week 6 — Payments (Stripe) + webhooks

- **Day 26:** Payment success path: booking updates + receipts; prevent duplicate charges
- **Day 27:** Payment failure paths: declined/insufficient funds/abandon; ensure no seat decrement if unpaid
- **Day 28:** Webhook idempotency: replay events; verify no double booking/transaction
- **Day 29:** Fee + payout logic sanity (hold until completion; payout after completion)
- **Day 30:** Automate: pay flow in test mode + webhook validations (where feasible)

### Week 7 — Messaging (Chat)

- **Day 31:** Verify chat only opens after booking request; only matched users can access
- **Day 32:** Message ordering, timestamps, refresh persistence; basic abuse cases (empty msg, long msg)
- **Day 33:** Chat behavior after booking cancelled/rejected (lock/read-only/allow) — document final decision
- **Day 34:** Automate: booking → open chat → send message
- **Day 35:** Regression: booking+payment+chat (critical path week)

### Week 8 — Ratings/Reviews + Admin

- **Day 36:** Rating gated by completion only; cannot rate early; 1–5 validation
- **Day 37:** Rating aggregation (average + count updates) across profiles/trip cards
- **Day 38:** Admin access control + view users/trips/bookings
- **Day 39:** Admin dispute flow: ensure state/payment consistency + audit notes (as implemented)
- **Day 40:** Full regression run #1 (end-to-end) + create release bugs list

### Week 9 — Hardening + UAT readiness

- **Day 41:** Performance sanity: search load time, booking latency, webhook processing stability
- **Day 42:** Security hardening checks: authz, IDOR attempts on booking/chat/admin routes, rate-limit notes
- **Day 43:** Cross-browser + mobile web compatibility sweep (Chrome/Edge/Firefox + iOS Safari)
- **Day 44:** UAT support: run through scripted UAT with stakeholders; capture feedback/defects
- **Day 45:** Full regression run #2 + finalize "Go/No-Go" checklist

### Week 10 — Release candidate → deployment → post-deploy verification

- **Day 46:** Freeze scope; test Release Candidate build (P0 + key P1)
- **Day 47:** Fix verification + re-test loop; confirm "0 blocker/critical"
- **Day 48:** Pre-deploy checklist: configs, Stripe live readiness (if going live), monitoring/logging, rollback plan
- **Day 49:** Deploy day: prod smoke immediately after deploy (auth, search, booking, payment sanity if live, chat)
- **Day 50:** Post-deploy monitoring + bug triage + hotfix verification; write QA release report

---

# RTM

> **Legend**  
> **Priority:** P0 (release-blocker), P1 (important), P2 (nice-to-have)  
> **Type:** M (Manual), A (Automated candidate)

## RTM v1 (PRD → Requirements → Test Cases)

| Req ID | PRD Source | Requirement | Pri | Test Cases (from your suite) | Type |
| --- | --- | --- | --- | --- | --- |
| G-01 | Product Vision/Goals | MVP enables drivers to post trips; travellers to discover/book/pay; includes trust + messaging | P0 | TRIP-001, SEARCH-001, BOOK-003, PAY-001, CHAT-001, RATE-001 | M/A |
| G-02 | Non-goals | No native apps, no real-time GPS, no dynamic pricing, no multi-stop routing | P2 | (Scope check) Release checklist | M |
| CJ-D-01 | Driver Journey | Driver journey supports: signup → profile → post trip → receive booking → accept/reject → complete trip → payout | P0 | AUTH-001, PROF-001, TRIP-001, BOOK-007/008, BOOK-011, PAY-008 | M/A |
| CJ-P-01 | Passenger Journey | Passenger journey supports: signup → search → view detail → request/book → pay → chat → complete → review | P0 | AUTH-001, SEARCH-001, DETAIL-001, BOOK-001/003, PAY-001, CHAT-001, RATE-001 | M/A |

### 4.1 Authentication & Accounts

| Req ID | PRD Source | Requirement | Pri | Test Cases | Type |
| --- | --- | --- | --- | --- | --- |
| AUTH-R-01 | 4.1 | Email + password signup/login supported | P0 | AUTH-001, AUTH-002, AUTH-008, AUTH-011 | M/A |
| AUTH-R-02 | 4.1 | Optional Google OAuth supported | P1 | AUTH-010 | M |
| AUTH-R-03 | 4.1 | Email verification required/available | P0 | AUTH-002, AUTH-003, AUTH-004 | M/A |
| AUTH-R-04 | 4.1 | Password reset supported | P0 | AUTH-005, AUTH-006, AUTH-007 | M/A |
| AUTH-R-05 | 4.1 Data Fields | User data fields exist (ID, names, email, pw hash, phone optional, photo optional, role) | P1 | PROF-001, PROF-005, AUTH-001 | M |

### 4.2 User Profiles

| Req ID | PRD Source | Requirement | Pri | Test Cases | Type |
| --- | --- | --- | --- | --- | --- |
| PROF-R-01 | 4.2 | Public profile shows: photo, first name, age range optional, rating+count, bio (≤300), trips completed | P0 | PROF-002, PROF-004, PROF-006 | M |
| PROF-R-02 | 4.2 | Private profile fields not visible to others: email, phone, payment details | P0 | PROF-003 | M |
| PROF-R-03 | 4.2 | Role exists: Driver / Passenger / Both and gates features | P0 | PROF-001 | M |

### 4.3 Trip Creation (Driver)

| Req ID | PRD Source | Requirement | Pri | Test Cases | Type |
| --- | --- | --- | --- | --- | --- |
| TRIP-R-01 | 4.3 Inputs | Trip requires origin/destination (autocomplete), date/time, seats (1–6), price/seat, vehicle make/model/colour, luggage yes/no, notes (≤500) | P0 | TRIP-001, TRIP-003, TRIP-004, TRIP-005, TRIP-006, TRIP-007, TRIP-008 | M/A |
| TRIP-R-02 | 4.3 Rules | Cannot create trip in the past | P0 | TRIP-003 | M/A |
| TRIP-R-03 | 4.3 Rules | Cannot reduce seats below confirmed bookings | P0 | TRIP-010, TRIP-011 | M |
| TRIP-R-04 | 4.3 Rules | Price × seats computed as total potential earnings | P2 | TRIP-009 | M |

### 4.4 Trip Discovery (Passenger)

| Req ID | PRD Source | Requirement | Pri | Test Cases | Type |
| --- | --- | --- | --- | --- | --- |
| SEARCH-R-01 | 4.4 Filters | Search filters: origin, destination, date, number of passengers | P0 | SEARCH-001, SEARCH-002 | M/A |
| SEARCH-R-02 | 4.4 Results Card | Trip card shows: driver name+rating, departure time, price/seat, seats remaining, vehicle summary | P1 | SEARCH-005 | M |
| SEARCH-R-03 | US-3 AC | Results sorted by departure time | P0 | SEARCH-003 | M/A |
| SEARCH-R-04 | US-3 AC | Search returns relevant trips | P0 | SEARCH-001 | M/A |

### 4.5 Trip Detail Page

| Req ID | PRD Source | Requirement | Pri | Test Cases | Type |
| --- | --- | --- | --- | --- | --- |
| DETAIL-R-01 | 4.5 | Trip detail shows route, date/time, price/seat, seats remaining, driver snippet, vehicle details, luggage policy, notes | P0 | DETAIL-001 | M/A |
| DETAIL-R-02 | 4.5 | Primary CTA displayed ("Request Seat" / "Book Seat") | P1 | DETAIL-002 | M |
| DETAIL-R-03 | 4.5 + Booking | Booking blocked if seats remaining = 0 | P0 | DETAIL-003 | M |

### 4.6 Booking Flow + States

| Req ID | PRD Source | Requirement | Pri | Test Cases | Type |
| --- | --- | --- | --- | --- | --- |
| BOOK-R-01 | 4.6 Steps | Flow: select seats → confirm request → pay → status updates (Pending/Confirmed) | P0 | BOOK-001, BOOK-002, BOOK-003 | M/A |
| BOOK-R-02 | 4.6 States | Booking states exist: Pending, Confirmed, Cancelled (driver/passenger), Completed | P0 | BOOK-010, BOOK-011, BOOK-008, BOOK-009 | M |
| BOOK-R-03 | US-2 AC | Seats decrement after booking | P0 | BOOK-004 | M/A |
| BOOK-R-04 | Inventory integrity | Prevent oversell under concurrency | P0 | BOOK-005 | M |

### 4.7 Payments (Stripe)

| Req ID | PRD Source | Requirement | Pri | Test Cases | Type |
| --- | --- | --- | --- | --- | --- |
| PAY-R-01 | 4.7 | Passenger pays upfront | P0 | PAY-001, PAY-002, PAY-003 | M/A |
| PAY-R-02 | 4.7 | Platform holds funds "escrow-like"; payout after completion | P0 | PAY-007, PAY-008 | M |
| PAY-R-03 | Assumptions | Stripe Payment Intents + Connect used | P0 | PAY-001, PAY-005 | M |
| PAY-R-04 | Assumptions | Platform fee fixed % (e.g., 10–15%) | P1 | PAY-006 | M |
| PAY-R-05 | Payment Data | Store booking ID, gross, fee, net payout, payment status | P1 | PAY-006, PAY-005 | M |
| PAY-R-06 | Reliability | Idempotent webhooks / no duplicates | P0 | PAY-004, PAY-005 | M/A |

### 4.8 Messaging

| Req ID | PRD Source | Requirement | Pri | Test Cases | Type |
| --- | --- | --- | --- | --- | --- |
| CHAT-R-01 | 4.8 | 1:1 chat between driver and passenger; text only | P1 | CHAT-004, CHAT-006 | M |
| CHAT-R-02 | 4.8 Rules | Chat opens only after booking request | P0 | CHAT-001, CHAT-002 | M/A |
| CHAT-R-03 | 4.8 Rules | Messages stored with timestamps | P1 | CHAT-004 | M |
| CHAT-R-04 | US-5 AC | Messages delivered in order | P0 | CHAT-005 | M/A |
| CHAT-R-05 | Access control | Only matched parties can access chat | P0 | CHAT-003 | M |

### 4.9 Ratings & Reviews

| Req ID | PRD Source | Requirement | Pri | Test Cases | Type |
| --- | --- | --- | --- | --- | --- |
| RATE-R-01 | 4.9 | Passenger can rate driver (1–5) after trip completion | P0 | RATE-001, RATE-002, RATE-003 | M/A |
| RATE-R-02 | 4.9 | Optional comment supported | P2 | RATE-004 | M |
| RATE-R-03 | 4.9 Optional | Driver can rate passenger (optional MVP) | P2 | RATE-007 | M |
| RATE-R-04 | Aggregation | Average rating displayed/updates correctly on profile | P0 | RATE-005, RATE-006, PROF-006 | M |

### 4.10 Admin (Basic MVP)

| Req ID | PRD Source | Requirement | Pri | Test Cases | Type |
| --- | --- | --- | --- | --- | --- |
| ADMIN-R-01 | 4.10 | Admin can view users | P1 | ADMIN-002 | M |
| ADMIN-R-02 | 4.10 | Admin can view trips | P1 | ADMIN-003 | M |
| ADMIN-R-03 | 4.10 | Admin can view bookings | P1 | ADMIN-004 | M |
| ADMIN-R-04 | 4.10 | Admin can manually resolve disputes | P0 | ADMIN-005, ADMIN-006 | M |
| ADMIN-R-05 | Admin security | Admin routes are protected (non-admin blocked) | P0 | ADMIN-001 | M |

---

## User Stories → Acceptance Criteria RTM (Explicit)

(These mirror the PRD bullets so you can track "done/not done" cleanly.)

| AC ID | Story | Acceptance Criteria (PRD) | Pri | Test Cases |
| --- | --- | --- | --- | --- |
| US1-AC1 | US-1 | Valid email/password creates account | P0 | AUTH-001 |
| US1-AC2 | US-1 | Verification email sent | P0 | AUTH-001 |
| US1-AC3 | US-1 | User can log in after verification | P0 | AUTH-002, AUTH-003 |
| US2-AC1 | US-2 | Required trip fields validated | P0 | TRIP-001, TRIP-004, TRIP-006, TRIP-007 |
| US2-AC2 | US-2 | Trip appears in search results | P0 | TRIP-001, SEARCH-001 |
| US2-AC3 | US-2 | Seats decrement after booking | P0 | BOOK-004 |
| US3-AC1 | US-3 | Search returns relevant trips | P0 | SEARCH-001 |
| US3-AC2 | US-3 | Results sorted by departure time | P0 | SEARCH-003 |
| US4-AC1 | US-4 | Payment processed successfully | P0 | PAY-001, PAY-002 |
| US4-AC2 | US-4 | Booking status updated | P0 | BOOK-003 |
| US4-AC3 | US-4 | Driver notified | P1 | BOOK-006 |
| US5-AC1 | US-5 | Chat opens after booking | P0 | CHAT-001 |
| US5-AC2 | US-5 | Messages delivered in order | P0 | CHAT-005 |
| US6-AC1 | US-6 | Rating allowed only after completion | P0 | RATE-001, RATE-002 |
| US6-AC2 | US-6 | Average rating updates correctly | P0 | RATE-005, PROF-006 |
