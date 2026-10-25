# Taskify — Independent R2-Rev5 Targeted Functional Design Review

**Date:** 2026-10-24 (Australia/Melbourne)  
**Reviewer:** Independent Functional Design Reviewer (R2)  
**Review boundary:** FR-010 authenticated navigation, stale sign-in, shared logout and the affected FR-012 session view; targeted regressions only  
**Independent verdict:** **READY_FOR_HUMAN_APPROVAL**  
**R2-Rev5 human gate:** **CLOSED — APPROVED by Barry Levy on 2026-10-24 (Australia/Melbourne)**; see §7  
**R3 gate:** **NOT_READY / unapproved**; this review does not authorise Tasks, Analyze or Build.

## 1. Authority, exact evidence and limits

| Item | Independent evidence |
|---|---|
| Supplied package | `Archive(1).zip`, SHA-256 `786e2886e786cd5b7a5e2e4cbd82efff2f07b3c6aec55f4fb6ae3a7f88a5b1db` |
| Scope | Canonical `specs/001-personal-task-manager/spec.md`, SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`, independently checked. Scope 1.1/AM-001 human approval is recorded in `independent-reviews/1_scope/Review_4/scope_review_report.md` §8. |
| Earlier functional approval | `independent-reviews/2_design-functional/Review_4/taskify_R2-Rev4_functional_design_review.md` §§8 and 136–168: independent READY, followed by Barry's explicit human APPROVED decision on 2026-10-23; exact approved file manifest at its lines 151–165. |
| Latest supplied independent R3 verdict | `independent-reviews/3_design-technical/Review_4/taskify_R3-Rev4_technical_design_review.md`: NOT_READY; TD-001 (High) and TD-008 (Medium) remain technical blockers there. The source documents refer to a later R3-Rev5 author proposal, but **no independent R3-Rev5 review report was provided**. |
| New change record | `review-followups/r3-navigation-clarification-evidence.md`, manifest `r3-navigation-clarification-changed-files.sha256` and eight revised design files. Independently recomputed all nine manifest SHA-256 values; **all match**. |

Reviewed the actual present `contracts/ui.md`, `contracts/http.md`, `auth-transition-protocol.md`, `technical-design.md`, `data-model.md`, `research.md`, `plan.md`, `quickstart.md`, canonical Scope and the earlier R1/R2/R3 review records. Author assurances and planned scenarios were not counted as proof of runtime behaviour. The package lacks the original R2-Rev4 *design file bytes*, although their exact hashes are recorded in that approval; consequently the targeted historical comparison relies on the approved R2 report's behavioural description and the candidate's explicit change markers, not a bytewise old/new source diff.

## 2. Verdict and actual approved-behaviour delta

**READY_FOR_HUMAN_APPROVAL**, not automatically approved. The intended four navigation/session outcomes are specified coherently and observably in the current candidate. No new **functional** blocking defect, Scope expansion or required product choice was substantiated beyond ratifying the one known change to previously approved R2 behaviour.

**R2-NAV-001 — an actual Functional Design change requiring explicit R2 human approval:** R2-Rev4 permitted a repeated, successful **same-user** credential login to create a fresh view lifetime/defaults. In the new candidate, an **already authenticated** visit to the login URL or submission of its form **does not log in again**: it redirects to the existing task list without changing account, login epoch or remembered view. A *genuinely fresh* login after logout/expiry still creates a new session with defaults. This is a reasonable clarification of FR-010 and consistent with FR-012's “still-valid login” distinction; nevertheless it **supersedes a behaviour in the human-approved R2-Rev4 HTTP design** and needs a new human R2 gate decision. Evidence: R2-Rev4 lines 68–74 (especially AM001-V04); current `contracts/http.md` lines 38–74, `contracts/ui.md` lines 13–28 and 179–185, `quickstart.md` lines 89–98 and 167–174.

The current candidate also makes the necessary dependent rules explicit: authentication-status lookup precedes the normal login route, stale/history forms must revalidate, account B may sign in after A's logout, and a shared logout revokes the same-profile session across tabs. These are implementation-level elaborations of the stated FR-010 navigation intent, not independent new user-facing capabilities. No canonical Scope amendment is recommended on this evidence.

## 3. Targeted traceability and behavioural coverage

`COMPLETE` means the **written functional contract is testable**, not that code or tests have passed.

| Requirement / scenario | Rating | Evidence and independently reviewed outcome |
|---|---|---|
| FR-010 / AC-010 — private account, same-profile new tab | **COMPLETE** | `ui.md:13–28`, `http.md:16–24,38–43`, NAV-01/02: authenticated entry reaches tasks with existing identity/session/view; no new login, cookie, epoch or preference reset. Account ownership remains enforced (`http.md:128–136`, `quickstart.md:77,110`). |
| FR-010 — attempted login while already authenticated | **COMPLETE; APPROVAL DELTA** | `http.md:21–22,45–54,64–74`, `ui.md:21–28`, NAV-03/04/08: GET 302 /tasks/, valid-CSRF POST 303 /tasks/ *before replacement-credential evaluation*; invalid CSRF 403; stale phase 409, without active-login replacement. R2-NAV-001 applies. |
| FR-010 — logout, switch account and other tabs | **COMPLETE** | `http.md:23,89–100`, `ui.md:100–101,127–169`, NAV-05: logout commits shared revocation, old tabs conceal/clear and cannot authenticate with the old credential; B or A can subsequently sign in fresh. Separate browser/device sessions remain independent. |
| FR-012/013 / AC-005 — view lifetime, filters/sorts | **COMPLETE** | `spec.md:173–174,192`, `ui.md:57–91,179–185`, `data-model.md:37–72`, AM001-V01–09, NAV-01/08: valid shared login retains acknowledged tuple; explicit new login resets; same-login other tabs restore current complete saved tuple; unrelated sessions independent. Four sorts/two directions and tie/undated rules retained. |
| FR-009/014 / AC-008 — expiry, loading and uncertain outcomes | **COMPLETE** | `ui.md:95–119,127–162`, `http.md:121–136`, NAV-06, AM001-V10, R2-T05: private view concealed on invalid/uncertain authority; failed task load retains Retry/support and blocks mutations; isolated view-state read/save failure remains silent and must not block otherwise loadable tasks. |
| FR-008 / AC-009 / SC-004 — task persistence and independent devices | **COMPLETE (regression check)** | `research.md:62–80,89–109`, `plan.md:50–61`, `quickstart.md:104–110`: saved task state remains account-wide; only the view is login-scoped, and separate device sessions are not logged out by a same-profile logout. |
| FR-015/016 — browser/registration boundary | **COMPLETE (targeted)** | `http.md:19–31`, `ui.md:30–39`, `plan.md:42–46`, `quickstart.md:77`: existing first-time registration and browser coverage retained. Browser engine execution remains a downstream test obligation. |

### Critical behaviour matrix

| Starting state and event | Required written result | Coverage |
|---|---|---|
| A is signed in; A opens Taskify root/new tab | Tasks for A, same epoch and remembered view, no new login | NAV-01, `http.md:18,40–43` |
| A is signed in; navigates to login GET/POST, with B or A credentials | Redirect to A tasks; do not validate replacement credentials or change session/view | NAV-02/03/08, `http.md:21–22,45–54` |
| Login form survives history; A has since signed in | Revalidate before submitting/revealing form; authenticated request navigates to tasks | NAV-03/04, `ui.md:21–28` |
| A logs out from tab 1 | Shared A session becomes invalid in all same-login tabs; stale private content hidden/cleared before re-reveal | NAV-05, `ui.md:127–169` |
| B logs in after A's committed logout | Fresh B account/epoch; default view, no access to A data or A's view | NAV-05/08, AM001-V04/09 |
| A's login expires; A signs in again | New login and default view; no accidental import of prior session view | NAV-08, AM001-V04 |
| Authority lookup is unavailable | 503, concealed Retry/support; **not** treated as signed-out or as a silent view failure | NAV-06, `http.md:18,21–22,79–83` |
| Login/logout overlaps stale requests and a failed task refresh | Old responses cannot reveal/mutate another account; normal task-load failure retains FD-001 protection | NAV-07, R5-V01a–c/V08a–d, `ui.md:127–169` (**written contract only; R3 mechanism unverified**) |

## 4. Finding and decision register

### R2-NAV-001 — previously approved repeated same-user login/reset semantics

- **Classification:** Change-control / human-gate item, **not a new functional defect finding**. **Severity:** MEDIUM governance/behavioural significance. **Likelihood:** N/A, deterministic changed path. **Confidence:** HIGH for the documented delta (with the old R2-Rev4 source-bytes limitation in §1). **Owner:** Barry (approve/reject behavioural replacement); Codex (record decision and reconcile dependent artifacts if required).
- **Counterexample under the old design:** A has a remembered nondefault view and explicitly submits A's credentials again while still signed in. The approved R2-Rev4 HTTP design allowed a successful repeated login to reset the view. The new `http.md:45–54,64–70` returns to the existing A tasks without changing the remembered view. These outputs differ.
- **Scope check:** FR-010 authorises private sign-in and account isolation but does not require re-authentication of an already-authenticated tab; FR-012 binds view state to a *still-valid login* and demands defaults for a **new** login. The revised distinction respects both requirements and the four user-stated clarifications.
- **Disposition:** **OPEN — HUMAN APPROVAL PENDING; design evidence independently VERIFIED.** Verification does **not** imply approval of the replacement. Do not amend the original R2-Rev4 approval record or pretend the old semantics never existed.
- **Required human decision:** Explicitly approve the R2-Rev5 navigation replacement, including same-user authenticated GET/POST retaining the existing view; confirm that only logout/expiry followed by sign-in starts a new view lifetime. If rejected, return to Functional Design; do not silently reinstate a re-login path.

### Targeted residuals; not new R2 blockers

| ID / item | Disposition | Reason / owner |
|---|---|---|
| **FD-001** | RESOLVED_VERIFIED historically; **no regression substantiated** | Task refresh/load failure still locks writes and preserves Retry/support (`ui.md:106–119,158–162`). Codex/Test to exercise R2-T05. |
| **FD-002/FD-005** | RESOLVED_VERIFIED historically; AM-001 supersession retained | Session-scoped, silent best-effort view remains the approved replacement. No account-preference GET/PUT or save-warning guarantee reintroduced. |
| **FD-003/FD-004** | **ACCEPTED_RISK — retained** | Interrupted memory-only task-operation identity and no credential recovery remain as previously accepted; no new acceptance made. |
| **E-001–E-003** | Historical dispositions unchanged | No targeted re-opening justified. |
| **TD-001 / TD-008** | **R3 UNVERIFIED / blocking there** | Proposed server family/one-cookie ordering and stale-request handling are technical mechanisms, including loss or reordering of cookie responses. Independent R3 must test/prove both-lock-order, partial-cookie, distinct anonymous families, logout and DOM concealment cases; R2 cannot close them. |
| **TD-002/003/004/005/007/009** | Earlier R3 design-level resolutions **not re-opened here** | Implementability and runtime proof remain downstream. |
| **TD-006** | OPEN nonblocking R5 operational-evidence dependency | Owner Barry; do not imply restore/operations readiness. |

**Minor, nonblocking UX consideration:** `http.md:19–20` still allows the registration page/POST while A is authenticated. A could create another account, then be redirected to a login route that sends A back to A's tasks. This does not authenticate as B, does not expose B's tasks, and does not violate FR-016's first-time registration requirement. Codex may improve authenticated registration-page guidance/redirect in a later approved UX pass; no new R2 finding or gate blocker is justified by this edge case.

## 5. R3-only dependencies and verification discipline

The proposal's **AuthorityFamily**, claim/generation locks, provisional auth cookie, PREPARE/ISSUE/ACTIVATE, maintenance rotation, cleanup deadlines, and Django session handling (`auth-transition-protocol.md:30–196`, `technical-design.md:31–91`) are engineering decisions **owned by R3**. This review accepts neither their correctness nor a relaxed privacy guarantee. Specific R3 replay is needed for:

1. Old captured headers after A signs in; same-family stale activation, missing proofs, both lock orders, dropped broadcasts and stale DOM/accessibility/paint.
2. Separate anonymous forms/families, late cookie headers, lost/provisional cookies, candidate cleanup and forced sign-in: ensure no silent A→B identity selection, cross-account reveal or stale mutation. The design allows loss of a credential to require sign-in; R3 must assess the mechanism and whether any real execution shows a broader user-facing availability change.
3. Logout from any tab during candidate issuance/activation, stale history restoration, same-epoch task refresh failure and independent-device nonrevocation.
4. Regression to AM001-V01–10, R2-T05/T07/T12 and all four sort/filter settings, without inventing an auth policy or weakening FD-001.

`quickstart.md:127–180` lists planned R5-V01a–c/V08a–d and NAV-01–08. They are **acceptance/technical test plans**, not executed evidence. Any implementation discovery requiring browser exclusion, cross-user private exposure tolerance, changed logout semantics, or weaker task-load protection must return to the authorised Scope/Functional Design/human process.

## 6. Exit gate and handoff

| R2-Rev5 targeted gate criterion | Result |
|---|---|
| Approved Scope and earlier R2 authority identified | **PASS** |
| Exact current candidate files verified against manifest | **PASS — 9/9** |
| Four user-intended navigation/session outcomes specified | **PASS** |
| Known repeated same-user login/view reset delta isolated and disclosed | **PASS — R2-NAV-001** |
| Material login/logout/expiry, cross-tab and view/sort regression coverage | **PASS at written-functional level** |
| New material R2 behavioural defect/gap requiring Codex correction | **NONE SUBSTANTIATED** |
| New Scope amendment required | **NO on reviewed evidence** |
| Human approval of the changed R2 design | **PENDING** |
| R3 technical sign-off / application runtime assurance | **NOT GRANTED** |

**Formal independent verdict: READY_FOR_HUMAN_APPROVAL.** Barry should either explicitly **APPROVE R2-Rev5** for the targeted navigation change identified in R2-NAV-001, or request correction. His clarification in the review request is evidence of intended behaviour, **not** a substituted final R2 gate decision.

Upon explicit human approval, Codex should record the new gate and exact accepted design-file identities, retain the prior R1/R2/R3 reports and FD dispositions untouched, then submit the affected authentication mechanism for **independent R3 targeted review**. An R2 approval would certify written functional behaviour only, not close TD-001/008 or authorise Tasks, Analyze or Build.

## 7. Post-review formal human R2-Rev5 decision — 2026-10-24

**This subsequent human decision supersedes only the pending-approval status in §§2, 4 and 6.** The original independent assessment, review evidence, findings history and reviewer verdict remain intact.

| Decision field | Recorded authority |
|---|---|
| Review / gate | Taskify **R2-Rev5** — targeted Functional Design approval for FR-010 authenticated navigation and its dependent FR-012 view lifetime |
| Independent recommendation | **READY_FOR_HUMAN_APPROVAL** — unchanged; this was the reviewer's verdict, not the human decision |
| Human decision | **APPROVED** — Barry Levy, authorised human product owner |
| Exact declaration | “I approve Taskify R2-Rev5 targeted Functional Design, including R2-NAV-001. Record the human R2-Rev5 approval.” |
| Decision date | **2026-10-24 (Australia/Melbourne)**; no unsupported time-of-day asserted |
| Approved Scope authority | Scope v1.1 / AM-001, approved R1-Rev4; current canonical `spec.md` SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca` |
| Approved Functional Design change | **R2-NAV-001 accepted:** an already-authenticated visit/submission at the login page preserves the existing account, session/epoch and remembered view, rather than creating a repeated same-user login/reset. Only an actual fresh login after logout/expiry starts a fresh default view. The four stated new-tab, logout-first, shared-revocation and cross-account-isolation outcomes are included. |
| Exact reviewed candidate | The eight design documents identified and individually hashed in `specs/001-personal-task-manager/review-followups/r3-navigation-clarification-changed-files.sha256`; manifest SHA-256 `78fa6c6d0287df35e8a0928fa6baae576693c48bc3947da786651935b2a82b1d`. The accompanying evidence map is included as the ninth matching hash. |
| R2-NAV-001 post-decision status | **CLOSED — APPROVED CHANGE**; independently verified at written-functional level. This is a change-control disposition, **not** proof of executable implementation or R3 architecture. |
| R2-Rev5 gate disposition | **CLOSED — APPROVED** for this targeted Functional Design, replacing only the relevant behaviour of the R2-Rev4 approved baseline; all unaffected prior approvals and accepted-risk records remain valid. |
| Still excluded | Technical mechanism validation and TD-001/008 closure; implementation/test execution; R3 human approval; Tasks, Analyze and Build. |

**Next authorised workflow action:** Codex may record/reference this explicit R2-Rev5 human approval without silently changing the reviewed functional baseline, and return the affected technical design for independent **R3** assessment. R3 remains **NOT_READY / unapproved**; this R2 decision does not grant downstream execution authority. The original R2-Rev4 approval record must not be overwritten.
