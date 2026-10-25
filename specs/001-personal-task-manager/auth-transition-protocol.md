# Authentication lifecycle — R3-Rev10 candidate

**Date:** 2026-10-25. **Status:** author technical proposal for independent review.
Scope v1.2 / AM-002 and R2-Rev8 remain APPROVED. R3 NOT_READY; TD-008/010/011 OPEN.
FD-009 remains RESOLVED_VERIFIED. No implementation, executed tests or gate approval.

## 1. ADR-AS-004 — native sessions with bounded integration

Use Django 5.2 authentication, PostgreSQL SessionStore, native password validation, login key
cycling, logout flushing and CSRF; retain existing account-owned repositories, receipts,
SessionViewState and polling. Genuine anonymous logins follow browser cookie response ordering.
Production cookie uses the configured native name, Secure/HttpOnly, SameSite=Lax, Path=/ and
host-only scope over HTTPS. Preserve same-origin/no-CORS and no-store private responses.
Remove AuthorityFamily, AuthorityHandle, AuthTransition, PREPARE/ISSUE/ACTIVATE and periodic
maintenance rotation from the candidate. No election, browser fence, Web Locks or IndexedDB.

Only three server integrations are justified: intended-login epoch plus a short Session-row
transaction guard; a logout confirmation receipt for lost-ACK recovery; and a response adapter
restricting authentication-cookie writes to successful fresh login. Client generations/lifecycle
checks and BroadcastChannel are coordination, never authentication authority.

[Pre-Rev10 protocol snapshot](review-followups/r3-pre-rev10-auth-protocol-snapshot.md) preserves
prior mechanism history. Its instructions are superseded here; independent records are unchanged.
[Technical design](technical-design.md) §§3–8 retains unrelated decisions and owners/exits.
[Coverage](review-followups/r3-rev10-validation.md) is planned, not executed.

## 2. Identity and session lifecycle

Django Session is the sole authentication record. Its encoded payload contains native
user/backend/auth-hash fields plus random UUID login_epoch and random 256-bit
logout_confirmation_token, generated once per actual login, including same-account new logins.
Epoch identifies intent; token proves only an already-committed logout. Neither substitutes for
the HttpOnly session credential. No token-only live revocation or task access is allowed.

SessionViewState remains an optional one-to-one session-key FK. Fresh login creates no view row
and therefore defaults. Valid-login navigation retains key/epoch/view/expiry unchanged. No
sliding save-every-request, periodic rekey or maintenance endpoint. Retain configured native
session lifetime/cookie policy; no new expiry/remember-me product rule.

| Event | Server result |
|---|---|
| Missing/expired/invalid Session | Page sign-in or API 401; no private tasks. Storage uncertainty is 503, not anonymity |
| Actual anonymous login | Native authenticate/login; fresh epoch/token; save final auth payload in one transaction before cookie/303 |
| Authenticated entry/stale form | GET 302 or CSRF-valid POST 303 /tasks/ before replacement credential evaluation; no cookie/Session/view write |
| Task/read/view operation | Locked live Session + intended epoch; no auth Session write/cookie refresh |
| Confirmed logout L | Atomic receipt plus native flush/deletion of L Session and cascading view; ACK only after commit |
| Expiry/security invalidation | Invalid authority, not proof of successful intended logout |
| Distinct valid anonymous response after revocation | May select N; L stays revoked, with no identity/view import |

Do not turn database exceptions into empty Session data. Signed-data decode failure is invalid
authority; storage access failure is technical uncertainty.

No live-session maintenance rekey is selected. Auth resolution must not silently cycle a key
outside this lifecycle, including Django auth-hash fallback rekey. Do not enable an unreviewed
live SECRET_KEY fallback/rekey deployment. R4 must review any future live rekey for preserved
epoch/view/expiry and response ordering, or a specifically authorised security revocation.
This is an upgrade prerequisite, not a runtime forced-login rule or changed expiry policy.
Password change/recovery endpoints remain absent.

## 3. Admission and task/logout ordering — TD-008

ATOMIC_REQUESTS remains false. Unsafe requests pass CSRF and wire validation before domain
dispatch. Middleware's lazy user/session cache is not final authority. In a short PostgreSQL
READ COMMITTED transaction, SELECT FOR UPDATE the captured Session row, re-decode that locked
payload and resolve native backend user/auth-hash/active status from it. Do not reuse pre-lock
auth caches. Check expiry against current database time after lock acquisition. Only this identity
may select owner; client account IDs never do.

