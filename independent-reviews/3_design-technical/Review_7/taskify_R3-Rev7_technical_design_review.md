# Taskify — Independent R3-Rev7 Technical Design Review

**Date:** 24 October 2026 (Australia/Melbourne)  
**Reviewer:** Independent R3 Technical Design Reviewer  
**Verdict:** **NOT READY — TD-008 remains OPEN / Medium / blocking**  
**Human R3 gate:** OPEN / not approved. No Tasks, Analyze, Build or runtime execution is authorised by this review.

## 1. Decision summary

**Maintenance rotation — corrected at written-design level.** The R3-Rev6 counterexample in which a delayed provisional rotation response overwrote A's only usable cookie is no longer reachable under the newly specified rotation interface: it changes only the server-side Django Session key and view state under a family lock, while retaining the existing browser handle, account, epoch and expiry. There is no replacement `Set-Cookie` on rotation (`auth-transition-protocol.md` §2; `contracts/http.md` §TD-008).

**Stale cross-tab CSRF — corrected at written-design level.** A fresh same-origin token request, bounded retry for trusted pre-dispatch CSRF failures, same-intended-epoch revalidation before logout retry, and an explicit prohibition on automatic password/task-write replay address the R3-Rev6 integration concern (`auth-transition-protocol.md` §3; HTTP/UI contracts). No successful logout is inferred from a 403 or lost acknowledgement. Actual browser/framework behavior has not been tested.

**Anonymous concurrent login — unresolved.** Two tabs can submit credential forms before either login is established. A later cancelled/provisional response must neither replace the newly valid A credential nor make authoritative any-tab logout impossible. Codex correctly withdrew the unsafe one-slot writer, but **has not supplied an operative replacement** (`auth-transition-protocol.md` §4; `technical-design.md` §2; `R7-T02/03`). This is the remaining TD-008 blocker, not an accepted FD-004 risk.

**P1 is not approved.** Atomic IndexedDB enrolment is a plausible *candidate* for sharing a pre-auth browser context; the storage transaction alone does not establish a secure session/cookie protocol. P1 explicitly leaves proof binding, cookie naming/quotas/cleanup and failure paths for further design and verification. Its proposed behavior when IndexedDB is unavailable in a fresh profile could alter the approved availability expectation and requires Barry's specific **P1-A** decision before adoption or a corresponding R2/Scope change.

## 2. Evidence baseline and preservation

