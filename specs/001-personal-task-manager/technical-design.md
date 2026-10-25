# Integrated technical design — R3-Rev10 candidate

**Date:** 2026-10-25. **Status:** R3 NOT_READY / NOT APPROVED; TD-008/010/011 OPEN.
**Approved functional inputs:** Scope v1.2 / AM-002 and [R2-Rev8 approval §4](../../independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md).
FD-009 is RESOLVED_VERIFIED per independent R2-Rev8 §2; R1/R2 approval history is unchanged.
R2-Rev7 NOT_READY remains historical. P1/IndexedDB stays paused. This author candidate selects ADR-AS-004 for independent review;
no architecture approval or runtime proof is claimed. Sections 3–8 retain unrelated mechanisms/owners/exits verbatim.

## 0. Governing R2-Rev8 contract and targeted R3 reassessment

CA-007 allows conventional genuine anonymous session-cookie response selection, not a first
browser winner or mandatory competitor conflict. Authenticated requests/stale forms retain
R2-NAV-001; intentional switching remains logout-first. New actual login defaults its own view.

CA-006 retains protected server authentication/ownership/intended-login guards; reject known stale
replies, clear on observed auth change, revalidate lifecycle/uncertainty boundaries. No atomic
validation/render or required independent trip per cached interaction/normal response.
Approved AC-010/RA-FD006-001 permits prior-authorised display/in-flight completion during an
unobserved same-profile change, with no maximum delay. No cross-account API access/mutation,
old-intent rebinding, known-mismatch rendering or post-observation private retention.

CA-008 retains confirmed intended shared logout, prompt signal/receiving-tab clear/login navigation
and safe uncertainty/suspension/loss handling. No guaranteed delivery, tab ACK or atomic clear.
Retain FD-001, task/receipt/version integrity, CSRF/fixation, independent devices and view isolation.

R3-Rev9 and ADR-AS-003 assessed stronger now-superseded guarantees. Do not recreate them as
technical blockers solely because their old counterexamples remain possible within the approved
residual. Requalify each open TD under the review-remediation policy against retained obligations.
Do not close findings automatically; specify a coherent proportionate mechanism, then obtain
independent R3 review and human approval. [Current handoff](review-followups/r3-rev10-handoff.md).

## 1. ADR-AS-004 / TD-008 — native lifecycle and logout ordering

**R3-Rev10 candidate, not approved.** [Integrated lifecycle](auth-transition-protocol.md) §§1–5
is the technical authentication contract. Use Django PostgreSQL SessionStore as sole authority,
native login fixation/CSRF and logout flush. Remove custom AuthorityFamily/Handle/AuthTransition,
phase endpoints and periodic maintenance rotation from the selected candidate. Historical
protocol is [archived](review-followups/r3-pre-rev10-auth-protocol-snapshot.md); independent
reviews and approved functional contracts remain unchanged.

Genuine anonymous logins may create independent valid sessions and actual cookie response order
selects authentication. Authenticated GET/POST redirects remain unchanged; normal navigation
never cycles key/epoch/view. Same-account new login has a new epoch/default view. No first-winner
or browser-capability requirement.

A short Session-row guard precedes existing task-domain locks. Re-read native authentication and
intended epoch under that lock; retain through command or logout commit. Command-first commits
before revocation; logout-first removes authority before a command effect. All writes/replays
remain owner-scoped/versioned/receipted. Existing account-state → receipt → task order is retained
inside this outer auth guard; no inverse acquisitions. Independent devices remain independent.

The cookie-response adapter emits the auth cookie only for a committed fresh login, avoiding
response-time Session re-save and delayed logout/401 cookie deletion that could erase a newer
login. Native cookie security/encoding remain. Invalid old cookie retention grants no authority.
This targeted adapter is justified by retained continuity/revocation ordering, not winner election.

