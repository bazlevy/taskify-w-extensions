# Quickstart Validation Guide

**Approved functional baseline:** Scope v1.2 / AM-002 and R2-Rev8, Barry Levy 2026-10-25.
[R2-Rev8 approval §4](../../independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md); FD-009 RESOLVED_VERIFIED per §2. CA-T01–16 below are the
current approved written acceptance scenarios, planned and not executed. Historical approvals
and R2-Rev7 NOT_READY are preserved. R3 NOT_READY; TD-008/010/011 OPEN; P1 paused.

This is a future validation guide, not evidence that code/dependency locks/Compose/tests exist.
No commands below are executed during incorporation; no Tasks/Analyze/Build authority.
Contracts: [HTTP](contracts/http.md), [UI](contracts/ui.md), [data model](data-model.md).
[Current R3 handoff](review-followups/r2-rev8-r3-handoff.md).

## Prerequisites and local setup

Python 3.13, Docker with Compose, supported desktop/mobile browsers, and network access for initial dependencies/browser engines. Use PostgreSQL 17 locally and in integration tests; SQLite is not an acceptable substitute for locking tests. Use disposable accounts/data. The local database role must be able to create an isolated test database; production credentials must not have that privilege.

From repository root, after implementation has supplied these files:

```sh
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.lock
cp .env.example .env
```

The example environment must document shell-compatible local-only values for DJANGO_SECRET_KEY, DJANGO_DEBUG, DJANGO_ALLOWED_HOSTS, PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD and SUPPORT_CONTACT. Review .env locally; never commit it. Load it without printing secrets:

```sh
set -a
source .env
set +a
docker compose up -d db
docker compose exec db pg_isready
python manage.py migrate
python manage.py check
python -m playwright install chromium firefox webkit
python manage.py runserver 127.0.0.1:8000
```

Expected: migration/check success, database ready, registration page at http://127.0.0.1:8000/accounts/register/. Localhost is sufficient for browser secure-context UUID generation. For actual mobile access, use a development HTTPS origin and appropriate allowed hosts/cookie configuration; do not use production accounts.

## Automated validation

Planned test locations and invocation:

```sh
python -m pytest tests/unit
python -m pytest tests/integration tests/contract
python -m pytest tests/e2e --browser chromium --browser firefox --browser webkit
python manage.py makemigrations --check --dry-run
```

Configure pytest-django to use isolated PostgreSQL test databases and the browser suite to start a live Django test server. Integration/concurrency tests require separate connections and transaction-enabled tests; transaction-wrapped unit fixtures alone cannot prove lock behavior. Browser cases include desktop and mobile viewport/context variants.

Before production release, provide production settings and run:

```sh
python manage.py check --deploy
```

Expected: relevant automated suites pass, no missing migrations, and production security findings are resolved. Document browser/engine versions, real-device results and observed timings; no unapproved numerical success threshold is introduced.

## End-to-end scenarios

Every protected server operation authenticates/owner-checks/intended-login guards. Known stale
identity/query/generation/revision responses are discarded; observed auth change clears old state.
Revalidate appropriate lifecycle/uncertainty boundaries; no independent auth trip per cached
action/normal reply or browser-wide atomic render fence. Approved AC-010/RA-FD006-001 permits
prior-authorised display/in-flight completion while a change is unobserved, not API/rebinding or
known-stale exceptions. FD-001 and silent view failures stay. Native response ordering selects
genuine overlapping anonymous logins; authenticated redirects remain R2-NAV-001.
CA-T01–16 supersede conflicting old functional expectations; historical technical fixtures do not
select an architecture or reinstate superseded guarantees.

