# Taskify — Independent R2-Rev4 Functional Design Review

**Review date:** 2026-10-23 (Australia/Melbourne)  
**Reviewer role:** Independent Functional Design Reviewer (R2)  
**Review type:** Targeted AM-001 amendment review, with affected-behaviour and material regression checks  
**Independent reviewer verdict:** **READY_FOR_HUMAN_APPROVAL**  
**Human R2 amendment gate:** **CLOSED — APPROVED by Barry Levy on 2026-10-23 (Australia/Melbourne)**  
**Scope authority:** Taskify Scope v1.1 / AM-001, human **APPROVED** in R1-Rev4 Section 8  
**Design authority:** Current amended Plan/design artifacts remain **UNAPPROVED** until separate human R2 sign-off  
**Method:** Static inspection of actual supplied Markdown artifacts, cross-document traceability and adverse-scenario replay. No implementation or runtime tests were performed.

## 1. Controlled baseline and evidence

| Item | Evidence / result |
|---|---|
| Reviewed package | `Archive.zip`, SHA-256 `74fb1d4b5b7d0d9162ba4fa9b7958b060b67a146a90a95f9e489eedba0731ba7` |
| Current canonical scope | `specs/001-personal-task-manager/spec.md`, SHA-256 **`b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`** — directly recomputed |
| Approved AM-001 candidate | SHA-256 `61062311e5d1423f1d13f485c555158964e09f0548d13263bfefa0f048447fd4`, human approval recorded in `independent-reviews/1_scope/Review_4/scope_review_report.md`, Section 8 |
| Product-definition integrity | The canonical specification is **byte-identical from `## Product Boundary` onward** to R1-Rev4's `spec_proposed.md` in this archive. R1-Rev4 Section 7 separately records that the approved candidate's product-definition body was identical to that proposal. This corroborates Codex's metadata-only-change statement. The approved candidate bytes themselves are not in this archive, so a direct candidate-to-current byte diff was **not** possible. |
| Previous R2 approval | `independent-reviews/2_design-functional/Review_3 (final)/taskify_R2-Rev3_functional_design_review_APPROVED.md` records reviewer READY and later human APPROVED for the *previous* design package `specs(2).zip` SHA-256 `e75582cd7f849b918ce8e8de559449144936d0ea44cba518d4bdf7188b206f4f`. This does **not** carry approval to AM-001. |
| Previous R3 result | R3-Rev1 **NOT_READY**, TD-001–TD-007 OPEN, with TD-002 triggering the scope amendment. Its Addendum A is historical direction, superseded where inconsistent with the final Scope-approved **silent** fallback rule. |

**Current design materials inspected:** `plan.md`, `research.md`, `data-model.md`, `contracts/http.md`, `contracts/ui.md`, `quickstart.md`, both checklists and `review-followups/am001-r2-review-handoff.md`; current canonical `spec.md`; R1-Rev4 report; prior R2-Rev3 report; and R3-Rev1 report. Paths in the remainder are relative to `specs/001-personal-task-manager/` unless otherwise stated.

**Diff limitation:** The old *actual* `specs(2).zip` design files were not supplied in this archive, so claims that individual unaffected files/paragraphs are byte-identical to R2-Rev3 cannot be independently hash-verified. The current contracts were inspected substantively for regressions, and the earlier report was used for historical dispositions, not as proof of the new design.

## 2. Executive assessment

**READY_FOR_HUMAN_APPROVAL.** The amended design represents all three approved AM-001 decisions at functional level:

1. The four-setting view is applied immediately; unsuccessful or unconfirmed remembering leaves the current view usable **silently**. An unavailable view alone never blocks successfully loadable task data/actions.
2. View persistence is bounded to the **current still-valid authenticated login** shared by its tabs. A new login or a separate browser/profile/device session starts with defaults, while tasks remain account-persistent and synchronised.
3. Status, priority, sort method and direction are one atomic view: if any remembered element is missing/invalid/unreadable, **all four** revert to defaults without a warning.

