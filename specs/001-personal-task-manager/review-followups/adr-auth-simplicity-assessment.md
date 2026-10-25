# Proposed ADR-AS-001 — Django-first authentication simplicity assessment

**Date:** 2026-10-24. **Status:** Recommendation only; not adopted.
P1-A and further TD-008 remediation are PAUSED at the user's instruction. The earlier P1-A
question requires no answer during this assessment. R3 remains NOT_READY; TD-008 OPEN.
No requirements, operative design, review disposition or prior approval is changed.

**Authority:** [R3-Rev7 simplicity addendum](../../../independent-reviews/3_design-technical/Review_7/taskify_R3-Rev7_simplicity_reassessment_addendum.md),
[Rev7 review](../../../independent-reviews/3_design-technical/Review_7/taskify_R3-Rev7_technical_design_review.md),
Scope v1.1 FR-010/012/014 and human-approved R2-Rev5 §7, including R2-NAV-001.

## Recommendation

Use **Django database sessions plus small application guards** as the next candidate to assess.
Remove custom maintenance rotation from that proposal. Propose eliminating AuthorityFamily,
AuthorityHandle and PREPARE/ISSUE/ACTIVATE rather than maintaining an inner and outer session.

This is sufficient for the approved **ordinary** workflow; it is not demonstrated sufficient for
every overlapping-login/logout/private-display schedule below. Do not adopt the removal until
R2/human decisions D1–D3 and independent R3 assessment establish the exceptional boundary.
No particular anonymous winner is required by Scope/R2, but rare occurrence does not authorise
privacy exposure, inaccessible live credentials, view loss or misleading logout.
If strict cancellation of every pre-existing anonymous attempt is retained, additional ordering
may be needed; necessity of these particular bespoke tables/protocols has not been demonstrated.
P1 is not the default answer and stays paused.

## Minimal candidate and requirement trace

| Outcome | Django-first mechanism / necessary app logic |
|---|---|
| FR-010/015/016 private accounts, independent devices | Standard Django forms/authentication and PostgreSQL SessionStore; one Secure/HttpOnly/SameSite session cookie; existing support/no-recovery disclosure. Every repository scopes to authenticated owner; retain rate limiter, HTTPS, CSRF and no-store |
| R2-NAV-001 same-profile entry/login URL | LoginView authenticated redirect, fixed /tasks/ destination; root routes by current auth. Small adapter preserves approved GET 302 / authenticated POST 303 and CSRF 403; stock view returns 302 and accepts safe next URLs, so its flag alone is not the full wire contract |
| Shared current-session logout | CSRF-protected POST invokes Django logout/flush, acknowledge only after server effect; clear local private data, hint other tabs and conceal/revalidate on focus/pageshow before reveal. No user-account-wide revocation that would sign out independent devices |
| FR-012/013 / AC-005 view lifetime | Keep one dedicated SessionViewState record and atomic full-tuple save; never mutable view tuples in cached whole-session dictionaries. New login defaults, same valid login restores, missing/unreadable state silently defaults, no preference bootstrap |
| Stale page A submits using B cookie | Keep a random public login_epoch inside Django session and compare expected epoch before task/view writes; reject mismatch. Account ownership alone is insufficient for an old create payload that otherwise becomes B's new task. Not a bearer credential or authority registry |
| FD-001 / authority/CSRF failures | Preserve task-load Retry/support and all-write/retry lock, snapshot generations/revisions, current-token CSRF acquisition and bounded same-intended-epoch housekeeping/logout retry; never replay credentials/task writes automatically |

