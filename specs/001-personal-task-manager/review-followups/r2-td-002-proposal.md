# TD-002 — Proposed R2 Functional Design Amendment

> **Superseded proposal — 2026-10-23:** R3-Rev1 Addendum A rejects blocking task access for view
> restoration. This never-approved proposal is retained below verbatim for audit. Its successor is
> the [R1 session-view proposal](r1-session-view-amendment-proposal.md) and
> [R2 session-view proposal](r2-session-view-amendment-proposal.md). Their formal approvals remain
> pending; TD-002 remains OPEN. The original status and requested decision below are historical.

**Date:** 2026-10-23
**Status:** PROPOSED — awaiting human Functional Design approval and independent R2 review.
**Authority:** [R3-Rev1](../../../independent-reviews/3_design-technical/Review_1/taskify_R3-Rev1_technical_design_review.md), TD-002.
**Parent baseline:** R2-Rev3, human-approved 2026-10-23; this proposal does not amend that approval.

## Decision requested

Approve holding initial task access until the account's saved filter preferences can be read and
validated. This closes only the undefined first preference-GET failure path. No default fallback,
preference overwrite, new recovery feature, or Scope change is proposed.

The benefit is truthful restoration and a single ordered startup flow. The trade-off is that a
preference-service failure temporarily prevents task access even when task storage is available.
An alternative allowing a temporary default view would require a separately approved functional
policy; it is not incorporated here.

## Proposed observable rules

- **P-TD002-01:** At startup show “Loading your saved filters” while retrieving the authenticated
  account's preference pair/version. Do not request the first task snapshot, submit preference
  changes, or enable task creation, edits, status changes, deletion, or pending-task-write retries.
- **P-TD002-02:** A timeout, unavailable response, malformed schema, invalid enum/version, or wrong
  account/session response must not substitute All/All or imply that preferences were restored.
  Show “Your saved filters could not be loaded. Retry to open your task list,” with Retry and support.
  All/All is valid only when returned as the account's authoritative preference, including first use.
- **P-TD002-03:** Retry repeats only the preference GET; coalesce repeated retry clicks. After a
  second consecutive preference-load failure explicitly recommend support with a nonsensitive
  request reference. Do not change saved preferences or task data to recover.
- **P-TD002-04:** On a valid same-account/session preference response, apply its pair/version and
  request the first task snapshot using that pair and the existing fresh-session sort default.
  Keep task writes disabled until a valid task snapshot succeeds. Task-snapshot failure then uses
  the existing FD-001 load-error rules; a remaining unknown task operation still requires its
  original ID/payload recovery after successful loading.
- **P-TD002-05:** A 401 follows existing sign-out handling. Account/session changes invalidate old
  requests and restart startup for the newly authenticated account. Never render or apply the prior
  account's responses. Conceal any history-restored content pending revalidation.

## Boundary and state transitions

`preference_loading → preference_error → preference_loading` on Retry;
`preference_loading → task_loading` only on a validated preference;
`task_loading → ready / task_load_error` under the existing task-state rules.

The preference bootstrap gate is distinct from later FD-005 preference-save reconciliation. A
failed reconciliation GET after a task list has loaded retains the current visible selection as
unconfirmed under the approved rules; this proposal adds no new restriction to that established flow.
FD-003 memory-only interrupted-operation risk and FD-004 no-password-recovery policy remain intact.

## Proposed acceptance scenarios

| ID | Setup/action | Observable result |
|---|---|---|
| P-TD002-A1 | Saved pair Completed/High; initial preference GET returns 503 while tasks endpoint works | Clear preference error and Retry/support; no task GET, preference PUT, task write, or default restored view |
| P-TD002-A2 | Initial GET times out or returns malformed/invalid preference data | Same protected error path; never a successful empty task list |
| P-TD002-A3 | Retry fails twice, then returns Completed/High at version 4 | Support recommendation after repeated failure; valid response starts task GET with that pair; no writes until a valid task snapshot |
| P-TD002-A4 | Preferences load but first task snapshot fails with an uncertain task operation retained | FD-001 blocks every task-write submission, including replay; recovery uses the original key/payload only after a valid task snapshot |
| P-TD002-A5 | Account changes while initial preference GET is delayed | Old reply is ignored; new account startup validates and restores only its own preferences |
| P-TD002-A6 | First-use account returns valid All/All; later preference PUT loses its response | Initial All/All is accepted as authoritative; later failure follows FD-005 reconciliation without claiming definite non-persistence |

## Incorporation and approval record

After human approval and R2 review, reconcile `research.md` §7, `contracts/http.md` Filter
preferences, `contracts/ui.md` startup/failure states, `data-model.md` browser state, and
`quickstart.md` acceptance scenarios. R3 then specifies validators, request generations and state
guards. Approved Scope `spec.md` stays unchanged.

**Human decision:** PENDING. **Decision reference/date:** not yet supplied.
**Independent R2 amendment assessment:** PENDING. **TD-002 finding:** OPEN.
Approval of this proposal would authorise its controlled incorporation, not application Build or
the next development phase; independent re-review and explicit gate decisions remain necessary.
