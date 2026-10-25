# Taskify — Independent R3-Rev4 Technical Design Review

**Review date:** 2026-10-23 (Australia/Melbourne)  
**Reviewer:** Independent Technical Design Reviewer (R3)  
**Independent verdict:** **NOT READY — targeted technical correction required**  
**Human R3 gate:** **OPEN — not approved; no new risk accepted**  
**Method:** Static architecture, concurrency, security and design-evidence review. No browser, application, database, capacity, backup/restore or deployment test was executed.

## 1. Executive conclusion

The R3-Rev4 candidate is a substantive technical improvement. It removes the rejected Web Locks proposal, specifies three-stage authentication (PREPARE → ISSUE → ACTIVATE), makes authentication tickets attempt-specific, and serializes a returning tab's authority claim against an already-dispatched activation. **When the browser retains and presents every relevant transition cookie**, the written ordering addresses the exact delayed-activation race raised in R3-Rev3. No new Scope or Functional Design amendment is presently indicated.

**The proof is not yet complete.** Its pivotal assumption—an activation request having carried a marker/ticket means a later request from another tab must also carry those cookies—is not guaranteed by the HTTP cookie model. Cookies can disappear between two request dispatches. In that case the returning tab's claim can validate and reveal account A while an *already-dispatched but unobserved* activation of account B subsequently commits. That reproduces the forbidden stale-private-content ordering. The attempt-specific-cookie and high-water scheme also needs a safe selection/failure rule under partial loss, cookie limits and cleanup, not merely under out-of-order delivery.

**Blocking dispositions:** TD-001 remains **Open / High**, narrowed to incomplete cookie presentation; TD-008 remains **Open / Medium**, narrowed to transition visibility and no-fallback guarantees under partial cookie loss/retention. TD-002/003/004/005/007/009 retain their independently verified **Resolved (written design)** dispositions. TD-006 remains **Open, nonblocking R5 evidence dependency**, with R3 accountability assigned. No new finding ID is necessary: the remaining gap is within TD-001/008. All runtime validations remain outstanding.

## 2. Verified input identity and control of baselines

