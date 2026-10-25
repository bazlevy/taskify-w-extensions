# Taskify — Independent R3-Rev3 Technical Design Review

**Review date:** 2026-10-23 (Australia/Melbourne)  
**Reviewer:** Independent Technical Design Reviewer (R3)  
**Independent verdict:** **NOT READY — targeted corrections required**  
**Human R3 gate:** **OPEN — not approved; no new risk acceptance**  
**Review type:** Static, evidence-based technical design re-review. No implementation, database/browser/load tests, deployment, restore drill or production validation was performed.

## 1. Executive decision

The R3-Rev3 package makes substantial, verifiable progress. The former capacity, receipt privacy, authentication throttling and optional-view-read blockers now have coherent written mechanisms, resource limits and planned validation. The explicit-login protocol fixes the original same-user Django-login reuse flaw, but leaves a narrower concurrent-login cookie-ordering guarantee unspecified. The browser concealment protocol still permits a concrete cross-tab account-change race after focus revalidation. The latter concerns private-account isolation and must not be closed solely because the proposed test names dropped signals.

**Blocking findings still open:** **TD-001 (High)** and **TD-008 (Medium, narrowed)**. The remaining corrected technical blockers **TD-003, TD-004, TD-005, TD-009** are **Resolved at written-design level**. **TD-002** remains Resolved by AM-001. **TD-007** is Resolved at design level with explicit R4/upgrade evidence later. **TD-006** has its R3 ownership/handoff condition satisfied but remains **Open—nonblocking, R5 execution evidence outstanding**. No new finding ID is necessary: the remaining races fall within prior TD-001 and TD-008.

The recommended next step is a *targeted* TD-001/008 author correction and R3-Rev4 re-review. No new R1/R2 amendment is currently necessary if Codex can preserve approved observable behaviour. If an effective solution needs weaker privacy guarantees or changes task-load failure visibility, refer that specific trade-off for human/Functional Design consideration rather than assuming acceptance.

## 2. Controlled evidence and review boundaries