1. **Private registration, returning access, and credentials** — FR-010/015/016; AC-009/010/012; US4. Register account A with username/password and no invitation. Invalid username, duplicate/case-only duplicate, password mismatch and weak password show corrections. Before registration, confirm the visible warning that forgotten credentials cannot be recovered. From login and support, confirm that no automated/manual password reset, recovery, or account transfer is offered or promised. Sign in on desktop and a separate mobile browser/context. Register B separately. Direct requests from B must not read/edit/complete/reopen/delete A's tasks; unauthenticated API requests return 401. Logout clears displayed account data. Session restoration/history navigation does not reveal cached tasks after revalidation fails. Test POST CSRF rejection and login/logout behavior. **R2-T12 / FD-004**: verify all credential-loss disclosures and absence of recovery routes.

2. **Create and validate** — FR-001/002/003; AC-002; US1. Create using only title; expect active, Medium, absent description/date. Create with all optional details and valid past/leap-day dates; retain supplied values. Reject whitespace-only titles, impossible dates and date-time input. Title/description markup displays as text. Refresh and sign in again; committed values persist.

3. **Maintain and delete** — FR-004/005/006; AC-001/003; US2. Edit all details, then clear description/date/priority; expect null/null/Medium. Complete, then reopen; original details remain. Cancel deletion; task stays. Confirm deletion; task disappears after refresh and on the other device; no restore UI. Verify deletion removes the task row and receipts do not retain task contents.

4. **Calendar semantics** — FR-011; AC-004. With a controlled browser clock/timezone, verify yesterday is overdue only when active; today/tomorrow/undated are not. Complete/reopen and cross local midnight. Test a timezone differing from UTC and a year boundary; saved date remains identical. Verify two devices use their own local calendar and do not shift the date itself.

5. **Filter, sort and authenticated-session view** — FR-007/012/013; AC-005/006/007/011; US3/4. Fixtures span both statuses, all priorities, dates 2026-10-19/21/30, undated tasks, tied dates/priorities/creation timestamps and titles differing in case/Unicode normalization. Check each filter/combined AND and four sorts in both directions; preserve undated-last and domain ties. Successful no-match result has its own empty state. The AM-001 cases below replace account-preference restoration/reconciliation scenarios for this draft, without rewriting historical R2-T06 / FD-002/005 verification.

| Case | Planned challenge / observable result |
|---|---|
| AM001-V01 | New valid login: bootstrap GET /api/tasks/ directly with no preference request; All/All/due_date/asc. Choose Completed/High/title/desc, obtain view-save acknowledgement, refresh and open a same-login tab: restore all four |
| AM001-V02 | Keep a tab on its old view while another explicitly saves a new tuple: no required live update; new/refreshed tab restores latest server-committed complete view |
| AM001-V03 | Two valid sessions for same account on distinct browser/profile/device: new login defaults; each later remembers only its own view while task changes still synchronise; revisiting an already-valid session retains its own view |
| AM001-V04 | Logout/login as same or different account, expire/login, repeat same-account login after logout/expiry: new defaults; authenticated login navigation keeps current view, no prior-view import. Tab close/reopen and browser restart follow actual login validity, without inventing authentication expiry |
| AM001-V05 | Remove/corrupt each component or schema independently, simulate isolated view-read outage: all four defaults silently; authenticated task snapshot and actions remain usable; no partial tuple, notice or repair write from GET |
| AM001-V06 | View save definitely fails or commits with lost/malformed reply: chosen controls/view remain usable silently, no false success/failure or restoration claim; later refresh restores actual valid committed tuple or complete defaults |
| AM001-V07 | Rapid view changes during a save: one active save per tab, latest queued explicit tuple only; late reply cannot replace newer controls or confirm a tuple it did not carry; no automatic replay of an uncertain old tuple |
| AM001-V08 | Concurrent explicit saves in same-login tabs: full-tuple atomicity and last server commit; no account-version 409 dialog or cross-device view sync. Delay an unrelated middleware session save: it must not revert the SessionViewState tuple |
| AM001-V09 | View save races logout/rotation/expiry/new login: session-first guard and ownership revalidation prevent old-row resurrection, account/session carryover and mutation of new login settings. Verify old request/session-generation responses are ignored |
| AM001-V10 | Authentication or task load fails while view restoration also fails: auth error prevents private access; task error follows FR-014/FD-001 Retry/support and locked task writes, including uncertain replay; never a successful empty task list |

