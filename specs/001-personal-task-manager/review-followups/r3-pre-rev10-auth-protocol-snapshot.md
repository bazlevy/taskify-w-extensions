# Archived pre-Rev10 protocol

Original body retained below with relative link destinations adjusted for this archive location.
Historical proposal only; [current lifecycle](../auth-transition-protocol.md) supersedes its mechanism.

# Authentication protocol review boundary — R2-Rev8 and historical R3 snapshot

**Date:** 2026-10-25. **Status:** historical technical candidate with current functional boundary.
Scope v1.2 / AM-002 and [R2-Rev8 approval §4](../../../independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md) are approved. FD-009 RESOLVED_VERIFIED
per §2; R2-Rev7 remains NOT_READY in its historical report. R3 NOT_READY; TD-008/010/011 OPEN.
P1/IndexedDB paused; no architecture or runtime proof is granted.

## 0. Governing R2-Rev8 contract

Genuine overlapping anonymous responses follow conventional cookie selection. There is no
first-browser winner, mandatory competitor conflict, outer authority registry or transition
protocol obligation arising from FD-007. Authenticated GET/POST/stale forms retain R2-NAV-001.

Protected operations authenticate/owner/intended-login guard. Reject known stale responses and
clear on observed auth change; revalidate at initial/restored lifecycle and uncertainty boundaries.
No universal per-response/local-action auth trip or atomic validation-to-render requirement.
Approved AC-010/RA-FD006-001 covers prior-authorised display/in-flight completion while change is
unobserved; it never permits API isolation/rebinding, known-mismatch or post-observation violations.

Confirmed intended logout precedes success; promptly signal same-login tabs, and receiving runnable
tabs clear/navigate. No delivery guarantee/tab ACK/atomic clearing; missed messages recover at
boundaries/existing synchronisation. Newer-login and other-device authority stay protected.
Retain FD-001, native fixation/CSRF, bounded same-intended-epoch recovery and silent view failures.

## 1. Historical mechanism boundary

Sections 2–5 preserve the earlier R3-Rev7 snapshot for audit, not current endpoint/schema/acceptance
requirements. Its first-winner/fresh-render/generation-cancellation assumptions do not override
R2-Rev8. No compatibility writer or new mechanism is implemented. R3 must reassess a simplest
coherent mechanism against current [HTTP](../contracts/http.md)/[UI](../contracts/ui.md) and CA-T01–16.
Useful CSRF/continuity protections remain review inputs; specific family/phase/rotation routes
are not selected by this reconciliation. See [R3 handoff](../review-followups/r2-rev8-r3-handoff.md).

## 2. Rotation correction — no browser credential issuance

Maintenance rotation is a single server transaction, not PREPARE → ISSUE → ACTIVATE.
POST /api/auth/rotation/ authenticates the current browser handle, intended public epoch and CSRF.
Under AuthorityFamily → Session → SessionViewState locks:

1. Recheck current ACTIVE handle/live family and unchanged account/epoch. If a captured
   generation precondition is stale, reject 409 with no change; never rebase across logout/login.
2. Allocate a new server-only Django Session key; transfer validated auth data and complete valid
   SessionViewState atomically. Preserve account, public login_epoch and original expiry.
3. Update the family's current_session pointer, remove the old Django Session/view rows, advance
   generation and commit. Keep the SAME current browser AuthorityHandle ACTIVE.
   Retarget live session FKs before deletion; historical transition references must be SET_NULL/
   non-cascading metadata where appropriate, never cascade-delete the live handle/family.
4. Return only after commit; emit NO authority-cookie Set-Cookie, expiry, alias, provisional handle,
   CSRF rotation or Django login hook. It is key maintenance, not browser-credential replacement.
   A compromised browser credential requires revocation/logout, not this maintenance adapter.

There is no rotation ISSUE response to arrive late. A previously queued stale rotation ISSUE/
ACTIVATE route is rejected (409/410), with no cookie or Session mutation; never retain a legacy
cookie writer as compatibility. No deployed code/migration exists to preserve.

Current same-epoch requests resolve the family's current inner Session under guard. Do not turn
an old captured inner Session key into an auth 401 after a valid maintenance rotation: re-resolve
only through the same ACTIVE handle/account/epoch to the transferred Session/view. No request can
rebind after logout/new epoch or recreate a deleted key. Task ownership/version/receipt checks are
unchanged. A racing view save either commits before transfer or applies to the current same-epoch
view afterward, never revives old rows.

Claim-first may reject a stale rotation precondition; rotation-first leaves the browser handle
usable and a later claim validates the same account/epoch/view. Both orders preserve continuity.
Logout uses that same valid handle, locks family, advances generation, revokes handle/Session/view
and cancels registered pending candidates. Old maintenance requests cannot resurrect them.
Independent device/browser families remain unaffected.

