# Taskify — Independent R3 Technical Design Review (R3-Rev1)

**Review date:** 2026-10-23 (Australia/Melbourne)  
**Reviewer:** Independent Technical Design Reviewer (R3)  
**Independent verdict:** **NOT READY — targeted technical design corrections required**  
**Human R3 gate:** **OPEN — no human R3 approval recorded**  
**Review type:** Static architecture/design assessment; no code, deployment, runtime tests or threat-penetration tests performed.

**Post-issuance note (2026-10-23):** See **Addendum A** for the user's revised filter/sort persistence direction prompted by TD-002. It is recorded within **R3-Rev1**, not treated as a second independent review. The original findings and gate verdict below remain the first-pass assessment of the originally supplied, approved design.

## 1. Baseline, scope and assurance limitations

**Supplied input:** `Archive.zip`, SHA-256 `304cd6588d777888aa0eb154409bd0537ca901e247064bb0c5698d9fde35c089`, containing the nine `specs/001-personal-task-manager/` design documents and the R1/R2 review histories. File references below are relative to `specs/001-personal-task-manager/` inside that archive.

**Approved Scope:** `spec.md`, SHA-256 `4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`. This hash was independently recomputed from both the working `spec.md` and the final R1 review copy, and matches the R2 approval record. Requirements FR-001–FR-016 and AC-001–AC-012 are the controlled product boundary.

**Functional Design gate:** The final R2 record, `independent-reviews/2_design-functional/Review_3 (final)/taskify_R2-Rev3_functional_design_review_APPROVED.md`, records independent reviewer result `READY_FOR_HUMAN_APPROVAL` and a later distinct human gate decision **APPROVED**, dated 2026-10-23. It identifies the original approved design package `specs(2).zip`, SHA-256 `e75582cd7f849b918ce8e8de559449144936d0ea44cba518d4bdf7188b206f4f`, zero open R2 findings, FD-001/002/005 verified resolved, and FD-003/004 human-accepted risks. Those risks remain accepted, **not eliminated**. R3 does not retroactively reverse R2 approval.

**Provenance limitation:** This upload is a newly packaged `Archive.zip`, not the original `specs(2).zip`. The approved Scope file can be hash-verified, and the supplied design files correspond structurally and substantively to the R2 description; however, the approved R2 ZIP-level hash cannot be recomputed against this repackaging. A copy of the original approved ZIP or an approved per-file manifest is needed for cryptographic equivalence of *all* Functional Design files.

**Other inputs not supplied:** actual application code, migrations, deployment/IaC definition, lockfile, database, deployed configuration, test results, the asserted constitution v1.0.0 text, and formal ADR files. Those absences are not represented as implementation failures. This is a design review; no approved performance SLA, availability target, data-loss RPO, or task-volume limit is invented.

## 2. Architecture reconstruction

- **Runtime/topology:** One Django 5.2 LTS / Python 3.13 web application using templates and browser ES modules, Gunicorn in production, Psycopg 3 and PostgreSQL 17. An HTTPS proxy is implied but provider/configuration/topology are not yet selected. No SPA, external identity provider, queue, WebSocket service or native application (`plan.md:16–41`).
- **Trust boundary:** Public self-registration and login create Django database sessions. The browser sends same-origin, session-authenticated, CSRF-protected API requests. All task/revision/receipt/preference ownership comes from the authenticated principal, never a submitted account ID (`research.md:13–17,75–79`; `contracts/http.md:5–44`).
- **Persistent state:** `accounts.User`, account-scoped `TaskListState`, `Task`, `TaskFilterPreference`, `MutationReceipt`, and Django sessions. Task UUID/version and account revision identify changes; a physical deletion removes the task but retains receipt metadata (`data-model.md:7–88`).
- **Write path:** Acquire account-state row lock, locate owner-scoped operation receipt, validate expected task version, apply task effect, increment account revision, insert a durable receipt, commit, and only then acknowledge. A replay of the exact key/payload acknowledges historical success without repeating the effect (`contracts/http.md:110–164`).
- **Read/sync path:** Account-state lock → materialise all account-owned tasks and revision → release lock → filter/sort/serialize → browser snapshot validation. Initial load, post-write, focus and visible-period polling converge devices. Response identity/query/generation/revision guards reject stale data (`research.md:21–27`; `contracts/http.md:46–77`).
- **Browser state:** Explicit loading/ready/writing/refreshing/load-error/outcome-unknown/conflict/signed-out states. Pending operation keys are memory-only by approved FD-003 risk acceptance; preference selections are account-backed with optimistic versions; calendar overdue is derived from local date (`data-model.md:90–136`; `contracts/ui.md`).

