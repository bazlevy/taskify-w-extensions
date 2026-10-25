# R2-Rev6 approved Functional Design incorporation — change/evidence map

**Date:** 2026-10-24. R2-Rev6 APPROVED by Barry Levy; R3 NOT_READY, TD-008 OPEN.
P1/IndexedDB paused. Documentation reconciliation only; no Tasks, Analyze, Build or runtime tests.

## Approval evidence

The attached record is already present at
[Review_6/Taskify_R2-Rev6_Targeted_Functional_Design_Review_APPROVED.md](../../../independent-reviews/2_design-functional/Review_6/Taskify_R2-Rev6_Targeted_Functional_Design_Review_APPROVED.md).
It is retained byte-for-byte, not regenerated or amended. Record SHA-256:
`bf5655e36c120dd79b35542430915de56817f4587245774abc50cad2ffafbf6f`.
Its §2 independent written verification and §3 human approval are distinct from this author
incorporation. The exact approved proposal, evidence map and proposal-manifest hashes in §1
all match local files; their historical pending-status prose is retained as audit history.

| Reviewed identity | Verified SHA-256 |
|---|---|
| r2-rev6-remediated-functional-design-proposal.md | de6cd300e6971fe24b243a0ce01eb7484367647efc2889673cdac0b4cda44757 |
| r2-rev6-remediation-evidence.md | 764f16b8438886c56d629009432b1bf719d49ef24942f07fe9824534d2bc725e |
| r2-rev6-remediation.sha256 | 8d1fc84287dfb76638e4af586d19dff4e588dea31b68c6f60c5fcd6ed0adec63 |

## Per-finding incorporation

| Authority / requirement | Incorporated operative evidence | Validation / residual |
|---|---|---|
| FD-007; FR-010/AC-010, FR-012/013/AC-005 | UI concurrent-login clause and HTTP auth/conflict rules protect first browser-established credential/account/epoch/view, including same account; visible conflict cannot mask cookie replacement. Data-model login lifetime agrees. | Approved T01/02/04; both header orders and server-completion orders. Actual ordering mechanism remains R3 proof, not resolved here. |
| S2-DEC-001 + FD-007 | UI/HTTP/state rules permit a distinct still-valid late login after original revocation, forbid revoked/rejected revival and protect any subsequently established login. Fresh defaults vs winner-view retention explicit. | T03/04/16; no universal product promise to cancel every pending anonymous attempt. |
| FD-008 / S4-DEC-001; FR-010 | UI logout and state rules require prompt notification/clear/login navigation for runnable same-login tabs; concealed resume/focus/history despite suspension/loss; no other-device revocation or newer-login logout from stale intent/notice. HTTP confirms intended revocation before ACK and preserves uncertain outcomes. | T05/06/11/12/15. Notification scheduling/failure mechanism is R3-owned; no simultaneous suspended rendering or tab-ACK wait. |
| FD-006 + accepted Q1; FR-010/AC-010, FR-009/014 | UI/HTTP validate current identity after private payload arrival and before reveal, discard mismatch and conceal uncertainty; all Q1 cached/request actions gated, no per-keystroke auth. Browser states retain old-intent clearing. | T07–10/13. R3 must prove response/validation/render ordering and service epoch guards. |
| R2-NAV-001; Scope/AM-001 and prior FD guarantees | GET 302/authenticated POST 303/CSRF 403 remain; authenticated navigation retains view. Task schemas/versions/receipts, FD-001 locks, FD-003/004, silent view fallback/isolation and independent sessions preserved. | T10–15; exact accepted 16 rows copied to quickstart; no runtime passes claimed. |

## Changed artifacts

| Artifact | Targeted change |
|---|---|
| contracts/ui.md | Approved authority; concurrent winner/conflict/S2, Q1 inventory, after-arrival gate, logout UX/loss/uncertainty and privacy states; legacy mechanism references qualified. |
| contracts/http.md | Approved authority and wire-visible auth/409/redirect rules, bounded S2, intended logout ACK, private epoch/response validation; inherited routes explicitly unapproved. Task wire payloads unchanged. |
| data-model.md | Logical login/view/revocation lifetime and browser authority states; physical tables/fields unchanged, old mechanism not proof of new outcomes. |
| research.md / plan.md | Record approved decision provenance and governing boundaries; remove obsolete current pending-policy metadata, keep P1 paused and technical proof open. Historical Plan check retained. No Plan rerun. |
| quickstart.md | Incorporate all 16 approved R2-6R scenario rows verbatim; mark prior technical schedules as inherited inputs governed by current functional outcomes. Future command examples not run. |
| auth-transition-protocol.md / technical-design.md | Governing R2-Rev6 overlay, distinct technical status and proof gaps; preserve inherited candidate rather than select a replacement. Technical-design §§3–8 unchanged. |
| checklists/feature.md | Append unchecked CHK053–060 handoff prompts; prior text/checkmarks unchanged, no independent R3 pass inferred. |
| This evidence map and r2-rev6-r3-handoff.md | New incorporation/audit map and targeted technical review request. |
| r2-rev6-incorporation.sha256 | New repository-root-relative manifest of affected files, handoff/evidence and unchanged authoritative review/Scope inputs; excludes itself. |

The approval record was supplied in the requested directory before this work; no duplicate or
replacement independent report was necessary. All independent review files, Scope, old proposals,
old manifests and unrelated requirements remain preserved. R2-Rev6 §2 explicitly finds no Scope
amendment necessary; this author reconciliation neither amends Scope nor reapproves R2-Rev5.

## Validation and next gate

Static author checks cover approval identities, preservation of unaffected files/sections,
contract traceability, exact approved scenario rows, links, tables, whitespace and manifest
checksums. These checks are not independent R3 verification or runtime assurance.

- Exactly nine pre-existing operative documents changed; the other 59 of 68 pre-existing files
  under specs/ and independent-reviews/ remain byte-identical. All 28 independent-review files,
  including the supplied R2-Rev6 record, are preserved.
- Scope SHA-256 remains `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`;
  R2-Rev5 approval SHA-256 remains `f08218d808638187bb5227a386e5b8c91913e481308a62a2f62b9016559eca90`.
- Nineteen targeted preservation comparisons pass, including all physical entity field tables,
  task/receipt/throttle rules, task/view mutation wire contracts, task form semantics, FD-003
  text, future setup commands, maintenance/CSRF candidate details, historical checklist prefix
  and technical-design §§3–8.
- The copied approved T01–16 table is byte-identical to the reviewed evidence table; all IDs are
  complete, unique and ordered. Approval candidate hashes and the historical remediation
  manifest verify. Local links, code fences, whitespace and newlines pass static checks.

The [targeted R3 handoff](r2-rev6-r3-handoff.md) identifies remaining mechanism/proof obligations,
required regression evidence and change-control boundaries. No new material product decision is
currently requested; an actual inability to satisfy approved outcomes must be reported, not waived.
R3 remains NOT_READY; TD-008 OPEN. Barry's TD-006/007 ownership and exit requirements remain intact.
