# HTTP Contract

Internal same-origin interface for the web application. All paths are relative to the application origin. Session cookies authenticate requests; no caller-supplied account ID is accepted. See [../data-model.md](../data-model.md) and [ui.md](ui.md).

## Pages and authentication

| Method/path | Behavior |
|---|---|
| GET /accounts/register/ | Username/password/confirmation form with CSRF token |
| POST /accounts/register/ | Validate and create private account plus list state atomically; 302 to sign-in on success; 200 form with field errors on failure |
| GET /accounts/login/ | Sign-in form; link to registration and support |
| POST /accounts/login/ | Standard Django auth form; 302 to /tasks/ on success; 200 generic credential error on failure |
| POST /accounts/logout/ | CSRF-protected session logout; 302 to login; GET must not log out |
| GET /tasks/ | Authenticated responsive task shell with public account UUID and CSRF token; anonymous requests redirect to login |
| GET /support/ | Support instructions and configured contact; accepts optional nonsensitive request reference; no data reset or automatic password reset |

Use Django's validated same-origin next handling, not arbitrary redirects. Rejected registration identifies invalid/duplicate username or invalid password/confirmation without creating a partial account. Login failures do not reveal account existence. Production proxy throttling applies to registration and login.

## Common JSON rules

Prefix: /api/. Send/receive application/json, UTF-8. Unsafe methods require valid same-origin CSRF token (X-CSRFToken). API authentication failure is 401 JSON, not an HTML sign-in redirect. GET has no mutation side effects. Authenticated pages/API responses use Cache-Control: no-store.

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

GET /api/tasks/?status=all&priority=all&sort=due_date&direction=asc

Defaults as shown. status: all/active/completed; priority: all/low/medium/high; sort: due_date/priority/created_at/title; direction: asc/desc. All filtering and ordering is executed by domain services using the deterministic rules in the data model.

200 response:

```json
{
  "account_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
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

Clients validate response identity, query, full schema and revision before accepting it. Discard wrong-session, wrong-query or older-generation responses, revisions below the latest accepted revision, and revisions below the latest acknowledged mutation. Different query results may legitimately have equal revisions.

## Mutations

All mutations are POST. One logical operation uses one operation_id UUID, preserved with the exact submitted request for retries. Create also retains one task UUID. Commands are validated/canonicalized before receipt digest comparison; default-equivalent omitted/null/empty optional values canonicalize identically. Include kind/route, task ID and expected version in the digest. Ownership is always derived from the authenticated session.

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
| New create UUID already occupied | 409 task_id_unavailable | No mutation; generic response revealing no owner; no blind retry |
| Server/database unavailable | 503 service_unavailable when possible | Clear failure feedback; unknown outcome unless known rollback is explicitly established |
| Throttled | 429 rate_limited | Preserve form; Retry-After where available |
| Timeout/broken or malformed response | Transport/unknown | Retain exact pending operation, lock other writes, recover using same operation |

Receipt lookup precedes existence/version checks for valid replay. A committed create replay after remote deletion succeeds historically and never recreates the task. Conflicting edits/deletes never silently overwrite current data. Validation or server failure before commit leaves task, revision and receipt unchanged.

A retry may replay the same pending mutation while ordinary mutations are locked. This is the sole recovery exception to the UI lock. Refresh success alone cannot prove whether a pending operation committed; receipt-backed replay settles it. If recovery still fails, remain locked and direct the user to support. No endpoint replaces the whole list or resets saved data.
