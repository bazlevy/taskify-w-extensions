# R3-Rev1 — Targeted Correction Plan

> **Addendum A routing update — 2026-10-23:** The TD-002 blocking-startup recommendation and
> account-preference preservation assumptions below are superseded as proposals by the
> [R1 session-view amendment](r1-session-view-amendment-proposal.md) and
> [R2 session-view amendment](r2-session-view-amendment-proposal.md). Original plan text is retained
> for audit. Follow R1 review/approval, then R2 review/approval before controlled incorporation and
> R3 Rev2; the earlier R2-only “No Scope amendment” route no longer applies. Preserve historical
> FD-002/005 verification, while reviewing deliberate changes to their preference guarantees.
> TD-001 and TD-003–007 remain open under this plan. No approval or finding closure is implied.

**Prepared:** 2026-10-23. **Status:** PROPOSED — human review pending; TD-001–TD-007 remain OPEN.
This document plans corrections and evidence; it does not enact changes to approved baselines.

## Controlled baseline and review boundary

- [R3-Rev1 report](../../../independent-reviews/3_design-technical/Review_1/taskify_R3-Rev1_technical_design_review.md): NOT READY; human R3 gate OPEN.
- [Approved Scope](../spec.md): SHA-256
  `4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`, verified locally.
- [R2-Rev3 approval](../../../independent-reviews/2_design-functional/Review_3%20%28final%29/taskify_R2-Rev3_functional_design_review_APPROVED.md): APPROVED 2026-10-23;
  approved package `specs(2).zip`, SHA-256
  `e75582cd7f849b918ce8e8de559449144936d0ea44cba518d4bdf7188b206f4f`.
- Working design reviewed: `plan.md`, `research.md`, `data-model.md`, HTTP/UI contracts,
  `quickstart.md`, and both checklists. Existing artifacts and review records are preserved.
  A repackaged archive is not proof of the approved R2 ZIP's identity. Obtain the original approved
  ZIP or an authoritative per-file manifest before claiming full cryptographic equivalence.

FD-001 task-write lock, FD-002 account preferences, FD-005 reconciliation, task versions/receipts,
ownership, and the FD-003/004 accepted risks must survive every later amendment. No task limits,
product SLA, offline queue, password recovery, reset, or in-app restore are introduced.

## Finding-by-finding correction proposals

### TD-001 — Conceal data when tab session authority is uncertain

**Owner:** Technical Design lead with UI/security reviewer. **Gate:** R3 blocker.
**Affected artifacts:** `research.md` §§3/5/8, UI contract Synchronisation and privacy, HTTP auth
contract, data-model browser states, quickstart multi-tab/session scenarios, `plan.md` constraints.

Propose a same-origin cross-tab invalidation signal containing only an event type and random
generation, never task data, credentials or account contents. Use BroadcastChannel where available;
correctness must not rely on delivery. Conceal account content synchronously on hiding/blur and
before rendering on visibility/focus/history return; disable writes and revalidate the session and
account before revealing data. Logout/account-change signals immediately invalidate request
generations and clear rendered task content, drafts, pending task payloads and preference queues.
Failed or wrong-account validation keeps content concealed and follows sign-in/retry guidance.
Ordinary same-session background refresh may retain stale rows only while session authority is
still valid; it cannot override concealment. Already submitted writes may still commit, as FD-003
allows. No durable pending journal is added.

**Decision/verification:** Technical/UI-security review must approve mechanism and browser fallback;
route material changes to approved visible behaviour through R2 change control. Scenario V-TD001:
A in two tabs, hide tab 2, log out in tab 1 and sign in as B; delay validation, drop the cross-tab
signal, restore from history, and return tab 2 to focus. Assert no A content is revealed, old replies
are ignored, invalidated drafts/payloads are cleared, and a network failure keeps content concealed.
R3 exit requires coherent contract/state changes and static replay; browser evidence follows Build.

### TD-002 — Define failed initial preference loading

**Owner:** Functional Design owner/product owner → Technical Design lead.
**Gate:** R2 amendment approval and independent review before R3 closure.
**Affected artifacts:** research preferences, HTTP preference GET, UI startup/error workflow,
data-model browser states and quickstart. No Scope amendment.

