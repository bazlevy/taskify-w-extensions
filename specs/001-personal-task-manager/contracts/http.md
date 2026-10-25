# HTTP Contract

**Approved functional baseline:** Scope v1.2 / AM-002 and R2-Rev8.
[R2-Rev8 approval §4](../../../independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md) records Barry Levy's 2026-10-25 decision; §2 records
FD-009 RESOLVED_VERIFIED. Historical R2-Rev5/R2-NAV-001 and R2-Rev6 approvals are preserved;
targeted supersession only. R3 NOT_READY; TD-008/010/011 OPEN. P1/IndexedDB paused.

Same-origin interface; cookies authenticate and caller-supplied account IDs are not authority.
See [UI](ui.md) and [data model](../data-model.md). No new architecture or runtime proof is
approved. Prior technical interfaces are historical inputs, not functional requirements.

## Pages and authentication

| Method/path | Behavior |
|---|---|
| GET / | Live authenticated login: 302 /tasks/; signed out/expired: 302 /accounts/login/; authority lookup failure: 503, never assumed anonymous |
| GET /accounts/register/ | Username/password/confirmation form with CSRF token and no-recovery disclosure |
| POST /accounts/register/ | Validate and create private account plus list state atomically; 302 to sign-in on success; 200 form with field errors on failure |
| GET /accounts/login/ | Authenticated: 302 /tasks/, no form/new login. Signed out/expired: 200 sign-in form and existing registration/no-recovery links. Lookup failure: 503 Retry; provisional state: reconcile, never implicit login |
| POST /accounts/login/ | CSRF first; authenticated: 303 /tasks/ before replacement credential evaluation, same login/view. Genuine overlapping anonymous requests may create distinct valid logins selected by ordinary browser cookie ordering; no winner-enforcement 409 required. Storage uncertainty: 503; invalid CSRF: 403; generic form errors retained |
| POST /accounts/logout/ | CSRF + intended login; success only after shared login/view revocation commits, then sign-in destination and prompt tab notification/clearing/navigation per UI. Distinct valid late login may authenticate under S2; revoked login stays invalid. GET must not log out |
| GET /tasks/ | Authenticated responsive task shell with public account UUID and CSRF token; bootstrap tasks directly, without a preference GET; anonymous requests redirect to login |
| GET /support/ | Support instructions and configured contact; accepts optional nonsensitive request reference; no data reset, password reset/recovery, or account transfer |

Use Django's validated same-origin next handling, not arbitrary redirects. Registration displays that
forgotten credentials cannot be recovered in this release before account creation. Rejected
registration identifies invalid/duplicate username or invalid password/confirmation without
creating a partial account. Login failures do not reveal account existence. Forgotten-credentials
and support text must state that support cannot reset credentials or transfer account access.
Production auth admission is enforced by the shared application limiter behind the trusted ingress;
technical-design §6 specifies source normalization, database counters, exact initial engineering
budgets, 429/Retry-After and fail-closed 503. The ingress strips untrusted forwarding headers and
the application is not publicly reachable except through that peer. Invalid credentials remain
generic; no permanent account lockout, password reset or credential transfer is introduced.

### Authenticated navigation and stale forms — FR-010

Same-profile new tabs share the still-valid login and reach /tasks/ directly. Login GET redirects
authenticated requests to that fixed local path regardless of next/username parameters; no
Session, cookie, epoch or view writes. Avoid redirect loops; shell authority/snapshot checks
remain mandatory. Logout racing a redirect may require sign-in when the shell validates.

Login POST checks CSRF, resolves live authority and returns 303 /tasks/ when authenticated,
before credential validation/candidate creation. CSRF failure stays 403 without changing login.

**FD-007 / CA-007:** Already-submitted anonymous requests may both succeed; subsequent requests
use the actual current valid cookie. Later application can select a different login/epoch without
intervening logout. No first-browser/server/submission winner or required competitor conflict.
This exception does not evaluate replacement credentials on an authenticated request.
Normal deliberate switching remains logout-first; restored stale forms revalidate.