R3 must verify the proposed session-row lifecycle/lock mechanism before these can be treated as
implementable release tests. TD-001 cross-tab concealment needs its separate scenario and correction;
AM001-V09 does not close that finding. No planned scenario is recorded as executed here.

6. **Saved changes across devices** — FR-008; AC-009; SC-004. With A signed in on two isolated browser contexts, create/edit/complete/reopen/delete from one; focus/refresh or visible polling on the other must converge after successful synchronisation. Close/reopen and sign in again. No full-list replacement request occurs. Record that the polling interval is a design setting, not an availability SLA.

7. **Load/refresh failure protection** — FR-009/014; AC-008. Inject failed initial snapshot and malformed JSON: show unavailable, Retry/support, block task writes, never claim an empty saved list. After one successful load inject a failed refresh: retain visibly stale rows and lock writes. Repeat failure: explicitly recommend support. Restore the endpoint and retry: valid authoritative data replaces stale data; writes unlock if no pending unknown operation. If an operation is unknown, keep all task writes—including its retry—disabled until the valid snapshot succeeds; only then enable its exact same-ID/payload retry. No saved-data reset appears.

8. **Failed, uncertain, and interrupted writes** — FR-008/009/014; SC-004. Return validation errors and known rollback errors; retain drafts and saved values. Drop a response after commit; show unknown outcome. After a successful saved-task snapshot, retry identical operation/key: one effect and historical acknowledgement. Repeat with create → dropped response → deletion on device two → delayed create retry: task stays deleted. **R2-T05 / FD-001**: simulate an uncommitted edit/delete with a lost response, then fail the snapshot; assert that selecting Retry cannot submit any task mutation until the snapshot succeeds, then assert the original operation ID and unchanged payload are used. **R2-T07 / FD-003**: interrupt the browser while an outcome is unknown; verify the warning does not imply rollback, no durable retry identity is restored, and re-entry guidance requires reviewing the current list before a new attempt. Verify receipt/task/revision atomic rollback on injected transaction failure and same-key/different-payload rejection. If reconciliation fails, remain locked with support guidance.

9. **Concurrency and response ordering** — FR-008/009/010; SC-004. Edit the same version from two contexts: one succeeds, the other gets an explicit conflict preserving its draft. Race confirmed delete with a newer edit: stale delete is rejected. Delay a pre-write GET until after acknowledgement: it cannot replace newer state. Change filters rapidly: only the latest query renders. Logout/account switch while requests are pending: old-account responses are discarded. A successful empty snapshot clears rows; a failed snapshot never does.

## Historical R2-Rev6 scenarios — superseded targeted expectations

These exact rows are preserved as R2-Rev6 approval history, not current acceptance where
CA-T01–16 supersede them (first winner, fresh per-interaction/after-arrival gates and delivery).
They were copied without changing their historical expectations. Use both header-delivery orders, reversed server completion,
separate same-profile tabs and independent device contexts; record actual auth/cookie effects,
server revocation, DOM/accessibility and intended/current epochs without secrets. No execution
or independent R3 pass is asserted. Browser establishment, not server creation alone, chooses
the winner. The approval does not choose any mechanism. Retain NAV/AM001/FD regressions.

