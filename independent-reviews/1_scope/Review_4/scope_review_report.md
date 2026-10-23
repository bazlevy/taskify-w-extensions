# Taskify — Independent R1-Rev4 Targeted Scope Amendment Review

**Amendment:** AM-001 — authenticated-session filter and sort view  
**Reviewer:** Independent R1 Scope & Requirements Reviewer  
**Review / clarification date:** 2026-10-23 (Australia/Melbourne)  
**Review type:** Targeted amendment; unaffected R1 requirements are not re-reviewed  
**Latest independent reviewer verdict:** **READY FOR HUMAN SCOPE AMENDMENT APPROVAL** (Section 7)
**Formal human amendment gate:** **CLOSED — APPROVED on 2026-10-23** for Scope 1.1 / AM-001, exact candidate hash `61062311e5d1423f1d13f485c555158964e09f0548d13263bfefa0f048447fd4` (Section 8)
**Canonical source of truth:** `specs/001-personal-task-manager/spec.md` — unchanged by this reviewer  
**Reviewer-only candidate:** `independent-reviews/1_scope/Review_4/spec_proposed.md`, SHA-256 `137ba7e14d60413c86ebda2abad30e79eb81c7567ce25d05c231a2838aeae8f1` — **NOT CANONICAL / NOT APPROVED**

## 1. Authority, baseline and review boundary

The supplied Archive.zip includes:

- Canonical Scope-approved `specs/001-personal-task-manager/spec.md`, SHA-256 `4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae` (also byte-identical to the supplied final R1-folder spec copy).
- Prior R1 approval and findings history under `independent-reviews/1_scope/`. Original Section 12 approved hash `53bb1c39db8433556ceb505bcafb18c9258879579a7458a1d204a54a2daa025e`. The user subsequently explicitly confirmed the metadata-only corrected `4ef...` baseline, recorded as Section 13 in prior work; this archive's report copy omits Section 13.
- R2-Rev3 independently reviewed and later human-APPROVED Functional Design. Prior FD-002/FD-005 corrections were verified; FD-003/004 are accepted residual risks. This historical approval remains valid for the design reviewed at that time, not the proposed amendment.
- R3-Rev1 Technical Design Review with **NOT READY**, TD-001–007 still OPEN, especially TD-002 and Addendum A. Addendum A captures human-endorsed direction but is **not** formal Scope amendment approval.
- Unapproved `review-followups/r1-session-view-amendment-proposal.md`, SHA-256 `9d2b631ba29e5b6f77406cd1266b77948dbe7c46e45f2cb933872c97c73bf1f3`, and dependent unapproved R2 session-view proposal.

**Product-change authority:** The original cross-session filter restoration, four sorting methods/directions, task persistence/sync, account privacy and saved-task-load protection are **EXPLICIT approved baseline** requirements. The narrower session view, within-login sort restoration and silent view-state fallback are **PROPOSED amendments** from TD-002/Addendum A, clarified directly by the authorised human on 2026-10-23. These clarifications authorise a reviewer proposal but do **not** constitute final approval of Codex's eventual canonical file.

**Scope boundary:** Only FR-012, a narrow FR-013 adjustment, AC-005 and directly conflicting narrative/entities/assumptions are in play. No full R1 redo, new product capability, task-persistence change, offline guarantee, reset workflow, sharing, auth-expiry policy, or new sort method/tie rule is authorised.

## 2. Review iterations and human clarification

**First targeted pass (R1-AM-Rev1):** Identified R1-AM-001–004, issued CORRECTIONS REQUIRED for the original unapproved amendment proposal. In the subsequent interactive clarification, the reviewer explained choices and the human made the following decisions.

