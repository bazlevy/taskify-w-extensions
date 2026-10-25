# R3-Rev4 — Targeted author correction / evidence map

**Prepared:** 2026-10-23. **Status:** Submitted for independent R3-Rev4 review; R3 NOT APPROVED.
TD-001/008 remain open in the independent ledger. Author changes are not independent verification,
finding closure, risk acceptance or Tasks/Analyze/Build authority.

## Controlled inputs

[Independent R3-Rev3 report](../../../independent-reviews/3_design-technical/Review_3/taskify_R3-Rev3_technical_design_review.md):
only TD-001 High and TD-008 Medium narrowed remain blockers.
Scope v1.1 / AM-001 is byte-identical at
`b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`.
R2-Rev4 approval remains its exact historical manifest in
[§8 of its independent report](../../../independent-reviews/2_design-functional/Review_4/taskify_R2-Rev4_functional_design_review.md).
Fresh technical hashes do not inherit approval. All independent reviews, checklists, old proposals,
AM-001 handoff and R3-Rev3 author map/manifest remain unchanged.

## Human direction / rejected option

Barry clarified shared-session logout from any tab; no stale private information before revisit;
normal multi-tab use and independent browser/device sessions unchanged. He explicitly chose
**“Seek an alternative preserving current availability.”** The initial author Web Locks option's
capability/stalled-holder trade-offs are rejected, not approved. Earlier clarification requests
about that option are superseded. Protocol §D retains its decision history.

The current alternative uses atomic server-side reveal claims plus staged cookie activation.
There is no browser lock, new supported-client requirement, wait for another tab, finite exposure
window or change to approved task-refresh stale rows. No new product-policy/risk-acceptance choice
is proposed. If the written invariants fail in supported browsers, reopen review and escalate;
do not silently reduce availability or privacy to make tests pass.

## Per-finding correction / evidence

| Finding / trace | Targeted correction and affected evidence | Planned independent/downstream verification |
|---|---|---|
| TD-001 — OPEN High blocker; FR-010 / AC-010, FR-009/014 | technical-design §1 / ADR-T01; protocol §A; HTTP POST /api/session/claim/; UI concealed/claim/snapshot state; data-model authority states; Plan/research. A returning page atomically cancels presented incomplete login/rotation or completes authorized pending logout before validating actual account/epoch. Every auth continuation checks actual focus/visibility/generation before dispatch and after each await. In-flight activation already has cookie proofs; claim first cancels it, activation first is seen by claim. Activation emits no later auth ticket. Optional broadcasts are not correctness evidence. Ordinary focused same-epoch stale rows/error/FD-001 remain | quickstart R4-V01a–c and retained R3-V01: reviewer exact delayed-B-after-A ordering, both ledger-lock orders, lost signals, failed later reads, late cookies/callbacks, context death, DOM/accessibility/history paint. Real browser/PostgreSQL evidence outstanding |
| TD-008 — OPEN Medium narrowed blocker; FR-010/012 / AC-005, TD-001 coupling | technical-design §2 / ADR-T02; protocol §B; HTTP phase/claim routes; data-model AuthTransition; UI recovery flow; Plan/research. PREPARE persists provisional session/signed pending marker; ISSUE needs processed marker and delivers distinct-name provisional ticket; ACTIVATE needs actual delivered ticket, commits ACTIVE/ordinal and predecessor invalidation with no new auth-ticket issuance. Cancelled tickets are inert. Highest acknowledged floor prevents expiry/late-cookie fallback. No original Session row needed; same-user fresh login, maintenance preservation and unrelated-session isolation remain | quickstart R4-V08a–d and retained R3-V08: no-cookie/no-old-row different-account logins, reversed commit/header/result delivery, cancelled cookies after active B, expiry/floor cleanup, every phase/claim loss/rollback, same-user/rotation and shared logout. Verify actual HttpOnly cookies, not only mocked promises |

R4-V names this R3-Rev4 submission's scenarios, not permission to enter the separate R4 phase.
A claim is auth housekeeping, not a task mutation/default preference write; GET stays read-only.
Successful logout revokes the shared session/view, never just one tab. Claims complete authorized
pending logout instead of cancelling it back to a usable login. Independent browser/device sessions
remain valid; normal task edits/tab view choices remain concurrent.

## Preserved independent dispositions and downstream owners

R3-Rev3 resolves TD-002/003/004/005/009 at their recorded design boundaries; TD-007 is resolved
at design level with R4/Build upgrade evidence due. TD-006 remains open, nonblocking, R5 execution
evidence outstanding. These are inherited independent dispositions, not new author closures.
Technical-design §§3–8, including all budgets and §7 owner/exit contracts, are verbatim retained.

Barry Levy remains accountable TD-006 Operations/R5 and TD-007 Maintenance/R4; Codex prepares
technical designs/corrections, validation plans and handoff documentation. R5 still needs policy/
access/configuration sign-off and successful restore/migration/diagnostic evidence; R4 confirms
upgrade fixture handoff and later Build/Test executes compatibility fixtures. Assignment is not
risk acceptance or technical approval. FD-003/004 retain only their original accepted bounds.

## Changed artifacts / validation limits

- technical-design.md: revise only TD-001/008 mechanisms plus candidate metadata.
- auth-transition-protocol.md (new): focused continuation, atomic claim, staged cookie/ledger,
  shared logout, interruption/selection proof and rejected-option history.
- contracts/http.md: internal phases and POST authority claim; later task/view contracts unchanged.
- contracts/ui.md: claim-before-reveal and guarded continuations; ordinary multi-tab/stale-row rules.
- data-model.md: AuthTransition and orthogonal authority/lifecycle state.
- plan.md / research.md: mechanism/rationale, current independent ledger and gate metadata.
- quickstart.md: seven targeted schedules plus pending-logout/background-callback regressions.
- this map / r3-rev4-changed-files.sha256: fresh candidate identities; old records untouched.

Author static checks compare with files at this turn's start: Scope/reviews/checklists/history
unchanged; technical-design §§3–8 byte-identical; HTTP snapshot/view-save/task mutation/failure
sections byte-identical; UI task forms/actions and view remembering unchanged; User/TaskListState
and Task/MutationReceipt/AuthThrottleBucket/ordering sections unchanged. Prior R3-V03/04/05/06/
07/09 rows remain byte-identical. FD-001 task-load retry lock, FD-003 memory-only identity, FD-004
credential disclosures and AM001-V01–10 remain intact.
Relative document links, finding/scenario consistency, diff whitespace and SHA-256 manifest are
checked locally. No application code, browser/database/load/restore tests or Spec Kit Tasks/Analyze/
Build are run. Independent R3-Rev4 must assess the written proof and planned fixtures; explicit
human R3 approval remains required before any next phase.

[SHA-256 manifest](r3-rev4-changed-files.sha256): nine changed/new Markdown documents, relative to
specs/001-personal-task-manager/. Self-hash omitted to avoid circularity. Hashes identify this
unapproved candidate, not independent verification.