| ID | Schedule / required observable outcome |
|---|---|
| R2-6R-T01 / FD-007 | Submit A/B while anonymous; establish A in browser first, hold B headers. Let A save a nondefault view, then release B. B fails with conflict/error; A remains authenticated with exactly the same account/epoch/view, task access and logout capability. Repeat with both server-completion orders and with the same account in both requests. No temporary competitor cookie installation followed by repair. |
| R2-6R-T02 / FD-007 | Reverse header delivery: B becomes the first usable browser login, A delayed. B remains unchanged/usable and A conflicts. Again reverse server completion and repeat same-account distinct epochs; request/commit ordering must not substitute for browser establishment. |
| R2-6R-T03 / S2 | Hold a distinct still-valid login's headers; original L becomes current, then logout L commits successfully; only afterward deliver that eligible late login. It may authenticate with its own fresh epoch/defaults. Direct revoked-L replay against private APIs fails; no L view/draft/queue revival or old-account effects. Repeat same/different account. |
| R2-6R-T04 / FD-007/S2 | Reject a competitor while L remains active; confirm that error responses cannot install authentication or alter L, even if their headers arrive late. Revoke L; the terminally rejected attempt does not revive automatically. S2 applies only to a distinct still-valid login. |
| R2-6R-T05 / FD-008 | With L visible in multiple runnable tabs, confirm logout in one. Observe prompt notification dispatch; receiving tabs clear private rows/labels/drafts/dialogs/task-view queues and navigate to login without waiting for a poll/action. No private content in accessibility state. No requirement to wait for all tab ACKs before confirming server revocation. |
| R2-6R-T06 / FD-008 | Suspend/hide or history-cache an L tab; revoke L elsewhere. Drop notifications, then resume/focus/pageshow it. It starts concealed and revalidates before private reveal/action; 401 stays private-free, 503 stays concealed with Retry/support. No simultaneous rendering during suspension is required. Also drop notification to an already-focused tab and trigger a protected action/new private response: mandatory gates still enforce invalidation. |
| R2-6R-T07 / FD-006 | Hold an A private response. Successfully revoke A and establish B via allowed fresh/late login; drop logout notifications. Deliver A data while the old tab is focused. Validate current B after arrival, discard A payload before any new private DOM/accessibility reveal; clear old state, no A draft/queue transfer. Repeat with same-account new epoch and known invalidation. |
| R2-6R-T08 / FD-006 | Deliver a same-epoch private response, then fail its required current-auth validation. Data stays concealed; Retry/support, no empty-success or view-default bypass. Recovery reveals only data matching the newly validated identity/generation/query/revision. Race auth changes around the validation/reveal boundary; R3 must demonstrate the ordering protection, not count a pre-dispatch check as proof. |
| R2-6R-T09 / FD-006/Q1 | Exercise every accepted protected-action entry, including cached detail/edit/delete/filter/sort, remembering and logout. After validation of A, revoke A and establish B before dispatch; send old A intent with B's valid CSRF. Expected-epoch guard rejects before private access/effects/receipt/view mutation. Never rebind or replay old intent to B; no per-keystroke checks in an open draft. |
| R2-6R-T10 / FD-001 | Separately inject ordinary task-refresh failure after authorised success, required-auth-gate failure, and view-only read/save failure. Task failure retains visibly stale rows/error and locks all task mutations/retries until valid task load; auth uncertainty conceals. View-only failure stays silent/isolated. Auth success alone never unlocks an unknown task operation. Retain exact operation ID/payload and explicit retry. |
| R2-6R-T11 / FD-008 | A separate browser/device has same-account login and its own nondefault view. Logout this profile. Other device remains authenticated, keeps its view and can synchronise saved tasks. No account-wide revocation or cross-login settings inheritance. |
| R2-6R-T12 / FD-008/S2 | After confirmed L logout, establish valid N, then deliver a delayed notification for revoked L. Clear no N-owned view or authority, never send old logout as N. Any login-URL navigation obeys R2-NAV-001 and reaches validated N tasks; no old L content reappears. |
| R2-6R-T13 / R2-NAV-001/CSRF | Authenticated root/new tab and GET login reach existing tasks/view; valid-CSRF login POST gives 303 before evaluating A/B replacement credentials. Invalid CSRF remains 403; auth storage failure 503, never anonymous. Stale pre-login token recovery is bounded to intended login; no password/task automatic replay or newer-login logout. |
| R2-6R-T14 / retained R2 | Regress AM001-V01–10, NAV-01–08 intent, R2-T05/T07/T12, task ownership/version/receipt replay, filter/sort ties, all-or-nothing silent defaults and valid-login rekey view retention. Fresh login after revocation defaults; competing failure and authenticated navigation preserve winner view. No browser/storage coverage change. |
| R2-6R-T15 / logout/task ordering | Test task commit before logout, then revocation winning before command effect; no old-authority command commits after logout success. Lose/malform logout ACK or return CSRF failure: remain uncertain, not false success. Clear local data safely; identity change alone is not proof of revocation. Recover through existing Retry/support without replay against a newer login. |
| R2-6R-T16 / FD-007/S2 | After L's revocation, establish C before a distinct delayed B response. C is now protected: B cannot replace its auth/account/epoch/view. S2 is not a bypass of another established login. Repeat same-account distinct epochs and opposite header order for which eligible fresh login first establishes. |

