# Proposed ADR-AS-002 — Option B: Django-native authentication design

**Date:** 2026-10-24. **Status:** Standalone proposal for targeted independent R3 review; NOT ADOPTED.
Scope v1.1 and R2-Rev5/R2-NAV-001 remain APPROVED. R3 NOT_READY; TD-008 OPEN.
P1/IndexedDB stays paused. No application code, new browser storage or operative-design rewrite.

This follow-up develops [ADR-AS-001](adr-auth-simplicity-assessment.md) Option B. The user has
authorised proposal work, not approval of the architecture or any residual. D1–D3 are not three
presumed approval gates: §7 distinguishes engineering choices from actual behavioural departures.

## 1. Architecture and limits

Use Django 5.2 database SessionStore in PostgreSQL and the configured single session cookie,
Secure/HttpOnly/host-only/SameSite. Keep Django credential verification, login fixation protection,
CSRF middleware and server logout/flush. Remove AuthorityFamily, AuthorityHandle, AuthTransition,
PREPARE/ISSUE/ACTIVATE, mutating authority claims and custom maintenance-rotation endpoints
**from this proposed candidate**. Existing historical documents are not deleted or amended.

Retain one public random login_epoch UUID in the Django session. It is created once on a
successful fresh login and remains stable across a valid-session key change. It is not a
credential, account selector, pre-auth browser ID or pending-login registry.
The session cookie remains the only browser authentication credential.

Application logic consists of route adapters, identity/epoch checks, short Session-row transaction
guards, the existing separate view record, UI response generations and one narrow middleware
response-policy change. No application SessionStore implementation or new auth table.

**Provenance:** Django supplies the core lifecycle, not cross-response ordering.
[Auth source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/__init__.py),
[DB backend](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/backends/db.py).
Source was inspected on 2026-10-24; a maintained patch must be pinned and its behaviour regressed
before implementation/release. Static reasoning below is not execution evidence.

**Coverage limit:** S3, task-write/logout ordering and rekey view identity have concrete proposed
controls. S1 selection is defined; S2 and S4 remain isolated technical blockers under unchanged
approved outcomes. Full live-key-rekey/logout support (S5) is not established. Do not call the
whole candidate ready merely because ordinary login is native.

## 2. Minimal HTTP and login lifecycle

| Route | Proposed behaviour |
|---|---|
| GET / | Valid authenticated session → 302 /tasks/; anonymous → login; storage uncertainty → technical error, not assumed anonymity |
| GET /accounts/login/ | Authenticated → fixed 302 /tasks/, retaining epoch/view; anonymous form has CSRF and existing disclosures |
| POST /accounts/login/ | CSRF first; authenticated → 303 /tasks/ before replacement credentials. Anonymous → Django form/limiter/login; success reaches tasks, failure generic. No custom candidate cookie/provisional session protocol |
| POST /accounts/logout/ | CSRF + intended login_epoch, fresh Session-row validation, atomic view removal and Django flush; acknowledge after commit, no session-cookie deletion response |
| GET /api/session/ | No-store authenticated account_id/login_epoch only; 401 invalid, 503 lookup failure; no login creation or default writes |
| GET /api/csrf/ | Same-origin/no-store current masked token; only CSRF-cookie side effects, no authenticated-session creation |
| Existing task/view routes | Same payload/receipt/version/filter/sort contracts; expected epoch is an authentication precondition |

Use LoginView's authenticated redirect policy plus a small method-aware adapter: stock
HttpResponseRedirect is 302 and safe next URLs can differ from the approved fixed /tasks/
destination. Preserve GET 302, authenticated POST 303 and invalid-CSRF 403 exactly.
[View source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/views.py).

On an actual fresh login, call Django login and set a fresh epoch in the new authenticated session
before normal response persistence. Never lazily create it on GET. Session persistence must succeed
before a successful login response is emitted. Authenticated navigation does not create another
login, reset the view, change cookie/epoch, or evaluate replacement credentials.

Two POSTs that were anonymous when submitted can both succeed. Their final cookie is whichever
valid login header the browser applies last; there is no first-submit/first-commit winner promise.
A subsequent request uses that session's account and fresh-login defaults. This ordering definition
does not permit false logout or stale private display: those are separate S2/S4 obligations.
Per-tab disabling and the authenticated-route guard reduce overlap but are not cross-tab locks.

## 3. Session identity, view state and transactional data protection

### View key and native rekey preservation

Keep the existing view tuple/schema, changing its proposed key from the transient Django key to:

| Field | Role |
|---|---|
| login_epoch UUID, primary key | Server-derived logical-login identity, already stored in verified Django session |
| owner_id FK | Verified owner check; never supplied as authority by JSON |
| schema_version + status/priority/sort/direction | Existing complete-tuple semantics; no preference version/conflict protocol |

