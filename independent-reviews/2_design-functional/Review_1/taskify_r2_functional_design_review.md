# Taskify / Personal Task Manager — Independent R2 Functional Design Review

**Review ID:** R2-R1  
**Review date:** 2026-10-22 (Australia/Melbourne)  
**Reviewer:** Independent Functional Design Reviewer (R2)  
**Gate verdict:** **NOT_READY** for R2 human approval  
**Scope of review:** Design evidence supplied in `/mnt/data/specs.zip`; static artifact review, not implemented-system testing  
**Review status:** Initial review complete; FD-001 and FD-002 resolutions approved and IN_PROGRESS (design amendments not yet verified); FD-003 residual risk ACCEPTED_RISK (documentation amendments outstanding); FD-004 policy decision APPROVED / ACCEPTED_RISK (documentation amendments outstanding)

## 1. Authority, inventory and review boundary

The canonical Scope baseline is `specs/001-personal-task-manager/spec.md`, SHA-256 `4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`. This matches the previously human-confirmed Scope-approved baseline. The ZIP contains nine Markdown files: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `quickstart.md`, `contracts/http.md`, `contracts/ui.md`, `checklists/requirements.md` and `checklists/feature.md`. The scope-review approval report referenced by `spec.md:13` is **not included** in the bundle; the independently computed specification hash is the baseline identity used for this review. If a decision dispute arises, consult the authoritative R1 record, rather than treating the author-created checklist as approval evidence.

**Evidence convention:** `research.md:39` means line 39 in `specs/001-personal-task-manager/research.md`. All line numbers refer to the unmodified files inside this ZIP. Design documents may describe planned behaviour but are **not evidence that the behaviour has been implemented or tested**. `tasks.md` is absent by design: task generation belongs after R2/R3.

**Approved Scope boundaries considered:** Private self-registered account; browser-based responsive web app; one private list per account; persisted and cross-device-synchronised saved task data; create/edit/complete/reopen/delete; optional description/priority/date with Medium default; date-only due dates and device-local overdue choice delegated to Design; four sort methods, two filter dimensions and session-restored filters; explicit confirmed deletion; no user-facing restore/reset, no sharing/native apps, and no offline guarantee. In particular, an unresolved failure to load saved tasks must not permit a change capable of overwriting or destroying those tasks (`spec.md:39–46,132–147`).

### Review profile

- **Actors:** first-time registrant; signed-in account owner; another account; unauthenticated/expired-session requester; support operator (contact only, authority not specified).
- **Interfaces:** responsive registration/login/task UI; same-origin JSON API; browser preference storage; database-backed session; support page.
- **Persistent entities:** User, TaskListState, Task and MutationReceipt. Browser preferences are stored separately.
- **Material external/system dependencies:** PostgreSQL, browser cookies and JavaScript, connectivity, localStorage for chosen filter implementation, support destination.
- **Sensitive/irreversible effects:** Account-owned task content; private account sessions; permanent task deletion; uncertain server-committed mutations.
- **Concurrency exposure:** multiple tabs/devices, lost/late responses, optimistic version checks, retried mutation receipts, stale GET snapshots.

## 2. Executive verdict

**NOT_READY**. The design is substantial and coherent over most normal and faulted flows. However, an explicit recovery exception permits a previously uncommitted destructive mutation to execute while saved-task loading is still failing; this conflicts with approved `FR-014`. A second design exception expressly declines to restore saved filters when localStorage is unavailable, although `FR-012` contains no such exception. Both now have approved behavioural resolutions (FD-001 Option A; FD-002 Option A), but the supplied design files have not been updated or verified. These require evidence-backed design amendments before gate clearance. Do **not** silently weaken either guarantee in implementation. A further interrupted-write scenario warrants behavioural clarification but is separated from the proven Scope conflicts.

