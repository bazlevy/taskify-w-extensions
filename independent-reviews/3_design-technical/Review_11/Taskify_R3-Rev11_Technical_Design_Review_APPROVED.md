# Taskify — Independent R3-Rev11 Technical Design Review

**Independent review date:** 2026-10-25 (Australia/Melbourne)  
**Candidate reviewed:** Codex R3-Rev10 / ADR-AS-004 native-session technical design  
**Independent verdict:** **READY FOR HUMAN APPROVAL**  
**Subsequent human R3 gate:** **CLOSED — APPROVED** (see §8)  
**Review boundary:** Written architecture, interfaces, failure ordering, risk proportionality, regression plans and artifact integrity. No software implementation, runtime testing or deployment.

## 1. Executive decision

The R3-Rev10 candidate resolves the three open technical design gaps under approved Scope v1.2 / AM-002 and R2-Rev8. Django/PostgreSQL sessions remain the sole authentication authority. A bounded Session-row guard serializes protected effects with logout; a restricted response adapter prevents stale non-login auth-cookie writes; client identity/generation checks enforce known-state safety; and epoch-scoped BroadcastChannel notification addresses confirmed logout without a delivery guarantee.

The design preserves account isolation, intended-login integrity, session-scoped view state, task revisions and mutation receipts, existing failure/uncertainty rules and the specifically accepted unobserved-display residual. It does **not** reinstate superseded first-browser-winner, atomic-render, guaranteed-notification or mandatory browser-coordination requirements.

**Independent conclusion:** No substantiated material design defect remains. TD-008, TD-010 and TD-011 are **RESOLVED_VERIFIED at design level**. Zero R3 blocking findings remain. The remaining implementation and runtime checks are assigned to subsequent authorized phases. No new Scope, Functional Design or material risk-acceptance decision is required.

## 2. Evidence and baseline integrity

| Artifact / verification | Independent result |
|---|---|
| Submitted `Archive(1).zip` | SHA-256 `d582d23efb676ef7df77f07353c584a36d7359f3333bf3c08c90fc904415e1a4` |
| Canonical Scope v1.2 / AM-002, `spec.md` | SHA-256 `688b57c11785af800b072843d7763c14875dbc6cbb5b35594201936cee03549b`; unchanged |
| Human-approved R2-Rev8 report | SHA-256 `15e93e9da14b4ce7badd9ddf6e557cb7961ae5a6d01b0ea950102d997fb627f1`; unchanged |
| Approved UI contract | SHA-256 `da3fabe5cb3b403b05e466a5ea0401e724b8116a1fcf04d5544deed1f43dc39d`; unchanged |
| Approved HTTP contract | SHA-256 `ce2c48e635207bed5a9a79aca9222819c380426a0f0802b9e9795ad1b5035637`; unchanged |
| R3-Rev10 integrated `technical-design.md` | SHA-256 `41ce09515e7ef3054a8aac3606e8991576f4e4b59d36b5eaab82aa27ef4413c2` |
| R3-Rev10 `auth-transition-protocol.md` | SHA-256 `229dba1ecbbd09d868c6c15069284a350087b98ae2c776a600fd032176413201` |
| R3-Rev10 deterministic validation plan | SHA-256 `ecb87cc9af7e2425da25b8a1c6fb6e9640ef90704815a6f2cc242d9f323ebad6` |
| R3-Rev10 changed-file manifest | **21/21** entries independently rehashed and matched |
| Approved CA-T01–16 acceptance rows | Byte-identical to preceding archive |
| `technical-design.md` §§3–8 | Byte-identical to preceding archive |
| Prior independent reviews and approval records | Unchanged by comparison with preceding archive |

The seven modified pre-existing substantive documents are `quickstart.md`, `auth-transition-protocol.md`, `research.md`, `data-model.md`, `plan.md`, `technical-design.md` and `checklists/feature.md`. New historical snapshot, evidence, validation and handoff artifacts are appropriately separated. This verifies document identity and scope, **not** execution of the written scenarios.

## 3. Finding TD-008 — Authentication lifecycle and logout correctness

**Original classification:** Design Gap; **severity:** Medium; **likelihood:** unmeasured; **confidence:** High in the design-level mechanism; **action:** prior BLOCK → now NOTE for downstream verification; **status:** **RESOLVED_VERIFIED (design-level)**.

