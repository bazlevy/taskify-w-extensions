# Taskify / Personal Task Manager — Independent R2 Functional Design Review (R2-Rev3)

**Review ID:** **R2-Rev3** (third independent Functional Design review iteration; previously labeled `R2-R3`)  
**Independent review date:** 2026-10-22 (Australia/Melbourne)  
**Human approval recorded:** 2026-10-23 (Australia/Melbourne; decision explicitly stated by the product owner in this conversation)  
**Reviewer:** Independent Functional Design Reviewer (R2)  
**Independent reviewer verdict:** **READY_FOR_HUMAN_APPROVAL** — evidence assessment completed before human sign-off  
**Formal R2 human gate:** **APPROVED** — explicit product-owner approval received after R2-Rev3 review  
**Approved design package:** `specs(2).zip`, SHA-256 **`e75582cd7f849b918ce8e8de559449144936d0ea44cba518d4bdf7188b206f4f`**  
**Approved Scope:** `specs/001-personal-task-manager/spec.md`, SHA-256 **`4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`**  
**Evidence type:** Static inspection, original-to-revision line comparison, scenario replay against written behaviour, SHA-256/inventory checks. **No application code or executable tests were supplied or run.**  
**Approval boundary:** Human approval is for the reviewed **Functional Design baseline**, including the two explicitly accepted residual risks. It does not certify implementation, technical design, test execution or deployment readiness.

## 1. Baseline and controlled changes

- **Scope authority:** `specs/001-personal-task-manager/spec.md`, SHA-256 **`4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`**, byte-identical in all three uploaded ZIPs. Scope requirements FR-001–FR-016; AC-001–AC-012. Earlier R1 approval is evidenced by the approved hash/recorded human decision; the R1 approval report itself was not part of this `specs/` ZIP.
- **Initial design / R2-R1:** `specs.zip` (ZIP SHA-256 `43efe6e57bd587d233bf2d0f45a7fc6cc660bc509341008c370e79f3c8e6ec2f`). R2-R1 raised FD-001–FD-004.
- **Prior reviewed design / R2-R2:** `specs(1).zip` (ZIP SHA-256 `202e72e8518164cd23f7dc324a1e0847681ac11af5ee2eff51bab676e39f046e`); see `taskify_r2_rereview_R2_with_decisions.md`. FD-001/002 were independently verified, FD-003/004 accepted risks with documentation verified, and FD-005 plus three editorials were outstanding. The human owner withheld R2 approval until all four were independently checked.
- **Current reviewed and human-approved design / R2-Rev3:** `specs(2).zip` (ZIP SHA-256 **`e75582cd7f849b918ce8e8de559449144936d0ea44cba518d4bdf7188b206f4f`**). Nine Markdown artifacts inventoried. **Six changed** since R2-R2: `research.md`, `data-model.md`, `contracts/http.md`, `contracts/ui.md`, `quickstart.md`, and `checklists/feature.md`. **Three unchanged:** `spec.md`, `plan.md`, and `checklists/requirements.md`. The archive contains no application code or `tasks.md`; these are appropriately outside the present design review.
- **Evidence locations:** All path:line references below are relative to `specs/001-personal-task-manager/` inside **`specs(2).zip`**. The author's Codex message was not treated as proof of corrections.

## 2. Overall finding register and iteration history

