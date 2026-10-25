# Taskify — Independent R3-Rev8 Targeted Technical Design Review

**Review date:** 2026-10-24 (Australia/Melbourne)  
**Independent reviewer verdict:** **NOT_READY**  
**Human R3 gate:** **OPEN — NOT APPROVED**  
**Review scope:** Incorporation of human-approved R2-Rev6 Functional Design into operative artifacts, and directly affected technical design mechanisms (TD-008, newly identified TD-010/011). Review of design documentation only; no implementation or runtime tests.

## 1. Decision summary

**The R2-Rev6 incorporation is verified, but R3 cannot close.** The approved FD-006/007/008 and Q1 requirements are accurately represented in the operative functional contracts. Codex has not selected or demonstrated an authentication architecture capable of satisfying the additional guarantees. This is correctly disclosed as a technical hold, not hidden as functional approval.

The principal blocker remains **TD-008**, especially the first browser-established login winner under concurrent anonymous POSTs. The former Django-native Option B remains an assessment candidate: its natural last-applied `Set-Cookie` behaviour does not satisfy now-approved FD-007. Two further directly related technical design gaps are tracked as **TD-010** (newly arrived private response reveal ordering) and **TD-011** (prompt cross-tab logout notification).

R3 remains **NOT_READY**. No Tasks, Analyze, Build, implementation, deployment or human risk acceptance is authorised by this review.

## 2. Authoritative baseline and verification

