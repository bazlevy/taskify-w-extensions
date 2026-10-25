# R3-Rev5 — Targeted correction / evidence map

**Prepared:** 2026-10-24. **Status:** Author candidate for independent R3-Rev5; NOT APPROVED.
TD-001/008 remain open in the independent ledger. No author finding closure, runtime pass,
human technical approval, risk acceptance or Tasks/Analyze/Build authority is asserted.

## Controlled inputs and history

Input: [independent R3-Rev4 report](../../../independent-reviews/3_design-technical/Review_4/taskify_R3-Rev4_technical_design_review.md).
Scope v1.1 / AM-001 remains byte-identical:
`b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`.
R2-Rev4 human approval remains its exact historical manifest in
[its independent report §8](../../../independent-reviews/2_design-functional/Review_4/taskify_R2-Rev4_functional_design_review.md).
Fresh technical hashes do not inherit that approval. Older independent reports/approvals, author
maps/manifests, checklists and AM-001 handoff remain unchanged. The nine Rev4 input hashes matched
their historical manifest before this revision.

The rejected Web Locks availability option remains rejected. Rev4's cookie-completeness/client-
floor proof is explicitly superseded, with its review/map/manifest preserved as history.

## Finding / correction / evidence delta

| Finding and approved trace | Concrete delta from Rev4 / affected artifacts | Planned validation / evidence gate |
|---|---|---|
| TD-001 — OPEN High blocker; FR-010 / AC-010 and FR-009/014 | technical-design §1, protocol §A, HTTP claim/activate, UI authority state, data-model registry, Plan/research. Register every transition under AuthorityFamily before issuance. Current A handle locates F; claim locks F, increments generation and cancels ALL family candidates by FK even if B proofs vanished. ACTIVATE locks F/rechecks captured generation. Claim first fences B; B first retires A so A-only claim is 401/concealed. No cookie/registry authority means no reveal. No completeness or signal-delivery premise | R5-V01a–c extend exact R4-V01a/b with post-dispatch selective omission while keeping B's captured proof valid, A-only and no-cookie claims, both transaction orders, lost broadcasts, failed later task reads, DOM/accessibility/history paint and stale-epoch write rejection |
| TD-008 — OPEN Medium blocker; FR-010/012/015 / AC-005 | technical-design §2, protocol §B, HTTP single-cookie/phase/failure rules, UI sign-in recovery, data-model Family/Handle/Transition, Plan/research. One fixed authority slot replaces growing attempt/floor cookies; server current pointer is the floor. Current ACTIVE handle + live Session mandatory; old/unknown/provisional/ambiguous/missing values cannot select older accounts. ISSUE emits each provisional value once; ACTIVATE/logout/ordinary responses never rewrite/delete it. Anonymous roots cannot change another family's state or browser's selected cookie. Concrete deadlines/admission/cleanup remove undefined cookie-retention horizon | R5-V08a–d: anonymous/same-family races, omission/eviction/rejection, retired same-family A restoration, no old-cookie fallback, one-time Set-Cookie under 512 bytes, fifth PREPARE, 120-second deadline, guarded 60-second cleanup/deletion/late requests and pending metadata/load measurements; shared logout and all functional regressions |

The authority decision is server data, not a “highest surviving cookie.” Current family/handle
remains for actual Django session lifetime. Terminal/unreferenced rows can be purged safely:
missing is unauthorized and cannot be recreated by stale ACTIVATE or ordinary middleware.
One 43-character cookie, four incomplete transitions/family, nonextendable 120-second preparation,
60-second cleanup, 64-family/transaction ×16 batches/run and existing connection/timeout budgets
are explicit testable engineering settings, not account/task caps or product SLAs. The protocol
defines the 1,500-admission burst envelope, skipped-run/age alerts and backlog/storage measurements.

## Behaviour and decision boundaries

- Scope and R2 task/view rules unchanged: task-first bootstrap, full silent view defaults,
  same-login view semantics, FD-001 all-task-write/retry lock and existing stale-row task-error UI.