| Review finding / origin / type / owner / severity | Human clarification (2026-10-23) | Reviewer proposal and required verification | Current authoritative status |
|---|---|---|---|
| **R1-AM-001**; PROPOSED; MISSING/AMBIGUOUS; SCOPE; MEDIUM | **DEC-AM-001-01:** Keep the selected view working if remembering it fails or is uncertain. **No save or restoration warning.** Later refresh/tab may show valid stored view or defaults. Preference convenience must not block task access. This supersedes the warning suggested by R3 Addendum A and both unapproved R1/R2 proposals. | FR-012, AC-005, Story 3/4 and edge-case edits in `spec_proposed.md`. Check that no false guaranteed restoration of an unconfirmed choice or view-error blocker survives. | **OPEN at pre-incorporation review; see Section 7 for verified final status** |
| **R1-AM-002**; PROPOSED; AMBIGUOUS; SCOPE; LOW | **DEC-AM-001-02:** Each still-valid authenticated browser/profile/device session has its **own** view. A distinct new login starts at defaults; revisiting an already-valid separate login restores its own saved view. Tasks still synchronise by account. | FR-012, AC-005, Story 4 and assumptions. Test two sessions active simultaneously without cross-session preference import. | **OPEN at pre-incorporation review; see Section 7 for verified final status** |
| **R1-AM-003**; PROPOSED; AMBIGUOUS; SCOPE; LOW | **DEC-AM-001-03 (Option A):** Treat status, priority, sort method and direction as one complete view. If any remembered component is invalid/missing/unreadable, reset **all four** to defaults, without notification. | FR-012, AC-005, edge cases/entities. Verify all-or-nothing fallback and ordinary first-use defaults. | **OPEN at pre-incorporation review; see Section 7 for verified final status** |
| **R1-AM-004**; approval-history evidence gap; SCOPE governance; MEDIUM | Human confirmed that `spec.md` is the source of truth, with readable version/amendment history, dates, rationale and approvals; detailed evidence stays in the independent report. SHA-256 is a technical exact-file check, not the human-facing version label. Treat missing Section 13 in the supplied report copy as documentary housekeeping, not a new product defect. | Add a concise version/amendment register to proposed spec; ask Codex to recover/reference authoritative Section 13 without overwriting original Section 12 or fabricating approvals. | **OPEN — historical record not recovered; see Section 7** |

**Status convention:** A decision can be **recorded** without the finding yet being `RESOLVED`. Findings become `RESOLVED` only when the canonical changes are incorporated and separately verified against the recorded human decisions. No finding is silently discarded or marked approved.

## 3. Recommendation and proposed requirements

The reviewer has prepared a complete **`spec_proposed.md`**, copied from the exact approved baseline and limited to the amendment's affected parts. It includes:

1. **Document control and concise AM-001 register:** proposed 1.1 version, proposed amendment ID, reason (TD-002 preference bootstrap blocked otherwise available tasks), affected IDs, human clarification decisions and **PENDING** approval; baseline 1.0 remains identified as approved. These are proposed display labels, not fabricated historical Git tags.
2. **FR-012:** a complete four-setting view within a still-valid login shared by tabs; restoration on refresh/new tab when valid settings are remembered; independent defaults for new logins and separate sessions; no live-update requirement for existing tabs; no warning or task-access block solely from view-state failures; full-default fallback if any remembered component is invalid. Task data and authentication protections still govern.
3. **FR-013:** unchanged four sorts/two directions, Due Date initial default, undated-last in both Due Date directions and priority/creation tie rules **only when Due Date is the primary sort**. Session restoration is now controlled by FR-012; other ties remain R2-owned.
4. **AC-005:** observable same-login restoration, independent sessions/new-login defaults, all-or-nothing fallback without warning, unconfirmed-save exception and preservation of AC-008.
5. **Related text:** historical clarification retained and labelled pending supersession; original authorised decisions preserved as history; Story 3/4 independent tests, Story 4 scenario 4, edge cases, Filter/Sorting entity boundaries, assumptions/exclusions aligned.

The agreed **no-warning decision is deliberate**. It differs from the earlier Addendum A and unapproved R2 proposal, whose restoration/save warning language must be revised through R2 after Scope approval. This reviewer neither approves R2's last-commit-wins concurrency policy nor mandates Django session-row locking or endpoint contracts at Scope.

## 4. Targeted regression assessment

| Existing requirement / history | Conclusion and downstream check |
|---|---|
| **FR-007, Story 3** | AND-combined status/priority filtering unchanged; preserve all selection options, current-view usability and empty-match feedback. |
| **FR-013, AC-006/007/011, prior R1-006** | Preserve exact four bidirectional sorts; undated-last for either Due Date direction; Due Date-only fixed secondary rules. No reintroduction of the old universal tie-break contradiction. |
| **FR-008, FR-010, SC-004, AC-009/010** | Task records remain account-persistent and cross-device synchronised. Only display view is session-scoped. Another account's task content/settings cannot leak. |
| **FR-009 vs view-state failures** | Existing task-operation/sync/load feedback remains required. No new preference warning is required; auth and task-data failures must not be disguised as view-state convenience failures. |
| **FR-014 / AC-008, FD-001** | Failed task loading still requires Retry/support and blocks task mutations, including uncertain retry, until a valid snapshot. Successful task data must not be blocked solely because view state is unavailable. |
| **FR-015 / session lifetime** | No new browser compatibility, login duration, remember-me or offline promise. Browser restart follows whether the authenticated login remains valid; tab close alone does not necessarily end it. |
| **R2-Rev3, FD-002/FD-005** | Account-preference GET/PUT/bootstrap, version conflicts and save reconciliation remain **historically approved design for the old Scope**, to be superseded only after controlled R2 revision. Preserve review history and accepted FD-003/004 risks. |
| **R3 TD-001/TD-002** | TD-001's cross-tab logout/account-switch stale-content exposure remains HIGH and OPEN; view reset is insufficient. TD-002 remains OPEN until a reviewed replacement design removes or handles the old preference bootstrap failure. TD-003–007 unaffected. |
| **Plan, research, data model, HTTP/UI contracts, quickstart and checklists** | All contain related old account preference and/or fresh-page sort assumptions; Codex updates them under the subsequent Design process, then R2 reviews changed behaviour and R3 verifies technical risks. |

