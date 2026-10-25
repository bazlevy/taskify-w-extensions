# R2-Rev6 — proposed decisions and targeted Functional Design delta

**Prepared:** 2026-10-24. **Status:** PROPOSED; for independent R2-Rev6 review.
Scope v1.1 and R2-Rev5 remain the approved baselines. This is not an amendment incorporated into
either baseline, an independent verdict, or a new phase approval. R3 NOT_READY; TD-008 OPEN.
P1/IndexedDB remains paused. No application implementation or Tasks/Analyze/Build.

## 1. Decision authority and boundaries

Authority is the current user instruction reporting Barry's confirmation of the following exact
decisions. It supplies human policy direction for this proposal, not independent R2 approval or
a substitute for the subsequent R2 gate. No new signature, time of decision or historical approval
is inferred. Barry Levy is the human decision owner; Codex prepares the proposed design/evidence.

| ID | Confirmed decision, verbatim | Proposed bounded interpretation |
|---|---|---|
| S2-DEC-001 | A later valid login response may authenticate the browser after an earlier logout. The revoked session must remain invalid. | Logout revokes the intended current shared login. It need not cancel every already-submitted anonymous login. A distinct valid login delivered later can become current, including for the same account; this does not resurrect the revoked login. |
| S4-DEC-001 | Instantaneous cross-tab UI clearing is not required. Revalidate authentication before protected actions and prevent stale cross-account access or mutations. | A previously authorised display can temporarily remain in another tab until invalidation is observed or a required gate runs. It grants no permission to use old-account data for protected actions under a new login. Mandatory action and lifecycle gates remain; no stale-access waiver or polling-based privacy proof. |

