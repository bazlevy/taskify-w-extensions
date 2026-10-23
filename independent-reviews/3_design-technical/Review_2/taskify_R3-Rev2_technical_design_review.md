# Taskify — Independent R3-Rev2 Technical Design Review

**Review date:** 2026-10-23 (Australia/Melbourne)  
**Reviewer:** Independent Technical Design Reviewer (R3)  
**Independent verdict:** **NOT READY — R3 corrections required**  
**Human R3 gate:** **OPEN — no R3 approval or new risk acceptance inferred**  
**Review type:** Evidence-based static architectural re-review. No application source, migrations, running service, deployment, load tests, browser tests or restore drill were supplied or executed.

## 1. Executive conclusion

The amended Scope and Functional Design approvals are established. The current Scope file and **all nine design/handoff files** identified by the R2-Rev4 approval manifest match their recorded SHA-256 hashes. The archive is a repackaging; it need not match the older R2 ZIP hash to establish the identity of the newly approved R2-Rev4 *individual files*.

The one-service Django/PostgreSQL approach remains proportionate and technically plausible for a personal task manager. Task ownership, account-list revisions, transactionally durable mutation receipts, optimistic task versions, and snapshot/retry semantics are coherent as written. These are design properties, **not demonstrated implementation results**.

**TD-002 is resolved by an approved product/design amendment:** the mandatory standalone preference GET is gone; the first task GET falls back silently to a complete default view if session view data is unavailable. This is not a claim that every possible fault in the new session-view mechanism is solved.

**TD-001 and TD-003–TD-005 remain materially open.** TD-006 remains an unassigned operational handoff and TD-007 an open upgrade-compatibility item. Two AM-001-specific technical gaps are newly recorded: **TD-008** (same-user explicit re-login does not automatically create a new session lifetime under stock Django login) and **TD-009** (the best-effort view-read path lacks a bounded fault-containment mechanism). Consequently R3 is **NOT READY**; the design must return for targeted technical corrections and independent verification before a separate human R3 decision. No new Scope or Functional Design amendment is presently demanded by these findings.

## 2. Controlled baseline, evidence and boundaries

### 2.1 Exact inputs and approval provenance

- **Reviewed upload:** `Archive(3).zip`, SHA-256 **`e2d3ad1f066df835e6a37caf3a46c99b8fec5b718d92b98ceeda8f9ea20597d6`** (locally recomputed). This contains the R1, R2 and R3 review histories, approved Scope and current design documents.
- **Scope:** v1.1 / AM-001, human **APPROVED** in `independent-reviews/1_scope/Review_4/scope_review_report.md` §8, for the approved candidate hash `61062311e5d1423f1d13f485c555158964e09f0548d13263bfefa0f048447fd4`. The canonical metadata-updated `spec.md` SHA-256 is **`b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`**, independently recomputed here. R1-Rev4 explains the metadata-only distinction and the deferred archival R1-AM-004 issue.
- **Functional Design:** R2-Rev4 reviewer recommendation **READY_FOR_HUMAN_APPROVAL**, followed by a separately recorded Barry Levy **APPROVED** decision on 2026-10-23, in `independent-reviews/2_design-functional/Review_4/taskify_R2-Rev4_functional_design_review.md:136–170`. The following supplied files were independently hash-checked **PASS** against its approved manifest (same relative directory as the Scope file):

| Approved R2-Rev4 file | Independently reproduced SHA-256 |
|---|---|
| `plan.md` | `b55d38011604e0d93578cbf300b8ecb177fa4a16fbec759971e77b47f8170224` |
| `research.md` | `3b7958f2e52af060621c77453b362f686c77eba5e343c60ef4f0d5d96c2a3a8d` |
| `data-model.md` | `44fe7219af32708ea1afd76a049b7b413e54d606e4f06d0e18733c479a5c2cdc` |
| `contracts/http.md` | `b179d110424b71af81789e5d9ec2fd8cca6981e8a08da9370f7c3a9cd0213a5a` |
| `contracts/ui.md` | `7d617894547a5b06c878b09304d85c5424881dc69cce111690ea0ba8d56ec266` |
| `quickstart.md` | `ee774bee1fcdb832eee99ac7e96417206cd6ac377283f7ac252d871abab8c7cb` |
| `checklists/feature.md` | `f1d97f2a603129faee2176fa2422f85fa1c73022a10c148ee7f00128eee0917f` |
| `checklists/requirements.md` | `68e439444144a050894246f28538cad798eba01154607dbd1f63e219209f99ec` |
| `review-followups/am001-r2-review-handoff.md` | `da4899f7471d9f59f91d43b08bb1e2949924f8c64aaf46fd9240782d6de0bf3a` |

