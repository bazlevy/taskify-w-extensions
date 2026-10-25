# Post-Rev5 authentication navigation — correction / evidence map

**Date:** 2026-10-24. **Status:** Author clarification candidate; R3 remains unapproved.
No Tasks/Analyze/Build, application code, runtime pass, finding closure or risk acceptance.

## Authority and review boundary

The user explicitly clarified approved FR-010: authenticated same-profile new tabs go directly
to tasks; login-page access redirects to tasks; changing accounts requires logout first.
Treat this as navigation clarification, not Scope expansion. Canonical Scope v1.1 is unchanged.
Existing R2/R3 approvals and independent reports remain immutable.

The independent R3-Rev5 report is not present in this workspace (latest supplied independent file
is Review_4). No unseen report text, new finding ID or independent verdict is fabricated.
This candidate addresses the navigation gap exactly as supplied by the user, retaining existing
TD-001/008 safeguards until independent review. Old Rev3/Rev4/Rev5 maps/manifests stay unchanged.

## Change / evidence

| Area | Correction / affected artifacts | Planned evidence |
|---|---|---|
| FR-010 entry and navigation | UI Screens/navigation; HTTP GET /, /tasks/, /accounts/login/. Same-profile active login enters tasks, no form/new login/epoch/view reset. Login GET is fixed local 302; task shell still validates before private reveal | NAV-01/02: nondefault acknowledged view, same account/epoch, no new auth rows/cookie/reset; query/next/history and logout-racing-redirect |
| Stale login submissions | HTTP login POST: CSRF-valid authenticated request gets 303 /tasks/ before credential/candidate processing; bad CSRF 403 leaves session intact. UI handles redirected fetch as navigation, never credentials replay. Internal ISSUE/ACTIVATE guard observed authority/target-family state and reject stale/conflicting candidate 409 | NAV-03/04/07: stale same-/different-account valid/invalid credentials, history form, UI bypass, delayed captured headers, no identity/cookie/view replacement |
| TD-001 simplification / safety | protocol §N; technical-design §0/§1. Remove normal active-family account replacement from threat path: login PREPARE only anonymous/expired; live-family login ACTIVATE is forbidden. Retain claim/generation/focus, missing-cookie fail-closed and stale response handling for rotation/logout/already-dispatched anonymous requests | Refined R5-V01a/b use stale invalid login-candidate rejection and legitimate rotation lock orders; NAV-07 retains actual delayed ISSUE/ACTIVATE, cookie loss, dropped signals, failed later reads and DOM/epoch evidence |
| TD-008 lifecycle separation | protocol §N/§B; HTTP rotation preparation; data-model invariants; Plan/research. Fresh login creates empty family; only designated non-login rotation transfers live same-account authority/epoch/view/expiry. Login navigation never enters three-phase cookie protocol. No added schema/cookie/browser capability | NAV-04/05: illegal live replacement rejects; logout then A/B fresh login defaults; shared revocation/rotation/independent-device and ordinary multi-tab regressions |
| Failure guarantees | Unknown authority lookup is 503/Retry, never assumed anonymous; stale/invalid credentials cannot erase an active cookie; mandatory auth/snapshot gate and FD-001 task-write/retry protection retained. Optional view failure remains silent | NAV-06 plus existing R2-T05/07/12, AM001-V01–10 and resource/receipt/throttle/optional-reader scenarios |

NAV-01–08 are planned scenarios, not test results or a new development phase.
No redirect is assumed to cancel every in-flight request or prove current browser cookies to a
server processing older captured headers. The registry/one-time issuer is deliberately retained;
stock LoginView redirect handling alone would not justify deleting those concurrency guarantees.

## R2-NAV-001 — targeted Functional Design confirmation required

The approved R2-Rev4 HTTP artifact has SHA-256
`b179d110424b71af81789e5d9ec2fd8cca6981e8a08da9370f7c3a9cd0213a5a`.
The unchanged Git HEAD copy matches that approved identity. It explicitly gives repeated same-user
successful login a fresh view lifetime/defaults. The candidate now redirects a still-authenticated
same-user URL/POST and preserves its epoch/view; fresh defaults apply after logout/expiry.

This is the concrete behavioural delta requiring targeted R2 review, not merely an editorial
claim that all approved behaviours are unchanged. The user authorised preparing the correction;
no independent R2 verification or human R2 sign-off is inferred. R2 must reconcile that direct
re-login/reset clause and AM001-V04/R3-V08 expectations with FR-010 navigation and FR-012/AC-005
valid-login retention. Do not amend Scope or rewrite the old R2 approval. Confirmation or any
required human decision remains outstanding before this delta is treated as an approved baseline.

## Preservation / checks

Eight current technical documents are revised: contracts/ui.md, contracts/http.md,
auth-transition-protocol.md, technical-design.md, data-model.md, plan.md, research.md, quickstart.md.
This map and its SHA-256 manifest are new; prior author/reviewer records are not edited.

Author static checks: Scope/reviews/approvals/checklists/history hashes unchanged; technical-design
§§3–8 byte-identical; HTTP snapshot/view/task mutation/failure sections byte-identical; task
forms/actions, filtering/remembering and User/TaskListState/Task/receipt/throttle/sort rules
unchanged. Targeted scenario updates are AM001-V04, R3-V08, R5-V01a/b/V08a/d plus NAV-01–08;
unrelated AM-001 and R3-V03/04/05/06/07/09 rows retained. Document links, scenario IDs, whitespace
and manifest identities checked. No browser/database/restore/load or application execution.
Barry/Codex downstream TD-006/007 owners/exit contracts remain unchanged; assignments are not approval.

[Changed-file SHA-256 manifest](r3-navigation-clarification-changed-files.sha256) covers eight
changed source-design documents plus this map, relative to specs/001-personal-task-manager/.
It excludes its own hash. Return for targeted R2-NAV-001 and independent R3 assessment;
R3 stays unapproved and no next phase is authorised.