**Assessment of core pattern:** A single Django/PostgreSQL service, short per-account transactional serialization, optimistic task versions and durable idempotency receipts are coherent and feasible for a small personal task manager. The architecture does not need microservices, queues or realtime streams merely to satisfy the approved requirements. PostgreSQL row locks serialize conflicting account operations as described, provided every relevant database path actually takes the specified lock and query materialisation occurs within the transaction. These are planned properties, not verified implementation results.

## 3. Findings register

| ID | Classification | Severity | Confidence | Disposition | Gate effect |
|---|---|---|---|---|---|
| **TD-001** | Design Gap | **High** | High | **Open** | **R3 blocker** |
| **TD-002** | Design Gap | **Medium** | High | **Open** | **R3 blocker; R2 escalation** |
| **TD-003** | Architectural Risk | **Medium** | High | **Open** | Capacity argument before R3 closure |
| **TD-004** | Assumption / Design Gap | **Medium** | Medium–High | **Open** | Security/privacy decision before R3 closure |
| **TD-005** | Design Gap | **Medium** | High | **Open** | Security mechanism before R3 closure |
| **TD-006** | Architectural Risk | **Medium** | High | **Open** | Explicit R5 operational handoff acceptable |
| **TD-007** | Architectural Risk | **Low** | Medium | **Open** | Non-blocking upgrade-compatibility handoff |

### TD-001 — Previously rendered task data may remain visible after logout or account switch in another tab

- **Classification / severity / confidence:** Design Gap / **High** / **High**.
- **Evidence:** `contracts/ui.md:55,74,81–87`; `research.md:23,39–41,77`; `quickstart.md:59,86`.
- **Requirement:** FR-010, AC-010 (account privacy), FR-009 (clear failure/session feedback).
- **Technical counterexample:** Account A is signed in to two tabs in the same browser profile. Tab 1 logs out, invalidating the shared session. Tab 2 is hidden and stops polling, but its rendered A task data remains in memory. Later, after account B signs in using the same profile, tab 2 can be brought forward. The design requires a focus refresh and clears on a *received* 401/account mismatch, but does not require concealment *before* validation on ordinary visibility return or cross-tab logout notification. The stated normal refresh policy retains old rows while refreshing. Old A content can therefore be exposed while validation is pending; a hidden tab can retain it indefinitely.
- **Impact:** Exposure of private task content after sign-out on a shared browser, despite server-side object ownership being sound. This is a UI data-remanence risk, **not** an assertion that account B can query account A's data through the API.
- **Minimum remedy:** Define an observable same-profile sign-out/account-change invalidation path for other tabs and a conceal-until-revalidated rule whenever a tab may have lost session authority (including visibility/focus and history restoration). A cross-context signal is one feasible technique, not a mandated architecture. Clear drafts and pending task payloads when invalidated, preserving FD-003 semantics. Add a multi-tab A→logout→B test that asserts no A content is rendered before/after revalidation.
- **Owner / escalation / disposition:** Technical Design + UI/security; no product expansion intended. **Open — R3 blocker**; human security-risk acceptance would need explicit consequences if not fixed.

### TD-002 — Initial filter-preference GET failure has no defined behaviour

- **Classification / severity / confidence:** Design Gap / **Medium** / **High**.
- **Evidence:** `contracts/http.md:79–89`; `contracts/ui.md:35–51,59–75`; `quickstart.md:67–78,82`. All describe normal initial preference retrieval and later *PUT/reconciliation* failure, but not a failing initial preference GET.
- **Requirement:** FR-012, AC-005 (restore saved filters); FR-009, FR-014 (load failure feedback/protection).
- **Technical counterexample:** The authenticated preference GET returns 503/malformed JSON while the task-snapshot endpoint could respond. Since the shell must read preferences before requesting its first task snapshot, neither the next request nor the UI's precise error/retry state is defined. Silently substituting All/All would misrepresent the last saved filter pair; indefinitely waiting without explanation undermines returning access.
- **Impact:** An unhandled bootstrap failure can leave a valid account unusable or show an unconfirmed view as if restored. The distinction from a failed *task* snapshot matters to write-lock and retry behaviour.
- **Minimum remedy:** **Escalate the observable policy to a narrowly scoped R2 Functional Design addendum:** what the user sees, whether task requests/writes are held, and how Retry/support and eventual reconciliation work when the *first* preference read fails. R3 then specifies the request-state mechanism, response validation and test cases. Do not silently invent a default-restoration policy in implementation.
- **Owner / escalation / disposition:** Functional Design owner (behaviour) → Technical Design (mechanism); **R2 escalation**. **Open — R3 blocker until the ambiguity is resolved and incorporated into the approved baseline.**

