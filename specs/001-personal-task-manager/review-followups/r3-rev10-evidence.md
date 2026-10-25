# R3-Rev10 correction and evidence map

**Date:** 2026-10-25. **Author recommendation:** submit ADR-AS-004 for targeted independent R3
assessment. **Gate unchanged:** R3 NOT_READY; TD-008/010/011 OPEN. No independent resolution,
runtime result, risk acceptance or Build authority is claimed.

## Authority and provenance

Scope v1.2 / AM-002, FR-010/AC-010 and R2-Rev8 are binding. Sources:
[Scope](../spec.md), [R2-Rev8 approval §§1–4](../../../independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md),
[operative UI](../contracts/ui.md), [HTTP](../contracts/http.md),
[approved incorporation](r2-rev8-incorporation-evidence.md) and
[Review Remediation Policy](../../../independent-reviews/review-remediation-policy.md).
[R3-Rev9 report](../../../independent-reviews/3_design-technical/Review_9/taskify_R3-Rev9_technical_design_review.md)
is preserved, including its prior blocker classifications and reasoning under stronger R2 wording.

The stopping-rule reassessment has occurred: prior custom architecture expanded to meet a first
winner and validation/render fence; independent R3-Rev9 identified disproportionate complexity;
Barry then authorised targeted R2/R1 amendments, independently reviewed and approved as
AM-002/R2-Rev8. Rev10 uses that new authority to simplify, not another incremental fence patch.
No newly disputed requirement requires further RECONSIDER/human waiver here.

## Per-finding qualification and proposed remedy

Original classification for all three: **Design Gap, Medium, blocking; High reviewer confidence;
OPEN**, from R3-Rev9 §3. Author recommended action remains **BLOCK** for substantiated retained
design gaps until independently assessed; this does not preserve superseded requirements or
close/reclassify historical findings. The proposed corrections supply design evidence only.

| ID | Retained requirement, failure and impact | Qualification and minimum correction | Design / planned regression evidence |
|---|---|---|---|
| TD-008 | CA-007/R2-NAV-001/CA-008; FR-010/AC-010 and CA-T15 retain owner/intended-login enforcement, true revocation and task/logout ordering. Cached auth can commit after logout; delayed native cookie deletion can erase N; absence-only logout success cannot prove L. Impacts account privacy/integrity or recoverable continuity, depending on path. First anonymous winner is no longer a requirement. | Wrong-owner/old-intent consequences potentially HIGH; cookie erasure/unknown ACK usually recoverable continuity. Exposure UNKNOWN, design-counterexample confidence HIGH; no incident rate inferred. Native SessionStore supplies credential/session validity, but task transaction and truth-of-ACK need application integration. Use outer Session guard, small logout receipt, cookie-response adapter. Remove family/handle/transition/maintenance layers; do not add election. | Lifecycle §§2–5; technical design §1; CA-T01–05/10/13–15, X01–03; V08/V06. Two lock orders prove no post-revocation command commit; cookie-writer matrix excludes stale deletion; receipt proves exact committed L. |
| TD-010 | CA-006/Q1/AC-010: wrong-account API/old intent, known mismatch rendering or retained private state after observed change prohibited. Delayed replies may otherwise restore A state after N validated. Unobserved prior-authorised rendering is explicitly accepted, including in-flight completion. | Retained privacy/integrity harm potentially HIGH, likelihood UNKNOWN, confidence HIGH in reasoned stale-response trace. Conventional owner/epoch checks plus local generations and concealed lifecycle boundaries address retained failures. Additional auth probes cannot produce atomic render; that old obligation was formally removed. | Lifecycle §6; technical design §2; CA-T05–09/13/15; V01/NAV/AM001. Captured generations distinguish stale callbacks from current identity observations; FD-001/task and auth uncertainty are separate. |
| TD-011 | CA-008: after confirmed intended logout dispatch promptly; receiving runnable same-login tabs clear/navigate; late L must not affect N. Unknown logout must not be broadcast as confirmed. Delivery loss alone is no longer a blocker under R2-Rev8. | Failure to clear after received signal can expose private state; false confirmed ACK misleads user. Potential confidentiality HIGH; transport-only UX impact lower; likelihood UNKNOWN, confidence HIGH in protocol reasoning. BroadcastChannel is established optional transport, not guaranteed delivery. Reuse epoch-scoped event/handler, safe unknown/newer-login rules and lifecycle/poll discovery. | Lifecycle §7/§5; CA-T10–14; V01/R7-T04–07. Dispatch follows exact committed/replayed ACK; unknown outcomes never signal success; dropped/unsupported delivery follows approved fallback without new capability. |

## Simplicity, costs and limitations