Django supplies authenticated identity, login key/CSRF rotation and logout flushing; it does not
erase another tab's already-painted DOM or guarantee final cookie order. Source inspection also
shows stock login can reuse a same-user session, which the approved authenticated-route redirect
makes irrelevant to the ordinary flow.
[Django auth](https://docs.djangoproject.com/en/5.2/topics/auth/default/),
[LoginView/LogoutView source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/views.py).

## What can be removed, and what cannot be inferred

| Existing custom item | Assessment for the proposed simpler candidate |
|---|---|
| AuthorityFamily / AuthorityHandle | Not required to implement normal sign-in, current-session logout or per-owner tasks; propose removal. Removing them also removes the current TD-001 family-fence proof, so independent regression assessment is required; historical resolved status is not overwritten |
| Transition table, generation claims, ISSUE/ACTIVATE/reconcile endpoints | Propose removing with the authority registry. A read-only framework-session identity/epoch check replaces the normal claim path. It does not prove cancellation of unseen anonymous sessions or eliminate S1/S2 |
| Periodic/custom maintenance key rotation and endpoints | Remove: no Scope/R2 requirement mandates it. Keep Django's own login fixation protection; do not equate removing an app rotation feature with disabling framework security behaviour |
| Separate session view and short persistence guard | Retain. View writes and the logout wrapper take Session-row → view-row order before updating/flushing, avoiding resurrection and inconsistent cascade lock order. No outer authority needed. Native valid-session rekey must preserve the tuple: assess a small transfer hook or stable login-epoch view key before adoption |
| Custom SessionStore/finalizer | Stock DB save uses force-update for an existing key and middleware raises SessionInterrupted on concurrent deletion; do not duplicate that as a new authority system. Assess only the narrow response-policy hardening in option B |
| Epoch/UI generations/owner checks/CSRF | Retain: these solve stale intent, identity response mixups and request security, not browser-wide login election |

Use SESSION_SAVE_EVERY_REQUEST=False and avoid ordinary app writes to auth-session dictionaries.
Django can still rekey on a configured secret-key fallback; view continuity and late rekey responses
must be checked if that deployment path is supported. Do not invent a periodic rekey policy.
[Sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/),
[DB backend](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/backends/db.py),
[auth rekey source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/__init__.py).

## Deterministic residuals and risk assessment

These are static counterexamples/planned fixtures, not observed incidents. Labels assess ordinary
likelihood qualitatively; there is no usage/attack dataset or numerical probability.

| Schedule | Result / impact / likelihood and exploitability |
|---|---|
| **S1: two anonymous logins.** Both credential POSTs pass CSRF before either response; persist S_A/S_B, delay B response; A becomes usable; then deliver B cookie | Final browser session can become B. Different-account overlap is low-likelihood ordinary usage; same-account duplicate forms are more plausible, still unmeasured. Same-account view reset/sign-in conflict is a reliability concern. With different accounts, old A DOM may remain while cookie authenticates B: potential confidentiality impact, not automatically accepted |
| **S2: late login after logout.** As S1, but flush S_A and show logout ACK before releasing B's response | S_A is invalid, yet live S_B can authenticate afterward. Repeat with both forms for A: the same account can regain access after logout. Low ordinary likelihood, potentially high shared-device privacy impact. This is a new session, not resurrection of the deleted DB row; current-session flush does not revoke it |
| **S3: delayed cookie deletion.** Hold an earlier logout/empty-session response; establish new S_A; release the old deletion header | New cookie is removed although S_A remains live. User may be forced to sign in and normal logout may lack S_A proof. Low, unmeasured overlap; material continuity/logout failure. Not a server owner-check bypass |
| **S4: failed refresh with stale private DOM.** Reveal A; late B login cookie arrives, notifications lost, next reads fail | Epoch guard can reject wrong-session writes, but cannot erase painted A rows. Focus/history gating protects a later revisit, not continuous certainty while already focused. Exposure duration is not bounded by polling when reads fail. Concealing on every ordinary refresh failure would change approved stale-row behaviour |
| **S5: framework rekey, if enabled.** Secret-key fallback cycles S_0 to S_1; hold response; logout using old captured S_0; release S_1 cookie | Native rekey can introduce a related late-session effect. Configuration-dependent, not a normal per-request event. Remove unrequested maintenance, but test/define supported framework/deployment rekey paths rather than claiming no rotation can occur |

S1/S2 require overlapping credential-bearing same-profile requests, not merely opening a normal
new tab after sign-in. Network/server delay widens the window. Cross-site attackers do not gain
this ability solely from the race: valid credentials and same-origin CSRF checks still apply.
It can occur accidentally; physical/same-origin control changes exploitability. Owner-scoped
server queries remain isolated; stale DOM, wrong-account **intent**, and unexpected post-logout
authentication are separate issues. Per-tab submit disabling cannot serialize other anonymous tabs.

[RFC 6265 §4.1.1](https://www.rfc-editor.org/rfc/rfc6265.html#section-4.1.1) warns of concurrent
Set-Cookie races. Cookie deletion is a response write too; Django's middleware emits it for an
empty session with an incoming session cookie.
[SessionMiddleware source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/middleware.py).
These event sequences are this assessment's inference from those mechanisms.

## Alternatives and human boundaries

| Option | Simplicity / coverage | Remaining condition |
|---|---|---|
| **A: framework defaults + required owner/UI/CSRF guards** | Lowest change; normal outcomes largely native; no authority tables/new storage | S1–S5 remain; stock response policy and route details are not the exact approved contract |
| **B: Django-first + thin adapters (recommended next candidate)** | A plus exact redirect/epoch/view guards and tightly scoped cookie-response policy. Avoid ordinary session saves; assess suppressing empty-session cookie deletion, relying on server flush, to eliminate S3. No new identity store or browser feature | Must explicitly review retained revoked-cookie hygiene/framework compatibility; must not suppress login fixation protection or fabricate saves. Still does not solve S1/S2/S4, nor automatically S5 |
| **C: strict shared pre-auth ordering** | Additional mechanism only if actual approved exceptional guarantees demand it; current bespoke design/P1 incomplete, higher cost | Not proved viable; P1/IndexedDB remains paused. Do not resume it or add availability restrictions through this ADR |

**D1 — concurrent initial sign-in outcome:** Does R2 permit response-order session selection and a
detected sign-in/view conflict for requests submitted while anonymous? No exact winner is currently
required; ordering alone is an engineering choice, not a new mandatory human gate. Refer actual
continuity/view-loss departures for targeted R2 decision. Acceptance of a reliability outcome
alone does **not** accept S2 or private-data exposure.

**D2 — logout boundary:** Must logout prevent every already-submitted anonymous login from later
authenticating, or only invalidate the Session identified by the logout request? Django provides
the latter. Adopting that narrower promise where R2 means shared browser sign-out requires explicit
targeted R2/human approval; it is not an already accepted FD-004 risk.

**D3 — rendered-data boundary:** Revisit concealment is mandatory. Whether identity changes after
successful focus validation require a stronger active-display guarantee must be explicitly settled.
No finite exposure window or stale-private-DOM risk is accepted here. Polling/broadcasts are not
proof of zero exposure. Changing ordinary task-failure stale rows needs R2 review.

Keep current approved outcomes unless changed by the proper targeted review/human decision.
If D2/D3 require the strict outcomes, **B cannot yet be certified sufficient**: return for narrowly
scoped technical assessment after the boundary decision, not wholesale adoption of P1. No
user-account-wide logout, new browser dependency or weakened owner checks is recommended.

## Evidence and gate

Next-review fixtures: AS-01 approved new-tab/redirect/view/expiry/independent-device matrix;
AS-02 S1 same and different accounts; AS-03 S2 both accounts plus old-cookie direct replay;
AS-04 S3 and option-B response policy; AS-05 S4 with dropped signals/failed reads/history/epoch
writes; AS-06 stale CSRF/logout retry and FD-001; AS-07 view-save/logout/native-key-change races;
AS-08 enabled framework rekey/deployment configuration. Observe actual cookies, server Session
rows, intended epoch, DOM/accessibility and final revocation; no imagined test passes.

Adoption requires targeted R2 decisions for actual departures, independent R3 proof of the
selected candidate and explicit human R3 approval. Existing TD dispositions/FD-003/004 acceptance
remain historical and unchanged. This assessment neither modifies the operative protocol nor
closes TD-008. See the [affected-artifact map](auth-simplicity-affected-artifacts.md).
