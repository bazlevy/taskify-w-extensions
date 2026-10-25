# Taskify — R2-Rev8 Independent Targeted Functional Design Review and Human Approval

**Review ID:** R2-Rev8  
**Review date:** 2026-10-25 (Australia/Melbourne)  
**Independent reviewer verdict:** **READY_FOR_HUMAN_APPROVAL**  
**Subsequent human R2-Rev8 gate:** **CLOSED — APPROVED** by Barry Levy, 2026-10-25 (Australia/Melbourne)  
**Boundary:** FD-009 closure and proposed CA-006/007/008, RA-FD006-001, Q1, R2-NAV-001 and directly dependent FR-010/AC-010, FR-012/013, FR-008/009/014 and CA-T01–16.  
**R3:** **NOT_READY**, TD-008/010/011 OPEN. No Tasks, Analyze or Build authority.

## 1. Review chronology, baseline and exact evidence

This is the **new independent R2 review after R1-Rev5 Scope approval**, not a retrospective relabelling of R2-Rev7. The preceding `R2-Rev7` reviewer verdict was **NOT_READY** because FD-009 required authoritative R1/AC-010 clarification. R1-Rev5 independently verified the targeted AM-002 amendment and Barry then approved Scope v1.2. R2-Rev8 therefore reassesses the **unchanged** conventional-auth Functional Design proposal against the **new approved** Scope baseline.

| Source | Verified identity / authority |
|---|---|
| Reviewed archive | `Archive(2).zip` SHA-256 `16f0d341116b7eede772d3c7e6014ef6e8a8fa0719343dc89ec51f7336e58bdb` |
| Approved Scope v1.2 / AM-002 | Independently reviewed and human-approved product-definition candidate `specs/001-personal-task-manager/spec.md` SHA-256 `87bf1574788c46eedf6412782e1b6a60915a4db0d4b643483d9e79736a2c745c`; subsequent canonical approval-metadata-only file SHA-256 `688b57c11785af800b072843d7763c14875dbc6cbb5b35594201936cee03549b` |
| R1 authority | `independent-reviews/1_scope/Review_5/Taskify_R1-Rev5_Targeted_Scope_Review_AC010.md` SHA-256 `992ebc81381920a86d94041c6164f48235722c299f11184a9b3183c3a3716dbf`; independent verification in §8 and human Scope approval in §9 |
| Previous R2-Rev7 | `independent-reviews/2_design-functional/Review_7/Taskify_R2-Rev7_Targeted_Functional_Design_Review_NOT_READY.md`; original `NOT_READY` determination preserved; FD-009 referred to R1 |
| Functional Design replacement candidate | `review-followups/r2-conventional-auth-amendment-proposal.md` SHA-256 `8199decf14eb62f063bab7db16bb4d98a54686783584b4cd1dc51aeb3370385e` |
| Evidence map and planned acceptance tests | `r2-conventional-auth-amendment-evidence.md` SHA-256 `5237fa167a79923b17589c34560ab70d38d754d985f531035126e57e6c00709a`; `r2-conventional-auth-amendment-scenarios.md` SHA-256 `009f33f8cfedd227a7342b079feb1705f8b6154e01b5b8ed13535b20e3207a15` |
| Previous approved R2 clauses | R2-Rev5/R2-NAV-001 and R2-Rev6/FD-006/007/008/Q1 approvals remain historical source authorities; their superseded clauses remain binding until this new human decision is incorporated |

The current `spec.md`, R1 report and three proposed R2 documents were independently rehashed and matched the listed identities. Independently reversing the supplied AM-002 approval-metadata diff against the current canonical `spec.md` reproduced the **exact** R1-Rev5 human-approved candidate hash `87bf1574...2c745c`, establishing that approval metadata changed but the approved product-definition text did not. These are **static/written checks**, not executed tests or R3 architectural proof.

## 2. Targeted findings, Scope trace and risk qualification

