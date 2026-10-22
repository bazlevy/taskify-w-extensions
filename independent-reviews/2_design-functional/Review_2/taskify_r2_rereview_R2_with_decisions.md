# Taskify / Personal Task Manager — Independent R2 Functional Design Re-review

**Review ID:** R2-R2 (iteration 2 of initial R2-R1)  
**Review date:** 2026-10-22 (Australia/Melbourne)  
**Reviewer:** Independent Functional Design Reviewer (R2)  
**Formal gate verdict:** **READY_FOR_HUMAN_APPROVAL** — not automatically approved; authorised human must still explicitly grant R2 approval.  
**Review type:** Static comparison, scenario replay and cross-document requirements analysis. **No application or automated tests executed.**
**Post-review human decision (2026-10-22):** R2 human approval **WITHHELD** until FD-005 and the three editorial corrections are amended and independently verified. The formal gate verdict below records the historical R2-R2 assessment, not approval.


## 1. Baseline, evidence and independence

- **Approved Scope:** `specs/001-personal-task-manager/spec.md`, SHA-256 `4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`. Identical in original and revised ZIP; matches previously confirmed approved baseline. The R1 scope approval report itself is not in either design bundle.
- **Original design:** `/mnt/data/specs.zip`, ZIP SHA-256 `43efe6e57bd587d233bf2d0f45a7fc6cc660bc509341008c370e79f3c8e6ec2f`.
- **Revised design:** `/mnt/data/specs(1).zip`, ZIP SHA-256 `202e72e8518164cd23f7dc324a1e0847681ac11af5ee2eff51bab676e39f046e`.
- **Initial findings baseline:** `/mnt/data/taskify_r2_functional_design_review.md`, review R2-R1; four findings FD-001–FD-004 and 12 representative R2 tests. Historical human decisions have been retained unchanged.
- **Inventory:** 9 Markdown documents in both bundles. Six changed: `research.md`, `data-model.md`, `contracts/http.md`, `contracts/ui.md`, `quickstart.md`, `plan.md`. Three byte-identical: `spec.md` and both files under `checklists/`. `tasks.md` was not supplied and is expected later in the process.
- **Evidence convention:** Paths and line numbers below refer to files in **revised** `specs/001-personal-task-manager/`. The submitted Codex completion note was treated solely as an author claim and was checked against the documents; checklists and author assurances are not independent evidence of implementation.

**Profile:** Authenticated browser-based single-account task list; Django session-backed pages and same-origin API; PostgreSQL persistence; task mutations, confirmation of permanent deletion, account-owned filter preferences, multiple tabs/devices, potentially lost HTTP responses, browser interruptions, account isolation. No offline queue, task restore, account transfer or password recovery. No application code was included; technical feasibility and test execution belong to later gates.

## 2. Executive result and audit trail

**Result:** The original two functional-design blockers are **RESOLVED_VERIFIED**. The two explicitly accepted residual risks have now been documented and checked; their dispositions remain **ACCEPTED_RISK**, not “risk eliminated”. All 16 approved FRs have sufficiently specified functional behaviour for R2. One **new low-severity non-blocking improvement** (FD-005) corrects overconfident preference-save error messaging. Minor stale editorial references are also noted. There is no verified remaining R2 blocker.

| Iteration | Evidence | Original blockers | Accepted risks | Additional issue | Gate |
|---|---|---|---|---|---|
| R2-R1, 2026-10-22 | `specs.zip` | FD-001, FD-002 OPEN then IN_PROGRESS after decisions | FD-003 and FD-004 approved, documentation pending | — | NOT_READY |
| **R2-R2, 2026-10-22** | `specs(1).zip` | **FD-001, FD-002 RESOLVED_VERIFIED** | **FD-003, FD-004 ACCEPTED_RISK; documentation verified** | **FD-005 OPEN, LOW, non-blocking** | **READY_FOR_HUMAN_APPROVAL** |

**Change control:** Scope stayed byte-identical; no original Scope guarantee was relaxed and no unapproved capability was added. This independent review does not change the submitted design documents or the earlier R2-R1 report.

## 3. Original findings: evidence-based re-verification

### FD-001 — Uncertain operation retry during unresolved saved-task load failure