The revised HTTP, UI, data, state and test contracts also specify concrete behaviour for concurrent tabs, rapid choices, lost responses and logout/expiry. No newly substantiated **R2 blocking** inconsistency was found. This is evidence of **written behavioural completeness**, not implementation correctness. Remaining security, locking, resource and operations issues belong to R3/R5 and stay open.

## 3. Requirement traceability

`COMPLETE` means the current design specifies outcomes sufficiently for behavioural acceptance tests, not that tests passed.

| Approved requirement | Assessment | Inspected evidence / result |
|---|---|---|
| FR-001 — create/title/date validation | COMPLETE | `data-model.md` Task; `contracts/http.md` Mutations; `contracts/ui.md` Task forms |
| FR-002 — effective priority/Medium default | COMPLETE | `data-model.md` Task; HTTP create/edit defaults |
| FR-003 — display saved task attributes | COMPLETE | HTTP snapshot DTO; UI task list/detail and absent values |
| FR-004 — edit/clear optional details | COMPLETE | HTTP full edit replacement and validation; UI Edit |
| FR-005 — complete/reopen/details invariant | COMPLETE | HTTP status command; UI complete/reopen |
| FR-006 — explicitly confirmed permanent deletion | COMPLETE | HTTP confirmed delete/version check; UI confirmation/cancel |
| FR-007 — status/priority combined filtering | COMPLETE | HTTP query; data model AND predicates; UI controls |
| FR-008 — saved writes and cross-device task synchronisation | COMPLETE | `research.md` §§3–5; account locks/revisions/receipts; polling; snapshot semantics |
| FR-009 — task error/sync feedback | COMPLETE | HTTP error/rejection/unknown outcomes; UI failure-state table; `quickstart.md` §§7–9 |
| FR-010 — private accounts/no foreign task access | COMPLETE | HTTP auth and owner-derived access; UI invalidation; server protection described; TD-001 remains separate OPEN R3 risk |
| FR-011 — overdue date-only semantics | COMPLETE | `research.md` §6; derived calendar rule; UI midnight/focus recomputation |
| **FR-012 — four-setting session view and silent fallback** | **COMPLETE** | `research.md` §7; `data-model.md` SessionViewState; HTTP bootstrap/POST; UI Filters; AM001-V01–10 |
| **FR-013 — bidirectional sorting and restored method/direction** | **COMPLETE** | `research.md` §6; `data-model.md` Task Sorting; UI/HTTP query; `quickstart.md` §5 |
| FR-014 — saved-task load protection/retry/support | COMPLETE | UI loading/load_error/outcome_unknown; HTTP task-mutation lock; `quickstart.md` §§7–8 |
| FR-015 — desktop/mobile browser access | COMPLETE | Plan platform; responsive/accessible UI; future browser tests listed |
| FR-016 — self-registration | COMPLETE | HTTP registration/login; UI disclosure/validation; quickstart scenario 1 |

**Coverage: 16 COMPLETE / 16 total; 0 PARTIAL; 0 MISSING; 0 INSUFFICIENT_EVIDENCE** for the written Functional Design. AC-001–AC-012 and SC-004 were checked for consistency with the amended contracts; only AC-005 changed under approved Scope. No new product rule or numerical service threshold was assumed.

## 4. AM-001 adverse-scenario replay

