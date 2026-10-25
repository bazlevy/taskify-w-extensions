# Taskify — R1-Rev5 Targeted Independent Scope Review: FR-010 / AC-010

**Review date:** 2026-10-25 (Australia/Melbourne)  
**Review ID:** **R1-Rev5** (targeted referral from R2-Rev7 / FD-009)  
**Finding ID:** **R1-AC010-001**  
**Independent R1 pre-incorporation verdict:** **TARGETED SCOPE AMENDMENT REQUIRED**  
**Subsequent human decision:** **AM-002 AUTHORISED FOR INCORPORATION** by Barry Levy on 2026-10-25; see §7.  
**Independent R1 post-incorporation verdict:** **READY_FOR_FINAL_HUMAN_SCOPE_APPROVAL** for the exact v1.2 / AM-002 candidate independently verified in §8.  
**Final human Scope decision:** **APPROVED** by Barry Levy on 2026-10-25 for the exact independently verified v1.2 / AM-002 candidate; see §9.  
**Current R1 stage:** **R1-REV5 CLOSED — HUMAN SCOPE APPROVED**; Codex's strictly metadata-only approval-status update and its hash check are still to be performed.  
**Scope gate:** **v1.2 / AM-002 CLOSED — APPROVED** against the exact §8 candidate SHA-256; v1.1 / AM-001 remains a preserved historical approved baseline.  
**R2-Rev7:** **NOT_READY**; FD-009 OPEN. R3 remains NOT_READY, TD-008/010/011 OPEN. No Tasks, Analyze or Build authority.

**Chronology:** §§1–6 record the initial independent assessment and pre-decision handoff; §7 records subsequent human amendment authorisation; §8 records independent verification of the actual Codex-incorporated candidate; §9 records the subsequent final human Scope approval. Later sections supersede earlier *stage/status* statements without erasing review history or changing earlier exact-baseline approvals.

## 1. Basis and authority

This is a written-artifact review, not implementation/runtime testing or an approval on Barry's behalf. Reviewed the attached `taskify-r1-ac010-handoff.md` and the relevant sources in `Archive.zip`:

| Source | Evidence and authority |
|---|---|
| Canonical `specs/001-personal-task-manager/spec.md` | Scope v1.1 / AM-001, SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca` independently verified. FR-010: task data not accessible through another account. AC-010: distinct accounts cannot “view or modify” each other's tasks. Product Boundary, Story 4 scenario 3 and independent test repeat private-list isolation. FR-012's already-open-tab exception concerns *view settings*, not task confidentiality. |
| `independent-reviews/1_scope/Review_3 /scope_review_report.md` | DEC-R1-003/005 expressly adopted private sign-in, account-specific lists and no other-user access; detailed authentication mechanics were delegated to Design. No specific stale-UI-display exception approved. |
| `independent-reviews/1_scope/Review_4/scope_review_report.md` §8 | Barry's explicit 2026-10-23 approval of Scope v1.1 / AM-001; FR-010/AC-010 were unchanged by AM-001. Review 4's earlier note that TD-001 privacy concealment was not risk-accepted is historical, not a prospective prohibition on an expressly approved new amendment. |
| `independent-reviews/2_design-functional/Review_6/Taskify_R2-Rev6_Targeted_Functional_Design_Review_APPROVED.md` | SHA-256 `bf5655e36c120dd79b35542430915de56817f4587245774abc50cad2ffafbf6f` verified. Barry approved strict fresh validation/reveal and browser-established login selection on 2026-10-24. These additional R2 obligations remain binding until formally amended. |
| `review-followups/r2-conventional-auth-amendment-proposal.md` §§2–5 and associated evidence/scenarios | Proposal SHA-256 `8199decf14eb62f063bab7db16bb4d98a54686783584b4cd1dc51aeb3370385e` verified. CA-T05–08 distinguish server owner/epoch protection and known-stale rejection from delayed display of previously authorised content. Proposed only; Barry's bounded risk direction dated 2026-10-25 is not Scope or R2 design approval. |
| `independent-reviews/2_design-functional/Review_7/Taskify_R2-Rev7_Targeted_Functional_Design_Review_NOT_READY.md` | FD-009 is OPEN / SCOPE_ESCALATION; R2 final gate explicitly blocked awaiting R1 disposition. |
| `independent-reviews/review-remediation-policy.md` | Distinguish approved requirements from later R2 guarantees, assess actual consequence/proportionality, pause repeated custom fixes, preserve review IDs/history and human gates. |

## 2. Counterexample and interpretation

1. Tab A is legitimately authenticated as Account A and has already displayed A tasks; alternatively, a response for A was authorised and remains in flight.
2. Another same-profile tab changes the browser's valid login to Account B. Tab A has not yet received a notification, resumed, focused, or otherwise observed that change.
3. Tab A may still show A tasks or complete rendering an A-authorised response. A person now using the browser under B can see that content. After observation/revalidation, the proposed design clears it.
4. Independently, B-authenticated requests cannot retrieve or mutate A records, an A-intended operation with B's cookie is rejected, and known stale/mismatched replies cannot be displayed.

The difference between **new unauthorised server access** and **previously authorised client display** is technically meaningful. It does not, however, settle the existing product promise: AC-010 expressly prohibits one signed-in account's user from *viewing* the other's tasks, not only from requesting them via the API. A reader can reasonably apply that promise to the visible Tab A content after B's login. Product Boundary, Story 4 and the original R1-005 decision reinforce the privacy intent. FR-012's no-live-update allowance is confined to the four view preferences. Consequently, **the approved text does not clearly permit this acknowledged display residual**; R2 cannot deem it allowed solely because server-side isolation remains sound or Barry accepted the risk in principle.

This is an ambiguity with a credible conflicting interpretation, not evidence of a demonstrated cross-account database/API vulnerability. The distinction supports a *narrow* amendment, not weakening FR-010 or adding browser-wide coordination.

## 3. Findings, risk and dispositions

| ID | Origin / type / phase | Risk and evidence | Action / authoritative status |
|---|---|---|---|
| **R1-AC010-001** (new; maps to **FD-009**) | **EXPLICIT** approved AC-010 privacy wording versus **PROPOSED** CA-006/RA-FD006-001; **AMBIGUOUS / CONFLICTING interpretation**; **SCOPE**; human approval relevant | **HIGH potential consequence**: disclosure of private tasks to a shared-screen/browser observer; potentially irreversible once seen. **Likelihood/exposure UNKNOWN**; unusual cross-tab transition plus unobserved state, not measured. **Confidence HIGH** in written conflict and timing counterexample, not in event frequency or an implemented defect. Delayed observation can be prolonged; no numeric exposure bound. | **RECONSIDER** proportionality of absolute display reading while preserving current binding obligations; **OPEN — BLOCKS R2-Rev7 final gate** pending Scope authority. Barry's bounded risk direction is recorded, not a Scope amendment or R2 final approval. |
| **FR-010 server access and mutation isolation** | EXPLICIT / SCOPE; protective invariant | Proposed CA-T05 and §3 require request authentication, account owner and intended-login checks; no evidence of a planned weakening. | **NO CHANGE RECOMMENDED**; remains binding; regression checks downstream. |
| **R2-Rev6 FD-006/007/008 and R2-NAV-001** | Human-approved Functional Design, subsequent proposed replacement | R2-Rev6 adds stronger browser winner/fresh-render obligations; their R2 origin does not cancel their current binding status. Revised CA-006/007 and consequential CA-008 require independent R2 gate. | **HISTORICAL APPROVAL PRESERVED; PROPOSED REPLACEMENT PENDING.** FD-009 remains OPEN under R2-Rev7. |
| **R3 TD-008/010/011** | TECHNICAL_DESIGN | Mechanisms and feasibility must be reassessed against whatever final R2 contract is approved. | **OPEN / NOT_READY unchanged.** No premature runtime-test requirement or design solution imposed by R1. |

**Established-practice/proportionality finding:** Standard authentication and owner-scoped protected requests are compatible with Taskify's product needs. Ordinary browser cookie ordering may select the last applied valid login after concurrent anonymous responses (RFC 6265 notes response-cookie races); this does not itself authorise cross-account task access. A browser-wide atomic render fence is materially stronger than request authentication and has driven repeated design cycles. Barry's explicit preference for conventional behaviour is reason to reconsider the *UI timing promise*, but not to waive the existing privacy criterion by implication. R1 does not certify a Django implementation or assert zero privacy harm.

## 4. Minimum proposed Scope amendment — **authorised for incorporation; not final Scope-approved**

**Proposal:** `AM-002`, provisional **Scope v1.2**. Keep **FR-010 exactly unchanged**. Replace only AC-010 with the following complete wording, resolving the broad “view” sentence rather than appending a potentially contradictory exception:

> **AC-010:** Given two distinct signed-in accounts, when either account makes an authenticated request to access or change personal tasks, the request must be authorised for that account and must not access or change the other account's tasks. Task operations initiated under a previous login must not be applied under a different login. A tab may continue to display, or finish rendering, task content previously authorised under its prior login while a login change in another tab of the same browser profile remains unobserved; no browser-wide zero-exposure timing or maximum observation delay is guaranteed. Once the change is observed, prior-login private content must be cleared or concealed, and responses known to be stale or mismatched must not be rendered. This limited display residual does not authorise cross-account API reads, mutations or reuse of previous-login task intent.

**Conforming edits, now authorised for reconciliation under §7:**

- **Story 4, Independent Test:** qualify the existing phrase that a different account's tasks/settings are “inaccessible” as *no access through that account's authenticated operations, subject only to the AC-010 previously-authorised display residual*.
- **Story 4, Acceptance Scenario 3:** qualify “the first account's task list is not accessible” as *inaccessible through the new account's authenticated requests; the narrow unobserved prior-authorised display case is governed by AC-010*.
- **Amendment register:** new `AM-002` entry with decision, exact reviewed baseline, human approval record and explicit residual-risk statement; do not alter original approval entries or backdate authorisation.

No other FR/AC change is justified by this targeted review. In particular, FR-008/009/012/013/014, AC-005/008/009 and R2-NAV-001 are unaffected. The exact notification, lifecycle revalidation, local response-generation checks and logout mechanisms remain **R2/R3** rather than separate new R1 requirements. Do not interpret this amendment as permitting known mismatched rendering, continued private display after observed identity change, cross-account requests, an old-intent mutation rebound or session resurrection.

## 5. Acceptance impact and required human decisions

**Acceptance boundary:** CA-T05 must still reject direct wrong-owner reads/writes and A-intent-with-B-cookie; CA-T06 must still clear on observed change/discard known mismatch; CA-T07 may accept prior-authorised stale display while an identity change remains unobserved; CA-T08 must conceal/revalidate at applicable entry, focus, resume and uncertainty boundaries. Test indefinite/unobserved notification loss without inventing a maximum duration. These are **planned checks**, not executed results. R2 checks the written functional scenarios; R3 owns mechanisms; Build/Test executes later.

**Human choice presented at the original pre-decision review (subsequently answered in §7):** Barry was asked to choose either:

1. **Authorise targeted AM-002 wording for canonical reconciliation** (reviewer recommendation), explicitly accepting that previously authorised A content may remain visible while B's same-profile login change is unobserved, potentially long enough for irreversible observation; or
2. **Retain literal AC-010** (no Scope change), in which case the proposed CA-006/RA-FD006-001 exception remains inconsistent and R2 must revise or reject that behaviour. No specific architectural remedy is prescribed.

Prior verbal risk acceptance is evidence of intent, **not** the formal decision on this exact Scope text. If Option 1 is chosen: Codex prepares a *pending*, tightly diffed canonical `spec.md` with the approved-for-reconciliation wording and provisional version; independent R1 verifies actual text, conforming edits and no regressions; Barry then gives **one final explicit human Scope approval** of the exact reconciled hash/version. Scope v1.1 remains the operative baseline until that gate. Do not declare the report or an unreconciled proposal itself Scope-approved.

## 6. Concise R1 → R2 handoff

**Handoff type B — targeted Scope amendment; human authorisation subsequently recorded in §7, final gate pending.** The R2 proposed conventional-auth contract is **functionally coherent as to server owner isolation, old-intent rejection, known-stale clearing and bounded residual display**, but its CA-T07/§3 display exception is not safely derivable from the approved AC-010 “view” promise. Proposed AM-002 resolves that one contradiction while leaving FR-010 and all unrelated requirements intact. R2-Rev7 FD-009 remains OPEN/NOT_READY until R1 canonical verification and final human Scope approval, then independent R2 performs only a targeted consistency/gate re-review and seeks Barry's separate written-design approval. R3 TD-008/010/011 remain OPEN/NOT_READY; no architecture or Build authority conveyed.

**Historical preservation:** This review does not edit the approved `spec.md`, previous independent reports/approvals, operative contracts or the supplied archive. The reviewer does not approve Scope or close any downstream gate.

## 7. Subsequent human amendment authorisation — 2026-10-25 (Australia/Melbourne)

**This section records the later human decision without retrospectively treating the initial assessment as post-incorporation verification.** The decision followed an explicit question whether Barry authorised *the proposed AM-002 clarification for incorporation, subject to independent verification and subsequent final Scope approval*. Barry answered: **“I approve the amendment.”** In that immediate context, this is authority for Codex to incorporate the precise §4 proposal, **not final approval of a canonical file that has not yet been prepared or verified**.

| Field | Decision / status |
|---|---|
| Authorised decision maker | Barry Levy, human product owner |
| Human declaration | “I approve the amendment.” |
| Decision date | 2026-10-25 (Australia/Melbourne); no time of day asserted |
| Decision ID | **DEC-AM-002-01** — authorisation to incorporate the narrow AC-010 stale-display clarification |
| Approved-for-reconciliation text | The **complete replacement AC-010 in §4** plus the identified minimum Story 4 conforming edits; the originally supplied R1 assessment is identified by SHA-256 `a29778d0f5b7ffd6dd1124da862a0c9e4ea8a12dcbf52e96ef8ce56b9c8638d8` |
| Explicit risk boundary | The previously-authorised stale-display/in-flight-response residual while a same-profile login change is unobserved; no maximum observation delay promised. No cross-account API access/mutation, old-intent rebinding, known-mismatch rendering or post-observation retention authorised. |
| FR-010 and unaffected requirements | **Remain unchanged and binding** |
| Canonical Scope v1.1 / AM-001 | **APPROVED, operative and unchanged**; SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca` is the independently checked source from the original R1 review |
| Proposed Scope v1.2 / AM-002 | **AUTHORISED FOR CODEX INCORPORATION; NOT YET RECONCILED OR VERIFIED** |
| R1 finding R1-AC010-001 | **OPEN — pending canonical reconciliation, independent verification and final human Scope gate**; severity and phase unchanged |
| Final human Scope v1.2 approval | **NOT GRANTED**; must identify the independently verified canonical file/version/hash |
| R2/R3 and later gates | R2-Rev7 FD-009 remains OPEN/NOT_READY; R3 TD-008/010/011 OPEN/NOT_READY; no Tasks/Analyze/Build |

