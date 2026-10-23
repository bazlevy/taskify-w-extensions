# R2 proposal — Session view workflow and minimal mechanism

**Prepared:** 2026-10-23. **Status:** PROPOSED; dependent R1 amendment, independent R2 review and
formal human R2 approval pending. Source: R3-Rev1 Addendum A.
**Parent:** R2-Rev3 approved 2026-10-23, package SHA-256
`e75582cd7f849b918ce8e8de559449144936d0ea44cba518d4bdf7188b206f4f`.

This is the successor to [the blocking-startup proposal](r2-td-002-proposal.md), which was never
approved and is now superseded for proposal purposes. Neither proposal amends an approved artifact.
See [the R1 requirement proposal](r1-session-view-amendment-proposal.md) for product authority.

## Proposed functional rules

- **R2-VIEW-01:** Startup obtains authenticated task data and the session view together. No
  account-preference GET is a prerequisite. Use remembered status/priority/sort/direction when
  valid; otherwise defaults All/All/due_date/asc. A genuinely absent view is normal first use.
  Unavailable/invalid restoration shows “Your previous view could not be restored. Showing the
  default view.” A view-settings warning never holds otherwise successful task access.
- **R2-VIEW-02:** Keep one full view tuple, not separately persisted filter/sort fragments.
  A user change immediately changes controls and requests a matching task snapshot. Persist the
  tuple separately from task mutations. Display old rows as refreshing until a matching snapshot
  arrives; discard obsolete query/session responses under existing generation guards.
- **R2-VIEW-03:** A confirmed view save is remembered for refresh/new tabs in that login. A lost
  acknowledgement is unconfirmed: say “Could not confirm your view will be remembered. You can
  keep using these filters and sorting.” Retain usable current controls, provide optional Retry,
  and never claim task saving failed because remembering the view failed. No success claim before
  commit. No indefinite spinner or task lock for this convenience failure.
- **R2-VIEW-04:** Each tab sends one view save at a time, coalescing rapid later choices to the
  newest tuple. Within the shared session, last successfully committed explicit view change wins;
  simultaneous tabs use server commit order, not browser clocks. There is no automatic stale-page
  resave or background retry. Explicit Retry means reapplying that tab's full current tuple and
  may replace another tab's remembered tuple; label this effect. An uncertain earlier request may
  commit later; a refresh/new tab sees the latest commit available when it reads. Already-open
  tabs keep their own visible tuple. These are proposed concurrency semantics needing R2 approval.
- **R2-VIEW-05:** Logout/expiry/account change invalidates the session view and pending view-save
  queue. A later login starts with no remembered tuple. Delayed requests must not revive the old
  session or attach its tuple to a new account/session. Preserve TD-001 concealment and account
  validation requirements. Preserve FD-003 memory-only pending task operations and FD-004 policy.
- **R2-VIEW-06:** Task-load failure retains FR-014 and FD-001: no new task writes or uncertain-task
  retries until a valid task snapshot. View persistence neither unlocks tasks nor modifies task
  rows/revisions/receipts. Authentication/session-store failure is not a mere view warning: deny
  private access until auth validates; never bypass ownership to make the default view work.

## Proposed HTTP and UI contract changes

Replace the account Filter preferences GET/PUT contract and `preference_version_conflict` recovery
with a session-view contract. Retain existing task mutation contracts unchanged.

| Interface | Proposed contract |
|---|---|
| GET /api/tasks/ with no view parameters | Use valid session tuple, else defaults; return ordinary snapshot, query echo and a `view_restore` value: `session`, `defaults`, or `unavailable`. Unavailable/invalid view is a successful task response only if task loading and authentication succeed |
| GET /api/tasks/ with view parameters | Supply all four fields for an explicit page selection; validate enums as before; render that query without changing shared settings. GET has no save effect |
| POST /api/session/view/ | CSRF/session authenticated; body contains exactly status, priority, sort, direction. Validate all enums; atomically replace only the current login's tuple. Return 200 with the applied tuple after commit; no expected-version or account ID accepted |
| View save failures | 400 confirmed validation rejection; 401 session invalidation; 403 CSRF; transport/5xx/malformed reply unconfirmed. Retain current-page filtering/sorting where auth and task loading succeed; offer optional reapply/Retry |
| Task-load failures | Existing task error envelope, Retry/support and FD-001 lock remain authoritative; no successful empty-list conversion |