## Approved R2-Rev8 acceptance scenarios — CA-T01–16, planned

The following rows are copied verbatim from the exact reviewed/approved amendment scenarios.
They govern current targeted acceptance; historical R2-6R/R7/A9 material is not a requirement to
preserve the former first-winner, atomic-render or guaranteed-delivery outcomes.
Capture actual header order independently of server/callback order, protected owner/intended
epoch effects, known/observed vs unobserved state and recovery. No execution is claimed.

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

## Completion evidence

### Retained R3 scenarios — planned, interpreted under R2-Rev8

| Scenario | Required fixture, action and observable/assertable evidence |
|---|---|
| R3-V01 / TD-001 | A in same-profile tabs; hide/blur/resume/history, switch selected login, drop messages and delay/fail reads. Enforce boundary concealment/revalidation, known-stale discard, observed-change clearing and owner/epoch guards; permit only approved AC-010 unobserved prior-authorised display. No atomic render/first-winner requirement. Reverify affected TD-001 support if R3 replaces its old mechanism |
| R3-V03 / TD-003 | Reproducible technical-design §4 fixture matrix: 100 registered/50 active accounts, 100 visible tabs/6.7 polls per second, 1,000/10,000 tasks, long text, 1/4/10 million receipts, concurrent writes/view saves. Record resource/lock/payload/browser/receipt-index measurements; inject body, queue, connection, lock and deadline limits. No SLA/cap or measured success implied; no silent truncation, receipt expiry or failed-data replacement |
| R3-V04 / TD-004 | Persist HMAC format-1 receipts with independent key custody; verify no plaintext task command/response in receipts or logs, changed-input conflict and constant-time compare use. Rotate key, replay old request, remove old key temporarily: historical acknowledgement or safe 503/no new effect, never re-key/new operation. Document table-only versus combined key/DB threat rather than claiming absolute erasure |
| R3-V05 / TD-005 | Parallel workers share login/registration source/pair/service counters; forged Forwarded/X-Forwarded-* and direct private-upstream access cannot bypass enforcement. Include IPv4/IPv6, shared NAT, fixed-window bursts, distributed sources, limiter outage/timeout and nonexistent usernames. Assert generic 429/Retry-After or fail-closed 503, no partial user/session, no password logs, valid existing task sessions not revoked by limiter outage |
| R3-V06 / TD-006 | Barry accountable; Codex prepares drill/runbook. In disposable restore, use consistent users/tasks/revisions/receipts/schema and separate historical keys; invalidate restored sessions/views, clear logout confirmation receipts and limiter state. Verify owner isolation, pending replay without deleted-task resurrection, safe request-ID diagnostic lookup, migration failure/forward recovery and compatible rollback. Record retention/access approval and observed restore/loss window without promising RPO/RTO |
| R3-V07 / TD-007 | Barry accountable for R4/upgrade compatibility; format-1 default/Unicode/date fixtures survive deployment of format 2 and key rotation. Repeat create→delete→historical replay; altered content still conflicts. Compatible rollback passes preflight; incompatible rollback is blocked for forward recovery, never silently reinterprets old receipts |
| R3-V08 / TD-008 | Separate PostgreSQL connections/barriers: actual login after logout/expiry creates fresh session/epoch/defaults; authenticated login GET/POST preserves current login/view. Cover anonymous cookie response ordering, logout/command locks, receipt ACK recovery and delayed non-login responses with no auth-cookie writes. No maintenance rotation endpoint in R3-Rev10; no invalidated key/view resurrection or unrelated-session revocation. Full schedules: CA-T01–16 and R10-X01–03 in the R3-Rev10 validation companion |
| R3-V09 / TD-009 | Hold ACCESS EXCLUSIVE SessionViewState relation lock while auth/task relations work. Assert bounded 150 ms optional wait and complete silent defaults, independent successful task snapshot, no late view-result application. Also block pool/transport/cancellation, occupy all optional slots and probe breaker; primary pools/timeouts remain unaffected. Corrupt each view component and fail auth/task separately to retain proper errors/FD-001 |