| Finding | Category | Severity | Likelihood | R2-R2 status | R2-Rev3 status | Blocking | Evidence / rationale |
|---|---|---|---|---|---|---|---|
| **FD-001** — Task mutation retry during saved-task loading failure | FUNCTIONAL_DESIGN_DEFECT | HIGH | UNKNOWN | RESOLVED_VERIFIED | **RESOLVED_VERIFIED (regression checked)** | No | `contracts/http.md:159–164`; `contracts/ui.md:70–79`; `data-model.md:122–135`; `quickstart.md:82,84` — all mutation submissions, including uncertain replay, prohibited until successful valid snapshot. |
| **FD-002** — Restoring selected filters | FUNCTIONAL_DESIGN_DEFECT | MEDIUM | UNKNOWN | RESOLVED_VERIFIED | **RESOLVED_VERIFIED (regression checked)** | No | `data-model.md:28–47,92–98`; `contracts/http.md:79–108`; `contracts/ui.md:35–51`; `quickstart.md:67–78` — versioned account-backed storage, no browser preference fallback. |
| **FD-003** — Interrupted in-memory uncertain mutation | FUNCTIONAL_DESIGN_GAP | MEDIUM | UNKNOWN | ACCEPTED_RISK, disclosure verified | **ACCEPTED_RISK (disclosure regression checked)** | No | `contracts/ui.md:85–91`; `quickstart.md:84` — duplicate user intent/delayed effects remain possible after reload; owner already accepted this boundary. |
| **FD-004** — No credential recovery | SCOPE_ESCALATION (policy clarification) | MEDIUM | UNKNOWN | ACCEPTED_RISK, disclosure verified | **ACCEPTED_RISK (disclosure regression checked)** | No | `contracts/http.md:7–22`; `contracts/ui.md:7–11,93–95`; `quickstart.md:59` — explicitly no automated/manual reset or account transfer; owner already accepted consequence. |
| **FD-005** — Misreporting uncertain filter-save result and rapid-change state | NON_BLOCKING_IMPROVEMENT | LOW | UNKNOWN | IN_PROGRESS (correction approved) | **RESOLVED_VERIFIED** | No | `contracts/http.md:91–108`; `contracts/ui.md:35–51`; `research.md:57–73`; `data-model.md:40–47`; `quickstart.md:69–78`. |

**Confidence:** HIGH for the stated documentation-level dispositions (the relevant artifacts were directly inspected). Accepted risks FD-003 and FD-004 are **not removed** by their dispositions. No additional new R2 finding was substantiated in this revision.

### Audit trail

**Naming convention:** The final third-iteration review is identified as **R2-Rev3** throughout the approval record. Historical records `R2-R1` and `R2-R2` remain under their original names; `R2-R3` is an alias for this same third iteration, not a separate review.

| Iteration | Package | Functional design outcome | Human gate outcome |
|---|---|---|---|
| R2-R1 | `specs.zip` | NOT_READY — FD-001 and FD-002 blocking; FD-003/004 decided subsequently | No approval |
| R2-R2 | `specs(1).zip` | READY_FOR_HUMAN_APPROVAL, but FD-005 LOW and three editorial items remained | Owner explicitly WITHHELD approval until all four corrections verified |
| **R2-Rev3** (previously R2-R3) | **`specs(2).zip`** | **READY_FOR_HUMAN_APPROVAL — FD-005 and all three editorials independently verified; no open R2 findings** | **APPROVED — explicit human gate approval recorded 2026-10-23** |

## 3. FD-005 — independent change verification and counterexample replay

**Original risk:** A preferences PUT may commit on the server while its response is lost. A UI claim that the filters "were not saved" would then be false. Two or more rapid filter changes also created risk of confirming or rendering an older selection as current.

**Required outcome:** Distinguish confirmed 409 rejection from transport/5xx/malformed-response uncertainty; accurately describe unknown outcome; reconcile with account-authoritative GET before overwrite; prevent wrong-cause messaging and out-of-order regression; give observable acceptance scenarios.

