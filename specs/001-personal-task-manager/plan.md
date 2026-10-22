# Implementation Plan: Personal Task Manager

**Feature identifier**: `001-personal-task-manager` | **Date**: 2026-10-22 | **Spec**: [spec.md](spec.md)

**Input**: Scope Approved specification, reconciled 2026-10-22.
**Git context**: Actual checkout is `main`; setup reports `001-personal-task-manager` from its feature-directory fallback. Planning does not create or switch branches.

## Summary

Build one responsive Django application backed by PostgreSQL. Users self-register with username/password and manage their own tasks through session-authenticated interfaces. Server-rendered pages and a small JavaScript controller refresh complete filtered snapshots on focus and periodically while visible. Domain services own validation, ordering and state transitions; repositories own account-scoped persistence. Durable operation receipts and task versions prevent duplicate retries and silent overwrites across devices.

Design decisions delegated by Scope are resolved in [research.md](research.md). The approved specification remains unchanged. This workflow ends at Phase 1; `tasks.md` belongs to the subsequent task-generation workflow.

## Technical Context

**Language/Version**: Python 3.13, maintained patch; browser JavaScript ES modules and HTML/CSS.

**Primary Dependencies**: Django 5.2 LTS, Psycopg 3, Gunicorn for production. Exact maintained patches locked during implementation. No SPA, separate auth provider, task queue or WebSocket service.

**Storage**: PostgreSQL 17, maintained patch, for users, sessions, tasks, account list revisions,
durable operation receipts, and account-owned filter preferences. Browser storage holds no task
data, credentials, pending operations, or preferences.

**Testing**: pytest, pytest-django and pytest-playwright; pure domain tests, real-PostgreSQL integration/concurrency tests, HTTP contract tests and browser journeys.

**Target Platform**: Desktop Chrome/Edge/Firefox/Safari; mobile Chrome on Android/Safari on iOS. Record current stable versions during release validation. Automate Chromium/Firefox/WebKit and perform actual mobile browser smoke checks. JavaScript, cookies and connectivity required.

**Project Type**: Single web application and database, with a same-origin JSON interface supporting its UI.

**Performance Goals**: No approved numeric latency, task-count or usability thresholds. Visible-page polling every 15 seconds is an adjustable design parameter, not an SLA. Record observed timings/list sizes; do not reinstate removed SC-001–SC-003.

**Constraints**: Ownership enforced for every read/write; successfully committed changes persist and
converge across devices. Date-only due dates. Failed load/refresh blocks every task-mutation
submission, including uncertain-operation retries, until an authoritative refresh succeeds. A
write is acknowledged only after commit. Account-owned filter preferences restore across sessions
and devices with explicit version-conflict handling. No offline guarantee, sharing, native apps,
saved-data reset, user-facing restore, password recovery, or account transfer. Pending-operation
recovery is memory-only; browser interruption can lose its identity and leave a duplicate or delayed
mutation risk, as an accepted limitation.

**Scale/Scope**: One logical list per account. Complete filtered snapshots without pagination initially; maximum accounts/list size unspecified. One application service plus database. Production support contact is a required release configuration input.

## Constitution Check

*Gate evaluated before Phase 0 and repeated after Phase 1 against constitution v1.0.0.*

| Principle / constraint | Before research | After design / evidence |
|---|---|---|
| I. Explicit, testable requirements | PASS: FR-001–FR-016 and AC-001–AC-012 approved | PASS: HTTP/UI contracts define observable success/failure; quickstart maps coverage |
| II. Ambiguity is a blocker | PASS: Scope explicitly delegates authentication, sync, recovery, timezone, ties and preferences to Design | PASS: research resolves those choices; no unresolved clarifications or unapproved product assumptions |
| III. Simple, modular architecture | PASS: greenfield, no legacy stack | PASS: one app; ordinary polling; receipts address a concrete lost-response hazard |
| IV. Separation of concerns | PASS: distinct layers proposed | PASS: UI adapters → domain/application services → repository protocols → ORM adapters; UI never performs ORM operations |
| V. Automated quality and decision records | PASS: risk-appropriate testing required | PASS: research records decisions; unit, integration, contract and browser checks specified |
| Delivery workflow and exceptions | PASS: implementation not started | PASS: no exceptions; planned tests are not claimed as executed |

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
└── checklists/requirements.md
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

**Structure Decision**: One Python project keeps deployment and auth simple. Register `accounts` and `taskify.persistence` as Django apps. Composition in `config` supplies ORM adapters to services. Browser calendar and preference-transport modules are separated from DOM rendering. Use Django's supported forms/session infrastructure for authentication.

## Delivery and Verification Boundaries

Establish dependency locking/database/auth first, then pure rules and transactional repositories, then HTTP/UI contracts and browser interactions. Validate according to [quickstart.md](quickstart.md). Production readiness includes HTTPS, migrations, backups, secrets, auth rate limiting and a configured support destination. Operational backups do not expose an in-application restore feature.

## Complexity Tracking

No constitutional exceptions. Receipts prevent a lost create response followed by remote deletion from causing resurrection on retry. Per-account locking/revision supplies consistent snapshots without messaging infrastructure. Both are justified by cross-device correctness.