### Representative amendment cases

- Valid view Completed/High/Title descending, successfully remembered → refresh/new tab in same login restores all four; existing tab need not live-update.
- Chrome and Safari logged in as the same account → independent initial defaults and subsequent independent remembered views, but identical synchronised saved tasks.
- Logout/login as same or different account → new session defaults, prior view never imported; existing TD-001 concealment/security obligations still apply.
- Remembering view selection fails/confirmation is lost → current selected view remains usable with no preference notification; later refresh reads actual complete valid remembered view if available, otherwise defaults.
- Any one remembered field invalid/unreadable/missing → all four defaults silently; task list usable if authenticated task load succeeds.
- Task snapshot fails while view settings are valid or unavailable → FR-014/AC-008 Retry/support and task-write protection, not a successful default empty list.

## 5. Residual risks and explicit limitations

- **Intentional convenience loss:** Later logins or distinct devices do not inherit the last view. Human accepted this direction; task sync remains unchanged.
- **Silent reset:** User may be mildly surprised by All/All/Due Date on refresh. Human expressly preferred no notice because view settings are low-consequence conveniences. Defaults may reveal more *authorised* tasks than a previously filtered display; cross-account task exposure remains prohibited.
- **Competing tabs:** Existing tabs may diverge; a later tab sees whichever complete view was actually remembered. Precise concurrency winner, acknowledgement and optional retry behaviour need independent **R2** assessment, not unilateral R1 invention.
- **Authentication security:** Failed authentication must prevent private data access. TD-001's stale-render concealment remains unresolved and is not risk-accepted by this amendment.
- **Approval provenance:** The current supplied R1 history copy omits later Section 13, even though the metadata-corrected baseline was explicitly confirmed. Codex must trace the authoritative record; neither this report nor a new spec version should rewrite prior approvals.

## 6. Pre-incorporation independent reviewer disposition and gate

**Verdict:** **READY FOR CODEX INCORPORATION, SUBJECT TO TARGETED VERIFICATION OF THE RESULTING CANONICAL SPEC.**

All **material product clarifications** found in R1-AM-001–003 have explicit human decisions and are represented in the reviewer-only proposed specification. R1-AM-004 is documentary housekeeping. The affected requirements and downstream regression boundary have been reviewed; no further fundamental product decision or uncontrolled Scope expansion was identified in this targeted pass.

**This is not READY_FOR_FINAL_HUMAN_APPROVAL yet.** The canonical file has not been changed, the proposed version is unapproved, and independent verification cannot attest to changes Codex has not made. The four findings remain OPEN pending reconciliation/evidence; the final disposition table should be updated in this review only after the actual Codex result is checked.

### Controlled handoff and exit conditions

1. **Codex** reads this report and `spec_proposed.md`, compares them with current canonical `spec.md`, and implements only human-decided AM-001 changes in the canonical file. Keep a clear version/amendment register including rationale, affected requirements, approval status, dates and references. Preserve earlier Scope approval and original review history; check missing Section 13 provenance. No automatic approval.
2. **Independent R1 reviewer** inspects the changed canonical file and its diff, checks FR-012/013, AC-005, every affected narrative/assumption, immutable task guarantees and amendment decision trace. Disposition R1-AM-001–004 with source → decision → change → verification evidence; record exact file version/commit/hash as a technical verification reference.
3. **Human Scope owner** explicitly approves that exact reconciled canonical Scope version and AM-001 (or requests correction). Only then is the new baseline formally APPROVED; do not predate approval to the clarification discussion.
4. **Codex** updates affected Spec Kit Plan/checklists and Functional Design source artifacts; independent targeted R2 amendment review and human R2 approval follow. Then correct remaining R3 findings and conduct R3-Rev2. No Tasks/Analyze, R4/R5 or Build authorisation is implied.