| Scenario | Replayed written behaviour | Evidence | Result |
|---|---|---|---|
| PUT commits v2, response lost | Visible pair remains active; show **"Could not confirm your filters were saved"**; pause queued PUTs and GET saved pair/version; a matching authoritative pair confirms saved state. No blind repeat write. | `contracts/http.md:91–105`, `contracts/ui.md:43–49`, `quickstart.md:69–71` | **PASS** |
| Server returns `409 preference_version_conflict` | 409 is a confirmed version rejection, **not** proof another device caused it. No preference overwritten; read authoritative pair/version; if different from current selection require **explicit user approval** before a new PUT. | `contracts/http.md:98–105`, `contracts/ui.md:45–49`, `research.md:61–69`, `quickstart.md:71–73` | **PASS** |
| P1 in flight, user selects P2 then P3 | Exactly one active PUT; only the newest unsent pair is retained. Acknowledgement for P1 updates the confirmed version but cannot label P3 as saved; after a normal acknowledgement, the newest pair is sent against the next version. | `contracts/http.md:93–96,105–107`, `contracts/ui.md:37–41,49–50`, `research.md:59–60`, `quickstart.md:74–76` | **PASS** |
| Old reply/read arrives after later visible selection | Responses are associated with request/selection generation; ignore stale/out-of-order responses that could revert the visible choice or lower the confirmed version. | `contracts/http.md:105–107`, `contracts/ui.md:49–50`, `data-model.md:43–47` | **PASS** |
| Outcome uncertain, reconciliation GET unavailable | Keep attempted selection visible/unconfirmed; do not assume persistence and do not submit an unconfirmed automatic overwrite. The acceptance scenario explicitly challenges this. | `contracts/http.md:100–105`, `quickstart.md:76–78` | **PASS** |
| Another tab/device changes preference between GET and proposed resave | New PUT must use the re-read expected version and explicit user approval; server checks version and rejects a now-stale update without overwriting. | `data-model.md:37–47`, `contracts/http.md:91–105` | **PASS** |

**Disposition:** **FD-005 RESOLVED_VERIFIED**, documents only; Functional Design owner has made the agreed behavioural correction. Later Technical Design should refine error envelope structure and implementation sequencing, and Test must execute the loss/conflict/race cases. This is not a claim that server/browser code already implements the rules.

## 4. Three editorial corrections independently verified

| Item | Earlier discrepancy | Actual amendment | Result |
|---|---|---|---|
| **E-001** | `public_id` wrongly described as for browser preference namespacing | `data-model.md:9` now identifies it as the immutable opaque account identifier returned in authenticated snapshots; this aligns with `contracts/http.md:54–58`. | **VERIFIED** |
| **E-002** | CHK027/CHK031 referenced browser-based preference storage | `checklists/feature.md:63` now asks about account-owned tasks/preferences and session privacy; `:70` addresses server-save uncertainty, reconciliation and version conflicts. | **VERIFIED** |
| **E-003** | FD-001 test ID incorrectly duplicated FD-003 | `quickstart.md:84` now uses **R2-T05 / FD-001** and keeps **R2-T07 / FD-003**; separate FD-005 scenario is **R2-T06** at `:69`. | **VERIFIED** |

Searches across the provided nine files found no remaining old `browser preference namespacing`, `retained browser preferences`, `preference-storage assumptions` or `R2-T07 / FD-001` wording. Legitimate `localStorage` references now occur only in descriptions explicitly **prohibiting** browser preference storage or explaining why it was rejected. Unchecked checklist prompts are prompts, **not passed-test evidence**; editorial corrections do not transform them into test results.

## 5. Approved-requirements traceability (full re-check)

`COMPLETE` means functional behaviour specified at a level sufficient to write acceptance tests. It does not mean application or integration tests passed.

