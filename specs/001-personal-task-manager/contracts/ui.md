# UI Contract

Scope: responsive desktop/mobile web application. See [http.md](http.md) for requests and [../data-model.md](../data-model.md) for state/ordering rules.

## Screens and navigation

Registration: username, password, confirmation, inline validation and sign-in link. Login: username/password, registration link and support link for access problems. Registration needs no invitation/admin. Successful registration leads to login, then the task list.

Task screen: account identity and POST sign-out; New Task; status filter; priority filter; sort method/direction; load/sync feedback; task list; support link. Each row/card exposes title, priority and active/completed state. Show description and due date when present, either in the row or accessible detail view; do not invent placeholders as saved values. Long text wraps. Active past-due tasks have a text overdue indicator as well as styling.

Use a single-column card/form layout on narrow screens, with controls wrapping without horizontal page scrolling. Buttons have accessible names, inputs have visible labels, errors are associated with fields, keyboard focus is managed when forms/dialogs open and close, and status/error messages use appropriate live announcements. Do not communicate status only through color.

## Task forms and actions

New task: title required; description, priority and due date optional. Priority presents Medium as effective default and offers a clear/default state. Date input accepts a real calendar date; past dates remain valid. Save validates on both client interaction and server; server rules are authoritative. Display a saved state only after the server acknowledges commit.

Edit: prefill current details and retain its base version. Clearing description/date removes the value; clearing priority restores Medium. Cancel does not write. Failed validation preserves entered values with correction guidance.

Complete/reopen: explicit action sets desired state with the displayed version. Reopening changes status only. Open drafts survive background refresh, including when the underlying task changes; do not advance their base version silently.

Delete: open confirmation identifying the task and explaining permanent removal. Cancel makes no request; only Confirm sends confirmed=true. No undo/restore action. Stale confirmation cannot delete a newer version without renewed review and confirmation.

While one write is pending, disable additional task writes and duplicate submits. Preserve the exact operation ID/payload for recovery. After acknowledgement invalidate older in-flight refreshes and request the current snapshot; the resulting list may hide the task because of current filters or later remote changes. Make the acknowledged action clear without claiming the task must still exist.

## Filters, sorting and calendar

Status options: All, Active, Completed. Priority: All, Low, Medium, High. Combine with AND. Restore valid account-scoped status/priority preferences at page start, initially All/All. Persist user changes immediately. If preference storage fails, show that restoration is unavailable and retain current session selections.

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
| Unknown write outcome | Explain that save outcome is unconfirmed; preserve draft; Retry same operation; block other writes |
| Version conflict | Explain another change; retain draft; refresh current version and require review before a new operation |
| Edited task remotely removed | Explain removal; retain draft for copying; no automatic recreation |
| Session expiry | Hide/clear account data and drafts; sign-in navigation |
| Valid successful snapshot after failure | Replace stale view; unlock only when pending operation is also settled |

Retrying a read never uploads local tasks. Retrying an unknown operation uses the original request/key, not a newly generated create. A second failure does not authorize reset.

## Synchronisation and privacy

Poll every 15 seconds while visible, refresh on focus and after mutations, and expose Retry for manual recovery. Coalesce triggers. Reject stale revisions, wrong-account/session responses and outdated query generations. Each tab starts by validating its own session. A tab receiving a 401 or detecting an account mismatch must clear its task display immediately.

On logout clear rendered tasks, drafts and pending in-memory data; retain only namespaced filter preferences. Hide data before revalidating a page restored from browser history. Do not store task content, passwords or session tokens in localStorage. Browser reload does not falsely label an uncertain operation as failed or saved; a new authoritative load shows current state.

Support is reachable from load errors and auth screens. It provides a configured contact and nonsensitive reference, with no self-service task-data reset.
