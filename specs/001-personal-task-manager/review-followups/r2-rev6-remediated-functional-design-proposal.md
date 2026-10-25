# R2-Rev6 — consolidated Functional Design remediation proposal

**Revision:** 2, prepared 2026-10-24. **Status:** proposed for targeted independent R2 re-review;
R2-Rev6 NOT_READY. Scope v1.1/R1-Rev4 and R2-Rev5, including R2-NAV-001, remain APPROVED.
R3 NOT_READY; TD-008 OPEN. P1/IndexedDB remains paused. No operative contract change or
implementation, Tasks, Analyze or Build is authorised by this document.

This new candidate revises the [initial R2-Rev6 proposal](r2-rev6-proposed-decisions-and-functional-delta.md)
without editing it or its evidence/manifest. The initial proposal's response-order winner rule,
optional-only logout notification wording and open Q1/Q2 are superseded **in this candidate**
by the consolidated human direction below. Earlier ADRs, approvals and reviews remain history.

## 1. Decision provenance and dispositions

Authority: the current user instruction, “Taskify R2-Rev6 — consolidated human decisions and
Functional Design remediation,” reporting Barry Levy's confirmations. This records human product
direction, not an independent R2 verdict or formal approval of the remediated design. No independent
R2-Rev6 report is present in the workspace or Archive.zip at preparation; the NOT_READY status
and FD-006/007/008 IDs are taken from the instruction, not a fabricated review citation.
Codex prepares corrections; Barry owns any further material human decisions.

| ID | Consolidated direction recorded | Candidate disposition |
|---|---|---|
| FD-007 | Never replace an established login without logout. Of overlapping anonymous submissions, the first login established in the browser wins; a delayed competitor fails with an appropriate conflict/error, preserving authentication/account/epoch/view, even for the same account. Preserve authenticated GET/POST redirects and S2's post-revocation allowance. Error display alone does not prove cookie safety. | Functional clauses and scenarios revised; independent verification/closure pending. Supersedes the initial proposal's last-delivered-valid-cookie selection. |
| FD-008 | Successful logout revokes the intended shared login; promptly notify other same-profile tabs, clear their private content/drafts/queues and navigate to login. Prefer established mechanisms such as BroadcastChannel; R3 chooses. Suspended or notification-missing tabs remain concealed and revalidate on resume/focus/history reveal. No impossible simultaneous suspended rendering; other devices unaffected; preserve confirmed/uncertain outcomes. | Functional clauses and scenarios revised; independent verification/closure pending. Notification is required UX work, not sole authentication authority. |
| FD-006 | Validate current authentication/account/epoch before revealing newly arrived private data; discard mismatched/invalidated responses and conceal on uncertainty. Never move old-account requests/drafts/mutations to a newer login. Preserve ordinary refresh failure and FD-001 locks. | Functional clauses and scenarios revised; independent verification/closure pending. Resolves the product direction behind initial Q2; no technical proof is implied. |
| Q1 | The previously proposed protected-action inventory is accepted; no per-keystroke auth checks. | Human direction recorded; no further Q1 product choice requested. Written coverage remains subject to independent review. |
| S2-DEC-001 | A later valid login response may authenticate the browser after an earlier logout. The revoked session must remain invalid. | Retained, bounded by FD-007: no replacement of another established login. Not permission to revive revoked credentials or a terminally rejected competing attempt. |
| S4-DEC-001 | Instantaneous cross-tab UI clearing is not required. Revalidate authentication before protected actions and prevent stale cross-account access or mutations. | Retained with FD-008 prompt notification/clearing and FD-006 before-reveal validation. No new stale-private-access allowance. |

## 2. Scope and approved Functional Design reconciliation