A single LogoutReceipt per confirmed epoch stores token hash/time/cleanup deadline independently
of the deleted Session. It supports truthful read-only ACK recovery after a lost reply, even if
a newer cookie is now selected. Token cannot authorise new logout without matching live cookie,
intended epoch and CSRF. Missing authority alone never produces an ACK. See protocol §5 for
concurrent logout, retention, memory-only interruption and R5 restore handling.

No maintenance rekey is needed. Valid-login view/key/expiry stays unchanged. Any future live
framework-key rekey/upgrade must be separately planned with preserved view/epoch/expiry and cookie
ordering under TD-007; no unreviewed auth-hash fallback rekey is enabled by this design.

## 2. TD-010 / TD-011 — local stale-state protection and notification

Protocol §§6–7 specify no-store authority discovery, lifecycle/uncertainty concealment and local
generation/identity/query/revision checks. All protected APIs remain live-auth/owner/intended-login
guarded. Known stale replies are discarded and observed change clears private state. No extra
probe per ordinary response/cached interaction and no atomic validation-to-render guarantee.

After confirmed intended revocation, promptly dispatch an epoch-scoped BroadcastChannel event.
Receiving runnable same-login tabs clear/navigate immediately in their handler. No delivery
guarantee, receipt-token broadcast, tab-ACK wait, faster polling, Web Locks or IndexedDB.
Unsupported/lost/delayed messaging falls back to existing polling and concealed lifecycle recovery.
Later-login messages/intents cannot revoke or clear newer identity. Logout uncertainty is not ACK.

Approved AC-010/RA-FD006-001 remains the sole unobserved prior-authorised display residual. Ordinary
task-refresh failure retains stale rows and FD-001 locks; auth uncertainty conceals; optional-view
failure stays silent. Unchanged FD-003/004 limits and all task integrity rules remain.

Independent assessment uses [R3-Rev10 evidence](review-followups/r3-rev10-evidence.md) and
[deterministic planned coverage](review-followups/r3-rev10-validation.md). No scenario has run;
TD-008/010/011 remain OPEN and R3 NOT_READY pending independent review and human decision.

## 3. TD-009 — Bounded optional view read

**Decision ADR-T03:** Authenticate through the normal mandatory session path first. The optional
SessionViewState lookup uses an independent read-only connection/transaction, never the task
snapshot transaction or an outer ATOMIC_REQUESTS transaction (ATOMIC_REQUESTS stays false).
This bounds only convenience work, not authentication or the primary task-data dependency.

Engineering defaults, configurable and not product SLAs:

| Boundary | Candidate value / containment |
|---|---|
| Optional-reader admission | At most 2 reader jobs per Gunicorn worker; no waiting work queue; otherwise immediate defaults |
| Dedicated warmed optional pool | At most 2 connections per worker, acquire at most 25 ms; no cold connect on request path |
| View SELECT transaction | SET LOCAL lock_timeout 40 ms; statement_timeout 80 ms; transaction_timeout 120 ms; validate full tuple/schema |
| End-to-end optional wait | 150 ms including pool acquisition/query result; one attempt only, no inline retry |
| Timeout/transport stall | Request chooses complete defaults at deadline and proceeds with independent task connection; cancel/quarantine optional connection/job; never return an uncertain connection to pool |

Use a bounded executor and nonblocking admission, not unbounded threads or a second web service.
If cancel/rollback fails, the isolated job may occupy its slot until connection teardown; all slots
occupied means subsequent requests skip the optional read immediately. A circuit breaker skips
reads for 30 seconds after 3 consecutive failures, with one probe and redacted diagnostics. On
normal SQL error the job exits/rolls back its transaction before reuse; on deadline the request does
not wait for cleanup. Set timeouts transaction-locally so pooled primary connections are unaffected.
No SQL network read is assumed bounded merely because server statement_timeout exists.

Only a full result arriving within its generation/deadline may select the bootstrap view. Ignore
late results entirely; do not overwrite controls/defaults after the task response. Authentication
failure never becomes defaults with private data. Task snapshot failure still returns its ordinary
error and FD-001 lock. Skip reads/corrupt schema silently; no repair write, view warning or false
empty task response. An entire PostgreSQL failure need not leave task reads available.

