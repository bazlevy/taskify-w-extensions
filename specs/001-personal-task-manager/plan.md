# Implementation Plan: Personal Task Manager

**Current baseline:** Scope v1.2 / AM-002 and R2-Rev8 APPROVED.
[R2-Rev8 approval §4](../../independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md) records Barry Levy's 2026-10-25 decision; FD-009 is
RESOLVED_VERIFIED per §2. R2-Rev5/R2-NAV-001 and R2-Rev6 remain historical approvals;
R2-Rev7 NOT_READY is preserved. R3 NOT_READY; TD-008/010/011 OPEN. P1/IndexedDB paused.

R2 incorporation remains approved. R3-Rev10 now proposes ADR-AS-004 in
[technical design](technical-design.md) and [lifecycle contract](auth-transition-protocol.md).
This is documentation-only technical remediation, not a Spec Kit Plan rerun, test execution or
Tasks/Analyze/Build approval. [R2 evidence](review-followups/r2-rev8-incorporation-evidence.md)
retains its original incorporation identity; [R3-Rev10 handoff](review-followups/r3-rev10-handoff.md)
identifies the current unapproved technical candidate.
R3-Rev9/ADR-AS-003's strict-winner/atomic-render incompatibility assessment is history, not a
reason to reimpose now-superseded requirements or reopen already supplied human decisions.

**Feature identifier**: `001-personal-task-manager` | **Date**: 2026-10-24 | **Spec**: [spec.md](spec.md)

**Input**: Scope v1.2 / AM-002, approved by Barry Levy 2026-10-25 in R1-Rev5 §9 for candidate
SHA-256 `87bf1574788c46eedf6412782e1b6a60915a4db0d4b643483d9e79736a2c745c`; current
approval-metadata file SHA-256 `688b57c11785af800b072843d7763c14875dbc6cbb5b35594201936cee03549b`.
**Design status**: R2-Rev8 human APPROVED against the exact conventional-auth amendment.
Historical R2-Rev5 navigation remains; only CA-006/007/008, aligned Q1 and bounded risk supersede
affected prior clauses. FD-009 resolution is written-contract verification, not technical/runtime proof.
**Git context**: Actual checkout is `main`; setup reports `001-personal-task-manager` from its feature-directory fallback. Planning does not create or switch branches.

## Summary

Build one responsive Django application backed by PostgreSQL. Users self-register with username/password and manage their own tasks through session-authenticated interfaces. Server-rendered pages and a small JavaScript controller refresh complete filtered snapshots on focus and periodically while visible. Domain services own validation, ordering and state transitions; repositories own account-scoped persistence. Durable operation receipts and task versions prevent duplicate retries and silent overwrites across devices.

AM-001/AM-002 and R2-Rev8 decisions are recorded in [research.md](research.md) and the independent
approvals. Approved Scope remains unchanged in this incorporation. The implementation stack and
unrelated choices remain; the earlier Plan rerun is historical. Stop at the R3 handoff: no Plan
rerun, tasks.md generation, implementation or technical approval.

## Technical Context

**Language/Version**: Python 3.13, maintained patch; browser JavaScript ES modules and HTML/CSS.

**Primary Dependencies**: Django 5.2 LTS, Psycopg 3, Gunicorn for production. Exact maintained patches locked during implementation. No SPA, separate auth provider, task queue or WebSocket service.

**Storage**: PostgreSQL 17, maintained patch, for users, sessions, tasks, account list revisions,
durable operation receipts, SessionViewState and the R3-Rev10 proposed LogoutReceipt. Prior
AuthorityFamily/AuthorityHandle/AuthTransition tables are removed from this candidate and retained
only as historical text. Native Session remains sole authentication authority; the logout receipt
only confirms committed revocation.
The login-scoped view replaces account-owned preferences and remembers filters/sorting without
cross-login/device inheritance. No new storage service or browser-persisted settings. Session save,
logout concurrency and failure isolation use the Session-row guard/response adapter, subject to
independent R3 review. No periodic maintenance rotation or live rekey route is selected.

**Testing**: pytest, pytest-django and pytest-playwright; pure domain tests, real-PostgreSQL integration/concurrency tests, HTTP contract tests and browser journeys.

**Target Platform**: Desktop Chrome/Edge/Firefox/Safari; mobile Chrome on Android/Safari on iOS. Record current stable versions during release validation. Automate Chromium/Firefox/WebKit and perform actual mobile browser smoke checks. JavaScript, cookies and connectivity required.