Authority: [Scope](../spec.md) FR-010/AC-010, FR-008/009/012/013/014, AC-005/008/009;
[R2-Rev5 §7 approval](../../../independent-reviews/2_design-functional/Review_5/Taskify_R2-Rev5_Targeted_Functional_Design_Review.md#7-post-review-formal-human-r2-rev5-decision--2026-10-24).
Current canonical design files include later unapproved R3 text; comparison relies on approved
functional outcomes and their historical manifests, not a claim that current files equal R2-Rev5 bytes.

| Baseline obligation | Reconciliation / exact proposed delta |
|---|---|
| FR-010/AC-010 private accounts; no other account's task access/modification | Retained. Establishing B cannot authorise access to A's cached data, new A payloads or A intents. All private reveals/actions are gated and every service operation remains owner/login scoped. Rendering cannot be simultaneous in suspended tabs; that scheduling limit is not permission for stale cross-account access. No Scope amendment is proposed; independent R2 must verify this consistency. Any demonstrated need to weaken FR-010/AC-010 requires targeted R1/human review before incorporation. |
| R2-NAV-001: authenticated same-profile entry and login GET/POST preserve login/view | Unchanged: root/new tab reaches tasks; login GET 302 /tasks/; valid-CSRF authenticated POST 303 /tasks/ before replacement credential evaluation; invalid CSRF 403; lookup uncertainty 503. A previously anonymous competing submission is a separate concurrency case, not permission to turn authenticated POSTs into fresh logins or conflict errors. |
| R2 shared logout and exceptional login ordering | FD-007 explicitly specifies the first browser-established winner where the initial R2-Rev6 proposal allowed response-order replacement. S2 remains the narrow post-revocation exception: a distinct still-valid late login may authenticate after the original is revoked, if it does not displace another established login. FD-008 requires prompt notification/clearing/navigation; simultaneous rendering in suspended tabs is not required. |
| FR-012/013/AC-005 view lifetime | Winning established login retains its exact remembered view, even against a same-account competitor. A distinct fresh login after revocation begins All/All/Due Date earliest first. Same valid login navigation/rekey does not reset it; other sessions retain their independent views. No tuple/sort/fallback changes. |
| FR-008/009/014 and accepted FD-003/004 | No change to task persistence, owner scoping, receipt/version rules, Retry/support, uncertain-operation protection, memory-only interruption limitations or no credential recovery. Identity validation alone never unlocks task mutation/retry during failed saved-task loading. |

## 3. Proposed affected contract clauses

These are replacement/additional Functional Design clauses for future reconciliation after
the applicable review and human gates. They do not adopt an authentication architecture.

### R2-6R-LOGIN — contracts/ui.md and contracts/http.md

An **established login** is the first successful login that becomes the browser's current usable
authentication. First request submission or first server session creation alone is not the winner
criterion. Header-delivery tests must demonstrate the actual browser outcome in both orders.

Once login L is established, no competing login may replace its credential, account, login epoch
or remembered view until L has been successfully revoked by logout (or legitimately expires).
This applies to different accounts and the same account. A delayed competing anonymous submission
must fail and display an appropriate conflict/error, such as: “Another tab has signed in. Continue
to the current task list. Sign out there before switching accounts.” No automatic credential retry.
For the same-origin JSON conflict, retain the existing 409 authentication_transition_changed
contract; page/controller handling must present the conflict rather than treating it as login success.
No particular internal phase or new endpoint is required by this clause.

The loser must not temporarily install competing authentication and then repair it or merely show
an error. The winner must remain usable for tasks and logout, with unchanged epoch and remembered
view. R3 must prove response/cookie ordering, not just server commit order or final UI appearance.
Authenticated login GET/POST still follow R2-NAV-001 exactly; a stale form carrying the active
login must not evaluate replacement credentials.

After L's confirmed revocation, a **distinct still-valid** late login may authenticate under
S2-DEC-001; the revoked L stays invalid. A rejected attempt is not revived automatically.
If another login C has already become established, FD-007 protects C as well. New fresh login
starts its own default view; old drafts, pending operations and settings are never imported.

### R2-6R-LOGOUT — contracts/ui.md, contracts/http.md and auth states

On logout intent, capture intended L and conceal/clear local private rows, labels, drafts, dialogs
and task/view queues. Show success only after confirmed server revocation of shared L and its
view. Every subsequent use of the revoked credential fails authentication; independent sessions
on other browsers/devices stay valid. No account-wide logout is introduced.

After confirmed revocation, promptly notify other open same-profile tabs using L. Receiving
runnable tabs clear/conceal L's private content/drafts/queues and navigate to the login screen;
do not wait for a future poll or user interaction. Prompt dispatch and handling are required,
without inventing a numerical SLA. Prefer established browser mechanisms such as BroadcastChannel;
the mechanism and failure handling belong to R3. Logout success need not wait for every tab's ACK.

Suspended tabs need not render simultaneously. Initial, hidden, resume/focus/visibility and
history-restored states remain concealed until current authentication and matching private data
validate. Notification loss must not bypass those gates or the protected-action/before-reveal
gates. Notification is a UX trigger, not the authority for server revocation or private access.

Scope notification effects to the revoked login: a delayed L notification cannot revoke a later
valid login N, replay old logout intent against N, or restore L. Any navigation to the login URL
while N is current follows R2-NAV-001 back to N's task screen after validation. Do not erase N's
remembered view or mix old L data into it.

A lost/malformed logout reply, CSRF failure, 401, missing cookie or identity change alone is not
a confirmed logout. Keep uncertain-outcome Retry/support and safe revalidation; never claim the
server effect from UI clearing. Fresh-token recovery remains bounded to the same intended epoch;
an old intent must not log out a newer login. No automatic password/task replay.

### R2-6R-REVEAL — contracts/ui.md and contracts/http.md private responses

Newly arrived private data remains concealed until **current authentication, account and epoch
are validated before its reveal**, including initial/focus/history loads, refresh/filter/sort
responses and private details. A check made only before dispatching the private request does not
satisfy this requirement. Compare response identity with the validated current identity and local
generation/query/revision before exposing private DOM, account labels, dialogs or accessibility text.

Discard mismatched or invalidated responses. On authority uncertainty conceal and offer Retry/
support; do not render an empty-success fallback or treat optional view defaults as authentication.
After identity change clear old account state and load only the current login's data. Never apply
old drafts, creates, edits, state changes, deletes, view saves or retries to a newer login, even
when the account is the same but the epoch differs. Each private service request rechecks live
authentication, owner and intended epoch before protected effects or data access. A stale epoch
or revoked login uses existing 401/no private effect; lookup uncertainty uses 503.

Retain ordinary focused **task-refresh** failure after prior success: keep authorised rows visibly
stale, show failure, and lock every task mutation/retry under FD-001 until a valid saved-task
snapshot loads. This is distinct from failed required **authority** validation, which conceals.
No new data may be revealed through the ordinary task-error path. Optional view read/save failure
stays silent and isolated. A current auth check alone does not settle an unknown task operation
or unlock task writes; preserve original operation identity, receipts and explicit retry rules.

R3 must prove validation/reveal ordering under concurrent auth changes. This document does not
claim a prior check, notification or cached epoch makes cookie selection and paint atomic.

### R2-6R-ACTIONS — accepted Q1 inventory

Revalidate before task-data reads/reveals and refreshes; opening private detail/edit/delete dialogs;
create/edit/complete/reopen/delete submission and explicit retry; filter/sort actions exposing a
new task view; view remembering; and logout. Interactive cached-data access needs the gate as well
as server-side checks. Preserve intent identity from validation through dispatch/effect and reply.
No authentication round trip per keystroke in an already-open draft; submission is gated.
Local Cancel/public support do not grant private access. Mandatory gates must not turn isolated
view-state persistence failure into blocked task access.

### R2-6R-STATES — workflow and data-model semantics

| Event | Functional state/outcome |
|---|---|
| First browser login established | L current; fresh defaults, then L's own remembered view |
| Competitor delayed while L remains active | Conflict/error; L credential/account/epoch/view unchanged and usable |
| Logout pending/unknown | Local private state cleared; no success claim from absence of data/cookie |
| L revocation confirmed | L remains invalid; promptly notify/clear/navigate same-login tabs |
| Distinct valid late login after revocation | May become current with its own fresh defaults; cannot replace another established login |
| Suspended/missed-notification tab resumes | Concealed; validate current auth and matching private data before reveal |
| Private payload arrives | Held concealed; validate current identity, then reveal matching data or discard; uncertainty remains concealed |
| Ordinary same-login task refresh fails | Stale rows/error, all task writes/retries locked; no fabricated empty list |

Data-model.md needs only these logical lifetime/ownership clarifications; no physical key, table,
session backend, transition entity or schema change is proposed here. Tasks/receipts remain
account-owned; view state remains scoped to one valid login. Research/plan should reference these
decision IDs and gate status only after incorporation is authorised. Unrelated contracts remain intact.

## 4. R3 proof obligations and review exit

| Dependency | Required technical evidence, without prescribing architecture |
|---|---|
| FD-007 / TD-008 cookie ordering | Both response-header orders and reversed server completion; first browser-established login never overwritten while active, including same-account competitors; loser conflict alone insufficient. Reconcile S2 only after confirmed revocation and protect any later established login. |
| FD-008 logout and notifications | Confirmed authoritative shared revocation; prompt runnable-tab notification/clear/navigation; lost notifications, suspended tabs and delayed old-login messages handled without stale reveal/action or revoking another session. |
| FD-006 validation/reveal and stale intent | Fresh current identity validation before newly arrived data reveal, mismatch/uncertainty paths, same-account new epochs, late cookies/reads and queued actions. No assertion that one pre-dispatch check is enough. |
| Retained protections | Task/logout commit ordering, CSRF handoff/recovery, exact operation retries, session rekey/view continuity, optional-view failure isolation, browser coverage and independent-device continuity. |

No new authentication architecture, browser dependency or cookie modification is selected. Prior
Option B remains unapproved and its native-response-order assumption is not evidence of FD-007
compliance. If proof requires a product departure, browser restriction or risk acceptance, escalate
the exact change to Barry before incorporating it; do not silently weaken these outcomes.

No material product question is left open by the former Q1/Q2 in this candidate: the user supplied
their direction. Independent R2 must assess written consistency/completeness, then the applicable
human gate remains. Technical feasibility/proof is not an unresolved product permission or R2
finding closure. See the [targeted evidence map](r2-rev6-remediation-evidence.md) for planned scenarios.