- **Original:** FUNCTIONAL_DESIGN_DEFECT; HIGH; likelihood UNKNOWN; blocking YES; formerly IN_PROGRESS.
- **Verified changed evidence:** `research.md:31,39`; `contracts/http.md:146,150–154`; `contracts/ui.md:61–70`; `data-model.md:118–130`; `plan.md:32–35`; `quickstart.md:71,73`.
- **Replayed counterexample:** Confirm a delete; initial POST did not commit and response is lost; subsequent saved-task snapshot fails. The new contract **prohibits any task mutation submission**, including retry of the original delete, until a valid saved-task snapshot is loaded. After recovery, only the original ID and payload may be retried; server receipts, ownership and version checks still apply. A request sent *before* the read failure may still commit, and the design does not promise retroactive cancellation.
- **Assessment:** The prior exception in `contracts/http.md` is gone. The UI, state model, acceptance scenario and main contract agree on the blocking transition and unlocking condition; `FR-014` is preserved.
- **Current disposition:** **RESOLVED_VERIFIED** (design artifacts only). **Owner:** Functional Design; downstream implementation/tests: Build and Test. **Likelihood:** UNKNOWN; **confidence:** HIGH. **R2-T05:** design-specified; not executed.

### FD-002 — Saved filter preferences must restore between sessions

- **Original:** FUNCTIONAL_DESIGN_DEFECT; MEDIUM; likelihood UNKNOWN; blocking YES; formerly IN_PROGRESS.
- **Verified changed evidence:** `research.md:57–63`; `data-model.md:13–15,28–43,88–94`; `contracts/http.md:79–99`; `contracts/ui.md:33–42`; `plan.md:20–22,35–36`; `quickstart.md:67`.
- **Replayed counterexample:** Browser `localStorage` rejects writes, but account services work. A filter selection accepted by `PUT /api/preferences/filters/` is saved to the account's single versioned record, and a later session/device fetches that pair by `GET /api/preferences/filters/` before requesting its initial filtered task snapshot. Browser storage is no longer a dependency for restoration.
- **Concurrency and failure review:** Positive expected-version check serialises concurrent server updates; 409 returns the authoritative pair/version without overwriting it. Client retains a local attempted selection, must say it is **unconfirmed** on transport failure and must obtain user approval before overriding a genuinely different authoritative pair. Versioned server state is independent of task data/revision. Cross-device retrieval, no-silent-overwrite conflict and negative save paths are represented by validation scenarios.
- **Assessment:** The original unapproved browser-storage fallback is removed; first-use defaults, server ownership, cross-session/device restoration and conflict control are specified. The *additional* message-precision issue for uncertain preference saves is recorded separately as FD-005, not concealed as a closure condition for the original defect.
- **Current disposition:** **RESOLVED_VERIFIED** (design artifacts only). **Owner:** Functional Design; subsequent R3 technical validation of locking and privacy; Test to execute cases. **Likelihood:** UNKNOWN; **confidence:** HIGH. **R2-T06:** design-specified; not executed.

### FD-003 — Browser interruption loses in-memory operation identity

- **Original:** FUNCTIONAL_DESIGN_GAP; MEDIUM; likelihood UNKNOWN; blocking NO; user approved Option A.
- **Verified changed evidence:** `research.md:39–41`; `contracts/ui.md:76–82`; `data-model.md:122–130`; `plan.md:37–39`; `quickstart.md:73`.
- **Replayed counterexample:** Pending create sent, browser reloads, an early snapshot does not show it, user recreates task; original create may later commit. The design now explicitly says pending identity exists only in memory, an early read is not proof of rollback, and repeated intent can create duplicates or delayed changes. User-facing guidance recommends confirming outcomes and reviewing the current list before re-entry; it does **not** promise exactly-once intent.
- **Assessment and risk:** **ACCEPTED_RISK** (authorised product-owner decision from R2-R1); required explanatory design edits **VERIFIED**. This is not a solved reliability guarantee. **R2-T07:** design-specified, not executed. **Confidence:** HIGH for documentation; likelihood of residual risk UNKNOWN.

### FD-004 — No forgotten-password recovery in initial release