### TD-003 — Full-account snapshots and lifetime receipts have no capacity envelope

- **Classification / severity / confidence:** Architectural Risk / **Medium** / **High**.
- **Evidence:** `plan.md:30–41`; `research.md:21–27`; `data-model.md:64–66,84–86`; `contracts/http.md:46–77`.
- **Requirement:** FR-008/009/014/015 and SC-004 (sync, usable read failures, mobile browsers, persistence).
- **Risk scenario:** Every normal refresh copies all task rows and sends all matching rows, at a nominal 15-second visible-tab cadence. Tasks per account, description size, open tabs, account count, mutation rate and lifetime receipt count have no stated bound. A single unusually large or automated account can increase database work, account-lock hold times, Gunicorn memory, serialization latency, network use and mobile-browser memory; authenticated clients can generate many task creates. There is no evidence for where this design stops being proportionate.
- **Minimum remedy:** Define a *technical* operating envelope or representative workload assumptions and measure query/lock duration, payload size, client memory and aggregate polling/write load. Specify request-size and abuse protections, timeout/backpressure and monitoring, with a migration trigger to pagination/conditional reads if evidence warrants. Do not invent a product task-count cap or numerical SLA without Scope authority. Receipt storage growth must be included in capacity modelling.
- **Owner / escalation / disposition:** Technical Design + performance/operations; **Open**. A supported small-list envelope and targeted validation plan may close R3 without implementing pagination.

### TD-004 — Plain SHA-256 receipt digests may reveal deleted low-entropy task content

- **Classification / severity / confidence:** Assumption / Design Gap / **Medium** / **Medium–High**.
- **Evidence:** `data-model.md:68,70–86` defines a fixed SHA-256 digest of the canonical command retained for account life and asserts receipts do not retain recoverable task contents; `research.md:31–35` contrasts metadata with deleted text.
- **Requirement:** FR-006, FR-010; no in-app restore; data minimisation claimed by design.
- **Risk scenario:** If `request_digest` means *unkeyed* SHA-256 of canonical task input, an actor who obtains the receipt table and knows its schema/recorded UUIDs can compute candidate digests for common short titles or descriptions. Hashes of low-entropy content are not equivalent to erasure. This is a conditional cryptographic design risk, **not** proof that the implementation already stores raw or reversible text.
- **Minimum remedy:** State whether the digest is keyed or unkeyed and assess offline guessing against deleted data. A keyed digest using separately protected secret material is one possible remedy; if chosen, preserve deterministic replay across key rotation/deployment. Alternatively demonstrate that the canonical digest does not expose guessable user text. Do not falsely claim absolute irrecoverability.
- **Owner / escalation / disposition:** Technical Design + security/privacy; **Open**. Record an explicit decision and residual risk before R3 gate closure.

### TD-005 — Public-account authentication throttling is a deployment assertion, not an assessable mechanism

- **Classification / severity / confidence:** Design Gap / **Medium** / **High**.
- **Evidence:** `research.md:13–17,75–79`; `contracts/http.md:7–22`; `plan.md:103–105`; `quickstart.md:88–90`. Production proxy throttling is named for registration/login, but no enforcement boundary, source-identity trust, policy treatment for distributed proxies or failure mode is documented.
- **Requirement:** FR-010, FR-016, AC-010/012 (private auth and open self-registration).
- **Risk scenario:** Depending on topology, an untrusted forwarded IP can bypass or concentrate per-IP throttles; multiple edge instances without shared state can enforce different limits; automated registrations or login guesses can consume resources or attack accounts. No actual vulnerability or rate-limit test is established from the design alone.
- **Minimum remedy:** Choose and document the rate-limiting enforcement point, identity/forwarded-header trust model, handling of distributed instances, observable 429 behaviour and a representative abuse test. Parameters may be tuned during R5; the mechanism and responsible layer should be decidable before Build. Retain Django's password validation and generic credential failure semantics.
- **Owner / escalation / disposition:** Technical Design + deployment/security; **Open**. R3 closure requires a credible configuration/design, not an implemented proxy.