| Mechanism | Why retained / cost containment |
|---|---|
| Native SessionStore and auth | Single authority; reuse supported password/CSRF/fixation/lifecycle. No external provider/cache/new service. |
| Epoch + Session-row guard | Guards old intent and serialises against revocation; uses existing row/database and short bounded transactions. Read-only in-memory facade lets native auth flush invalid validation state without GET database writes; rejects unsupported rekey. Adds interface/lock compatibility fixtures and contention measured later under V03. No alternate crypto, account-wide login exclusion or network-held lock. |
| LogoutReceipt | One compact record per confirmed logout, token-hash proof only; no live-session registry or task data. Necessary to distinguish committed logout from expiry/absence after lost ACK. Bounded cleanup and R5 restore rule; proof loss stays unknown rather than false success. |
| Cookie-response adapter | Restricts native response save/delete rather than inventing credentials or conditional cookies. Needed to prevent delayed logout/401 deleting N and post-guard save. Framework coupling requires explicit maintained-patch/upgrade fixtures. |
| Local generations + lifecycle | Existing memory state, no persistent private journal. Reject known stale and clear on observation. Accepts only existing RA-FD006-001 unobserved display limitation. |
| BroadcastChannel | No storage, durable queue, fallback service or browser availability gate. Prompt dispatch and receiving-handler action; no delivery/ACK/atomic guarantee. |

Periodic maintenance rotation has no retained requirement and is removed. A future live rekey
must demonstrate preserved view/epoch/expiry and response ordering under TD-007 before deployment;
the candidate does not enable a hidden auth-hash fallback path. No native-security protection is
waived. Retained operational risks are assessed/measured at existing gates, not silently accepted.

RA-FD006-001 remains approved with potential HIGH confidentiality consequence, UNKNOWN exposure,
HIGH counterexample confidence and no maximum observation delay. No extension to server APIs,
known mismatches or old-intent effects. FD-003/004 remain unchanged. New numerical session expiry,
SLA, guaranteed message fallback or browser feature requirement is not proposed.

## Changed artifacts and preservation

| Artifact | Change |
|---|---|
| auth-transition-protocol.md | Replace prior unapproved mechanism with complete ADR-AS-004 lifecycle, guards, ordering, wire recovery, client and notification contract; native-source links. |
| technical-design.md | Integrate Rev10 decision and finding summary in §§1–2; retain §§3–8 verbatim. |
| data-model.md | Native Session payload/outer lock order and minimal LogoutReceipt; historical custom entity tables remain explicitly nonoperative. Existing task/command receipt/throttle/view field definitions retained. |
| plan.md / research.md | Select unapproved Rev10 candidate and consistent dependencies/lock order; preserve stack, product decisions and historical checks. |
| quickstart.md | Link deterministic companion; align V08 no-maintenance and V06 receipt clearing. Approved CA rows remain verbatim. |
| checklists/feature.md | Append unchecked CHK069–074; preserve all previous rows/markers. |
| review-followups/r3-pre-rev10-auth-protocol-snapshot.md | Retain original protocol body with only archive-relative links adjusted and a historical wrapper. |
| review-followups/r3-rev10-validation.md | All 16 CA schedules, X01–03 integration assertions, prior-test mapping, execution ownership and exits. |
| This map / r3-rev10-handoff.md / r3-rev10-changed-files.sha256 | Targeted author evidence, independent-review request and exact affected/input identities. |

Approved Scope/UI/HTTP and independent reports must remain byte-identical to the starting
workspace. Prior manifests/evidence remain attached to their historical versions; do not replace
them to suggest those older files had today's hashes. R2-Rev7 NOT_READY remains untouched.
TD-001's resolved account-isolation substance is revalidated under the replacement mechanism;
no prior finding status is reset. TD-006/007 retain Barry's accountability and existing R5/R4 exits,
with receipt restore/adapter-compatibility fixtures added to the handoff.

## Verification and decisions

Design evidence is the explicit lifecycle, two lock-order cases, restricted cookie writers,
receipt truth conditions and local generation/message rules. Deterministic runtime tests remain
DEFER to authorised Build/Test, with Codex preparing plans and Barry retaining release/phase
decisions. This is not a deferred unresolved mechanism or claim of executed proof.

Static author checks and hashes are recorded with the delivery. No material remaining product
decision identified; independent R3 may identify a concrete design defect or actual product
departure. Escalate that exact issue rather than reinserting removed guarantees. Only independent
review and explicit human R3 approval can change the current gate. No Tasks/Analyze/Build.

Static results: all 73 protected inputs (including 28 independent review Markdown records,
canonical Scope, approved UI/HTTP contracts and Spec Kit/agent/constitution inputs) retain their
starting bytes. Technical-design §§3–8, approved CA-T01–16 rows and all previous checklist text
are unchanged. Eleven affected Markdown files pass local-link, table/fence and whitespace
inspection; CA-T01–16 technical rows are complete and unique; git diff --check passes.
These checks inspect documentation only and are not application test execution.