Each actual new login defaults its own view; valid-login navigation retains its tuple. On
observed selection change, discard prior-login client state and load only the selected account/
login's view. Never import drafts/queues/settings. No account-wide preference semantics.

S2 permits a distinct valid late login after confirmed revocation; revoked/invalid credentials
stay invalid. No obligation cancels/revokes every other valid anonymous session. Selected identity
change alone never proves the intended old logout. No credential/task automatic replay.

R2-NAV-001 stays CLOSED/APPROVED under R2-Rev5; historical approvals remain exact. CSRF, native
fixation protection, intended-login guards and lookup uncertainty protections remain mandatory.

### Technical interface boundary

The former PREPARE/ISSUE/ACTIVATE, family claim/reconciliation and custom rotation routes are
historical unapproved R3 proposals, not requirements of the operative conventional-auth contract.
See [protocol history](../auth-transition-protocol.md). R3 must select/rejustify minimal mechanisms;
this incorporation does not choose a new route/schema/transport or retire security protections.
No stale internal compatibility path may bypass authentication, ownership or intended-login checks.

GET /api/csrf/ remains same-origin/no-store, returning a masked token with only CSRF-cookie
side effects; no auth Session/task/view/default mutation. No required separate auth probe per
normal interaction or private response. Lifecycle/uncertainty revalidation is defined in UI.

### CSRF token handoff and recovery

A cached pre-login form token can be stale after another tab logs in. Obtain GET /api/csrf/
at required restoration/stale-token recovery before the unsafe request, then send current X-CSRFToken. GET changes only
the CSRF cookie, not auth/session/task/view state. Keep existing Origin/Referer/no-CORS checks.

Trusted application 403 csrf_failed with outcome=rejected is generated before domain dispatch.
One automatic claim/reconciliation retry may follow fresh token + authority revalidation.
For logout, capture intended epoch on click and conceal/clear private state immediately; one
retry is permitted only if read-only authority still matches that epoch. If identity changed,
never submit old logout against the new account. No success until actual shared revocation ACK.
Second rejection/failure uses concealed Retry/support or safe no-store shell reload.
A generic/proxy HTML 403 or transport loss is not trusted no-effect evidence.

Task commands do not automatically replay: preserve original ID/payload, obtain fresh token,
validate authority and saved tasks, then normal explicit retry subject to FD-001/version/receipt.
View remembering stays silent, with fresh token for a later explicit choice and no old-tuple
automatic replay. Login token recovery navigates authenticated users to tasks under R2-NAV-001;
anonymous passwords are never automatically resubmitted. CSRF refresh cannot unlock task writes
while saved-task loading fails. Full state/failure rules and R7-T04–07 are in protocol §3.

## Common JSON rules

Prefix: /api/. Send/receive application/json, UTF-8. Unsafe methods require valid same-origin CSRF token (X-CSRFToken). API authentication failure is 401 JSON, not an HTML sign-in redirect. GET has no mutation side effects. Authenticated pages/API responses use Cache-Control: no-store.

**FD-006 / CA-006 and aligned Q1:** Every protected request validates live authentication, owning
account and intended login. Task/view requests retain X-Taskify-Login-Epoch; invalid/revoked login
or absent/stale intended epoch returns 401 authentication_required before protected access/effect.
Authority lookup uncertainty returns 503, never anonymous success or private-default fallback.
Cookies/CSRF/owner checks remain mandatory. Do not rebind old payloads to another login.

Clients reject known account/epoch/query/generation/revision mismatches and invalidated replies;
clear private state on observed auth change. Revalidate at initial protected entry, restored
focus/resume/history/stale forms and uncertainty/recovery boundaries. No mandatory independent
auth round trip for each cached interaction, normal response/refresh or atomic rendering.
Approved AC-010/RA-FD006-001 permits prior-authorised content/in-flight completion while a change
is unobserved; known mismatch, cross-account API access/mutation and old-intent reuse remain
prohibited. See UI for bounded display-risk and task-error distinctions.