Protected task reads/writes and view saves require X-Taskify-Login-Epoch; compare with the locked
session before receipt/task/view access, even for same-account new logins. Missing/stale epoch or
invalid authority returns existing 401; uncertainty 503. Never rebind an old receipt/draft to a
different epoch merely because the account matches.

Decode with native SessionStore, then run Django get_user/backend/hash validation against an
in-memory session facade built from that locked payload, discarding pre-lock lazy-user caches.
Its flush clears only the validation snapshot; it cannot delete/save a database Session.
Its cycle_key rejects an unsupported rekey before protected work. This narrow read-only facade
is part of the guard, justified because native get_user can otherwise flush/rekey even on GET.
Real login/logout still use native SessionStore. No custom password/session signature verifier,
alternate identity store or weakened native invalidation check. Invalid lookup returns 401/page
sign-in without writing Session; unexpected rekey returns technical failure before effects.

Order: Session → TaskListState → owner/operation receipt → task; view save: Session → view.
Existing account-state-first order remains the order *within* the task domain. No reverse
task→Session acquisition. Limiter transactions complete separately before auth/task locks.
Different device sessions share task-domain serialization but not revocation authority.

Snapshots hold Session then account lock while materialising owned rows/revision; release before
sorting/JSON/network work. Optional bootstrap view read remains independent/bounded under TD-009:
capture guarded key/epoch, release auth lock, attempt optional lookup, then re-enter Session guard
and verify the same identity for mandatory task snapshot. Never wait for optional work holding
Session/task locks. Ignore a view result for another login; fallback cannot bypass authentication.

Commands retain Session lock through receipt/version/task/revision commit. Logout takes the same
lock through revocation commit:
- Command first: effect/receipt commit before logout can commit or acknowledge.
- Logout first: deletion commits; waiting/later command finds no live row and rejects before
  receipt replay or new effect.
Middleware authentication before waiting is insufficient. No revoked-authority command can
commit after confirmed logout; already-authorised reads may arrive late under AC-010.
No browser/render/network wait is inside these transactions.

Use existing mandatory DB timeouts. Wait/deadlock/connection failure never means success.
Definite rollback may be rejected; connection loss around COMMIT is outcome-unknown. No automatic
task replay or new operation ID. FD-001 valid-snapshot locks and exact explicit retry remain.

### Anonymous login

After CSRF/admission limiting, resolve captured-cookie authority before credentials. If live,
redirect unchanged. Otherwise validate with native Django, then recheck that captured cookie in
the short login transaction before native login/save. If now live, redirect; uncertain fails
closed. A captured cookie is not proof of what another tab currently holds.

Genuinely anonymous requests may create independent sessions, irrespective of same account.
No account-wide exclusion, winner election or competitor revocation. Successful native response
cookies apply in either order; callback/server completion order is not selection authority.
After any login reply navigate to a clean task shell and discover current selected authentication;
never render tasks from login response metadata or replay passwords after redirected HTML.

## 4. Response adapter — authentication-cookie discipline

Retain native SessionStore encoding/key generation and native login/logout functions. A small
SessionMiddleware response adapter has these rules:
- Only an explicitly marked successful fresh login emits the native configured session cookie.
  Its final auth/epoch/token payload was persisted in the login transaction. Response processing
  emits that committed key without a second save or a second key allocation.
- All other responses emit neither set nor deletion of the auth cookie: logout/receipt replay,
  anonymous GET, redirects, 401/403/503, CSRF refresh, tasks and views. Preserve Vary/Cache-Control
  and the independent Django CSRF-cookie processing.
- Check for unexpected Session writes before leaving the protected transaction: reject/roll back
  guarded effects with safe diagnostics. Native logout flush is intentional. After commit no
  code may depend on a Session write; the response adapter never saves one. If a later middleware
  unexpectedly changes the in-memory session, diagnose/reject response processing without
  claiming already-committed domain effects rolled back. Existing unknown-outcome recovery
  applies. Middleware cannot recreate the Session.
- SESSION_SAVE_EVERY_REQUEST is false; no session-backed messages/preferences or anonymous
  CSRF storage may introduce another cookie writer.

