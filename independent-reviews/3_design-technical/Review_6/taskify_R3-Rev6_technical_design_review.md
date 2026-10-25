# Taskify — Independent R3-Rev6 Technical Design Review

**Date:** 2026-10-24 (Australia/Melbourne)  
**Reviewer:** Independent R3 Technical Design Reviewer  
**Independent verdict:** **NOT READY — TD-008 remains a Medium design blocker**  
**Human R3 gate:** **OPEN / not approved**. No new technical risk has been accepted. No Tasks, Analyze, Build, runtime tests or deployment authorised by this assessment.

## 1. Executive assessment

The human-approved **R2-Rev5** functional clarification removes the *ordinary* workflow in which account B logs in while account A remains signed in. Authenticated new tabs and authenticated login GET/POST retain A's existing session and view; switching accounts requires a successful logout. The submitted R3 technical candidate has made these rules operative in the UI/HTTP contracts and server login-family transitions.

**TD-001 remains RESOLVED at written-design level:** server-owned `AuthorityFamily` records, common row locking and generation fencing no longer require later Cookie headers to reproduce earlier transition cookies. The submitted navigation guard further rules out live-family account replacement. This conclusion does not claim a browser/DB implementation has been tested.

**TD-008 remains OPEN / Medium / blocking:** `ISSUE` can still overwrite the **only usable authority cookie** with a *PROVISIONAL* handle before `ACTIVATE`. An already-issued but delayed `Set-Cookie` can arrive after the family has cancelled that attempt, making a live A session inaccessible. A tab that has just validated A then cannot use its normal logout endpoint to revoke A across tabs. The same single-cookie race also exists between two login attempts submitted while anonymous, even though new login attempts from an *observed* authenticated state are now forbidden. These failures are not justified by the approved R2-Rev5 behaviour or an existing accepted risk.

**Verdict: NOT READY.** No new human product decision is needed merely to request a technical correction. If Codex proposes accepting forced sign-out or inability to complete an authoritative shared logout as normal behaviour, that must instead return for the appropriate explicit R2/human decision.

## 2. Controlled evidence and review boundary