| Case / counterexample | Required observable result in current design | Evidence / disposition |
|---|---|---|
| **AM001-V01** — Choose Completed/High/Title descending and successfully save | Controls apply immediately; same-login refresh/new tab restores **all four** from the complete persisted tuple | HTTP bootstrap/POST; UI Filters; `quickstart.md` §5 — **COVERED** |
| **AM001-V02** — Two tabs, one changes view | Existing tab keeps its controls; later loaded/refreshed tab reads last committed valid tuple; no mandatory live synchronisation | Research §7; UI Filters — **COVERED** |
| **AM001-V03** — Two independently authenticated devices, same account | Each session has independent default/remembered view; **task** changes still converge account-wide | Data model SessionViewState/Task; HTTP read; quickstart — **COVERED** |
| **AM001-V04** — Logout, expiry, fresh same-account login, browser restart | Fresh explicit login uses defaults; valid existing login retains its own view. Tab/browser lifecycle alone is not treated as new login | HTTP auth; data model guarded lifetime; UI Filters — **COVERED**; mechanism to R3 |
| **AM001-V05** — One tuple field/schema corrupt or view read fails | All/All/Due Date ascending, without notice or GET repair write; tasks still load when auth/task storage works | HTTP Read snapshot; data model SessionViewState; UI failure states — **COVERED** |
| **AM001-V06** — View POST commits but response is lost, or definitely fails | Current selection stays functional without warning or untruthful save status; subsequent load uses actual committed complete view or defaults | HTTP Session view; Research §7; UI Filters — **COVERED** |
| **AM001-V07** — Rapid selections while save in flight | One in-flight save per tab; newest queued tuple retained; old response never rewrites/claims newer visible controls; no blind replay | HTTP Session view; data model SessionViewState; UI Filters — **COVERED** |
| **AM001-V08** — Competing tabs and delayed server commits | Complete tuple replaces atomically; last actual server commit wins, not click time; another tab's controls do not switch; middleware session-write risk explicitly referred to R3 | Research §7; data model SessionViewState; HTTP POST — **COVERED at R2 behaviour level** |
| **AM001-V09** — Old POST races logout/expiry/rotation/new login | Old request must not recreate an invalid session, leak prior view, or change a new login's view; old client responses ignored | Data model guarded lifecycle; HTTP auth/POST; UI clearing — **COVERED at R2 behaviour level**; locking feasibility to R3 |
| **AM001-V10** — View error coincides with failed authentication or task snapshot | Auth failure clears private data; task-load error shows retry/support and blocks task writes, including uncertain replay; no fake empty success | HTTP Read snapshot and failures; UI failure table; quickstart §7 — **COVERED** |

**Important bounded trade-off:** A timed-out earlier view save can commit *after* a later explicit selection and become the remembered view. The documents acknowledge commit-order precedence and no guaranteed replay/reconciliation. This is consistent with the human-approved **silent, best-effort convenience** policy and is not treated as a new correctness promise. It must be exercised in implementation tests.

## 5. Material regression checks against prior R2

| Prior item or invariant | Current status | Evidence / consequence |
|---|---|---|
| **FD-001** — No task mutation/retry while saved task loading unresolved | **RESOLVED_VERIFIED historically; CURRENT DESIGN REGRESSION PASS** | HTTP Transaction/failure semantics; UI load_error/outcome_unknown; data model state table; `quickstart.md` R2-T05. A view-only failure does not remove the *task-load* lock. |
| **FD-002** — Earlier account-backed filter restoration | **RESOLVED_VERIFIED for old baseline; intentionally SUPERSEDED for AM-001** | Account preference GET/PUT replaced by session view bootstrap/POST. New policy assessed in §4. Do not claim old account-wide persistence survives. |
| **FD-003** — Browser interruption loses in-memory pending mutation identity | **ACCEPTED_RISK — RETAINED** | Research §5; UI Sync/privacy; `quickstart.md` R2-T07. A delayed first task operation and repeated user intent may duplicate/alter state. |
| **FD-004** — No password/credential recovery | **ACCEPTED_RISK — RETAINED** | HTTP auth; UI registration/login/support disclosure; `quickstart.md` R2-T12. No reset or transfer is promised. |
| **FD-005** — Earlier preference save uncertainty/version handling | **RESOLVED_VERIFIED for old baseline; intentionally SUPERSEDED in relevant parts** | New save outcome is silent and unconfirmed; no old preference version/409 reconciliation. Stale replies still cannot alter newer controls or claim an unconfirmed save. |
| **E-001–E-003** — Prior editorial fixes | **No regression substantiated in inspected current content** | `public_id` remains correctly described, R2-T05/T07 distinct, and old account-preference wording is marked historical/superseded where retained. |
| Task persistence/privacy/ordering | **REGRESSION PASS at documented-behaviour level** | Account task revisions, receipt-first idempotency, explicit conflict resolution, date-only overdue, all four sorts/ties, and account ownership retained. |

The historical dispositions and prior R2 human approval remain attached to **R2-Rev3 only**. This assessment does not silently rewrite its report or retroactively approve changed design.