Scope authority: [spec.md](../spec.md), FR-008/009/010/012/013/014/015 and AC-005/008/009/010.
Functional authority: [R2-Rev5 review §7](../../../independent-reviews/2_design-functional/Review_5/Taskify_R2-Rev5_Targeted_Functional_Design_Review.md#7-post-review-formal-human-r2-rev5-decision--2026-10-24),
including R2-NAV-001; earlier R2-Rev4/Rev3 decisions remain historical. Current canonical design
files contain later, unapproved R3 corrections: they are edit targets, not byte-identical copies
of the approved R2-Rev5 candidate. The historical approved manifest remains untouched.

## 2. Precise baseline comparison

| Area | Approved behaviour / written baseline | Proposed change and Scope assessment |
|---|---|---|
| Logout and late logins | R2-Rev5 §3/§5 requires shared revocation and no silent account replacement; later technical contracts additionally cancel pending transitions. Earlier Option B S2 was an unaccepted blocker. | S2-DEC-001 explicitly permits a distinct late valid login after logout. This is an actual targeted R2 change to the exceptional outcome, not merely a renamed technical fix. FR-010 does not specify cancellation of anonymous login attempts; revoked credentials and owner isolation remain mandatory. No Scope edit proposed. |
| Cross-tab private display | R2-Rev5 retains clear/conceal on invalidation and before revisit, with stale replies unable to reveal another account. The earlier Option B assessment treated already-focused stale DOM as an unaccepted S4 gap. | S4-DEC-001 removes any instantaneous cross-tab clearing requirement while retaining gates and access isolation. Previously painted passive content is distinguished from new protected access. FR-010/AC-010 remain binding: R2 must explicitly assess this distinction against “neither can view or modify the other's personal tasks.” If the proposed boundary requires an exception to Scope, refer to targeted R1/human review before incorporation; this package does not authorise that exception. |
| View lifetime | FR-012/013/AC-005: valid login retains its view; fresh login defaults; distinct sessions independent. | No change. A distinct late fresh login starts defaults even for the same account. An authenticated login-page visit/submission still redirects and retains the current login/view under R2-NAV-001. |
| Saved tasks and failures | FR-008/009/014: account persistence, errors/Retry/support and protected uncertain writes. | No change. Revalidating identity never unlocks task writes without the required valid saved-task snapshot; no cross-account replay, reset or new recovery policy. |

R2-Rev5 approval is not reopened generally. Only the two exceptional boundaries and their
dependent tests are submitted. S2/S4 do not approve Option B, its middleware changes, removal of
technical fences, P1, native rekey ordering or any other R3 mechanism.

## 3. Minimum proposed replacement/additional contract text

These clauses are the proposed Functional Design revision set. Apply only after required review
and human gate decisions; do not copy them into the operative contracts during this preparation.
Technical family/handle/claim terminology is intentionally not a functional requirement here.

### R2-6-UI-01 — contracts/ui.md: navigation and logout

> Logout from any tab revokes the intended login shared by tabs using that session. Show logout
> success only after that revocation commits. Every subsequent request using the revoked
> credential fails authentication; independent browser/device sessions remain valid. Clear this
> tab's private content, drafts and queues on logout intent and notify other tabs as a prompt.
> A distinct valid login response already in flight may subsequently authenticate this browser,
> including as the same account. That is a new login, not reversal of the completed logout.
> Do not promise that logout cancels all pending login responses or guarantees the browser will
> remain signed out. Normal account switching still starts with logout; submitting replacement
> credentials when the request is authenticated redirects without evaluating those credentials.

The task screen continues to identify the current account. If a later login is discovered,
navigate/revalidate using it and load its own task list and fresh-login defaults. Discard old
account data/drafts/queues; never migrate them. Do not automatically log out that new login to
fulfil an earlier intent. A lost/failed logout response remains unconfirmed; a 401, absent cookie
or changed identity is not its commit acknowledgement. Existing Retry/support remains.

### R2-6-UI-02 — contracts/ui.md: protected actions and reveal gates

> Other tabs need not clear simultaneously with a logout or login elsewhere. On observed
> invalidation/account-or-epoch mismatch, clear/conceal old private state immediately. Retain
> concealment before initial load, focus/visibility/history restoration and private re-reveal.
> Revalidate authentication before a protected action; carry the expected login identity through
> the action and reject a changed, expired or revoked login before protected access or effects.
> Never treat a previously rendered task, cached account label, broadcast or CSRF token as
> current authority. Clear old account state on mismatch and load only the validated account.

**Proposed protected-action inventory for R2 review (Q1):** task-data reads/reveals and refreshes;
opening private task details/edit/delete dialogs; create/edit/complete/reopen/delete submission
and explicit retry; filter/sort actions exposing a new task view; view remembering; logout.
Local Cancel and access to public support do not authorise private access. Editing within an
already-open authorised draft does not require a server round trip per keystroke; its eventual
submission is a mandatory gate. This inventory is proposed, not an already confirmed expansion
of S4-DEC-001.

Server requests always authenticate/validate owner and expected login at their service boundary.
Interactive access to cached private data additionally needs UI revalidation before proceeding;
a successful check must not let a subsequent request silently run under another login. Responses
retain identity/epoch/query/revision/generation checks. No atomic browser-cookie/paint guarantee
is inferred from a prior check; the pending-response boundary is explicitly Q2 below.

Auth lookup failure at a required gate: conceal and do not proceed; show Retry/support, never
infer anonymity or a successful empty list. Preserve ordinary focused task-refresh failure:
previously authorised rows may remain visibly stale, but all task writes/retries stay locked
under FD-001. Such rows are not permission for a new protected action. Isolated view remembering
failure remains silent and does not block otherwise available task operations. No new view-error
banner, continuous mandatory polling or instantaneous cookie-change detection is required.

### R2-6-HTTP-01 — contracts/http.md: observable auth preconditions

> Logout success certifies revocation of the intended shared login, not cancellation of every
> distinct pending login. Replaying the revoked credential cannot authenticate, create a session
> or restore its view. A distinct valid late login may authenticate only as its own verified
> account and fresh epoch; private responses/actions remain owner-scoped.
> Keep authenticated login GET 302 /tasks/, valid-CSRF POST 303 /tasks/ before replacement
> credential evaluation, invalid CSRF 403, and lookup uncertainty 503. The late-response allowance
> is not permission to process replacement credentials on an authenticated request.
> Private API calls require live authentication and the intended login epoch. Missing/stale
> epoch or invalid/revoked login returns existing 401 authentication_required with no protected
> task/view effect; storage uncertainty returns 503. Do not flush a different current login,
> reinterpret old payloads as its data, or automatically replay passwords/task mutations.

Retain CSRF/Origin checks and bounded fresh-token recovery for the same intended logout login.
Token refresh does not prove identity. No response error implies rollback unless the trusted
contract establishes rejection. Validly committed pre-logout task effects remain saved; logout
cannot acknowledge while a command using that revoked authority can still commit afterward.
Precise transaction ordering and cookie response handling remain R3 obligations.

### R2-6-STATE-01 — workflows, states and data-model semantics

| Event / state | Required proposed functional transition |
|---|---|
| Logout intent | Conceal/clear local private state; capture intended login; pending confirmation |
| Confirmed revocation | That login stays invalid; signed-out destination is valid then, not a guarantee against future distinct login delivery |
| Distinct late login becomes current | Treat as its own fresh login/defaults, never a revival or view/task transfer from the revoked login |
| Observed identity change / action-gate mismatch | Clear old rows, labels, drafts, dialogs and task/view queues; reject old actions; fresh validated load for current identity |
| Required auth gate unavailable | Concealed Retry/support; no protected action or false logout success |
| Passive tab without notification | No instantaneous clearing guarantee; next mandatory action/lifecycle gate must run before protected access |
| Known stale/invalidated reply | Discard; never re-reveal or apply old-account data |

In data-model.md retain the logical separation of account-owned tasks/receipts and login-owned
view state. Revocation cannot revive the old login/view; distinct fresh login defaults. No task
schema, view tuple, receipt/version semantics or physical SessionViewState key change is proposed
by R2-Rev6. Option B's epoch-based physical key and locking are separate unapproved R3 proposals.
Research/plan receive only these functional boundaries, decision IDs and review status after
approval; technical structures are not removed by this package.

## 4. Material questions isolated for targeted review

| ID | Remaining question / proposed treatment | Owner and resolution gate |
|---|---|---|
| R2-6-Q1 | Does “protected actions” include local task-detail/edit/delete-dialog and filter/sort interactions with cached data? Proposed inventory above includes them, without per-keystroke checks. Confirm or correct it; do not infer server-only checking is sufficient. | Independent R2 reviewer assesses completeness; Barry decides any unresolved material boundary before R2-Rev6 approval/incorporation. |
| R2-6-Q2 | A private read was authorised under A, but its response completes after an unobserved B-cookie delivery. S4 confirms no instantaneous clearing of already-painted A, not explicitly whether a newly arriving A payload may be revealed then. Retain rejection on known mismatch; do not mark this unobserved late-reveal schedule accepted. R2 must distinguish existing passive display from new protected reveal, with R3 identifying what can be proven at the service/reveal boundary. | Independent R2 assessment plus R3 feasibility evidence; Barry only if an actual additional product boundary needs choosing. Unanswered newly stale reveal is not a passed scenario. |

R2 must also explicitly record its Scope-consistency assessment of S4 against FR-010/AC-010.
No particular winner among two genuinely anonymous submissions needs a human decision: natural
response-order selection remains an engineering choice, tested in T12. That is distinct from
processing replacement credentials on an authenticated request, which remains prohibited.
Only a demonstrated additional view, privacy or navigation departure would require escalation;
the mere existence of two valid initial logins is not a new D1 approval gate.
No general demand to reapprove S2/S4 direction or all former D1–D3 questions is made. Only the
bounded questions above, any demonstrated Scope departure and the formal revised-design gate
remain. Keep stricter unresolved outcomes as review gaps until decided, not hidden risk acceptance.

The [review evidence/change map](r2-rev6-targeted-review-evidence.md) supplies proposed acceptance
scenarios and preservation checks. No finding is independently verified/closed by this proposal.