- **Prior R3 review:** `independent-reviews/3_design-technical/Review_1/taskify_R3-Rev1_technical_design_review.md`, **NOT READY**, with TD-001–007 originally **OPEN**. Its Addendum A directed controlled amendment and allowed TD-002 closure if the old bootstrap was demonstrably superseded. The superseded proposal/correction-plan files in `review-followups/` are **not operative technical corrections**.
- **Approval accounting:** The R1/R2 approvals apply to the identified baselines. R2-Rev4 itself explicitly leaves TD-001–007 to R3. Historical FD-003 (lost in-memory operation identity on interruption) and FD-004 (no credential recovery/transfer) remain human-accepted risks, **not resolved**. R1-AM-004 remains **DEFERRED** archival evidence, not a technical blocker.

All paths and `file:line` references below are relative to `specs/001-personal-task-manager/` within the upload unless otherwise stated. Line numbers refer to those exact hash-verified files. External primary-source references are listed in §8.

### 2.2 Review limitation and architectural reconstruction

There is no production environment or implementation to audit. No empirical statements about security, browser support, capacity, restore success or concurrency tests are made. Design-time technical decisions are assessable now; test execution, configuration and operational demonstrations belong to later gates. Neither a product task-count cap nor a numeric SLA/RPO/RTO is invented.

**Topology:** A Django 5.2 LTS / Python 3.13 application with templates and ES modules, Gunicorn and PostgreSQL 17 through Psycopg 3; standard database sessions; HTTPS reverse proxy not yet selected (`plan.md:21–52,92–120`). No cross-service queue, websocket, managed auth or browser-persistent task cache.

**Trust/data flow:** Browser cookie + CSRF → authenticated HTTP adapter → domain/application services → owner-scoped ORM repository → PostgreSQL. The server derives account and session identity; it does not accept caller-provided owner IDs. Persistent state consists of User, account TaskListState, Task, MutationReceipt, Django Session and new **SessionViewState** linked to a still-valid session key (`data-model.md:10–30,32–71,73–112`; `contracts/http.md:3–40`).

**Task write:** Lock account list-state row → check owner receipt before task lookup → validate task version → commit task/revision/receipt atomically → acknowledge → read at least the acknowledged revision. This deals with concurrent edits and a lost create response followed by another device deleting the task (`contracts/http.md:132–185`). **Read:** Materialise account-owned rows and revision under the same account lock; filter/sort/serialize after release; poll on visible tabs and focus (`research.md:21–35`). **View convenience:** Parameterless first `GET /api/tasks/` tries to obtain a complete session view; an invalid/unavailable view yields four defaults, while later requests explicitly carry the current four settings. `POST /api/session/view/` atomically replaces the session-bound tuple; concurrent tabs use last server commit (`research.md:57–101`; `contracts/http.md:56–130`).

**Overall assessment:** Clear and feasible boundaries for the task lifecycle. Specific session, privacy, performance and abuse/recovery control gaps remain; approval of the concept is not approval of those mechanisms.

## 3. R3 findings and dispositions

`Resolved` denotes demonstrated correction in the **written design**; it does not mean a runtime test passed. `Open` does not erase history or imply prior rejection of the whole architecture. `Deferred`/`Accepted Risk` is not inferred from an author's stated intention.