**Role integrity:** This reviewer may propose the full specification, but **never writes the canonical repo `spec.md`**. Codex authors it; the reviewer independently verifies; the human approves. The review output is stored exclusively under `independent-reviews/1_scope/Review_4/` for repository incorporation. No approved artifact, existing gate, or prior finding disposition has been changed.

## 7. Post-incorporation independent targeted verification (2026-10-23)

**Authority:** This is the latest R1-Rev4 assessment and supersedes Section 6's earlier *pre-incorporation* readiness recommendation. It does not change or erase the original R1/R2 human approvals, this review's initial findings, or Codex-owned canonical artifacts.

### 7.1 Evidence and exact candidate

- **Input package:** User-supplied `Archive(1).zip` containing Codex's incorporated canonical `specs/001-personal-task-manager/spec.md`.
- **Prior approved Scope:** `independent-reviews/1_scope/Review_3 /spec.md`, independently SHA-256 checked as `4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`.
- **Incorporated canonical candidate:** **Scope version 1.1 / AM-001**, independently SHA-256 checked as **`61062311e5d1423f1d13f485c555158964e09f0548d13263bfefa0f048447fd4`**. This identifies the exact **unapproved** candidate reviewed, not a formal approval.
- **Reviewer proposal:** `spec_proposed.md`, SHA-256 `137ba7e14d60413c86ebda2abad30e79eb81c7567ce25d05c231a2838aeae8f1`. The canonical candidate's content is **byte-identical from `## Product Boundary` to end of file**; the only differences from this proposal are its intentionally distinct, Codex-authored document control and amendment-history section.
- **Method:** Independent content-diff inspection, requirement/criterion ID comparison, and representative failure, session, sorting, security and task-data regression analysis of written Scope. No application or executable test was performed.

### 7.2 Change integrity and targeted checks

| Check | Result and evidence |
|---|---|
| Numbered requirements | **PASS:** 16 FRs retained with unique IDs; only **FR-012 and FR-013** changed. No FR added or removed. |
| Acceptance criteria | **PASS:** 12 ACs retained with unique IDs; only **AC-005** changed. No AC added or removed. |
| Success criterion | **PASS:** SC-004 text unchanged. |
| Other guarantees | **PASS:** FR-008/009/010/014, AC-008/009/010 and SC-004 byte-identical to approved baseline. |
| Selected view and silent failure | **PASS:** FR-012/AC-005 apply view immediately, permit uncertain save with current view retained and **no view-state warning**, restore only confirmed complete valid settings or use defaults; task actions are not blocked solely by view-state failure. |
| Distinct active sessions | **PASS:** New logins/defaults and independent still-valid sessions are explicit; tabs sharing a valid authenticated session restore its last complete valid saved view without mandatory live updates. |
| All-or-nothing fallback | **PASS:** Missing, unreadable, invalid or incomplete component resets all four settings silently. |
| Sort behaviour | **PASS:** Four methods, two directions, default Due Date earliest first, undated-last, and Due Date-only secondary ordering preserved. |
| Scenario and narrative impact | **PASS:** Story 3/4 tests, Story 4 Scenario 4, edge cases, entities, clarifications and assumptions align; earlier cross-session requirement preserved as labelled history pending formal supersession. |
| Version/amendment provenance | **PASS WITH DOCUMENTARY EXCEPTION:** Canonical header and register disclose 1.1 / AM-001 and PENDING approval, identify prior original and metadata-corrected hashes, list decisions, and do not invent an absent Section 13 approval record. |

### 7.3 Finding disposition and remaining issue

| Finding | Status after independent verification | Rationale |
|---|---|---|
| **R1-AM-001** | **RESOLVED** | DEC-AM-001-01 reproduced in FR-012, AC-005, narrative and edge cases; no warning or unwarranted task block. |
| **R1-AM-002** | **RESOLVED** | DEC-AM-001-02 restored view only within still-valid browser login; distinct valid sessions independent, saved tasks unchanged. |
| **R1-AM-003** | **RESOLVED** | DEC-AM-001-03 complete-default silent fallback across all four settings verified. |
| **R1-AM-004** | **OPEN — non-product documentary exception; human disposition requested** | Original Section 12 approval verified; no authoritative Section 13 record located in provided history. Later metadata-corrected baseline is corroborated by human decision history and R2-Rev3's identified approved Scope hash. Codex documented the limitation without fabricating evidence. Human may accept the bounded record gap while approving this new precise baseline, or ask for further historical retrieval. |

