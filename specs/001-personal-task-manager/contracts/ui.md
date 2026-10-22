# UI Contract

Scope: responsive desktop/mobile web application. See [http.md](http.md) for requests and [../data-model.md](../data-model.md) for state/ordering rules.

## Screens and navigation

Registration: username, password, confirmation, inline validation, sign-in link, and a conspicuous
notice: "If you forget these credentials, Taskify cannot reset them or restore account access."
Login: username/password, registration link, and a forgotten-credentials/support explanation that
states no password reset, recovery, or account transfer is available. Registration needs no
invitation/admin. Successful registration leads to login, then the task list.

Task screen: account identity and POST sign-out; New Task; status filter; priority filter; sort method/direction; load/sync feedback; task list; support link. Each row/card exposes title, priority and active/completed state. Show description and due date when present, either in the row or accessible detail view; do not invent placeholders as saved values. Long text wraps. Active past-due tasks have a text overdue indicator as well as styling.

Use a single-column card/form layout on narrow screens, with controls wrapping without horizontal page scrolling. Buttons have accessible names, inputs have visible labels, errors are associated with fields, keyboard focus is managed when forms/dialogs open and close, and status/error messages use appropriate live announcements. Do not communicate status only through color.

## Task forms and actions

New task: title required; description, priority and due date optional. Priority presents Medium as effective default and offers a clear/default state. Date input accepts a real calendar date; past dates remain valid. Save validates on both client interaction and server; server rules are authoritative. Display a saved state only after the server acknowledges commit.

Edit: prefill current details and retain its base version. Clearing description/date removes the value; clearing priority restores Medium. Cancel does not write. Failed validation preserves entered values with correction guidance.

Complete/reopen: explicit action sets desired state with the displayed version. Reopening changes status only. Open drafts survive background refresh, including when the underlying task changes; do not advance their base version silently.

Delete: open confirmation identifying the task and explaining permanent removal. Cancel makes no request; only Confirm sends confirmed=true. No undo/restore action. Stale confirmation cannot delete a newer version without renewed review and confirmation.

While one write is pending, disable additional task writes and duplicate submits. Preserve the exact
operation ID/payload in memory for recovery. After acknowledgement invalidate older in-flight
refreshes and request the current snapshot; the resulting list may hide the task because of current
filters or later remote changes. Make the acknowledged action clear without claiming the task must
still exist.

## Filters, sorting and calendar

Status options: All, Active, Completed. Priority: All, Low, Medium, High. Combine with AND.
Retrieve the authenticated account's saved status/priority pair before the initial task snapshot;
the first-use saved pair is All/All. Apply each user selection immediately to the current page and
save the pair to the account. Send one preference save at a time; when controls change rapidly,
retain only the newest unsent pair and do not display an older acknowledgement as confirmation of
the current selection. Confirm "saved across your devices" only after the server acknowledges the
current pair or an authoritative re-read returns that pair.

On a preference transport/5xx/malformed-response outcome, say "Could not confirm your filters
were saved"—never that they were not saved—and retain the visible selection. On a preference
version conflict, say the submitted selection conflicted with a later saved preference, without
attributing its origin to another device. In either case, pause further saves, re-read the
authoritative pair/version, and compare it with the current visible pair. A match becomes saved; a
difference remains visible but unconfirmed and requires explicit user approval to overwrite using
the re-read version. Ignore stale/out-of-order save or re-read responses so they cannot replace a
newer visible selection or lower the confirmed version. Do not use browser localStorage for
preferences.

Sort exactly by Due Date, Priority, Date Created or Title, with both directions. Reset to Due Date earliest-first on a fresh page session. Label direction meaningfully for each method. Undated tasks stay last in either due-date direction. Fixed ties and locale-neutral title ordering follow the data model. No extra sort/search/tag features.

Changing filters/sort requests a fresh ordered snapshot; retain old rows marked as refreshing until the matching response arrives, then apply atomically. Do not label old rows as matching the new query. Coalesce rapid selections and discard outdated query responses.

Overdue uses the browser's local calendar, including midnight/focus recalculation. Due dates are displayed without timezone conversion. A completed task loses its overdue label; reopening restores it when appropriate.

## Empty, loading and failure states

| Situation | Required display/action |
|---|---|
| Initial request pending | Loading state; task writes disabled |
| Successful empty owned list | No tasks yet, with New Task action |
| Tasks exist but filters match none | No tasks match these filters; controls remain available |
| Initial load failure | Task list unavailable; Retry/support; no write controls enabled |
| Refresh failure after prior success | Keep rows visibly stale; show failure; lock writes; Retry/support |
| Second consecutive load failure | Explicitly recommend contacting support with request reference |
| Definite write rejection | Field/action error; preserve draft and last saved display |
| Unknown write outcome with valid saved-task view | Explain that save outcome is unconfirmed; preserve draft; permit Retry same operation; block other task writes |
| Unknown write outcome with saved-task loading failure | Explain that no task action, including retry, can be submitted until saved tasks load; offer read Retry/support and preserve the pending operation in memory |
| Version conflict | Explain another change; retain draft; refresh current version and require review before a new operation |
| Edited task remotely removed | Explain removal; retain draft for copying; no automatic recreation |
| Session expiry | Hide/clear account data and drafts; sign-in navigation |
| Valid successful snapshot after failure | Replace stale view; if an operation remains unknown, enable only its exact retry; otherwise unlock task writes |

Retrying a read never uploads local tasks. Retrying an unknown operation uses the original
request/key, not a newly generated create, and is unavailable until a valid saved-task snapshot has
loaded. A second failure does not authorize reset.

## Synchronisation and privacy

Poll every 15 seconds while visible, refresh on focus and after mutations, and expose Retry for manual recovery. Coalesce triggers. Reject stale revisions, wrong-account/session responses and outdated query generations. Each tab starts by validating its own session. A tab receiving a 401 or detecting an account mismatch must clear its task display immediately.

On logout clear rendered tasks, drafts and pending in-memory data. Hide data before revalidating a
page restored from browser history. Do not store task content, passwords, session tokens, pending
operations, or preferences in localStorage. Pending operation identity/payload is memory-only: a
browser reload, close, crash, or navigation can discard it. Warn the user to wait for a confirmed
result; after interruption, direct them to sign in/refresh and review the current list before
re-entering the action. An early snapshot is not proof of rollback: the first request can later
commit, so a repeated action can create a duplicate or a delayed unexpected change.

Support is reachable from load errors and auth screens. It provides a configured contact and
nonsensitive reference, with no self-service task-data reset, password reset/recovery, or account
transfer.