| Evidence | Independently established result |
|---|---|
| Reviewed attachment | `Archive(2).zip`, SHA-256 `13ff59f3269b75fb5ad3faa7da9b39f0018556cde45db4ee3b6330e79ab0b950` |
| Approved Scope | `specs/001-personal-task-manager/spec.md`, SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`; R1-Rev4 preserved |
| R2-Rev5 | Human-approved R2-NAV-001; independent report SHA-256 `f08218d808638187bb5227a386e5b8c91913e481308a62a2f62b9016559eca90` |
| R2-Rev6 | `independent-reviews/2_design-functional/Review_6/Taskify_R2-Rev6_Targeted_Functional_Design_Review_APPROVED.md`, SHA-256 `bf5655e36c120dd79b35542430915de56817f4587245774abc50cad2ffafbf6f`; independent READY_FOR_HUMAN_APPROVAL followed by Barry Levy's formal APPROVED decision |
| Reconciliation manifest | `review-followups/r2-rev6-incorporation.sha256`: **17/17** entry hashes independently recomputed and matched |
| Preservation comparison | Against preceding attached `Archive(1).zip`: **68** shared `.md`/`.sha256` paths, **9 changed operative files**, **59 byte-identical**; no unrelated historical approval changes |
| Scenario coverage | R2-6R-T01–16 incorporated unchanged into `quickstart.md`; these are **planned** acceptance scenarios, not executed tests |
| Previous technical review | R3-Rev7 technical review and simplicity addendum retained; TD-008 remained Medium/blocking, TD-006 nonblocking Operations/R5 dependency |

The nine modified operative files are `contracts/ui.md`, `contracts/http.md`, `data-model.md`, `research.md`, `plan.md`, `quickstart.md`, `auth-transition-protocol.md`, `technical-design.md` and `checklists/feature.md`. The change/evidence map and targeted R3 handoff were also examined. Original R2-Rev6 proposals and audit records remain preserved.

Review method: static artifact comparison, requirement-to-design tracing, concurrency/failure scenarios and technical feasibility assessment. No browser execution, tests, database inspection, deployment, build or implementation was performed. Codex's static-check claims were not treated as runtime proof.

## 3. Reconciled approved functional obligations

| Obligation | Incorporated contract | R3 consequence |
|---|---|---|
| FD-007 / FR-010 | First **browser-established usable login** prevails against delayed competing anonymous responses, even for the same account; no temporary credential/account/epoch/view replacement; loser gets appropriate conflict/error | Server lock order or a 409 message alone cannot prove browser cookie delivery safety |
| S2-DEC-001 | Distinct still-valid late login **may** authenticate after confirmed revocation of the original login; original remains invalid; a subsequently established login remains protected | Must distinguish independent valid late authority from revival of revoked/rejected authority; no universal cancellation promise |
| FD-006 | After a private payload arrives, validate current authentication/account/epoch before a **new reveal**; discard mismatch, conceal uncertainty; never rebind old intent | Must prove response/validation/paint ordering; initial/focus-only checking is insufficient |
| FD-008 / S4-DEC-001 | Confirmed logout revokes intended login; runnable same-profile tabs promptly clear/navigate using notification; suspended/missed-notification tabs validate on resume/focus/history; no simultaneous-render or tab-ACK guarantee | Requires a specified notification mechanism scoped to the revoked login and fail-closed action/reveal gates |
| Accepted Q1 | Private reads and cached reveals, dialogs, mutations/retries, revealing filter/sort, view remembering and logout are gated; no per-keystroke check | Technical guard must bind intended epoch from input through server effect and reply |
| Retained design | R2-NAV-001 redirects, valid-login view persistence, FD-001 write locks, exact receipt/version semantics, isolated silent view fallback, CSRF recovery and independent devices | Regress only affected interfaces; do not reopen unrelated decisions |

No demonstrated Scope amendment or additional product decision is required by the reconciliation itself. R2-Rev6 human approval does **not** select an authentication design or close a technical finding.

## 4. Finding register

### TD-008 — Anonymous concurrent authentication and session continuity

- **Classification / severity / confidence:** Design Gap / **Medium, BLOCKING** / High.
- **Disposition:** **OPEN**, carried from R3-Rev7.
- **Evidence:** `technical-design.md` §§0–2, `auth-transition-protocol.md` §§0,4 and `contracts/http.md` authentication section explicitly label inherited family/handle mechanisms unapproved and anonymous login delivery a technical hold. `review-followups/option-b-django-native-design-proposal.md` §2 permits last-applied-cookie selection rather than FD-007's first-established winner.
- **Requirement:** FD-007, S2-DEC-001, FR-010/AC-010, R2-NAV-001, valid-login view lifetime and confirmed shared logout.
- **Failure mode / impact:** Two previously anonymous tabs submit login forms; browser response/header application order differs from server commit order. A delayed competitor may overwrite a usable login or its cookie, impair task access/logout or select another account/view. A 409 without actual cookie suppression is not a remedy. Concurrent `Set-Cookie` behaviour is a recognised HTTP race (RFC 6265 §4.1.1).
- **Minimum remedy:** Select and specify one cohesive credential issuance/selection/revocation mechanism. Demonstrate both header orders, same/different accounts, reversed server completion, no competing auth-cookie overwrite, winner's view and logout continuity, post-revocation bounded S2, and no invalid-session resurrection. Reassess the need for each bespoke registry/phase/rotation mechanism; prefer the simplest established Django-compatible approach that meets all approved behaviour.
- **Owner / escalation:** Codex — design and evidence. Refer any genuinely necessary change to FD-007, cookie/browser support or availability to Barry for a narrowly authorised R2/Scope decision **before** incorporation; do not silently relax it or adopt paused P1/IndexedDB.
- **Exit:** Targeted independent R3 review verifies a coherent complete design and deterministic R2-6R-T01–04/16 scenario mapping. Downstream implementation tests remain planned.

### TD-010 — Newly arrived private response validation and reveal ordering

- **Classification / severity / confidence:** Design Gap / **Medium, BLOCKING** / High.
- **Disposition:** **OPEN — NEW in R3-Rev8**.
- **Evidence:** `contracts/ui.md` FD-006 and `contracts/http.md` FD-006 prescribe current auth/account/epoch verification after payload arrival; `technical-design.md` §0 and `auth-transition-protocol.md` §0 expressly state that the inherited focus/claim mechanism does not prove this requirement. `r2-rev6-r3-handoff.md` identifies the unresolved validation/reveal race.
- **Requirement:** FD-006, FR-010/AC-010, accepted Q1, R2-6R-T07–09.
- **Failure mode / impact:** An A-private response is buffered; the current login becomes B before reveal or between an earlier validation and render. Without a coherent guard, a new A payload might be exposed in B's UI or an old draft/action might be applied under B's session. This is distinct from **already-painted** private rows persisting until the next required interaction under accepted S4 behaviour.
- **Minimum remedy:** Specify a verifiable identity/epoch/generation and response-discard sequence spanning arrival, required fresh validation and DOM/accessibility reveal, with safe handling of cookie changes and authority uncertainty. Preserve ordinary same-login task-refresh errors/FD-001 locks and silent isolated view failure; no unconditional hiding of legitimate stale rows to mask the gap.
- **Owner / escalation:** Codex. Escalate only if the approved privacy contract proves infeasible under a proportionate design; no exposure interval is presently accepted.
- **Exit:** Written mechanism and deterministic T07–T10/race coverage verified independently, integrated with TD-008's session-selection mechanism.

### TD-011 — Confirmed logout cross-tab notification and fallback

- **Classification / severity / confidence:** Design Gap / **Medium, BLOCKING** / High.
- **Disposition:** **OPEN — NEW in R3-Rev8**.
- **Evidence:** `contracts/ui.md` FD-008 and `data-model.md` require prompt clear/login navigation after confirmed shared logout; `technical-design.md` and `auth-transition-protocol.md` retain earlier BroadcastChannel-as-hint language without adopting and proving the new prompt-notification protocol. `r2-rev6-r3-handoff.md` leaves mechanism/failure handling to R3.
- **Requirement:** FD-008, S4-DEC-001, S2-DEC-001, R2-6R-T05/06/11/12/15.
- **Failure mode / impact:** A runnable tab continues to display old private data after confirmed logout because no specified notification is dispatched/handled; delayed L notices may erroneously clear newer valid login N. Suspended/missed-notification tabs also require safe resume/interaction gates.
- **Minimum remedy:** Prefer established same-origin browser messaging such as `BroadcastChannel`, scoped by intended login epoch, with prompt local clearing/navigation on confirmed revocation; specify lost/late/duplicate notification, suspension, bfcache/history, uncertain ACK and revalidation fallbacks. No per-tab ACK wait or instantaneous clearing in suspended contexts is needed.
- **Owner / escalation:** Codex. No further product decision currently required.
- **Exit:** Mechanism and T05/06/11/12/15 failure scenarios written and independently verified; server revocation remains authority, not a broadcast.

### Prior findings and residual dependencies

| Finding | R3-Rev8 status | Qualification |
|---|---|---|
| TD-001 High | Previously RESOLVED at design level; retained | Reverify cross-account privacy if its supporting family/claim mechanisms are removed or materially changed |
| TD-002/003/004/005 | Previously RESOLVED; retained | No new regression demonstrated; downstream tests/measurements still required |
| TD-006 Medium | **OPEN, NONBLOCKING** | Barry owns Operations/R5 backup/restore/diagnostics exit evidence |
| TD-007 Low | Previously RESOLVED; retained | Barry owns maintenance/R4 compatibility and upgrade fixtures |
| TD-008 Medium | **OPEN, BLOCKING** | New FD-007 makes response-order-only Django Option B noncompliant |
| TD-009 Medium | Previously RESOLVED; retained | Independent optional-view timeout/failure isolation must regress |
| TD-010 Medium | **OPEN, BLOCKING — NEW** | FD-006 after-arrival reveal ordering not technically established |
| TD-011 Medium | **OPEN, BLOCKING — NEW** | FD-008 prompt notification mechanism not technically established |

Previously accepted FD-003/004 risks remain accepted **only within their original documented bounds**; none of the blockers above is newly accepted. No claimed privacy incident or observed runtime failure is inferred from these documentation gaps.

## 5. Simplicity, established solutions and architecture reframing

**Challenge necessity:** The ordinary Taskify flow is straightforward Django session authentication, a shared same-profile cookie, per-account data isolation and session-scoped view state. Concurrent anonymous login forms are an exceptional but permitted flow. R2-Rev6 now **explicitly** requires a first-browser-established winner, so the reviewer cannot dismiss that contract because the race is rare. The actual probability is not measured; effects can be consequential for account continuity and privacy.

**Prefer established mechanisms:** Django-native authentication and DB-backed sessions remain the appropriate baseline. Standard logout invalidates the request's Session but cannot revoke an independent second Session minted by another previously anonymous request. Concurrent `Set-Cookie` responses can race independently of server locks. `BroadcastChannel` is a suitable established browser notification primitive, not an auth/session authority. Keep framework protection against fixation, CSRF and owner isolation; add only independently justified coordination.

**Custom mechanisms only where necessary:** `AuthorityFamily`, `AuthorityHandle`, PREPARE/ISSUE/ACTIVATE, bespoke rotation and P1/IndexedDB should **not** be reinstated wholesale merely because FD-007 is demanding. Codex should show why each additional mechanism is needed and how it preserves availability, no-secret-in-client-storage constraints, failure isolation and ordinary task/view performance. If successive patches expand the state machine, reconsider the overall approach and, if genuinely unavoidable, escalate the exact approved product trade-off rather than accumulating exceptions.

**Current judgment:** The revised R2-Rev6 documents correctly stop short of choosing a solution. A new architectural decision is necessary; technical closure is not possible from functional-contract incorporation alone.

## 6. Required remediation and verification handoff

1. **Codex — integrated technical design:** Resolve TD-008, TD-010 and TD-011 together where they share authentication/epoch state; identify the smallest viable architecture, select/justify mature mechanisms and explicitly retire superseded candidate text. Keep P1 paused unless a separately approved need emerges.
2. **Codex — targeted artifacts:** Update `technical-design.md`, `auth-transition-protocol.md`, `contracts/http.md`, `contracts/ui.md`, `data-model.md`, `research.md`, `plan.md` and affected validation materials only as warranted by the selected mechanism. Supply an ADR, requirement/decision trace, preserved-historical-evidence map and verified changed-file hashes. Do not rewrite independent prior reviews or Scope/R2 approvals.
3. **Codex — deterministic validation design:** Include both actual response-header orders, server-order inversions, same-account epochs, third-login-after-logout, old-session replay, task/logout commit race, after-arrival validation/paint races, lost/broadcast-delay/suspension, CSRF retry and retained task/view/receipt invariants. Map to R2-6R-T01–16 plus prior NAV/AM001/R3 scenarios. No runtime pass may be claimed before execution.
4. **Independent R3 reviewer:** Reassess the revised mechanisms, significant new risks and affected regressions; update stable finding dispositions and give a fresh verdict. Previously resolved unrelated TDs remain closed unless evidence demonstrates regression.
5. **Barry Levy:** Explicitly approve R3 only after a READY/appropriately justified conditional verdict and review of residual material risk. Do not infer approval from R2-Rev6 or from this report.

## 7. Exit gate

**Independent R3-Rev8 verdict: NOT_READY. Human R3 gate OPEN.** Blocking: **TD-008, TD-010, TD-011**. Nonblocking dependency: **TD-006** for Operations/R5. No new human product choice is requested now. The next authorised activity is Codex technical **design/remediation documentation** followed by independent R3 re-review; **not** Tasks, Analyze, Build, testing, deployment or production release.

This report was created after the in-chat targeted assessment and does not retroactively assert that a prior R3-Rev8 file existed in the submitted archive.

## Primary technical references

- [Django 5.2 authentication and session management](https://docs.djangoproject.com/en/5.2/topics/auth/default/)
- [Django 5.2 server-side sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/)
- [Django 5.2 SessionMiddleware source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/middleware.py)
- [RFC 6265, §4.1.1 — concurrent Set-Cookie responses](https://www.rfc-editor.org/rfc/rfc6265.html)
- [MDN Broadcast Channel API](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API)
- [PostgreSQL row-level locks](https://www.postgresql.org/docs/17/explicit-locking.html)