### Historical pre-Rev7 technical schedules (not current acceptance outcomes)

R5-V identifies the earlier submission, not release-phase R5 authority. These rows are retained
as trace history; R7-T below supersedes browser-handle rotation/forced-relogin expectations.
They are not permission to implement the withdrawn provisional-cookie writer. These refine/supersede
Rev4's R4-V schedules whose full-cookie premise was rejected; prior reports/maps/manifests remain
history. Retain original R3-V01/V08 regression intent with the current single-cookie/family
expectations. Never accept an old-account snapshot merely because a request omitted newer proofs.

| Case | Required scheduling / evidence |
|---|---|
| R5-V01a / TD-001, refined | Reject a stale B login candidate targeting live A at PREPARE/ACTIVATE. Test legitimate missing-proof family claim-first ordering using rotation instead: all pending family candidates are fenced and delayed activation is 409. Drop broadcasts/fail later task reads; no cross-account private DOM/accessibility exposure, retain approved stale rows/FD-001 lock |
| R5-V01b / TD-001, refined | Reverse legitimate rotation order: pin new same-account handle first; old-handle claim is 401/concealed. Account B is entered only after logout/A revocation and fresh anonymous login; old A cannot reveal tasks. Repeat claim with no auth cookie, only obsolete cookies and failed registry lookup; no fallback/private reveal |
| R5-V01c / TD-001 | Delay PREPARE registration/ISSUE headers/claim replies; blur between every pre-dispatch check/await. No background continuation; already-dispatched activation is family-fenced. History/bfcache paint/accessibility concealed before valid authority/snapshot |
| R5-V08a / TD-008 | Authenticated same-family forms redirect/reject without candidates; separate anonymous roots, no original Session row; reverse ISSUE/ACTIVATE responses, selectively omit proofs after dispatch, then claim actual one-slot handle. Only current ACTIVE family-pinned handle authenticates; late other-family activation cannot change browser identity |
| R5-V08b / TD-008 | Remove newest cookie/obsolete floors while retaining/restoring retired same-family A; test provisional/cancelled/retired/revoked/expired/unknown/duplicate values before and after cleanup. All fail closed; no old-account selection or middleware resurrection |
| R5-V08c / TD-008 | One Set-Cookie per handle and below 512 bytes; header loss/rejection/eviction, per-origin cookie pressure, fifth PREPARE, 120-second boundary, 60-second cleanup cadence/backlog and expired in-flight request. Late ISSUE is inert, no blind resend/ever-active-cookie rewrite; actual committed success can be read only with valid current handle |
| R5-V08d / coupled regressions | Logout from every same-login tab, omit pending B cookie, delay valid B ACTIVATE across logout: family revocation wins. Other tabs clear/conceal before revisit, independent device unaffected. Regress same-user fresh defaults after logout/expiry, authenticated stale-form preservation, rotation expiry/view preservation, multi-tab task/view concurrency, FD-001/003/004 and AM001-V01–10 |


R5-V01a/b's old live-family different-account login is now an invalid-state rejection fixture.
Retain partial-cookie and both-lock-order evidence with valid rotation plus logout/fresh-login
fixtures. A redirect does not remove already-dispatched anonymous requests; test delayed ISSUE/
ACTIVATE, omitted cookies, lost broadcasts, failed later reads and stale-epoch writes under the
retained family/handle/focus guards.
Cookie storage pressure may evict/reject the current cookie; it cannot authorize a surviving old
attempt name. Restoring retired same-family A still fails family.current_handle validation.