| Finding | Classification | Severity | Confidence | R3-Rev1 → R3-Rev2 disposition | R3 effect |
|---|---|---|---|---|---|
| **TD-001** Browser data remanence after cross-tab logout/account switch | Design Gap | **High** | High | Open → **Open** | **Blocker** |
| **TD-002** Failed mandatory first preference GET | Design Gap | **Medium** | High | Open → **Resolved (superseded by AM-001)** | No longer blocker; see TD-009 |
| **TD-003** Full snapshots and lifetime receipt capacity | Architectural Risk | **Medium** | High | Open → **Open** | **Blocker** |
| **TD-004** Receipt digest exposure/deletion privacy | Assumption / Design Gap | **Medium** | High | Open → **Open** | **Blocker** |
| **TD-005** Authentication throttling trust/enforcement | Design Gap | **Medium** | High | Open → **Open** | **Blocker** |
| **TD-006** Diagnostics, backup and recovery | Architectural Risk | **Medium** | High | Open → **Open; R5 handoff only proposed** | Owner/evidence-plan condition at R3; execution at R5 |
| **TD-007** Versioned operation canonicalisation | Architectural Risk | **Low** | Medium | Open → **Open; R4/upgrade handoff only proposed** | Nonblocking with explicit ownership |
| **TD-008 (new)** Fresh-login session lifetime vs Django login reuse | Design Gap | **Medium** | **High** | New → **Open** | **Blocker** |
| **TD-009 (new)** Unbounded best-effort view read can delay task bootstrap | Architectural Risk | **Medium** | Medium | New → **Open** | **Blocker** until failure-isolation mechanism is bounded |

### TD-001 — Cross-tab task data remains displayable after session authority changes

- **Status:** **OPEN**, High, Design Gap; high confidence. **Owner:** Technical Design + UI/security. **Links:** FR-010 / AC-010 and FR-009.
- **Observed design evidence:** `contracts/ui.md:95–110` says a tab clears on its own logout, a received 401 or account mismatch, and conceals browser-history restoration; it **explicitly declares cross-tab conceal-before-revalidation open**. Ordinary focus/visibility return is refreshed (`research.md:21–25`), but `contracts/ui.md:67,81,97–100` otherwise retains rendered rows while refreshing. `quickstart.md:86–88` says its session-view scenario does not close TD-001.
- **Failure path:** Tab 2 holds A's tasks while hidden. Tab 1 signs A out and B in. A cross-tab notification can be missed, and tab 2 may reveal A's existing DOM on focus before B-session verification returns, or indefinitely while network is unavailable. API owner isolation does not remove already-rendered A content.
- **Minimum correction / exit evidence:** Define a reliable conceal/invalidate policy for hidden/blurred tabs, visibility/focus and back-forward restoration **before** old content is exposed, with a best-effort inter-tab logout/account-switch signal that is **not the only safety dependency**. Clear task rows, drafts, pending payloads and queued view saves on invalidation; ignore old responses. Add static state/sequence contract and planned A→logout→B, dropped signal, delayed/failed revalidation and history-return cases. No Scope enlargement or UI redesign required. Human acceptance of residual cross-user content exposure is **not recorded**.

### TD-002 — Old blocking preference bootstrap removed

- **Status:** **RESOLVED by supersession in the approved written design**, Medium, Design Gap; high confidence. **Owner:** Functional Design (resolved policy), Technical Design (validated removal). **Links:** FR-012/013, AC-005, FR-009/014.
- **Direct evidence:** Scope `spec.md:173–175,192` now explicitly requires silent full-default fallback and prohibits blocking otherwise loadable tasks. R1-Rev4 §8 and R2-Rev4 §§4,8 approve the changed policy. `research.md:65–78`, `contracts/http.md:56–69,97–130`, `contracts/ui.md:37–43,73–77`, and `data-model.md:32–49` have **no initial account-preference GET/PUT prerequisite**. Bootstrap is one task GET with a complete default tuple if view data cannot be read. `quickstart.md:75–84` plans relevant regression scenarios.
- **Closure boundary:** The specific R3-Rev1 undefined failure of a **separate mandatory preference GET** no longer exists. Historical TD-002 is retained, not deleted. This closes a design gap, **not** an implementation test or an unconditional assurance that any database-backed view lookup cannot delay a task GET. The distinct new fault-containment issue is TD-009.

### TD-003 — No defensible launch operating envelope for unbounded snapshots/receipts

- **Status:** **OPEN**, Medium, Architectural Risk; high confidence. **Owner:** Technical Design + performance/operations. **Links:** FR-008/009/014/015, SC-004.
- **Evidence:** `plan.md:39,52` expressly leaves maximum accounts/list size and numeric performance assumptions unspecified. `research.md:21–27` polls complete snapshots every nominal 15 seconds while visible; `data-model.md:90,108–110,137–145` performs in-memory sorting and lifetime receipt retention. `contracts/http.md:40,56–95` asserts 413 and full matching set but gives no ingress/application body limit, worker/DB budget, load-shedding implementation or measured representative envelope. `quickstart.md:59,100` requests future observed timings only. AM-001 adds per-session view traffic but does not cure the account-data load.
- **Impact:** Large accounts, long descriptions, many visible tabs or concentrated writes can exhaust workers, memory/network or DB/lock capacity. Receipt/index accumulation grows for the life of every account. There is no evidence where the simple architecture becomes unacceptable.
- **Minimum correction / exit evidence:** Document a **technical workload hypothesis** (tasks, description bytes, active accounts/tabs, poll and mutation rates, retained receipt volume), bounded server/ingress body sizes and concurrency/timeout/backpressure policy, monitoring points and criteria for revisiting pagination/conditional reads. Give a reproducible load/lock/response/browser-memory measurement plan with representative resource budget. Actual stress measurements occur during Build/Test/R5, unless feasibility cannot be argued without a targeted prototype. This does **not** authorise a new user-facing cap or invented SLA.

