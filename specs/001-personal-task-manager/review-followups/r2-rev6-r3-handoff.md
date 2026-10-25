# Targeted R3 Technical Design Review handoff — approved R2-Rev6

**Date:** 2026-10-24. **Functional gate:** R2-Rev6 CLOSED — APPROVED by Barry Levy.
**Technical gate:** R3 NOT_READY; TD-008 OPEN. No Tasks, Analyze, Build or implementation.
P1/IndexedDB and P1-A remain paused. This handoff neither selects an architecture nor claims a
complete technical correction. The inherited candidate still needs targeted proof/reassessment.

## Authority and review boundary

[R2-Rev6 record](../../../independent-reviews/2_design-functional/Review_6/Taskify_R2-Rev6_Targeted_Functional_Design_Review_APPROVED.md)
§1 identifies the exact candidate; §2 independently verifies FD-006/007/008 and Q1 at written
level; §3 records Barry's approval; §4 authorises incorporation and targeted R3 review.
The local record was already supplied at Review_6 and is retained byte-for-byte; SHA-256
`bf5655e36c120dd79b35542430915de56817f4587245774abc50cad2ffafbf6f`.
Its exact reviewed proposal/evidence/manifest hashes match the preserved local files.

Scope v1.1/FR-010 and R2-Rev5/R2-NAV-001 remain approved against their historical identities.
R2-Rev6 §2 found no demonstrated Scope amendment necessary; account isolation and task protection
remain binding. Earlier review findings/approvals and accepted FD-003/004 risks remain intact.
The operative [UI](../contracts/ui.md), [HTTP](../contracts/http.md),
[states](../data-model.md) and [validation guide](../quickstart.md) now carry the approved outcomes.

Use independent [R3-Rev7](../../../independent-reviews/3_design-technical/Review_7/taskify_R3-Rev7_technical_design_review.md)
and its [simplicity addendum](../../../independent-reviews/3_design-technical/Review_7/taskify_R3-Rev7_simplicity_reassessment_addendum.md)
as technical history, qualified by the subsequently approved R2-Rev6 outcomes. Earlier Option B
and P1 documents are assessment history, not an approved authentication solution. In particular,
natural last-response cookie selection is not evidence for FD-007 compliance.

## Required targeted technical assessment

| Approved outcome | Technical proof/remaining gap | Planned regression evidence |
|---|---|---|
| FD-007: first browser-established usable login wins | Demonstrate both actual header-delivery orders and reversed server completion, same/different account competitors, unchanged credential/account/epoch/nondefault view and usable logout. A 409/error followed by cookie replacement, even temporarily, fails. Neither a server lock nor redirect alone proves browser ordering. | R2-6R-T01/02/04; capture header/cookie/auth traces without secrets, winner usability and private response effects |
| S2 with FD-007 | Distinct still-valid late login may authenticate after original confirmed revocation; revoked and terminally rejected authority cannot revive. If C is already established, late B must not replace C. Distinguish revoking L from globally cancelling every anonymous attempt. | T03/04/16; old-credential replay, same-account fresh epochs/defaults, delayed login/logout headers |
| FD-008: logout UX and authority | Prove intended shared revocation before success plus prompt same-profile notification/clear/login navigation in runnable tabs. Prefer established browser mechanisms; choose/prove failure handling. Suspended/missed-notification tabs remain gated on resume/focus/history, without impossible simultaneous rendering or tab-ACK waits. Delayed L notice cannot revoke N. | T05/06/11/12/15; dispatch/handler/paint/navigation observations, lost notices, bfcache, suspension, independent device and uncertain ACK |
| FD-006: new private reveal | Prove current auth/account/epoch validation after payload arrival and before revealing private DOM/accessibility/account labels. Resolve races around validation/reveal and late cookie delivery; pre-dispatch-only checking is insufficient. Mismatch discards, uncertainty conceals, no old intent/draft binds to new epoch. | T07/08/09; blocked payloads/validation, known and unobserved changes, same-account new epoch, late replies and action queues |
| Q1 and retained guarantees | Gate all accepted private actions without per-keystroke checks. Preserve FD-001 valid-snapshot lock, exact retry/receipt/version rules, ordinary refresh-error stale rows vs auth concealment, silent isolated view failures, CSRF recovery and rekey/view continuity. | T09/10/13/14/15; retained NAV/AM001/R2-T05/T07/T12 and prior task/owner/concurrency fixtures |

The approved full 16 rows are copied unchanged into quickstart.md. They are planned acceptance
scenarios, not executed tests. Runtime proof belongs to separately authorised downstream work;
this handoff requests a technical design assessment, not test execution or Build.

## Artifact and change-control expectations

Reassess the smallest coherent technical design satisfying the approved outcomes. Current
family/handle/transition/maintenance descriptions remain explicitly unapproved inherited details;
this incorporation does not remove them, adopt Option B or resume P1. Technical documents now
identify where prior pre-dispatch claims and cookie ordering are insufficiently demonstrated.
Preserve unrelated capacity, receipt-key, throttle, optional-view and recovery/compatibility
decisions; technical-design.md §§3–8 remain byte-identical to the start of incorporation.

Barry Levy remains accountable for TD-006 Operations/R5 and TD-007 Maintenance/R4; Codex prepares
design/validation/handoff documents. Existing owners, exit conditions and downstream verification
requirements are unchanged. Neither ownership nor Functional Design approval is risk acceptance.

If the mechanism cannot meet browser ordering, reveal privacy or logout continuity, report the
specific technical blocker. Do not silently change product behaviour, browser coverage, storage
availability, task-data protection or accepted-risk bounds. Refer an actual required departure
to Barry before incorporating it. Do not reopen confirmed Q1 or R2-NAV-001 as missing permissions.

R3 exit still requires independent technical assessment and explicit human R3 approval. This
handoff closes no technical finding and supplies no readiness recommendation. See the
[incorporation evidence map](r2-rev6-incorporation-evidence.md) and
[SHA-256 manifest](r2-rev6-incorporation.sha256) for this package's exact identities.