**Counts:** 16 requirements traced: **13 COMPLETE, 3 PARTIAL, 0 MISSING**. Findings: **2 blocking** (1 HIGH, 1 MEDIUM), **2 non-blocking** (2 MEDIUM). No CRITICAL finding established. FD-001 and FD-002 are IN_PROGRESS (human resolutions agreed; supplied design artifacts unchanged); FD-003 is ACCEPTED_RISK by product owner (risk documentation not yet revised); FD-004 is ACCEPTED_RISK by product owner (no password recovery in first release; documentation not yet revised). Likelihood is UNKNOWN where exposure has not been evidenced; severity does not imply a quantified probability.

## 2A. Review decisions and disposition history

| Date | Finding | Human decision | Current status | Verification requirement |
|---|---|---|---|---|
| 2026-10-22 | FD-001 | **Option A approved.** During unresolved saved-task loading failure, suppress **all new write submissions**, including retries of uncertain operations. Allow read-only loading/retry/support. Once the saved task list successfully loads, the original uncertain operation may be recovered/retried with its **same operation ID and unchanged payload**, subject to normal version and concurrency checks. | **IN_PROGRESS** — decision agreed; **no design artifact change reviewed** | Functional Design owner revises `research.md`, `data-model.md`, `contracts/http.md`, `contracts/ui.md`, and relevant acceptance/recovery scenarios. R2 independently verifies changed design against FD-001 counterexample and cross-flow behavior before `RESOLVED_VERIFIED`. |
| 2026-10-22 | FD-002 | **Option A approved.** Persist status/priority filter preferences against the authenticated user account on the server rather than relying on `localStorage`. Restore across sessions and devices, preserving approved `FR-012` without a Scope exception. | **IN_PROGRESS** — decision agreed; **no design artifact change reviewed** | Functional Design owner revises `research.md`, `data-model.md`, `contracts/http.md`, `contracts/ui.md`, and tests. Specify authoritative preference retrieval/save, cross-device conflict ordering, default on first use, save-failure feedback/retry, and return-session restoration. R2 must inspect and replay tests before `RESOLVED_VERIFIED`. |
| 2026-10-22 | FD-003 | **Option A approved; residual risk explicitly accepted by product owner.** Keep operation identity and pending payload in memory only. A browser reload, close, or interruption may discard the recovery identity; a new user attempt may create a duplicate task or otherwise cause a delayed, unexpected mutation if the first request later commits. No durable pending-operation journal or exactly-once user-intent guarantee is added. | **ACCEPTED_RISK** — product decision captured; **functional documentation and review checks outstanding** | Functional Design owner explicitly documents the interruption boundary, that reload/early snapshot is not proof of rollback, appropriate non-misleading user guidance and re-entry behaviour. R2 checks wording and acceptance scenario R2-T07; acceptance does not mean the underlying risk has been eliminated. |
| 2026-10-22 | FD-004 | **Option A approved; residual risk explicitly accepted by product owner.** No automated or manual password reset/recovery in first release; username/password registration remains, without required email. Support may answer questions but **must not offer or perform credential reset or account transfer**. Disclose the possibility of losing account access if credentials are forgotten. | **ACCEPTED_RISK** — policy settled; **functional documentation and review checks outstanding** | Functional Design owner updates `research.md`, `contracts/ui.md` and appropriate onboarding/support/quickstart flows: clear warning at registration, accurate forgotten-credentials/support wording, no recovery promise, and test R2-T12. Review cannot mark document edits verified until supplied. |

**Interpretation:** For FD-001, a write already submitted before the read failure could still commit; prohibiting *new submissions* does not imply retroactive cancellation. A non-mutating status/receipt lookup may be designed as an optional recovery mechanism, but must not submit a fresh write while loading is unresolved. For FD-002, an account-backed preference design must define the moment a selection counts as saved and communicate failures; do not promise restoration of an uncommitted selection during a server outage. The cross-device conflict rule and failure-retry behaviour are still design details to specify and review, not additional Scope decisions recorded here. The Scope baseline and source ZIP remain unchanged. The R2 gate verdict remains **NOT_READY**. FD-003 and FD-004 have human acceptance of their residual risks, rather than independently verified design changes.

