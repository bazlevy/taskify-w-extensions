# Research: Personal Task Manager

**Date**: 2026-10-22. Authority: [spec.md](spec.md). These decisions resolve details explicitly delegated to Design. Repository inspection found no existing application stack. Two focused research agents checked authentication and synchronisation patterns against primary documentation.

## 1. Runtime and architecture

**Decision**: Python 3.13, Django 5.2 LTS, Psycopg 3 and PostgreSQL 17, using maintained patches locked during implementation. Django templates/CSS and small browser ES modules provide one same-origin application.

**Rationale**: Django supplies supported auth, forms, sessions, CSRF and migrations. The chosen runtime/database majors are compatible. [Django 5.2 release notes](https://docs.djangoproject.com/en/5.2/releases/5.2/), [PostgreSQL backend](https://docs.djangoproject.com/en/5.2/ref/databases/#postgresql-notes).

**Alternatives considered**: SPA plus API duplicates infrastructure; managed auth adds another service; local-only storage fails cross-device availability; PostgreSQL is preferable to SQLite for the planned multi-device transactional deployment and concurrency tests.

## 2. Authentication and ownership

**Decision**: Self-register with username, password and confirmation. Declare a minimal AbstractUser subclass before the first migration. Use a UserCreationForm subclass with standard username/password validation and case-insensitive duplicate rejection; retain Django's username matching on login. Redirect registration success to sign-in. Use LoginView, POST LogoutView and database-backed sessions. Email is not required. Forgotten credentials link to support; automated password reset is not included.

**Rationale**: These supported building blocks meet private self-registration without email/provider dependencies. Every task repository method requires authenticated owner identity; unknown and foreign IDs return indistinguishable 404 responses. Django model permissions alone do not enforce object ownership. [Auth views/forms](https://docs.djangoproject.com/en/5.2/topics/auth/default/), [Custom user models and object permissions](https://docs.djangoproject.com/en/5.2/topics/auth/customizing/), [Sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/).

**Alternatives considered**: Federated/email login adds dependencies without a Scope need. Anonymous accounts do not meet the account boundary. Built-in User is sufficient today but declaring the minimal subclass initially avoids later substitution.

## 3. Authoritative snapshots and synchronisation

**Decision**: Fetch complete filtered snapshots on initial load, after writes, on focus/visibility return, on Retry and every 15 seconds while visible. Coalesce triggers and allow one normal refresh at a time. Stop polling while hidden. Never upload a replacement list. Use account revision, request/query generation and session generation to reject stale responses. A snapshot materialises task DTOs and revision within one short transaction holding the account-state row lock; filtering/sorting/serialization follow outside the lock.

**Rationale**: Periodic authenticated reads of shared storage provide convergence without another service. Hidden tabs throttle timers, so focus refresh matters. All writers acquire the same account-state lock first; no external work occurs inside a transaction. [Django transactions](https://docs.djangoproject.com/en/5.2/topics/db/transactions/), [Row locking](https://docs.djangoproject.com/en/5.2/ref/models/querysets/#select-for-update), [Page Visibility API](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API).

**Alternatives considered**: WebSockets/change streams add infrastructure. Full-list uploads risk destructive overwrite after a failed load. The polling interval is configurable, not a numeric product guarantee.

## 4. Conflict and uncertain-write recovery

**Decision**: Every mutation carries a stable operation UUID; creation also has a stable task UUID. Commit the mutation, list revision increment and durable receipt atomically. Check receipt before task existence/version. Identical replay returns its historical acknowledgement; a different payload with the same key returns conflict. Receipts retain metadata/digests, not task contents, for the account lifetime. Require expected task version on edit, status change and delete; reject stale versions with 409. No automatic merge.

**Rationale**: A UUID alone cannot prevent resurrection when a create commits, its response is lost, another device deletes the task and the first retries. A receipt proves the old create already happened. A timeout does not prove rollback. Keep the exact payload/key for explicit retry, then refresh current state; a historical receipt is not a current task representation. [PostgreSQL isolation](https://www.postgresql.org/docs/17/transaction-iso.html), [Fetch status/error handling](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch).

**Alternatives considered**: Last-write-wins silently discards concurrent work. Expiring receipts permit late duplicate operations. Storing full responses retains deleted text unnecessarily. An offline write queue adds an unsupported operating mode.

## 5. Errors, drafts and support

**Decision**: Disable task mutations until initial load succeeds. Any later refresh failure preserves a visibly stale snapshot and locks mutations until refresh succeeds. Show Retry and support guidance immediately; after a second consecutive failure explicitly recommend support. No reset action. Validation/definite rollback errors preserve drafts. Timeout, broken response or unclassified server error enters outcome-unknown; preserve the pending request in memory and reconcile by replaying its exact key/payload before allowing other mutations. A successful refresh alone does not resolve that pending operation.

Do not replace open drafts during polling. Remote edit/deletion requires review or cancellation; never silently recreate. On session expiry/logout, hide and clear task views/drafts. A browser reload can discard an unsaved draft; subsequent snapshots reflect whatever actually committed.

**Rationale**: This distinguishes saved state from attempted state and protects existing data. A support page supplies the configured contact and request reference. Missing support configuration blocks release, not design.

**Alternatives considered**: Optimistic success feedback can lie about persistence; self-service reset is explicitly excluded; offline task caching is unnecessary.

## 6. Calendar and ordering

**Decision**: Store nullable date-only due dates and serialize strict real `YYYY-MM-DD` values, years 0001–9999. Use the viewing device's local calendar for overdue, recalculated at local midnight and on focus. Completed/undated tasks never count as overdue. Devices in different timezones may show different overdue labels.

Due Date ordering always places undated tasks last. Within each date, including the undated group, order priority High → Medium → Low, creation timestamp ascending, UUID ascending. Other sorts reverse only their primary key: priority rank, creation timestamp, or NFC-normalized casefolded title in Unicode code-point order. Primary ties use creation timestamp ascending then UUID ascending; Date Created ties use UUID alone. Display original title spelling.

**Rationale**: This completes delegated timezone/tie behavior without inventing a due time or user timezone preference. Domain sorting runs in Python independently of database collation. [PostgreSQL date/time](https://www.postgresql.org/docs/17/datatype-datetime.html), [Python normalization](https://docs.python.org/3.13/library/unicodedata.html), [Python casefold](https://docs.python.org/3.13/library/stdtypes.html#str.casefold).

**Alternatives considered**: UTC instants shift date-only values; reversing an entire list violates undated-last and fixed secondary rules; locale-sensitive title ordering varies between environments. This release chooses explicit locale-neutral order.

## 7. Preferences

**Decision**: Versioned localStorage keys include the account's opaque ID. Restore status/priority filters, defaulting All/All; invalid/corrupt values revert to defaults. Preferences do not sync across devices. Reset sort to Due Date earliest-first on each fresh page session. Logout clears in-memory state but retains account-scoped filter preferences for return visits. If storage is unavailable, show a restoration warning and keep session filtering usable.

**Rationale**: Scope mandates restoration while delegating cross-device behavior and sort persistence. Namespacing prevents applying one account's filters to another. No tasks/credentials enter this store.

**Alternatives considered**: Server preferences add writes without a cross-device requirement. A single shared browser key mixes accounts. Persisted sort is allowed but unnecessary.

## 8. Security, schema and testing

**Decision**: Keep CSRF middleware, escaped templates, safe DOM text rendering, same-origin credentials and no permissive CORS. Production requires HTTPS, secure/HttpOnly session cookies, valid hosts/secrets, `DEBUG=False`, no-store authenticated responses and proxy auth throttling. On history restoration, hide task views until session revalidation. Use indexed owner foreign keys, real date/timestamp types, checked enums/versions and short transactions with consistent lock order. Use least-privilege database access and backups.

**Rationale**: Account isolation must withstand crafted requests, not merely hidden controls. The Postgres best-practices skill informed indexes, types and transaction boundaries. Browser tests prove interactions; real-PostgreSQL tests prove ownership/concurrency; pure tests prove business rules. [CSRF](https://docs.djangoproject.com/en/5.2/ref/csrf/), [Deployment checklist](https://docs.djangoproject.com/en/5.2/howto/deployment/checklist/), [Playwright browsers](https://playwright.dev/python/docs/browsers).

**Alternatives considered**: Browser-only tests miss transaction races. SQLite cannot prove PostgreSQL locking. Additional numeric success thresholds would contradict the approved Scope.

## Resolution

All design unknowns are resolved. Hostname, secrets and support contact are deployment inputs. No Scope amendment, application implementation or infrastructure provisioning occurred. Constitution gates pass before and after design; see [plan.md](plan.md).