**Verification:** R3-V09 holds an ACCESS EXCLUSIVE relation lock on SessionViewState from another
connection while task/auth relations remain usable. Assert the optional deadline/default tuple and
successful independent task snapshot; test corruption of each component, blocked/cold pool, hung
transport/cancel, all reader slots occupied, breaker probe, late-result discard and unchanged primary
connection timeout settings. Repeat with failed task read and failed auth to assert their errors.

## 4. TD-003 — Capacity hypothesis, resource boundaries and migration triggers

**Decision ADR-T04:** Retain one Django service and PostgreSQL; no speculative pagination/service.
The following is a testable launch engineering hypothesis, NOT a task-count cap, user entitlement,
performance/availability SLA, or evidence of measured capacity. R3 assesses plausibility; R5 must
accept actual measured capacity before launch. Failure requires redesign/review, not silent limits.

| Workload dimension | Nominal hypothesis / stress variations |
|---|---|
| Registered / concurrently active accounts | 100 / 50; stress 200 active |
| Visible tabs | 2 per active account, 100 total; stress 4 per account |
| Live tasks per active account | 1,000; stress 10,000 and one concentrated large account |
| Text payload | Average 1 KiB description, 128-byte title; mixed 100 KiB descriptions, up to request boundary |
| Polling | Existing 15-second visible cadence, coalesced/focus/after-write; 100/15 ≈ 6.7 poll GET/s before bursts |
| Writes / view saves | 2 task writes/s sustained, 10/s burst; 2 view saves/s, rapid per-tab coalescing |
| Durable receipt history | 100 committed commands/account/day; 3.65 million/year at 100 accounts; 1/4/10 million receipt fixtures |

Initial resource configuration to validate: application 2 vCPU/4 GiB, four synchronous Gunicorn
workers, ≤512 MiB observed RSS/worker budget; PostgreSQL 2 vCPU/4 GiB with 40 GiB initial storage.
Normal application DB connections ≤8, optional readers ≤8 total, limiter connections ≤4, diagnostics/
migrations ≤8, reserve 12: max_connections 40. Use bounded pools and 25 ms optional acquisition;
mandatory pool acquisition ≤500 ms. No scaling replicas/ingresses may multiply pools without
recomputing these budgets. This is a proposed deployment profile, not provisioned infrastructure.

Enforce 2 MiB ingress AND Django request-body limits, including chunked bodies and login forms;
return 413 without truncating task fields. This is byte abuse protection, not a product title or
task-count limit. Configure ingress active upstream requests 4 plus a queue of 16 with 1-second
admission deadline; reject excess with 503/Retry-After. Reuse app pools; primary task transactions
use lock_timeout 500 ms, statement_timeout 2 s, transaction_timeout 3 s; endpoint deadline 5 s,
upstream timeout 6 s and worker watchdog 10 s. No request waits indefinitely for a DB lock. Keep
all external work/serialization outside lock transactions. Timeouts/worker interruption after a
mutation submission remain outcome-unknown unless known rollback is established; receipts and
FD-001 govern recovery, with no automatic new operation ID.

Normal successful polling retains its cadence; on failed background reads use jittered backoff
15/30/60 seconds, coalesce focus/manual triggers, no hidden polling and no overlapping task read.
Manual Retry remains available. View write failures remain silent with no automatic old replay.

Measurement acceptance for engineering review (not customer promises): at nominal load observe
app/DB CPU <70%, app aggregate RSS <3 GiB, account-lock wait p95 <50 ms, task snapshot endpoint p95
<250 ms, total request deadline respected, no pool exhaustion, and no unbounded memory growth after
an hour. Record p50/p95/max, lock hold/wait, SQL rows/time, serialization time/bytes, task/view request
rates, errors, worker RSS, mobile browser heap, DB pools, table/index bytes and disk/WAL growth.
For low-end supported mobile test contexts, use a 128 MiB incremental heap review budget and record
layout/input responsiveness; no user-facing performance threshold is invented.

