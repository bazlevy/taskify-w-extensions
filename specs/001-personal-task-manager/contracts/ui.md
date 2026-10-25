# UI Contract

**Approved Functional Design:** Scope v1.2 / AM-002 and R2-Rev8, approved by Barry Levy on
2026-10-25; [R2-Rev8 approval §4](../../../independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md). FD-009 is RESOLVED_VERIFIED per
that record §2. Historical R2-Rev5/R2-NAV-001 and R2-Rev6 approvals remain attached to their
baselines; only targeted clauses are superseded. R3 NOT_READY; TD-008/010/011 OPEN.
P1/IndexedDB paused. No technical architecture or runtime evidence is approved here.

Scope: responsive desktop/mobile web application. See [HTTP](http.md) and
[data model](../data-model.md). Scope v1.2 AC-010 and RA-FD006-001 define the accepted display
residual. FD-003/004 and silent view rules remain within their original bounds.

## Screens and navigation

Authenticated users opening Taskify in another tab within the same browser profile go directly
to their task list using the shared login. This is navigation, not a new login: retain account,
epoch and the complete valid remembered view. Accessing /accounts/login/ while authenticated
redirects to /tasks/ without showing a sign-in form. Normal deliberate account switching requires
logout first; confirmed logout invalidates its intended shared login. Genuine anonymous requests
already in flight follow the separate conventional response-order rule below.

A stale sign-in page restored from history or revisited revalidates before enabling submission.
If already authenticated, discard credential fields and replace navigation with /tasks/.
Requests carrying an authenticated login also cannot evaluate replacement credentials or reset its view. Lookup uncertainty
shows technical Retry/support rather than treating the user as signed out. Task-screen private
content still follows the concealed authority/snapshot gate before reveal.
The fetch-based login controller treats an authenticated 303 to /tasks/ as navigation, discards
credential/candidate queues and loads that task shell; it never retries credentials because the
response was redirected HTML instead of a fresh-login JSON result.

### Concurrent anonymous logins — FD-007 / CA-007

Genuine requests submitted while anonymous may both create valid distinct logins. Ordinary
browser session-cookie application/selection governs subsequent requests, including same-account
distinct epochs. A later response may select a different valid login without intervening logout.
There is no first-submission, server-commit or first-browser-winner guarantee, and no required
competitor conflict solely to protect that winner. This exception does not permit replacement
credential evaluation on an authenticated request; R2-NAV-001 remains unchanged.

Each actual fresh login starts its own default view; valid-login navigation retains its tuple.
On observed selection change, clear prior-login client state and load only the selected login's
account/view; never transfer drafts, queued operations or settings. Browser cookie order is not
account-wide preference precedence. The task screen identifies the current account.

S2 remains: a distinct valid late login may authenticate after another login's confirmed
revocation. Revoked/invalid credentials cannot authenticate; no promise cancels/revokes every
other valid session created by anonymous overlap. No automatic credential replay.

Registration: username, password, confirmation, inline validation, sign-in link, and a conspicuous
notice: "If you forget these credentials, Taskify cannot reset them or restore account access."
Login: username/password, registration link, and a forgotten-credentials/support explanation that
states no password reset, recovery, or account transfer is available. Registration needs no
invitation/admin. Successful registration leads to login, then the task list.

Task screen: account identity and POST sign-out; New Task; status filter; priority filter; sort method/direction; load/sync feedback; task list; support link. Each row/card exposes title, priority and active/completed state. Show description and due date when present, either in the row or accessible detail view; do not invent placeholders as saved values. Long text wraps. Active past-due tasks have a text overdue indicator as well as styling.

Use a single-column card/form layout on narrow screens, with controls wrapping without horizontal page scrolling. Buttons have accessible names, inputs have visible labels, errors are associated with fields, keyboard focus is managed when forms/dialogs open and close, and status/error messages use appropriate live announcements. Do not communicate status only through color.

## Task forms and actions

**Q1 aligned by CA-006:** Every protected server operation authenticates the presented login,
enforces ownership and intended-login guards before access/effects. Preserve CSRF, task versions,
receipts and exact retry rules. Never rebind an A intent to B, including same-account new epoch.

Local cached details/edit/delete dialogs, filter choices, normal responses/refreshes and typing
do not require an independent auth round trip for every interaction. Check known identity/epoch/
query/generation/revision before applying responses. Initial protected entry, restored stale forms,
focus/visibility/resume/history and auth uncertainty trigger appropriate concealed revalidation;
observed changes clear old private state. Local Cancel/public support grant no protected access.
Isolated view failure must not block otherwise available tasks.

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
Bootstrap with the task snapshot directly; its effective query supplies all four controls. Restore
the complete valid view remembered in this still-valid login. If any component is missing, invalid,
incomplete or unreadable, use all four defaults silently: All/All/Due Date earliest first. Never
require a separate preference request or hold available tasks for a view failure.