## 3. Requirements coverage matrix

| Req | Outcome required by approved Scope | Design evidence | Coverage | Reviewer note |
|---|---|---|---|---|
| FR-001 | Create active task with only title; validate optional date | `contracts/http.md:78–85`; `data-model.md:26–41`; `contracts/ui.md:13–17` | COMPLETE | Date-only validation and empty title covered. |
| FR-002 | Exactly Low/Medium/High; omitted/cleared defaults Medium | `data-model.md:34`; `contracts/http.md:80–85`; `contracts/ui.md:15–17` | COMPLETE | Default and clearing consistent. |
| FR-003 | Display details/status; absent fields remain absent | `contracts/ui.md:7–11`; `contracts/http.md:49–68` | COMPLETE | UI and response fields observable. |
| FR-004 | Edit details; clear optional attributes | `contracts/http.md:81,85`; `contracts/ui.md:17–19`; `data-model.md:32–39` | COMPLETE | Full replacement with prefilled values specified. |
| FR-005 | Complete/reopen; preserve details | `contracts/http.md:82,100`; `contracts/ui.md:19`; `data-model.md:36` | COMPLETE | Same-state status command increments version, explicitly specified. |
| FR-006 | Delete after confirmation; no in-app restore | `contracts/ui.md:21`; `contracts/http.md:83,100`; `data-model.md:45` | COMPLETE | Standard path clear; protected-state exception separately covered in FR-014. |
| FR-007 | Status AND priority filters | `contracts/http.md:43–45`; `data-model.md:69–71`; `contracts/ui.md:25–31` | COMPLETE | Both criteria and empty results covered. |
| FR-008 | Saved task mutations persist and converge across signed-in devices | `research.md:21–35`; `contracts/http.md:41–123`; `quickstart.md:69–75` | COMPLETE | At-commit acknowledgement, snapshots, versions, receipts and reordering documented; interrupted-page risk in FD-003 is not a proven violation of this saved-data guarantee. |
| FR-009 | Clear failure feedback for writes, loading and sync | `contracts/http.md:104–123`; `contracts/ui.md:35–52`; `research.md:37–43` | PARTIAL | Normal and uncertain errors specified; post-navigation visibility of a still-in-flight prior mutation remains incompletely treated (FD-003). |
| FR-010 | Private account access; deny cross-account data | `research.md:13–19`; `contracts/http.md:3,108–116`; `quickstart.md:59` | COMPLETE | Ownership/opaque 404 and session clearing specified. |
| FR-011 | Date-only past due; active overdue, completed not | `research.md:47–55`; `data-model.md:79–83`; `contracts/ui.md:33` | COMPLETE | Device-local calendar with midnight/focus transitions resolved. |
| FR-012 | Restore most recently chosen status/priority filters between sessions | `research.md:57–63`; `contracts/ui.md:27`; `data-model.md:69–71` | PARTIAL | Explicit no-restoration fallback when storage is unavailable, without Scope-authorised exception (FD-002). |
| FR-013 | Four sorts/both directions; undated last, priority/creation tie rules | `research.md:47–55`; `data-model.md:73–77`; `contracts/ui.md:29`; `quickstart.md:67` | COMPLETE | Secondary order, Unicode title comparison and sort persistence resolved. |
| FR-014 | Failure loading saved data: retry/support; block potentially destructive changes | `research.md:37–43`; `data-model.md:91–104`; `contracts/http.md:119–123`; `contracts/ui.md:42–52` | PARTIAL | Retrying an unknown edit/delete may newly commit during unresolved read failure (FD-001). |
| FR-015 | Desktop/mobile web browsers; no offline/native guarantee | `plan.md:14–32`; `contracts/ui.md:5–11`; `quickstart.md:59` | COMPLETE | Supported families identified; exact release versions appropriately deferred. |
| FR-016 | Uninvited self-registration | `research.md:13–19`; `contracts/http.md:5–17`; `quickstart.md:59` | COMPLETE | Account plus list-state creation atomic; invalid/duplicate input paths specified. |