Logout acknowledges only confirmed intended shared revocation; every revoked-credential replay
fails. Preserve unknown-outcome recovery; old intent cannot log out a new identity. Prompt
notification/clearing/navigation and suspension/loss handling follow [UI](ui.md).
Committed pre-revocation task effects persist; no command under revoked authority may commit
after logout success. Ordering proof remains R3 work.

Auth housekeeping cannot unlock FD-001 task mutations/retries without a valid saved-task snapshot.
No GET writes auth Session/task/view/defaults; GET /api/csrf/ may set only its CSRF cookie.

Dates: null or strict valid YYYY-MM-DD, no timezone/time. Timestamps: UTC ISO-8601 strings with Z and microsecond precision. Task versions and list revisions: nonnegative decimal strings in JSON to avoid JavaScript integer precision loss; task versions start at "1". Counts are ordinary nonnegative JSON integers.

Unknown fields and unsupported enum values are rejected with 400; do not silently accept owner/status/created_at on creation or owner/id/created_at changes on edits. Malformed JSON, invalid UUID/version syntax and invalid query values return 400. Unsupported media type returns 415. An oversized request returns 413 without truncation. Method mismatch returns 405. API errors follow this envelope, including CSRF and relevant framework errors:

```json
{
  "error": {
    "code": "validation_error",
    "message": "Correct the highlighted fields.",
    "fields": {"title": ["Enter a title."]},
    "request_id": "opaque-reference",
    "outcome": "rejected"
  }
}
```

Fields is an object, empty for non-field errors. References contain no task text or credentials. outcome=rejected is emitted only for a known noncommitted attempt. Unknown/invalid responses and transport failures cannot establish rollback. API 5xx errors are treated as outcome-unknown unless a trusted application response explicitly establishes rejection.

## Read snapshot

Bootstrap: GET /api/tasks/ (no query parameters).

For bootstrap, use the complete valid view remembered in the current login, or all/all/due_date/asc
if any view component is absent, invalid or unreadable. Do not warn or hold otherwise available
tasks solely for view-state failure. After bootstrap, each refresh/poll/filter/sort request includes
all four current page values, for example
GET /api/tasks/?status=all&priority=all&sort=due_date&direction=asc. Explicit partial queries return
400 validation_error; unknown fields or enum values are rejected. status: all/active/completed;
priority: all/low/medium/high; sort: due_date/priority/created_at/title; direction: asc/desc. Domain
services apply existing deterministic filtering/ordering. GET never remembers settings or writes
defaults. TD-009 optional view lookup has no queued admission, a warmed dedicated pool and a 150 ms
total wait (25 ms pool, 40 ms lock, 80 ms statement, 120 ms transaction budgets); failed or late work
is isolated/cancelled/quarantined and full defaults proceed on an independent task connection. No
retry waits for the optional lookup. Authentication is mandatory before this fallback, and primary
task snapshot errors still invoke FR-014. See technical-design §3 for bounded jobs/circuit breaker.

200 response:

```json
{
  "account_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "login_epoch": "dddddddd-dddd-4ddd-8ddd-dddddddddddd",
  "revision": "12",
  "query": {"status": "all", "priority": "all", "sort": "due_date", "direction": "asc"},
  "total_count": 1,
  "matching_count": 1,
  "tasks": [{
    "id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
    "title": "Book appointment",
    "description": null,
    "priority": "medium",
    "due_date": null,
    "status": "active",
    "created_at": "2026-10-22T04:00:00.000000Z",
    "updated_at": "2026-10-22T04:00:00.000000Z",
    "version": "1"
  }]
}
```