- **Original:** SCOPE_ESCALATION / clarified product policy; MEDIUM; likelihood UNKNOWN; blocking NO; user approved Option A.
- **Verified changed evidence:** `research.md:15–19`; `contracts/http.md:7–22`; `contracts/ui.md:7–11,84–86`; `plan.md:35–38`; `quickstart.md:59`.
- **Replayed counterexample:** Registered user forgets username/password and contacts support. Registration explicitly discloses risk of permanent loss of access; login/help/support do not offer automated/manual reset, identity recovery or account transfer. Support answers queries only. No email verification channel is invented.
- **Assessment and risk:** **ACCEPTED_RISK** (authorised product-owner decision from R2-R1); required registration/login/help/support design edits **VERIFIED**. No Scope amendment required. **R2-T12:** design-specified, not executed. **Confidence:** HIGH for documentation; likelihood UNKNOWN.

## 4. Requirements-to-design coverage

COMPLETE = approved outcome and significant normal/faulted behaviour specified to a testable level. This is not implementation certification. No invented scope additions were required.

| FR | Coverage | Key revised design evidence | Assessment |
|---|---|---|---|
| FR-001 | COMPLETE | `contracts/http.md:103–112`; `contracts/ui.md:17–21` | Title-only task creation, validation |
| FR-002 | COMPLETE | `data-model.md:51–60`; `contracts/http.md:107–112` | Priority enum/default/clearing |
| FR-003 | COMPLETE | `contracts/http.md:54–72`; `contracts/ui.md:13–15`; `data-model.md:49–60` | Required title/priority/status and optional description/due-date display without invented placeholders |
| FR-004 | COMPLETE | `contracts/http.md:108,127`; `contracts/ui.md:21` | Edit and optional-value clearing |
| FR-005 | COMPLETE | `contracts/http.md:109,127`; `contracts/ui.md:23` | Complete/reopen status without altering details |
| FR-006 | COMPLETE | `contracts/http.md:110,143`; `contracts/ui.md:25` | Explicit permanent delete confirmation and stale-version check |
| FR-007 | COMPLETE | `contracts/http.md:48–50`; `data-model.md:88–94` | Status/priority AND filter |
| FR-008 | COMPLETE | `research.md:21–35`; `data-model.md:17–26,66–82` | Commit/persist/synchronise, revision/receipt concurrency |
| FR-009 | COMPLETE | `research.md:37–45`; `contracts/ui.md:50–70` | Loading, validation, write and sync failures, uncertainty |
| FR-010 | COMPLETE | `research.md:13–19`; `contracts/http.md:5–26` | Self-registration, private auth, account isolation |
| FR-011 | COMPLETE | `data-model.md:102–106`; `contracts/ui.md:48` | Local-calendar overdue and completed/reopened behaviour |
| FR-012 | COMPLETE | `research.md:57–63`; `contracts/http.md:79–99`; `contracts/ui.md:35–42` | Account-backed restored filter choices, failure and conflicts |
| FR-013 | COMPLETE | `research.md:47–55`; `data-model.md:96–100` | Four bidirectional sorts, deterministic ties and undated-last |
| FR-014 | COMPLETE | `research.md:39–41`; `contracts/http.md:146–155`; `data-model.md:118–130` | Failed read write-lock and safe post-read replay |
| FR-015 | COMPLETE | `plan.md:24–28`; `contracts/ui.md:13–15` | Responsive web, browser/accessibility interactions |
| FR-016 | COMPLETE | `contracts/http.md:7–22`; `contracts/ui.md:7–11` | Private uninvited signup and returning access; recovery exclusion disclosed |

**Total: 16 COMPLETE; 0 PARTIAL; 0 MISSING; 0 INSUFFICIENT_EVIDENCE** for the reviewed specification and provided design. External approval provenance is known by baseline hash but the R1 report is not bundled.

## 5. Workflow, state, failure and consequence coverage