**Coverage ratings:** COMPLETE = testable, coherent evidence for the approved requirement; PARTIAL = material contract or recovery aspect needs change/decision; MISSING = no relevant behaviour; INSUFFICIENT_EVIDENCE = unavailable artifact prevents assessment. These are **design coverage ratings**, not test pass results.

## 4. Material workflow, transition and fault analysis

| Workflow / challenge | Evidence | Assessment |
|---|---|---|
| Register, invalid/colliding username, login, logout, expired session | `contracts/http.md:5–17,108–113`; `quickstart.md:59` | Covered; FD-004 policy now settled as no first-release password recovery; disclosure remains to be documented. |
| Create title-only or all options; invalid title/date | `contracts/http.md:74–85`; `data-model.md:26–41` | Covered. |
| Edit/clear defaults, cancel edit, complete, reopen, status no-op | `contracts/ui.md:13–23`; `contracts/http.md:80–100` | Covered; status no-op semantics explicit. |
| Confirm/cancel deletion; edit/delete race; removed remote task | `contracts/ui.md:17–23,45–49`; `contracts/http.md:113–121` | Covered on ordinary path; special recovery exception blocks R2 (FD-001). |
| Empty list vs no filter matches vs load failure | `contracts/ui.md:35–50`; `contracts/http.md:70–72` | Covered and distinguishable. |
| Status/priority AND filter; defaults/restore; storage denied | `research.md:57–63`; `contracts/ui.md:27` | Restore exception conflicts with FR-012 (FD-002). |
| Four sort directions, ties, undated, Unicode titles | `research.md:47–55`; `data-model.md:73–77` | Covered with deterministic order. |
| Local midnight / timezone change; past-date reopen | `data-model.md:79–83`; `quickstart.md:65` | Covered. |
| Cross-device snapshot consistency, stale GET, rapidly changed query | `research.md:21–27`; `contracts/http.md:41–72`; `quickstart.md:69–75` | Covered; revision/query/session generation guard specified. |
| Same-version concurrent edits, stale deletion | `research.md:29–35`; `contracts/http.md:108–121` | Covered; version-conflict review and no silent merge. |
| Duplicate operation replay, lost create response, later remote deletion | `data-model.md:47–65`; `contracts/http.md:100–123` | Covered as long as operation ID/payload survive. |
| Unknown mutation followed by page reload, closure or sign-out | `research.md:39–41`; `contracts/ui.md:56–58` | Accepted residual interruption risk; documentation pending (FD-003). |
| Failed initial/refresh load, retry/support, blocked writes | `contracts/ui.md:35–52`; `data-model.md:91–104` | Largely covered; read-failure + uncertain-mutation retry conflicts with Scope (FD-001). |
| Failed database migration, backups, authentication throttle, supported browser releases | `plan.md:94–96`; `quickstart.md:77–81` | Downstream technical/release verification dependencies; not evidenced as executed. |

### Key state and invariant results

1. **Task lifecycle:** active → completed → active; deletion is terminal. Creating a task starts at active, version 1; accepted edit/status increments task version and list revision. Detailed fields remain unchanged on reopen. Evidence `data-model.md:26–45`; `contracts/http.md:78–102`.
2. **Ownership:** session-derived owner for reads, receipts, mutations and list revisions. No owner ID from request bodies; foreign/unknown task identity treated alike. Evidence `contracts/http.md:3,25,106–121`.
3. **Idempotency:** same key and canonical payload replays without extra effect, even when the referenced task was subsequently deleted; different digest with same key rejects. Evidence `data-model.md:47–63`; `contracts/http.md:74–123`.
4. **Snapshot order/freshness:** account revision serialises task changes; query/session generations reject stale responses. Successful *empty* result is distinct from failed load. Evidence `research.md:21–27`; `contracts/http.md:41–72`.
5. **Protection invariant — exception detected:** a retry of an uncommitted edit/delete is still a mutation and cannot be assumed read-only merely because it reuses an operation ID. Evidence `spec.md:145`; `contracts/http.md:123`.
6. **Persistent preference invariant — exception detected:** a UI warning does not fulfil an unconditional cross-session restoration requirement. Evidence `spec.md:143`; `research.md:59`.