See [the separate unapproved R2 proposal](r2-td-002-proposal.md). Recommend error/Retry/support with
task bootstrap and writes held until saved preferences validate, followed by the ordinary task-load
gate. Scenarios P-TD002-A1–A6 cover outage, malformed data, retry, task-load failure, account switch,
and first-use/later-save separation. Human approval of the access trade-off is required; do not
silently incorporate this choice into the approved contracts.

### TD-003 — Bound the full-snapshot capacity argument

**Owner:** Technical Design lead plus performance/operations owner.
**Gate:** Credible operating envelope and validation/protection design before R3 closure.
**Affected artifacts:** plan scale/resources, research snapshots, data-model receipt growth,
HTTP request/backpressure rules, quickstart load-validation plan.

Retain full snapshots and lifetime receipts initially. Propose a measured workload matrix, explicitly
labelled engineering assumptions rather than supported product caps: 100/1,000/10,000 tasks per
account; 1/4 visible tabs; 1/10/100 simultaneously active accounts; 1/10/100 KiB descriptions;
and 10,000/100,000/1,000,000 retained receipts. Include concentrated writes and polling together,
not just isolated reads. Model polling as visible tabs divided by the configured interval and
receipt growth as committed operations/day multiplied by retention days and indexed receipt size.
Choose a launch envelope from actual host budgets and results, not these illustrative values alone.

Specify aligned ingress/application body-byte protections with explicit 413 and no truncation,
bounded workers/connections, request/lock timeouts, coalesced polls with jitter/backoff, and safe
429/503 recovery. Timeouts retain unknown-write semantics and FD-001 locks. Measure lock hold/wait,
query/serialization time, response bytes, worker/DB/client memory, receipt/index growth, and aggregate
read/write rates. Record chosen resource budgets and margins before launch. Crossing a documented
budget or repeatedly failing snapshots under the envelope triggers technical review of conditional
reads/pagination; any observable contract change needs appropriate R2 review. No receipt expiry.

**Decision/verification:** Technical/operations reviewers must accept the envelope, resource budgets
and protection values; product owner approval is needed only for a proposed product limit/SLA.
V-TD003 exercises the matrix and an oversize request under concurrent polling/writes, records
resource evidence and tests backpressure without lost acknowledgements or silent data replacement.
If R3 must assess measured feasibility before code exists, use a disposable bounded prototype and
record its limits under separate authority; this task authorises no implementation or measurement run.

### TD-004 — Protect retained receipt digests against offline guessing

**Owner:** Technical Design lead and security/privacy reviewer. **Gate:** R3 security decision.
**Affected artifacts:** research receipt rationale, data-model receipt digest fields/claims,
HTTP replay canonicalization, plan secrets and quickstart replay/security scenarios.

Propose HMAC-SHA-256 over a versioned canonical command including owner context, operation kind,
target UUID and expected version, using a dedicated secret held separately from the database and
session-signing secret. Store digest scheme and key ID with each receipt. New receipts use the
active key; replay uses the receipt's original key and canonicalization version. Keep prior keys
available as long as their lifetime receipts need replay; never silently re-key from unavailable
plaintext or expire receipts to rotate. Missing keys cause a controlled unavailable outcome with
support diagnostics, not a false key-reuse rejection or fresh mutation.

Qualify data-minimisation wording: no plaintext task payload is retained, but this is not absolute
erasure; compromise of both digest keys and receipt data can enable guessing. Protect backup keys
and restrict key access. Do not describe deleted content as cryptographically irrecoverable.
**Decision/verification:** Human security/privacy approval of key custody, long-term replay and
residual exposure is required before adoption. V-TD004 checks deterministic replay and mismatched
payload rejection across rotation, key separation, missing-key fail-safe behaviour, and absence of
plaintext in receipts/logs; do not claim a test proves absolute secrecy. Coordinate with TD-007.

### TD-005 — Make authentication throttling enforceable

**Owner:** Technical Design lead with deployment/security owner. **Gate:** R3 mechanism approval.
**Affected artifacts:** research authentication/security, HTTP auth/429 behaviour, plan topology,
data-model operational limiter state (if adopted), quickstart abuse scenarios.