Sort exactly by Due Date, Priority, Date Created or Title, with both directions. Restore method and
direction with filters on same-login refresh/new tabs; reset all four only on new login or full
view fallback. Label direction meaningfully for each method. Undated tasks stay last in either
due-date direction. Fixed ties and locale-neutral title ordering follow the data model. No extra
sort/search/tag features.

Apply every control choice immediately to the current tab and request matching task data while
remembering its complete tuple independently. Allow one view save per tab; retain only the latest
queued choice. Failed or unconfirmed remembering leaves the chosen controls/view usable without a
save/restoration warning, success message or special Retry flow. Do not claim either failure or
success as certain after a lost response. Later refresh/new tabs use what actually committed, else
complete defaults; no guaranteed restoration of an unconfirmed selection. A delayed save reply
cannot change a newer visible choice or falsely acknowledge it. New explicit choices may attempt
remembering; no automatic retry of an uncertain old choice.

Tabs sharing this login start from its last committed view when opened/refreshed. Already-open
tabs keep their current selections, with no required live update. For concurrent explicit changes,
last server commit wins, not browser time; no account-version conflict/reconciliation prompt is
shown. Another still-valid browser/profile/device session keeps its own view. Logout, expiry or a
new login starts defaults rather than importing any other session's settings. Closing/opening a
tab or restarting a browser follows whether the login remains valid, with no new remember-me rule.

Changing filters/sort requests a fresh ordered snapshot; retain old rows marked as refreshing
until a matching response passes checks against known account/epoch/query/generation/revision.
Do not label old rows as matching a new query. Concealment/clearing takes precedence on required
lifecycle/uncertainty boundaries or observed invalidation. No separate post-arrival auth request
or browser-wide atomic rendering is required; the bounded AC-010 display residual applies.
Coalesce rapid selections and discard known outdated query/session-generation replies.

Overdue uses the browser's local calendar, including midnight/focus recalculation. Due dates are displayed without timezone conversion. A completed task loses its overdue label; reopening restores it when appropriate.

## Empty, loading and failure states

| Situation | Required display/action |
|---|---|
| Initial request pending | Loading state; task writes disabled |
| Authority uncertain at initial entry, focus/resume/history or recovery boundary | Conceal private content/actions and revalidate; Retry/support; never infer anonymity or empty success |
| Private response arrives | Reject known identity/query/generation/revision mismatch; no mandatory independent auth probe per response; AC-010 residual applies to unobserved changes |
| Valid overlapping anonymous login responses | Actual valid session cookie governs; no first-winner/conflict guarantee; observed change clears old client state |
| Logout pending or outcome unknown | Clear/conceal local private state; retain intended epoch; no success claim without confirmed revocation |
| Logout confirmed | Promptly dispatch notification; receiving runnable same-login tabs clear/navigate to login; no guaranteed delivery; revoked login remains invalid |
| Cross-tab auth change / account or login_epoch mismatch | Clear tasks, drafts, pending payloads and view queues; discard old replies; sign-in/clean revalidation |
| View absent/invalid/unreadable, task load succeeds | Complete defaults, no view-state notice; normal task controls available |
| View remembering fails or is unconfirmed | Current selection stays usable without warning; task actions depend only on auth/task load and task-operation state |
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

Poll every 15 seconds while visible, refresh on focus/after mutations and expose Retry. Coalesce
triggers; failed background reads back off at 15/30/60 s with jitter; manual Retry stays available.
Reject stale revisions, wrong account/epoch/session and outdated query generations.

Initial/history-restored shells conceal private DOM, account labels, drafts, dialogs and
accessibility text. Blur/hidden/pagehide conceals and invalidates local generations; focus,
visibility/resume/history return revalidates before normal private access. Observed auth changes,
trusted auth failure or identity mismatch clear old rows/labels/drafts/dialogs/task-view queues
and discard known invalidated responses. Required authority uncertainty stays concealed with
Retry/support. Never treat view defaults as authentication.

**FD-006 / CA-006:** Authenticate and owner/intended-login guard every protected server operation.
Before applying replies, reject known account/epoch/query/generation/revision mismatches. No
browser-wide atomic validation/render guarantee, continuous cookie observation, per-local-action
or mandatory per-response auth round trip. Successful ordinary protected requests provide server
auth evidence; required lifecycle and uncertainty checks remain. No old requests/drafts/mutations/
view saves/retries may be rebound to a newer login, including same-account different epoch.