## 6. Current findings and improvements

**New blocking FD findings: NONE.** No new substantiated functional-design defect/gap requiring a Scope decision or Codex correction was found in AM-001's written behaviour. Prior FD-003/004 remain accepted risks, not resolved defects.

**Non-blocking documentary improvement (not an R2 gate condition):** `checklists/feature.md` CHK012/020/027/031 retain historical account-preference terminology; CHK031 specifically still asks about account-backed reconciliation/version conflicts. The AM-001 appendix explicitly states those prompts are superseded where incompatible and adds CHK037–052. For the next checklist consolidation, Codex should give those legacy questions an unambiguous **superseded/not-applicable** disposition rather than treating them as current criteria or marking obsolete behaviour `[x]`. This is process hygiene, not evidence of a missing product behaviour. The 16 AM-001 checklist questions remain deliberately unchecked, and no checkbox is interpreted as test execution.

**Assurance limitation, not a defect:** No application source, runtime tests, prior actual design ZIP for bytewise diff or standalone approved-candidate file were provided. Written scenarios are **planned**. The review does not certify Django locking/session semantics, browser history privacy, cross-tab invalidation, receipt digest privacy, capacity, abuse control or operational recovery.

## 7. Open downstream dependencies and owners

| Finding | Status retained | Required next evidence / owner |
|---|---|---|
| **TD-001** — stale task content across logout/account switch | **OPEN — HIGH; R3 blocker** | Technical Design + UI/security: conceal before revalidation, cross-tab invalidation/account-switch behaviour and A→logout→B scenario. AM-001 session reset does **not** fix this. |
| **TD-002** — failed old preference bootstrap | **OPEN pending R3 re-verification** | R3: verify removal of standalone blocking preference GET; isolation of view read/save failure from task snapshot; timeout/locking/transaction safety and truthful default response. Propose closure only after technical evidence. |
| **TD-003** — full snapshot/receipt capacity | **OPEN** | R3 performance/operations: bounded workload/resource model and later stress evidence; no arbitrary product cap/SLA. |
| **TD-004** — receipt digest privacy | **OPEN** | R3 security/privacy: keyed/unkeyed choice, replay/rotation and risk assessment. |
| **TD-005** — authentication throttling | **OPEN** | R3 security/deployment: trust boundary, shared enforcement and fail behaviour. |
| **TD-006** — diagnostics/backup/migration recovery | **OPEN; R5 handoff proposed** | Named operations owner, traceable redacted request references and disposable restore/migration test. |
| **TD-007** — canonicalisation compatibility | **OPEN; R4/upgrade handoff proposed** | Versioning/replay compatibility and rollback/migration evidence. |
| **R1-AM-004** — unavailable historical Section 13 | **DEFERRED, non-product** | Scope/document-control owner: archival retrieval if possible; not an AM-001 product gate. |

**FD-003/FD-004:** Preserve explicit owner-accepted consequences in future tasks, tests, support copy and release decisions. No new human risk acceptance is inferred here. **R3** must also test Django session-key rotation, expiry, logout and concurrent view saves, including whether best-effort view reads can stall otherwise available task snapshots. These are implementation-mechanism checks, not authority to change Scope.

## 8. Independent R2 exit-gate assessment

| Criterion | Result |
|---|---|
| Approved Scope 1.1 / AM-001 authority established | **PASS** — R1-Rev4 Section 8; scope product-body correspondence checked through supplied R1 proposal and verification |
| Changed FR-012/FR-013/AC-005 traced to design | **PASS** |
| All 16 approved FRs materially covered | **PASS — 16/16 COMPLETE** |
| Session lifetime, four-field restoration/defaults, cross-tab precedence | **PASS** at behavioural level |
| Uncertain/failed view saving handled silently without false guarantee | **PASS** |
| View convenience failure separated from auth/task-loading failures | **PASS** |
| Earlier task-load/idempotency protections and accepted risks preserved | **PASS** at documented-behaviour level |
| New R2 blocker requiring correction/human product clarification | **NONE FOUND** |
| Existing R3 technical blockers resolved | **NO — TD-001–007 remain OPEN**; independent R3-Rev2 required |
| Application tests executed | **NO — not within supplied evidence** |
| Human R2 approval for amended design | **PASS — APPROVED by Barry Levy on 2026-10-23, subsequent to this independent assessment** |