### 7.1 Required next action — Codex incorporation

Codex may apply **only** the §4 AM-002 text and directly necessary conforming edits to canonical `specs/001-personal-task-manager/spec.md`, with a provisional Scope v1.2 / AM-002 document-control entry marked **PENDING final human Scope approval**. Preserve FR-010, all other unaffected requirements, historical approvals and the existing R2/R3 contracts. Return the exact diff, change/evidence map and SHA-256 of the revised canonical file. Do not mark the new Scope baseline approved or advance the workflow.

### 7.2 Required next action — independent R1-Rev5 post-incorporation review

Inspect the **actual amended canonical `spec.md`** against the §4 authorised text and prior Scope v1.1 hash. Verify AC-010, the Story 4 conforming edits, requirements/acceptance IDs, traceability and absence of privacy/security regressions or unrelated changes. Record source → finding → human decision → canonical change → verification evidence, then disposition R1-AC010-001 as warranted. Issue a **separate post-incorporation independent readiness recommendation** within this R1-Rev5 report. No such verification has yet occurred; no candidate hash or passing result is claimed.

### 7.3 Final human Scope gate and R1 → R2 handoff

After favourable independent verification, Barry must explicitly approve the **exact** canonical Scope v1.2 / AM-002 baseline. Only then may Codex mark its approval metadata and return the approved Scope to independent R2-Rev7 for targeted FD-009 consistency/gate re-review and separate human Functional Design approval. R3 then reassesses its still-open findings under any approved R2 changes. Historical R1-Rev4, R2-Rev5 and R2-Rev6 approvals remain attached to their original baselines.