Native SessionMiddleware can delete an empty-session cookie on a delayed response, including
logout, or save modified Session data during response processing. Delayed L deletion could erase
N; post-guard saving could violate revocation ordering. These are retained defects, not winner
election. Suppressing a header *after* an unintended save is insufficient.

Logout leaves an opaque invalid L cookie until native expiry/replacement. It is HttpOnly/Secure,
grants no access and cannot restore Session/view. This wire choice preserves coherent N continuity
without conditional cookie writes. Fresh anonymous login responses still follow CA-007.
No custom conditional Set-Cookie or cross-browser winner guarantee.

Native login still rotates the separate CSRF cookie. Delayed CSRF-cookie responses may produce
recoverable 403, never account authority. Preserve Origin/Referer, same-origin and native tokens.

## 5. Confirmed logout, receipts and recovery

A validated no-store shell/authority boundary provides the session's logout token to tab memory.
Never place it in URLs/logs/analytics/channel messages. POST /accounts/logout/ includes intended
epoch, token and current CSRF. Immediately clear/conceal local private state; retain only scoped
logout recovery context separately from cleared task drafts/queues.

LogoutReceipt has unique epoch, SHA-256 of the random token, revoked_at and retain_until (original
Session expiry plus 24 hours, an engineering cleanup budget). No Session FK, task content, account
identifier, raw token or session key. Only committed revocation may insert it; constant-time token
hash comparison. The receipt is confirmation evidence, never an active-session registry.

Wire binding for the existing logout route: same-origin POST JSON body contains exactly
logout_confirmation_token; X-Taskify-Login-Epoch carries the captured intended UUID and
X-CSRFToken carries the current masked CSRF token. Reject malformed/unknown fields under the
common error envelope. The receipt branch is only historical ACK lookup, not a protected
operation re-bound to the current cookie. Ordinary task/view wire contracts remain unchanged.

After CSRF:
1. Matching existing receipt/token/epoch can return the historical L ACK even with N/no cookie.
   This is read-only: no N logout, view change or claim that the browser is currently anonymous.
2. Otherwise acquire captured Session guard, then recheck receipt after waiting. New revocation
   requires live intended epoch, matching token and ordinary authenticated identity.
3. Insert receipt and native logout/flush Session plus cascading view deletion atomically.
   Only return success after commit.

Changed identity/no receipt cannot revoke or attest L. Never create a receipt from expiry, absent
cookie, CSRF failure or local clearing. Concurrent duplicate logout is idempotent: a waiting row
lock may find deletion, then a new READ COMMITTED receipt statement observes the committed receipt.
Token alone has no pre-commit revocation capability and no private read capability.

ACK shape: {schema:1, outcome:"logout_confirmed", login_epoch:L}. Verify trusted shape and exact
intended epoch. Lose/malform reply or COMMIT connection outcome: remain unknown. Explicit Retry
uses exact token/epoch, including read-only confirmation when N is current. A 401/403, changed
identity or missing cookie alone is never an ACK.

Trusted application 403 csrf_failed/outcome=rejected means no domain dispatch. Refresh
GET /api/csrf/ and read authority; allow one automatic housekeeping/logout retry only if the
same intended epoch is still authenticated. Otherwise do not replay logout as N. Second rejection
or lookup failure stays concealed with Retry/support. Task commands never auto-replay: recover
CSRF/authority/saved-task snapshot then explicit same-ID retry, subject to FD-001. View failure
stays silent, with no old-tuple replay. Stale login forms revalidate; never auto-submit passwords.
Generic/proxy errors and network loss are not trusted no-effect evidence.

Proof lost on reload or receipt cleaned up: no fabricated ACK; safe revalidation/Retry/support.
This respects memory-only interruption limits. Cleanup uses existing bounded maintenance after
retain_until; delayed cleanup cannot revive authority. R5 restore invalidates all sessions/views
and clears logout receipts before serving, preventing restored evidence from attesting revocation
of a restored live login. TD-006 restore exit verifies this addition.

## 6. Authority discovery and stale-state handling — TD-010

GET /api/auth/session/ is minimal no-store same-origin discovery: live cookie returns
{account_id, login_epoch, logout_confirmation_token}; invalid 401; uncertain 503. No task payload,
view/default/Session mutation or auth-cookie effect. This initial/lifecycle/recovery endpoint
discovers identity when no intended epoch is yet known; it never performs an old-intent operation.
All task/view reads/writes require the discovered intended epoch.