### TD-004 — Stored request digest remains unqualified fixed SHA-256

- **Status:** **OPEN**, Medium, Assumption/Design Gap; high confidence. **Owner:** Technical Design + security/privacy. **Links:** FR-006/010, data minimisation on permanent deletion.
- **Evidence:** `data-model.md:100–110` still specifies `request_digest | fixed SHA-256 hex` from the canonical command, with lifetime retention and a statement that receipts cannot recover deleted task contents. `research.md:29–35` retains receipts without payload but gives no keyed/unkeyed distinction, secret custody or rotation scheme. `contracts/http.md:132–135,179` is likewise silent on digest version/key identity.
- **Impact:** If ordinary unkeyed SHA-256 is used, a receipt-table disclosure may permit offline guesses of low-entropy deleted text from known command structure; the absolute nonrecoverability wording is unjustified. Keying a digest without rotation/replay design can break durable idempotency in another way.
- **Minimum correction / exit evidence:** Explicitly choose and justify keyed versus unkeyed canonical digest; protect against foreseeable offline guessing or document a bounded accepted residual exposure. If keyed, define domain separation, independent secret custody, retained historical key/version identifiers, missing-key failure semantics and compatible replay/rotation. Qualify deletion privacy wording. Security owner evaluates residual consequences; no human risk acceptance has been supplied.

### TD-005 — Authentication throttling has no enforceable trust boundary

- **Status:** **OPEN**, Medium, Design Gap; high confidence. **Owner:** Technical Design + deployment/security. **Links:** FR-010/016, AC-010/012.
- **Evidence:** `contracts/http.md:21–26,175–177` and `research.md:109–113` mention production proxy throttling and a 429 response. `plan.md:118–120` and `quickstart.md:98–100` defer it to release configuration. None specifies enforcement placement, trusted forwarded-client identity, distributed shared counters/rate policy, bypass prevention or limiter-failure response. The proposed application is publicly self-registering (`contracts/http.md:11–18`).
- **Impact:** A proxy misconfiguration, forged forwarding headers or multiple independent edges can defeat limits or block shared-network users; password guessing and registration abuse remain inadequately controlled at design level.
- **Minimum correction / exit evidence:** Specify responsible enforcement layer and a verified client-address trust boundary (including no direct application bypass), shared/distributed enforcement if needed, separate registration and login policy dimensions, observable 429/Retry-After, limiter outage behaviour and abuse test matrix. Numeric production thresholds can be tuned at R5. Preserve generic login errors and CSRF.

### TD-006 — Recovery/diagnostic readiness still lacks an accepted operational handoff

- **Status:** **OPEN**, Medium, Architectural Risk; high confidence. **Owner:** named Operations individual **not yet identified**; Technical Design accountable for handoff. **Links:** FR-008/009/014, SC-004.
- **Evidence:** `contracts/http.md:42–54` has an opaque request reference; `research.md:37–43`, `plan.md:118–126` and `quickstart.md:98–102` mention support, backups and deployment validation. `review-followups/am001-r2-review-handoff.md:68–80` allocates a generic Operations lead at R5, but does not demonstrate reference propagation, redaction, restore consistency, key preservation, migration/rollback handling, agreed retention/access or a named accountable handoff.
- **Minimum correction / exit evidence:** Before R3 conditional readiness, identify accountable owner and R5 evidence deliverables: end-to-end nonsensitive request-ID tracing, useful health/errors, backup/restore scope for users, tasks, list revisions, receipts, sessions/views and digest keys, cleanup/retention, migration preflight/recovery, and a disposable restore/replay/ownership drill. **Executing** the runbook and drill is an R5 readiness requirement, not a demand to build deployment machinery within this review. No new RPO/RTO or end-user restore is implied.