Receipt sizing starts at a conservative 512 bytes/receipt INCLUDING indexes for modelling only:
3.65M × 512 ≈ 1.87 GB/year; measure actual table/index/TOAST overhead and revise. Include live task
text, sessions/views/limiter cleanup, WAL and backups separately. Alert at 70% disk/pool/resource
budgets; sustained exceedance at nominal load or stress causing unusable snapshots triggers a
technical capacity review of conditional reads/pagination, indices or deployment resources BEFORE
raising limits. Do not expire lifetime receipts to save space. Any altered visible pagination or
new product cap needs R2/R1 authority. Nominal profile not meeting budgets blocks release readiness;
no performance risk is accepted by this hypothesis.

**Verification:** R3-V03 is a reproducible load fixture matrix with separate database connections,
clocked poll/focus bursts, long descriptions, concentrated account writes and lifetime receipt
fixtures. Record host/browser/patch versions and raw results; inject 413/queue/pool/lock/timeouts and
confirm bounded resources, 429/503 recovery, atomic task/receipt rollback and no snapshot truncation.
No load measurement has been executed for this correction package.

## 5. TD-004 / TD-007 — Keyed, versioned durable receipt protocol

**Decision ADR-T05:** Store HMAC-SHA-256 rather than unkeyed SHA-256 of a guessable command. Use a
dedicated random 256-bit receipt key, independently provisioned from Django SECRET_KEY/session keys
and database credentials; secrets live in deployment secret storage outside the receipt DB/backup.
Use constant-time digest comparison. Store canonical_version, digest_scheme and key_id per receipt;
never store the key, plaintext task command or a reversible response in the receipt.

Canonical format 1 is UTF-8 JSON of an ordered array (no insignificant whitespace, Unicode emitted
as UTF-8, null literal, no floats). Domain-separate with `taskify.task-command`, schema version 1,
owner public UUID, operation UUID, route-kind, target task UUID, expected-version decimal string or
null, and an ordered details array. Canonical UUIDs are lowercase standard hyphenated strings;
versions have no leading zeroes; date is strict YYYY-MM-DD or null. Title uses the existing pinned
Unicode-trim rule, with no new normalization/case folding. Description follows existing empty/null
normalization and otherwise preserves exact text; priority normalizes to medium, other enums stay
as validated. Create/edit details are [title,description,priority,due_date]; set_status [status];
delete [true]. Reject duplicate JSON keys as malformed input. Keep all format-1 defaults, Unicode
tables/normalizer fixtures and encoding behaviour fixed even if later releases use new rules.

Replay algorithm: authenticate owner → account-state lock → owner/operation receipt lookup → if
present parse/canonicalize using ITS stored canonical version, scheme and key → constant-time
compare. A match returns historical acknowledgement before task existence/version checks; a
mismatch returns operation_key_reused. No receipt: use active supported format/key, normal task
validation/version checks, then atomically commit task effect/list revision/receipt metadata. Wire
syntax/auth/CSRF rejection remains earlier. Missing/unsupported historical key/format returns 503
receipt_verification_unavailable with known no-new-mutation outcome; retain the pending operation
ID and support diagnostics, never recompute against the new key or generate a new operation.

Rotation deploys a new active key ID for new receipts; old keys become verify-only and remain
available for account-lifetime receipts. Never expire receipts or destroy a still-referenced key
as routine rotation. Rollback binaries must understand all formats/keys already committed, or that
rollback is prohibited in favour of a compatible forward repair. Deployment preflight checks keys,
known receipt fixtures and canonicalizers before traffic. No historical key is derivable from its ID.

