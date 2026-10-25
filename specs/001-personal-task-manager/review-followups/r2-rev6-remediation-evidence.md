# R2-Rev6 remediation — targeted change/evidence map

**Prepared:** 2026-10-24. **Review candidate:** revision 2 of the proposed Functional Design.
**Gate:** R2-Rev6 NOT_READY, targeted independent re-review pending; R3 NOT_READY, TD-008 OPEN.
Scope v1.1 and R2-Rev5/R2-NAV-001 approvals remain intact. P1 remains paused.

Review [the remediated functional clauses and decision record](r2-rev6-remediated-functional-design-proposal.md).
Barry's consolidated direction is recorded there; no independent finding is marked verified or
closed, and no formal revised-design approval is inferred. The earlier proposal/evidence/manifest
remain unchanged as history. This map supersedes their current-candidate expectations only.

## Per-finding change and traceability

| Finding / decision | Exact correction | Affected functional targets / trace | Review evidence |
|---|---|---|---|
| FD-007 | Replace last-delivered-cookie selection with first browser-established login protection; same-account competitors also conflict without altering auth/account/epoch/view. Preserve authenticated GET/POST redirects. | R2-6R-LOGIN/STATES; UI navigation, HTTP auth/conflict semantics, logical login/view lifetime; FR-010/012/013, AC-005/010, R2-NAV-001 | T01–04/13/16; R3 proof must include actual cookie/header scheduling, not error text alone |
| S2-DEC-001 | Retain distinct valid late login after confirmed revocation, without reviving the revoked login or a rejected attempt, or displacing another established login. | LOGIN/LOGOUT; FR-010, FR-012/AC-005; explicit exceptional boundary vs normal logout-first flow | T03/04/16; approved Scope unchanged |
| FD-008 / S4-DEC-001 | Require prompt logout notification, clearing and login navigation in runnable same-login tabs. Missed/suspended tabs remain gated on resume/focus/history; no impossible simultaneous rendering. Preserve authoritative revocation and unknown-outcome handling. | LOGOUT/STATES; UI privacy/navigation/failures, HTTP logout guarantees; FR-010/AC-010, FR-009/014; independent devices unaffected | T05/06/11/12/15; mechanism belongs to R3, BroadcastChannel preferred but not mandated as sole authority |
| FD-006 | Require current auth/account/epoch validation after private payload arrival and before new reveal; discard mismatches, conceal uncertainty, prevent cross-login intent replay; retain ordinary task error and FD-001 behaviour. | REVEAL/ACTIONS; UI response and action gates, HTTP private preconditions; FR-008/009/010/014, AC-008/010 | T07–10/13; former Q2 human direction supplied, no unobserved late-reveal waiver |
| Accepted Q1 | Full protected-action inventory accepted, including local detail/edit/delete/filter/sort gates; no per-keystroke checks. | ACTIONS; UI forms/filters, cached access, private requests and logout | T09/10/13; no additional Q1 product approval requested |
| Retained guarantees | Task ownership/persistence, receipt/version protection, silent isolated view fallback, fresh-login defaults and valid-login retention, FD-003/004, browser coverage | FR-008/010/012/013/014/015; R2-NAV-001; unchanged contracts and historical approvals | T10–14 and retained NAV/AM001/R2 scenarios; no architecture adopted |

## Proposed artifact reconciliation — no operative files edited

| Artifact | Minimum prospective revision after applicable gates |
|---|---|
| contracts/ui.md | Apply LOGIN/LOGOUT/REVEAL/ACTIONS/STATES clauses only to affected auth/privacy/action/failure sections; preserve task forms, ordering, notices and unrelated states. |
| contracts/http.md | Clarify winner protection/conflict, S2 exception, logout acknowledgement and expected-epoch/private-response preconditions. Preserve GET 302/POST 303/CSRF 403, 401/503 distinction, task payloads/receipts. |
| data-model.md | Logical login/view/revocation lifecycle only; no physical schema/key, backend or transition-protocol decision. |
| research.md / plan.md | Record FD-006/007/008, S2/S4 and Q1 provenance; separate R2 direction from still-unapproved R3 mechanisms. No Plan rerun here. |
| quickstart.md | Add T01–16 below; revise only superseded current-candidate expectations after approval, retaining old scenario history and unaffected regression intent. |
| checklists/feature.md | Prospective targeted addendum for CHK016/022/034/036/047/049/050/052; no old text/checkmarks rewritten and no checklist marked independently passed. |
| Scope, independent reports/approvals, previous ADRs/proposals/manifests, R3 operative artifacts | No edits. Prior response-order assumptions are historical, not evidence of compliance with FD-007. |