Check issuance retries/proxy behaviour: no-store POST responses cannot replay an ever-active
handle Set-Cookie. Only first ISSUE can set a fresh provisional value; activation/logout/errors/
GET/task/view must contain no auth-cookie write. Cleanup with delayed valid request headers must
reject missing/expired/cancelled rows without upsert; record queue/count/age/backlog under existing
limiter/primary-pool budgets. No browser lock, new supported-client exclusion or privacy allowance.
Extend R3-V06 restore fixtures to clear/revoke restored Family/Handle/Transition state before
traffic alongside Django sessions/views; delayed valid headers cannot reconstruct authority.

### Navigation clarification scenarios — planned

NAV identifies this correction, not a development-phase gate. No scenario is an observed pass.

| Case | Fixture and expected outcome |
|---|---|
| NAV-01 / FR-010, FR-012, AC-005 | A valid login with acknowledged nondefault view; open Taskify root and /tasks/ in a same-profile tab. Direct task entry, same epoch/account, complete remembered view; no login form, candidate, cookie write or default reset |
| NAV-02 / FR-010 | GET /accounts/login/ while A authenticated, including next/username parameters: 302 /tasks/, no form or auth/session/view write. Task-shell requests remain authenticated; observed logout/identity change conceals/clears. Approved AC-010 governs any unobserved prior-authorised display/in-flight completion |
| NAV-03 / stale POST | Load anonymous form before another tab signs in as A; submit valid or invalid B/same-A credentials with current A cookie. CSRF-valid POST is 303 /tasks/ without credential replacement, new candidate, cookie or view reset. Invalid CSRF is 403 with A intact |
| NAV-04 / history and internal paths | Restore stale login form: revalidate before submission and redirect if authenticated. Any retained internal route must preserve authenticated-request guards and ownership; no PREPARE/ISSUE/ACTIVATE architecture or competitor-conflict endpoint is required by R2-Rev8 |
| NAV-05 / shared logout and switching | A cannot change account at login page. Logout from any same-session tab, wait for committed shared revocation, then explicitly sign in B (or A anew): fresh epoch/defaults. All old tabs conceal/clear before revisit; independent device session unaffected |
| NAV-06 / failure | Fail authority lookup at entry/login GET/POST: technical error/Retry/support, no assumed anonymous state, candidate or private reveal. Task-load failure after redirect retains FD-001 and AM001-V10; view-only failure remains silent full defaults |
| NAV-07 / in-flight races | Reorder genuine anonymous login headers under conventional cookie selection; authenticated requests still redirect without credential replacement. Drop notices/delay reads: reject known mismatches and old-intent operations, clear on observed change and allow only approved AC-010 unobserved prior-authorised display. No browser-wide winner/render fence |
| NAV-08 / R2-NAV-001 | GET/POST login while already A preserves epoch and acknowledged view. Same-user login after explicit logout/expiry creates fresh defaults. R2-Rev5 §7 approves this navigation distinction; executable behaviour remains untested |

### Inherited R3-Rev7 mechanism scenarios — planned historical inputs

R7-T names historical technical material, not current acceptance or a downstream gate. CA-T01–16
govern conventional ordering, known-stale/boundary checks and non-guaranteed notification.
Retain useful race stimuli, not the old first-winner/atomic-render expectations or mandatory
family/phase architecture. P1/R7-T08 remains paused, not expected implementation.