Privacy wording: physical task deletion and receipts contain no plaintext/reconstructable task
payload for an in-app restore, but metadata/digests are not absolute erasure. HMAC prevents a
receipt-table-only actor from testing text guesses without the separate secret. Combined key/DB
compromise can enable guessing; live process compromise and existing backups are separate threats.
Access controls/key separation and restore handling mitigate rather than silently accept these
risks. Security review must evaluate this threat model; no human risk acceptance is inferred. If
deployment cannot preserve separation or historical keys, escalate before changing this protocol.

**Verification:** R3-V04 covers guessed low-entropy inputs without the key, no plaintext logs/rows,
same-key/mismatched payloads, key rotation, unknown-key failure, and constant-time comparison usage.
R3-V07 stores format-1 fixtures, upgrades to format 2 with changed defaults, replays identical old
wire commands and default-equivalent requests, and repeats create→remote delete→old replay. Verify
historical acknowledgement/no resurrection, changed-command conflict, and compatible rollback or
preflight prohibition. These are planned security/compatibility checks, not proof of perfect erasure.

## 6. TD-005 — Authentication throttle enforcement and trust boundary

**Decision ADR-T06:** Enforce in a Django pre-auth service invoked before credential checking and
account creation by the login/registration adapters. All workers/instances use atomic PostgreSQL
window counters in a separate short transaction; no process-local/independent edge counters as
the only control. A single HTTPS Nginx ingress is the launch topology; it strips incoming Forwarded/
X-Forwarded-* and writes one canonical client address. Application listener is private and accepts
connections only from the configured ingress; application trusts headers ONLY from that peer.
Normalize IPv4/IPv6 consistently; do not accept arbitrary client headers, nested unknown proxies or
raw login usernames as source identity. Additional ingresses require the same shared counters and
an explicit reviewed trusted-proxy chain; direct upstream access is not a permitted deployment.

Initial tuneable security budgets: login 30 attempts/minute per source address, 5/minute per
source+normalized-username pair, 300/minute service-wide; registration 5/hour per source and
100/hour service-wide. These are fixed-window engineering settings, not a promised product quota;
evaluate boundary bursts, NAT/IPv6 fairness and distributed attacks before R5. Counters cover both
existing and nonexistent usernames and successful/failed attempts; no account-existence leak or
permanent user-account lock is added. Choose generic responses/timing and retain password validation,
CSRF and username validation. Cross-source attacks are constrained by the shared service budget;
monitor and retune rather than claiming per-source limits alone defeat distributed abuse.

Atomic dimension acquisition uses sorted counter IDs and bounded 100 ms lock / 200 ms statement
timeouts; increment applicable attempt counters once per admitted attempt, rollback together on DB
error. Bucket identifiers are HMACs with a separate throttle secret, not raw passwords/usernames/IP
logs. Expire counters after window end plus 1 hour; hourly bounded cleanup, no growth without limit.
Use a separately bounded pool of at most 4 total connections so limiter traffic does not exhaust
task/optional pools. Throttle-state writes do not acquire session/view or account task locks.

Budget exhaustion yields generic 429 with Retry-After to the next permitted window; registration/
login forms keep nonsecret fields and show retry guidance. No password is reflected. If shared
limiter enforcement is unavailable or times out, fail those NEW login/registration submissions
closed with generic 503/Retry-After; no partial account/session creation and no silent bypass.
Existing valid task sessions are not explicitly invalidated by limiter outage. The approved API
already supports unavailable/throttled outcomes; this mechanism adds no permanent lockout or new
identity-recovery policy. If implementation needs CAPTCHA, account lockout or new identity rules,
escalate as a material R2/product decision before adoption.

**Verification:** R3-V05 tests login/registration floods, distributed source pairs, shared counters
across workers, window boundaries, forged forwarding headers, trusted/untrusted proxy paths, direct
upstream denial, IPv6 aliases/shared-NAT recovery, limiter failure/timeout, generic nonexistent-user
responses, 429/Retry-After, 503 and unchanged existing-session task access. No live attack run here.