### TD-007 — Canonical replay format remains unversioned

- **Status:** **OPEN**, Low, Architectural Risk; medium confidence. **Owner:** Technical Design/maintenance, R4 and compatible upgrade owners. **Links:** FR-008/009 and historical retry semantics.
- **Evidence:** `data-model.md:100–110` and `contracts/http.md:132–135` define canonicalisation and a fixed digest, without an explicit canonical format revision or how historical receipts survive changed normalization/defaults. `review-followups/am001-r2-review-handoff.md:78–79` names an R4/upgrade dependency but does not close it.
- **Minimum correction / exit evidence:** Specify versioned deterministic command encoding/digest metadata and replay of historical canonicalizers/keys, or an equivalent stable compatibility rule. Include create→delete→historical replay and upgrade/rollback fixtures, and an accountable R4/maintenance owner. **Nonblocking for R3 once explicitly owned**, but no human deferral/acceptance is invented.

### TD-008 (NEW) — Explicit same-account re-login can reuse the old view lifetime

- **Status:** **OPEN**, Medium, Design Gap; **high confidence**. **Owner:** Technical Design + authentication/session owner. **Links:** FR-012 / AC-005; AM-001 DEC-02.
- **Evidence:** The approved Scope demands defaults on **each new login** (`spec.md:173,192`). `research.md:15–17` selects ordinary Django `LoginView` and database sessions, while `research.md:80–86`, `data-model.md:58–67` and `contracts/http.md:28–32` assert that every successful explicit login, including the same account, gets a fresh session key/view. The artifacts give no actual login hook, transactional invalidation or middleware coordination that establishes this invariant. In Django 5.2 `django.contrib.auth.login()`, an existing session for the **same user with matching auth hash** is **not** flushed/cycled; `cycle_key()` is in the `else` branch when the current session has no authenticated user. The relevant stock source is [Django 5.2 auth login](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/__init__.py) (the `login()` implementation).
- **Concrete counterexample:** In browser session S, user A saves Completed/High/Title-desc. Without logging out, A visits the login page and explicitly signs in again as A. The chosen standard Django flow can retain session key S; the FK-bound SessionViewState survives and the next bootstrap restores the old view, contrary to the approved new-login default. Distinct-session isolation and the stated same-key guard do not fix this path by themselves. This is a feasibility gap in the *selected mechanism*, not a new product decision or a demonstrated implemented bug.
- **Minimum correction / exit evidence:** Define how explicit login establishes a new view lifetime even for same-user reauthentication while preserving unrelated valid sessions and preventing races with in-flight view saves, logout and ordinary session-key rotation. Specify when a legitimate non-login key rotation **moves** the view versus when login **discards** it, with consistent lock/transaction order and middleware/cookie handling. Include a same-account re-login scenario, plus different-account login, expiry and save/rotation races with actual PostgreSQL integration tests planned. No new browser settings store is required by this finding.

### TD-009 (NEW) — Inline optional view read has no bounded failure path

- **Status:** **OPEN**, Medium, Architectural Risk; **medium confidence**. **Owner:** Technical Design + database/session reliability. **Links:** FR-012 / AC-005, FR-014.
- **Evidence:** `research.md:65–69` and `contracts/http.md:56–69` make the first task GET read SessionViewState *before* taking the independent task snapshot. `data-model.md:69–71` prescribes a separate transaction/savepoint on a thrown view-read error, which is a sensible correction, but neither document states a timeout/lock-wait budget or a bounded fallback path for a view read that **does not promptly fail**. `plan.md:27–31,39` leaves session concurrency/failure isolation to R3. `quickstart.md:79,84,86` plans corruption and outage scenarios but no isolated long lock wait. The view row lives in the same PostgreSQL service as tasks; ordinary row updates do not necessarily block a SELECT, but a relation-level schema lock or impaired query can.
- **Impact:** A technically optional view lookup may delay/hang an otherwise loadable task snapshot, defeating AM-001's nonblocking convenience boundary. This is a conditional availability risk, not evidence that normal view reads are blocked or that the entire DB can fail while tasks remain available.
- **Minimum correction / exit evidence:** Define a bounded attempt for the optional view read and a safe fallback that abandons its transaction cleanly, uses all four defaults and proceeds to a separately valid/authenticated task snapshot. Ensure a failed auth check never degrades to anonymous/private access, and a failed task snapshot still follows FR-014. Add a deliberate session-view relation lock/timeout scenario and partial corruption/retry case to the PostgreSQL integration contract. A short engineering timeout is not a product SLA.