- Before focus/history reveal, mandatory authority claim plus matching snapshot still required.
  Missing/unusable credential is 401/concealed sign-in; registry failure is 503/concealed Retry/
  support. No fallback through an older/provisional handle to retain access.
- Invalid credentials preserve prior login before ISSUE. Interrupted valid login/cookie rejection
  can leave no usable current credential; existing auth-error/sign-in guidance applies. This is
  explicitly visible to independent review, not a promise to restore a lost credential or private
  content. Actual committed success is recovered only through its current live handle.
- Any-tab logout atomically revokes family/session/view and all pending activations, including
  those whose cookie is missing; old cookie/session callbacks cannot reactivate it. Independent
  browser/device families and ordinary multi-tab task/view use remain unaffected.
- Explicit same-user login still creates fresh key/epoch/defaults; maintenance rotation alone
  preserves epoch/view/original expiry. FD-003 memory-only task identity and FD-004 no credential
  recovery retain only their previously accepted bounds.

No new product-policy change, reduced browser coverage, other-tab acknowledgement wait, exposure
interval or TD risk acceptance is proposed. Independent R3 must assess the written safety proof
and failure behaviour. If implementing it requires a product-policy change or weakening a
guarantee, stop for Barry and the appropriate R1/R2 process rather than inferring approval.

## Retained findings / downstream owners

R3-Rev4 retains TD-002/003/004/005/007/009 resolved at written-design level; TD-006 is open,
nonblocking, R5 operational evidence outstanding. This is inherited disposition, not author closure.
Technical-design §§3–8 are byte-identical, including old status history and §7 owner/exit contracts.
New family authentication/claim/cleanup costs must be measured within those budgets, not silently
increase pools or supersede the TD-003 capacity gate.

Barry Levy remains accountable TD-006 Operations/R5 and TD-007 Maintenance/R4; Codex prepares
technical corrections, validation and handoffs. Protocol §B extends the restore handoff to revoke/
clear restored authority families/handles/candidates before traffic, alongside existing Session/
view invalidation. This adds required registry coverage, not operational approval or a passed drill.
R4 upgrade fixture handoff and later Build/Test compatibility evidence remain due; no new receipt
format/key/canonicalization change is introduced.

## Changed files and author validation

- technical-design.md: targeted §§1–2 and current submission metadata.
- auth-transition-protocol.md: server-family invariant, one-slot credential, all lifecycle/
  failure/cleanup bounds and seven Rev5 schedules; explicit Rev4 proof withdrawal.
- contracts/http.md / contracts/ui.md: consistent family/handle authority, phase/claim/logout and
  fail-closed recovery rules.
- data-model.md: Family/Handle registry, captured generation, constraints/deletion guards.
- plan.md / research.md: mechanism/rationale, engineering scope and current R3 gate.
- quickstart.md: Rev5 missing-cookie schedules and retained task/view/other-TD regressions.
- this map / r3-rev5-changed-files.sha256: current candidate identities.

Author static checks compare with this turn's input: Scope and all historical/review/checklist/
handoff artifacts unchanged; technical-design §§3–8 byte-identical; HTTP snapshot/view-save/task
mutation/failure sections byte-identical; task form/actions/view remembering semantics unchanged;
User/TaskListState and Task/MutationReceipt/AuthThrottleBucket/sort sections unchanged.
All AM001-V01–10 and R3-V03/04/05/06/07/09 rows are retained byte-for-byte.
Finding/scenario IDs, local document links, diff whitespace and manifest hashes are checked.
These are document checks only: no application code or browser/database/load/restore execution,
and no Spec Kit Tasks/Analyze/Build. Independent R3-Rev5 and explicit human R3 approval remain due.

[Changed-file SHA-256 manifest](r3-rev5-changed-files.sha256) covers the nine changed/new Markdown
documents relative to specs/001-personal-task-manager/. It excludes itself to avoid circular
self-hashing. Hashes identify this unapproved candidate, not independent verification.
