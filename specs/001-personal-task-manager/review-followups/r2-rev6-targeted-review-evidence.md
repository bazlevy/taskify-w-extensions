# R2-Rev6 — targeted review evidence, scenarios and change map

**Date:** 2026-10-24. **Status:** Author proposal, independent R2 review pending.
Scope v1.1 / R2-Rev5 APPROVED; R3 NOT_READY; TD-008 OPEN. P1 remains paused.
The user's report of Barry's S2-DEC-001/S4-DEC-001 confirmations is decision input, not an
independent R2 verdict or formal approval of this revised design.

**Review target:** [proposed decision record and functional clauses](r2-rev6-proposed-decisions-and-functional-delta.md).
Only new follow-up documents are prepared; all operative artifacts and historical records stay
unchanged. This package does not modify earlier ADR-AS-001/002 claims or their manifests: their
unaccepted S2/S4 statements describe the earlier assessment, now qualified by this new proposal.

## Finding/decision and artifact map

| Input / affected artifact | Precise proposed delta | Trace / review evidence |
|---|---|---|
| S2-DEC-001; UI navigation/privacy, HTTP logout and auth workflows | Logout revokes its intended shared session permanently; separate valid late login may later become current. Remove only the universal pending-login cancellation promise from the proposed functional outcome. Keep authenticated redirects and logout-first normal navigation. | Proposal §§1–3 UI-01/HTTP-01/STATE-01; FR-010, R2-NAV-001; T01–04/09–10 |
| S4-DEC-001; UI privacy/failure states and HTTP private-action preconditions | Remove instantaneous cross-tab clearing obligation. Require action/lifecycle gates, owner/epoch checks, clearing on observed invalidation and rejection of old intent; no stale-access/mutation waiver. | UI-02/HTTP-01/STATE-01; FR-010/AC-010, FR-009/014/AC-008; T05–08/11–12; Q1/Q2 |
| data-model.md | Clarify functional fresh-login/revocation/view ownership only. No schema, key, tuple, task version or receipt change; do not adopt Option B physical design through R2. | FR-008/012/013, AC-005/009; T02/04/07/10–11 |
| research.md / plan.md | Add two decision IDs, qualified exceptional boundaries and gate references; preserve all unrelated choices and accepted FD-003/004 risks. | Proposal §§1–4; no Plan rerun or technical adoption |
| quickstart.md | Add T01–12 below; qualify older universal cancellation/instantaneous-display expectations prospectively, retain their audit history and unaffected NAV/AM001/FD scenarios. | Explicit allowed late login vs forbidden revoked-key replay; known mismatch vs Q2; no fabricated test passes |
| checklists/feature.md (future targeted addendum only) | Review CHK016/022/034/036/047/049/050/052 for decision provenance, protected-action definition, Scope consistency and unapproved R3 dependencies; leave historical checkmarks and old checklist text intact. | No checklist is marked complete or independently verified here |
| Scope v1.1 / R2-Rev5 reports, manifests and prior reviews | No edits. Scope comparison in proposal §2; route any demonstrated FR-010/AC-010 departure to R1/human process. | Historical approvals are not rewritten; new human policy direction is distinct from gate approval |
| ADR-AS-001/002 and R3 technical artifacts | New functional proposal qualifies earlier S2/S4 assumptions only if reviewed/approved; no edits/removal of technical fences yet. | TD-008 remains OPEN; R3 must still prove revocation, action isolation, task ordering, CSRF and rekey continuity |

## Targeted deterministic acceptance scenarios — planned, not executed

Use same-profile tabs plus isolated device sessions, distinct A/B accounts and same-account
distinct login epochs. Schedule response **headers**, not just JavaScript callbacks; inspect
server revocation/effects and UI/DOM/accessibility state. Do not count author reasoning as proof.
T05's local-action inventory is proposed pending Q1; T08 exposes the unresolved late-reveal
boundary rather than declaring it passed. IDs below are R2-6-T01 through R2-6-T12.