## 7. TD-006 / TD-007 — Downstream ownership and exit contract

**Assignment authority:** On 2026-10-23 the user assigned Barry Levy as accountable human owner for
both TD-006 Operations/R5 and TD-007 Maintenance/R4. Codex prepares technical designs, corrections,
validation plans and handoff documentation. This records assignment only: it is neither risk
acceptance nor technical approval, and independent finding dispositions remain R3-owned.

| Finding | Accountable role / timing | Exit evidence required |
|---|---|---|
| TD-006 | Barry Levy, accountable Operations/R5 owner; Codex prepares design/runbook/validation/handoff | Barry accepts the operational evidence package before R5 approval: backup/retention/access runbook, trace/metrics demonstration, disposable restore/migration drill and recorded results. R3 assesses named ownership and this deliverable plan; execution is later |
| TD-007 | Barry Levy, accountable Maintenance/R4 and future-release owner; Codex prepares compatibility design/fixtures/handoff | Barry confirms R4 compatibility planning/fixture ownership; Build/Test execute R3-V07; each upgrade/rollback release verifies compatibility before deployment and records Barry's gate decision |

TD-006 diagnostics: ingress generates validated opaque request IDs; app propagates them to response,
structured service logs and DB operation spans without recording SQL bind contents, task text,
cookies, passwords, session keys or receipt keys. Support lookup maps ID to route/outcome, duration,
error class and safe trace spans. Use restricted log access and a documented initial 14-day log
retention policy; review data minimisation with the owner. Health/metrics distinguish auth failures,
optional view skips/timeouts, task-read/write/replay errors, pool/lock pressure and backup failure.

Backup proposal for owner sign-off: daily encrypted PostgreSQL consistent backups retained 14 days
in restricted separate storage, schema/migration manifest, receipt-format/key-ID manifest; retained
receipt secrets are separately escrowed with access/restore controls for their reference lifetime.
This cadence/retention is operational configuration, not an approved RPO/RTO/zero-loss guarantee.
Include users/tasks/revisions/receipts and sessions/views/limiter data in consistent DB capture;
on operational restore invalidate all restored auth sessions/views and clear limiter buckets before
serving traffic, avoiding resurrected revoked access. Tasks/receipts/revisions restore together.
No self-service task or credential restore endpoint is added. An owner requesting new durability
promises, old-session reuse or shorter key retention must escalate rather than assume approval.

Drill R3-V06: restricted operator restores a disposable database with matching schema/keys; confirms
owner isolation, saved tasks/revisions, create→delete→receipt replay without resurrection, expired
sessions rejected, new logins default views, and references can diagnose failures without contents.
Document checksum/backup-age/restore-duration observations and known loss window without asserting
an SLA. Migration preflight checks backup/readability, format/key compatibility and lock budgets;
quiesce writes for incompatible changes, rehearse failure. Prefer transactional reversible changes;
unsafe rollback stops traffic for reviewed forward recovery or consistent restore, never partial
task-only restore. Operator records evidence and approval before release readiness.

## 8. Gate and review evidence

R3-Rev2 dispositions are preserved: TD-002 resolved by supersession; TD-001/003/004/005/008/009
submitted for correction assessment; TD-006/007 OPEN with handoff conditions. This author document
does not change independent dispositions or grant R3 approval. Scope/R2 behaviour and FD-003/004
accepted consequences remain fixed. No Tasks/Analyze, code, infrastructure or tests executed.

Primary references: [Django LoginView](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/views.py),
[Django auth](https://github.com/django/django/blob/stable/5.2.x/django/contrib/auth/__init__.py),
[SessionMiddleware](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/middleware.py),
[database backend](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/backends/db.py),
[PostgreSQL timeout controls](https://www.postgresql.org/docs/17/runtime-config-client.html), and
[Python HMAC](https://docs.python.org/3.13/library/hmac.html). Choices and numeric budgets above
are repository design decisions for review, not claims those references prescribe their values.