| Finding / dependency | Independent R2-Rev8 assessment | Disposition |
|---|---|---|
| **FD-009 — AC-010 Scope consistency** | Amended AC-010 explicitly permits prior-login-authorised content to persist or finish rendering while a same-profile login change remains *unobserved*, with no maximum observation delay. It requires clearing/concealment on observation and forbids displaying known mismatches, cross-account API reads/mutations and old-intent rebinding. CA-006 §3 and CA-T05–08 stay within these limits. | **RESOLVED_VERIFIED**. Earlier R2-Rev7 blocker no longer applies under approved Scope v1.2. |
| **FD-006 / CA-006** | Conventional server authentication, owner-scoped protected reads/writes, intended-login epoch checks and local rejection of known stale response/account/query generations retained. Lifecycle/uncertainty boundaries trigger revalidation. No browser-wide atomic validation-to-render guarantee or per-local-interaction auth request. | **ACCEPTED at written functional level**, subject to human R2 approval recorded below; R3 owns implementation feasibility. |
| **FD-007 / CA-007** | Genuine in-flight overlapping **anonymous** login responses may follow ordinary browser session-cookie application/selection; no first-browser-established-winner custom coordination. Already-authenticated GET/POST/stale form paths retain R2-NAV-001; deliberate switching requires logout first, CSRF and logout/revocation remain guarded; a new actual login starts default view. | **ACCEPTED at written functional level**; TD-008 remains R3 work. |
| **FD-008 / CA-008** | Confirmed intended-login shared revocation, prompt same-profile notification dispatch and receiving-tab clear/navigation, missed-message focus/resume recovery, later-login nonrevocation, independent-device continuity and no false logout ACK retained. No guaranteed message delivery, tab-ACK or atomic clear. | **RETAINED / consequential alignment ACCEPTED**; TD-011 remains R3 work. |
| **Q1 / FD-001 / FR-014 and task/view rules** | Sensitive server operations remain authenticated and intended-login guarded. Cached local UI actions need not issue needless independent auth round trips; observed change/uncertainty still conceals/revalidates. Task-load failure locks mutations and retries, view-only remembering failure stays silent, operation IDs, versions and receipts remain protected. | **NO MATERIAL REGRESSION SUBSTANTIATED**. |
| **CA-T01–16** | Written cases cover anonymous response order, authenticated navigation, wrong-owner/old-intent reads and writes, stale and unobserved responses, confirmed logout/missed messaging, CSRF/recovery, task/view persistence and unchanged regressions. | **COMPLETE for targeted written R2**, planned and **not executed**. |

**Residual RA-FD006-001:** A previously authorised A view or already-authorised response may be displayed temporarily on stale Tab A after another same-profile tab selects B before that change is observed. This can have **HIGH potential confidentiality consequence** on a shared device, with **UNKNOWN likelihood/exposure** and **HIGH confidence in the reasoned counterexample**, not a measured incident. No finite exposure duration is promised. Barry authorised the bounded trade-off and approved the exact AC-010 Scope definition; server cross-account access, malicious rebinding, intentional known-mismatch display and post-observation retention remain prohibited. The accepted risk is not a blanket privacy exception.

**Evidence/proportionality:** The exceptional first-winner and atomic-render obligations arose during prior R2 remediation, not expressly from original Scope. Their removal is now an authorised product change after repeated R3 complexity assessments. Standard framework/browser tools remain a credible foundation but must be designed and reviewed against the **retained** protections. No new R2 preference or technical mechanism is made binding by this review. The earlier FD-008 evidence-map wording recommending RECONSIDER of guaranteed delivery is a **minor nonblocking classification issue**; notification mechanism verification belongs to TD-011/R3.

## 3. Independent R2-Rev8 gate verdict

**READY_FOR_HUMAN_APPROVAL.** All material targeted requirements, changed outcomes, failure/recovery consequences and representative tests are coherently defined. Approved R1-Rev5 Scope v1.2 closes FD-009 without a further design amendment. There are **zero open material R2 blockers in this review**. This verdict is independent written-design assessment, not human sign-off, runtime validation, automatic TD closure or Build authority.

## 4. Subsequent human R2-Rev8 approval — 2026-10-25 (Australia/Melbourne)

After the R2-Rev8 numbering correction was expressly raised, Barry was asked: **“Do you approve R2-Rev8 for the same Functional Design you just approved, with no substantive changes?”** He answered **“yes.”** This is the explicit human approval of the exact design under the **R2-Rev8** review identity. Earlier **“approve”** and **“approved”** messages were supplied in response to an incorrectly labelled R2-Rev7 approval request; they evidence approval of the *same content* but are **not silently relabelled** as an R2-Rev8 declaration. This subsequent explicit confirmation resolves the review-identity issue.

| Decision field | Formal outcome |
|---|---|
| Decision maker | Barry Levy, human product owner |
| Human declaration | **“yes”** to the explicit R2-Rev8 approval question above |
| Date | 2026-10-25 (Australia/Melbourne); time-of-day not asserted |
| Human gate | **R2-Rev8 CLOSED — APPROVED** |
| Approved change | CA-006, CA-007, consequential CA-008, RA-FD006-001 bounded residual risk, FD-009 resolution and CA-T01–16 against approved Scope v1.2 / AM-002; proposal/evidence/scenario identities in §1 |
| Retained approvals | Historical R1-Rev5 Scope, R2-Rev5 navigation and R2-Rev6 Functional Design approvals remain in place at their exact baselines; only the targeted changed outcomes supersede relevant prior clauses |
| Excluded | R3 approval or TD-008/010/011 closure; tests/build/deployment or new browser capability/architecture |

**Next authorised action:** Codex may preserve this signed-off independent R2-Rev8 report under `independent-reviews/2_design-functional/Review_8/`, reconcile **only** approved behavioural changes into affected operative UI/HTTP/data-model, review checklists/test plans and architecture handoff materials, maintain historical approvals and evidence, and submit the revised design for independent R3 assessment of TD-008, TD-010 and TD-011. R3 remains **NOT_READY / NOT APPROVED** until separately reviewed and human-approved. No Tasks, Analyze or Build authority is granted.