| ID / trace | Given / schedule | Required observation / limit |
|---|---|---|
| R2-6-T01 / S2, FR-010 | A is current in two tabs; logout A commits; replay captured A credential directly against shell, tasks, view and mutation routes. | A never authenticates or revives its view; no private API data or task/view effect. Other A tabs cannot act using revoked A. No logout ACK before revocation commit. |
| R2-6-T02 / S2, FR-010/012 | Anonymous A/B submissions were already sent; persist B but hold its response headers. A becomes current and logout A commits; then release B. Repeat B=A with a distinct fresh epoch. | B may authenticate: not a test failure by itself. A's revoked credential stays invalid. Current account/epoch is B's, new-login view defaults; no A draft/queue/view transfer or automatic logout of B. |
| R2-6-T03 / S2, R2-NAV-001 | A already authenticated when login GET or POST carries its valid cookie; supply A or B replacement credentials. Also test invalid CSRF and storage failure. | GET 302 /tasks/; valid-CSRF POST 303 before replacement credential evaluation; same account/epoch/view, no new login. Invalid CSRF 403; uncertainty 503, not assumed anonymous. S2 does not bypass this guard. |
| R2-6-T04 / S2, CSRF | Capture logout intent for epoch A; install B before logout dispatch or during token refresh. Separately drop A logout's response after commit. | Old intent cannot revoke B; identity change is not A's logout ACK. No automatic credential/task replay; token recovery bounded to same intended epoch. Lost response stays unconfirmed; direct A replay demonstrates actual server revocation separately. |
| R2-6-T05 / S4, FR-010/AC-010 | A is already painted in a focused tab; another tab logs out or a valid B response arrives; drop notifications. Invoke each proposed protected action, including local detail/edit/delete/filter/sort and submission/retry. | Passive stale paint alone is not an instantaneous-clear failure. Revalidate before protected action; invalid/mismatched identity blocks old access and clears old state before proceeding. No old A payload becomes B's data. Run full inventory once Q1 is settled. |
| R2-6-T06 / S4, privacy lifecycle | Hidden/history-restored tab contains A rows, labels, draft, dialog and accessibility text; current auth is revoked A or B. Return/focus it; fail lookup, then recover. | Concealed before reveal; uncertainty stays concealed with Retry/support; invalid/mismatched identity clears old state. Only a matching authenticated task load enables the current account. No view-failure defaults bypass auth. |
| R2-6-T07 / S4, stale intent | Revalidation initially finds A, then B becomes current before create/edit/status/delete/view-save dispatch. Use B's valid CSRF with the captured A epoch; vary delivery order. | Server rejects before old-intent effects/receipt creation or view mutation, does not flush B, and does not rebind A's task text to B. Known old replies are discarded. Same-account new epoch is also a mismatch. |
| R2-6-T08 / S4, Q2 | Hold private A response authorised before B's delayed login header; lose notifications; release A payload while tab is focused. Repeat with explicit known invalidation. | Known invalidation: discard, no reveal. Unobserved B change/new A reveal: obtain explicit R2 boundary and R3 proof; do not count passive-paint allowance as permission for every newly arriving payload. This case remains a review gap pending Q2. |
| R2-6-T09 / S2, FR-008/014 | Interleave an A task command and logout: command commits first, then reverse so revocation commits first. Lose command reply and fail saved-task reload. | Pre-revocation committed effect persists with receipt/revision; no command under revoked A commits after logout ACK. If revocation wins, no effect. FD-001 blocks all task writes and exact retry until valid saved-task load; auth success alone does not unlock them. |
| R2-6-T10 / retained Scope | A is signed in on another device/profile with nondefault view. Logout this profile; release late fresh same-account login; navigate authenticated login URLs in both. | Other session remains live with its own view; new login defaults. Valid login navigation preserves account/epoch/view. Saved tasks still synchronise; no account-wide logout or account-preference inheritance. |
| R2-6-T11 / FD/AM001 | Ordinary same-epoch task refresh fails after prior success; separately fail required action auth gate and optional view read/save. Regress AM001-V01–10 and R2-T05/T07/T12. | Ordinary task error keeps visibly stale rows, Retry/support and locked writes; auth-gate uncertainty conceals and blocks protected action. View-only failure remains silent/isolated. No reset, password recovery, durable pending identity or weakened interruption guidance. |
| R2-6-T12 / S1, FR-010/012 | Two genuinely anonymous credential POSTs overlap; deliver A then B, with no logout. Reverse response order; repeat same/different accounts; also submit an actually authenticated POST. | Final cookie selects the valid delivered login; no exact first-submit winner needs human approval. Each distinct fresh login starts defaults; enforce action/owner/epoch isolation under T05–08. Authenticated POST must still satisfy T03. Selection alone is not a waiver of any demonstrated privacy/view/navigation departure. |

## Review decisions and downstream handoff

Independent R2-Rev6 should assess the two policy deltas, Scope consistency, proposed protected
action inventory (Q1), and overlapping private-response reveal (Q2); regress anonymous-response
selection separately without inventing a D1 approval gate. Escalate to Barry only for a material unresolved policy choice; do not ask
him to reconfirm both recorded decisions as though they were absent. Formal human approval of
the reviewed revised Functional Design is still required before incorporating it.

No independent findings are closed. S2 changes the expected outcome of the earlier Option B
B-03 reproducer; it does not demonstrate valid-session continuity, cookie correctness or old-key
revocation. S4 changes the passive-clearing boundary of B-09, not the stale-intent protections or
unresolved newly revealed payload question. S3 deletion policy and S5 native rekey/view handling
remain technical review subjects. R3 NOT_READY / TD-008 OPEN persists; no downstream phase.

New document identities are recorded in [the SHA-256 manifest](r2-rev6-proposal.sha256), excluding
the manifest itself. Preservation hashes, links, scenario identities and whitespace are static
author checks only. No browser, database, application or runtime tests are run for this package.

Preservation check: all 61 pre-existing files under specs/ and independent-reviews/ match their
start-of-follow-up hashes. Scope SHA-256 remains
`b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`;
R2-Rev5 approval-record SHA-256 remains
`f08218d808638187bb5227a386e5b8c91913e481308a62a2f62b9016559eca90`.
Local document links, unique T01–12 identities, final newlines and whitespace pass static checks.
