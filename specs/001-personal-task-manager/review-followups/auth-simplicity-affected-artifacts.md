# Auth simplicity assessment — affected-artifact / decision map

**Date:** 2026-10-24. **Assessment only.** P1-A and TD-008 remediation paused by the user.
R3 NOT_READY; TD-008 OPEN. No operative design/requirement/review file changed.

[Proposed ADR-AS-001](adr-auth-simplicity-assessment.md) recommends the Django-first **candidate**,
not adopting an unproven simplification or accepting its exceptional risks. Normal Scope/R2
coverage does not itself prove logout or painted-DOM behaviour under arbitrary response order.

## Prospective changes only — after direction/review

| Artifact | Smallest proposed change / preserved material | Gate / evidence |
|---|---|---|
| spec.md; independent R1/R2/R3 records | No edits. Preserve Scope v1.1 and R2-Rev5/R2-NAV-001 approval, FD-003/004 bounds and review history | Actual requirement departures use targeted R2/Scope/human process; no retroactive rewriting |
| technical-design.md §§0–2 | Replace authority-registry/protocol/rotation architecture with Django auth/session lifecycle and necessary thin adapters, if selected | Independent R3 must reassess removed TD-001 proof and TD-008 residuals. Keep historical dispositions and §§3–8 budgets/HMAC/throttle/ops/compatibility unchanged |
| auth-transition-protocol.md | Archive/supersede current bespoke flow only after approval; normal login/logout would use framework views/services rather than ISSUE/ACTIVATE or maintenance endpoints | ADR S1–S5 and AS-02–05/08 must support the chosen boundary; do not delete history now |
| contracts/http.md | Exact authenticated redirects (GET 302/POST 303), CSRF errors and read-only current-session/epoch validation. Remove custom authority/rotation routes in a future candidate; keep task/view/receipt/FD-001 interface | Stock LoginView redirect flag alone is not the exact R2 wire contract. Thin response policy addresses delayed deletion only, not late login success |
| contracts/ui.md | Keep same-profile task entry, logout-first navigation, epoch/intended-action guards, DOM/draft clearing, before-reveal revalidation, optional broadcast and CSRF recovery. Remove family/candidate terminology only after adoption | D3 is unresolved; focus/polling is not guaranteed instantaneous invalidation. Do not weaken stale-row task-failure rule to claim privacy solved |
| data-model.md | Propose dropping AuthorityFamily, AuthorityHandle and AuthTransition. Retain Django Session, minimal session epoch and SessionViewState; assess native rekey view preservation | AS-07/08; validate Session→view locking/cleanup and logout wrapper. No new auth store, whole-session mutable view tuple or account-wide device revocation |
| plan.md / research.md | Record selected standard stack/configuration and only justified adapters; remove custom maintenance/P1 dependencies if simplification adopted | No Plan rerun now. Reassess capacity/auth correctness interactions without reopening unrelated resolved mechanisms gratuitously |
| quickstart.md | Proposed AS-01–08 coverage replaces bespoke implementation-specific schedules while retaining NAV/AM001/FD/ownership/receipt/throttle/optional-view regression intent | Planned only; no test/Build execution or checklists marked passed |
| Prior proposals/maps/manifests, including P1 | Preserve untouched as audit records. This assessment records the user's pause; earlier pending P1-A prompt is not a current approval request | Do not resume P1, amend availability or run further remediation from this ADR |

## Decisions needed before adopting a simplified candidate

- **Ordering versus reliability:** no exact two-anonymous-form winner is mandated. State a
  deterministic expected response-order outcome for testing; refer any material session/view
  continuity departure to R2 rather than asserting a new absolute winner requirement.
- **Logout boundary:** assess current-cookie Session invalidation versus preventing all previously
  submitted anonymous/rekey responses from later authenticating. A narrower promise requires
  human/R2 approval if it departs from approved shared logout; no default acceptance.
- **Private display:** retain mandatory revisit concealment. Resolve any exceptional already-
  focused stale-DOM consequence explicitly; no unapproved exposure interval or false logout.
  If strict guarantees remain, the thin Django candidate is not yet proven sufficient.
- **Technical policy:** evaluate suppressing stale empty-session cookie deletion, retaining safe
  server invalidation, and supported native-key-change/view handling. Source inspection is not
  framework/browser compatibility proof.

Only this map and ADR are new. Baseline preservation hashes and relative links are checked;
no browser/database/security/restore/load execution, application code, Tasks/Analyze/Build or
operative design change. R2-Rev5 approval record remains
`f08218d808638187bb5227a386e5b8c91913e481308a62a2f62b9016559eca90`;
Scope remains `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`.
Neither assessment document changes an independent finding's disposition.