Django cycle_key() preserves session data, including the epoch, so the same view row survives
without copying, cascade deletion or an auth-registry lookup. Fresh login has a new UUID and
defaults; logout deletes the old epoch's row. Never look up a view from an untrusted request
header alone: derive epoch/owner from the revalidated session and compare the expected header.
Missing/invalid/failed optional lookup still selects full defaults silently; GET does not repair it.
[Session key implementation](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/backends/base.py).

Expired/orphan view cleanup is conservative: use a consistent database snapshot of live Django
sessions and verified epoch references; do not delete a row merely because an old key disappeared.
Defer cleanup if liveness cannot be established. Keep scans off the request path, bounded under
existing maintenance resources; record backlog/storage and test concurrent rekey. Logout normally
removes the row directly. This is view housekeeping, never authority to authenticate an old epoch.
Cost/expiry correctness remains a required validation item, not a new retention promise.

### Task writes, view saves, snapshots and logout

For every private service operation, capture the request's session key but reload/revalidate its
persisted live session/owner/epoch under the Session row lock. Do not trust a cached request.user
or session dictionary across concurrent logout. Use the existing expiry/auth-hash checks.
There is no re-creation/upsert of missing Session rows.
Resolve Django authentication before entering the guard; inside it, verify the freshly decoded
persisted identity/hash/expiry without calling a hook that could flush or rekey. Missing, expired
or revoked session / mismatched expected epoch returns 401 before private work; database or
unreadable-session uncertainty returns 503, never an anonymous-success or empty-task fallback.