## Deterministic acceptance scenarios — planned only

Use two same-profile tabs, distinct A/B accounts and same-account distinct epochs. Independently
schedule server completion, actual response headers, DOM updates and notification delivery.
Record cookies/selected auth, server revocation/effects, account/epoch/view, DOM/accessibility and
navigation. A usable credential applied first defines browser establishment; server commit alone
does not. Preserve a nondefault remembered view in the winner before releasing a competitor.
R2-6R-T01–16 below are new revision-2 IDs; earlier R2-6-T01–12 remain historical.

| ID | Schedule / required observable outcome |
|---|---|
| R2-6R-T01 / FD-007 | Submit A/B while anonymous; establish A in browser first, hold B headers. Let A save a nondefault view, then release B. B fails with conflict/error; A remains authenticated with exactly the same account/epoch/view, task access and logout capability. Repeat with both server-completion orders and with the same account in both requests. No temporary competitor cookie installation followed by repair. |
| R2-6R-T02 / FD-007 | Reverse header delivery: B becomes the first usable browser login, A delayed. B remains unchanged/usable and A conflicts. Again reverse server completion and repeat same-account distinct epochs; request/commit ordering must not substitute for browser establishment. |
| R2-6R-T03 / S2 | Hold a distinct still-valid login's headers; original L becomes current, then logout L commits successfully; only afterward deliver that eligible late login. It may authenticate with its own fresh epoch/defaults. Direct revoked-L replay against private APIs fails; no L view/draft/queue revival or old-account effects. Repeat same/different account. |
| R2-6R-T04 / FD-007/S2 | Reject a competitor while L remains active; confirm that error responses cannot install authentication or alter L, even if their headers arrive late. Revoke L; the terminally rejected attempt does not revive automatically. S2 applies only to a distinct still-valid login. |
| R2-6R-T05 / FD-008 | With L visible in multiple runnable tabs, confirm logout in one. Observe prompt notification dispatch; receiving tabs clear private rows/labels/drafts/dialogs/task-view queues and navigate to login without waiting for a poll/action. No private content in accessibility state. No requirement to wait for all tab ACKs before confirming server revocation. |
| R2-6R-T06 / FD-008 | Suspend/hide or history-cache an L tab; revoke L elsewhere. Drop notifications, then resume/focus/pageshow it. It starts concealed and revalidates before private reveal/action; 401 stays private-free, 503 stays concealed with Retry/support. No simultaneous rendering during suspension is required. Also drop notification to an already-focused tab and trigger a protected action/new private response: mandatory gates still enforce invalidation. |
| R2-6R-T07 / FD-006 | Hold an A private response. Successfully revoke A and establish B via allowed fresh/late login; drop logout notifications. Deliver A data while the old tab is focused. Validate current B after arrival, discard A payload before any new private DOM/accessibility reveal; clear old state, no A draft/queue transfer. Repeat with same-account new epoch and known invalidation. |
| R2-6R-T08 / FD-006 | Deliver a same-epoch private response, then fail its required current-auth validation. Data stays concealed; Retry/support, no empty-success or view-default bypass. Recovery reveals only data matching the newly validated identity/generation/query/revision. Race auth changes around the validation/reveal boundary; R3 must demonstrate the ordering protection, not count a pre-dispatch check as proof. |
| R2-6R-T09 / FD-006/Q1 | Exercise every accepted protected-action entry, including cached detail/edit/delete/filter/sort, remembering and logout. After validation of A, revoke A and establish B before dispatch; send old A intent with B's valid CSRF. Expected-epoch guard rejects before private access/effects/receipt/view mutation. Never rebind or replay old intent to B; no per-keystroke checks in an open draft. |
| R2-6R-T10 / FD-001 | Separately inject ordinary task-refresh failure after authorised success, required-auth-gate failure, and view-only read/save failure. Task failure retains visibly stale rows/error and locks all task mutations/retries until valid task load; auth uncertainty conceals. View-only failure stays silent/isolated. Auth success alone never unlocks an unknown task operation. Retain exact operation ID/payload and explicit retry. |
| R2-6R-T11 / FD-008 | A separate browser/device has same-account login and its own nondefault view. Logout this profile. Other device remains authenticated, keeps its view and can synchronise saved tasks. No account-wide revocation or cross-login settings inheritance. |
| R2-6R-T12 / FD-008/S2 | After confirmed L logout, establish valid N, then deliver a delayed notification for revoked L. Clear no N-owned view or authority, never send old logout as N. Any login-URL navigation obeys R2-NAV-001 and reaches validated N tasks; no old L content reappears. |
| R2-6R-T13 / R2-NAV-001/CSRF | Authenticated root/new tab and GET login reach existing tasks/view; valid-CSRF login POST gives 303 before evaluating A/B replacement credentials. Invalid CSRF remains 403; auth storage failure 503, never anonymous. Stale pre-login token recovery is bounded to intended login; no password/task automatic replay or newer-login logout. |
| R2-6R-T14 / retained R2 | Regress AM001-V01–10, NAV-01–08 intent, R2-T05/T07/T12, task ownership/version/receipt replay, filter/sort ties, all-or-nothing silent defaults and valid-login rekey view retention. Fresh login after revocation defaults; competing failure and authenticated navigation preserve winner view. No browser/storage coverage change. |
| R2-6R-T15 / logout/task ordering | Test task commit before logout, then revocation winning before command effect; no old-authority command commits after logout success. Lose/malform logout ACK or return CSRF failure: remain uncertain, not false success. Clear local data safely; identity change alone is not proof of revocation. Recover through existing Retry/support without replay against a newer login. |
| R2-6R-T16 / FD-007/S2 | After L's revocation, establish C before a distinct delayed B response. C is now protected: B cannot replace its auth/account/epoch/view. S2 is not a bypass of another established login. Repeat same-account distinct epochs and opposite header order for which eligible fresh login first establishes. |