On bootstrap, disable task writes until the task snapshot succeeds as today. There is no separate
`preference_loading`/`preference_error` gate. View notification and save status are independent UI
state alongside ready/task_load_error. No persisted browser settings, account-wide sync, or saved
version conflict UI is proposed. Requests/replies remain no-store and scoped by authenticated
identity/session generation; the server never exposes a raw authentication session key to scripts.

## Data-model and technical mechanism evaluation

Recommend a small namespaced JSON tuple in the existing database-backed Django session, bound to
the current login. It contains `schema_version`, `status`, `priority`, `sort`, and `direction`.
The schema version selects validation format; it is not an optimistic preference version.
Login initializes a clean view namespace; logout/expiry invalidates it. Benign authentication key
rotation within the same login may retain the tuple, but a new login or account change must not.
Do not tie this lifetime to tab/window closure or assume browser restart necessarily logs out.

| Candidate | Assessment |
|---|---|
| Existing server-side authenticated session | Preferred: naturally shared by same-login tabs, isolated from other sessions; no account-preference table, bootstrap GET, or new storage service |
| sessionStorage | Separate tab storage does not provide the required shared latest view for new tabs without extra coordination |
| localStorage / persistent custom cookie | Additional login binding, stale-data cleanup and expiry logic; persistent lifetime is broader than requested |
| Keep account TaskFilterPreference | Requires artificial session partitioning plus existing version/reconciliation machinery; unnecessary for the revised boundary |