Propose one Django enforcement service before registration/password authentication, using atomic
shared PostgreSQL counters across workers/instances; retain the existing application/database
topology. Apply separate login source-IP and normalized-username budgets, plus registration source-IP
and service-wide budgets. Never key limits only on a user-controlled forwarding header. The trusted
ingress strips client-supplied forwarding headers and writes a validated client address; application
ingress accepts only that trusted proxy chain, with direct public application access blocked.
Store minimal short-lived limiter identifiers, with keyed username identifiers and a cleanup policy;
never passwords. Account for IPv6 normalization and shared-NAT false positives in tuning.

On exhaustion return 429 with Retry-After and a generic retry message, without account-existence
leaks or permanent account locking. If limiter storage fails, reject new auth/registration attempts
with 503 and retry guidance rather than silently bypassing limits; existing authenticated task
access remains subject to its ordinary database/session availability. Numeric budgets/windows are
deployment configuration to be accepted and tested at R5. Keep Django validation, CSRF and generic
credential failures. Replace vague proxy-only wording consistently if this mechanism is approved.

**Decision/verification:** Security/deployment reviewers must approve enforcement placement, trusted
network boundary and unavailable-limiter policy; R2 reviews any material new auth failure UX.
V-TD005 replays forged forwarding headers, IPv4/IPv6 sources, concurrent requests across workers,
repeated usernames from different sources, registration bursts and limiter outage. Assert shared
limits, correct 429/503, no direct bypass, and no account enumeration. No live abuse test is run here.

## Downstream risk register

| Finding | Accountable owner / timing | Exit condition and required evidence |
|---|---|---|
| TD-006 — Recovery/diagnostics | Operations lead; Technical Design lead owns handoff. Assign a named individual before R5 readiness review; OPEN R5 dependency | Approved runbook traces nonsensitive request IDs through ingress/app/DB diagnostics; redacted logs and error/health metrics demonstrated. Specify backup scope for users/tasks/revisions/preferences/receipts, schema and digest keys; session handling on restore must prevent resurrecting revoked access. Record retention/access policies and run a disposable restore drill checking receipt/revision consistency, replay, ownership and supported schema. Document migration preflight, failed-migration handling and rollback/forward recovery; test it. Any RPO/RTO remains a product decision if required, not an invented guarantee. No self-service restore. |
| TD-007 — Replay across upgrades | Technical Design/maintenance lead; R4 integrated design and Build contract ownership, then every compatible upgrade review. Assign a named maintainer before implementation handoff; OPEN non-blocking risk | Define stable wire/canonical encoding, digest scheme version and key ID; retain old canonicalizers/keys for lifetime receipts. Add cross-version regression fixtures: an old committed command replays after deployment without new effect, while altered content still conflicts; include create→remote delete→old replay without resurrection, default-equivalent inputs, and upgrade/rollback compatibility. R4 accepts compatibility plan; Test executes fixtures before relevant releases. |

Roles above are accountable assignments for the proposed handoff, not evidence that individuals have
accepted them. Named assignees and policy values remain owner actions at the stated gates. TD-006/007
are not claimed closed by documenting their exit conditions.

## Review sequence and human decisions

1. Human review of this plan and TD-002 proposal. Obtain the narrow R2 behavioural approval before
   amending bootstrap contracts; preserve the original R2 approval record and append new authority.
2. Following authorisation, incorporate only accepted corrections into affected design artifacts,
   with TD IDs and scenario references; verify Scope hash, baseline diffs and inherited guarantees.
3. Present a complete revised package and change manifest for independent R2 amendment review and
   R3 re-review. Gate evidence must include the capacity argument, digest/privacy decision,
   authentication boundary and TD-006/007 owner handoffs; no author self-verification disposition.
4. Obtain explicit human R3 approval after independent assessment. This plan grants no authority
   for Tasks, application Build, deployment or progression to the next phase.

**Current decisions pending:** TD-002 observable bootstrap policy; technical/UI-security selection
for TD-001; launch engineering assumptions/resources for TD-003; digest secret/rotation/privacy
decision for TD-004; throttle enforcement/trust/failure policy for TD-005. Existing FD-003 and
FD-004 risk acceptances are retained without reopening them. No finding is marked resolved.