**Residual risks:** Silent view reset and separate-login defaults are deliberate, human-directed convenience trade-offs. Concurrent tabs' precise write winner and acknowledgment mechanics remain **FUNCTIONAL_DESIGN**, with no new Scope demand for live tab synchronisation. The previously accepted FD-003/FD-004 Functional Design risks persist. R3 TD-001 and TD-002 and other open Technical Design findings **remain open**; AM-001's Scope reconciliation does not resolve or approve Technical Design. Saved-task-load protections, account privacy and persistence are unchanged.

### 7.4 Readiness recommendation and human decision required

**Independent R1 verdict: READY FOR HUMAN SCOPE AMENDMENT APPROVAL (with transparent historical-documentary exception).** There are **zero open material product-definition findings** for AM-001. All three product decisions were authorised by the human and have now been incorporated and independently verified. The only outstanding finding, R1-AM-004, concerns the missing historical Section 13 evidence copy; it has not been silently resolved or accepted on the human's behalf.

**Gate status when this verification was issued: PENDING HUMAN APPROVAL.** The subsequent authoritative human approval is recorded in Section 8. After approval, Codex may make a **strictly metadata-only** record of the sign-off in canonical `spec.md`; verify/report its resulting new hash without repeating the complete R1 assessment. Any substantive change requires a new targeted check.

**Next-phase boundary following approval:** Codex updates affected Spec Kit Plan/checklists and Functional Design artifacts; independent targeted R2 and human R2 sign-off follow; then R3-Rev2 examines the updated technical design and unresolved TD findings. Prior approval records stay intact. No R3 approval or build authorisation is implied.

## 8. Final human Scope amendment approval — 2026-10-23 (Australia/Melbourne)

**This is the authoritative R1-Rev4 human gate record.** Sections 2 and 6 show the earlier pre-incorporation state; Section 7 shows the subsequent independent verification and readiness recommendation. This Section 8 records the later human decision without rewriting those historical review steps.

| Field | Authoritative decision |
|---|---|
| Decision maker | Barry Levy — human product owner |
| Human declaration (verbatim) | “In that case, yes. I approve Taskify scope version 1.1. Let's move ahead to design.” |
| Approval date | 2026-10-23 (Australia/Melbourne) |
| Scope document | Codex-owned `specs/001-personal-task-manager/spec.md` |
| Approved version and amendment | **1.1 / AM-001** |
| Approved exact candidate | SHA-256 **`61062311e5d1423f1d13f485c555158964e09f0548d13263bfefa0f048447fd4`** |
| Independent verification | Section 7: **READY FOR HUMAN SCOPE AMENDMENT APPROVAL**; FR-012, FR-013, AC-005 and related text verified; all unaffected numbered requirements and task guarantees preserved |
| Current R1-AM finding dispositions | R1-AM-001, -002, -003: **RESOLVED**. R1-AM-004: **DEFERRED**, non-product archival-evidence gap noted transparently, with no claim that the absent Section 13 was recovered or that its risk was separately accepted |
| **Human Scope gate** | **CLOSED — APPROVED** for this exact 1.1 / AM-001 candidate |
| Design authority | Permits controlled updates of affected Plan/checklists and Functional Design artifacts; does **not** approve a revised R2 or R3 Design |

**Meaning of approval:** The owner approved the canonical candidate independently checked in Section 7, including DEC-AM-001-01–03 and the changed FR-012/013 and AC-005, while preserving all unchanged Scope requirements and prior human approvals. The previously disclosed gap in archival proof of the old *metadata-only* approval was not made a condition of this approval; it remains recorded as a nonblocking historical-documentary follow-up. No new product behaviour is authorised by this disposition.

**Required Codex record update:** Mark Scope 1.1 / AM-001 human-approved on 2026-10-23 in the canonical `spec.md` header and version/amendment history, referencing this Review 4 Section 8 and the approved candidate hash. Restrict the change to approval metadata and document-control wording; do not change FRs, ACs, scenarios, assumptions, or product rules. Compute the resulting new byte hash and report a targeted no-substantive-change diff for reviewer verification. Never retroactively alter original approvals or invent Section 13. The candidate hash above continues to identify the *approved product-definition contents* even when this approved-status metadata produces a different file hash.

**Next workflow:** Once the metadata-only update is verified, Codex regenerates/reconciles the affected Spec Kit Plan and checklists and updates Functional Design documents to match AM-001. Targeted independent **R2 amendment review**, explicit human R2 amendment approval, and then **R3-Rev2** follow. The existing historical R2-Rev3 approval remains intact for its original design baseline; R3's prior open TD findings remain open. Build is not authorised.