### Formal independent reviewer verdict: **READY_FOR_HUMAN_APPROVAL**

The amended written Functional Design is sufficiently complete, internally coherent and testable for a human R2 approval decision. The residual best-effort view convenience trade-off was explicitly authorised in Scope AM-001. No new blocker was identified by the independent targeted re-review. This verdict is **not** human approval and is **not** technical/build readiness.

### Formal human R2 decision record — 2026-10-23 (Australia/Melbourne)

| Field | Authoritative recorded decision |
|---|---|
| Gate | R2 — Independent Functional Design Review, AM-001 |
| Canonical review ID | **R2-Rev4** |
| Reviewer recommendation | **READY_FOR_HUMAN_APPROVAL** |
| Human decision | **APPROVED — Barry Levy, human product owner** |
| Human declaration | “I approve” — direct reply to the independent R2-Rev4 recommendation and request for explicit Taskify R2-Rev4 / AM-001 sign-off |
| Human decision date | **2026-10-23 (Australia/Melbourne)** |
| Gate disposition | **CLOSED — APPROVED** for the exact amended Functional Design baseline below |
| Approved Scope | Version 1.1 / AM-001; original candidate SHA-256 `61062311e5d1423f1d13f485c555158964e09f0548d13263bfefa0f048447fd4`; metadata-updated canonical `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca` |
| Reviewed design | The design files in `Archive.zip` SHA-256 `74fb1d4b5b7d0d9162ba4fa9b7958b060b67a146a90a95f9e489eedba0731ba7`, excluding unrelated historical source files from the **design approval scope**; the current design files are identified by path and SHA-256 below |
| Remaining gate boundary | R3-Rev2 independent Technical Design Review; explicit R3 human approval; no Tasks/Analyze/Build yet |

### Exact reviewed design file SHA-256 manifest

All paths are relative to `specs/001-personal-task-manager/`. These hashes identify the **specific written design** being recommended; they are not implementation/test results.

```text
plan.md                          b55d38011604e0d93578cbf300b8ecb177fa4a16fbec759971e77b47f8170224
research.md                      3b7958f2e52af060621c77453b362f686c77eba5e343c60ef4f0d5d96c2a3a8d
data-model.md                    44fe7219af32708ea1afd76a049b7b413e54d606e4f06d0e18733c479a5c2cdc
contracts/http.md                b179d110424b71af81789e5d9ec2fd8cca6981e8a08da9370f7c3a9cd0213a5a
contracts/ui.md                  7d617894547a5b06c878b09304d85c5424881dc69cce111690ea0ba8d56ec266
quickstart.md                    ee774bee1fcdb832eee99ac7e96417206cd6ac377283f7ac252d871abab8c7cb
checklists/feature.md            f1d97f2a603129faee2176fa2422f85fa1c73022a10c148ee7f00128eee0917f
checklists/requirements.md       68e439444144a050894246f28538cad798eba01154607dbd1f63e219209f99ec
review-followups/am001-r2-review-handoff.md
                                 da4899f7471d9f59f91d43b08bb1e2949924f8c64aaf46fd9240782d6de0bf3a
```

**Meaning of approval:** The human owner accepted the independent R2-Rev4 recommendation for the exact reviewed amended Functional Design, with the previously recorded FD-003/FD-004 residual risks retained. The human approval is a separate decision from the reviewer's READY_FOR_HUMAN_APPROVAL verdict. It approves no application code, runtime test result, R3 technical mechanism, production readiness or Build. TD-001–007 remain open and require their prescribed dispositions.

**Next action:** Codex should record this human R2-Rev4 approval without changing the reviewed design baseline, then address the remaining R3 technical corrections and return for **independent R3-Rev2** review and separate human R3 approval. Preserve prior R2-Rev3/R3-Rev1 histories and R1-AM-004's DEFERRED status. Do not generate Tasks, run Analyze, implement or deploy until the relevant downstream gates authorise those actions.