## 5. Findings register — blockers

### FD-001 — Retrying an uncertain mutation can breach the saved-data load-failure lock

- **Category:** FUNCTIONAL_DESIGN_DEFECT (approved-Scope conflict)
- **Severity:** HIGH; **Likelihood:** UNKNOWN; **Confidence:** HIGH; **Blocking:** YES; **Status:** IN_PROGRESS — resolution approved; awaiting artifact amendments and independent verification
- **Requirement/flow:** `FR-014`, `FR-006`, `FR-008`, `FR-009`; unknown write outcome + saved-data read failure + retry.
- **Evidence:** Approved prohibition `spec.md:43,145`; explicit mutation retry exception `contracts/http.md:119–123`; `research.md:37–41`; UI `contracts/ui.md:42–52`.
- **Concrete counterexample:** User explicitly confirms deletion. First POST fails without a commit but the response is lost. Refresh fails so saved tasks cannot be loaded. Client is in an unknown-result state. User selects Retry; server receives the same ID for the first time as a successfully committing operation and deletes the task even though read failure remains unresolved.
- **Expected from approved Scope:** During unresolved failure to load saved tasks, task changes capable of overwriting or destroying saved data remain blocked; prior operation outcome may be reconciled only without violating that protection.
- **Specified:** `contracts/http.md:123` allows a retry that may perform the original edit/delete, calling it the sole exception to the UI lock.
- **Consequence:** Permanent removal or overwritten details during the very failure condition that Scope requires the app to protect against; could be invisible without a valid snapshot.
- **Remediation/owner:** Functional Design should distinguish **read-only outcome determination** from a retry capable of first-time mutation, and define a recovery transition that respects the protected loading state. Technical Design can evaluate how to obtain operation status. If the team wants an exception to FR-014, it must be routed to the authorised Scope owner for an explicit Scope decision **before** relying on it. No implementation-only workaround or silent Scope amendment.
- **Reverification:** Simulate lost response after a precommit rollback + failed snapshot + retry intent. Assert no new destructive commit while loading remains unresolved, and confirm eventually recoverable state is defined.

### FD-002 — Filter restoration fallback weakens the unconditional approved requirement

- **Category:** FUNCTIONAL_DESIGN_DEFECT (requirement exception not approved)
- **Severity:** MEDIUM; **Likelihood:** UNKNOWN; **Confidence:** HIGH; **Blocking:** YES; **Status:** IN_PROGRESS — Option A approved; awaiting revised design and independent verification
- **Requirement/flow:** `FR-012`, `AC-005`; selection, browser storage unavailability and new session.
- **Evidence:** `spec.md:143,162,186`; `research.md:57–63` (warn and continue without restoration if localStorage unavailable); `contracts/ui.md:25–27`; `data-model.md:69–71`.
- **Concrete counterexample:** A signed-in user selects status `Completed` and priority `High` while browser preference storage is unavailable or rejects writes. The UI continues the current session but cannot persist the selections; in the next session it falls back to `All/All`.
- **Expected from approved Scope:** Most recently selected filters are restored on a later session. No localStorage exception is approved by the specification.
- **Specified:** Notify the user that restoration is unavailable and use session-only filtering.
- **Consequence:** Returning user's list appears under different filters than last selected, potentially hiding that a restoration guarantee was not met even if a warning was shown earlier.
- **Approved resolution/owner:** **Option A (user-approved 2026-10-22):** Functional Design must replace browser-only filter persistence with authenticated **account-backed server persistence**, restoring the most recently successfully persisted status/priority selections on return across sessions/devices, without changing `FR-012`. Define read/write contracts, first-use defaults, conflicting changes from multiple devices, save confirmation/failure feedback and retry, and safe behaviour while the server is unavailable. Do not represent an unsuccessful save as restored preference state. Technical Design owns mechanisms, not the product rule.
- **Reverification:** Simulate blocked/unavailable `localStorage` with working account services: change filters, confirm persisted response, sign out/in or open another device and verify the saved selections restore. Simulate preference-save failure and conflicting device edits; verify the specified confirmation, precedence and retry/error semantics. Do not close based on the approved choice alone.

