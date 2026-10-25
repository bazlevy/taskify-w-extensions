# Data Model: Personal Task Manager

**Functional baseline:** Scope v1.2 / AM-002 and R2-Rev8 APPROVED on 2026-10-25.
[R2-Rev8 approval §4](../../independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md); FD-009 RESOLVED_VERIFIED per §2. R3 NOT_READY;
TD-008/010/011 OPEN; P1 paused. No new schema/backend/auth architecture is adopted.
See [research](research.md), [HTTP](contracts/http.md) and [UI](contracts/ui.md).

## Approved login and private-state lifetime — R2-Rev8

Genuine overlapping anonymous responses may select distinct valid logins by normal cookie
ordering, including same-account different epochs. No first-browser-winner or competitor-conflict
requirement. Authenticated request redirects remain R2-NAV-001; deliberate switching is logout-first.
New actual login defaults its view; valid login/navigation retains its own tuple, with no cross-login
import. Tasks/receipts remain account-owned; settings remain login-scoped.

Protected server operations authenticate/owner-check/intended-login guard; old intents cannot
become another login's data/effect. Known stale/mismatched replies are discarded and observed
auth changes clear prior-login state. Lifecycle/uncertainty boundaries revalidate; local cached
interactions/ordinary replies need no independent auth round trip or atomic render fence.

Approved AC-010/RA-FD006-001 permits prior-authorised display/in-flight completion while a
same-profile change is unobserved, with no maximum delay. Known mismatches/post-observation
retention and cross-account API access/mutation/rebinding remain prohibited.
FD-008 confirms intended shared revocation before ACK, promptly signals same-login tabs and
requires receiving runnable tabs to clear/navigate. No guaranteed delivery/atomic clearing;
missing messages recover at boundaries. Other devices and later-login authority remain protected.
FD-001 and silent optional-view failure rules remain unchanged.

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