| Item | Independent result |
|---|---|
| Submitted attachment | `Archive(8).zip`, SHA-256 `6175cc77bca2791d10ea23190726675f3af8f1d1242f85764fe20c549f56d1ed` |
| Current correction manifest | All **10/10** entry SHA-256 digests independently recomputed and MATCHED the files inside the attachment |
| Approved Scope | `specs/001-personal-task-manager/spec.md` SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`, unchanged from Rev6 attachment |
| Approved functional design | `independent-reviews/2_design-functional/Review_5/Taskify_R2-Rev5_Targeted_Functional_Design_Review.md` SHA-256 `f08218d808638187bb5227a386e5b8c91913e481308a62a2f62b9016559eca90`, unchanged; human approval includes R2-NAV-001 |
| Other preservation | Requirements checklist unchanged; `technical-design.md` §§3–8 byte-identical between Rev6 and Rev7 attachments |
| Actual verification | Document and static consistency review only. R7-T01–09, NAV-01–08, R3/R5 fixtures, database/browser/security/restore tests remain **planned, not executed** |

Sources under `specs/001-personal-task-manager/` include `auth-transition-protocol.md`, `technical-design.md`, `contracts/http.md`, `contracts/ui.md`, `data-model.md`, `plan.md`, `quickstart.md`, `research.md`, `review-followups/r3-rev7-correction-evidence.md`, and `review-followups/r3-rev7-browser-context-proposal.md`. Source assertions were checked against those documents, not treated as independent test evidence.

## 3. Finding register and dispositions

| Finding | Rev7 independent disposition | Next evidence/owner |
|---|---|---|
| **TD-001** High | **RESOLVED (written design), retained**; family claim/generation and privacy concealment unchanged. No regression demonstrated. | Actual cross-tab/DB ordering and privacy tests at implementation |
| TD-002 Medium | RESOLVED (design), retained | Task-first bootstrap and silent complete view fallback regression |
| TD-003 Medium | RESOLVED (design), retained | Load, resource and timeout measurement downstream |
| TD-004 Medium | RESOLVED (design), retained | HMAC receipt/key retention and replay security tests downstream |
| TD-005 Medium | RESOLVED (design), retained | Ingress/throttle enforcement tests downstream |
| **TD-006** Medium | **OPEN, nonblocking R5 evidence dependency**, retained | Barry owns Operations/R5 sign-off; Codex prepares restore/runbook evidence |
| TD-007 Low | RESOLVED (design), retained | Barry owns Maintenance/R4 compatibility; upgrade/replay fixtures downstream |
| **TD-008** Medium | **OPEN, BLOCKING**. Rotation and CSRF subissues corrected in writing; anonymous concurrent issuance, account continuity and authoritative logout have no completed operative mechanism. | Codex redesign + independent R3 re-review; Barry decides P1-A only if proposed availability trade-off pursued |
| TD-009 Medium | RESOLVED (design), retained | Isolated bounded optional-view tests downstream |

R2-Rev5 navigation approval and FD-003/004's previously accepted *specific* risks remain intact. No new risk acceptance, R3 approval or R1/R2 reopening is inferred.

### TD-008 residual counterexample

1. Both tabs are signed out and submit A/B login forms before either response has completed.
2. A becomes ACTIVE and its task list is usable. B's older response arrives later.
3. Any shared provisional credential destination that B can overwrite can make A's browser proof unusable while A remains live on the server. A subsequent logout then cannot authoritatively revoke A from that browser context.

The approved redirect for *already authenticated* login GET/POST does not cancel requests dispatched when both tabs were anonymous. Rev7 **acknowledges rather than repairs** this permitted order. A safe P1 alternative would need one pre-auth context fence **and** independent confirmed-credential delivery/selection, with full loss, expiry and logout semantics. IndexedDB transaction serialization is a suitable building block but does not independently supply those guarantees ([W3C IndexedDB 3.0](https://www.w3.org/TR/IndexedDB-3/)).

### Editorial consistency to correct during next iteration (not a new blocking finding)

- `contracts/ui.md:147–149` still says “B first retires A” and “No ... new capability restriction” as if it were an operative, fully resolved transition; this conflicts with Rev7's no-live-account-replacement rule and explicit P1-A HOLD. Clarify what, if anything, that ordering denotes; do not carry a retired mechanism forward as a contract.
- `quickstart.md:122` states “Only maintenance rotation moves view/epoch,” whereas current rotation must **preserve** account, login epoch and complete view tuple. Adjust the expected fixture; `quickstart.md:125–159` correctly labels superseded R5 schedules historical.

These phrases do not independently undermine the corrected maintenance/CSRF mechanism but could misdirect a later implementation or test. Require reconciliation before R3 approval.

## 4. Human decision P1-A — explicitly pending

**Question:** May fresh sign-in require successful IndexedDB browser-context enrolment when neither a context nor an existing valid session has been established? Under P1, a browser/profile where that storage is blocked or unavailable would show technical Retry instead of signing in, even if cookies and JavaScript otherwise work. An existing live session must continue to show tasks and support logout without IndexedDB.

**Option A — preserve existing availability (no new dependency):** Reject mandatory IndexedDB enrolment and ask Codex for another design that preserves the approved experience. TD-008 remains open.

**Option B — investigate a conditional amendment:** Authorise a *targeted assessment/proposed R2 availability/storage amendment* (and R1 review if it changes Scope), including browser/private-mode/error tests and alternatives. This is **not** approval to incorporate P1 or sign off TD-008; explicit later human approval remains necessary.

**Reviewer recommendation:** Do **not approve P1's dependency or R3 yet**. If Barry is open to reassessing the narrow fresh-profile availability trade-off, Option B permits evidence-based evaluation without pre-committing. Otherwise choose Option A. It would be premature to assert that no cookie-only/availability-preserving solution exists.

## 5. Required next pass and gate

1. Obtain and record P1-A direction. If considering P1, run targeted functional-availability assessment and necessary explicit human approvals **before** making it an operative design requirement.
2. Codex completes anonymous login/session delivery and binding semantics: atomic first-context enrolment; resistance to a guessed/copied context ID; strict candidate-vs-confirmed proof isolation; current-session selection; cookie names/quotas/cleanup; missing or delayed headers; cross-tab revocation; independent device/fresh-profile failures; safe retries and expiry. Or propose a comparably complete availability-preserving solution.
3. Correct the two current-text inconsistencies above. Provide updated design, precise per-finding evidence, changed-file hashes and deterministic R7-T02/03/08 test scenarios. Preserve the unchanged Scope, R2 approvals and historical review records.
4. Independent reviewer rechecks **TD-008 and directly affected interfaces/regressions only**; do not unnecessarily reopen resolved TDs. Runtime scenarios are downstream verification, not proof of R3 approval today.

**R3-Rev7 independent verdict: NOT READY. Human R3 gate OPEN.** Do not run Tasks, Analyze or Build.
