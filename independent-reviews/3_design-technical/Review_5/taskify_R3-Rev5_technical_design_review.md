# Taskify — Independent R3-Rev5 Technical Design Review

**Review date:** 2026-10-24 (Australia/Melbourne)  
**Reviewer:** Independent Technical Design Reviewer (R3)  
**Independent verdict:** **NOT READY — one narrowed technical/behavioural blocker**  
**Human R3 gate:** **OPEN; no approval or new risk acceptance inferred**  
**Review method:** Static design review and exact-byte integrity checks. No browser, PostgreSQL, application, restore, load or production tests were run.

## 1. Independent conclusion

The R3-Rev5 author proposal **corrects the material cross-account privacy race** identified in R3-Rev4. It expressly withdraws the unsound premise that an in-flight request's cookies must be present in later requests. A server-owned `AuthorityFamily` now registers a login transition before issuance, and claim/activation serialize on that family's lock and monotonic generation. A claim with old account A's handle can therefore discover and cancel an already-dispatched login B even when B's cookie is missing; if B activates first, A is retired and A's claim fails closed. **TD-001 is Resolved at written-design level**, subject to actual implementation/browser/DB race testing.

The replacement **single pinned authority cookie** also fixes the earlier accumulation, high-water-floor and selective-cookie-fallback problems. However, the staged flow deliberately **replaces the currently usable account cookie with an unauthenticated PROVISIONAL cookie at ISSUE, before login is committed**. A later, cancelled ISSUE response can overwrite a valid login *after another tab has revalidated it*. That can unexpectedly force sign-in and, more importantly, prevent a user from completing a logout from an otherwise authenticated task tab: the logout endpoint no longer receives the currently active handle needed to revoke that server session. The design acknowledges forced sign-in on interrupted logins but does not reconcile this with the approved usable-login and shared-logout behaviour or obtain a human policy decision. **TD-008 remains Open / Medium and blocks R3 approval** until the logout/credential-loss outcome is corrected or explicitly escalated as a product-policy trade-off. This is a **new, narrower concern within TD-008**, not a reopening of TD-001's cross-account privacy proof.

The other outcomes are unchanged: **TD-002/003/004/005/007/009 Resolved (design only)**, **TD-006 Open—nonblocking R5 operations evidence**, with Barry accountable and Codex preparing evidence. Previous FD-003/FD-004 human-accepted risks retain only their previously recorded bounds. No new risk has been accepted.

## 2. Controlled inputs and integrity

