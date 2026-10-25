# ADR-AS-003 — R3 Option A availability assessment

**Date:** 2026-10-24. **Status:** assessment and escalation; no new architecture adopted.
Scope/R1-Rev4, R2-Rev5 and R2-Rev6 remain APPROVED. R3 NOT_READY; TD-008/010/011 OPEN.
P1/IndexedDB paused. No Tasks, Analyze, Build, implementation or runtime tests.

## Outcome and controlling direction

No complete, proportionate design satisfying all three approved guarantees has been established
within the existing availability boundary. Stop expanding the authentication state machine here;
do not present ordinary Django plus polling as a correction of TD-008/010. This is a bounded
engineering assessment, **not a proof that every possible cookie/browser protocol is impossible**.
TD-011 has a concrete notification protocol below, but its safe reveal fallback depends on the
still-unproved TD-010 mechanism; no independent closure is claimed.

Barry's latest direction selects **Option A: preserve existing availability**, with no mandatory
Web Locks, IndexedDB or other browser capability. It supersedes the pending asynchronous Web
Locks question; no answer to that earlier question is needed. The earlier rejection is also
recorded in [the Rev4 evidence history](r3-rev4-correction-evidence.md#human-direction--rejected-option).
Security/privacy/logout guarantees remain unchanged. A dependency or functional relaxation needs
its own specific human decision and applicable review before incorporation.

Authority: [R3-Rev8 report §§4–7](../../../independent-reviews/3_design-technical/Review_8/taskify_R3-Rev8_technical_design_review.md),
[R2-Rev6 approval §§2–4](../../../independent-reviews/2_design-functional/Review_6/Taskify_R2-Rev6_Targeted_Functional_Design_Review_APPROVED.md),
operative [UI](../contracts/ui.md)/[HTTP](../contracts/http.md) contracts and Scope FR-010/AC-010.
The current native-Django assessments and inherited family protocol are unapproved alternatives,
not evidence that either meets the newly approved guarantees.

## 1. Start with the established Django baseline

Retain Django credential checking, CSRF, session fixation protection, PostgreSQL SessionStore,
Secure/HttpOnly session cookies and owner-scoped task repositories. Django login writes session
identity and rotates the key; logout flushes the identified session. Those primitives do not
elect a first browser-established winner among independent cookie responses.
[Django authentication](https://docs.djangoproject.com/en/5.2/topics/auth/default/),
[sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/).

Dedicated login-scoped view state, intended-epoch guards, receipts/versions, short task/logout
transaction guards and concealed lifecycle states remain useful. They protect task data and
stale intent; they are not a browser cookie compare-and-set or a render fence. No new cookie
response modification is selected by this assessment. Removing periodic custom rotation is a
reasonable simplification candidate, but does not itself solve concurrent initial logins.

## 2. TD-008 — the specific ordering incompatibility

With two genuine anonymous submissions, native Django can create valid independent sessions A
and B. Their shared cookie-name responses are applied by the browser independently of server
commit order. A can become usable first; delayed B then replaces its cookie while A remains
active. That violates FD-007 even if both accounts are the same or a later script displays an
error. With a different account it can also change account/view and impair intended logout.

HTTP has no conditional Set-Cookie operation meaning “apply only if the browser still has no
established login.” JavaScript receives the response after cookie handling and cannot remove
Set-Cookie from a received response. Those are the relevant protocol limitations, not a claim
that all server coordination is impossible.
[RFC 6265 §4.1.1](https://www.rfc-editor.org/rfc/rfc6265.html#section-4.1.1),
[Fetch cookies and forbidden response headers](https://fetch.spec.whatwg.org/).

| Proposed shortcut | Why it does not establish the approved result |
|---|---|
| Server transaction / account lock | Orders commits, not application of already-sent response headers. Account-wide exclusion would also affect independent sessions. |
| Reject authenticated login requests | Required by R2-NAV-001, but both captured requests can have been anonymous. A conflict/error with a competing auth cookie still fails. |
| Single shared anonymous-session/bootstrap cookie | Its own first creation can race: root X response is held; root Y/login A establishes; late X replaces the proof. Safely initialising a common browser context is itself the missing coordination problem. |
| Unique candidate cookie names / cookie-list ordering | Avoids direct same-name overwrite but needs an authoritative selected-login rule under omitted/removed cookies, bounded cleanup and delayed headers. Those are the selective-proof/continuity problems already reviewed; do not restore them as solved. |
| BroadcastChannel / per-tab submit disabling | Does not exclude an anonymous tab that missed/delayed a message. Per-tab blocking is not shared exclusion. |

S2 does not remove this incompatibility: it permits a distinct valid login **after** original
revocation, not replacement while the winner remains active. Nor may a delayed competitor
replace C after C has subsequently become established. Cookie cancellation/abort must be proved,
not assumed to roll back a server login or all cookie effects.

## 3. TD-010 — a fresh check is not a validation-to-reveal fence

An A-private payload arrives and is held. A fresh identity check validates A on the server.
Before its reply is consumed, another tab successfully logs out A and establishes B; notifications
are lost. The delayed identity reply still says A and matches the old tab's payload/generation.
Revealing A now fails FD-006. This trace uses approved logout-first switching; it does not assume
an FD-007-violating A→B replacement.

Repeating the probe merely moves the final unchecked interval. Local generations cannot detect an
unobserved cross-tab change; broadcasts are not atomic with cookie processing. Server Session
locks protect reads/writes but do not span the browser's eventual DOM/accessibility reveal.
This is reasoning from the response-order model, not an observed incident.

A browser-wide shared/exclusive fence covering auth changes and the final fresh-check/reveal can
address that particular interleaving. Without such a fence, a custom server reveal-grant protocol
would need expiry, revocation, crash/suspension and stale-client enforcement proofs. Waiting for
every tab to acknowledge conflicts with FD-008; a timeout alone does not prevent a resumed stale
client from revealing. Rebuilding such machinery is disproportionate without a demonstrated
end-to-end proof. Do not continue adding registries/leases to label this solved.

The distinction is important: S4 permits asynchronous clearing of **already revealed** data;
FD-006 still gates **newly arrived** data. Ordinary task-refresh failure must retain approved
stale rows/error and FD-001 locks; concealing every ordinary task failure would be another
functional change, not a solution.

## 4. TD-011 — concrete established notification protocol

Use BroadcastChannel opportunistically at application-controller startup, before private reveal,
on a fixed same-origin channel `taskify.auth.v1`. No durable browser state or credentials are
stored. Initial construction/posting failure must not be interpreted as failed server revocation
or as permission to bypass concealment. Do not add a mandatory messaging capability.

After authoritative logout ACK for intended epoch L, clear the initiating tab locally and post
one versioned message `{schema:1, kind:"logout_committed", login_epoch:L, event_id:<random UUID>}`.
There is no account ID, task text, credential or signed logout proof in the message. Dispatch
promptly in the ACK handler; no future poll and no wait for other-tab ACKs. A 401/403, cookie
absence or lost/malformed logout reply does not trigger a “confirmed” broadcast.

Receiving runnable tabs validate message shape and epoch. If rendered/pending state belongs to
L, synchronously conceal/clear its rows, labels, drafts, dialogs and task/view queues, invalidate
local generations, then navigate to the login URL. Duplicate messages are idempotent; a bounded
in-memory event cache is optional for redundant navigation only, never authentication authority.
If the tab now represents N≠L, do not revoke N, clear N's remembered view or replay L's logout.
Any login navigation while N is current obeys R2-NAV-001.

On pagehide/hidden/blur conceal before suspension; on resume/focus/pageshow/history return stay
concealed until current authority and matching private data validate. Missed, late or unsupported
channel delivery never releases that gate. Retain normal task polling as an additional discovery
path, not as proof of prompt message delivery or atomic reveal. All protected actions and new
payloads still require the approved gates. There is no privacy-safe native fallback for TD-010
established by merely specifying these hooks; the integrated design remains blocked.

[HTML BroadcastChannel](https://html.spec.whatwg.org/multipage/web-messaging.html#broadcasting-to-other-browsing-contexts)
provides same-storage-partition messaging, not durable guaranteed delivery or a lock. If a reviewer
requires a guaranteed prompt transport when that channel is unavailable, record that additional
transport/capacity obligation explicitly; do not silently introduce storage, SSE infrastructure or
faster polling into the previously reviewed capacity design.

## 5. Simplest alternatives and exact decisions

| Alternative | Mechanism and feasibility boundary | Approval consequence |
|---|---|---|
| A / current decision | Native Django plus owner/epoch/receipt guards, lifecycle checks and the notification protocol above. No complete proportionate mechanism for TD-008/010 established. | Preserves availability direction; cannot recommend R3 readiness. No new permission requested merely because the work is difficult. |
| Mandatory Web Locks candidate | Same-origin exclusive coordination around **all** auth-cookie-writing requests and shared coordination through fresh after-arrival validation plus synchronous DOM reveal. Native SessionStore remains; remove unnecessary outer authority machinery only after complete proof. | Changes the earlier availability boundary: unavailable/denied coordination or a stalled owner can block login/logout/reveal. Requires a specific human availability decision and targeted R2/Scope assessment if applicable, then independent R3 proof. **Not adopted.** |
| Optional Web Locks | Use the fence where supported; ordinary-native fallback elsewhere. | Does not satisfy unchanged guarantees on the fallback path. Viable only with another complete proven fallback, or an explicitly approved weaker functional guarantee for that path. Not a free availability-preserving fix. |
| Native-only functional amendment | Accept natural response-order login selection and define private access at a server-validation point rather than guarantee identity through final reveal. | Would change FD-007 and FD-006 and may weaken privacy/account isolation. Requires Barry's explicit targeted R2 decision and R1 review if FR-010/AC-010 changes. No such amendment or risk acceptance is proposed for incorporation. |

The mandatory-fence alternative is the simplest established **candidate** identified for the two
ordering gaps, not a completed failure-proof design. It must still specify anonymous bootstrap,
duplicate/stale HTTP admission, no ordinary response cookie writes, native fixation, cancellation/
document destruction, key/view lifetime, CSRF, task/logout ordering and held-lock recovery.
Do not promise that adding one API call closes the findings. In particular, forcing a lock release
or falling back after timeout is unsafe without fencing the old operation.

[Web Locks](https://www.w3.org/TR/web-locks/) supplies shared/exclusive coordination within a
storage bucket, conditional acquisition and lifetime rules. It does not prove that the API is
available in every currently admitted environment, that acquisition has bounded wall time, or
that server side effects roll back on termination. A modern browser support list alone is not
approval of a new fail-closed availability condition.

## Recommendation / stop condition

Honour Option A and stop further architectural expansion. Keep all approved guarantees and
technical blockers open; do not incorporate a dependency, restore P1 or widen accepted risks.
Return the concrete incompatibilities and conditional alternatives for review. There is no new
human decision silently assumed here: changing the selected availability boundary or either
functional guarantee needs a **new explicit** direction, not another answer to the closed question.
See [targeted scenarios and handoff](r3-rev9-option-a-review-handoff.md).