**Project Type**: Single web application and database, with a same-origin JSON interface supporting its UI.

**Performance Goals**: No approved numeric latency, task-count or usability thresholds. Visible-page polling every 15 seconds is an adjustable design parameter, not an SLA. Record observed timings/list sizes; do not reinstate removed SC-001–SC-003.

**Constraints**: Ownership enforced for every read/write; successfully committed changes persist and
converge across devices. Date-only due dates. Failed load/refresh blocks every task-mutation
submission, including uncertain-operation retries, until an authoritative refresh succeeds. A
write is acknowledged only after commit. A complete view restores on refresh/new tabs within the
same valid login; independent sessions have independent views, each starting at All/All/due_date/asc.
Missing/invalid/unreadable view silently uses complete defaults; failed remembering keeps current
selections usable and never blocks otherwise available task data/actions. No offline guarantee, sharing, native apps,
saved-data reset, user-facing restore, password recovery, or account transfer. Pending-operation
recovery is memory-only; browser interruption can lose its identity and leave a duplicate or delayed
mutation risk, as an accepted limitation.

**Scale/Scope**: One logical list per account. Complete filtered snapshots without pagination initially; maximum accounts/list size unspecified. One application service plus database. Production support contact is a required release configuration input.

**TD-003 operating hypothesis:** technical-design §4 specifies 100 registered/50 active accounts,
100 visible tabs at nominal 6.7 polls/s, 1,000 tasks/account with stress variations and lifetime
receipt growth. Candidate hosts are app 2 vCPU/4 GiB (four sync workers) and DB 2 vCPU/4 GiB/40 GiB
disk. Forty DB connections are explicitly budgeted, with separate bounded primary/optional-view/
limiter pools. These are engineering assumptions, not product caps or measured capacity. Aligned
2 MiB body limits, finite queues and DB/endpoint deadlines provide backpressure; no response
truncation or receipt expiry. Measurement/review triggers apply before launch.

**R2-Rev8 / R3 handoff:** CA-007 allows conventional anonymous cookie response selection; no first
browser winner/competitor conflict or shared election obligation. Authenticated requests/stale
forms retain R2-NAV-001. CA-006 retains server ownership/intended-login guards, known-stale
discard/observed-change clearing and boundary revalidation; no atomic rendering or unnecessary
fresh round trip per cached interaction/normal reply. Approved AC-010/RA-FD006-001 covers
unobserved prior-authorised display/in-flight completion, not API isolation or known-stale exceptions.
CA-008 retains confirmed intended logout, prompt signal/receiver clear/navigation and safe recovery,
without guaranteed delivery/atomic clearing. FD-001, native security and silent view isolation stay.

Old family/transition/maintenance and Option B proposals remain historical inputs. R3-Rev10
selects ADR-AS-004 as an unapproved candidate: native Session guard, small logout receipt,
cookie-response adapter, local generations and optional BroadcastChannel. No automatic TD
closure. P1 stays paused; see [handoff](review-followups/r3-rev10-handoff.md).

**TD-009 retained:** optional-view bounded reader with 150 ms total wait/independent task
transaction remains technical-design §3, unchanged from the reviewed mechanism.

**TD-004/005 security:** independent HMAC receipt keys, stored canonical/digest/key versions and
historical replay/rotation preflight; private upstream behind trusted Nginx source normalization
plus shared DB auth counters. Operational throttle buckets and digest metadata are in the data
model. No new external service or identity-recovery flow is added.

## Constitution Check

The table below is the historical Plan-rerun author check, retained unchanged. Its pending-decision
phrases do not override the subsequent Scope v1.2/R2-Rev8 approvals or paused P1 status above.

*Gate re-evaluated before research and after artifact reconciliation against constitution v1.0.0.
These are author checks for design quality, not independent phase approvals.*