### TD-006 — Recovery and diagnostics are named but not yet operationally verifiable

- **Classification / severity / confidence:** Architectural Risk / **Medium** / **High**.
- **Evidence:** `research.md:43,75–79`; `contracts/http.md:30–44`; `plan.md:103–105`; `quickstart.md:88–90`.
- **Requirement:** FR-008/009/014 and SC-004; support guidance and saved-change durability.
- **Risk scenario:** A support reference is displayed but no end-to-end request-ID propagation, safe structured logging or diagnostic lookup is specified. Backups are required but their restorable scope (task/list/preference/receipt/session consistency), restore procedure, access control and verification are unspecified. A successful scheduled backup alone does not prove recovery; no product RPO/RTO has been approved.
- **Minimum remedy:** Assign concrete R5 readiness evidence: trace support reference to appropriately redacted service/database diagnostics; add health/error metrics sufficient to detect failures; specify backup scope/retention/access and a disposable-environment restore drill including receipts/revisions and schema/migration compatibility. Clarify failure and rollback plan for migrations. Do **not** add a self-service restore endpoint or invent a zero-data-loss SLA.
- **Owner / escalation / disposition:** Operations / Technical Design for handoff; **Open**, acceptable as an explicit **R5 dependency** rather than a pre-Build implementation blocker if owned and tracked.

### TD-007 — Receipt canonicalization is not versioned across future deployments

- **Classification / severity / confidence:** Architectural Risk / **Low** / **Medium**.
- **Evidence:** `contracts/http.md:110–124`; `data-model.md:76–86`; `research.md:29–35`.
- **Requirement:** FR-008/009; historical same-key replay.
- **Risk scenario:** A committed old request is replayed after a deployment that changes normalization/default rules or command encoding. Recomputing its canonical digest with *new* rules may produce `operation_key_reused` for an originally identical request, preventing historical acknowledgement. Current release has no demonstrated upgrade scenario, so this is a forward-compatibility risk rather than a proven release-one defect.
- **Minimum remedy:** Define a stable wire/canonicalization format or stored digest-scheme version, including how old receipts will be checked after compatible upgrades. Include one cross-version replay regression scenario in change-control guidance.
- **Owner / escalation / disposition:** Technical Design/maintenance; **Open — non-blocking**, track for R4/implementation design and future migration review.

## 4. Requirements-to-architecture assessment

| Scope requirements | Principal proposed mechanism | R3 result |
|---|---|---|
| FR-001–006 — task lifecycle, fields, permanent delete | Server domain validation; Task types/constraints; versioned commands; explicit delete confirmation | **Covered**; receipt privacy caveat TD-004 |
| FR-007, FR-011–013 — filters, calendar, sorts and restoration | Account-backed versioned filters, domain deterministic ordering, device-local date-only overdue | **Mostly covered**; failed first preference load TD-002 |
| FR-008 — saved/cross-device change availability | Atomic PostgreSQL writes, list revisions, durable receipts, polling snapshots and version conflicts | **Mechanistically covered**; capacity/operational risk TD-003/006 |
| FR-009, FR-014 — error, retry and saved-data protection | Read failure lock; outcome-unknown state; exact operation replay; Retry/support | **Covered for task reads/writes**; TD-002 and TD-006 remain |
| FR-010 — account isolation | Django auth/session/CSRF; owner-scoped repositories; 404 for foreign IDs | **Server boundary covered**; tab-level residual exposure TD-001 and abuse control TD-005 |
| FR-015, FR-016 — browser delivery and self-registration | Responsive Django pages + JS; standard user creation/auth | **Covered in principle**; browser and deployment tests not run |

There is no evidence that the chosen runtime majors are intrinsically incompatible: Django 5.2 LTS documents support for Python 3.13 and PostgreSQL 17. The design appropriately uses real PostgreSQL for concurrency tests. Version-pinning and actual browser/device verification remain implementation duties.

## 5. Adversarial scenario assessment