| Item | Independent check / boundary |
|---|---|
| Submitted package | `Archive(5).zip`, SHA-256 `c48658a5cd99244971a1a89985c0cb84531febf947551d9dae57b229b4e5e4cf` |
| Approved Scope v1.1 / AM-001 | `specs/001-personal-task-manager/spec.md`, SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`, matches prior approved identity. R1-Rev4 remains historically human APPROVED. |
| Approved Functional Design | Independent R2-Rev4 report §8 records Barry Levy's later human **APPROVED** decision. The nine-file historical manifest is not retroactive approval of subsequently edited technical design documents. Both approved checklists and `am001-r2-review-handoff.md` independently match their recorded hashes. |
| Prior R3 review history | The R3-Rev2 and R3-Rev3 reports match their earlier SHA-256 identities (`1c046240...2a1bcd77` and `593ed588...39afd40e67` respectively). Previous findings and human gate decisions remain unchanged. |
| Current submission | **All nine** files in `review-followups/r3-rev4-changed-files.sha256` were individually re-hashed from the supplied archive and match. `technical-design.md`: `6186352ffd37d15571df56c27a0e71cf98851ff8b0c1b8066d6f3411e44494be`; new `auth-transition-protocol.md`: `4ef134edf329fe1bc881f5368ec67d2beb7fa0c4ac1fa2f07877c31e970ce441`. |
| Retained mechanisms | Independently compared `technical-design.md` §§3–8 against the supplied Rev3 package: **byte-identical**. Accordingly, historical design-level resolutions are retained absent an identified interaction regression; no runtime proof is inferred. |

Relative evidence paths below are under `specs/001-personal-task-manager/` unless otherwise stated. The author correction map is evidence of stated intent, **not** independent proof. The new protocol is an operative candidate specification, not a built-in Django/HTTP guarantee.

## 3. Stable finding ledger

| Finding | Rev3 → **Rev4 independent disposition** | Severity / evidence boundary |
|---|---|---|
| **TD-001** — Cross-tab private content | Open → **OPEN, BLOCKER** | **High**. Atomic claim correct for fully presented proofs; fails to cover already-sent activation whose proof cookies are missing from a later claim. |
| **TD-002** — Preference bootstrap | Resolved → **RESOLVED, retained** | Medium. Approved AM-001 direct task bootstrap/silent defaults remain. |
| **TD-003** — Workload/capacity | Resolved (design) → **RESOLVED (design)** | Medium. Bounded budgets/measurements retained; real measurements due downstream. |
| **TD-004** — Receipt privacy | Resolved (design) → **RESOLVED (design)** | Medium. Versioned HMAC receipts/key custody/qualified deletion retained; runtime/key tests due. |
| **TD-005** — Auth throttling | Resolved (design) → **RESOLVED (design)** | Medium. Trusted ingress/shared atomic limiter/fail-closed path retained; deployment verification due. |
| **TD-006** — Operational recovery | Open, nonblocking → **OPEN, NONBLOCKING R5 DEPENDENCY** | Medium. Barry accountable, Codex preparer; no approved policy, successful restore or production evidence yet. |
| **TD-007** — Upgrade compatibility | Resolved (design) → **RESOLVED (design)** | Low. R4 fixture handoff and Build/Test upgrade/rollback proof remain due. |
| **TD-008** — Explicit-login lifecycle | Open → **OPEN, BLOCKER** | Medium. Distinct cookies/staging/ledger correct ordinary response-order gap; safe selection under missing cookies and bounded retention not demonstrated. |
| **TD-009** — Optional view read isolation | Resolved (design) → **RESOLVED (design)** | Medium. 150 ms optional fallback/isolation retained; concurrency/DB tests due. |

Previous **FD-003** interrupted memory-only pending-operation identity and **FD-004** absence of credential recovery retain only their earlier, explicitly human-accepted R2 consequences. **No TD risk is accepted** merely by being documented, assigned an owner or included in a validation scenario.

### TD-001 — The atomic claim can miss an already-sent activation

**Classification:** Correctness Defect / Design Gap. **Severity:** High. **Confidence:** High that the design relies on an unsupported cookie-retention premise; specific browser reproduction remains unexecuted. **Disposition:** **Open, R3 blocker**. **Owner:** Codex (authentication/client architecture); Barry only for any required product-policy/risk trade-off. **Requirements:** FR-010, AC-010; coupled FR-009/014 failure behaviour.

**Design evidence:** `auth-transition-protocol.md:26–40` checks focus and then states at lines **32–39** that if B ACTIVATE was dispatchable, its marker and ticket are still in the cookie jar when A's claim runs. `technical-design.md:16–38` and `contracts/http.md:43–72` elevate this to the proof that one transaction must observe/cancel the other. The planned tests `quickstart.md:130–150` reverse headers and callbacks but do not assert partial removal between ACTIVATE dispatch and another tab's claim.

**Concrete counterexample left possible by the written mechanism:**

1. Account **A** has a valid existing ticket; tab B reaches ISSUE for account **B**, obtains its attempt-specific pending marker and ticket, and **dispatches ACTIVATE** carrying both cookies. Hold the request before its database lock/commit.
2. **After dispatch**, the browser selectively removes/omits B's marker and ticket (for example under cookie controls/storage pressure) while A's old ticket remains. B's already-sent request still contains valid copies of its proofs. No B ACTIVE high-water cookie was delivered yet: ACTIVATE has not committed/responded.
3. A's returning tab invokes `POST /api/session/claim/`. Its Cookie header presents **A but not B**. The specified claim locks only *presented* transition records; it therefore cannot discover or cancel B's in-flight activation. It commits A authority, gets an A snapshot and reveals A's private rows.
4. Resume B ACTIVATE. Its pre-dispatch proofs still validate, and no B transition was cancelled by A's claim. The activation may commit B and revoke A. The already-focused A tab has **no new focus gate**; with lost broadcasts and failed subsequent task refresh, its stale A rows can remain visible, recreating the Rev3 failure.

This is a **design-level event-order counterexample**, not a claim that it occurred in a real browser. HTTP cookie standards explicitly permit cookies to be omitted, rejected or evicted; they do not promise that a cookie observed when one request is dispatched remains available for another request. See RFC 6265 §§5.3, 5.4 and 6.1: https://www.rfc-editor.org/rfc/rfc6265.html . The design's rule that rejected/missing cookies cannot produce *login success* does not close this gap because **B's activation request already carries valid cookies while A's separate authority claim lacks them**.

**Minimum correction:** Establish a defensible **cross-tab authority/activation ordering invariant even when a later claim presents only a subset of the cookies previously sent by another request**. Define a fail-closed outcome with no previous-account reveal when server authority is uncertain; do not assume a browser feature, accepted exposure interval or weaker task-load failure rule. Extend the exact R4-V01a/b test with deliberate selective cookie loss/omission *after B ACTIVATE dispatch*, A claim first, B commit second, dropped broadcasts, failing subsequent reads, DOM/accessibility observation and writes rejected under stale epoch. If not achievable within approved browser/UX scope, explicitly elevate the necessary trade-off to R2/Scope and Barry rather than silently accepting it.

### TD-008 — Attempt-specific cookie visibility/selection is not a reliable browser-wide ordering proof

**Classification:** Design Gap / Architectural Risk. **Severity:** Medium. **Confidence:** High for the unproven assumption; medium for any additional cookie-capacity impact (unmeasured). **Disposition:** **Open, R3 blocker**, coupled to TD-001. **Owner:** Codex (auth/session architecture). **Requirements:** FR-010/012, AC-005; supported browser access FR-015.

**What is corrected:** `auth-transition-protocol.md:60–136`, `contracts/http.md:49–86` and `data-model.md:82–111` establish a plausible three-phase new-session lifecycle, distinct-name tickets, ledger activation order, SessionMiddleware exclusivity, same-user fresh epoch/defaults, rotation transfer, shared logout revocation, and rejection of late cancelled tickets. In a stable full-cookie presentation, later ACTIVATE dispatch observes prior tickets, and claims can serialize competing anonymous logins without an original Session row. This **narrows** the Rev3 problem materially.

**Remaining mechanism gap:** Cookie-based discovery/ordering is restricted to records *presented by the browser* (`auth-transition-protocol.md:79–103`). The scheme stores multiple attempt-named auth, marker and high-water cookies; its no-fallback assurance depends on the newest authority evidence not selectively disappearing while an older still-valid ticket remains. The terms **“highest acknowledged floor”**, **“until ... delayed responses cannot validly return”**, and **“bounded cookie parsing/admission and cleanup”** give no concrete safe rule when some transition cookies disappear after a request has been sent, nor a specified measurable bound/retention policy for attempts and floors. A `Cookie` header has no completeness proof. `quickstart.md:135–138` mentions limits/rejection but does not state or exercise the critical **post-dispatch selective-loss** ordering.

**Minimum correction:** Provide a coherent rule for incomplete cookie sets and related server ledger lifetimes—especially simultaneous anonymous logins, a previously sent activation, high-water loss and old-ticket fallback. Ensure missing cookies cannot silently validate an older account against a later in-flight activation. Specify sufficient bounded cleanup/capacity assumptions with cross-browser testable acceptance conditions. Retain unrelated-browser/device isolation, ordinary multi-tab access, fresh explicit-login defaults and shared logout behaviour. No specific redesign is mandated; a new browser restriction or privacy-risk acceptance would require human direction.

## 4. Other architecture and regression checks

| Area | Independent assessment and required downstream evidence |
|---|---|
| **Scope/R2 integrity** | Session-scoped four-part view, all-or-default silent fallback, task-first bootstrap, same-session view save ordering, and per-owner private tasks retained. The staged login is internal design work; no approved user policy change identified. |
| **Auth/security** | CSRF, signed opaque markers, HttpOnly tickets, independent browser sessions, server-authorized provisional sessions and highest-ordinal selection are plausible. **TD-001/008 remain open** because cookie presentation is not a reliable concurrency fence. Browser paint/DOM tests have not run. |
| **Data and concurrent writes** | Transition → Session → view lock order and distinct account-state → receipt → task mutation order documented. Account ownership, revision/version guards and historical receipts are unchanged. Real PostgreSQL deadlock, rollback, race and session-store resurrection tests remain due. |
| **Fault and recovery** | POST claim is a mandatory authority revalidation, not optional view persistence; failed claim conceals data. Existing FR-014/FD-001 same-epoch task refresh state and writes lock are retained in writing. Capacity/timeout effect of adding claim and staged auth must be measured later. |
| **Runtime and operational fit** | No new service or browser-only Web Locks requirement, but a new ledger plus multiple transient cookies and more auth round trips increases complexity. Authenticate delivery, cookie limits/cleanup, primary-pool/deadline budgets and resilience in supported browser engines before Build/release. R5 restore/key/ingress policies remain unexecuted. |
| **Documentation** | Old R2-Rev4 approval metadata is historically accurate. Historical preapproval snippets within unchanged older sections were previously noted as nonblocking editorial debt. No historical reports were edited. |

## 5. Minimal author correction / independent re-review request

1. **TD-001:** Close the missing-proof activation ordering, not just the full-cookie case. Demonstrate the returning claim either observes/cancels all still-committable activations affecting its selected login **or remains concealed**. Preserve approved stale-snapshot semantics and no exposure-window assumption.
2. **TD-008:** Specify secure account/session selection when attempt cookies or floors are partially removed, rejected or omitted, with concrete high-water/cleanup and cookie-capacity rules. Cover separate anonymous roots, expiry/late headers and shared logout.
3. Update only affected current technical design/ADR, HTTP/UI contracts, data model, plan/research and targeted validation cases as needed. Provide a per-finding evidence delta and exact changed-file hashes; preserve approved `spec.md`, R2/older R3 reports, accepted-risk record, current resolved TD mechanics and unrelated requirements.
4. Return for **R3-Rev5 targeted independent assessment**, with no Tasks/Analyze/Build or presumed human risk acceptance. Browser and PostgreSQL tests remain planned downstream unless Codex supplies genuine execution evidence separately.

**No blocking human product decision is required merely to send these engineering gaps back to Codex.** If a correction requires reducing browser coverage, weakening private-content isolation, changing logout semantics or altering the approved task-load failure experience, stop and seek Barry's explicit choice through the appropriate R1/R2 amendment/risk process.

## 6. Formal gate / history

**Formal R3-Rev4 independent verdict: NOT READY.** TD-001 **High** and TD-008 **Medium** remain technical-design blockers on one coupled cookie-presence assumption. TD-002/003/004/005/007/009 remain **Resolved at design level only**; TD-006 remains a **nonblocking R5 operational-evidence dependency**. No runtime tests were performed, no new risk accepted and no human R3 approval inferred. **Do not advance to Spec Kit Tasks/Analyze or Build on this candidate.**

| Review | Recorded progression |
|---|---|
| R3-Rev1 | NOT READY; TD-001–007 open. |
| R1-Rev4 / R2-Rev4 | AM-001 scope and amended Functional Design separately human **APPROVED**. |
| R3-Rev2 | NOT READY; TD-002 resolved; TD-008/009 added; others open. |
| R3-Rev3 | NOT READY; TD-003/004/005/007/009 design-resolved; TD-006 nonblocking owner/R5 dependency; TD-001/008 open. |
| **R3-Rev4 (this review)** | Nine submitted hashes and unchanged baseline/history verified; staged protocol independently analyzed. Old races largely closed under full-cookie presentation, but TD-001/008 still **Open** under partial cookie loss. **Human gate OPEN**. |