## 4. Cross-cutting architecture, security and regression assessment

| Area/scenario | Static design result | Residual risk / evidence boundary |
|---|---|---|
| **Task owner isolation and CSRF** | Owner from authenticated principal; foreign/unknown IDs return indistinguishable 404; CSRF, no-store and secure-cookie requirements documented (`http.md:3–40,162–179`; `research.md:13–19,109–113`) | Credible mechanism; cannot attest configuration without code/host. TD-001 is **client-side remanence**, not evidence of API cross-owner reads. |
| **Concurrent task edits, delete vs edit** | Account row lock and expected task version make one winner; others receive conflict (`data-model.md:21–30,73–110`; `http.md:158–179`) | Real-PostgreSQL races and rollback still future tests. Transaction lock order must be uniformly implemented. |
| **Lost acknowledgement, retry and remote delete** | Durable receipt-first historical acknowledgement prevents a replayed create from resurrecting a deleted task (`research.md:29–35`; `http.md:162–185`) | Accepted FD-003 browser interruption can lose pending identity and later duplicate user intent. TD-004/007 affect privacy and long-term replay. |
| **Failed task refresh** | UI keeps stale rows visibly marked, blocks all task writes including exact unknown retry until valid snapshot, and offers Retry/support (`data-model.md:147–165`; `ui.md:73–93`) | Preserves FD-001; not relaxed by AM-001. No live test. |
| **Session view tuple and tab updates** | Full valid four-tuple, atomic replacement, last actual server commit, explicit-query polling and lost-save silence documented (`research.md:57–78`; `http.md:56–130`; `ui.md:37–67`) | Correct approved convenience trade-off. TD-008 lifecycle and TD-009 optional-read timing still open. |
| **Logout, expiry, account switching** | HTTP auth and server session guards planned (`data-model.md:58–71`; `http.md:28–32`) | The session-key lifecycle is not fully specified (TD-008); rendered A data may survive in another tab (TD-001). |
| **Input/data model** | Nullable dates, defaults, version/enum checks and deterministic sorting are specified (`data-model.md:73–145`; `research.md:47–55`; `http.md:34–40`) | Unicode validation/DB trim difference correctly identified as app invariant; implementation tests needed, not a new R3 defect. |
| **Runtime/operability** | Django/Python/PostgreSQL stack and short DB transactions are a plausible fit (`plan.md:21–37`; `research.md:5–11`) | Resource/deployment implementation still unknown (TD-003/005/006). No rationale for adding services or queues. |

**AM-001 regression verdict:** FR-001–011 and FR-014–016 task rules are still represented by the amended design; direct bootstrap, session-scoped four-field restoration and complete silent defaults trace to FR-012/013 and AC-005. The old `TaskFilterPreference`/preference GET/PUT and account-level conflict handling are intentionally superseded, not secretly retained. The separate Django session-linked row introduces TD-008/009. Historical checklist CHK012/020/027/031 still mentions old account preferences; R2-Rev4 §6 already identified their cleanup as **nonblocking documentary work**. The plan, contracts and handoff also still say R2 approval is pending (`plan.md:7–9`; `contracts/http.md:5–7`; `contracts/ui.md:5–7`; `review-followups/am001-r2-review-handoff.md:3–5`). Those headers should be updated **metadata-only after R3**, without implying the previously approved design body has changed; they are not substantive R3 defects.

## 5. Required correction package and phase handoffs

**Return to Codex/Technical Design for a narrowly scoped correction cycle; do not alter Scope v1.1 or approved observable R2 behaviour without separate change control.**

1. **Privacy and lifecycle (TD-001, TD-008):** specify conceal-before-validation for cross-tab/account/history paths, a new-login vs non-login-rotation session-row protocol, failure/race ordering and targeted scenarios.
2. **View fault containment (TD-009):** bound optional SessionViewState lookup, preserve silent whole-tuple defaults and independent successful task retrieval, test partial view failures and lock waits. TD-002 remains historically resolved.
3. **Capacity (TD-003):** document realistic technical workload and hosting budgets, request limits, backpressure/timeout strategy, receipt growth, monitoring and later load measurements.
4. **Receipt privacy (TD-004):** choose defensible digest design, key/version/replay/rotation treatment and precise residual privacy language; coordinate with TD-007.
5. **Auth abuse (TD-005):** choose enforceable throttling/trusted ingress and limiter-unavailability semantics with abuse tests.
6. **Explicit handoffs (TD-006/007):** name responsible Operations and maintenance owners and R5/R4 deliverables. Keep drills/production configuration/upgrade fixtures at their correct later gates unless the technical design cannot be judged without further evidence.

