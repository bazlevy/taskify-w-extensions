# Taskify — Independent R3-Rev9 Targeted Technical Design Review

**Review date:** 2026-10-24 (Australia/Melbourne)  
**Independent reviewer verdict:** **NOT_READY**  
**Human R3 gate:** **OPEN — NOT APPROVED**  
**Review boundary:** Assessment of Codex's Option A (preserved browser availability) feasibility/stop decision against approved R2-Rev6, and directly affected TD-008, TD-010 and TD-011. Design and evidence review only; no code, browser fixture, test execution or implementation.

## 1. Decision summary

**Assessment accepted as an honest bounded technical study, not a complete design.** The analysis identifies credible, requirement-relevant counterexamples to native Django cookie ordering (TD-008) and an uncoordinated after-arrival validation/reveal sequence (TD-010). It appropriately avoids declaring those particular approaches compliant, does not claim a mathematical impossibility theorem, and stops accumulating bespoke auth mechanisms after the human choice to preserve existing availability.

The proposed BroadcastChannel logout protocol is a credible *partial design* for TD-011: it defines an ACK-triggered message, epoch-scoped local clearing, safe delayed-message handling and suspended-tab gates. It does **not** yet satisfy the approved prompt-notification obligation in admitted environments where BroadcastChannel cannot be constructed or delivery fails; next-interaction validation is a security fallback, not a substitute for prompt runnable-tab UX. TD-011 therefore remains OPEN independently of TD-010.

**Overall:** R3 stays **NOT_READY** with TD-008/010/011 blocking and TD-006 a nonblocking Operations/R5 dependency. No mandatory Web Locks/IndexedDB, new availability restriction or weakening of FR-010, FD-006/007/008 is authorised. The next meaningful activity is a **narrow human trade-off decision** with applicable R2/R1 review where required, not another unbounded engineering revision under mutually constraining assumptions.

## 2. Evidence baseline, preservation and review scope