Shell markup contains no restorable private rows. Per-tab memory holds known account/epoch,
authGeneration, queryGeneration, accepted/acknowledged revision and exact pending operation.
Capture these at dispatch; before applying replies compare captured generation/identity and
response metadata with known state/query/revision floors. Reject known mismatches. Aborting fetch
is optimisation, never rollback evidence. Old callbacks cannot clear N or rebind old payloads.

On observed change clear old rows/labels/drafts/dialogs/task-view queues; separately scoped logout
confirmation context may remain. No durable browser private state.

Initial entry, focus/visibility return, pageshow/history restoration, stale-form restoration and
uncertain-auth recovery start concealed with a new boundary generation. Conceal on blur/pagehide/
hidden before suspension when hooks run. Only the latest boundary discovery may adopt identity;
clear old state first, then load matching task/view data. Valid saved-task loading and existing
pending-operation checks control write readiness. No simultaneous erasure while suspended is
promised; restore remains concealed through validation.

An unexpected current-generation identity result clears/revalidates; never attach old requests
to it. Stale prior-generation metadata/401 cannot invalidate a newer validated login. No extra
auth round trip per normal private response/cached UI interaction. Visible polling discovers
changes through guarded requests; no guaranteed message delivery/observation deadline.
Authority uncertainty at required boundaries conceals. Ordinary same-login task refresh failure
retains stale rows/error with FD-001 locks; optional view-only failures remain silent.

RA-FD006-001 remains exact: unobserved other-tab change may leave prior-authorised content visible
or permit its authorised response to finish rendering without a maximum delay. No atomic render
fence is required. Wrong-account APIs, old-intent effects, known mismatches and post-observation
retention remain prohibited. This is existing human risk acceptance, not a new exception.

## 7. Confirmed cross-tab notification — TD-011

Construct BroadcastChannel("taskify.auth.v1") at startup where supported, before reveal.
After a trusted committed/replayed L ACK promptly post
{schema:1, kind:"logout_committed", login_epoch:L, event_id:randomUUID}.
No token/account/task/credential data; no waiting for tab ACKs. Never broadcast an unknown logout
as confirmed. An ACK for L while N is locally current may notify L tabs, but cannot clear N.

A runnable recipient validates shape; if displayed/pending state belongs to L, invalidate local
generations, conceal/clear content/drafts/queues and navigate /accounts/login/ in that handler.
No wait for polling/action. Duplicates are idempotent, late L does not clear N. Unknown/in-progress
boundary work remains concealed, invalidated and revalidated; it cannot release an old result.
Login navigation with actual N follows R2-NAV-001.

Constructor/posting failure, unsupported/dropped/late delivery neither fails committed logout
nor imposes a capability requirement. Existing lifecycle/uncertainty checks and visible polling
provide discovery, without a new transport, faster poll or maximum delay. Suspended/history tabs
remain concealed through restoration checks. "Confirmed" means server revocation, never guaranteed
transport delivery or atomic clearing. Independent browsers/devices remain unaffected.

## 8. Evidence, alternatives and review boundary

Taskify-specific integration is a design deduction, not a framework guarantee.
[Django sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/) supplies native storage.
[SessionMiddleware source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/middleware.py)
shows response save/delete; [auth source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/__init__.py)
shows native lifecycle/auth-hash handling. [PostgreSQL row locks](https://www.postgresql.org/docs/17/explicit-locking.html)
support server serialization. [BroadcastChannel](https://html.spec.whatwg.org/multipage/web-messaging.html#broadcasting-to-other-browsing-contexts)
is messaging, not durable delivery. Sources consulted 2026-10-25; exact maintained patches and
adapter compatibility must be locked/verified at authorised implementation and upgrade review.

Rejected alternatives: account-wide exclusion affects independent devices; family/transition
registries and periodic rotation solve superseded guarantees; browser locks/storage add avoidable
availability dependencies; blindly native response save/delete misses retained logout ordering;
UI errors alone do not enforce isolation; "already absent" ACK cannot prove intended revocation;
per-response probes do not create atomic rendering and are not required.

No new product departure or risk acceptance proposed. Independent R3 must assess this candidate,
including cookie adapter/receipt complexity and deterministic schedules. Findings remain OPEN.