**RA-FD006-001 / approved AC-010:** Prior-login-authorised content may remain visible or finish
rendering while a same-profile cross-tab auth change is unobserved. Barry approved this bounded
residual through Scope v1.2 and R2-Rev8. No maximum observation delay or zero-exposure promise.
Shared-device confidentiality consequences can be serious; likelihood is unmeasured. This is
not permission for cross-account API reads/mutations, old-intent reuse, known-mismatch rendering
or continued private-state retention after observed change.

Ordinary same-login task-refresh failure retains visibly stale rows/error and every FD-001
mutation/retry lock until valid saved-task load. Do not relabel every task network failure as
authority uncertainty. Auth success alone does not settle an unknown task operation. Isolated
view remembering/restoration errors stay silent and cannot block otherwise available tasks.

**FD-008 / CA-008:** Capture intended L and clear/conceal initiating-tab private state on logout
intent. Show success only after confirmed shared L/view revocation; revoked credential replay
fails. Promptly dispatch the existing notification mechanism. Receiving runnable same-profile
L tabs clear private state and navigate to login without waiting for a poll/action. No guaranteed
delivery, tab-ACK wait, atomic clearing or simultaneous suspended rendering is required.
Missed/late/duplicate/unavailable delivery follows lifecycle/uncertainty revalidation and existing
synchronisation; do not add a guaranteed transport or faster polling requirement.

Other browser/device sessions remain unaffected. Delayed L notices or old logout intent cannot
revoke N, reset N's view or restore L. Login navigation while N is authenticated obeys R2-NAV-001.
If selection changes before the intended logout can be authorised, reject/revalidate, never revoke
the new login or claim L was revoked. A distinct valid anonymous response may select another login
under ordinary ordering, including after revocation; no promise locates all superseded sessions.

A lost/malformed logout reply, 401/403, missing cookie or changed identity alone is not a confirmed
ACK. Preserve unconfirmed-outcome Retry/support, bounded same-intended-epoch CSRF recovery and
no password/task automatic replay.

The prior first-winner/atomic-render/mandatory-interaction gates are superseded. Historical
family/claim/rotation proposals select no current architecture; R3 must reassess retained guards
and lifecycle/recovery against the conventional contract. P1 remains paused.

On logout clear rendered tasks, drafts and pending in-memory data. Hide data before revalidating a
page restored from browser history. Do not store task content, passwords, session tokens, pending
operations, or preferences in localStorage. Pending operation identity/payload is memory-only: a
browser reload, close, crash, or navigation can discard it. Warn the user to wait for a confirmed
result; after interruption, direct them to sign in/refresh and review the current list before
re-entering the action. An early snapshot is not proof of rollback: the first request can later
commit, so a repeated action can create a duplicate or a delayed unexpected change.

Fresh same-user login after logout/expiry starts defaults. Authenticated login-page navigation/
submissions preserve current account/epoch/view: R2-NAV-001 is CLOSED — approved in R2-Rev5 §7.
The inherited unapproved maintenance candidate keeps browser identity/epoch/view/original expiry;
it is not a fresh login. R3 must prove continuity; this functional reconciliation adopts no mechanism.

### Cross-tab CSRF recovery

At a required stale-form/restoration or CSRF recovery boundary, obtain GET /api/csrf/'s fresh
no-store masked token before the unsafe request rather than relying on a stale cached token. Login in another tab can rotate the
secret after that read, so handle a trusted csrf_failed 403 as no service dispatch, never success.
Keep private content concealed during mandatory revalidation.

When auth housekeeping is required, refresh token/revalidate authority and retry it at most once.
For logout, capture intended epoch on click and conceal/clear private state; retry once only
after fresh token plus read-only authority still identifies that same epoch. If account/epoch
changed, never log out the new login with the old intent. Another 403 or lookup failure leaves
technical Retry/support/full safe reload. Success requires actual shared-revocation ACK; a 403,
401, vanished cookie or changed identity alone is not logout completion.

A stale login form discovering authenticated A navigates to tasks; do not replay credentials.
Task commands preserve original pending ID/payload and require explicit retry after valid saved
tasks, with FD-001 intact. View-only remembering failure stays silent with no old-tuple replay;
token refresh for a later explicit choice does not lock otherwise available task actions.
Generic HTML/proxy 403 or lost response is not trusted rollback evidence. Never treat CSRF
refresh as authority to expose data or bypass the saved-task load lock.

Registration/login admission may return generic temporary 429 with Retry-After or 503 when the
shared limiter cannot enforce admission. Preserve nonsecret fields, never echo passwords, and use
existing retry guidance. Invalid credentials remain generic and no permanent account lock or
credential recovery is offered. Existing valid task sessions are not invalidated by limiter outage.

Support is reachable from load errors and auth screens. It provides a configured contact and
nonsensitive reference, with no self-service task-data reset, password reset/recovery, or account
transfer.
