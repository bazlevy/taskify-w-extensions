# Data Model: Personal Task Manager

See [research.md](research.md) for decisions and [contracts/http.md](contracts/http.md) for wire formats. This describes the planned schema and domain entities; it is not a migration.

**Revision:** 2026-10-23, aligned to Scope 1.1 / AM-001. Targeted R2 review and human Design approval
are pending; the historical R2-Rev3 package and findings remain unchanged.

## Persistent entities

### User Account — accounts.User

Minimal Django AbstractUser subclass configured as AUTH_USER_MODEL before the initial migration. Standard fields include primary key, unique username, password hash, active flag and creation metadata. Add an immutable unique UUID `public_id` as the opaque account identifier returned in authenticated snapshots; internal numeric IDs need not be exposed.

Registration accepts username/password/confirmation through Django validation. Reject case-insensitive username collisions, including concurrent registration, with a database uniqueness constraint on Lower(username) as well as application validation. No invitation or email is required. Password values never enter logs or task DTOs. Accounts are never accepted from client-submitted ownership fields.

Relationships: one account owns one TaskListState, zero or more Tasks, zero or more MutationReceipts
and database-backed sessions. Each authenticated session may have one SessionViewState. It is not
an account-level preference and is never imported into another session. Use Django's supported
session model/storage for authentication rather than inventing a replacement identity store.

### TaskListState

| Field | Type | Rules |
|---|---|---|
| owner_id | User FK, primary key / one-to-one | Required, unique, indexed |
| revision | bigint | Nonnegative; starts at 0; increases once per successful new mutation |

Create with the account in the same registration transaction. No separate user-visible list record, list name or list-management interface.

Every task snapshot and mutation locks this row first. A snapshot copies owned rows and revision under the lock; sorting/serialization happen after release. Mutations check the receipt, validate version, change the task, increment revision and record the receipt under one atomic transaction. Replays do not increment revision. All operations follow account-state → receipt → task lock order.

### SessionViewState — AM-001 replacement design, approval pending

Optional view-convenience record tied to a single current Django login session; registration does
not create an account preference record. An absent record is normal first use.

| Field | Type | Rules |
|---|---|---|
| session_id | One-to-one FK to Django Session.session_key; primary key | Server-derived, never accepted from JSON or exposed; cascade deletion |
| schema_version | positive integer | Current tuple format 1, not an optimistic conflict version |
| status | text enum | all, active, completed |
| priority | text enum | all, low, medium, high |
| sort | text enum | due_date, priority, created_at, title |
| direction | text enum | asc, desc |

Replace all four values atomically; valid schema and every component are required. A missing,
incomplete, invalid or unreadable record means the entire default tuple all/all/due_date/asc,
silently, provided authentication and task loading succeed. Do not repair or persist defaults on a
GET. Task data, task versions, account list revisions and MutationReceipt are unaffected.

The browser applies its current tuple independently of remembering. It sends one view save at a
time and retains only the newest queued selection. An old reply cannot change newer controls or
claim a newer tuple was remembered. Failures/unknown outcomes cause no view warning or automatic
retry; a later explicit user selection can save the latest tuple. Concurrent tabs use last server
commit order; a refreshed/new tab reads the latest committed valid tuple at read time. A previously
uncertain request may still commit; no wall-clock or exactly-once view-intent guarantee is added.

Technical guard for R3 review: capture the request's original server session key; in a short atomic
save lock the live Django session row before the view row, validate existence, expiry and that its
authenticated user still matches, then upsert the tuple. Do not create/replace authentication
session records while saving a view. Coordinate logout/new-login invalidation using the same guard:
an earlier save may commit before invalidation and then be deleted; an old save after invalidation
must fail. Every explicit successful login uses a fresh session key with no imported view, even
when login is for the same account; other valid sessions remain independent. Routine page refresh
or closing a tab does not create a login. Preserve a view across non-login session-key maintenance
within a valid login through a guarded move; invalidate the old key. R3 must assess this lifecycle
and middleware races before approval. Expired rows must never be restored even before cleanup.

On a view-read failure, roll back its isolated read/savepoint before the independent task snapshot
transaction; never swallow an error inside an aborted transaction and claim task loading succeeds.
An authentication/database failure preventing task access still produces its normal protected error.

### Task

| Field | Type | Rules |
|---|---|---|
| id | UUID primary key | Stable client-generated v4 UUID on create; never changes |
| owner_id | User FK | Required, indexed; session-assigned; immutable |
| title | text | Required, Unicode-trimmed, at least one non-whitespace character |
| description | nullable text | Omitted/null/empty string becomes null; nonempty content preserved |
| priority | text enum | low, medium, high; missing/null/empty input normalizes to medium |
| due_date | nullable date | Real calendar date, years 0001–9999; no time; past permitted |
| status | text enum | active or completed; creation always active |
| created_at | timestamptz | Server-generated UTC instant; immutable |
| updated_at | timestamptz | Server-generated UTC instant on accepted mutation |
| version | bigint | Starts at 1; positive; increments on each new edit/status command |

DB constraints enforce title nonempty after database trimming, enum membership, positive version and owner FK. Application Unicode-aware validation is authoritative for whitespace beyond database trim semantics; tests cover this boundary. Use Django migrations for constraints/indexes. No artificial product title/description length limit; configure request-size protection separately and report oversize requests rather than truncating content.

