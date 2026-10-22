# Quickstart Validation Guide

This is a planned validation/run guide, not evidence that application code exists. At planning time the repository contains specification tooling and design documents only. The application, dependency lock, Compose configuration and tests named below must be produced during implementation before these commands run.

Contracts: [HTTP](contracts/http.md), [UI](contracts/ui.md). Entities and ordering: [data-model.md](data-model.md). Decisions: [research.md](research.md).

## Prerequisites and local setup

Python 3.13, Docker with Compose, supported desktop/mobile browsers, and network access for initial dependencies/browser engines. Use PostgreSQL 17 locally and in integration tests; SQLite is not an acceptable substitute for locking tests. Use disposable accounts/data. The local database role must be able to create an isolated test database; production credentials must not have that privilege.

From repository root, after implementation has supplied these files:

```sh
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.lock
cp .env.example .env
```

The example environment must document shell-compatible local-only values for DJANGO_SECRET_KEY, DJANGO_DEBUG, DJANGO_ALLOWED_HOSTS, PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD and SUPPORT_CONTACT. Review .env locally; never commit it. Load it without printing secrets:

```sh
set -a
source .env
set +a
docker compose up -d db
docker compose exec db pg_isready
python manage.py migrate
python manage.py check
python -m playwright install chromium firefox webkit
python manage.py runserver 127.0.0.1:8000
```

Expected: migration/check success, database ready, registration page at http://127.0.0.1:8000/accounts/register/. Localhost is sufficient for browser secure-context UUID generation. For actual mobile access, use a development HTTPS origin and appropriate allowed hosts/cookie configuration; do not use production accounts.

## Automated validation

Planned test locations and invocation:

```sh
python -m pytest tests/unit
python -m pytest tests/integration tests/contract
python -m pytest tests/e2e --browser chromium --browser firefox --browser webkit
python manage.py makemigrations --check --dry-run
```

Configure pytest-django to use isolated PostgreSQL test databases and the browser suite to start a live Django test server. Integration/concurrency tests require separate connections and transaction-enabled tests; transaction-wrapped unit fixtures alone cannot prove lock behavior. Browser cases include desktop and mobile viewport/context variants.

Before production release, provide production settings and run:

```sh
python manage.py check --deploy
```

Expected: relevant automated suites pass, no missing migrations, and production security findings are resolved. Document browser/engine versions, real-device results and observed timings; no unapproved numerical success threshold is introduced.

## End-to-end scenarios

1. **Private registration, returning access, and credentials** — FR-010/015/016; AC-009/010/012; US4. Register account A with username/password and no invitation. Invalid username, duplicate/case-only duplicate, password mismatch and weak password show corrections. Before registration, confirm the visible warning that forgotten credentials cannot be recovered. From login and support, confirm that no automated/manual password reset, recovery, or account transfer is offered or promised. Sign in on desktop and a separate mobile browser/context. Register B separately. Direct requests from B must not read/edit/complete/reopen/delete A's tasks; unauthenticated API requests return 401. Logout clears displayed account data. Session restoration/history navigation does not reveal cached tasks after revalidation fails. Test POST CSRF rejection and login/logout behavior. **R2-T12 / FD-004**: verify all credential-loss disclosures and absence of recovery routes.

2. **Create and validate** — FR-001/002/003; AC-002; US1. Create using only title; expect active, Medium, absent description/date. Create with all optional details and valid past/leap-day dates; retain supplied values. Reject whitespace-only titles, impossible dates and date-time input. Title/description markup displays as text. Refresh and sign in again; committed values persist.

3. **Maintain and delete** — FR-004/005/006; AC-001/003; US2. Edit all details, then clear description/date/priority; expect null/null/Medium. Complete, then reopen; original details remain. Cancel deletion; task stays. Confirm deletion; task disappears after refresh and on the other device; no restore UI. Verify deletion removes the task row and receipts do not retain task contents.

4. **Calendar semantics** — FR-011; AC-004. With a controlled browser clock/timezone, verify yesterday is overdue only when active; today/tomorrow/undated are not. Complete/reopen and cross local midnight. Test a timezone differing from UTC and a year boundary; saved date remains identical. Verify two devices use their own local calendar and do not shift the date itself.