Every task snapshot and mutation locks this row first within the task domain, after the outer
Session authentication guard. A snapshot copies owned rows and revision under the lock;
sorting/serialization happen after release. Mutations check receipt/version, change the task,
increment revision and record receipt atomically. Replays do not increment revision. Full order:
Session → account-state → receipt → task; no reverse acquisition. See
[R3-Rev10 protocol §3](auth-transition-protocol.md#3-admission-and-tasklogout-ordering--td-008).

### SessionViewState — approved AM-001 behaviour, technical candidate

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

R3-Rev10 selects the native Session guard and receipt below as an unapproved technical candidate.
The historical registry/transition tables remain audit text only and are removed from the planned
schema. No migration or approved Functional Design change is performed by this selection.

### Native Django Session — R3-Rev10 candidate

Retain Django's session_key/session_data/expire_date schema and supported SessionStore.
Encoded data adds immutable-per-login login_epoch (random UUID) and logout_confirmation_token
(random 256-bit secret), beside native user/backend/hash fields. Neither grants authority apart
from the native cookie. New login generates both; valid-login navigation never rewrites them.
No maintenance rekey/registry. Session-row locking and guarded fresh auth precede protected work;
ordinary responses never save auth data. Cookie-response discipline is in protocol §§3–4.

### LogoutReceipt — R3-Rev10 confirmation evidence

| Field | Type / constraint | Purpose |
|---|---|---|
| login_epoch | UUID primary key | Exact immutable login whose revocation committed |
| token_digest | 32-byte SHA-256 digest | Confirms random proof supplied by that login; constant-time comparison |
| revoked_at | UTC timestamp | Written only with committed Session/view revocation |
| retain_until | UTC timestamp, indexed | Original session expiry plus 24 hours; bounded maintenance cleanup |

No Session FK (must survive deletion), account ID, raw token/key or task contents. Only a live
matching-cookie/epoch/CSRF logout may create it; a matching existing receipt may acknowledge
that event read-only even with a newer cookie. It cannot authorise new revocation or task access.
Session flush/view deletion and receipt insert commit atomically; duplicates read the same proof.
Expiry/absence alone never inserts a receipt. After cleanup/lost memory proof, no false ACK.
R5 restore clears these receipts with Session invalidation; technical-design §7 ownership and
exit remain, with this additional fixture. No product retention/expiry guarantee is added.

### Optional view read boundary — retained TD-009

Technical guard TD-009: optional view reads have an independent warmed pool/executor, 25 ms pool
acquisition, 40 ms lock / 80 ms statement / 120 ms transaction budgets and a 150 ms total wait. No
inline retry or outer ATOMIC_REQUESTS transaction. Exhausted admission, invalid data, timeout or
isolated SQL/transport error selects full defaults silently. Quarantine unfinished work/connections;
the primary authenticated task snapshot does not wait for optional cleanup. See technical-design
§3 for worker bounds/circuit breaker. Auth/task storage failures still produce protected errors;
defaults never mean unauthenticated/private access or a fabricated empty list.

### Historical R3 snapshot only — AuthorityFamily / AuthorityHandle

**Not an operative schema or interface requirement:** the following prior candidate is retained
for audit; its mandatory registry/phase/first-winner rules are superseded as functional obligations.
R3 must remove or rejustify any mechanism against R2-Rev8, preserving security and unrelated TDs.
The snapshot's original wording follows, not current instructions:

Django remains the identity/session store. The server registry supplies the ordering floor;
Cookie header membership is not an index of pending transitions or authority completeness.

| Entity / field | Rules |
|---|---|
| Family.id | UUID; server-assigned, not accepted as caller ownership |
| Family.generation | bigint, monotonically advanced under family lock by successful current claim/activation/logout; no wrap; claims do not change current session epoch/view |
| Family.current_handle / current_session / epoch | Nullable pinned current authority; validate ACTIVE/live, never select another surviving cookie |
| Family.status | ACTIVE / SIGNED_OUT; missing row cannot be recreated by ordinary auth middleware |
| Handle.digest | Unique SHA-256 digest of 256-bit random secret; raw value only in HttpOnly cookie |
| Handle.family / transition | Immutable FKs; transitions indexed by family, discoverable without their cookie |
| Handle.state / expires_at | PROVISIONAL / ACTIVE / RETIRED / REVOKED, absolute existing credential/session expiry; ACTIVE must also equal family.current_handle |

Only __Host-taskify-authority is an authentication cookie. One 43-character value, full Set-Cookie
under 512 bytes; duplicate/missing/unknown/provisional/noncurrent/expired values give 401 without
private data. Database uncertainty gives 503/concealed Retry. No client-side high-water/floor
cookies or historical per-attempt credentials. A cookie can be lost at any time; missing means
unauthenticated, never restore an old account.

### Historical R3 snapshot only — AuthTransition

| Field | Rules |
|---|---|
| id / family_id | Random UUID, required indexed family FK registered before issuance; reference alone grants no private authority |
| kind | login candidates only, anonymous delivery redesign HOLD; rotation/logout are single guarded family transactions, never provisional-cookie candidates |
| captured_generation / old_handle / old_session | Server-captured expected authority; recheck under family lock on every phase |
| candidate_session / epoch | Fresh-login candidate only; no maintenance provisional handle; new-login delivery contract remains HOLD, not approved schema |
| state | PREPARED → ISSUED → ACTIVE; incomplete → CANCELLED / EXPIRED; no terminal-to-ACTIVE |
| created_at / deadline | Database time; deadline exactly 120 seconds after PREPARE, never extended |
| issue_attempted / preparation proof | At-most-once cookie delivery; CSRF-bound opaque issue proof, not an auth credential |

Navigation invariant: entry/login GET in a live session shares that session/view, creating no
Family/Handle/Transition/epoch. Authenticated login POST also redirects/rejects with no reset.
Login PREPARE cannot attach to an active family; ACTIVATE(kind=login) rechecks no live current
session under family lock. Active-family replacement is permitted only for same-account rotation.
Existing claim/generation/registry guards remain inherited candidates for stale requests/rotation/
logout; their sufficiency for R2-Rev6 still needs R3 proof. No new schema field or cookie is selected
here. R2-NAV-001 is CLOSED — approved by Barry in R2-Rev5 §7.

At most four incomplete transitions/family; fifth valid PREPARE cancels oldest incomplete under
the family lock. Cleanup every 60 seconds uses family → transition/handle → Session → view.
Delete terminal records/provisional rows under that guard; unknown/deleted late request rejects,
never upserts. Live family/current handle must survive for actual Django session lifetime;
delete a signed-out/expired family only without live pointers/nonterminal transitions/references.
Keep the current ACTIVE handle and its referenced transition while live; FKs prohibit deletion
of live references. One maintenance connection uses the existing diagnostics/migrations budget;
64-family batches, up to 16 per minute, existing DB timeouts. Alert on two missed runs or oldest
eligible backlog above 120 seconds. The protocol defines the 1,500-admission burst fixture and
live-session/storage measurements; cleanup delay can never extend or restore authority.
No minimum retained cookie-floor horizon is necessary: old unknown/noncurrent handles cannot
authenticate. Lifecycle deletion must not cascade-delete a still-live authority record.
These bounds are engineering admission/cleanup defaults, not user/account/task caps.

The earlier rule allowing a provisional cookie to overwrite the active cookie is withdrawn.
It does not preserve live-session continuity/shared logout under anonymous overlap. Candidate
delivery/selection redesign remains HOLD; P1's atomic browser enrolment is a separate unapproved
proposal and cannot be assumed in this schema.

Maintenance locks Family → Session → view, creates new server-only Session key, transfers complete
valid view/auth data, updates family pointer/deletes old rows and advances generation atomically.
Retarget live FKs first; historical candidate-session references must not cascade-delete the
current Family/Handle/login-transition record when the retired inner Session is removed.
Keep current AuthorityHandle ACTIVE and unchanged; no auth-cookie/CSRF write or login hook.
Same-handle/same-account/epoch requests resolve current inner key under guard, not auth 401 for
a retired captured inner key. Never rebind across logout/new epoch or resurrect old rows.
Logout remains immediate family/handle/Session/view revocation plus pending-candidate cancellation.
Cross-tab CSRF tokens are fetched/refreshed outside domain mutation; see protocol §3.

End of historical authentication snapshot. The following task/domain entities retain their
unaffected operative rules.

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

Deletion physically removes the Task row. There is no deleted flag, recycle bin, task archive or
restore endpoint. Receipts retain no plaintext command/response for restoring task content; keyed
digests and metadata do not imply absolute erasure. Separate DB/key compromise, backups and live
process exposure are addressed in technical-design §5 rather than assumed impossible.

### MutationReceipt

| Field | Type | Rules |
|---|---|---|
| id | internal primary key | Implementation detail |
| owner_id | User FK | Required, indexed |
| operation_id | UUID | Unique with owner; generated once per logical client operation |
| request_digest | HMAC-SHA-256 hex | Keyed canonical command digest; constant-time comparison; key stored outside database |
| canonical_version | positive integer | Starts at 1; historical canonicalizer retained for replay |
| digest_scheme | text enum | hmac-sha256-v1; versioned separately from canonical encoding |
| key_id | opaque nonsecret text | Identifies independently protected current or verify-only receipt key |
| kind | text enum | create, edit, set_status, delete |
| task_id | UUID scalar | No task FK; survives task deletion |
| committed_version | nullable bigint | Resulting version; null for delete |
| list_revision | bigint | Revision produced by this operation |
| committed_at | timestamptz | Server timestamp |

Only committed successes generate receipts. Validation/auth/version errors do not. The task effect
and receipt commit atomically. Look up owner/operation receipt before current validation/task
existence; use its stored canonical_version/digest_scheme/key_id to verify replay with the original
encoding/key, including default-equivalent values. Compare digests in constant time; a different
canonical command yields 409 operation_key_reused. Technical-design §5 fixes format-1 ordering,
domain separation and Unicode/default handling. Missing key/canonicalizer returns 503
receipt_verification_unavailable without a new effect, never key-reuse or a replacement operation.

Keep receipts for account lifetime in this release. No expiry job. Receipts are not a task history and cannot restore deleted task content. They acknowledge a historical operation, so a replay may succeed even when a later operation deleted or changed that task. A new snapshot determines current state.

Historical receipt keys remain verify-only for as long as referenced, outside DB backups in
restricted separate secret custody. Version/key inventory is a deployment/restore preflight.
Ordinary rotation cannot destroy a referenced key. R3-V04/V07 cover new-key writes, old replay,
missing key, changed defaults and compatible rollback. Existing task fields and list revision
semantics are unchanged.

### AuthThrottleBucket — TD-005 operational state

An internal expiring fixed-window counter keyed by HMAC(dimension, normalized source/username
identity, window). Store counter_key, window_start/end, attempt_count and expires_at, with unique
bucket key and cleanup index. It contains no task content, credentials, raw session key or product
account lock flag. All workers share atomic database updates in sorted-key order; isolated limiter
transactions do not hold task/session locks. Expire after window end plus 1 hour and run bounded
hourly cleanup. Technical-design §6 defines policy dimensions, separate secret, trust boundary,
bounded pool, 429 and fail-closed 503. This is deployment abuse control, not a new user entity.

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

Every server operation remains authenticated/owner/intended-login guarded. Local responses use
known identity/query/generation/revision checks; boundary uncertainty revalidates/conceals.
No universal fresh-interaction or atomic render gate. Auth success alone cannot unlock a task
load_error or settle outcome_unknown.

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

Authority states include concealed boundary/revalidating, known-login visible, observed-invalid/
cleared, and logout pending/unknown vs confirmed. Suspended/history/focus-restored tabs start
concealed until appropriate validation. Known mismatches invalidate/discard old replies; observed
change clears rows/labels/drafts/dialogs/task-view queues. Old intents never bind to a new epoch.
Prior-authorised content may remain/finish rendering during an unobserved change under AC-010;
this is not a server authorisation or known-mismatch exception.

After confirmed logout, promptly dispatch notification; receiving runnable same-login tabs clear/
navigate. Delivery may fail/delay: boundary recovery and synchronisation remain, without guaranteed
delivery/tab ACK/atomic clearing. Old-L notices cannot revoke N. No success from 401/403, cookie
absence or UI clearing. Ordinary task errors keep stale rows/FD-001 locks; auth uncertainty
conceals and view failure never defaults authentication. Historical family/claim/maintenance
details are not the current architecture; R3 chooses a proportionate compliant mechanism.

Read failures never replace saved data with an empty list. While loading is unresolved, the UI must
not submit any task mutation, including a retry that might be the first server-observed operation.
After a valid snapshot, an exact retry remains subject to receipt and version checks. The mutation
lock is a UI safety measure; server endpoints always perform validation, ownership and version
checks independently. UI state cannot establish an absolute global freshness guarantee across
devices; task versions protect concurrent changes.

View restoration/remembering is an independent convenience state and does not enter task
load_error/outcome_unknown. When only it fails, keep controls/current view usable silently. Session
invalidation clears the view-save queue alongside existing task/draft clearing. TD-001 concealment
and TD-008/009 lifecycle/fault containment now have candidate mechanisms and planned checks in
technical-design; independent finding verification remains pending.