## 6. Findings register — non-blocking, decisions and improvements

### FD-003 — Pending mutation identity is lost on interruption before a definitive outcome

- **Category:** FUNCTIONAL_DESIGN_GAP (interruption recovery boundary)
- **Severity:** MEDIUM; **Likelihood:** UNKNOWN; **Confidence:** MEDIUM; **Blocking:** NO; **Status:** ACCEPTED_RISK — user-approved Option A 2026-10-22; documentation updates not yet verified
- **Requirement/flow:** `FR-009`; transport-unknown create/edit/delete + reload/close/sign-out.
- **Evidence:** `research.md:29–41` stores exact operation only in memory and permits unsaved draft loss on reload; `contracts/ui.md:46,56–58`; `contracts/http.md:76,119–123` requires the same ID/payload for safe retry.
- **Counterexample:** A create request continues processing when the browser reloads. The new page fetches a snapshot *before* that create commits, showing no task. The user repeats the intended create as a new operation. The first request later commits, leaving two separately identified tasks. Reusing the original operation ID is no longer possible from the reloaded page.
- **Expected vs specified:** The approved Scope does not expressly require exactly-once intent across browser interruption. The design specifies safe replay only while pending data remains in memory, but does not explicitly classify or explain this post-reload overlap and its consequence.
- **Consequence:** Possible unintended duplicate tasks or delayed changes without an opportunity to replay/settle the exact prior key after navigation. No evidence of corruption from the same operation ID has been established.
- **Approved risk disposition/owner:** **Option A approved by product owner on 2026-10-22.** Retain in-memory-only pending operation identity; no durable interruption recovery or exactly-once user-intent guarantee. After reload/close/sign-out, an in-flight operation may still commit; if the user repeats an absent-looking create, duplicates may result (and edits/deletions may finish unexpectedly). Functional Design owner must document this limitation, accurate outcome feedback, and safe repeat guidance; no false assurance that a fresh snapshot means a prior pending write rolled back. This is a consciously accepted residual risk, **not** a claim that duplicate intent is prevented. No Scope modification authorised.
- **Reverification:** Delay a first create commit past reload/first snapshot, simulate a repeat create, and verify documented behaviour and risk disposition.

### FD-004 — Password-loss policy: no account recovery in the initial release (decision approved)