**Re-review submission:** provide changed-file list, concise per-TD change/evidence map, relevant design sections/contracts/scenarios, any new architectural decision record needed for digest/auth/session choices, and exact file hashes. The reviewer will check fixes, upstream invariant preservation and regressions **only as far as affected**, preserving all nine finding IDs and earlier reports. If a correction changes FR/AC or observable behaviour, escalate to targeted R1/R2 amendment rather than reinterpreting approval.

**R4/R5/Build boundaries:** TD-007 compatibility planning is suitable for R4 and future release tests once owned; TD-006 operational proof belongs at R5 with named responsibility. Real database concurrency tests, browser privacy/session journeys, security control configuration, load evidence, restore rehearsal and rollout/migration validation are downstream evidence—not falsely asserted here. The accepted FD-003/004 risks still need product/test/support traceability. No implementation, deployment, tasks/analyze or self-service data recovery is authorised by this review.

## 6. R3 exit gate and human decision

| R3 gate question | Result |
|---|---|
| Correct human-approved Scope / R2 Functional Design inputs identified | **PASS** — R1-Rev4 and R2-Rev4 decisions; exact current hashes verified |
| Prior findings maintained with disposition evidence | **PASS** — TD-001–007 retained; TD-002 resolved by approved supersession |
| Material privacy/session requirements mapped to feasible mechanisms | **FAIL** — TD-001, TD-008 and TD-009 |
| Capacity and security threat controls sufficiently specified | **FAIL** — TD-003, TD-004, TD-005 |
| Operational/upgrade residual risks owned for downstream gates | **NOT YET** — TD-006/007 assignments proposed but not accepted/named |
| Dynamic correctness, performance and recovery verified | **NOT PERFORMED** — intentionally outside static R3 |

### Formal independent R3-Rev2 verdict: **NOT READY**

**Blocking technical corrections:** TD-001, TD-003, TD-004, TD-005, TD-008 and TD-009. **Gate conditions for downstream handoff:** named accountable owner and evidence plan for TD-006, explicit maintenance/R4 ownership for TD-007. **Resolved finding:** TD-002 only. **Human-accepted residual risks:** previously FD-003 and FD-004 under R2 authority; **none of the new or remaining TD risks has been accepted by the human**.

**Human R3 gate:** **OPEN / NOT APPROVED.** The reviewer does not grant build authorisation or infer an R3 human decision. After targeted design corrections and an independent targeted R3 re-review reach Ready/Conditionally Ready, request Barry's explicit human R3 approval before moving to the next authorised phase.

## 7. Review history integrity

- **R3-Rev1:** original NOT READY assessment; TD-001–007 Open. Addendum A documented the proposed narrowed session-view direction, not approval or closure.
- **R1-Rev4:** Scope 1.1 / AM-001 subsequently approved by Barry; R1-AM-004 historical archive issue deferred.
- **R2-Rev4:** amended Functional Design independently assessed and separately human-approved for its exact file manifest; historical FD-003/004 accepted risks persisted.
- **R3-Rev2 (this report):** independent static comparison against that exact approved file set. TD-002 resolved by supersession; TD-001/003/004/005/006/007 remain Open; TD-008/009 newly Open. Verdict NOT READY; **no canonical design documents or earlier review histories were modified**.

## 8. Primary technical references consulted

- [Django 5.2 session documentation](https://docs.djangoproject.com/en/5.2/topics/http/sessions/) — database-backed sessions, `cycle_key()`, `flush()`, expiry and cleanup.
- [Django 5.2 `django.contrib.auth.login()` and `logout()` implementation](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/__init__.py) — same-user re-authentication branch vs `cycle_key()`, logout flush.
- [Django 5.2 SessionMiddleware implementation](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/middleware.py) — middleware save/cookie and interrupted-session behaviour.

These primary references inform feasibility analysis only. They do not substitute for repository-specific implementation or executed tests.