The response contains the complete matching set at its revision. total_count counts all owned tasks; matching_count equals tasks.length. Valid empty results are successful snapshots. Failure/malformed data is never converted to a successful empty result. Serialize only after materialising a consistent owner-scoped snapshot under the account-state lock.

Clients check response identity against the known account/epoch plus query/full schema/revision
and local generations; discard known mismatches or invalidation. Apply lifecycle/uncertainty
revalidation, without a mandatory independent auth probe for each arriving response or atomic
render guarantee. Approved AC-010 governs unobserved prior-authorised display. Discard wrong-session, wrong-query or older-generation responses, revisions below the latest accepted revision, and revisions below the latest acknowledged mutation. Different query results may legitimately have equal revisions.

## Session view remembering — revised AM-001 contract

POST /api/session/view/ accepts exactly the full current view:

```json
{"status":"completed","priority":"high","sort":"title","direction":"desc"}
```

The server derives the active session/owner from authentication; reject caller-supplied session,
account IDs, expected_version and unknown fields. Require valid enums, CSRF and same-origin
credentials. Atomic replacement affects only the current login's SessionViewState; successful 200
returns the applied tuple after commit. No task version, list revision or mutation receipt changes.
There is no account-preference GET/PUT bootstrap, preference version conflict, durable view retry
journal, or cross-device view persistence. The old /api/preferences/filters/ routes are removed from
the planned interface; no compatibility write to account preferences is specified.

Client saves one tuple at a time and coalesces rapid later choices to the latest queued tuple.
Explicit user choices in different tabs use last successful server commit order; the visible view
in an existing tab never changes merely because another tab saved. On any save failure or lost/
malformed reply, retain the current working selection silently. No view-error toast, retry dialog,
automatic repeat of the uncertain tuple, or claim of successful restoration. A queued newer
explicit selection may be submitted after the preceding request settles; no request means a promise
that the shared state cannot subsequently be changed by a delayed earlier commit or another tab.
A reply associated with an older selection/session cannot change newer controls or confirm that
newer tuple. The next explicit change may attempt remembering again; refresh/new tabs use actual
complete valid remembered settings or all four defaults.

Errors keep the common envelope: 400 validation_error for a definite invalid tuple, 401 for absent/
expired authentication, 403 for CSRF, and 503 where an unavailable save can be reported. Transport,
5xx and malformed responses cannot prove rollback. View-only failures have no user warning and do
not lock available task actions. Authentication failures always follow account clearing/sign-in
rules; they cannot be suppressed as convenience errors. A normal task snapshot error still
requires FR-014/FD-001 error/Retry/support and task-write protection. A view-save endpoint must not
write authentication session data or resurrect an invalidated session. No GET writes state.

## Mutations

All task mutations are POST. One logical operation uses one operation_id UUID, preserved with the
exact submitted request for retries. Create also retains one task UUID. Ownership comes from the
authenticated session and epoch guard. After basic wire/auth checks, look up owner/operation
receipt before task existence/version checks; existing receipts use their historical canonicalizer,
scheme and verify-only key, not newly deployed normalization/defaults. New operations use active
canonical format 1 and HMAC-SHA-256 with separate secret custody. Technical-design §5 fixes domain
separation, ordered UTF-8 JSON array, field/default normalization, constant-time compare and
key/version rotation. Digest key IDs/versions remain internal receipt metadata, not user input.

| Endpoint | Required body | Optional body / behavior |
|---|---|---|
| /api/tasks/create/ | operation_id, id (task UUID), title | description, priority, due_date; default null/medium/null; active and timestamps assigned by server |
| /api/tasks/{id}/edit/ | operation_id, expected_version, title | Full editable-details replacement: omitted description/due_date clear them; omitted priority defaults Medium; status unchanged |
| /api/tasks/{id}/status/ | operation_id, expected_version, status | status is active/completed; details unchanged |
| /api/tasks/{id}/delete/ | operation_id, expected_version, confirmed=true | Physically remove task; false/missing confirmation is rejected |