| Input | Independently verified result |
|---|---|
| Submission | `Archive(6).zip` SHA-256 `74cd60b6a7b6c7221d26346f7f5c7d3812405bc20b3336805108fac501f36393`. |
| Scope baseline | `specs/001-personal-task-manager/spec.md` SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca` — exact approved Scope v1.1/AM-001 identity. R1-Rev4 human approval remains historical. |
| Functional Design | R2-Rev4 independent report §8 documents Barry's explicit human approval of the nine-file historical design baseline. Later technical edits do not inherit it. The two approved checklists and AM-001 handoff individually retain their known SHA-256 identities. |
| Review history | Supplied R3-Rev4 independent review has SHA-256 `891511a1af05183110c746a3cfa7c7fb7091024a42abb590259783b691024874`, matching the earlier reviewer-owned report. R3-Rev1–Rev3 remain supplied as history. |
| Author correction manifest | **All nine** `review-followups/r3-rev5-changed-files.sha256` entries independently match supplied file bytes. `technical-design.md` SHA-256 `5a5f2b8671e14e2b74714a4da4569ec579a300537b788b9f05221da0da5818be`; `auth-transition-protocol.md` SHA-256 `14a58ef521f0e1707fa232b72cafe66ab8198deb202cf21b5171090de8564cdb`. |
| Previously resolved work | Independently compared `technical-design.md` §§3–8 with the Rev4 submission: **byte-identical**. The existing capacity, receipt privacy/compatibility, throttling, optional-view and operations-design mechanisms are inherited at their recorded design-only dispositions. |

Locations below are relative to `specs/001-personal-task-manager/`. The author evidence map is a handoff claim, not independent verification. Existing independent reports and canonical artifacts were not edited.

## 3. Finding ledger: retained IDs and new revision

| Finding | R3-Rev4 → **R3-Rev5** | Basis / downstream boundary |
|---|---|---|
| **TD-001 — Cross-tab private data** | Open, High → **RESOLVED (design)** | Server-family registration, atomic claim/activation lock and captured generation remove reliance on cookie completeness. Real DOM/browser/DB tests due. |
| **TD-002 — Preference bootstrap** | Resolved → **RESOLVED** | AM-001 task-first bootstrap, silent full defaults. |
| **TD-003 — Capacity** | Resolved (design) → **RESOLVED (design)** | Budgets retained; new family/claim load must be measured later. |
| **TD-004 — Receipt privacy** | Resolved (design) → **RESOLVED (design)** | HMAC/key custody and conservative deletion bounds retained. |
| **TD-005 — Auth throttling** | Resolved (design) → **RESOLVED (design)** | Trusted ingress, shared DB limiter and fail-closed outage behaviour retained. |
| **TD-006 — Recovery readiness** | Open, nonblocking → **OPEN, nonblocking R5 evidence** | Barry accountable/Codex prepares; restore handoff now includes authority registry invalidation. Drill/policy sign-off not executed. |
| **TD-007 — Upgrade compatibility** | Resolved (design) → **RESOLVED (design)** | R4 fixture handoff and later Build/Test upgrade checks still due. |
| **TD-008 — Login/session lifecycle** | Open, Medium → **OPEN, Medium blocker (narrowed)** | One-cookie/ledger fixes old fallback, but late provisional cookie overwrite can orphan a live login and block its shared logout. |
| **TD-009 — Optional view containment** | Resolved (design) → **RESOLVED (design)** | Isolated 150 ms fallback/primary task isolation retained; PostgreSQL race tests due. |

### TD-001 — **Resolved at design level: family registration and atomic generation fence**

**Classification:** Previously Correctness/Design Gap. **Inherited severity:** High. **Confidence in written correction:** High. **Disposition:** **Resolved (design)**. **Owner:** Codex, with downstream browser/DB test evidence. **Trace:** FR-010/AC-010; FR-009/014 interaction.

**Evidence:** `auth-transition-protocol.md:7–46`; `technical-design.md:16–36`; `contracts/http.md:53–66`; `data-model.md:81–112`; targeted scenarios `quickstart.md:133–148`.

The key missing Rev4 invariant is now explicit: every possible B ACTIVATE belongs to a server-owned family **before ISSUE**. A successful A-handle claim and B activation lock the *same family* and compare the captured generation. Claim first increments generation and cancels B, so its already-dispatched ACTIVATE fails. Activation first retires A, so A-only claim cannot return A tasks. A no-cookie/provisional-cookie claim is 401/concealed. This remains valid if the later request omits B proof entirely, provided it still authenticates using current A; without any current handle there is no private reveal. Lost broadcasts cannot override the committed authority. The exact post-dispatch missing-cookie race and both lock orders are now specified for later tests.

**Residual, not a blocker for R3:** No implementation or timing verification was performed. Test `R5-V01a–c` against genuine HttpOnly cookie scheduling, serializable family transactions, fail/rollback and DOM/accessibility traces. A newly discovered counterexample would reopen TD-001; none is established from the submitted written contract.

### TD-008 — **Open: a cancelled ISSUE can defeat a currently valid login and shared logout**

**Classification:** Correctness Defect / Design Gap (session lifecycle and observable logout). **Severity:** Medium. **Confidence:** High in the allowed response ordering; user impact depends on timing. **Disposition:** **Open, R3 blocker.** **Owner:** Codex (technical remedy); escalate to R2/Barry if proposing a weaker successful-logout/session-continuity policy. **Trace:** FR-010/012, AC-005 and Barry's explicit same-browser cross-tab logout direction.

**Substantial corrections accepted in writing:** `auth-transition-protocol.md:48–75,82–103,113–170` now specifies exactly one host-only 256-bit opaque auth handle, pinned ACTIVE family authority, no old-cookie fallback, one-time issuance, same-user fresh login, maintenance-only view carryover, family-wide revocation and bounded 120-second attempts/60-second cleanup. Missing/retired/provisional/unknown credentials fail closed; these rules resolve the former cookie-set selection and indefinite high-water-retention subproblem. The family → transition/handle → Session → view lock order is coherent in principle, with short transactions and runtime contention testing due.

**Remaining counterexample requiring a decision or correction:**

1. **A is valid and displayed in tab 1.** Tab 2 begins a valid new login B. Its PREPARE registers B under A's family; its ISSUE commits B's **PROVISIONAL** handle. Hold the ISSUE HTTP response, including its `Set-Cookie` header, before browser delivery.
2. The user returns to tab 1. Its request still carries A's ACTIVE cookie. `POST /api/session/claim/` legitimately locks the family, advances generation, **cancels B**, validates A and displays A tasks. This is the intended TD-001 privacy-safe outcome.
3. The **already-generated, now-late ISSUE response** then reaches the browser and sets the single `__Host-taskify-authority` cookie to **cancelled PROVISIONAL B**. This replaces usable A in the browser, even though no login B succeeded and the server still considers A's session ACTIVE. `Cache-Control: no-store` prevents cache reuse; it does not cancel an already-dispatched response (RFC 9111 §5.2.2.5).
4. The user now presses **Log out** on tab 1's already-validated A screen. The request presents provisional/cancelled B, **not A's current handle**. But `POST /accounts/logout/` expressly requires the current ACTIVE handle/epoch (`auth-transition-protocol.md:91`, `contracts/http.md:59`). It returns 401 and **cannot revoke A's server family/session or its other tabs**. The app cannot truthfully report successful logout. Even without a logout click, ordinary A task use will require another sign-in despite A's still-live server session.

The proposal **explicitly acknowledges forced sign-in** after late provisional/cancelled responses (`auth-transition-protocol.md:95–105`; `contracts/ui.md:130–133`; author handoff §4), but that is not the same as defining what an apparent logged-in tab does when the user requests family-wide logout. It is also a predictable application-generated credential loss during normal tab switching, **not** an external browser-cookie eviction. The planned `R5-V08a–d` covers lost headers and logout races generically, but does **not** verify this *claim(A) → delayed cancelled ISSUE cookie overwrites A → user clicks logout* sequence with an expected authoritative revocation outcome.

**Minimum remedy:** Define a safe, verifiable response to **late cancelled ISSUE overwriting a currently usable credential**, including current visible task state, current-login continuity and the ability to complete the user's cross-tab logout intent. Do not represent a 401/failed logout as successful server revocation. Either preserve the approved shared-logout/session behaviour under this ordering or explicitly identify the narrower product behaviour/risk and obtain the appropriate human/Functional Design approval before treating it as acceptable. Extend targeted tests with this exact schedule plus a previously successful B login overwritten by an older cancelled ISSUE response, failed/lost announcements, CSRF and server-session status assertions. No particular cookie/locking redesign is mandated.

**Related, nonblocking implementation contract check:** Django 5.2 rotates CSRF tokens on login and notes that pages opened before a login in another tab hold invalid CSRF tokens until refreshed. Because `POST /api/session/claim/` is mandatory before revealing the newly authoritative login, specify whether the client reads the **current** CSRF cookie/token at dispatch or performs a safe reload/rebootstrap on 403; ensure a stale tab cannot misreport a CSRF failure as a completed logout. Test the two-tab login/CSRF rotation case. Official reference: https://docs.djangoproject.com/en/5.2/ref/csrf/ . This belongs in TD-008's session handoff rather than multiplying finding IDs.

## 4. Cross-cutting architecture, security and feasibility

| Dimension | Independent result |
|---|---|
| **Requirement fit** | Approved AM-001 all-four-view state, task-first bootstrap, defaults on new completed login, old/new tab controls, ownership and task-loading errors are retained. The late-ISSUE/logout mismatch remains a material unapproved interaction. |
| **Security and trust** | High-entropy opaque HttpOnly host-only handle, server pinning, missing-cookie fail-closed and family-wide generation are a stronger security model than Rev4. CSRF, auth throttling and live Session checks remain obligatory. No new browser capability is required. |
| **Data integrity/concurrency** | Separate state transitions, FK discoverability, lock ordering, non-reuse of stale generation and guarded deletion are specified. FKs must actually protect current live records; race/deadlock/deadline/restore evidence still outstanding. |
| **Failure and recovery** | Unknown/missing handle means 401/conceal; registry/storage error means 503/conceal. This is secure but may be **self-induced** through a delayed CANCELLED ISSUE response. The resulting logout and re-login UX needs explicit closure. |
| **Capacity/complexity** | One cookie and explicit four-per-family/120-second/cleanup bounds improve over Rev4. The additional registry on auth paths and cleanup backlog are plausible within existing pools as a hypothesis; test 1,500-admission envelope plus sustained live-session storage before release. No new production performance claim is proven. |
| **Operational handoff** | TD-006 restore plan now expressly revokes Family/Handle/Transition alongside Django Session/View before traffic resumes. This is a correct addition, not evidence of a passed restore drill or approved RPO/RTO. |
| **Historical decisions** | Rejected Web Locks alternative remains rejected. No cross-user exposure interval or unsupported-browser exception is requested. TD-001 closure is **reviewer design assessment**, not acceptance of new residual privacy risk. |

## 5. Required next step and gate

1. **Codex:** Address TD-008's late-ISSUE replacement case and resulting inability to perform an authoritative logout from a tab that was just validated under A. Clarify whether aborted/competing valid login attempts may unexpectedly force sign-in, and why that outcome is consistent with the approved experience. Complete the CSRF cross-tab revalidation/reload contract and tests. Preserve all other resolved technical mechanisms.
2. **If Codex can correct without changing approved behaviour:** Return a narrow per-finding evidence map, targeted modified-file hashes and deterministic planned scenarios for **R3-Rev6 independent review**. Preserve Scope, R2 approvals and R3 review history. No Tasks/Analyze/Build.
3. **If correction requires accepting failed logout or interrupted-login forced sign-out as intended behaviour:** Escalate the precise user-visible consequence to the appropriate R2/Scope decision with Barry. Do **not** infer such approval from existing FD-003/FD-004 risk records or from this reviewer report.

**No human choice is necessary just to send this technical gap to Codex.** A human decision becomes necessary if the remedy cannot preserve the intended multi-tab logout/session behaviour and Codex proposes accepting the altered outcome.

### Formal verdict

**R3-Rev5: NOT READY.** TD-001 is **RESOLVED at static design level**. TD-008 remains **OPEN / Medium blocker** for a concrete, ordinary response-order/session-logout interaction, with one CSRF integration check attached. TD-002/003/004/005/007/009 remain design-resolved; TD-006 remains an open, nonblocking R5 readiness evidence dependency. **Human R3 approval: not given.** No tests or release-readiness checks were performed. R3 must remain open; do not advance to Spec Kit Tasks/Analyze or Build.

## 6. Independent review history

- **R3-Rev1:** NOT READY, TD-001–007 open.
- **R1-Rev4/R2-Rev4:** AM-001 Scope and amended Functional Design independently reviewed and subsequently explicitly human APPROVED; historical FD-003/004 accepted risks retained.
- **R3-Rev2:** NOT READY; TD-002 resolved; TD-008/009 introduced.
- **R3-Rev3:** NOT READY; TD-003/004/005/007/009 resolved at design level; TD-006 R5 accountability recorded; TD-001/008 still open.
- **R3-Rev4:** NOT READY; staged multi-cookie correction remained dependent on unsupported cookie-presence invariant, leaving TD-001/008 open.
- **R3-Rev5 (this report):** Nine current hashes, approved scope/checklists/handoff and R3-Rev4 review identity independently checked; unchanged §§3–8 compared to Rev4. Server-family/generation resolves **TD-001 in writing**; single-cookie approach addresses old TD-008 fallback but introduces the documented late-provisional overwrite/logout consequence. **TD-008 stays open; no human R3 decision or risk acceptance recorded.**
