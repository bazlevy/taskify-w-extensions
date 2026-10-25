# Targeted R2 amendment proposal — conventional authentication behaviour

**Prepared:** 2026-10-25. **Status:** PROPOSED; independent R2 review and revised-design human
approval pending. No amendment is incorporated into operative contracts or Scope.
Scope v1.1/R1-Rev4, R2-Rev5 and R2-Rev6 remain the approved historical baselines.
R3 NOT_READY; TD-008/010/011 OPEN. P1/IndexedDB remains paused. No Tasks, Analyze or Build.

## 1. Authority, purpose and decision record

Authority is Barry's human direction in the current user request, not a reviewer recommendation.
The request authorises this targeted proposal and explicitly accepts the narrow display risk
below; it is not independent verification or formal approval of this exact revised design.
Preserve [R2-Rev6 approval §§1–4](../../../independent-reviews/2_design-functional/Review_6/Taskify_R2-Rev6_Targeted_Functional_Design_Review_APPROVED.md)
and [R2-NAV-001 approval](../../../independent-reviews/2_design-functional/Review_5/Taskify_R2-Rev5_Targeted_Functional_Design_Review.md)
against their exact baselines. This proposal supersedes only the identified clauses if approved.

Apply [the remediation policy](../../../independent-reviews/review-remediation-policy.md): the
exceptional browser-wide guarantees have already driven multiple substantial remediation cycles.
Pause further custom remediation of those disputed guarantees while their amendment is reviewed.
That does not waive current approved obligations, change finding statuses or open a development gate.

| Proposal reference | Human direction recorded | Approval boundary |
|---|---|---|
| CA-007 / FD-007 | Adopt conventional Django/browser session-cookie response ordering for overlapping anonymous login requests. Remove first-browser-established-winner/custom coordination obligations. Preserve R2-NAV-001, including stale forms. | Direction confirmed; exact replacement contract below still needs review and human revised-design approval. |
| CA-006 / FD-006 | Retain server authentication, ownership/intended-login guards, known-stale rejection, observed-change clearing and appropriate lifecycle/uncertainty revalidation. Remove atomic validation/rendering and unnecessary per-interaction round trips. | Direction confirmed; no cross-account API access, mutation or old-intent rebinding is permitted. |
| RA-FD006-001 | Barry accepts that previously authorised content may temporarily remain visible during an unobserved cross-tab authentication change. | Human risk acceptance is recorded here, within §3's narrow bounds; Scope consistency and this exact revised contract remain review items. Not an independent risk verdict or a general privacy waiver. |
| CA-008 / FD-008 | Retain intended shared logout, prompt notification, receiving-tab clearing/navigation and uncertain recovery. No guaranteed delivery or atomic clearing. | Consequential alignment only; no new transport, browser dependency, timing SLA or account-wide logout. |

## 2. Proposed FD-007 replacement — UI/HTTP login behaviour

Authenticated same-profile root/new-tab entry reaches the existing task list. Login GET returns
302 /tasks/; a valid-CSRF authenticated login POST returns 303 /tasks/ before replacement
credential evaluation, with the same account, epoch and remembered view. Invalid CSRF remains
403; authority lookup uncertainty remains a technical error, not presumed anonymity. A stale
login form revalidates on restoration/revisit and before credential submission; if authenticated,
discard its credential fields and navigate to the existing tasks. Do not replay passwords.

For genuine overlapping requests submitted while anonymous, ordinary session-cookie response
ordering is permitted. Both valid submissions may create distinct logins. The browser's actual
current valid cookie determines the account/login used by subsequent requests. A later response
may select a different login, including the same account with another epoch, without an
intervening logout. There is no first-submission, first-server-commit or first-browser-winner
promise and no required competitor 409 solely to enforce such a winner.

This exceptional in-flight case is distinct from accepting replacement credentials on an
authenticated request: R2-NAV-001 continues to prohibit the latter. Normal deliberate account
switching requires logout first. Do not silently keep a universal “no replacement without
logout” sentence that contradicts the explicitly allowed anonymous-response race.