| FR | Coverage | Material design evidence and verification |
|---|---|---|
| FR-001 | COMPLETE | `contracts/http.md:114–121`; `contracts/ui.md:17–21` — title-only create and validation |
| FR-002 | COMPLETE | `data-model.md:51–64`; `contracts/http.md:116–121` — priority/default/clearing |
| FR-003 | COMPLETE | `contracts/http.md:54–72`; `contracts/ui.md:13–15` — visible saved attributes; no invented values |
| FR-004 | COMPLETE | `contracts/http.md:117–121`; `contracts/ui.md:21` — edit and optional-value clearing |
| FR-005 | COMPLETE | `contracts/http.md:118–121`; `contracts/ui.md:23` — complete/reopen preserves details |
| FR-006 | COMPLETE | `contracts/http.md:119,144–155`; `contracts/ui.md:25` — confirmed permanent delete, concurrency |
| FR-007 | COMPLETE | `contracts/http.md:48–50`; `data-model.md:92–98` — combined status/priority filters |
| FR-008 | COMPLETE | `research.md:21–35`; `data-model.md:17–26,70–86` — committed task writes, snapshots, cross-device convergence |
| FR-009 | COMPLETE | `contracts/ui.md:59–79`; `contracts/http.md:140–164`; `quickstart.md:82–86` — rejection, uncertainty and read failure |
| FR-010 | COMPLETE | `contracts/http.md:5–26`; `research.md:13–19` — authentication and private ownership |
| FR-011 | COMPLETE | `data-model.md:106–110`; `contracts/ui.md:57` — date-only and local-calendar overdue |
| FR-012 | COMPLETE | `contracts/http.md:79–108`; `contracts/ui.md:35–51`; `quickstart.md:67–78` — account-backed filters, versioning, no silent overwrite, truthful save state |
| FR-013 | COMPLETE | `data-model.md:100–104`; `contracts/ui.md:53–55` — all sort orders and deterministic ties |
| FR-014 | COMPLETE | `contracts/http.md:159–164`; `contracts/ui.md:66–79`; `data-model.md:122–135` — no task mutations while saved task snapshot unavailable |
| FR-015 | COMPLETE | `plan.md:24–28`; `contracts/ui.md:13–15` — responsive desktop/mobile browser behaviour and accessibility |
| FR-016 | COMPLETE | `contracts/http.md:7–22`; `contracts/ui.md:7–11` — self-registration and returning sign-in |

**Coverage count:** **16 COMPLETE; 0 PARTIAL; 0 MISSING; 0 INSUFFICIENT_EVIDENCE** based on the supplied design artifacts.

## 6. Workflow, state, failure, side-effect and recovery coverage

| Flow or adverse scenario | Verdict | Principal evidence |
|---|---|---|
| Self-register, login, logout, invalid credentials, no recovery | COVERED / accepted no-recovery risk | `contracts/http.md:7–26`; `contracts/ui.md:7–15`; `quickstart.md:59` |
| Create/edit/status/delete; validation, explicit confirmation, cancellation | COVERED | `contracts/http.md:110–157`; `contracts/ui.md:17–31`; `quickstart.md:61–63` |
| Task data integrity; retries/duplicates/conflicting versions | COVERED | `data-model.md:17–26,70–86,118–136`; `contracts/http.md:140–164` |
| Sort/filter and server preference restoration, multi-device updates | COVERED | `data-model.md:28–47,92–104`; `contracts/http.md:79–108`; `contracts/ui.md:33–55` |
| Preference saving delayed/lost/409/re-read failure, rapid selections | COVERED; FD-005 verified | `contracts/http.md:91–108`; `contracts/ui.md:35–51`; `quickstart.md:69–78` |
| Initial and refresh saved-task load failures with pending unknown task writes | COVERED; FD-001 regression pass | `contracts/ui.md:59–79`; `contracts/http.md:155–164`; `quickstart.md:82–84` |
| Browser interruption of uncertain task operation | ACCEPTED_RISK; guidance intact | `contracts/ui.md:85–91`; `quickstart.md:84` |
| Session expiry, wrong-account/stale replies, privacy | COVERED functionally; technical enforcement later | `contracts/http.md:24–30,75–77`; `contracts/ui.md:74,81–95` |
| Empty matches, calendar overdue, accessibility and confirmation interactions | COVERED | `contracts/ui.md:13–15,25,53–68`; `data-model.md:106–110` |

**Regression counterexamples rechecked:** retry a delete while load is unavailable → blocked; late original create receipt after remote deletion → historical acknowledgement without resurrection; browser reload after pending create → the documented duplicate-intent risk remains accepted; forgotten credentials → no promised reset; selected filters after session/device return → authoritative pair; stale preference PUT → no server overwrite; false "not saved" after committed PUT/lost response → removed.

## 7. Outstanding decisions and downstream dependencies

**No outstanding R2 defect, gap, editorial correction or new Scope decision is identified from the supplied design.** The human owner has **approved the Functional Design gate** with FD-003 and FD-004 remaining **explicitly accepted risks**. Their residual consequences must not be represented as eliminated. The agreed R2-Rev3 evidence baseline is fixed by the hashes above; subsequent changes require change control and re-review proportional to their impact.

**Routing for downstream phases (not new R2 defects):**

