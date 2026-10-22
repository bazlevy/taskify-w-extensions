# Research: Personal Task Manager

**Date**: 2026-10-22. Authority: [spec.md](spec.md). These decisions resolve details explicitly delegated to Design. Repository inspection found no existing application stack. Two focused research agents checked authentication and synchronisation patterns against primary documentation.

## 1. Runtime and architecture

**Decision**: Python 3.13, Django 5.2 LTS, Psycopg 3 and PostgreSQL 17, using maintained patches locked during implementation. Django templates/CSS and small browser ES modules provide one same-origin application.

**Rationale**: Django supplies supported auth, forms, sessions, CSRF and migrations. The chosen runtime/database majors are compatible. [Django 5.2 release notes](https://docs.djangoproject.com/en/5.2/releases/5.2/), [PostgreSQL backend](https://docs.djangoproject.com/en/5.2/ref/databases/#postgresql-notes).

**Alternatives considered**: SPA plus API duplicates infrastructure; managed auth adds another service; local-only storage fails cross-device availability; PostgreSQL is preferable to SQLite for the planned multi-device transactional deployment and concurrency tests.

## 2. Authentication and ownership

**Decision**: Self-register with username, password and confirmation. Declare a minimal AbstractUser subclass before the first migration. Use a UserCreationForm subclass with standard username/password validation and case-insensitive duplicate rejection; retain Django's username matching on login. Redirect registration success to sign-in. Use LoginView, POST LogoutView and database-backed sessions. Email is not required. The registration form warns that forgotten credentials cannot be recovered. Forgotten-credentials and support views state that the first release provides no automated or manual password reset, credential recovery, or account transfer; support may answer questions only.

**Rationale**: These supported building blocks meet private self-registration without email/provider dependencies. Every task repository method requires authenticated owner identity; unknown and foreign IDs return indistinguishable 404 responses. Django model permissions alone do not enforce object ownership. [Auth views/forms](https://docs.djangoproject.com/en/5.2/topics/auth/default/), [Custom user models and object permissions](https://docs.djangoproject.com/en/5.2/topics/auth/customizing/), [Sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/).

**Alternatives considered**: Federated/email login adds dependencies without a Scope need. Anonymous accounts do not meet the account boundary. Password recovery would require an approved identity-verification policy and channel that this release does not have. Built-in User is sufficient today but declaring the minimal subclass initially avoids later substitution.

## 3. Authoritative snapshots and synchronisation

**Decision**: Fetch complete filtered snapshots on initial load, after writes, on focus/visibility return, on Retry and every 15 seconds while visible. Coalesce triggers and allow one normal refresh at a time. Stop polling while hidden. Never upload a replacement list. Use account revision, request/query generation and session generation to reject stale responses. A snapshot materialises task DTOs and revision within one short transaction holding the account-state row lock; filtering/sorting/serialization follow outside the lock.

**Rationale**: Periodic authenticated reads of shared storage provide convergence without another service. Hidden tabs throttle timers, so focus refresh matters. All writers acquire the same account-state lock first; no external work occurs inside a transaction. [Django transactions](https://docs.djangoproject.com/en/5.2/topics/db/transactions/), [Row locking](https://docs.djangoproject.com/en/5.2/ref/models/querysets/#select-for-update), [Page Visibility API](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API).

**Alternatives considered**: WebSockets/change streams add infrastructure. Full-list uploads risk destructive overwrite after a failed load. The polling interval is configurable, not a numeric product guarantee.

## 4. Conflict and uncertain-write recovery

**Decision**: Every mutation carries a stable operation UUID; creation also has a stable task UUID. Commit the mutation, list revision increment and durable receipt atomically. Check receipt before task existence/version. Identical replay returns its historical acknowledgement; a different payload with the same key returns conflict. Receipts retain metadata/digests, not task contents, for the account lifetime. Require expected task version on edit, status change and delete; reject stale versions with 409. No automatic merge. A retry capable of committing a task mutation is permitted only after a valid saved-task snapshot has loaded; it uses the same operation ID and unchanged payload, subject to the normal receipt and version checks.

**Rationale**: A UUID alone cannot prevent resurrection when a create commits, its response is lost, another device deletes the task and the first retries. A receipt proves the old create already happened. A timeout does not prove rollback. Keep the exact payload/key for explicit retry, then refresh current state; a historical receipt is not a current task representation. [PostgreSQL isolation](https://www.postgresql.org/docs/17/transaction-iso.html), [Fetch status/error handling](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch).

**Alternatives considered**: Last-write-wins silently discards concurrent work. Expiring receipts permit late duplicate operations. Storing full responses retains deleted text unnecessarily. An offline write queue adds an unsupported operating mode.

## 5. Errors, drafts and support

**Decision**: Disable task mutations until initial load succeeds. Any later refresh failure preserves a visibly stale snapshot and locks every new task-mutation submission, including a retry of an outcome-unknown operation, until a valid saved-task snapshot succeeds. Show Retry and support guidance immediately; after a second consecutive failure explicitly recommend support. No reset action. Validation/definite rollback errors preserve drafts. Timeout, broken response or unclassified server error enters outcome-unknown; preserve the pending request in memory. After a valid snapshot succeeds, the user may recover by replaying that exact key/payload; a successful snapshot alone does not resolve the pending operation.

Do not replace open drafts during polling. Remote edit/deletion requires review or cancellation; never silently recreate. On session expiry/logout, hide and clear task views/drafts. Pending operation identity and payload exist only in browser memory: browser reload, close, crash, or navigation can discard them. An early snapshot after interruption is not proof that an earlier request rolled back; it may commit later. The UI warns users to wait for a confirmed result, and after interruption directs them to sign in/refresh and review the current list before re-entering an action. A repeated user attempt can create a duplicate or a delayed first mutation can later appear; this accepted residual risk has no durable pending-operation journal or exactly-once user-intent guarantee.

**Rationale**: This distinguishes saved state from attempted state and protects existing data. A support page supplies the configured contact and request reference. Missing support configuration blocks release, not design.

**Alternatives considered**: Optimistic success feedback can lie about persistence; self-service reset is explicitly excluded; offline task caching is unnecessary.

## 6. Calendar and ordering

**Decision**: Store nullable date-only due dates and serialize strict real `YYYY-MM-DD` values, years 0001–9999. Use the viewing device's local calendar for overdue, recalculated at local midnight and on focus. Completed/undated tasks never count as overdue. Devices in different timezones may show different overdue labels.

Due Date ordering always places undated tasks last. Within each date, including the undated group, order priority High → Medium → Low, creation timestamp ascending, UUID ascending. Other sorts reverse only their primary key: priority rank, creation timestamp, or NFC-normalized casefolded title in Unicode code-point order. Primary ties use creation timestamp ascending then UUID ascending; Date Created ties use UUID alone. Display original title spelling.

**Rationale**: This completes delegated timezone/tie behavior without inventing a due time or user timezone preference. Domain sorting runs in Python independently of database collation. [PostgreSQL date/time](https://www.postgresql.org/docs/17/datatype-datetime.html), [Python normalization](https://docs.python.org/3.13/library/unicodedata.html), [Python casefold](https://docs.python.org/3.13/library/stdtypes.html#str.casefold).

**Alternatives considered**: UTC instants shift date-only values; reversing an entire list violates undated-last and fixed secondary rules; locale-sensitive title ordering varies between environments. This release chooses explicit locale-neutral order.

## 7. Preferences

**Decision**: Persist status and priority filter preferences as an authenticated account-owned server record. On first use, the authoritative preference is All/All at version 1. The task shell retrieves it before the first task snapshot. Each user selection updates the visible filter immediately; the client sends at most one preference PUT at a time and retains only the newest later selection as queued work. A successful response confirms restoration across sessions/devices only when it acknowledges the current selection; an acknowledgement for an older selection advances the confirmed version and then sends the queued newest selection.

Concurrent device updates use optimistic preference versions. A 409 is a confirmed rejection of the
submitted version, not evidence of which device caused it. A transport/5xx failure is an unknown
save outcome, not proof that the preference was not saved. For either result, pause queued PUTs and
re-read the authoritative pair/version. If the re-read pair equals the current visible selection,
mark it saved; if it differs, retain the visible selection as unconfirmed and require explicit user
approval before submitting a new PUT with the re-read version. Reconciliation reads and PUT replies
carry client request generations; an outdated reply or lower-version re-read cannot replace a newer
confirmed state or change the current visible selection. Reset sort to Due Date earliest-first on
each fresh page session.

**Rationale**: Scope requires filter restoration without a browser-storage exception. Account-owned preferences restore on any authenticated device while optimistic versions, serial local writes, and authoritative reconciliation make conflicts and lost acknowledgements observable without a blind overwrite. No task contents, credentials, or preferences are stored in browser localStorage.

**Alternatives considered**: Browser localStorage cannot satisfy restoration when storage is unavailable and cannot provide authoritative cross-device behavior. Last-write-wins would silently erase another device's selection. Persisted sort is allowed but unnecessary.

## 8. Security, schema and testing

**Decision**: Keep CSRF middleware, escaped templates, safe DOM text rendering, same-origin credentials and no permissive CORS. Production requires HTTPS, secure/HttpOnly session cookies, valid hosts/secrets, `DEBUG=False`, no-store authenticated responses and proxy auth throttling. On history restoration, hide task views until session revalidation. Use indexed owner foreign keys, real date/timestamp types, checked enums/versions and short transactions with consistent lock order. Use least-privilege database access and backups.

**Rationale**: Account isolation must withstand crafted requests, not merely hidden controls. The Postgres best-practices skill informed indexes, types and transaction boundaries. Browser tests prove interactions; real-PostgreSQL tests prove ownership/concurrency; pure tests prove business rules. [CSRF](https://docs.djangoproject.com/en/5.2/ref/csrf/), [Deployment checklist](https://docs.djangoproject.com/en/5.2/howto/deployment/checklist/), [Playwright browsers](https://playwright.dev/python/docs/browsers).

**Alternatives considered**: Browser-only tests miss transaction races. SQLite cannot prove PostgreSQL locking. Additional numeric success thresholds would contradict the approved Scope.

## Resolution

The R2 decisions FD-001 through FD-004 are incorporated in this functional design. The accepted browser-interruption and no-password-recovery limitations remain product risks rather than implementation claims. Hostname, secrets and support contact are deployment inputs. No Scope amendment, application implementation or infrastructure provisioning occurred. Constitution gates pass before and after design; see [plan.md](plan.md).