## 3. Cross-tab CSRF revalidation and bounded recovery

Login rotates Django's CSRF secret; an older tab's cached HTML/form token is not current authority.
GET /api/csrf/ is same-origin, no-store, returns a masked get_token() value and may set ONLY the
CSRF cookie. It neither creates an authenticated Session, returns private identity nor writes
tasks/view/authority credentials. No CORS relaxation; keep Origin/Referer and CSRF middleware.

Before mandatory claim/reconcile/logout from a restored/pre-login tab, obtain a current token
through this GET. Never rely only on a cached hidden input. Use that token in X-CSRFToken.
A later login can still race token acquisition, so a genuine trusted application 403 csrf_failed
with outcome=rejected means CSRF middleware rejected BEFORE auth/task service dispatch.

| Operation | After trusted CSRF rejection |
|---|---|
| claim / auth reconciliation | Stay concealed, GET fresh CSRF, perform read-only authority check and retry housekeeping at most once in current local generation. Second 403/lookup failure: technical Retry/support or safe no-store shell reload, no private reveal |
| logout | Clear/conceal on intent, retain intended account/epoch in memory. Fetch CSRF and read authority; only if the same intended epoch is still authenticated may one automatic retry send the original logout intent with fresh CSRF. If identity changed, do not log out the new account. No success until authoritative shared-revocation acknowledgement |
| login / stale form | Fetch CSRF and read authority. If already authenticated, navigate to tasks under R2-NAV-001; otherwise require explicit credential submission, never auto-replay passwords |
| task mutation / unknown operation | Preserve original operation ID/payload; do not silently resubmit. Refresh CSRF/authority/saved-task snapshot and require normal explicit retry, still subject to FD-001/task version/receipt guards |
| view remembering | Preserve current controls silently; obtain fresh token for a later explicit selection. Do not add warning, automatic old-tuple replay or task-action lock |

Opaque HTML/proxy 403, malformed response and transport loss are not a trusted no-effect proof.
Use existing unknown-outcome rules, never infer logout success or task rollback. GET 401,
missing cookie or a changed identity alone does not prove the intended old family's logout
committed. Show unconfirmed/retry where needed; never report success for CSRF rejection.
Actual login still rotates CSRF normally; rotation in §2 does not. Valid repeated login-page
navigation preserves epoch/view; no credential recovery/reset feature is added.

## 4. Anonymous continuity — unresolved mechanism, no accepted fallback

The former single-slot provisional ISSUE is withdrawn as a valid solution: a delayed B response
can erase A's usable credential and prevent authoritative logout. This is an internal defect,
not external cookie eviction, FD-004 or an accepted sign-in requirement.

The rotation fix alone does not repair two previously submitted anonymous login attempts with
distinct roots. This candidate does NOT claim TD-008 resolved. Do not implement the old
provisional-authority writer or treat forced re-login as an approved outcome.

[Historical unapproved P1 proposal](../review-followups/r3-rev7-browser-context-proposal.md) and
P1-A remain paused. The separate Django-native assessments are also unapproved. This reconciliation
chooses neither, creates no browser/storage dependency and does not assert that no alternative
exists. An actual need for a policy/browser/risk departure must return to Barry specifically.

The current private-account/navigation/logout contract remains mandatory, including anonymous
overlap. New-login delivery/reconciliation contracts remain a technical HOLD until the mechanism
satisfies them. No Build or independent readiness recommendation is justified while this remains.
Prior metadata/admission fixtures (four pending/family, absolute 120-second deadline, 60-second
cleanup, 64-family batches ×16/run, existing connection/timeouts and 1,500-admission stress case)
remain engineering regression inputs, not a proof of the withdrawn delivery scheme or approval
of new browser requirements. New delivery/cookie capacity contracts require the pending design.

## 5. Inherited R3-Rev7 planned scenarios — historical mechanism fixtures

R7-T is a historical submission prefix, not release-phase R5/Build authority.
CA-T01–16 in quickstart now govern functional acceptance, including conventional anonymous
ordering, boundary/known-stale checks and non-guaranteed notification. Retain R7 race challenges
only as historical stimuli, not old first-winner/atomic-render outcomes. R7-T08 stays paused.

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

Use real Chromium/Firefox/WebKit and supported devices, actual cookie headers, separate PostgreSQL
connections/barriers, DOM/accessibility/history paint and server revocation/CSRF traces without
secrets. All scenarios are planned; no runtime test or independent verification is claimed.

Primary sources: [Django CSRF](https://docs.djangoproject.com/en/5.2/ref/csrf/) documents login-time
secret rotation/token checks; [Django sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/)
documents server-backed keys. The continuity proposal is an author design, not an API guarantee.