| Input | Independent verification |
|---|---|
| Current attachment | `Archive(7).zip`, SHA-256 `9418ba969a06bab3922ba2ca4aa85219a1545e94ceb8f15edd7e537dbcdf1075`. |
| Scope | `specs/001-personal-task-manager/spec.md`, SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`, matches approved R1-Rev4 Scope v1.1/AM-001. |
| Functional authority | Attached `independent-reviews/2_design-functional/Review_5/Taskify_R2-Rev5_Targeted_Functional_Design_Review.md` §7 records Barry's **2026-10-24 human APPROVED** decision, including **R2-NAV-001**. Its approved change-manifest SHA-256 `78fa6c6d0287df35e8a0928fa6baae576693c48bc3947da786651935b2a82b1d` matches the supplied manifest bytes. |
| Current author change | Recomputed **9/9** `review-followups/r3-navigation-clarification-changed-files.sha256` entry hashes; all match. `technical-design.md` SHA-256 `3d572870bed7977a0f21f1fcdbf5f5a9d47f3ef61c795c8044d8747abe663163`; `auth-transition-protocol.md` `be7517e8a9e1b1fef8777617ae083b4c406824b7052bb591c713af2dfab67a1c`. |
| Historical R3 | The attached R3-Rev4 report matches SHA-256 `891511a1af05183110c746a3cfa7c7fb7091024a42abb590259783b691024874`. The previous independent R3-Rev5 reviewer file, available separately in this review workspace, matches SHA-256 `2cbebad17c0d489a33cc59a65f8a81e92730286f5b5c8174c65372e848c9cbbf`. It is not inside the current ZIP; its disposition is preserved rather than inferred from author assertions. |

Relative artifact references below are under `specs/001-personal-task-manager/`. Independently inspected the current technical design, authority protocol, HTTP/UI contracts, data model, plan, research, quickstart, correction map and predecessor independent reports. No application, real-browser, PostgreSQL, load, security-attack, restore or backup test was executed. NAV-01–08, R5-V01a–c and R5-V08a–d are **planned**, not passing, evidence.

## 3. Stable finding register

| ID | Classification / inherited severity | Rev5 → Rev6 disposition | Reason / downstream owner |
|---|---|---|---|
| **TD-001** | Correctness / Design Gap; **High** | **RESOLVED (design) → RESOLVED (design)** | Family-wide recorded-transition claim/activation fence and live-login rejection protect the cross-account stale-reveal ordering in writing. Codex tests actual DB/browser events downstream. |
| **TD-002** | Design Gap; Medium | **RESOLVED → RESOLVED** | Approved AM-001 task-first bootstrap and silent full view fallback remain. |
| **TD-003** | Architectural Risk; Medium | **RESOLVED (design) → RESOLVED (design)** | Workload/pool/timeout and receipt-growth budgets retained; load measurement due R5. |
| **TD-004** | Design Gap / Assumption; Medium | **RESOLVED (design) → RESOLVED (design)** | Versioned keyed HMAC receipts and historical-key replay retained; secret/key tests due. |
| **TD-005** | Design Gap; Medium | **RESOLVED (design) → RESOLVED (design)** | Trusted ingress, shared throttling and fail-closed outage retained; deployment tests due. |
| **TD-006** | Architectural Risk; Medium | **OPEN, nonblocking R5 dependency → same** | Barry accountable for runbook, policy and observed restore/operations evidence; Codex prepares. No sign-off or drill claimed. |
| **TD-007** | Architectural Risk; Low | **RESOLVED (design) → RESOLVED (design)** | Historical format/key compatibility and R4/Build fixtures retained; runtime upgrade/rollback evidence due. |
| **TD-008** | Correctness / Design Gap; **Medium** | **OPEN, blocker → OPEN, blocker** | Valid session can lose its sole authority cookie to a delayed or in-progress provisional ISSUE, defeating normal shared logout and continuity. Codex remedy; escalate any product compromise to Barry/R2. |
| **TD-009** | Architectural Risk; Medium | **RESOLVED (design) → RESOLVED (design)** | Bounded isolated optional view read retains silent defaults and independent task-load protections; PostgreSQL fault tests due. |

Prior accepted **FD-003** (memory-only interrupted operation identity) and **FD-004** (no credential recovery) remain accepted only for their documented consequences. Neither authorises self-induced live-session credential loss or an ineffective shared logout.

### TD-001 — Resolved in design; no regression established

**Confidence:** High for the server ordering expressed; runtime validation outstanding. **Trace:** FR-010 / AC-010 and FR-009/014 interactions. **Evidence:** `technical-design.md:17–52`, `auth-transition-protocol.md:8–69`, `contracts/http.md:38–62,87–101`, `contracts/ui.md:127–150`.

An authenticated login-page GET/POST redirects without creating a replacement transition. A login ACTIVATE is prohibited from replacing a live family. A valid *same-family maintenance rotation*, if already dispatched, is registered by family FK before cookie issuance. Claim and activation lock the same family and compare generation: claim-first cancels activation; rotation-first retires the old handle but retains the same account. Missing current proof fails closed. No evidence here reopens TD-001's resolved *cross-account* concurrency invariant. R5-V01a–c must later substantiate actual cookie omission, lock orders, DOM/accessibility paint, discarded notifications and failed later reads.

### TD-008 — OPEN: provisional-cookie overwrite can orphan an active account

**Classification:** Correctness Defect / Design Gap. **Severity:** Medium. **Confidence:** High in the permitted event ordering; incidence in production unknown. **Owner:** Codex for correction, Barry/R2 only if observable behaviour or risk acceptance changes. **Trace:** FR-010, FR-012, AC-005, R2-NAV-001 and the explicitly approved cross-tab logout outcome.

**Direct evidence:** `auth-transition-protocol.md:73–82,100–129,139–152`; `contracts/http.md:89–124`; `contracts/ui.md:140–169`; `data-model.md:115–140`. ISSUE changes the browser's one `__Host-taskify-authority` cookie to a not-yet-authenticated PROVISIONAL value. CLAIM and LOGOUT require a currently pinned ACTIVE handle. ACTIVATE and LOGOUT cannot write/restore an authority cookie. The design expressly allows a late/cancelled ISSUE response to overwrite a newer usable cookie and require sign-in.

**Counterexample 1 — ordinary same-account maintenance rotation (not forbidden B login):**

1. Barry's account A is active in tabs 1 and 2. A designated same-account rotation registers under family A. ISSUE commits a provisional handle P and produces its `Set-Cookie`; delay this HTTP response before browser receipt.
2. Tab 1 regains focus and sends CLAIM using its still-present live A cookie. Under the family lock, CLAIM cancels the pending rotation, validates A and permits a matching A task snapshot to be shown.
3. The old ISSUE response now reaches the browser and **overwrites the sole cookie with cancelled P**. A is still server-side ACTIVE, but subsequent requests carry P and fail 401.
4. Barry presses Log out in tab 1. `POST /accounts/logout/` requires A's current handle and cannot revoke family A when presented with cancelled P. It cannot satisfy the approved **logout from any tab revokes the shared login** guarantee. If sign-in is requested, the live A server session remains orphaned until expiry/operator handling.

This counterexample remains possible after R2-Rev5 because rotation, unlike a new credential login, is explicitly permitted under a live family. A claim's cancellation cannot retract an ISSUE response already in transit. `Cache-Control: no-store` cannot solve out-of-order live responses; the HTTP cookie standard warns of races among concurrent Set-Cookie responses. The same defect can arise if a tab focuses between a normal rotation ISSUE and ACTIVATE or the latter fails, not just with a cancelled attempt.

**Counterexample 2 — previously submitted anonymous logins:** Two tabs load anonymous login forms and dispatch attempts into distinct fresh families before either becomes authenticated. A ACTIVATE completes and A becomes usable. A delayed B ISSUE response, generated earlier, then overwrites A's one cookie with provisional B. Because activation is family-local, the server cannot prevent this cross-family browser overwrite. The design prevents B from silently revealing A data, which is valuable, but **does not preserve A's usable session or its subsequent logout path**. Authenticated-login redirects prevent *newly submitted* logins, not requests already in flight. `http.md:56–62,103–124` and `auth-transition-protocol.md:139–152` acknowledge this residual response race.

**Minimum correction:** Define a safe session/cookie issuance and reconciliation path for these orders that preserves an authoritative logout of the active shared family and a coherent live-session experience, without permitting old-account fallback or another-account data exposure. If the intended correction is to accept an unexpected forced re-login or inability to revoke a seemingly authenticated session, explicitly identify the observable residual and obtain targeted R2/human approval; **do not classify it as already accepted FD-004 credential recovery or arbitrary external cookie eviction**. Add exact deterministic rotation-claim-ISSUE and two-anonymous-family schedules, checking server revocation, browser cookie, user-visible account/view, DOM/accessibility, CSRF and later task mutations. Do not mandate any particular redesign.

**Related TD-008 integration check (not a separate finding):** Django rotates CSRF tokens on login. A pre-login page in another tab can retain a now-stale form/header token. `ui.md:21–28,127–132` requires safe revalidation; specify how mandatory `POST /api/session/claim/` obtains the current CSRF token or safely reboots after 403. `http.md:45–46` allows genuine 403, but it must not be misreported as successful shared logout. A completed same-tab login and simultaneous pre-login tab fixture should be included. [Django CSRF documentation](https://docs.djangoproject.com/en/5.2/ref/csrf/).

## 4. Regression and feasibility assessment

| Dimension | Independent assessment and outstanding proof |
|---|---|
| Functional fit | R2-Rev5 new-tab/redirect and view preservation now explicit (`ui.md:15–28`, `http.md:38–74`). Fresh sign-in after logout/expiry resets defaults. **TD-008 continuity/logout remains inconsistent** in exceptional yet permitted orders. |
| Security | High-entropy host-only HttpOnly handle, family pinning, fail-closed 401/503 and CSRF/owner guards are plausible; no cross-user task reveal established from the design. Race-driven loss of usable authority and ineffective logout remain material. |
| Concurrency/data integrity | Family → transition/handle → Session → view and separate account-state → receipt → task lock orders are specified; no new deadlock/receipt resurrection counterexample substantiated. Actual PostgreSQL and middleware commit/rollback tests are required downstream. |
| Failure handling | Optional view failures remain silent and isolated; mandatory task load failures keep FD-001 Retry/support and blocked writes. Auth failure must not be passed off as an optional-view default. |
| Scale/ops | One cookie and bounded transition TTL, cleanup and pools improve the prior design, but cannot establish HTTP delivery order. New authority tables/cleanup require load measurement; operational restore must invalidate registry, sessions and views together. |
| Review controls | Technical source headers still describe R2-NAV-001 as pending, although the independent R2-Rev5 report now records human approval. This is a nonblocking author-metadata reconciliation item; do not rewrite historical reviews. |

## 5. Exit gate and instructions for correction

| Check | R3-Rev6 result |
|---|---|
| R1-Rev4 Scope and R2-Rev5 human approvals verified | **PASS** |
| Current nine submitted hashes and unchanged Scope identity independently verified | **PASS** |
| TD-001 cross-account claim/activation fence | **PASS at design level; runtime proof outstanding** |
| TD-008 usable authority through legitimate rotation, late ISSUE and anonymous overlap; reliable shared logout | **FAIL — OPEN blocker** |
| TD-002/003/004/005/007/009 prior design-level resolutions | **RETAINED; downstream evidence outstanding** |
| TD-006 accountable Operations/R5 handoff | **PASS for R3 ownership; OPEN for R5 evidence** |
| Runtime/browser/DB/security/capacity/restore tests | **NOT RUN** |
| New human risk acceptance / R3 gate approval | **NONE** |

**Formal verdict: NOT READY.** Codex should correct only the targeted TD-008 lifecycle/credential-loss/logout mechanism and CSRF handoff, retain approved FR-010/FR-012/R2-NAV-001 behaviour, supply changed-file hashes and deterministic schedules, and return for **R3-Rev7** independent review. Escalate a genuinely required product-policy compromise; do not infer approval. Preserve all earlier R1/R2/R3 documents. No Tasks, Analyze or Build.

## 6. Review history

- **R3-Rev1:** NOT_READY; TD-001–007 open.
- **R3-Rev2:** NOT_READY; TD-002 resolved through approved AM-001; TD-008/009 added.
- **R3-Rev3:** NOT_READY; TD-003/004/005/007/009 design-resolved, TD-006 nonblocking R5 ownership/evidence, TD-001/008 open.
- **R3-Rev4:** NOT_READY; unsupported complete-cookie assumption left TD-001/008 open.
- **R3-Rev5:** NOT_READY; TD-001 resolved in writing using server-family claim/generation, TD-008 remained open over late PROVISIONAL overwrite and failed shared logout.
- **R2-Rev5:** Independently READY_FOR_HUMAN_APPROVAL, then Barry human APPROVED 2026-10-24 including R2-NAV-001. This replaces only the approved direct same-user re-login/reset behaviour, not technical correctness.
- **R3-Rev6 (this review):** Verified the nine current artifact hashes, Scope and new human-approved R2 baseline. Navigation guard simplifies ordinary account switching; legitimate rotation and previously dispatched anonymous requests still reproduce TD-008 credential-loss/logout race. TD-008 OPEN; R3 gate OPEN.