| ID | Required scheduling and evidence |
|---|---|
| R7-T01 / rotation | Live A in two tabs; race claim/rotation/logout and delay every rotation response. Assert zero authority-cookie writes, same browser handle/account/epoch/view/expiry, old Django key removed, requests re-resolve inner key, authoritative any-tab logout succeeds |
| R7-T02 / anonymous, HOLD | Two forms submitted before either login is acknowledged; A becomes usable, deliver delayed/cancelled B candidate response. A must remain usable, no account switch/view reset/forced sign-in, original active family must remain authoritatively revocable. P1 is conditional, not an adopted expected implementation |
| R7-T03 / shared logout | Delay candidate/activation headers across successful logout then fresh B sign-in. Old requests cannot reactivate A, erase B credential or misreport logout; all old tabs conceal/clear before revisit, independent device unaffected |
| R7-T04 / CSRF handoff | Keep pre-login tab with old form/header token; log in from another tab, rotate CSRF, restore first tab. GET csrf + mandatory claim succeeds with current token; stale login form navigates to tasks; no needless re-login/default reset |
| R7-T05 / logout CSRF | Force first logout/claim 403 csrf_failed; observe no domain effect, fresh token and one same-epoch retry. Success only on actual revocation ACK. Second 403/503 stays concealed with Retry, never false success |
| R7-T06 / changed identity | Switch A→logout→B while A's token refresh is delayed. Old A logout retry must not revoke B; discard old responses/drafts/queues and validate B through normal shell |
| R7-T07 / task/view recovery | CSRF rejection plus failed saved-task read: every task mutation/retry stays locked (FD-001), exact pending ID retained. View-only CSRF failure is silent, no old-tuple replay or task lock |
| R7-T08 / P1-A, conditional | Atomic first enrolment across windows; blocked/private/cleared storage, cancelled transactions, cookie pressure/partial loss and old-name cleanup. No new availability behaviour is a pass until the required decision/review |
| R7-T09 / regression | NAV-01–08/R2-NAV-001, AM001-V01–10, FD-003/004, ownership/epochs, original receipt/privacy/throttle/optional-reader budgets and R3-V06/07 handoffs |


R7-T01 replaces the old maintenance rotation ISSUE fixture: assert no such ISSUE is generated,
zero authority-cookie writes, stable handle, transferred inner key/view and both family-lock
orders. R7-T02 still tests the exact two-anonymous-family race rather than classifying induced
credential loss as external eviction. R7-T03 inspects actual server revocation and credential
continuity, never treating 403/missing cookie as a successful logout.

R7-T04/05 capture actual login-time CSRF-cookie change and stale older-tab header. Refresh via
GET /api/csrf/; first trusted pre-dispatch 403 permits only one same-epoch auth-housekeeping retry.
If account changes during the token fetch, never logout that new account. Repeat with second
403, 503, proxy HTML 403 and lost logout response: no false success. FD-001 and silent view
remembering remain in force; no password/task-command replay from CSRF recovery.

All resource/time numbers are engineering configuration under technical-design, not product success
thresholds. Execute only after the required design/Build authority, against disposable data. Collect
raw environment/patch/configuration/results and regress R2-T05 (FD-001), R2-T07 (FD-003), R2-T12
(FD-004), AM001-V01–10, ownership, sorting/calendar, task concurrency and lost-response recovery.
An author document check is not R3 verification or a runtime pass.

### R3-Rev10 technical validation companion — not executed

[ADR-AS-004 lifecycle](auth-transition-protocol.md) and the
[deterministic validation plan](review-followups/r3-rev10-validation.md) provide one schedule for
each approved CA-T01–16 plus cookie-adapter, lock, logout-receipt and legacy-regression fixtures.
The approved CA rows above remain unchanged. Earlier family/rotation/winner/reveal fixtures
remain history, not operative assertions contradicting CA-006/007/008.
R3 NOT_READY; TD-008/010/011 OPEN. No commands/tests in this guide have been run by this remediation.

### Future release evidence

Record command results, browser/device versions and mapped scenario outcomes in implementation validation notes. Verify HTTPS, secure cookies, no-store headers, proxy authentication throttling, least-privilege production database configuration, backup operation and a working support contact. These are release checks, not provisioning actions authorized by this planning command.

Do not run tests against production or create a user-facing reset/restore mechanism to facilitate validation. Reuse isolated test databases and fixtures.