| Material flow / failure | Checked design evidence | R2 assessment |
|---|---|---|
| Register → authenticate → logout → resume | `contracts/http.md:5–26`; `contracts/ui.md:7–15,65`; `quickstart.md:59` | COVERED; no-recovery warning specified |
| Create/edit/complete/reopen/confirm delete | `contracts/http.md:101–148`; `contracts/ui.md:17–31`; `quickstart.md:61–63` | COVERED; receipts and versions constrain repeats/races |
| Sort, filter, overdue, empty results | `data-model.md:88–106`; `contracts/ui.md:33–56` | COVERED; scope tie rules maintained |
| Session-return/device-return filter restoration | `contracts/http.md:79–99`; `contracts/ui.md:33–42`; `quickstart.md:67` | COVERED; preference read precedes task snapshot |
| Concurrent filter updates | `data-model.md:28–43`; `contracts/http.md:91–99`; `quickstart.md:67` | COVERED at business-rule level; see FD-005 for message precision |
| Initial/repeated saved-task read failure | `research.md:37–43`; `contracts/ui.md:54–70`; `quickstart.md:71` | COVERED; writes locked and support guidance |
| Uncertain task mutation followed by read failure | `contracts/http.md:146,150–154`; `data-model.md:122–130`; `quickstart.md:73` | COVERED; no new write before valid snapshot |
| Late/duplicate/out-of-order requests and device conflict | `contracts/http.md:127–148`; `research.md:21–35`; `quickstart.md:73–75` | COVERED; no replay resurrection, optimistic versions and staleness control |
| Browser interruption after unresolved mutation | `contracts/ui.md:76–82`; `research.md:41`; `quickstart.md:73` | ACCEPTED_RISK; warned and testable |
| Credential loss / support contact | `contracts/ui.md:7–11,84–86`; `contracts/http.md:15–22` | ACCEPTED_RISK; no reset permitted |
| Security/account isolation on stale responses | `contracts/http.md:3,26,77,133–148`; `contracts/ui.md:72–77` | COVERED functionally; technical verification deferred to R3/Test |

**Representative R2 scenarios:** T01–T12 are derivable from revised contracts and quickstart. T05, T06, T07 and T12 were explicitly replayed against the documents; no test was run against a live application. `quickstart.md:73` accidentally uses the label **R2-T07 / FD-001**; the original gate scenario for FD-001 is **R2-T05**, whereas **R2-T07** belongs to FD-003. This is an editorial traceability correction, not a behavioural defect.

## 6. New non-blocking finding and editorial corrections

### FD-005 — Preference-save uncertainty can be represented as definite non-persistence

- **Category:** NON_BLOCKING_IMPROVEMENT (cross-contract failure-feedback consistency); **Severity:** LOW; **Likelihood:** UNKNOWN; **Confidence:** HIGH; **Blocking:** NO; **Status:** OPEN; **Owner:** Functional Design.
- **Requirement/flow:** FR-012; preferences `PUT` → saved on server → lost response / same-tab concurrent filter changes; visible saved-state cue.
- **Evidence:** `contracts/http.md:91–99` correctly calls a transport/5xx preference save **unconfirmed**. `contracts/ui.md:38–42` says on save failure the app shows it **was not saved** for later sessions/devices. `quickstart.md:67` expects feedback saying **it will not restore**. These stronger claims are unjustified if the server has committed but the acknowledgement was lost. The UI also attributes a version conflict to “another device”, although same-tab overlapping updates or delayed acknowledgements could be responsible.
- **Counterexample:** User selects Completed + High; preference `PUT` commits version 2, but response is lost. The server will restore Completed + High later; UI/test copy nevertheless says the selection **was not saved**. Users may receive false failure/recovery guidance; no evidence of task corruption and version guard prevents a blind write over a known different version.
- **Expected:** Differentiate **confirmed rejection** from **unknown save outcome**; say “Could not confirm your filters were saved”, not “They were not saved.” On reread, compare the authoritative saved pair and version before presenting re-save or overwrite; don't assume the cause of a 409. Define how later preference-save acknowledgements are associated with the current selection when controls change rapidly. Supply a small response-loss and conflicting-update acceptance case.
- **Disposition:** **OPEN / NON-BLOCKING.** Correction is recommended before implementation/test-authoring and can be validated at R4 integrated design review; current issue does not defeat core persisted task guarantees or the new versioned preference storage design. This is **not** a claim the issue was fixed.

**Editorial consistency notes (not separate findings):**

1. `data-model.md:9` says `public_id` exists “for browser preference namespacing”, although filter preferences are now server-backed and browser storage is excluded. The public account ID still has other uses (snapshot account identity), so update the purpose rather than remove the field without analysis.
2. `checklists/feature.md:63,70` still refer to retained browser preferences/preference-storage fallback; update the prompts to account-backed preference failures and conflicts. Unchecked checklist items were never counted as evidence of coverage.
3. `quickstart.md:73` has the FD-001 test-ID typo described above; correct `R2-T07 / FD-001` to `R2-T05 / FD-001`.

