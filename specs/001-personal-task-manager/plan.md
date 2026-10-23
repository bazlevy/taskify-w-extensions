# Implementation Plan: Personal Task Manager

**Feature identifier**: `001-personal-task-manager` | **Date**: 2026-10-23 | **Spec**: [spec.md](spec.md)

**Input**: Scope 1.1 / AM-001, human-approved by Barry Levy on 2026-10-23 in R1-Rev4 Section 8 for
candidate SHA-256 `61062311e5d1423f1d13f485c555158964e09f0548d13263bfefa0f048447fd4`.
**Design status**: AM-001 author reconciliation; targeted independent R2 review and human approval
PENDING. Historical R2-Rev3 approval applies to its original package only. R3 remains NOT READY;
all TD-001–007 findings remain OPEN. No Tasks, Build or downstream phase approval.
**Git context**: Actual checkout is `main`; setup reports `001-personal-task-manager` from its feature-directory fallback. Planning does not create or switch branches.

## Summary

Build one responsive Django application backed by PostgreSQL. Users self-register with username/password and manage their own tasks through session-authenticated interfaces. Server-rendered pages and a small JavaScript controller refresh complete filtered snapshots on focus and periodically while visible. Domain services own validation, ordering and state transitions; repositories own account-scoped persistence. Durable operation receipts and task versions prevent duplicate retries and silent overwrites across devices.

AM-001 decisions and research are recorded in [research.md](research.md). Scope received only the
authorised approval-metadata update; product rules remain byte-identical to the approved candidate.
The existing implementation stack and unrelated design decisions are retained. This Plan rerun
ends with reviewable design artifacts, before R2 sign-off; `tasks.md` is not generated.

## Technical Context

**Language/Version**: Python 3.13, maintained patch; browser JavaScript ES modules and HTML/CSS.

**Primary Dependencies**: Django 5.2 LTS, Psycopg 3, Gunicorn for production. Exact maintained patches locked during implementation. No SPA, separate auth provider, task queue or WebSocket service.

**Storage**: PostgreSQL 17, maintained patch, for users, sessions, tasks, account list revisions,
durable operation receipts and a small SessionViewState bound to each existing login session.
This replaces account-owned filter preferences; it remembers filters and sorting without
cross-login/device inheritance. No new storage service or browser-persisted settings. Session save,
logout/rotation concurrency and failure isolation remain subject to R3 assessment.

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

## Constitution Check

*Gate re-evaluated before research and after artifact reconciliation against constitution v1.0.0.
These are author checks for design quality, not independent phase approvals.*

| Principle / constraint | Before research | After design / evidence |
|---|---|---|
| I. Explicit, testable requirements | PASS: Scope 1.1 FR-001–016 / AC-001–012 approved | AM-001 cases map silent fallback, full tuple, lifetime and retained task locks; independent R2 evaluation pending |
| II. Ambiguity is a blocker | Scope's three product choices settled by DEC-AM-001-01–03 | Author concurrency/storage choices are explicit for R2/R3 review; TD-001–007 remain tracked; overall downstream gate HOLD |
| III. Simple, modular architecture | PASS: greenfield, no legacy stack | PASS: one app; ordinary polling; receipts address a concrete lost-response hazard |
| IV. Separation of concerns | PASS: distinct layers proposed | PASS: UI adapters → domain/application services → repository protocols → ORM adapters; UI never performs ORM operations |
| V. Automated quality and decision records | PASS: risk-appropriate testing required | Research, handoff and planned AM-001/task regression scenarios documented; no runtime tests executed |
| Delivery workflow and exceptions | Authorised Plan/checklist/FD reconciliation only | HOLD before R2 sign-off; preserve historical approvals and review records; no design approval inferred |

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

This sequence is future work gated by targeted R2 approval, independent R3-Rev2 and human R3
approval. See [handoff](review-followups/am001-r2-review-handoff.md) for AM-001 traceability and
residual findings. TD-006 recovery/diagnostic readiness stays with Operations at R5; TD-007
canonicalization compatibility stays with maintenance/R4 and future upgrades. TD-001 cross-tab
privacy, TD-003 capacity, TD-004 receipt digest privacy and TD-005 throttling are not resolved by
replanning view convenience. R1-AM-004 archival Section 13 gap is DEFERRED per R1 Section 8.

## Complexity Tracking

No new constitutional exception is requested. Receipts prevent a lost create response followed by remote deletion from causing resurrection on retry. Per-account locking/revision supplies consistent snapshots without messaging infrastructure. Both are justified by cross-device correctness. SessionViewState replaces the old account-preference row with the same database and no optimistic preference version machinery; it isolates view convenience from task persistence and whole-session middleware saves.