**Binding requirements:** CA-007, CA-008, R2-NAV-001, FR-010/AC-010, CA-T01–05/10/13–15, and preserved task/view protections.

**Failure being prevented:** A command authenticates under login L, waits, and commits after another tab has received authoritative logout confirmation for L; or a delayed logout/error response deletes a subsequently selected N cookie; or absence of L is mistaken for proof of a successful intended logout. Wrong-owner or post-revocation mutations can have high integrity consequences; late cookie deletion generally causes recoverable continuity loss. Exposure is unknown; these are reasoned design scenarios, not observed incidents.

**Verified design:** The Session row is locked and reauthenticated before owner/operation/task/view access; commands hold that guard through commit, while logout holds it through revocation commit. Command-first and logout-first transactions therefore have a coherent order. The response adapter permits an auth-cookie write only for a committed fresh login, avoiding stale non-login saves/deletions. A compact logout receipt permits exact read-only confirmation following a lost acknowledgement without permitting token-only revocation or binding old intent to N. Native session encoding, login fixation defence, authentication checks and CSRF remain in use. Normal authenticated navigation does not change key/epoch/view; genuinely anonymous concurrent logins follow the approved browser-cookie order.

**Baseline/proportionality:** Django's supported session/auth functions and PostgreSQL row locks are suitable foundations. The additional Session guard and response discipline address concrete ordering problems; family/handle registries, multistage login, periodic rotation and mandatory browser coordination are no longer justified. The custom adapter's exact implementation remains subject to maintained-patch compatibility testing. See [Django sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/), [SessionMiddleware source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/middleware.py), [Django auth source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/__init__.py) and [PostgreSQL row locks](https://www.postgresql.org/docs/17/explicit-locking.html).

**Exit evidence:** Written transaction orders, cookie-writer matrix, receipt conditions and CA-T01–05/10/13–15 plus R10-X01–03 establish design coherence. Execute the real-database and browser integration scenarios during Build/Test. No additional R3 design blocker.

## 4. Finding TD-010 — Known-stale client state and private reveals

**Original classification:** Design Gap; **severity:** Medium; **likelihood:** unmeasured; **confidence:** High; **action:** prior BLOCK → now NOTE for downstream verification; **status:** **RESOLVED_VERIFIED (design-level)**.

**Binding requirements:** CA-006, approved AC-010/RA-FD006-001, Q1 and CA-T05–09/13/15.

**Failure being prevented:** An A response or callback, already known to be stale after selection of N, restores A's private state or rebinding of an A intent to N. Such known-mismatch exposure or wrong-login effects remain prohibited. By contrast, previously authorised A content that finishes rendering before a cross-tab change is observed is explicitly accepted by Scope v1.2, with no finite observation deadline.

**Verified design:** Per-tab known account/epoch, `authGeneration`, `queryGeneration`, revision floors and captured dispatch identities reject stale callbacks. Observed changes clear rows, labels, drafts, dialogs and queues. Initial/focus/resume/history/uncertainty boundaries conceal and revalidate. Server protected operations independently enforce owner and intended epoch. Ordinary task refresh failure keeps FD-001 write locks, while optional view failure stays silent. No per-interaction independent authentication call or atomic browser rendering fence is introduced.

**Exit evidence:** CA-T05–09/13/15 and R3-V01 provide appropriate known-versus-unobserved and task/view regression plans. The approved bounded privacy residual is unchanged. No further Scope/R2 decision or R3 correction is needed.

## 5. Finding TD-011 — Confirmed cross-tab logout notification

**Original classification:** Design Gap; **severity:** Medium; **likelihood:** unmeasured; **confidence:** High; **action:** prior BLOCK → now NOTE for downstream verification; **status:** **RESOLVED_VERIFIED (design-level)**.

**Binding requirements:** CA-008 and CA-T10–14.

**Failure being prevented:** A runnable tab receiving confirmed logout L does not clear/navigate promptly, or a late L message incorrectly clears a newer N view, or an unknown logout outcome is broadcast as confirmed. Potential confidentiality harm from a failed handler is serious; transport-only delivery gaps are within the approved limitation.

**Verified design:** A trusted committed/replayed L acknowledgement triggers prompt epoch-only BroadcastChannel notification. A receiving runnable L tab clears and navigates in its handler. Duplicates are harmless; late L notifications do not revoke N. Missed/unavailable messages and suspended/history restoration use existing concealed revalidation and visible synchronisation. No delivery guarantee, tab acknowledgements, faster polling or mandatory browser capability is assumed. [BroadcastChannel reference](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API).

**Exit evidence:** CA-T10–14, R3-V01 and relevant browser lifecycle/CSRF scenarios cover the design. Execute in supported browsers during Build/Test. No further R3 correction is needed.

## 6. Retained findings and regression obligations

| Finding | Independent status / subsequent responsibility |
|---|---|
| TD-001 | Resolved; repeat account-isolation regression under native session guard |
| TD-002 | Resolved; no substantiated new issue |
| TD-003 | Resolved at design level; capacity/session-lock measurements at later validation |
| TD-004 | Resolved; command-receipt privacy/integrity tests retained |
| TD-005 | Resolved; authentication-throttle tests retained |
| TD-006 | **OPEN, nonblocking**; Barry accountable for Operations/R5 restore, backup and diagnostic evidence, now including logout-receipt cleanup |
| TD-007 | Resolved; Barry accountable for R4/future upgrade compatibility, including adapter/rekey preflight |
| TD-008 | **RESOLVED_VERIFIED (design-level)**; Build/Test lifecycle verification |
| TD-009 | Resolved; optional view failure-isolation regressions retained |
| TD-010 | **RESOLVED_VERIFIED (design-level)**; Build/Test client-state verification |
| TD-011 | **RESOLVED_VERIFIED (design-level)**; Build/Test messaging/lifecycle verification |

FD-003/004 and RA-FD006-001 remain within their exact previously approved boundaries. R2-Rev7's historical NOT_READY and prior R3 findings/reviews are not retrospectively altered. No new finding ID is justified.

## 7. Nonblocking considerations and independent verdict

1. **Middleware compatibility — DEFER.** The response adapter and read-only authentication facade must be verified against the pinned Django patch and on upgrades. Their intended semantics and failure responses are sufficiently specified for R3; runtime verification belongs to Build/Test and compatibility releases.
2. **LogoutReceipt proportionality — NOTE.** Exact recovery after a lost logout acknowledgement is useful but is stronger than the minimum contract, which permits an unknown outcome. A simpler design could leave the outcome uncertain. The selected bounded receipt has no substantiated unacceptable security/integrity consequence, so its removal is **not** an R3 approval condition. Monitor operational cost/complexity without converting this optional consideration into a new product requirement.

**Independent verdict: READY FOR HUMAN APPROVAL.** There are **zero open R3 blockers**, no substantiated need for material redesign and no new human Scope/Functional Design/risk decision. The R3-Rev10 candidate provides sufficient coherent written design evidence; application tests are planned, not performed. TD-006 remains an Operations/R5 dependency and TD-007 retains its R4/maintenance obligations.

## 8. Subsequent human R3 approval — 2026-10-25 (Australia/Melbourne)

After receiving the independent READY verdict for this exact R3-Rev10 candidate, Barry Levy responded **“approved”** in the conversation on 2026-10-25. This is the explicit human gate decision, separate from and subsequent to the independent review. The time of day is not asserted.

| Decision field | Formal outcome |
|---|---|
| Decision maker | Barry Levy, human product owner |
| Declaration | “approved” |
| Human R3 gate | **CLOSED — APPROVED** |
| Approved candidate | R3-Rev10 / ADR-AS-004 native-session technical design, identified by §2 artifact hashes and submitted archive |
| Independent recommendation | R3-Rev11 **READY FOR HUMAN APPROVAL**, §§1–7 of this report |
| Finding dispositions | TD-008/010/011 RESOLVED_VERIFIED **at design level** |
| Retained obligations | FD-001/003/004, RA-FD006-001, other resolved-TD regressions, TD-006 Operations/R5, TD-007 R4 compatibility, Build/Test execution |
| Excluded | Implementation success, runtime verification, deployment, new product risk acceptance or automatic R4/R5/Build approval |

**Authorised workflow progression:** Record this independent report and human gate without changing the approved design; proceed to Spec Kit Tasks and Analyze, then R4 Integrated Product Design Review and R5 Pre-Build Readiness Review. Human Build approval remains separately required. No Tasks/Analyze/Build work was performed by this independent review.