| Principle / constraint | Before research | After design / evidence |
|---|---|---|
| I. Explicit, testable requirements | PASS: Scope 1.1 FR-001–016 / AC-001–012 approved | R2-Rev5 approved behaviour retained; R3-Rev7 rotation/CSRF specified, anonymous delivery/decision still HOLD |
| II. Ambiguity is a blocker | Scope's three product choices settled by DEC-AM-001-01–03 | Author concurrency/storage choices are explicit for R2/R3 review; TD-001/008 correction and TD-006/007 downstream evidence tracked; gate HOLD |
| III. Simple, modular architecture | PASS: greenfield, no legacy stack | PASS: one app; ordinary polling; receipts address a concrete lost-response hazard |
| IV. Separation of concerns | PASS: distinct layers proposed | PASS: UI adapters → domain/application services → repository protocols → ORM adapters; UI never performs ORM operations |
| V. Automated quality and decision records | PASS: risk-appropriate testing required | Research, handoff and planned AM-001/task regression scenarios documented; no runtime tests executed |
| Delivery workflow and exceptions | Authorised Plan/checklist/FD reconciliation only | R2-Rev4 retained; HOLD before R3 sign-off; historical records preserved; no downstream approval inferred |

The constitution's unknown historical ratification date is governance metadata. This feature uses its stated version without amending it.

## Project Structure

### Documentation (this feature)

```text
specs/001-personal-task-manager/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── http.md
│   └── ui.md
├── checklists/
│   ├── requirements.md
│   └── feature.md
└── review-followups/am001-r2-review-handoff.md
```

`tasks.md` is reserved for the task-generation workflow.

### Source Code (planned; application code does not yet exist)

```text
pyproject.toml
requirements.lock
compose.yaml
.env.example
manage.py
config/                    # settings, composition, URLs, WSGI
accounts/                  # minimal AbstractUser model and auth adapters
taskify/
├── domain/                # DTOs, validation, sorting, transitions; no Django imports
├── application/           # command/query services and repository protocols
├── persistence/           # ORM models, migrations, repository adapters
├── web/                   # forms, serializers, HTTP adapters; services only
├── templates/             # responsive pages and accessible forms
└── static/taskify/        # CSS; controller, transport, preferences, calendar JS
tests/
├── unit/
├── integration/
├── contract/
└── e2e/
```

**Structure Decision**: One Python project keeps deployment and auth simple. Register `accounts` and `taskify.persistence` as Django apps. Composition in `config` supplies ORM adapters to services. Browser calendar and view-transport modules are separated from DOM rendering. Use Django's supported forms/session infrastructure for authentication. A session-bound view row avoids a custom whole-session merge backend; its lock/lifecycle guard is still a proposed technical mechanism for R3 review.

## Delivery and Verification Boundaries

Establish dependency locking/database/auth first, then pure rules and transactional repositories, then HTTP/UI contracts and browser interactions. Validate according to [quickstart.md](quickstart.md). Production readiness includes HTTPS, migrations, backups, secrets, auth rate limiting and a configured support destination. Operational backups do not expose an in-application restore feature.

This sequence is future work gated by targeted independent R3 review and explicit human R3
approval. See [current handoff](review-followups/r3-rev10-handoff.md) and the
[historical AM-001 handoff](review-followups/am001-r2-review-handoff.md) for traceability and
residual findings. TD-006 recovery/diagnostic readiness stays with Operations at R5; TD-007
canonicalization compatibility stays with maintenance/R4 and future upgrades. TD-001 cross-tab
privacy, TD-003 capacity, TD-004 receipt digest privacy and TD-005 throttling are not resolved by
replanning view convenience. R1-AM-004 archival Section 13 gap is DEFERRED per R1 Section 8.

## Complexity Tracking

No new constitutional exception is requested. Receipts prevent a lost create response followed by remote deletion from causing resurrection on retry. Per-account locking/revision supplies consistent snapshots without messaging infrastructure. Both are justified by cross-device correctness. SessionViewState replaces the old account-preference row with the same database and no optimistic preference version machinery; it isolates view convenience from task persistence and whole-session middleware saves.

## R2-Rev8 incorporation / R3 gate

FD-009 is RESOLVED_VERIFIED from independent R2-Rev8 §2; §4 records the explicit human approval.
R2-Rev7's NOT_READY report remains unchanged. The [evidence map](review-followups/r2-rev8-incorporation-evidence.md)
and [R3 handoff](review-followups/r2-rev8-r3-handoff.md) trace the targeted incorporation.
R3 must reassess retained security/lifecycle/recovery requirements proportionately under the
review-remediation policy, not require old winner/atomic-render/delivery guarantees.
The historical Constitution Check is not a new run. Technical-design §§3–8 and TD-006/007
owners/exits stay unchanged. R3-Rev10 selects an author technical candidate within those bounds,
not an approved architecture, added risk acceptance or downstream gate.
