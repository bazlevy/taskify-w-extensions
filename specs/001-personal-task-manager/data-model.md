# Data Model: Personal Task Manager

See [research.md](research.md) for decisions and [contracts/http.md](contracts/http.md) for wire formats. This describes the planned schema and domain entities; it is not a migration.

## Persistent entities

### User Account — accounts.User

Minimal Django AbstractUser subclass configured as AUTH_USER_MODEL before the initial migration. Standard fields include primary key, unique username, password hash, active flag and creation metadata. Add an immutable unique UUID `public_id` as the opaque account identifier returned in authenticated snapshots; internal numeric IDs need not be exposed.

Registration accepts username/password/confirmation through Django validation. Reject case-insensitive username collisions, including concurrent registration, with a database uniqueness constraint on Lower(username) as well as application validation. No invitation or email is required. Password values never enter logs or task DTOs. Accounts are never accepted from client-submitted ownership fields.

Relationships: one account owns one TaskListState, one TaskFilterPreference, zero or more Tasks,
zero or more MutationReceipts and database-backed sessions. Use Django's supported session
model/storage rather than a custom session entity.

### TaskListState

| Field | Type | Rules |
|---|---|---|
| owner_id | User FK, primary key / one-to-one | Required, unique, indexed |
| revision | bigint | Nonnegative; starts at 0; increases once per successful new mutation |

Create with the account in the same registration transaction. No separate user-visible list record, list name or list-management interface.

Every task snapshot and mutation locks this row first. A snapshot copies owned rows and revision under the lock; sorting/serialization happen after release. Mutations check the receipt, validate version, change the task, increment revision and record the receipt under one atomic transaction. Replays do not increment revision. All operations follow account-state → receipt → task lock order.

### TaskFilterPreference

One account-owned record, created with the account in the registration transaction.

| Field | Type | Rules |
|---|---|---|
| owner_id | User FK, primary key / one-to-one | Required, unique, indexed |
| status | text enum | all, active, completed; starts all |
| priority | text enum | all, low, medium, high; starts all |
| version | bigint | Positive; starts at 1; increments once per accepted preference save |
| updated_at | timestamptz | Server-generated UTC instant on accepted preference save |

Preference updates lock this record and require its displayed version. A stale update returns the
authoritative saved pair and version without mutation. Preference records are independent of task
list revision and task-mutation receipts: saving filters never modifies task data and does not
relax the task-write lock during a saved-task load failure. The browser records the latest confirmed
pair/version separately from its current visible pair. It sends one preference update at a time;
later user selections replace a single queued desired pair. A confirmed acknowledgement advances
the confirmed version. A rejected or outcome-unknown update requires an authoritative re-read
before any new overwrite attempt; a lower-version or stale client-generation response is ignored.

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
filter never mutates tasks. The authoritative selected pair and preference version are persisted in
the account's TaskFilterPreference record; defaults are All/All. The current page may display an
unconfirmed attempted pair after a failed or outcome-unknown preference save, but only an
acknowledged or re-read matching record restores on later sessions/devices.

### Task Sorting

Method: due_date, priority, created_at, title. Direction: asc/desc. User labels: earliest/latest, Low–High/High–Low, oldest/newest, A–Z/Z–A. First-use and fresh-page default: due_date/asc. Session memory only.

Sort only the primary key in the chosen direction. Due dates always place null last, then tie by descending priority, ascending creation timestamp and ascending UUID. Other primary ties use ascending creation timestamp then UUID; creation-time ties use UUID. Title key is NFC normalization followed by casefold and Unicode code-point comparison. Task details are not changed by sort/filter.

### Overdue

`status == active AND due_date != null AND due_date < viewing_device_local_today`.

Compare calendar dates directly. Recompute at local midnight, on focus and after status/due-date changes. No persisted overdue flag. The JS calendar module handles year boundaries and dates below year 0100 without Date-constructor year remapping.

### Snapshot / mutation acknowledgement

Snapshot: account public ID, monotonic list revision, query echo, total and matching counts, ordered matching task DTOs. Copy the full account-owned task set and revision consistently, then derive the requested filtered ordering. No pagination/delta tokens initially.

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