Lock order: **Session → TaskListState → receipt → task** for task commands/snapshots;
**Session → SessionViewState** for view saves/logout. Private reads, task writes and view saves
take a PostgreSQL **FOR SHARE** Session-row guard; logout takes **FOR UPDATE** before flushing.
Shared guards are mutually compatible, so an isolated view-save failure cannot block task data
by holding an exclusive Session lock. Only real data locks serialize task/view changes. This
small parameterized repository primitive uses existing PostgreSQL; Django ORM's exclusive
select_for_update alone would not give the needed convenience/task fault isolation.
[PostgreSQL row-lock compatibility](https://www.postgresql.org/docs/17/explicit-locking.html#LOCKING-ROWS)
provides the shared-read/exclusive-delete relationship; this is not a table-level SHARE lock.
Inside a shared guard, revalidation is read-only: do not flush/save/rekey or upgrade its lock.
Lifecycle deletion takes exclusive first; failed checks reject without a shared→exclusive upgrade.
No path takes Session after account/task/view locks. Optional view lookup remains outside the
task materialisation transaction and retains its independent bounded pool/timeout; never hold
TaskListState while waiting for it. Do not hold account/task locks while deleting a view.
Reuse existing 500 ms lock,
2 s statement / 3 s transaction and primary-pool budgets; keep network/serialization outside.

- **Task holds shared Session guard first:** revalidate epoch, perform existing owner-scoped receipt-first command,
  effect/revision/receipt commit atomically. Logout waits, then flushes that Session and deletes its
  view. The effect precedes logout ACK; a late reply is discarded by the UI after invalidation.
- **Logout wins:** flush/deletion commits first. Later task/view revalidation finds no live Session;
  reject before effect/receipt/upsert. Old cached auth cannot resurrect a saved command.
- **Other-device session:** remains independently authenticated; its legitimate same-owner task
  changes continue through the existing account revision/receipt/version guard.
- **Old A draft with current B cookie:** expected epoch mismatch rejects before any create/edit;
  do not attach A text to B, flush B or retry as B. Identity change clears old drafts/queues.
- **Snapshots:** authenticate/materialise under the same order; failed reads never become empty
  success. A response buffered before logout is still a UI generation/privacy concern (§6).

Logout captures the intended epoch; under its Session lock, remove view then call Django logout/
flush within one transaction. ACK follows commit. No account-wide logout, and no claim that
a completed flush locates unrelated sessions minted by earlier anonymous requests.
A lost response remains unconfirmed. Response-time exceptions are not proof a task rolled back;
FD-001, exact operation IDs, historical receipts and explicit valid-snapshot retry remain intact.

## 4. Narrow cookie-response policy — S3

Keep stock DB SessionStore. SESSION_SAVE_EVERY_REQUEST=False and CSRF_USE_SESSIONS=False
(default cookie-based CSRF). Auth-session dictionaries are read-only outside legitimate lifecycle
operations; view tuples, flash UI state and last-seen timestamps do not cause normal cookie saves.

Replace only the response middleware's **empty-session deletion** behaviour:
Django may emit a session-cookie delete when the incoming request had a cookie and its session
is now empty. Suppress that deletion for logout/invalid/empty-session responses, leaving only an
opaque server-invalid value in the browser. Keep no-store/Vary, unrelated cookies and actual
server flush. Normal login still uses Django's save and Set-Cookie with standard attributes.
[Middleware source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/middleware.py).

This small adapter is necessary for S3: an old deletion response cannot erase a newer login cookie
if no deletion header is sent. It is NOT a candidate-credential issuer, key allocator or session
registry. Never strip CSRF cookies or hide a legitimate key change after creating a new session.
Do not blindly suppress arbitrary nonempty-session writes after they commit.

Required review: retained revoked-cookie hygiene, all empty-session paths, SessionInterrupted,
no auth cookie on ordinary APIs, and middleware compatibility. Revoked cookie replay must fail
against the deleted DB row; restore must invalidate restored sessions before traffic as already
required by TD-006. This policy preserves server revocation but **cannot stop a late login
Set-Cookie from installing a different still-live session (S2)**.

## 5. Supported key changes — S5

Remove custom periodic rotation and its endpoints. Keep standard Django key rotation on fresh
login. Default proposal has no password-change/recovery route and uses Django's default empty
SECRET_KEY_FALLBACKS; it does not enable live-session fallback rekey as an unreviewed feature.
[Default setting](https://docs.djangoproject.com/en/5.2/ref/settings/#secret-key-fallbacks).

A future native valid-session rekey retains the epoch/view by §3. That resolves the view-key
dependency, not its cookie/logout ordering: a new session key can still arrive after logout
using the old key. Full fallback/rekey operation is gated on S5 ordering evidence before enabling.
Do not disable fixation protection or accept S5 merely because the key change is framework-native.

No server-secret replacement or global session invalidation is authorised here. Such maintenance
needs its own reviewed deployment/rollback plan and Barry's existing R4/R5 handoff; ordinary logout
must still leave independent device sessions unaffected. If live fallback rekey is already required
by a deployment baseline, this candidate is incomplete for that configuration, not permission
to remove that requirement.

## 6. UI, CSRF and isolated privacy/ordering residuals

Retain concealed initial/history/focus state, no-store, same-generation identity/epoch/query/revision
checks and draft clearing on invalidation. BroadcastChannel only prompts earlier clearing.
Before restored/pre-login tabs POST claim-equivalent housekeeping/logout, fetch current CSRF.
One trusted pre-dispatch csrf_failed retry may follow fresh token and same-intended-epoch check;
never logout a new identity, replay passwords/task commands or claim success from 403/401.
View-only failures stay silent; FD-001 still blocks every task mutation/retry until valid saved data.

| Scenario | Disposition in this proposal |
|---|---|
| **S1** competing anonymous forms | Response-order selection defined; no exact winner added. Both authenticated server reads stay owner-scoped. Last response may select another fresh-login view; no claim that resulting S2/S4 consequences are acceptable |
| **S2** hold B login response, establish A, logout A, then release B (also B=A) | **OPEN technical blocker.** Flushing S_A does not delete S_B minted earlier. No control in this candidate prevents later authentication. It is not recovery of S_A and not an accepted FD-004 consequence |
| **S3** late logout/empty-session deletion after newer A login | Addressed by §4: no delete header can remove A's cookie. Actual current-session logout remains server-side flush |
| **S4** focused A already visible; late B cookie; notification lost and reads fail | **OPEN privacy proof gap.** Epoch guards prevent wrong-session writes; they cannot erase already-painted A. Revisit gating is mandatory but is not continuous knowledge of cookie changes. No exposure window is accepted |
| **S5** native fallback rekey response after old-key logout | View tuple survives through stable epoch, but late new-key authentication remains unproved. Default profile does not enable that path; extension gate in §5 |

An ordinary failed task refresh still retains stale rows/error and locks writes as approved.
An actual failed authority check conceals private data. Do not relabel every network failure as
auth failure to hide rows and then claim the approved stale-row experience is preserved.
A read-only session check also cannot close the interval after it returns but before/after paint.

S1/S2 are low-likelihood ordinary overlaps qualitatively, not measured; duplicate same-account
forms are more plausible than different-account entry. S2 can expose private data on a shared
device after sign-out; S4 can expose old-account DOM. CSRF and valid credentials remain required,
so this is not by itself an unauthenticated cross-site access bypass. Residuals are unaccepted.
[RFC cookie ordering](https://www.rfc-editor.org/rfc/rfc6265.html#section-4.1.1).

No global user logout, per-account lock, last-login timestamp or timeout is substituted for a
browser ordering guarantee. A DB lock can order commits, not already-buffered HTTP headers.
No signed cancellation claim or incomplete Cookie header is treated as discovery of every
anonymous attempt. P1 and new browser storage remain out of this proposal.

## 7. Human decisions: only actual departures, not D1–D3 by default

| Prior label | Treatment now |
|---|---|
| D1 | No human approval sought for response-order selection where no winner is specified. Normal new-login defaults/valid-login retention stay as approved; any concrete contrary view/continuity effect must be assessed rather than presumed accepted |
| D2 | Preserve approved shared logout. S2 is presently a technical shortfall, not a reason to ask Barry to redefine logout. Only a future proposal explicitly allowing a pending login to authenticate after logout would be an actual targeted R2/human change |
| D3 | Preserve approved privacy/revisit and ordinary failure behaviour. S4 remains technical work for independent assessment. Allowing a specified stale-private-display interval, or hiding ordinary task-error stale rows, would be actual R2/privacy changes; neither is proposed |

No functional weakening is requested in this candidate. If S2/S4 cannot be met by a simple
design, report the precise remaining mechanism or an explicit alternate proposal; do not make
all ambiguity a human gate or infer permission from rarity. R3 review may reject this incomplete
candidate without any product decision. Existing TD-001 resolution remains historical; removing
its family fence requires fresh supporting evidence, not automatic transfer of that resolution.

## 8. Deterministic regression specification — planned only

All cases use actual response-header scheduling, separate DB connections, CSRF/epoch capture,
server Session/view/receipt state and DOM/accessibility/history observations.

| Case | Schedule and required observation |
|---|---|
| B-01 / NAV-01–08 | Authenticated entry/GET login 302 and POST 303 preserve epoch/view, no auth cookie/reset; CSRF 403 preserves Session; logout then fresh same/different user has defaults; separate device stays live |
| B-02 / S1 | Two anonymous A/B POSTs pass CSRF before responses; reverse cookie-header delivery. Record final selected Session/account/epoch/view and owner isolation. Exercise S4 separately; do not label the whole schedule privacy-safe from correct API ownership |
| B-03 / S2 | Persist both logins; hold B headers; A task view then logout ACK; release B. Check GET tasks/session and old-key replay. Current framework proposal is expected to reproduce the blocker; renewed authentication cannot be marked a pass |
| B-04 / S3 | Hold an earlier logout/empty/expired-session response; establish new A; release old response. Assert no deletion/ordinary auth Set-Cookie, A remains usable, and subsequent A logout really deletes its Session |
| B-05 / stale writes | Deliver B cookie then submit old A create/edit/status/delete/view intent with valid current CSRF but old epoch. Reject before task/receipt/view effect; no A text in B, no automatic retry or flush of B |
| B-06 / task–logout | Task acquires Session first: commit task/revision/receipt, then logout commits; reverse locks: reject before any effect. Lose task reply, fail saved-task snapshot: FD-001 still blocks all retries. Independent device can still legitimately edit |
| B-07 / view identity | Save complete nondefault tuple; use standard cycle_key() retaining epoch. Same logical login finds identical tuple; old-key deletion cannot cascade it. Hold view row/relation locks and stall a view save holding its shared Session guard: independent task snapshot/write still succeeds within retained budgets; optional load defaults within 150 ms. Regress write/logout locks, fresh epoch defaults and invalid tuple/full fallback |
| B-08 / CSRF | Pre-login tab survives another tab's login; refresh token before unsafe auth work. Force first/second trusted 403 and identity change during refresh; no wrong-account logout, false success, password/task replay or view-warning regression |
| B-09 / S4 | Reveal A, release B's cookie while A remains focused, drop broadcasts, fail later reads and deliver late A payload. Record private paint/accessibility. If old A remains under B, record blocker; no acceptable time threshold is invented |
| B-10 / S5 | Check default no live fallback path and retained login fixation rotation. In explicit extension fixture, native rekey followed by old-key logout/late reply must preserve view AND prevent post-logout auth; extension cannot enable until both hold |
| B-11 / middleware/data | Ordinary task/view/token GETs do not modify auth session; test stale DB save/SessionInterrupted, no deleted-auth resurrection, cookie flags/no-store/Vary, server restore invalidation and view orphan cleanup across key changes |
| B-12 / retained quality | AM001-V01–10, R2-T05/07/12, task revisions/receipt replay/HMAC keys, trusted throttle, optional-view isolation and existing lock/pool/body/timeout budgets; measure added Session-lock wait/deadlocks before claiming capacity |

No code/tests/deployment have been executed. See the [evidence/change map](option-b-django-native-evidence-map.md)
for precise prospective artifact changes. S2/S4 and the S5 extension gate keep this proposal
unapproved; R3 NOT_READY / TD-008 OPEN is unchanged.