## 7. Gate, decisions and handoff

| Exit criterion | Assessment |
|---|---|
| Authoritative approved Scope baseline intact | **PASS** — byte-identical specification/hash |
| Requirements traced with testable design outcomes | **PASS** — 16/16 COMPLETE |
| User/system/API flows, states, invariants, failures and recovery | **PASS** for material R2 behaviours; accepted interruption/recovery exclusions transparent |
| FD-001 strict saved-data write lock | **PASS** — RESOLVED_VERIFIED |
| FD-002 account-backed filter restoration and concurrency | **PASS** — RESOLVED_VERIFIED; low-severity message ambiguity tracked as FD-005 |
| FD-003/FD-004 accepted residual-risk documentation | **PASS** — evidence verified; risks remain ACCEPTED_RISK |
| Unresolved CRITICAL/HIGH or material design blockers | **PASS** — none evidenced; FD-005 LOW non-blocking |
| Implementation/automated acceptance tests | **NOT_APPLICABLE** at R2; not executed |
| Human R2 approval | **PENDING** — cannot be inferred from review verdict |

**Independent gate verdict: READY_FOR_HUMAN_APPROVAL.** This is a recommendation to the authorised product owner/human gate, **not** an approval event and **not** permission to skip R3 or subsequent workflow gates.

**Recommended non-blocking follow-up:** Functional Design owner corrects FD-005 and editorial items before R4 / test case generation; R4 independently checks contract alignment and does not silently mark it resolved. **Downstream R3 Technical Design:** transactional idempotency and row locks in PostgreSQL, preference-version race handling, session/CSRF/ownership isolation, storage failure handling, security and accessibility. **Downstream Test:** execute browser/API/integration scenarios including lost response after commit, preference GET/save outages, rapid repeated selections, cross-device conflicts, task read-failure lock and accepted interruption case. **Human product owner:** formally grants/withholds R2 gate approval and retains FD-003/FD-004 accepted risk dispositions.

**No source ZIP, approved Scope, review R2-R1, or project repository was altered during this review.**

---

## 8. Post-review owner decision and verification queue (2026-10-22)

**Owner decision:** FD-005 recommended correction **APPROVED**. Owner requests **all four corrections** (FD-005 plus three editorial items) be incorporated and independently verified **before human R2 approval**. The design ZIPs were not amended as part of this decision.

**Current human gate state:** **APPROVAL WITHHELD**. The R2-R2 `READY_FOR_HUMAN_APPROVAL` verdict remains a historical independent readiness assessment; it is not a completed gate approval and does not override the owner's stricter condition.

| Item | Required amendment | Current status |
|---|---|---|
| FD-005 | Resolve preference PUT transport/5xx uncertainty across `contracts/http.md:91–99`, `contracts/ui.md:35–42`, `research.md:59` and `quickstart.md:67`. Distinguish confirmed 409 rejection from response-loss uncertainty; use accurate unconfirmed wording, re-read authoritative preference pair/version before a potential overwrite, remove assumed origin of conflicts, and specify how rapid local changes/out-of-order acknowledgements affect confirmed saved state. Add acceptance cases. | **IN_PROGRESS** — decision agreed; design amendment and re-review pending |
| Editorial 1 | `data-model.md:9`: replace `public_id` description “browser preference namespacing” with genuine opaque account identifier purpose consistent with task snapshot `account_id` in `contracts/http.md:56`; preserve field absent design evidence for removal. | **OPEN** |
| Editorial 2 | `checklists/feature.md:63` (CHK027) and `:70` (CHK031): replace old browser-preference references with account-owned preference privacy and server failure/conflict assumptions. | **OPEN** |
| Editorial 3 | `quickstart.md:73`: replace `R2-T07 / FD-001` with `R2-T05 / FD-001`; keep `R2-T07 / FD-003`. | **OPEN** |

**Independent re-review exit conditions:** receive revised complete `specs/` package; verify `spec.md` remains byte-identical to approved SHA-256 `4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`; verify all four corrections against source text and replay FD-005 cases (committed preference PUT with lost response, confirmed 409, quick successive changes, out-of-order responses and failed authoritative GET). Preserve FD-001/FD-002 verification and FD-003/FD-004 accepted-risk dispositions; check for regressions. Only then present a refreshed readiness verdict for an explicit human gate decision.