Every actual fresh login starts its own default view; valid-login navigation preserves its
remembered tuple. When a changed selection is observed, clear previous-login client state and
load only the selected account/login's view. Never transfer old drafts/queues/settings to the new
login or treat cookie response order as account-wide preference precedence.

S2-DEC-001 remains: a distinct valid late login may authenticate after confirmed revocation of
another login; the revoked login stays invalid. No promise is made to cancel/revoke every other
valid session created by an anonymous overlap. Invalid credentials do not authenticate. This
amendment does not authorise revoked-session resurrection, arbitrary cookie fallback, a new
credential-recovery feature or errors being presented as successful login.

## 3. Proposed FD-006 replacement — protected access and response safety

Every protected server operation authenticates the presented login, enforces account ownership
and validates its intended-login guard before data access/effect. Keep existing API 401 for
invalid/revoked authentication or intended-epoch mismatch, 403 for CSRF, and technical failure
for lookup uncertainty. Caller-supplied account IDs are never authority. An A intent delivered
with B's cookie, including same-account new epoch, must not become B's operation. Keep existing
task version, receipt, revision and unknown-outcome protections; no automatic credential/task replay.

The UI rejects responses **known** to be stale, mismatched or invalidated against its known
account/epoch, request/query generation and revision. Before applying a reply, perform these
local checks. On an observed auth change, expiry/revocation, trusted auth failure or identity
mismatch, clear/conceal old rows, account labels, drafts, dialogs and pending task/view queues;
discard old responses and load only the validated current account/login.

Revalidate at appropriate boundaries: initial protected entry; focus/visibility/resume/history
restoration while concealed; observed auth-change signals; and recovery when authentication is
uncertain, including stale-form/CSRF recovery. A failed required authority check remains
concealed with Retry/support. A successful ordinary protected request supplies authenticated
server evidence; a separate auth round trip is not required for each local interaction, each
private response, each keystroke or every normal refresh solely to prove atomic rendering.
R3 chooses proportionate request/response checks within these outcomes, not new R2 architecture.

The previous Q1 list still identifies sensitive server operations and cached private UI contexts.
Its universal fresh-authentication-before-each-local-action interpretation is superseded:
opening a cached detail/edit/delete dialog, changing filters or typing within the current known
login need not trigger an independent authentication request. Server submissions remain guarded;
an observed change/uncertainty or lifecycle boundary still triggers clearing/revalidation.

There is no browser-wide atomic check-to-render guarantee, continuous cookie observation,
mandatory browser lock, cross-tab acknowledgement fence or guaranteed zero interval between
auth change and UI observation. The following residual is expressly disclosed for review:

- **Accepted by Barry:** previously authorised content may temporarily remain visible while a
  cross-tab change is unobserved. **Proposed operational reading for review:** existing UI and a
  response authorised under the prior login that passes currently known stale-response checks
  may complete rendering before the change is observed. This in-flight consequence of removing
  atomic rendering is disclosed explicitly for approval of the written amendment, not claimed
  as a separate already-approved risk extension or a cross-account server grant.
- **Excluded:** new API access through another account, unguarded mutations, old-intent rebinding,
  intentional display of known mismatched/invalidated replies, retention after an observed change
  contrary to clearing rules, or a revoked session becoming valid again.
- **Mitigation/recovery:** server guards, local stale-response checks, current account display,
  prompt notifications and lifecycle/uncertainty revalidation. No numerical exposure duration is
  promised; loss/delay of notification or inactivity can delay observation. This matters on a
  shared screen/device and can have serious confidentiality consequences; no measured frequency
  or claim of harmlessness is made.
- **Owner/provenance:** Barry Levy, current human instruction, recorded 2026-10-25. This new
  bounded acceptance does not expand historical FD-003/004 risk records or approve a technical
  mechanism. Independent review must examine the stated in-flight boundary and Scope alignment.

Ordinary same-login task-refresh failure still retains visibly stale rows/error and locks every
task mutation/retry until a valid saved-task snapshot loads (FD-001). Do not relabel every task
network error as auth uncertainty to hide rows. Isolated view read/save failure remains silent
and must not block otherwise available tasks. Auth success alone does not settle an unknown
operation or unlock a task-load failure.

