# Taskify — R3-Rev7 Independent Simplicity Reassessment (Addendum)

**Date:** 24 October 2026 (Australia/Melbourne)  
**Status:** Independent architectural reframing; **NOT** a new Codex candidate, amended Scope or R3 approval.  
**Prior record:** `taskify_R3-Rev7_technical_design_review.md` remains historically intact: TD-008 OPEN / Medium / blocking; human R3 gate OPEN. This addendum **supersedes only the earlier recommendation to consider P1-A Option B next**: first assess a simpler Django-native alternative.

## 1. Necessity and proportionality

Approved Scope v1.1 `FR-010` requires private accounts, cross-device access by the same account and isolation between accounts. Approved R2-Rev5 adds: same-profile tabs reuse the active login; authenticated login URLs preserve its session/view; account switching requires logout first; logout ends the browser-shared login in the other tabs; a subsequent fresh login has default view settings. There is no approved requirement for a bespoke authority-family registry, periodic login-key rotation or an exact browser-wide winner among two forms submitted **before either tab is authenticated**.

The residual R3-Rev7 TD-008 race is real but exceptional in Taskify's permitted UX: both tabs must already possess anonymous login forms and submit them before either login is recognised. Its ordinary-usage likelihood is low in qualitative terms, but **not measured**. Potential impact may be significant if an old cookie response reinstates access after logout or a tab displays stale private tasks under another active account. These are not automatically acceptable just because the race is rare. Conversely, demanding arbitrary response-order continuity as a new absolute product invariant without human decision has driven repeated custom mechanisms disproportionate to a personal task manager.

## 2. Established Django baseline to evaluate first

Django 5.2 documents a mature server-side session model:

- `django.contrib.auth.login` saves authenticated identity in the session, rotates the session key on login to mitigate fixation and rotates CSRF. **No periodic custom maintenance rotation is mandated by Taskify Scope/R2.**
- `django.contrib.auth.logout` flushes the request's server-backed session; all tabs using the same valid browser session cookie subsequently fail authentication. It does **not** by itself instantly erase a previously painted screen in another tab.
- Server-backed PostgreSQL sessions, a single Secure/HttpOnly/SameSite session cookie, server-side owner checks and guarded login routes map naturally to Taskify's normal outcomes. A view-state row keyed to the valid session (or equivalent reviewed per-session state) can maintain approved filter/sort scope independently of account data; assess middleware concurrency without casually storing mutable view tuples in the entire Django session object.
- For UX immediacy, clear/conceal private DOM and drafts on successful logout/sign-in notification, use `BroadcastChannel` only as a prompt, revalidate on focus/`pageshow`/before private reveal, and fail closed on uncertain authority. Maintain FD-001 task-load/write protections, anti-CSRF, no-store and server ownership. An ignored or lost broadcast must not be sole authority.

This is a **candidate architecture for Codex to demonstrate**, not independently verified correctness for all race schedules. Standard Django does **not** guarantee browser cookie response ordering, instant cross-tab DOM clearing or that two simultaneous initial anonymous logins cannot race. HTTP RFC 6265 explicitly warns that concurrent `Set-Cookie` responses can cause unpredictable final cookie selection. Security review must distinguish **server-side owner isolation**, **visibility of already-rendered private data**, **current-cookie session invalidation**, and **possible late response after logout**.

## 3. Complexity comparison

| Concern | Django-first candidate | R3-Rev7 bespoke candidate / proposed P1 |
|---|---|---|
| Normal login and independent devices | Framework session cookie and server-backed sessions | AuthorityFamily, AuthorityHandle and separate inner Django Session |
| Cross-tab use / logout | Shared cookie, `logout()` invalidation, UI broadcast and focused revalidation | Family generation/claim/transition machinery; authoritative family revocation |
| Session-scoped view | Dedicated state by server session; retain AM-001 consistency/fallback | SessionViewState plus family/rotation transfers |
| Key rotation | Framework on login; avoid unrequested periodic rotation | Bespoke maintenance rotation and locking/transfer |
| Exceptional two-anonymous-submit ordering | Browser cookie outcome may depend on response order; investigate exact residual and product acceptability | Strict ordering sought via custom pre-auth context, candidates, cookie-selection rules; proposed IndexedDB enrolment requirement |
| New-browser availability | Cookies/normal framework dependencies | P1 would additionally require IndexedDB enrolment for first login |

No claim is made that the Django-first baseline fully satisfies the strict adversarial outcome without clarification or extra mechanism. That uncertainty is exactly what the next assessment should isolate, rather than embedding P1 by default.

## 4. Revised independent direction

**P1-A is premature.** Do not approve IndexedDB, request an R2 amendment solely to make room for it, or adopt a forced-sign-in/ineffective-logout fallback yet.

Ask Codex for a **non-implementation architecture simplification assessment** that:

1. Maps every approved normal and exceptional login/logout/view/privacy outcome to a minimal Django-native design; lists bespoke tables, locks, rotation endpoints and transitions that can be removed or retained with necessity evidence.
2. Uses official Django and cookie specifications, not generic claims that "all apps do this". Includes a deterministic two-anonymous-login, late-login-response-after-logout and stale-private-DOM sequence against the simpler proposal, distinguishing privacy breaches from recoverable sign-in conflicts.
3. Evaluates likelihood, user-visible effect, exploitability and residual reliability of those sequences without fabricating numeric probabilities. Reassesses whether they require a custom coordination protocol under *actual* Scope/R2 authority.
4. Presents the smallest viable alternative(s), precise departures (if any) from approved R2 behavior, cost/complexity, and regression/test obligations. If a material user-facing guarantee would need relaxing, submit a **narrow R2/human choice first**. No reviewer or Codex may silently accept privacy exposure, false logout or account-data access.
5. Produces a short recommendation/ADR and affected-artifact map. Preserve prior reports and approved baselines; do **not** alter the operative design, run Tasks/Analyze/Build or treat this assessment as R3 approval.

The independent R3 reviewer then judges the **simplified candidate** against Scope, approved R2, security, privacy and reliability, before accepting or rejecting P1 as necessary. Any actual Scope/Functional Design amendment uses targeted review and human approval, followed by updated Plan/Checklist and independent R3 re-review.

**Verdict remains: R3 NOT READY; TD-008 OPEN; no downstream gate authorised.**

## Primary sources

- [Django authentication: `login()`/`logout()`](https://docs.djangoproject.com/en/5.2/topics/auth/default/), and [Django 5.2 authentication source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/__init__.py).
- [Django server sessions, `flush()`, `cycle_key()` and cookie save behaviour](https://docs.djangoproject.com/en/5.2/topics/http/sessions/).
- [Django CSRF](https://docs.djangoproject.com/en/5.2/ref/csrf/).
- [RFC 6265 §4.1.1: concurrent `Set-Cookie` response race](https://datatracker.ietf.org/doc/html/rfc6265).
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html); [MDN Broadcast Channel](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API); [Chrome bfcache restore guidance](https://developer.chrome.com/docs/web-platform/bfcache-ccns).