Optional details accept null or empty string as clear/default. Nonempty description preserves its content. Trim title with Unicode whitespace rules. Validate real calendar dates, not merely a date-shaped string. expected_version is a positive decimal string.

Every new successful mutation returns 200 after commit:

```json
{
  "operation_id": "cccccccc-cccc-4ccc-8ccc-cccccccccccc",
  "kind": "create",
  "task_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
  "committed_version": "1",
  "revision": "12",
  "replayed": false
}
```

Delete uses committed_version=null. Each new successful command increments account revision once. Accepted edit/status commands increment task version even when setting the same values. Create starts version 1. Replay returns the original acknowledgement metadata and replayed=true, without changing data/version/revision.

After acknowledgement, fetch a snapshot at least as new as the committed revision. A replay acknowledges history; do not render a historical task from the receipt. A task may already be changed/deleted by another operation.

## Transaction and failure semantics

Acquire account-state lock first, then check owner-scoped receipt, then task. Receipt/task/revision changes share one atomic transaction. The account row also serializes concurrent receipt creation, avoiding duplicate execution.

| Condition | Status / code | Result and recovery |
|---|---|---|
| Invalid field/command | 400 validation_error | No mutation/receipt; preserve draft and correct fields |
| Session expired/absent | 401 authentication_required | Hide account data; sign in |
| CSRF rejected | 403 csrf_failed | No task mutation; revalidate session/token |
| Unknown or foreign task ID | 404 task_not_found | Identical behavior for both; refresh and show removal/unavailability |
| Version mismatch | 409 version_conflict | No mutation; preserve draft; refresh and require review/current version |
| Same key, different canonical command | 409 operation_key_reused | No new mutation; client defect or reuse; do not silently generate a new key |
| Historical receipt key/format unavailable | 503 receipt_verification_unavailable | Known no-new-effect result; preserve pending ID, show retry/support; never silently verify with a different key or create a replacement operation |
| New create UUID already occupied | 409 task_id_unavailable | No mutation; generic response revealing no owner; no blind retry |
| Server/database unavailable | 503 service_unavailable when possible | Clear failure feedback; unknown outcome unless known rollback is explicitly established |
| Throttled | 429 rate_limited | Preserve form; Retry-After where available |
| Timeout/broken or malformed response | Transport/unknown | Retain exact pending operation in memory, lock all task writes; retry only after a valid saved-task snapshot |

Receipt lookup precedes existence/version checks for valid replay. A committed create replay after remote deletion succeeds historically and never recreates the task. Conflicting edits/deletes never silently overwrite current data. Validation or server failure before commit leaves task, revision and receipt unchanged.

TD-003 admission and DB timeouts are detailed in technical-design §4: ingress/application body
limit 2 MiB (including chunked input) → 413 without truncation; bounded upstream queue/pools →
503/Retry-After; task lock/statement/transaction budgets 500 ms/2 s/3 s and endpoint deadline 5 s.
No task-count cap or pagination/truncated success is added. A deadline after submission is
outcome-unknown unless rollback is proven; HTTP timeout cannot cancel an already committed effect.
Poll backoff preserves explicit Retry and FD-001. Keyed receipts retain metadata, not plaintext
commands, but no absolute deleted-content erasure claim is made. Compatibility requires old
canonicalizers/keys and preflight fixtures on upgrades/rollback (TD-007).

While a saved-task load or refresh failure remains unresolved, the client MUST NOT submit any task
mutation, including an exact replay of a pending operation. A valid saved-task snapshot lifts that
load-failure lock but does not settle the pending operation; only then may the client replay the
same operation ID and unchanged payload, subject to receipt, ownership, and version checks. If
recovery fails, remain locked and direct the user to support. No endpoint replaces the whole list
or resets saved data.