## 8. Post-incorporation independent R1-Rev5 verification — 2026-10-25 (Australia/Melbourne)

**Authority and method:** This section is the later, independent written-specification review requested by §7.2. It supersedes the earlier statements that Codex incorporation and independent verification were pending; it does **not** amend or approve canonical `spec.md`. Inspected the user-supplied `Archive(1).zip`, directly recomputed content hashes, independently compared its candidate with the previously reviewed approved v1.1 bytes, compared the changed text with §4 and checked the included Codex diff/evidence. No application execution, browser testing, database testing or technical design verification occurred.

### 8.1 Exact source identities

| Source / evidence | Independently checked identity |
|---|---|
| Supplied `Archive(1).zip` | SHA-256 `746027c36afceafea5cbda09831bfc87be0d35d68a55387df654cfe7db0d22a1` |
| Previously approved, operative Scope v1.1 / AM-001 | `specs/001-personal-task-manager/spec.md`, SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`, independently checked in the earlier supplied archive and again in this review |
| Codex-incorporated **candidate Scope v1.2 / AM-002** | Canonical `specs/001-personal-task-manager/spec.md`, SHA-256 **`87bf1574788c46eedf6412782e1b6a60915a4db0d4b643483d9e79736a2c745c`**; exact file independently hashed |
| Codex exact diff | `review-followups/am002-scope-v1.1-to-v1.2.diff`, SHA-256 `e36032d67d907a676d784f7d789ebfc649d913db89225438a1ff270572f1ebf6`; independently regenerated with three context lines and **byte-matched** the supplied diff |
| Codex incorporation evidence | `review-followups/am002-scope-incorporation-evidence.md`, SHA-256 `90a4f4147490c825c888ea425652b0d7fce30ef0b60c450385c9cff64712efec`; treated as author assertion corroborated where independently checked, not proof by itself |
| Authorised amendment | R1-Rev5 §4 complete AC-010 replacement and Story 4 qualifications, human-authorised for incorporation under DEC-AM-002-01 in §7 |

### 8.2 Verification results

| Check | Independent finding |
|---|---|
| **Scope identity and provenance** | **PASS.** Candidate labels v1.2 / AM-002, records DEC-AM-002-01 as authorisation to incorporate, links R1-Rev5, preserves the v1.0/v1.1 approval history and clearly marks final v1.2 Scope approval as PENDING. No invented human approval or modified earlier decision. |
| **Authorised AC-010 wording** | **PASS.** The full substantive text matches §4 exactly; only Markdown list/ID punctuation differs (`**AC-010**:` in canonical versus `**AC-010:**` in the report quotation). It preserves owner-authorised requests, previous-login intent protection, bounded previously-authorised stale display/in-flight completion, no maximum observation delay, clearing upon observed change, known-mismatch rejection and no cross-account API/mutation permission. |
| **Story 4 independent test** | **PASS.** Only the account-isolation result is qualified: no access through the other login's authenticated operations, except the AC-010 prior-authorised display residual. Registration, cross-device persistence, session view and remaining test content unchanged. |
| **Story 4 scenario 3** | **PASS.** Qualified to authenticated-request isolation with explicit reference to AC-010. The other Story 4 scenarios are unchanged. |
| **Functional requirements** | **PASS.** All **16** uniquely numbered FRs are retained **byte-identical**, including FR-010 server/account isolation and FR-008/009/012/013/014 safeguards. |
| **Other acceptance criteria** | **PASS.** All **12** AC identifiers remain unique; the other **11** ACs are byte-identical to v1.1. No acceptance criterion removed or silently broadened. |
| **Unrelated product-definition text** | **PASS.** The full comparison shows only the three authorised substantive changes: AC-010, Story 4 independent test and Story 4 scenario 3. Product Boundary, remaining stories/edge cases, entities, success criteria and assumptions/exclusions are unchanged. |
| **Document-control changes** | **PASS.** Restricted to current/past version metadata, the AM-002 history entry and explanatory provenance. Scope v1.1 is explicitly preserved as the operative approval; R1-Rev4 history is not rewritten. |
| **Change isolation across project archive** | **PASS.** Of the 74 substantive files shared with the previous archive, only canonical `spec.md` changed. Three new substantive files are the R1-Rev5 report and the two Codex AM-002 evidence/diff artifacts; none of the previous substantive files are missing. Historical R1/R2/R3 reports, operative design contracts and remediation policy are unchanged. |
| **Diff integrity** | **PASS.** Regenerating the unified diff from the actual two specification byte streams produced a byte-identical result to Codex's supplied diff. |

**Boundary interpretation:** The unchanged Product Boundary/FR-010 still prohibits task access *through another account*. The newly authorised AC-010 distinguishes a previous login's already-authorised client display from data accessed through the subsequently selected login. This resolves R1-AC010-001's interpretive conflict without authorising wrong-owner API calls, old-intent task rebinding, known-stale rendering, or retained private state after an observed change. FR-012's view-setting exception remains separate from task-data confidentiality. No other material Scope contradiction or regression is identified in the actual v1.2 candidate.

### 8.3 Finding disposition, residual risk and readiness

| Finding / obligation | Verified disposition and boundary |
|---|---|
| **R1-AC010-001** — formerly OPEN, HIGH, SCOPE | **RESOLVED** for the exact candidate's written Scope contents, with **independent verification complete** and the human Scope v1.2 gate still pending. DEC-AM-002-01 → amended AC-010 / Story 4 → independently verified diff/hash is traceable. This is not an independent acceptance of the risk or approval of the Scope baseline. |
| **FR-010 / server account isolation** | **UNCHANGED / RETAINED.** No Scope permission for cross-account reads, writes or reuse of previous-login intent. Actual R2/R3 enforcement and Build/Test execution remain independently gated downstream. |
| **Residual RA-FD006-001** | Barry's human-authorised product direction is incorporated into the proposed Scope. Consequence remains potentially **HIGH confidentiality harm**, likelihood/exposure **UNKNOWN**, confidence **HIGH in the design counterexample**, with no fixed observation deadline and potential irreversible viewing. No incident frequency or runtime safety proof is asserted. |
| **R2 FD-009 / R2-Rev7** | **OPEN / NOT_READY** until Barry approves this exact Scope candidate and R2 independently rechecks its proposed functional contract. Human R2 approval remains separate. |
| **R3 TD-008/010/011** | **OPEN / NOT_READY**; no architecture accepted or technical finding closed. |
| **Historical R1-AM-004 documentary follow-up** | Unchanged historical **DEFERRED** status; no missing Section 13 reconstructed. |

**Independent R1-Rev5 verdict: READY_FOR_FINAL_HUMAN_SCOPE_APPROVAL.** All material authorised AM-002 product changes are incorporated into the actual canonical candidate and independently verified; **zero open material Scope-definition findings** remain within this narrow amendment. No further product choice is required before the final human decision. This is an independent *recommendation*, not the authorised human approval itself.

**Current Scope gate: OPEN — PENDING BARRY'S EXPLICIT APPROVAL** of **Scope v1.2 / AM-002** at canonical SHA-256 **`87bf1574788c46eedf6412782e1b6a60915a4db0d4b643483d9e79736a2c745c`**. Scope v1.1 / AM-001 remains operative until that decision. After approval, Codex may make a **strictly metadata-only** status/history update in canonical `spec.md`, report the resulting hash and show a no-product-change diff. Any substantive edit requires another targeted check. Then return to independent R2-Rev7 FD-009 review and its separate human Functional Design gate. R3 and later phases remain blocked as previously recorded.

## 9. Final human Scope v1.2 approval — 2026-10-25 (Australia/Melbourne)

**This is the authoritative subsequent R1-Rev5 human gate record.** Section 8 preserves the independent reviewer's prior `READY_FOR_FINAL_HUMAN_SCOPE_APPROVAL` recommendation and the then-pending gate; this Section 9 records the later owner's decision without rewriting the recommendation or claiming that the metadata-only post-approval update has already happened.

| Field | Authoritative decision |
|---|---|
| Decision maker | **Barry Levy**, human product owner |
| Human declaration (verbatim) | **“approved”** — direct response to the R1-Rev5 request to approve Scope v1.2 / AM-002 and the exact canonical hash listed below |
| Approval date | **2026-10-25 (Australia/Melbourne)**; no unsupported time of day inferred |
| Scope document | Canonical `specs/001-personal-task-manager/spec.md` |
| Approved version / amendment | **Scope v1.2 / AM-002** |
| Exact independently verified and human-approved candidate | SHA-256 **`87bf1574788c46eedf6412782e1b6a60915a4db0d4b643483d9e79736a2c745c`** |
| Independent R1 assessment | §8: **READY_FOR_FINAL_HUMAN_SCOPE_APPROVAL**, with the authorised AC-010 text, Story 4 qualifications, no other FR/AC changes and no new material Scope blockers independently verified |
| R1 finding R1-AC010-001 | **RESOLVED** — independently verified against the approved candidate; earlier OPEN history retained |
| Residual risk | Human-approved bounded prior-authorised stale-display/in-flight-response allowance while same-profile auth change remains unobserved, with no maximum observation delay; potentially serious confidentiality consequence. No cross-account API reads/mutations, old-intent rebinding, known-mismatch rendering or post-observation private-state retention permitted. |
| **Human Scope gate** | **CLOSED — APPROVED** for the exact v1.2 / AM-002 candidate |
| Historical Scope baselines | R1-Rev4 v1.1 / AM-001 and earlier approvals preserved against their original exact baselines |
| Unaffected downstream gates | **R2-Rev7 NOT_READY / FD-009 OPEN** pending separate targeted Functional Design re-review and human approval; **R3 NOT_READY / TD-008/010/011 OPEN**; no Tasks/Analyze/Build |

**Approval meaning:** Barry approves the Scope product definition independently verified in §8, including the precise AC-010 narrow stale-display allowance and its disclosed residual privacy risk. This closes the R1-Rev5 **Scope gate only**. It does not retroactively approve the proposed R2 conventional-authentication contract, select a technical mechanism, certify runtime behaviour or close R3 findings.

**Required Codex record update:** In canonical `spec.md`, record final human approval of **v1.2 / AM-002 on 2026-10-25**, reference this R1-Rev5 §9 record and the approved candidate hash. Restrict edits strictly to approval metadata/status and document-control text. Do **not** alter FRs, ACs, stories, assumptions, risk boundaries, operative R2/R3 designs or historical approval records. Return the exact resulting new SHA-256 and a precise no-product-definition-change diff for targeted integrity checking. The hash above remains the identity of the human-approved *product-definition candidate* even if metadata-only sign-off changes the file bytes.

**Next workflow:** Once the metadata-only status update is checked, provide the approved Scope v1.2 baseline and this R1-Rev5 record to **independent R2-Rev7** for narrow FD-009/CA-006/007/008 consistency and gate re-review, then obtain separate human R2 approval. Only subsequently should R3 reassess TD-008/010/011. No R3, Tasks, Analyze or Build authorisation is conveyed.