- **R3 Technical Design:** Validate session/CSRF isolation, transactional account locks/receipts, preference version locking, 409/error response schema, request-generation sequencing, authoritative-read error handling, browser compatibility, and operational recovery/support configuration.
- **Test:** Execute preference cases `R2-T06`, uncertain-task-write `R2-T05`, interruption `R2-T07`, credential disclosure `R2-T12`, plus baseline success/boundary/concurrency/security and cross-device journeys. Include preference GET outage/retry and same-tab versus cross-device conflict cases. Static scenario coverage is not execution evidence.
- **R4 Integrated Product Design / R5 readiness:** Ensure the eventual tasks/test plan actually covers these contracts and inherited accepted risks. Do not expand Scope or retroactively declare the risks solved.

## 8. Independent exit-gate assessment

| Criterion | Assessment |
|---|---|
| Approved Scope baseline byte-identical | **PASS** |
| FR traceability and testable material outcomes | **PASS — 16/16 COMPLETE** |
| Material workflows, transitions, failure/recovery and cross-flow consequences | **PASS** at functional-design level |
| FD-001/FD-002 previous blockers | **PASS — RESOLVED_VERIFIED** |
| FD-003/FD-004 documented product-owner risk decisions | **PASS — ACCEPTED_RISK, guidance checked** |
| FD-005 approved correction | **PASS — RESOLVED_VERIFIED** |
| Three requested editorial corrections | **PASS — VERIFIED** |
| Unresolved R2 blocker or outstanding owner-mandated correction | **NONE FOUND** |
| Application/runtime/integration tests | **NOT PERFORMED — future stages** |
| Explicit R2 human approval | **PASS — APPROVED by product owner on 2026-10-23** |

### Formal independent R2 reviewer verdict: **READY_FOR_HUMAN_APPROVAL**

The owner's additional condition from the prior iteration — independently verify FD-005 and all three editorial corrections **before** sign-off — was satisfied by R2-Rev3. This independent reviewer verdict remains a distinct evidence assessment and does not itself constitute human approval.

### Formal human R2 gate decision: **APPROVED**

**Decision record**

| Field | Recorded value |
|---|---|
| Gate | R2 — Independent Functional Design Review |
| Canonical review ID | **R2-Rev3** |
| Decision | **APPROVED** |
| Decision maker | Human product owner (explicit declaration in the conversation) |
| Decision recorded | **2026-10-23**, Australia/Melbourne |
| Reviewer recommendation | **READY_FOR_HUMAN_APPROVAL** |
| Approved Scope baseline | `specs/001-personal-task-manager/spec.md` SHA-256 `4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae` |
| Approved Functional Design baseline | `specs(2).zip` SHA-256 `e75582cd7f849b918ce8e8de559449144936d0ea44cba518d4bdf7188b206f4f` |
| Open R2 findings | **0** |
| Independently verified fixed findings | FD-001, FD-002, FD-005 |
| Explicitly accepted residual risks | FD-003 (interrupted in-memory operations), FD-004 (no password recovery) |
| Other corrections verified | E-001, E-002, E-003 |
| Gate disposition | **CLOSED — APPROVED**, for the specified Functional Design baseline |
| Next independent stage | **R3 — Technical Design Review**, followed by Spec Kit Tasks / Analyze according to the approved workflow |

**Human decision evidence:** In this conversation the product owner explicitly stated, “Human approval has been granted for Independent R2 Functional Design Review,” and requested the final review identifier `R2-Rev3`. This declaration is the human approval record; no separate digital signature, external approval workflow or immutable repository commit has been provided or asserted.

**Change-control condition:** Approval applies to the document contents in the hash-identified design ZIP. Any material behavioural change or new requirement requires assessment against the approved Scope and may require a further R2 review. This decision must not be interpreted as approval of application code or a replacement for R3/R4/R5 or later human approvals.

**Integrity note:** This report incorporates the original R2-Rev3 independent evidence assessment and the subsequent human gate decision. No supplied ZIP contents or earlier reports were modified to obtain this approval. Static documentation review, not runtime testing, supports the reviewer conclusions.