Default FK indexing supports owner-scoped reads. Do not add speculative indexes for all sort combinations: the first design materialises an account snapshot and performs domain sorting. Measure before adding indexes or pagination.

Deletion physically removes the Task row. There is no deleted flag, recycle bin, task archive or restore endpoint. Receipts retain no recoverable title/description/date payload.

### MutationReceipt

| Field | Type | Rules |
|---|---|---|
| id | internal primary key | Implementation detail |
| owner_id | User FK | Required, indexed |
| operation_id | UUID | Unique with owner; generated once per logical client operation |
| request_digest | fixed SHA-256 hex | Digest of canonical validated command, route/kind, target ID and expected version |
| kind | text enum | create, edit, set_status, delete |
| task_id | UUID scalar | No task FK; survives task deletion |
| committed_version | nullable bigint | Resulting version; null for delete |
| list_revision | bigint | Revision produced by this operation |
| committed_at | timestamptz | Server timestamp |

Only committed successes generate receipts. Validation/auth/version errors do not. The task effect and receipt commit atomically, so an unknown transport outcome can be replayed. Compare normalized canonical payloads; missing/default-equivalent values must canonicalize consistently. The same owner/key with a different digest produces 409 operation_key_reused.

Keep receipts for account lifetime in this release. No expiry job. Receipts are not a task history and cannot restore deleted task content. They acknowledge a historical operation, so a replay may succeed even when a later operation deleted or changed that task. A new snapshot determines current state.

Account deletion is not exposed by this feature. Any future operational account deletion must handle dependent rows and sessions as a separate authorized workflow.

## Nonpersistent entities and derived values

### Task Filter

Status: all, active, completed. Priority: all, low, medium, high. Combine predicates with AND. A
filter never mutates tasks. Selection is part of the complete session view; the current visible
tuple may differ from what was successfully remembered. Refresh/new tabs restore available valid
remembered state or complete defaults, never an unconditional promise about an unconfirmed save.

### Task Sorting

Method: due_date, priority, created_at, title. Direction: asc/desc. User labels: earliest/latest,
Low–High/High–Low, oldest/newest, A–Z/Z–A. New-login default: due_date/asc; restore method/direction
with filters within the same still-valid login. Missing/invalid view uses complete defaults.

Sort only the primary key in the chosen direction. Due dates always place null last, then tie by descending priority, ascending creation timestamp and ascending UUID. Other primary ties use ascending creation timestamp then UUID; creation-time ties use UUID. Title key is NFC normalization followed by casefold and Unicode code-point comparison. Task details are not changed by sort/filter.

### Overdue

`status == active AND due_date != null AND due_date < viewing_device_local_today`.

Compare calendar dates directly. Recompute at local midnight, on focus and after status/due-date changes. No persisted overdue flag. The JS calendar module handles year boundaries and dates below year 0100 without Date-constructor year remapping.

### Snapshot / mutation acknowledgement

Snapshot: account public ID, monotonic list revision, effective full query echo, total and matching
counts, ordered matching task DTOs. On a parameterless bootstrap choose a valid session view or
complete defaults; explicit full queries select only the current page. Copy the full account-owned
task set and revision consistently, then derive the requested filtered ordering. No pagination/delta
tokens initially. Settings fallback never converts a failed task snapshot to a successful empty list.

Acknowledgement: operation ID, kind, task ID, committed version (null for delete), committed list revision and replayed flag. Never reconstruct a deleted task from a receipt.

### Browser view and operation state

| State | Permitted behavior | Transition |
|---|---|---|
| loading | Display progress; no task writes | Valid snapshot → ready; failure → load_error |
| ready | Read/filter/sort/create/edit/status/delete-confirm | Submit → writing; failed refresh → load_error |
| writing | One operation pending; block other writes | Ack → refreshing; definite rejection → ready/conflict; uncertain response → outcome_unknown |
| refreshing | Await snapshot at least as new as acknowledged revision | Success → ready; failure → load_error |
| load_error | Stale/empty-unavailable view, retry/support; no task writes, including pending-operation retries | Valid snapshot → ready, or → outcome_unknown when a pending unknown operation remains |
| outcome_unknown | Preserve exact request/draft; after a valid snapshot, permit only an exact same-ID/payload retry | Historical ack → refreshing; definitive rejection → conflict/ready; read failure → load_error; failed recovery → remain locked |
| conflict | Preserve draft; show current task/deletion; no blind resubmit | Refresh/review → new explicit operation with current version, or cancel |
| signed_out | Hide/clear task data and drafts | Authenticate and new generation → loading |

Read failures never replace saved data with an empty list. While loading is unresolved, the UI must
not submit any task mutation, including a retry that might be the first server-observed operation.
After a valid snapshot, an exact retry remains subject to receipt and version checks. The mutation
lock is a UI safety measure; server endpoints always perform validation, ownership and version
checks independently. UI state cannot establish an absolute global freshness guarantee across
devices; task versions protect concurrent changes.

View restoration/remembering is an independent convenience state and does not enter task
load_error/outcome_unknown. When only it fails, keep controls/current view usable silently. Session
invalidation clears the view-save queue alongside existing task/draft clearing. TD-001's full
cross-tab concealment mechanism remains an open R3 dependency; session-scoped views alone do not
solve it.