| Scenario | Static result | Evidence or action |
|---|---|---|
| Two devices edit the same version | **Coherent design** | Account lock + expected task version; test separate DB connections (`http.md:142–157`) |
| Create commits, response lost, remote delete, replay old create | **Coherent design** | Receipt before task lookup prevents resurrection; current list must be re-read |
| Read/refresh fails while write outcome unknown | **Coherent design** | UI blocks all mutations, including exact replay, until valid saved-data snapshot |
| Request completes after client crash | **Accepted residual risk FD-003** | Operation ID can be lost and intent duplicated; do not recategorize as resolved |
| User forgets account credentials | **Accepted residual risk FD-004** | No automated/manual reset or account transfer; no new product promise |
| Initial preference read unavailable | **Undefined outcome** | TD-002; narrow R2 escalation needed |
| Session changes in another tab | **Exposure path** | TD-001; require concealment before authority revalidation |
| Account has very large tasks/number of tasks | **Unproven capacity** | TD-003; bound and stress the intended operating envelope |
| Database tables/backup exposed | **Conditional privacy/recovery risks** | TD-004; TD-006 |
| Registration/login automation | **Security mechanism incomplete** | TD-005 |
| Backup restored and migrations reapplied | **Unverified operational recovery** | TD-006; R5 drill |

## 6. Readiness verdict and exit conditions

### Formal R3 reviewer verdict: **NOT READY**

This is **not** an architectural rejection. The principal data ownership, atomic-write, optimistic concurrency and retry approach is technically sound for the intended simple product. The gate is held because **TD-001** exposes a credible private-data remanence path, **TD-002** leaves required initial-load behaviour undefined and requires an R2-owned decision, and **TD-003–005** need bounded capacity and specific security mechanisms before their risks can be assessed responsibly. No evidence justifies moving to services/queues or changing approved Scope.

**Required before R3 re-review:**

1. Resolve TD-001 with an explicit cross-tab sign-out/account-change concealment model and targeted multi-tab acceptance scenario.
2. Obtain a narrowly scoped, approved Functional Design resolution for TD-002; update affected HTTP/UI/startup state and test contracts without silently choosing product behaviour in R3.
3. Provide a credible launch operating envelope and protection/measurement plan for TD-003 (without inventing product limits/SLA).
4. Resolve TD-004's digest-privacy assumption and TD-005's authentication-throttling trust boundary with minimum sufficient design decisions.
5. Assign TD-006 a named R5 owner/evidence plan; track TD-007 for compatible upgrade and migration work.
6. Reconcile the revised files with the controlled R2 baseline. Obtain original approved `specs(2).zip` or an authoritative per-file manifest if exact cryptographic equivalence is required. Re-review changes for fixes, regressions and new findings, then request **explicit human R3 approval**; this reviewer verdict is not approval.

**Evidence still required later, not claimed now:** real-PostgreSQL rollback/locking races, HTTP/CSRF and ownership tests, old-response and session-generation browser scenarios, Android/iOS smoke tests, request-size/load stress, restore rehearsal and live deployment configuration checks. The intentionally unchecked `checklists/feature.md` items are review prompts, not proof of test success. Planned `tasks.md` and executable implementation artifacts are correctly absent at this phase.

**R3 gate state:** **OPEN — NOT APPROVED**. Findings above are first-pass `Open` dispositions. No corrective revisions, risk acceptances, dynamic verification or human sign-off have been supplied for this R3 iteration.

---

## Addendum A — Post-review product decision prompted by TD-002 (2026-10-23)

**Record type:** Human-endorsed design direction and change-control handoff **arising from R3-Rev1**, not a new reviewer iteration or a retrospective alteration of the original finding. **Change status:** proposed Scope/Functional Design amendments; their formal review and human approvals are **pending**. **R3 gate remains OPEN — NOT READY.**

### A.1 Why the requirement is being reconsidered

TD-002 exposed a consequence of the approved design: account-backed, versioned filter preferences are read in a separate startup step before the first task snapshot. If that preference read fails, an otherwise available task list may be delayed or blocked. The subsequent unapproved `review-followups/r2-td-002-proposal.md` would explicitly have blocked task access until preference recovery. Following discussion of that trade-off, the user rejected treating filter restoration as a prerequisite for access to saved tasks.

The product distinction is material: **saved tasks and their availability are core functionality; remembered filters and ordering are conveniences.** Unavailable preference state is an annoyance, not a reason to withhold a successfully loadable task list. A proportionate failure experience is to show the tasks with the default view, explain briefly that previous view settings could not be restored, and allow filters/sorting to be applied again. This does **not** relax FR-014's protective behaviour if the *task data itself* cannot load.