- **Submission:** `Archive(4).zip`, independently computed SHA-256 `6364a93a489d5c59276d4eda717698af4ea55cdc4330ba5d42c334d5dd56dff6`.
- **Scope:** `specs/001-personal-task-manager/spec.md` SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`, exact match to the approved Scope v1.1 / AM-001 canonical baseline. R1-Rev4 separately records human approval of the scope candidate and subsequent metadata-only identity.
- **Functional Design:** `independent-reviews/2_design-functional/Review_4/taskify_R2-Rev4_functional_design_review.md` §8 records the independent recommendation followed by Barry Levy's explicit human APPROVED decision for nine individually hashed files. Those nine are the *historical approved baseline*; the seven source-design files subsequently edited for R3 are new **unapproved technical candidates**, not retroactively covered by the old approval.
- **Review history:** `independent-reviews/3_design-technical/Review_2/taskify_R3-Rev2_technical_design_review.md` matches its previous SHA-256 `1c0462400543821054bcae2ed1a8378eca46385476a1b01bdde30a9cdeac7933`. Its prior disposition history is preserved. Both old R2 checklists and `review-followups/am001-r2-review-handoff.md` match the previously approved manifest hashes.
- **Current correction identity:** All eight entries of `review-followups/r3-rev3-changed-files.sha256` independently match the supplied contents. The new operative technical design `technical-design.md` hashes to `acb46e491b418999d065a5cb3c52e47977c9c93fcf518a23957c647bfb17d309`; correction map `review-followups/r3-rev3-correction-evidence.md` hashes to `06612f31afe0500238e6b78bcfe72198f3a012f0047eb4b902836098b66f9c74`.
- **Nonexecution boundary:** R3-V01/03/04/05/06/07/08/09 and AM001-V01–10 are planned test scenarios, not observed passes. Technical viability below means the *design as written* is sufficiently plausible unless an explicit gap is identified. Later Build/Test/R4/R5 results can reopen a design finding.

All relative evidence paths in the findings refer to `specs/001-personal-task-manager/`. `technical-design.md` is the R3-Rev3 operative correction supplement; prior review/proposal documents remain historical.

## 3. Finding-by-finding dispositions

| Finding | Classification; inherited severity | Rev1 → Rev2 → **Rev3** | Design-level evidence / gate consequence |
|---|---|---|---|
| **TD-001** cross-tab private data | Design Gap; **High** | Open → Open → **Open, blocker** | Conceal-before-focus is defined, but late cross-tab cookie commit after revalidation is not fail-safe. |
| **TD-002** mandatory preference bootstrap | Design Gap; Medium | Open → Resolved → **Resolved, retained** | Approved AM-001 removed separate preference GET; direct task GET with silent full defaults remains. |
| **TD-003** capacity/receipt growth | Architectural Risk; Medium | Open → Open → **Resolved (design)** | Numeric engineering hypothesis, finite admission/deadlines/pools, receipt growth, telemetry and future evidence thresholds supplied. |
| **TD-004** receipt privacy | Design Gap / Assumption; Medium | Open → Open → **Resolved (design)** | Independently keyed versioned HMAC receipts, historical-key replay, safe missing-key errors and qualified deletion wording supplied. |
| **TD-005** auth throttling | Design Gap; Medium | Open → Open → **Resolved (design)** | Shared atomic counters, trusted ingress/client identity, limits, generic 429/503 and fail-closed outage policy supplied. |
| **TD-006** recovery readiness | Architectural Risk; Medium | Open → Open → **Open, nonblocking R5 dependency** | Barry named accountable, Codex preparer; diagnostics, encrypted backup, keys/restore and migration drill contract specified, but operational owner sign-off and successful R5 drill absent. |
| **TD-007** receipt upgrade compatibility | Architectural Risk; Low | Open → Open → **Resolved (design); R4/Build evidence remains** | Versioned replay/old-key strategy, rollback preflight and owner/fixtures defined. No upgrade test claimed. |
| **TD-008** explicit-login session lifetime | Design Gap; Medium | New in Rev2 → **Open, narrowed blocker** | Forced same-user new session, locks, and rotation now defined. Competing cookie writes when no original live Session row remain under-specified despite an asserted invariant. |
| **TD-009** optional-view fault containment | Architectural Risk; Medium | New in Rev2 → **Resolved (design)** | Isolated bounded pool/executor, 150 ms full fallback deadline, cancellation/quarantine and task-independent recovery supplied. |

`Resolved (design)` is not evidence of implementation, security test, capacity measurement or release readiness. No TD risk is marked human-accepted. Prior **FD-003** and **FD-004** retain only their *previously* recorded R2 human-accepted consequences.

### TD-001 — Cross-tab authentication completion can outlive the focus gate

**Disposition:** **OPEN / High / blocker.** **Classification:** Correctness/Design Gap (client-side privacy). **Confidence:** High in the missing transition guarantee; actual exploitability depends on supported browser scheduling. **Owner:** Codex — UI/security design. **Requirements:** FR-010 and AC-010, with FR-009/014 refresh-failure handling.

**Evidence:** `technical-design.md:12–43` conceals private regions on initial render, blur/hidden/pagehide and focus return; revalidates account+epoch and snapshot before reveal; and makes cross-tab `BroadcastChannel` notifications explicitly best effort. `contracts/ui.md:105–126` repeats the gate, while `contracts/ui.md:90–92` and `:107–115` allow already-authorised focused tabs to retain visibly stale rows when a background refresh fails. The implementation test `quickstart.md:114` covers a logout/login before returning focus, but not a **login completion after revalidation**.

**Concrete counterexample not excluded by the specified mechanism:**

1. Account A is active in tab 1. The user initiates a login as B in tab 2; the network response is delayed. Tab 1 blurs and conceals correctly.
2. Before B's login commits/sets its cookie, the user returns focus to tab 1. Tab 1 checks the still-live A cookie/epoch, accepts an A snapshot, and legitimately reveals A's tasks.
3. Tab 2's delayed B login then commits and the shared browser cookie changes to B **without a further tab-1 blur/focus event**. As expressly permitted by the candidate design, pending/committed BroadcastChannel messages are lost or unavailable.
4. Tab 1 continues displaying A's task DOM under B's current browser authentication context until some later detection. If its next background read cannot complete, the normal *retain-visible-stale-rows* rule can prolong the exposure; epoch headers protect writes but do not remove already-visible A content.

Thus `conceal before each focus revalidation` does **not** establish the broader asserted invariant that an active tab cannot remain showing the previous account after a delayed cross-tab login completion. The mechanism guards **responses and submissions**, but not an auth-cookie change occurring after a successful authority check. This is a separate order-of-events concern from the R3-V01 happy ordering. It is a **design counterexample**, not a claim that a live browser exploit was executed.

**Minimum remedy:** Specify a cross-tab authentication-transition coordination/invalidation contract that remains safe when a login request is pending as a tab regains focus, and when notification delivery fails; do not require a particular library or architecture. Extend R3-V01 with delayed cookie finalisation *after* tab-1 A validation, lost notifications, failed subsequent reads and private DOM/accessibility assertions. If this cannot be enforced without revising FR-014's approved stale-data display or accepting a finite exposure window, explicitly escalate the product trade-off and seek the relevant human approval. **No risk acceptance exists now.**

### TD-008 — No-old-session concurrent login cookie ordering remains unspecified

**Disposition:** **OPEN / Medium / blocker, narrowed from Rev2.** **Classification:** Design Gap. **Confidence:** Medium-high. **Owner:** Codex — authentication/session architecture. **Requirements:** FR-010, FR-012 / AC-005; TD-001 authority-gate coupling.

**What is now corrected:** `technical-design.md:54–102`, `contracts/http.md:20–43` and `data-model.md:63–72` give a credible forced-new-key/epoch procedure even for same-user login; a Session-first lock prevents old view saves from resurrecting the removed login; ordinary maintenance rotation moves rather than resets the current view. This directly corrects the Django stock-login behaviour identified in R3-Rev2. PostgreSQL race tests remain downstream.

**Remaining gap:** `technical-design.md:71–81` says to lock the *original live Session row if present* and asserts that concurrent auth transitions do not overwrite the winner's session cookie. When two login form POSTs originate from an **anonymous browser with no existing Session row**, neither holds a shared original-session lock: both can create independent fresh sessions and issue valid `Set-Cookie` responses. If the earlier-committed response is delivered last, the browser's eventual cookie can change to that earlier account. No design rule identifies a winner or reconciles late cookie responses/redirected tabs. A row lock over an absent original row cannot, by itself, provide the stated cookie-ordering invariant. `quickstart.md:120` plans two-tab and delayed-cookie testing but contains no expected behaviour for this case.

This gap does **not** imply Django sessions are infeasible, nor that two independent new sessions automatically violate Scope. The blocker is the unsupported *claimed concurrency guarantee* and its overlap with TD-001's browser-identity protection. The design needs a consistent account/epoch outcome, not necessarily last-commit-wins ordering.

**Minimum remedy:** Define explicit safe semantics for competing login POSTs that share no prior live Session row, including stale/later `Set-Cookie` responses, and reconcile the server/browser authority gate with those semantics. Either bound/coordinate such transitions or deliberately specify response-order behaviour with mandatory clean client revalidation, clearing and tests proving no cross-account DOM/write confusion. Avoid adding unrelated-session revocation or new Scope policies. Extend R3-V08 with no-cookie, simultaneous different-account logins and reversed commit/response order. If a deliberate UX/security compromise is needed, escalate it before acceptance.

### TD-003, TD-004, TD-005, TD-009 — Resolved at static design level

- **TD-003:** `technical-design.md:147–206`, `plan.md`, `quickstart.md:115`: nominal 100 registered / 50 active accounts, 1,000 tasks/active account, 15-second visible polling, 1/4/10-million receipt fixtures, finite 2 MiB ingress/body limit, bounded four-worker/four-upstream admission, task/DB deadlines and monitoring. Receipt growth and concentrated-account load have measurable R5 acceptance/trigger criteria. The proposed 2 vCPU/4 GiB application and DB budgets are *hypotheses*, not demonstrated capacity. **Resolved design; R3-V03 must be measured in Build/Test and reviewed at R5.** Owner: Codex (performance design), Barry (R5 evidence accountability). Confidence high.
- **TD-004:** `technical-design.md:208–255`, `data-model.md:106–137`, `contracts/http.md:157–166,194–220`: HMAC-SHA-256 with independent long-lived key custody, canonical_version/digest_scheme/key_id metadata, receipt-first historical replay, missing-key known-no-effect 503, no re-keying, conservative language on deletion/metadata guessing. **Resolved design; enforce real secret separation and verify replay/key rotation at Build/Test.** The combined DB+key compromise remains an explicitly described threat, not a newly approved residual risk. Owner: Codex (security design), Barry (operational key custody sign-off at R5). Confidence high.
- **TD-005:** `technical-design.md:257–297`, `data-model.md:139–147`, `contracts/http.md:26–35`: private Django listener behind a single header-sanitising Nginx ingress, shared atomic PG counters, source/pair/service budgets, outage fail-closed 503, generic 429/Retry-After, bounded limiter pool and cleanup. **Resolved design; R3-V05/production ingress configuration and abuse tests remain outstanding.** Owner: Codex (auth design), Barry (R5 production/security evidence). Confidence high.
- **TD-009:** `technical-design.md:110–145`, `contracts/http.md:76–92`, `contracts/ui.md:43–47`: 150 ms optional view-read total, independent warmed read-only connections with fixed slots, transaction-local lock/statement/transaction timeouts, quarantine/circuit breaker, full silent fallback and independent mandatory auth/task errors. **Resolved design; R3-V09 must exercise relation locks, timeouts, late jobs and connection starvation on PostgreSQL.** Owner: Codex (session reliability design), Barry (R5 operational evidence). Confidence high.

### TD-002, TD-006, TD-007 — Historical closure and downstream gates

- **TD-002 — Resolved, unchanged:** R1-Rev4 and R2-Rev4 approved elimination of the separate blocking preference GET. `contracts/http.md:76–92` and `contracts/ui.md:41–62` continue direct bootstrap and silent all-four-default fallback. TD-009 was the separately assessed reliability mechanism, now resolved in writing.
- **TD-006 — Open, not an R3 blocker:** `technical-design.md:299–335` identifies **Barry Levy** as the accountable Operations/R5 owner and **Codex** as preparer. It specifies redacted request tracing and 14-day log retention *proposal*, daily encrypted consistent backups and 14-day retention *proposal*, key custody, session invalidation on restore, migrations, rollback/forward recovery, and a disposable R3-V06 drill. **R3 ownership/handoff condition satisfied; actual policy sign-off, access/configuration evidence and successful restore are required at R5.** No RPO/RTO or risk acceptance is implied.
- **TD-007 — Resolved design, with later evidence:** `technical-design.md:208–255,299–310` specifies frozen canonical format 1, versioned digest/key metadata, verify-only historical keys, upgrade fixtures and compatible rollback preflight. Barry is the named R4/upgrade accountable owner, Codex prepares fixtures. **R4 must confirm the compatibility/test handoff; Build/Test must execute R3-V07 before a relevant release.** A future incompatible upgrade reopens review, not a retrospective acceptance of broken replay.

## 4. Cross-cutting technical appraisal and regressions

| Area | R3-Rev3 assessment | Evidence boundary / follow-up |
|---|---|---|
| Scope/Functional Design trace | Task fields/defaults, per-owner access, four-setting session view, no separate preference GET, last server-committed view save, silent convenience failure, task-load write protection and existing sorting/date rules are preserved. | `spec.md:162–175,192–198`; `contracts/http.md:76–155,157–228`; `contracts/ui.md:25–103`. No new product rule approved. |
| Task integrity/concurrency | Per-owner account-list lock → receipt → task order, version preconditions, atomic receipt/revision/effect and snapshot generation remain coherent; historical receipt-first replays do not resurrect deleted tasks. | `data-model.md:26–35,82–137`; `contracts/http.md:157–228`; real PostgreSQL races, fail/rollback and restoration remain unexecuted. |
| Auth/session/security | Private ingress, CSRF, cookie security, epoch-gated task/view requests and targeted session-row lifecycle are plausible. | TD-001/TD-008 remain open on overlapping *browser-level* async auth transitions, not server-side owner FK isolation. |
| Fault recovery | Mandatory task read failures keep FR-014/FD-001 Retry/support and task-write lock; view-only failure safely chooses full defaults. Known historical receipt-key outage rejects new effects. | `contracts/ui.md:79–103`; `technical-design.md:110–145,227–240`; actual recovery tests later. |
| Runtime/platform/ops | Django 5.2 / Python 3.13 / PostgreSQL 17 + one Nginx ingress is proportionate; Postgres lock ordering/timeout budgets and bounded pools are specified. | `technical-design.md:52–206,257–335`; production deployment, patch/host constraints, monitoring and backup must be verified at R5. |
| Accepted prior risks | No-recovery credentials (FD-004) and interrupted memory-only pending-operation identity (FD-003) are unchanged. | R2 human acceptance retains its stated bounds. No TD risk is newly accepted or human-approved. |

**Nonblocking documentation note:** Several design documents retain pre-R2-Rev4 approval-pending headers beneath newer R3-Rev3 notices (`contracts/ui.md:9–11`, `quickstart.md:10–12`, `contracts/http.md:10–12`, `data-model.md:10–11`). The new notices correctly label that content historical, so there is no authority change; clean up stale authoring metadata when the next technical candidate is finalized. Do not rewrite immutable R2 review history. Earlier reviewer-owned checklists still contain previously acknowledged historical account-preference wording; treat as controlled documentation debt, not a substantive reintroduced endpoint.

## 5. Minimum targeted correction and evidence request

1. **TD-001:** Resolve the *focus-validates-A → later background B-cookie commit → signal absent → no new focus* ordering, including auth transition in progress and failed subsequent refresh; strengthen the state/sequence contract and R3-V01 accordingly, without exposing old DOM or weakening FD-001 silently.
2. **TD-008:** Define no-preexisting-Session concurrent login and delayed-cookie response outcomes; specify which response/cookie authority the page may reveal under, guard/correct any contradictory promise, and expand R3-V08 with reversed commit/response ordering. Keep same-user forced-new-login and view save/rotation safeguards.
3. Update only affected current `technical-design.md`, UI/HTTP contracts, data model/research/plan and quickstart scenarios as needed. Supply a succinct per-finding delta/evidence map, exact changed-file hashes and any upstream product-policy decision explicitly requiring Barry. Preserve canonical `spec.md`, human-approved R2 history, reviewer-owned past reports, accepted-risk history and unchanged task behaviours.
4. Return for **R3-Rev4 independent targeted verification**. Run no Build/Tasks/Analyze merely to close this gate. The reviewer will verify actual changes and inspect their immediate regressions rather than accept author assertions.

## 6. Exit gate and decision authority

| R3 exit question | Result |
|---|---|
| Correct R1-Rev4 Scope and R2-Rev4 approved baseline identified | **PASS** |
| Submitted exact file identities / preserved review histories verified | **PASS** |
| Technical remedies for TD-003/004/005/009 adequately specified | **PASS, static design only** |
| TD-002 historical resolution retained | **PASS** |
| TD-006 R5 and TD-007 R4 accountable handoff established | **PASS for R3 handoff; later gate evidence outstanding** |
| Privacy-safe cross-tab authentication transition across delayed commits | **FAIL — TD-001** |
| Consistent no-prior-session simultaneous login/cookie response ordering | **FAIL — TD-008** |
| Runtime code, browser, concurrency, load, backup/restore, production checks | **NOT EXECUTED — appropriate downstream evidence** |

**Formal R3-Rev3 independent verdict: NOT READY.** Blocking TD-001 and TD-008 have concrete unresolved ordering paths. All other prior R3 technical blockers are corrected sufficiently in the written candidate. The report does not grant R3 human approval, does not accept residual exposure or an auth-cookie race and does not authorise Tasks/Analyze, Build, deployment or relaxation of Scope/Functional Design. The correct next gate is targeted R3-Rev4 review after author corrections, followed—only if independently ready—by **Barry's explicit human R3 decision**.

## 7. Review-history ledger

- **R3-Rev1:** Initial **NOT READY**; TD-001–007 open.
- **R1-Rev4 / R2-Rev4:** AM-001 Scope and Functional Design independently reviewed and explicitly human-approved; FD-003/004 accepted prior risks retained.
- **R3-Rev2:** **NOT READY**; TD-002 resolved by approved supersession, TD-001/003–007 open, TD-008/009 added open.
- **R3-Rev3 (this review):** Submitted technical mechanisms and manifest independently inspected. TD-003/004/005/009 resolved as written; TD-007 design resolved with R4/Build evidence; TD-006 R3 ownership handoff fulfilled but operational evidence open at R5; TD-002 stays resolved; TD-001 and TD-008 remain open blockers for narrower concurrent browser login orderings. **No earlier report or canonical artifact was altered, no human approval inferred.**