- **Category:** SCOPE_ESCALATION (optional recovery policy clarification, now decided without altering approved Scope)
- **Severity:** MEDIUM; **Likelihood:** UNKNOWN; **Confidence:** HIGH for original missing policy, not for an actual defect; **Blocking:** NO; **Status:** ACCEPTED_RISK — user-approved Option A 2026-10-22; design/documentation edits not yet independently verified
- **Requirement/flow:** `FR-010`, `FR-016`; returning signed-in access and forgotten credentials.
- **Evidence:** `research.md:13–19` specifies no email, username/password credentials, support link on forgetting credentials and no automated reset; `contracts/http.md:11–17`; `contracts/ui.md:7`.
- **Counterexample:** An account owner forgets the password. The design directs them to support but supplies no verified ownership channel or defined criterion under which support could restore access. An attacker could also claim ownership; granting a reset without verification would risk access to private tasks.
- **Expected vs specified:** Scope requires account registration/sign-in and cross-session access by users who can authenticate, but does **not** require password recovery. Absence of recovery is therefore **not** a proven Scope violation. Previously, the support referral left a policy ambiguity; the product owner has now resolved it.
- **Consequence / accepted residual risk:** A user who loses their credentials may permanently lose access to their saved tasks. No unauthorised support reset or transfer is permitted. This risk is knowingly accepted for the initial release; the decision does not imply that saved tasks are deleted.
- **Approved decision/owner:** **Product owner approved Option A on 2026-10-22.** Username/password self-registration remains; email is not required; **no automated or manual forgotten-password recovery** is provided, including through support. Functional Design owner must state the limitation during registration, on the forgotten-credentials route and in support guidance, without implying a reset is available. No change to the approved Scope is needed. Any later addition of recovery requires a separately authorised Scope decision and security/privacy review.
- **Reverification:** Inspect revised `research.md`, `contracts/ui.md` and relevant quickstart/support copy. Test that registration prominently communicates the consequence, forgotten-password guidance does not promise recovery, support cannot reset or transfer account ownership, and normal username/password sign-in continues to work.

**Non-finding observations:**

- Case-insensitive username *registration collision* and case-sensitive login are both intentionally specified (`research.md:15`); user guidance on exact username case could reduce confusion but the two rules do not by themselves contradict approved Scope.
- Complete per-account snapshots with 15-second visible polling and no task count limit imply scaling/performance questions (`plan.md:28–32`); no numeric Scope goal authorises declaring a design defect solely from those choices. Measure and stress-test at Technical Design/Test gates.
- The evidence bundle does not contain `scope_review_report.md` or the constitution used in `plan.md:34–47`; the baseline hash is verified, but source approval/constitutional audit evidence should accompany later integration review if required. This is not treated as another R2 functional blocker.

## 7. Representative observable acceptance/recovery tests

These are **proposed acceptance scenarios**, not executed tests. Retain separate explicit test cases for each action (create/edit/status/delete) where a scenario speaks about generic writes.

| Test ID | Given | When | Then |
|---|---|---|---|
| R2-T01 | Signed-in user with an empty list | Create with valid title and no optional details | Active task saved; Medium priority; description and due date null; visible on later session/device. |
| R2-T02 | Active task with past date | Complete, then reopen on same local calendar day | Details unchanged; not overdue while completed; overdue after reopen. |
| R2-T03 | Two clients see same version | Both edit and save | First commit acknowledged; second gets version conflict, draft retained, no silent overwrite. |
| R2-T04 | A create commits but response is lost | Another device deletes that task; first client retries exact original request | Receipt replay reports historical success; deleted task remains absent, with no resurrection. |
| **R2-T05** | User confirms delete; first attempt uncommitted/unknown; refreshed saved data cannot load | User invokes the allowed uncertain-operation Retry | **No first-time destructive commit while unresolved load failure exists; outcome-resolution path specified.** FD-001 gate test. |
| **R2-T06** | Browser `localStorage` is blocked; authenticated preference service is available | User selects Completed/High, receives save confirmation, and starts a new session or opens another signed-in device | **Completed/High restore from the account-backed saved state;** confirm defined behaviour when saving fails or two devices change filters concurrently. FD-002 gate test. |
| R2-T07 | Create sent with lost acknowledgement | Browser reloads before original request commits; new load shows old state | App behaviour and remaining potential for duplicate user intent are explicit; acceptable recovery guidance/risk disposition recorded. FD-003. |
| R2-T08 | Saved list fails to load, then fails again | User selects Retry while offline/unavailable | No destructive writes; retry/support cues; never replace saved tasks with empty list. |
| R2-T09 | An old GET is delayed, a newer task change commits | Both GET/ACK responses arrive out of order | Old response cannot replace the newer accepted snapshot. |
| R2-T10 | Account A and account B exist | B requests A's task by known UUID or an API request survives logout | No task disclosure or write; account/session generation prevents stale display. |
| R2-T11 | Due-date ties and undated tasks exist | Reverse Due Date sort | Undated tasks remain last; same-date priority and creation tie rules remain fixed. |
| R2-T12 | User registers a username-only account after seeing the recovery limitation, then forgets the password | Opens forgotten-credentials/support route | No reset/transfer is offered or performed, and the UI/support text clearly states that forgotten-password recovery is unavailable and account access may be permanently lost. |