## 4. Consequential FD-008 alignment — no broader change

Logout captures the intended shared login, clears/conceals the initiating tab and reports success
only after confirmed server revocation. Subsequent use of that revoked credential cannot
authenticate. Other browser/device sessions remain unaffected. A 401/403, missing cookie,
changed login or lost/malformed reply alone is not a logout ACK; preserve Retry/support and
bounded same-intended-epoch CSRF recovery. Never reinterpret old logout intent as logout of B.

After confirmed logout, promptly dispatch the existing notification mechanism to same-profile
tabs; receiving runnable tabs promptly clear the intended login's private state and navigate to
login without waiting for a poll/action. No guaranteed message delivery, tab-ACK wait, atomic
clearing or simultaneous suspended-tab rendering is required. Missing/late/duplicate/unavailable
notification follows lifecycle/uncertainty revalidation and ordinary synchronisation; it must
not authorise API access or bypass clearing once a change is observed. Do not introduce a new
guaranteed transport or accelerated polling obligation merely to eliminate delivery failure.

Delayed L notices cannot revoke a later N, reset N's view or replay L's logout. Login navigation
while N is authenticated retains R2-NAV-001. If cookie selection changes before the intended
logout can be authorised, reject/revalidate rather than revoke the new login or claim the old
one was revoked. No browser-wide guarantee to locate/revoke all superseded anonymous sessions
is added; the observable logout contract is confirmed revocation of its intended login.

## 5. FR-010 / AC-010 Scope assessment and exact clarification proposal

FR-010 prohibits tasks being accessible through another account; server ownership/authorisation
and old-intent rejection remain absolute. Conventional anonymous response selection changes
which valid login is presented, not which account that login may access. FR-012/013 view lifetime
and FR-008/009/014 persistence/error protection remain unchanged.

AC-010 says neither signed-in account can “view or modify” the other's tasks. Its broad display
wording could be read to prohibit the expressly accepted unobserved prior-authorised display,
particularly an in-flight reply. It is unsafe to silently assert that R2 risk acceptance overrides
that wording. Conversely, no blanket account-isolation weakening or new Scope capability is needed.

**Recommend targeted R1 clarification of this exact boundary**, rather than another auth design
loop. Proposed explanatory text adjacent to AC-010, with FR-010 unchanged:

> Account isolation requires that protected data access and mutations are authenticated and
> authorised for the owning account, with prior-login intent never rebound to another login.
> Known stale or mismatched client responses are discarded and private state is cleared when
> authentication changes are observed. Content previously authorised for a login may temporarily
> remain visible, including completion of an authorised in-flight response, while a cross-tab
> authentication change is unobserved. This limited display residual does not permit cross-account
> API access, mutations or old-intent rebinding; browser-wide atomic rendering is not required.

This text is a **proposal only**. No canonical Scope edit, version change or R1 approval is made.
Independent R1/Scope authority should determine whether this is a clarification of existing
authorisation scope or requires a formal amendment; any canonical change needs Barry's applicable
Scope approval. Independent R2 may review the functional proposal in parallel, but must not
declare a literal Scope conflict waived. If the existing Scope is independently confirmed to
already cover this interpretation, record that finding instead of inventing an unnecessary
Scope revision. The human behavioural direction is supplied; the remaining issue is authoritative
Scope consistency and final approval of the reviewed written amendment, not reconfirmation of
conventional ordering or the same risk acceptance.

## 6. Handoff boundary

No architecture, endpoint, schema, middleware patch or browser capability is selected at R2.
After the appropriate amendment/Scope gates, R3 should reassess native capabilities and remaining
thin guards against the revised outcomes, retire unjustified first-winner/atomic-render proof
obligations, and retain credible security/continuity work. R3 finding IDs/statuses and gates
do not change automatically. See the [evidence map](r2-conventional-auth-amendment-evidence.md)
and [updated scenarios](r2-conventional-auth-amendment-scenarios.md).
