# Proposed acceptance scenarios — conventional authentication amendment

**Date:** 2026-10-25. **Status:** proposed, planned and not executed. These do not replace the
approved R2-6R-T01–16 until the applicable review/human gates and authorised incorporation.
Scope and operative contracts remain unchanged; R3 NOT_READY, TD-008/010/011 OPEN.

Use [the proposed clauses](r2-conventional-auth-amendment-proposal.md), including RA-FD006-001
and its Scope review boundary. Use same-profile tabs, separate devices, distinct A/B accounts,
and same-account distinct epochs. Control actual header application separately from server
completion and JavaScript callbacks. Inspect selected credentials through authenticated outcomes,
server effects/receipts and UI state; do not log raw credentials or private contents.

| ID / old scenario trace | Proposed schedule and observable result |
|---|---|
| CA-T01 / old T01 | Two genuine anonymous login requests both validate successfully. Deliver A's cookie, then B's same-name/domain/path cookie; reverse server completion independently. Subsequent requests may select B according to the cookie actually held. No first-browser-winner or mandatory competitor-conflict assertion. Observe current account/epoch/view and protected-operation ownership; known changes clear old state. Repeat with the same account and distinct epochs. |
| CA-T02 / old T02 | Reverse header application, delivering B then A. Current-cookie identity governs, not submission order, server commit order or promise callback order. In controlled complete-cookie fixtures the later applied cookie replaces the same cookie; do not generalise to a promised global winner under arbitrary transport failure. No response causes access through the wrong authenticated owner. |
| CA-T03 / old T13 | With A authenticated, open root/new tab/login GET and submit stale-form POST with A or B credentials and A's current cookie. GET 302 /tasks/ and valid-CSRF POST 303 /tasks/ before replacement credential evaluation; same account/epoch/view. Invalid CSRF stays 403; lookup failure is technical, never anonymous. Restored forms revalidate and discard password fields if already signed in; no password replay. |
| CA-T04 / old T03/04/16 | Revoke intended L successfully, then deliver a distinct valid anonymous login response: it may authenticate; revoked L replay still fails. A third established C has no special first-winner protection against another already-submitted valid anonymous response; actual cookie ordering applies. Requests carrying authenticated C still obey CA-T03. Invalid/revoked credentials never authenticate, and no task/view state is imported across logins. |
| CA-T05 / old T09 | A's draft/create/edit/status/delete/view-save or protected read is dispatched with B's cookie and A's intended-login guard, also same-account new epoch. Reject before protected access/effect; no A text, task mutation, view update or mutation receipt is rebound to B. Direct B requests cannot read or modify A task IDs, including forged owner IDs. No automatic retry against B. |
| CA-T06 / old T07/08 | Observe an auth-change signal, trusted auth failure, response account/epoch mismatch or invalidation while A replies are delayed. Clear old rows/labels/drafts/dialogs/task-view queues and discard known stale replies, including newly arrived responses. Query/generation/revision checks still reject outdated same-login data. No additional global render fence is required. |
| CA-T07 / old T07/08, A9-05/06 | Previously authorised A content is displayed or its authorised response is in flight; another tab changes authentication while notification/observation is delayed. If no mismatch is yet known, temporary old content/in-flight completion within RA-FD006-001 is not alone a failure of this proposed contract. Do not assert zero exposure or a numeric duration. On the next observed change/required boundary, clear/revalidate; old-intent mutations must still fail under CA-T05. This is a display residual, never a wrong-account API grant. |
| CA-T08 / old T06/09 | Initial entry, focus/visibility/resume/history restoration starts concealed and revalidates. Actual revoked/expired auth stays private-free; uncertainty stays concealed with Retry/support; matching current account loads normally. Cached details/dialogs/filter choices under an otherwise known login need no independent auth call on every interaction; no per-keystroke check or mandatory post-arrival auth GET. Local stale checks and server guards still apply. |
| CA-T09 / old T10 | Fail ordinary task refresh after prior same-login success: retain visibly stale rows/error, Retry/support and lock every mutation/retry until valid saved-task load. Separately fail a required authority check: conceal. Separately fail optional view read/save: full silent fallback or usable current selection, no task blocking solely for view failure. Auth success alone cannot settle an unknown operation or remove FD-001 locks. |
| CA-T10 / old T05/15 | Logout current intended L; success requires confirmed shared revocation and subsequent L replay fails. Lose/malform ACK or return 401/403/identity change: remain unconfirmed, not a success inferred from cleared UI/cookie loss. If cookie selection changes to N before old intent is authorised, do not revoke N or claim L was revoked. Recover using existing intended-login Retry/support rules. |
| CA-T11 / old T05 | After confirmed logout, promptly dispatch the existing notification. Receiving runnable same-login tabs clear/conceal their private state and navigate to login without waiting for a poll/action. No wait for tab ACKs and no guarantee every message is delivered. An unconfirmed logout must not be broadcast as confirmed. |
| CA-T12 / old T06 | Drop/delay/duplicate notifications or make messaging unavailable; suspend a tab and restore from history/bfcache. Required lifecycle/uncertainty revalidation and clearing on observed change remain. Missing delivery alone does not demand a guaranteed transport, new browser capability or atomic clearing. Existing synchronisation can discover change; it is not evidence that a failed message was delivered. Do not promise a maximum observation delay. |
| CA-T13 / old T12/13 | Deliver revoked-L notification or delayed CSRF/logout recovery after N is valid. Do not revoke N, reset N's view or replay L's logout as N. Fresh login-URL navigation with N follows R2-NAV-001. Preserve bounded same-intended-epoch CSRF recovery, trusted vs generic failure distinction and no password/task replay. |
| CA-T14 / old T11/14 | Keep another browser/device login with its own nondefault view; logout this profile and create/select fresh same-account login here. Other session remains valid with its own tuple; saved tasks still synchronise. Fresh login defaults, same valid login/navigation retains its tuple; no account-level preference inheritance or new expiry policy. |
| CA-T15 / old T15, R2-T05 | Interleave command commit before logout and revocation before command effect. Persist valid committed effects/receipt/revision; no command under revoked authority commits after logout success. A lost task reply plus failed snapshot blocks all task writes and exact retry; recovery uses original operation ID/payload, versions and receipts. No reset or manufactured empty success. |
| CA-T16 / old T14 | Regress AM001-V01–10, unchanged NAV intent, R2-T05/T07/T12, task lifecycle/order/date rules, owner isolation, native fixation/CSRF/cookie/no-store protections and supported browser availability. Preserve FD-003 interruption and FD-004 no-recovery disclosures. No mandatory Web Locks, IndexedDB, custom winner election or atomic render proof is required by this amendment; no architecture is selected. |

## Changed acceptance obligations and test phase

Old T01/02/04/16 winner-protection expectations are superseded only in this proposal; valid
anonymous competitors need not conflict or preserve the first view. Old T07/08 and Q1 universal
fresh round trips/atomic reveal checks become known-stale protection and boundary revalidation,
with the explicit display residual. Old T05/06/12 notification tests retain prompt dispatch and
receiving-tab response but do not require guaranteed delivery. All unaffected tests remain binding.

Independent R2 review verifies written outcomes/traceability. Following the appropriate approval,
R3 specifies feasible guards/recovery and a proportionate technical validation plan. Build/Test
executes authorised browser/HTTP/database scenarios later; no execution is required to approve
this written proposal absent a specific justified feasibility question. Codex prepares the plan;
existing authorised reviewers/human owners retain their gates. No scenario here closes a finding.