| Input/check | Independent result |
|---|---|
| Reviewed attachment | `Archive(3).zip`, SHA-256 `f71a7181acea370a50e7488018f0394b3a872339dc8215db2b0e87ea8e40479a` |
| Approved Scope | `specs/001-personal-task-manager/spec.md`, SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`; R1-Rev4 preserved |
| Approved Functional Design | R2-Rev5 / R2-NAV-001 and R2-Rev6 FD-006/007/008/Q1 approvals present and preserved; R2-Rev6 record SHA-256 `bf5655e36c120dd79b35542430915de56817f4587245774abc50cad2ffafbf6f` |
| Prior independent R3 | `independent-reviews/3_design-technical/Review_8/taskify_R3-Rev8_technical_design_review.md`, SHA-256 `24c41e8a0f536995239cb9df0197c007e729e190425976bea50046bfa1c539ed`, preserved |
| Submitted new assessment | `review-followups/r3-rev9-option-a-architecture-assessment.md` (ADR-AS-003), `r3-rev9-option-a-review-handoff.md`, and `r3-rev9-option-a.sha256` |
| Manifest integrity | **9/9 SHA-256 entries independently recomputed and MATCHED** |
| Change isolation | Of **63 shared substantive Markdown/manifest paths**, exactly `plan.md` and `technical-design.md` changed; the other **61** are byte-identical to `Archive(2).zip`. Both changes are top-level status/handoff metadata; design bodies unchanged. |
| Validation | Nine A9-01–09 scenarios are diagnostic/test **plans**, not executed results; no implementation, Tasks, Analyze, Build or deployment evidence is claimed |

The author says 72 pre-existing files (including other non-substantive entries), with two changed; the independent 63-path Markdown/manifest comparison supports the substantive preservation claim. No approved Scope/Functional Design contract or historical review was revised.

## 3. Technical findings and dispositions

### TD-008 — Browser-established login selection

- **Classification:** Design Gap. **Severity:** Medium, blocking. **Confidence:** High. **Disposition:** OPEN (retained).
- **Evidence:** ADR-AS-003 §2; A9-01–04. Native independent A/B anonymous POSTs can issue separate valid Django session cookies. A delayed second same-name Set-Cookie can overwrite the first browser-usable session irrespective of server commit order. A 409, account lock, pre-dispatch authenticated check, or JavaScript conflict message is insufficient once a competing Set-Cookie has been sent. The Fetch specification does not expose normal Set-Cookie response headers for JavaScript cancellation; RFC 6265 documents concurrent cookie response races.
- **Requirements:** Approved FD-007, bounded S2-DEC-001, FR-010/AC-010 and R2-NAV-001.
- **Independent judgment:** Counterexample credible for the examined native/cookie-only shortcuts. ADR correctly avoids claiming every conceivable alternative has been ruled out. The suggested shared anonymous bootstrap also has an unresolved fresh-profile concurrent-issuance problem. Neither prior bespoke family/handle proposals nor the Django-native Option B are now approved remedies.
- **Minimum remedy/exit:** One coherent issuance/selection/revocation and failure design must prove A9-01–04 and R2-6R-T01–04/16 without new unapproved capability or product relaxation. Otherwise ask for an explicit product trade-off, not another custom state-machine layer.
- **Owner/escalation:** Codex supplies feasibility and candidate design; Barry owns any proposed availability/FD-007 departure; R2 and potentially R1 independently review actual amendments.

### TD-010 — After-arrival identity validation and private reveal

- **Classification:** Design Gap. **Severity:** Medium, blocking. **Confidence:** High for the counterexample, not for universal impossibility. **Disposition:** OPEN (retained).
- **Evidence:** ADR-AS-003 §3; A9-05/06. A task payload arrives; a server validation returns identity A; another tab validly logs out A and logs in B while that reply is delayed; the earlier result then permits a newly exposed A payload if no coordination observes the intervening authority change. Local generation matching and extra independent polls do not by themselves eliminate the final interval.
- **Requirements:** FD-006, Q1, FR-010/AC-010, R2-6R-T07–10.
- **Independent judgment:** This is a real design-proof gap **under the current R2 wording**, which expressly calls for validation/reveal ordering under concurrent changes. However, distinguish (a) server-side wrong-account data access or task mutation, which must remain forbidden, (b) old already-painted DOM under accepted S4, and (c) a newly arriving private reveal after an auth change. Whether FD-006 requires strict browser-wide atomicity or a bounded check-before-reveal contract is a **product-level choice**, not something R3/Codex may silently reinterpret.
- **Minimum remedy/exit:** Provide integrated auth/paint fencing or seek a precisely scoped targeted R2 interpretation/amendment with explicit residual privacy analysis. Revalidate on focus/resume/actions; never rebind A intents to B; preserve ordinary same-login task-refresh stale-row/FD-001 rules.
- **Owner/escalation:** Codex, then Barry/R2 for any observable relaxation. Scope/R1 must assess actual FR-010/AC-010 consequences, not automatically approve privacy exposure.

### TD-011 — Cross-tab logout notification

- **Classification:** Design Gap. **Severity:** Medium, blocking. **Confidence:** High. **Disposition:** OPEN, partially specified.
- **Evidence:** ADR-AS-003 §4; A9-07/08. Proposed same-origin `taskify.auth.v1` BroadcastChannel event with `{schema, kind, login_epoch, event_id}`, sent after confirmed server logout ACK. Matching runnable tabs synchronously clear/conceal and navigate; delayed notices for old L cannot erase current N; lost ACK never triggers a false success broadcast. This is proportionate and broadly consistent with approved FD-008.
- **Residual independent gap:** ADR describes BroadcastChannel as *opportunistic*, and admits constructor/posting/delivery failure. The R2-Rev6 contract requires prompt notification/clearing of runnable same-login tabs without waiting for the next poll/action. If messaging is absent/lost while a tab remains runnable, the proposed next-interaction/focus fallback preserves some security but **does not meet that prompt UX requirement**. The missing case is not resolved solely by TD-010's private-reveal fence.
- **Minimum remedy/exit:** Demonstrate a proportionate notification fallback satisfying prompt behaviour within the approved availability boundary, or request targeted R2 clarification of the specific delivery-failure behaviour. Do not invent instant suspended-tab rendering or tab-ACK waits. Preserve server revocation authority and epoch-scoped action gates.
- **Owner/escalation:** Codex; Barry/R2 only if a required observable fallback changes.

### Retained findings

| ID | Independent R3-Rev9 disposition |
|---|---|
| TD-001 | Prior design-level RESOLVED, retained; re-evaluate relevant guarantees if its old supporting mechanism is replaced |
| TD-002–005 | Prior design-level RESOLVED; no new regression evidenced |
| TD-006 | OPEN, nonblocking, Operations/R5; Barry owns readiness/restore evidence |
| TD-007 | Prior design-level RESOLVED; maintenance/R4 follow-up retained |
| TD-008 | OPEN, BLOCKING |
| TD-009 | Prior design-level RESOLVED; optional-view isolation regression retained |
| TD-010 | OPEN, BLOCKING |
| TD-011 | OPEN, BLOCKING, partially specified notification mechanism |

FD-003/004's previously human-accepted bounded risks do not accept any of the above races. No new risk is implicitly accepted by approving an assessment document.

## 4. Proportionality and established alternatives

**Whole-system necessity:** A single-user-style personal task manager with ordinary Django authentication does not inherently require family/handle registries, bespoke credential transition protocols or periodic key rotation. Those layers arose from increasingly strict browser-level guarantees for unusual overlapping login/logout/reveal schedules. The requirements are currently approved, so engineering cannot simply ignore them; their marginal user value should nevertheless be weighed against the architecture cost.

**Established baseline:** Django PostgreSQL-backed sessions, standard CSRF, secure session cookies, server ownership/epoch guards, session-scoped view, established BroadcastChannel notification where available and concealed focus/history checks are the proportionate foundation. They cover many ordinary and security-critical paths, but native cookie ordering does **not** satisfy current FD-007 alone and unaffiliated client identity probes do **not** prove TD-010 under a strict atomic interpretation.

**Candidate requiring human choice:** Mandatory Web Locks supplies browser-origin coordination, but adds a potential fail-closed login/logout/reveal dependency on API availability and a stalled lock holder. Its actual lifecycle, abnormal termination and stale-header safety still need specification; browser compatibility is not proof of these guarantees. Optional locks with a native fallback leave the strict fallback unresolved. The earlier P1/IndexedDB and mandatory Web Locks decisions remain paused/rejected for this phase. Do not assume a Web Lock automatically cancels previously sent server responses or makes Set-Cookie conditional.

**Independent conclusion:** The chosen Option A feasibility assessment is sufficiently reasoned to justify **stopping further speculative architecture expansion**, but not to certify mathematical impossibility of all availability-preserving approaches. Under the current agreed stop rule, repeating comparable redesign attempts is less useful than bringing exact competing product/availability choices back to Barry.

## 5. Recommended next gate and human decision

No automatic R2/Scope amendment has occurred. The next requested action is **a targeted human product trade-off discussion**, using the existing ADR-AS-003 evidence rather than another unbounded Codex repair loop:

1. **Preserve strict FD-007/FD-006/FD-008** and consider an explicitly scoped mandatory browser-coordination/availability amendment (such as Web Locks) with clear fail-closed, stalls, browser coverage and recovery consequences. This still requires a completed R3 architecture and independent proof; approving the dependency alone does not close TDs.
2. **Preserve existing availability** and authorise a **narrow R2 feasibility/behaviour reassessment** of exceptional concurrent anonymous-login winner, validation-to-reveal timing and prompt-notification-unavailable fallback. Core server-side account isolation, invalidated-session rejection, intended-epoch write protection, confirmed logout, private-data concealment on required interactions and no cross-account task mutation remain non-negotiable unless explicitly escalated to Scope/R1. This is **not** preapproval of weaker privacy; exact residual risk must be described and accepted or rejected.

**Reviewer recommendation:** Given Taskify's proportionate-MVP objective and Barry's explicit preference to preserve availability, choose **targeted R2 requirements reassessment (Option 2) before mandating a browser coordination API**. The scope should be only the exceptional observable guarantees causing disproportionate complexity; maintain the ordinary flows and all substantive security/owner isolation. If Barry declines any relaxation and retains existing availability, keep R3 blocked and ask for a bounded, independently justified architecture exploration only on new evidence—not repeating the rejected state machines.

## 6. R3 exit and next authorised work

- **Now:** R3-Rev9 **NOT_READY**. TD-008/010/011 OPEN. Human R3 approval **not granted**. No Tasks, Analyze or Build.
- **If product trade-off requested:** Barry explicitly authorises a narrow R2 proposal/review (and R1 if Scope materially changes); independently verify amended functional contract and obtain human approval. Update operative artifacts while preserving audit history.
- **Then:** Codex selects a simplest coherent technical design satisfying the resulting approved baseline, with deterministic concurrency/logout/private-UI/recovery scenarios. Independent R3 re-reviews changes and regressions.
- **Only after READY and Barry's explicit R3 approval:** Advance to Spec Kit Tasks → Analyze → R4 → R5 and subsequent human build gate.

## 7. Primary technical sources

- [Django authentication](https://docs.djangoproject.com/en/5.2/topics/auth/default/) and [server-side sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/)
- [RFC 6265 §4.1.1](https://www.rfc-editor.org/rfc/rfc6265.html#section-4.1.1) — concurrent Set-Cookie races
- [WHATWG Fetch](https://fetch.spec.whatwg.org/) — forbidden Set-Cookie response headers for ordinary script response objects
- [W3C Web Locks API](https://www.w3.org/TR/web-locks/) — cross-context lock lifetime and termination, not a complete credential protocol
- [WHATWG HTML BroadcastChannel](https://html.spec.whatwg.org/multipage/web-messaging.html#broadcasting-to-other-browsing-contexts) — best-effort same-origin/storage-partition messaging

**Report boundary:** This document independently reviews the new ADR and handoff, not an adopted design. No hidden author-side test, gate approval, user choice or new requirement is inferred.