## Gate and residual dependencies

Q1 inventory and Q2 late-reveal direction are supplied by Barry; no additional product decision
is currently requested. Independent R2 re-review must verify clause/scenario consistency and
FR-010/AC-010 reconciliation. Formal approval of the revised Functional Design is still pending.
This package does not reopen R2-NAV-001 approval or alter accepted FD-003/004 risk records.

R3 owns proof of winner-preserving cookie ordering, shared revocation, notification failure and
suspension behaviour, current-auth validation/reveal ordering, stale-intent rejection, CSRF,
task/logout transactions and rekey/view continuity. No new architecture or mitigation is adopted.
Any inability to meet these outcomes remains a technical blocker; a required product/browser/risk
departure must be escalated specifically to Barry. R3 NOT_READY / TD-008 OPEN is unchanged.

Only the remediated proposal, this evidence map and [new SHA-256 manifest](r2-rev6-remediation.sha256)
are added. The manifest omits itself. Preservation, local links, table structure, scenario IDs and
whitespace are author static checks; no application/browser/database/runtime tests have run.

Preservation check: all 64 pre-existing files under specs/ and independent-reviews/ match their
start-of-remediation hashes, including the initial R2-Rev6 candidate and prior manifests.
Scope SHA-256 remains
`b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`;
R2-Rev5 approval-record SHA-256 remains
`f08218d808638187bb5227a386e5b8c91913e481308a62a2f62b9016559eca90`.
All 16 new scenario IDs are unique/complete; local document links, table structure and whitespace
pass static author checks. These checks are not an independent R2 verdict or runtime assurance.