## 8. Decisions, downstream ownership and next actions

**Must resolve before R2 human approval:**

1. **FD-001 — Functional Design owner**, with Product/Scope authority for any attempted exception: reconcile uncertain-operation retries with the strict saved-load-failure mutation prohibition. Update `research.md`, `data-model.md`, `contracts/http.md`, `contracts/ui.md` and relevant quickstart scenarios consistently; avoid changing Scope without authorised decision.
2. **FD-002 — Functional Design owner:** implement the approved design choice in documents: persist filter preferences to authenticated account-backed server state and restore them on return; define consistency and failure behaviour, API/UI contracts, and observable tests. No Scope relaxation is approved.

**Should resolve or explicitly record as accepted non-blocking risks:**

3. **FD-003 — Functional Design (accepted risk):** user approved Option A; document interrupted pending operation and re-entry boundaries, including late commits, repeat-intent duplication and careful user-facing guidance. R2 should inspect documentation, but accepted risk need not be engineered away.
4. **FD-004 — Functional Design (accepted risk):** product owner approved Option A: no password recovery, including through support, for the initial release. Amend registration and forgotten-credential/support messaging to make the irreversible access risk clear, and verify R2-T12. No Scope amendment is authorised or required.

**For R3 Technical Design / later Test / Deploy:** Verify transaction and receipt atomicity under real PostgreSQL concurrency; owner-scoped request queries/permissions; stale reply dismissal across tab changes; trusted CSRF/error envelopes; production secure cookie/cache settings; failure injection; migrations and database restore process; auth rate limiting; real-device/browser compatibility; configured support contact. These are dependencies, **not** proof of implementation failure.

**Re-review protocol:** Keep this review ID and finding IDs stable. Authors should supply changed artifact paths, exact revisions/diffs and claimed dispositions. R2 independently checks the actual updated contract and counterexamples, including any cross-document regressions. Status can move from OPEN to RESOLVED_VERIFIED only after evidence is inspected; ACCEPTED_RISK requires an authorised human, and any Scope guarantee change must return through Scope governance. Neither author assertions nor checked boxes close a finding.

## 9. Exit-gate assessment

| Gate criterion | Assessment |
|---|---|
| Approved Scope baseline identified | PASS — matching SHA-256; approval record itself outside bundle |
| All approved FRs mapped to design evidence | PASS — 16/16 traced |
| Primary workflows and lifecycle transitions sufficiently specified | PASS, subject to specific exception |
| Material failure/retry/concurrency/recovery coherent with Scope | **FAIL — FD-001** |
| Scope guarantees preserved in exceptional browser conditions | **FAIL — FD-002** |
| Material outcomes and risks testable | PARTIAL — FD-001/002 gate cases await design changes; FD-003/004 residual risks accepted, documentation updates still pending |
| Implementation/tests executed | NOT_APPLICABLE — design-stage static review |
| Human approval | NOT_GRANTED — decision must occur after independent verification |

**Formal R2 verdict: NOT_READY.** Do not declare `READY_FOR_HUMAN_APPROVAL` until blocking findings are resolved by changed design evidence or appropriately authorised Scope decisions; then re-run the affected scenario and traceability review. Nothing in this R2 report changes the approved Scope or prescribes architecture.