5. **Filter, sort and account preferences** — FR-007/012/013; AC-005/006/007/011; US3. Fixtures span both statuses, all priorities, dates 2026-10-19/21/30, undated tasks, tied dates/priorities/creation timestamps and titles differing in case/Unicode normalization. Check each filter and combined AND, then all four methods in both directions. Undated stays last; due-date ties stay high-priority then oldest regardless of direction; remaining ties follow the data model. Successful no-match result shows its specific empty state. Restart the page/session and restore server-saved filters; sort resets to due_date/asc. Sign in on a second device and verify the last acknowledged account preference restores there.

**R2-T06 / FD-005 preference reconciliation** — Drop a preference PUT response after the server
commits it; the UI says only that it could not confirm the save, then re-reads the authoritative
pair/version and marks the same pair saved. Return a 409 for a stale expected version; confirm that
the UI treats it as a confirmed rejection without claiming another device caused it, re-reads the
authoritative preference, and requires explicit approval before overwriting a different pair.
Rapidly change filters while a save is pending; assert that only one PUT is active, only the newest
queued pair is later sent, and a late acknowledgement or re-read cannot replace the newest visible
selection or regress the confirmed version. Fail the authoritative re-read and verify the visible
pair remains unconfirmed with no automatic overwrite. Browser storage availability cannot change
these behaviours.

6. **Saved changes across devices** — FR-008; AC-009; SC-004. With A signed in on two isolated browser contexts, create/edit/complete/reopen/delete from one; focus/refresh or visible polling on the other must converge after successful synchronisation. Close/reopen and sign in again. No full-list replacement request occurs. Record that the polling interval is a design setting, not an availability SLA.

7. **Load/refresh failure protection** — FR-009/014; AC-008. Inject failed initial snapshot and malformed JSON: show unavailable, Retry/support, block task writes, never claim an empty saved list. After one successful load inject a failed refresh: retain visibly stale rows and lock writes. Repeat failure: explicitly recommend support. Restore the endpoint and retry: valid authoritative data replaces stale data; writes unlock if no pending unknown operation. If an operation is unknown, keep all task writes—including its retry—disabled until the valid snapshot succeeds; only then enable its exact same-ID/payload retry. No saved-data reset appears.

8. **Failed, uncertain, and interrupted writes** — FR-008/009/014; SC-004. Return validation errors and known rollback errors; retain drafts and saved values. Drop a response after commit; show unknown outcome. After a successful saved-task snapshot, retry identical operation/key: one effect and historical acknowledgement. Repeat with create → dropped response → deletion on device two → delayed create retry: task stays deleted. **R2-T05 / FD-001**: simulate an uncommitted edit/delete with a lost response, then fail the snapshot; assert that selecting Retry cannot submit any task mutation until the snapshot succeeds, then assert the original operation ID and unchanged payload are used. **R2-T07 / FD-003**: interrupt the browser while an outcome is unknown; verify the warning does not imply rollback, no durable retry identity is restored, and re-entry guidance requires reviewing the current list before a new attempt. Verify receipt/task/revision atomic rollback on injected transaction failure and same-key/different-payload rejection. If reconciliation fails, remain locked with support guidance.

9. **Concurrency and response ordering** — FR-008/009/010; SC-004. Edit the same version from two contexts: one succeeds, the other gets an explicit conflict preserving its draft. Race confirmed delete with a newer edit: stale delete is rejected. Delay a pre-write GET until after acknowledgement: it cannot replace newer state. Change filters rapidly: only the latest query renders. Logout/account switch while requests are pending: old-account responses are discarded. A successful empty snapshot clears rows; a failed snapshot never does.

## Completion evidence

Record command results, browser/device versions and mapped scenario outcomes in implementation validation notes. Verify HTTPS, secure cookies, no-store headers, proxy authentication throttling, least-privilege production database configuration, backup operation and a working support contact. These are release checks, not provisioning actions authorized by this planning command.

Do not run tests against production or create a user-facing reset/restore mechanism to facilitate validation. Reuse isolated test databases and fixtures.