[Django session documentation](https://docs.djangoproject.com/en/5.2/topics/http/sessions/) supports
server-side per-visitor data, database sessions and logout flushing. The recommendation above is a
design inference, not proof that naive dictionary writes solve concurrent session access.

**R3 implementation-design obligation:** Serialize view updates against the active session row,
re-read current session data under the write guard and replace only the view namespace. Coordinate
with SessionMiddleware saves, login/logout and session rotation so a stale whole-session save
cannot revert a newer tuple or resurrect authenticated state. Reads/snapshots must not mark settings
modified or re-save stale tuples. Acknowledgement follows durable save, not middleware intent.
R3 must assess this interaction against the actual Django backend and race tests before approval.
If it requires disproportionate custom session handling, compare a tiny session-bound settings row
as a fallback; that is a technical alternative, not an approved new model.

Remove the proposed account-owned TaskFilterPreference entity/relationship and registration-time
record only after approval. TaskListState, Task, MutationReceipt, account public_id, task ownership
and session authentication remain unchanged. There is no application data/migration here; if a
deployed preference table is later discovered, inventory it and seek a separate data-transition
decision before deletion. No prior account preference is silently imported into a new login.

## Proposed acceptance scenarios

| ID | Setup/action | Required result |
|---|---|---|
| R2-VIEW-A1 | Save Completed/High/Title descending, refresh after acknowledgement | Same tuple restored; matching task query and ordering; no preference bootstrap request |
| R2-VIEW-A2 | Open a same-login tab after save; keep an older tab open | New tab starts with shared saved tuple; existing tab need not update live; refreshing it restores shared latest commit |
| R2-VIEW-A3 | Logout/login, expire/login, switch accounts, or use another device/profile/browser | Defaults for each new/distinct login; saved tasks still persist/sync; no prior account tuple or content leaks |
| R2-VIEW-A4 | Settings absent versus invalid/unreadable; task read/auth succeed | Absent → normal defaults; invalid/unreadable → defaults plus nonblocking notice, usable controls and task actions |
| R2-VIEW-A5 | Settings work but task read fails; uncertain task operation retained | Task Retry/support and FD-001 lock including replay; settings never unlock it |
| R2-VIEW-A6 | View save commits but reply is lost; user changes filters again | Accurate unconfirmed status; current controls usable; at most one active save per tab and newest queued tuple; old reply cannot change current view or label newer choice saved |
| R2-VIEW-A7 | Two tabs explicitly save different tuples concurrently | Shared state is last server commit; existing tab views remain independent; refresh/new tab uses shared state, with no conflict prompt or cross-device persistence |
| R2-VIEW-A8 | View save races logout/new login or stale SessionMiddleware save | No old tuple/auth resurrection or overwrite of new login data; invalid session cannot save; TD-001 concealment regression checked |
| R2-VIEW-A9 | View storage cannot be read, but session authentication itself also fails | No private task data shown; authentication failure is not treated as convenience fallback |
| R2-VIEW-A10 | Lost view save acknowledgement and another tab saves before optional Retry | Retry is clearly an explicit reapply of current tuple to the shared session; no automatic overwrite/reconciliation loop; no task effect |

Extend quickstart with these cases and regress R2-T05/FD-001, R2-T07/FD-003, R2-T12/FD-004,
account isolation and all sort/tie cases. Retain R2-T06/FD-005 in review history; its account-version
cases become superseded only when the amendment is approved and incorporated. Preserve truthful
uncertainty and response-generation safety; the changed storage lifetime and overwrite policy are
intentional amendments, not claims that previously verified FD-002/005 were defective.

## Artifact impact and TD-002 disposition

| Artifact | Proposed change after required approval |
|---|---|
| spec.md | Only through the dependent R1 proposal; FR-012, targeted FR-013/AC-005 and related scenarios/provenance |
| research.md §§3/5/7/8 | Session view decision, nonblocking restore/save failures, concurrency and login lifetime; remove account preference assumptions |
| contracts/http.md | Replace preference endpoints; bootstrap snapshot view metadata; session-only POST and error/ordering rules |
| contracts/ui.md | All four view controls restore within login; defaults notice; independent settings status; reset/logout/expiry rules |
| data-model.md | Session tuple replaces account preference entity; revise filter/sort and browser state descriptions, keeping task entities and locks |
| plan.md | Storage/summary/constraints reflect session settings; remove account version-concurrency claims for view settings |
| quickstart.md | R2-VIEW-A1–A10 plus retained task/session/privacy regression scenarios |
| checklists/feature.md | Review CHK003/012/020/027/031 and dependent prompts for session view lifetime/failure/concurrency; leave reviewer-owned markers untouched |
| checklists/requirements.md | Reevaluate affected criteria after approved Scope incorporation; do not assume current checks certify proposed policy |

TD-002's original missing preference-GET failure policy becomes obsolete only if these reviewed
changes remove that mechanism. TD-002 remains OPEN now. R3 Rev2 must verify the replacement paths,
privacy/races, task-failure lock and original counterexample before recommending disposition with
explicit supersession evidence. TD-001 and TD-003–007 remain separate open findings; TD-006's backup
scope would cover session settings according to operational session handling, not a retained account
preference record. No downstream handoff is silently closed.

## Precise approval decisions

1. Independent R1 amendment review and explicit human Scope approval of the companion requirement
   replacements, especially loss of later-login/device view persistence and inclusion of sorting.
2. Independent R2 amendment review and explicit human Functional Design approval of R2-VIEW-01–06,
   including nonblocking defaults/save uncertainty, last-server-commit concurrency, and optional
   explicit reapply replacing shared settings. Endorsement in Addendum A is not formal approval.
3. After those gates, authorisation to incorporate the reviewed amendments into controlled files
   with a new decision reference and baseline manifest. Preserve historical R1/R2 approvals.
4. Independent R3 Rev2 review of the session mechanism and remaining technical corrections, followed
   by explicit human R3 approval. Session-backend race handling remains technical evidence needed;
   this proposal is not acceptance of data leakage or session resurrection.

**Formal R1/R2 amendment decisions:** PENDING. **R3:** NOT READY, human gate OPEN.
No approved artifact, finding disposition, Tasks, application code or next-phase gate is changed.