Further examination showed that the original cross-session restoration requirement was broader than the actual user need. The user wants to avoid losing their chosen view on an accidental refresh or opening another tab, not to carry view settings to a later login or another browser/device. Narrowing this behaviour may eliminate account-level preference storage, optimistic preference versions, reconciliation, and the standalone preference-read dependency. The new narrower policy needs controlled product approval; this report does not implement it.

### A.2 User-endorsed intended behaviour (subject to controlled amendment)

| Situation | Filter status/priority and sort method/direction |
|---|---|
| User changes the view, then refreshes the same authenticated browser session | Restore the selected view |
| User opens Taskify in a **new tab of the same authenticated browser session** | Start that tab with the most recently applied view settings from the shared session |
| Another tab is already open when the selection changes | Live updating the already-open tab is **not** required; a subsequent refresh/reopen restores the shared session's latest selection |
| User logs out, then logs in again | Reset to defaults rather than restoring the previous login's view |
| User opens Taskify in a different browser, browser profile, or device | That distinct session starts with defaults, even for the same Taskify account |
| No view settings exist in this authenticated browser session | Default to Status **All**, Priority **All**, Sort **Due Date — earliest first** |
| Previous view settings cannot be restored, but tasks can be loaded | **Do not block task access solely for this reason**; show default view with a brief nonblocking notification and allow the user to reapply the view |
| Previously saved tasks cannot be loaded | Preserve existing FR-014 task-load error/retry/support and saved-data protection rules |

**Scope boundary:** Task data remains account-persistent and synchronised across devices under FR-008/FR-010. Only *view settings* become browser-authenticated-session scoped. A shared authenticated session across tabs is required, but instantaneous cross-tab synchronization of already-rendered views is not. On logout/account change, settings must not leak to the next account or session.

### A.3 Technical implications requiring revised design

- **Scope amendment / R1 approval:** Revise the persistence statements in `spec.md` (notably FR-012, AC-005, the related scenarios/assumptions and any needed FR-013 clarification) to specify both filters **and sorting** across refresh/new tabs, with reset on logout/new login or distinct browser/device. Preserve every task-persistence guarantee.
- **Functional Design amendment / R2 review and human approval:** Specify the startup/default/error behaviour and cross-tab boundary; revise `contracts/ui.md`, `contracts/http.md` and quickstart scenarios. The original R2 approval remains historical fact, not automatic approval of this changed policy.
- **R3 technical correction:** Evaluate a minimal session-scoped mechanism, potentially using existing Django authenticated-session infrastructure, instead of the account-level `TaskFilterPreference`, its separate GET/PUT bootstrap, version conflicts and reconciliation. This is a preferred simplification to evaluate, **not** a mandated implementation detail; reconcile `research.md`, `data-model.md`, `plan.md`, contracts and verification cases.
- **TD-002 disposition:** Remains **Open** until the revised Scope/Functional Design is formally approved and the resulting technical design is independently checked. If the problematic initial preference GET is removed, record TD-002 as **Resolved** by superseding that mechanism, with evidence, rather than silently deleting the finding. Supersede—but retain for audit—the **unapproved** blocking-startup proposal in `review-followups/r2-td-002-proposal.md`.
- **Other findings:** TD-001 and TD-003–TD-007 remain as originally recorded. In particular, shared browser sessions across tabs increase the need to resolve TD-001's logout/account-switch stale-content scenario; this decision does not solve it.

### A.4 Required sequence and review integrity

1. Place this updated **R3-Rev1** report in `independent-reviews/3_design-technical/Review_1/` and treat this addendum as the rationale/decision handoff.
2. Prepare **targeted, clearly unapproved** Scope and Functional Design amendment proposals. Obtain the applicable independent re-reviews and explicit human approvals before changing the authoritative approved baselines.
3. Incorporate approved amendments into the design, correct the other material R3 findings and return the revised package for an independent **R3 re-review (Rev2)**. Track dispositions with evidence and regression analysis.
4. Do not run Spec Kit Tasks/Analyze, proceed to R4/R5, or begin Build until the required upstream gates close in sequence.

**Approval accounting:** The user's agreement here establishes the intended product behaviour and authorises preparation of amendment proposals. It does **not** by itself constitute a new formal R1 Scope approval, R2 Functional Design approval or R3 Technical Design approval. The original **R3-Rev1 independent verdict remains NOT READY**; all seven TD findings remain **Open** pending controlled change/re-review.
