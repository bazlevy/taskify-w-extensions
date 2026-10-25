# R3-Rev10 deterministic validation plan

**Date:** 2026-10-25. **Status:** planned, NOT EXECUTED. R3 NOT_READY; TD-008/010/011 OPEN.
Approved [CA-T01–16](../quickstart.md#approved-r2-rev8-acceptance-scenarios--ca-t0116-planned)
remain verbatim; this companion specifies technical schedules under
[ADR-AS-004/lifecycle](../auth-transition-protocol.md). No new R2 outcome or phase authority.

## Harness and observable boundaries

At authorised Build/Test, use real PostgreSQL independent connections and controllable barriers:
before Session lock, after locked auth, before command commit, before logout commit, after commit/
before response, before browser header application, and before JS callback. Do not model header
application by merely resolving promises in reversed order. Separate same-profile tabs, separate
devices, different accounts and same-account distinct epochs. Use isolated clocks for expiry/
receipt cleanup. Observe account/epoch, task version/list revision/receipt effects, optional-view
tuple, auth-cookie presence or absence on actual wire, DOM/accessibility state and notification
dispatch/handler order. Do not capture raw cookies/proofs/passwords/task text in diagnostic logs.

Later test execution must cover Django's maintained locked patch and Chromium/Firefox/WebKit,
plus actual supported mobile smoke checks. Adapter semantics/native middleware compatibility are
explicit integration assertions, not inferred from mocked return values. No performance SLA,
absolute rendering fence or new browser-support prerequisite.

## CA-T01–16 technical schedules

| Approved ID | Controlled schedule and deterministic assertions |
|---|---|
| CA-T01 | Admit A/B requests anonymous before either login; persist both native Sessions. Hold headers independently of commits; apply A then B. New guarded discovery/task request authenticates B, with B epoch/default view. No winner 409 required; A may remain valid independently. Repeat same account with two epochs and deliberately different remembered tuples. No client callback imports another login's tuple. |
| CA-T02 | Apply B then A, independently reverse commit and callback order. Guarded requests select actual cookie A; callback order cannot overwrite controller identity using login response metadata. Only fresh-login responses carry the auth cookie; no family/candidate cookie exists. Interrupted header delivery has no fabricated winner guarantee. |
| CA-T03 | Authenticated A/nondefault view: root/new tab/login GET gives required 302; stale form POST, even wrong B credentials, gives 303 after valid CSRF and before authenticate(password). Invalid CSRF gives 403. Assert unchanged key/epoch/view/expiry and no auth Set-Cookie. Fail auth DB: 503, no anonymous fallback. Restore form during token rotation: fresh CSRF/discovery redirects without password replay. |
| CA-T04 | Revoke L and confirm receipt. Deliver a distinct valid anonymous N login response held earlier: N can authenticate/default, while direct L credential replay fails. Establish C then deliver a genuinely anonymous competing response: actual cookie may change. Do not assert C winner protection. Revoked/invalid cookie grants no task access; no view/draft migration. |
| CA-T05 | Capture A intended epoch/payload; dispatch with B cookie, then same-account new epoch. Test reads/create/edit/status/delete/view and original operation retry. Reject 401 before owner/receipt/task/view access/effect. Forged owner IDs rejected; foreign task IDs give existing indistinguishable 404 under rightful B epoch. No task receipt or list revision on rejected old intent; never retry as B. |
| CA-T06 | Hold A task/mutation/view replies; observe logout notice, current-generation auth failure or changed boundary identity. Clear old state and increment authGeneration, then deliver held replies: no private DOM/draft/control update. Adopt N; deliver old A 401/discovery/ACK callbacks: cannot clear N, rebind requests or lower revision. Reverse same-login query generations and revisions: discard old query/below-ACK replies. |
| CA-T07 | Hold already-authorised A response, change selected login in another tab, drop all notices and prevent boundary/poll observation. A can remain visible/finish rendering under approved residual; no zero-exposure assertion. Next observed change clears. Submit old intent using new cookie: CA-T05 rejection still mandatory. Delayed matching authority reply is not proof of current browser-wide identity. |
| CA-T08 | Enter/focus/visibility/pageshow/history/stale-form restoration: conceal immediately in lifecycle handler, increment boundary generation, hold discovery and task snapshot, reject earlier-generation completion. 401 keeps private-free; 503 stays concealed Retry/support. Matching discovery plus valid snapshot permits reveal/write readiness subject to pending-operation state. Repeated cached dialog/keystroke and normal task reply cause no independent auth probe. No suspended-tab simultaneous paint claim. |
| CA-T09 | After valid L snapshot fail ordinary task refresh: keep visibly stale rows/error; all task writes and retry locked. Fail required discovery separately: conceal. Fail/lock/timeout optional view table separately: full silent defaults and available task snapshot under TD-009 bounds; no auth fallback. Valid authority alone cannot unlock writes or settle unknown mutation. |
| CA-T10 | Commit logout L with receipt/view deletion; drop ACK. Retry exact epoch/token: trusted receipt ACK with no new effect, including when N cookie selected. No receipt plus N/absent cookie: never revoke N or report L success. Malformed ACK/401/403/COMMIT connection loss stays unknown until receipt confirmed. Test duplicate logout waiting on Session deletion, token mismatch, cleanup deadline and proof lost on reload; no false ACK. |
| CA-T11 | Receive matching committed/replayed L ACK: local clearing and channel post occur in ACK handler, before next poll; unknown outcomes produce no confirmed post. Deliver event to runnable L tab: clear rows/labels/drafts/dialogs/task-view queues, invalidate generations and navigate login in handler. No tab-ACK wait. A confirming tab now on N may signal L without clearing N. |
| CA-T12 | Throw channel constructor/post, drop/delay/duplicate messages; suspend tab and bfcache/history restore. Committed server logout still succeeds; no capability block/new transport. Restored controller remains concealed while discovery pending; revoked login cannot reveal. Visible existing poll may discover 401. Do not assert message receipt, deadline, faster polling or atomic suspension clearing. |
| CA-T13 | Deliver L event/ACK/CSRF refresh after N current: cannot revoke N, clear N view or replay old logout as N. Trusted CSRF rejection permits one same-epoch housekeeping/logout retry; change epoch before retry and it must not dispatch new revocation. Second rejection fails safely. Explicit receipt confirmation under N is read-only. Stale login navigation redirects. No password/task auto-replay. |
| CA-T14 | Give independent-device D distinct epoch/nondefault view; logout this profile L, select fresh same-account N. D remains live with own view; N defaults; same-login navigation retains N tuple. Session/view saves never import D/L settings. Shared account tasks still converge through ordinary snapshots. No maintenance rekey endpoint; no navigation-induced key/expiry changes. |
| CA-T15 | Barrier 1: command holds Session through effect/receipt commit; logout waits, then commits/ACKs. Barrier 2: logout holds Session and deletes first; command previously authenticated in middleware waits then rejects before effect/receipt replay. Repeat create/edit/status/delete, receipt replay, view save and concurrent logout. Lost command reply + failed snapshot keeps all writes/exact retry locked; valid snapshot then explicit original ID/payload retry obeys receipt/version guards. Effects committed before logout persist. |
| CA-T16 | Run retained AM001-V01–10, NAV-01–08 and R2-T05/T07/T12 with trace below. Validate native fixation, separate CSRF/cookie flags, no-store/escaping, owner isolation, login admission/throttling, task lifecycle/filter/order/date/version/receipt behavior and supported availability. No Web Locks/IndexedDB/election/render fence in runtime capability requirements. FD-003/004 disclosures and recovery bounds remain. |

## Additional integration assertions and prior-test disposition

| Reference | Retained stimulus / current assertion |
|---|---|
| R10-X01 / delayed middleware | Hold L logout ACK, L 401/error, anonymous GET and task/view response until after N cookie applies. None sets/deletes auth cookie or re-saves Session; N stays valid. Inject unintended Session write inside guard: effects roll back/error. Inject after commit: no silent save or false rollback claim; existing uncertain-outcome handling. Only fresh committed login cookie issuance is allowed; independently delayed CSRF cookie may produce bounded-recovery 403 only. |
| R10-X02 / session expiry and cleanup | Advance database time while waiting for Session lock: check current time after acquire, reject expired authority. Delete expired row/view: no recreation by delayed operation/middleware. Cleanup LogoutReceipt after retain_until: no authority change or manufactured confirmation. Include simultaneous logout/duplicate receipt commit. |
| R10-X03 / issuance failure | Fail native login transaction before commit: no valid cookie or partial auth payload/view. Lose response after commit: do not claim rollback; only explicit credentials may retry while anonymous. Native database key cycling and auth-hash verification remain; no periodic/live rekey route. Invalid hash/inactive/deleted user on GET: native validation rejects through memory-only facade, with no Session write/delete. Attempt unreviewed fallback rekey: fail closed before protected effects, diagnostic and upgrade preflight refusal. |
| R3-V01 / TD-001 | Revalidate preserved account-isolation substance under native guard/local boundaries, replacing family/fence assumptions. CA-T05–08/12 and X01 retain every server isolation/known-stale prohibition; only approved unobserved display is permitted. |
| R3-V03 / TD-003 | Keep existing workload/pool/payload/receipt-growth matrix; add Session-lock contention, logout receipt storage/index/cleanup volume and channel-disabled polls at unchanged interval. No new connection pool/service, host cap or SLA selected. Record measurements later. |
| R3-V04/V05/V07 | Receipt HMAC/canonicalizer/key-rotation privacy and compatibility, source-normalized shared auth limiting remain unchanged. Do not confuse logout token hash with command HMAC or reuse either proof for the other. Upgrade preflight also refuses an unplanned live auth-key rekey/view-loss path. |
| R3-V06 / TD-006 | Add LogoutReceipt to consistent backup inventory; restore invalidates Sessions/views and clears logout confirmation receipts and limiter before serving. Receipt-only confirmation cannot attest restored authority. Barry accountable at R5; Codex prepares drill; runtime evidence belongs to authorised Build/Test/R5. |
| R3-V08 | CA-T01–05/10/13–15 and X01–03 replace obsolete maintenance/family rotation outcomes. Valid navigation retains view/epoch/key; actual login defaults. Removed maintenance/phase routes have no auth effects (unrouted 404/405), not a backdoor compatibility path. |
| R3-V09 / AM001-V01–10 | Optional view read holds no mandatory Session/account lock while waiting. Block only optional relation/pool/transport; maintain existing bounded silent defaults, independent task read, no late-control apply. Auth/task failure still fails protected/FD-001. Last committed view and failed-save silence unchanged. |
| R2-T05/T07/T12; NAV-01–08 | Preserve FD-001 write/retry lock, FD-003 memory interruption, FD-004 no-recovery text and R2-NAV-001. Navigation never evaluates replacement credentials under live cookie; genuine anonymous races are separate. |
| R7-T01–03 / R5-T legacy races | Delay old headers through logout/new login. No maintenance issuance now exists; ordinary responses cannot erase N; distinct anonymous fresh response may select its valid session. Session-key replay after revocation fails. Do not restore first-winner/provisional-cookie expectations. |
| R7-T04–07 | Keep cross-tab CSRF handoff, one bounded same-epoch recovery, changed-identity no-replay and task/view failure separation via CA-T03/09/10/13/15. No claim/activation requests remain. |
| R7-T08 / P1-A | Still paused, not an unmet implementation fixture or current availability dependency. |
| R7-T09; A9-01–06 history | Keep owner/receipt/security/view regressions. Replace strict winner and after-arrival atomic-fence or guaranteed-delivery assertions with approved CA outcomes. A held identity reply is not a browser fence; CA-T07 residual is expected, not newly accepted. |

## Evidence ownership and exits

Codex prepares fixtures, request/transaction/header schedules and safe observables. Author static
review verifies the written map only. Independent R3 checks coherence/feasibility and retained
requirements; Barry alone decides the R3 human gate. After approval and separate authority,
Build/Test executes and records per-scenario results with exact versions. Release readiness
requires the retained security/integrity cases passing or an explicit applicable human decision;
this plan grants no waiver. TD-006/007 keep Barry's R5/R4 ownership and existing exits. No runtime
proof has been performed or implied by this document.
