# R3-Rev3 — Author correction and evidence map

**Prepared:** 2026-10-23. **Status:** Submitted for independent R3-Rev3; no technical approval or
author finding closure. Review input is [R3-Rev2](../../../independent-reviews/3_design-technical/Review_2/taskify_R3-Rev2_technical_design_review.md).

## Controlled inputs

Scope 1.1 / AM-001 remains byte-identical at
`b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`.
R2-Rev4 human approval remains the exact independent record/manifest in
[R2-Rev4](../../../independent-reviews/2_design-functional/Review_4/taskify_R2-Rev4_functional_design_review.md).
The source design files matched that manifest before this correction. No independent report,
approval history, Scope rule or old proposal is overwritten. Current technical supplements carry
fresh hashes; R2's historical hash approval is not retroactively transferred to changed files.

The six requested corrections are technical mechanisms within approved behaviour. No new product
cap/SLA, identity recovery, permanent lockout, task restore, offline queue or risk acceptance is
introduced. Security/privacy and feasibility remain subject to independent R3 assessment. If secret
custody, bounded pools, trusted ingress or safe lifecycle finalization cannot be implemented as
specified, stop and escalate; do not relax guarantees or infer acceptance to make the design pass.

## Per-finding correction and verification map

| Finding / inherited disposition | Candidate correction and operative evidence | Planned independent/downstream verification |
|---|---|---|
| TD-001 OPEN, blocker | technical-design §1 / ADR-T01; UI Synchronisation/privacy and authority state; HTTP session/epoch guard; data-model orthogonal concealment state | R3-V01 dropped signal, A→logout→B/same-user re-login, history, late/error revalidation; no private DOM/accessibility reveal before valid current generation |
| TD-002 RESOLVED by R3-Rev2 supersession | Preserved: direct task bootstrap, no mandatory preference GET, full silent defaults. Do not reopen/delete historical finding; TD-009 is the separate bounded-read issue | AM001-V01/05/10 and R3-V09, with FD-001/auth protection retained |
| TD-003 OPEN, blocker | technical-design §4 / ADR-T04; Plan engineering envelope/budgets; HTTP 2 MiB/finite admission/deadlines; research capacity decision | R3-V03 workload/receipt-growth/resource matrix; later measured readiness, no invented product SLA/count cap or silent snapshot truncation |
| TD-004 OPEN, blocker | technical-design §5 / ADR-T05; MutationReceipt HMAC/version/key metadata and qualified deletion privacy; HTTP historical verifier/no-key 503; research security rationale | R3-V04 table-only/combined-key threat review, rotation/replay/missing key, no plaintext/log exposure; security assessment required, no risk acceptance inferred |
| TD-005 OPEN, blocker | technical-design §6 / ADR-T06; shared PostgreSQL AuthThrottleBucket and trusted private ingress; HTTP/UI generic admission errors; Plan/research topology | R3-V05 source spoofing, distributed workers/sources, NAT/IPv6, outage and 429/503; thresholds tune at R5 within technical policy |
| TD-006 OPEN, ownership/exit condition | technical-design §7: Barry Levy named accountable Operations/R5 owner; Codex prepares designs, corrections, runbook/validation/handoff; request tracing, redaction, retention/access, backup/keys/schema, session invalidation and migration recovery | Barry accepts named deliverables/evidence before R5; R3-V06 disposable restore/replay/ownership/migration drill and safe support trace, no new RPO/RTO or user restore |
| TD-007 OPEN, nonblocking with ownership | technical-design §§5/7: Barry Levy named Maintenance/R4/upgrade owner; Codex prepares format/key compatibility plan/fixtures; frozen canonical format and rollback preflight | R4 confirms fixture ownership; R3-V07 old replay/default-equivalent input, create→delete→replay, upgrade/key rotation/compatible rollback before release |
| TD-008 OPEN, blocker | technical-design §2 / ADR-T02; concrete form_valid/post and SessionStore/finalizer protocol, forced same-user flush/new key+epoch, guarded maintenance move; HTTP auth and data-model lifetime | R3-V08 real PostgreSQL barriers plus cookie/middleware ordering, expiry/logout/save races and independent valid sessions |
| TD-009 OPEN, blocker | technical-design §3 / ADR-T03; independent warmed read-only pool/executor, no admission queue, 150 ms optional total deadline, DB limits/quarantine/circuit breaker; HTTP/data-model isolation | R3-V09 relation lock, cold/exhausted pool, hung transport/cancel, corruption/late result, successful primary snapshot and preserved auth/task errors |

## Ownership authority and gate boundaries

User instruction on 2026-10-23 names **Barry Levy** accountable for TD-006 Operations/R5 and TD-007
Maintenance/R4. **Codex** prepares technical designs, corrections, validation plans and handoff
documentation. The user expressly states these assignments are not risk acceptance or technical
approval. They do not claim runbooks, retention configuration or restore tests already passed.

Prior FD-003 interrupted task-identity and FD-004 no-credential-recovery accepted risks remain
unchanged. All task/persistence/calendar/ownership/view tuple behaviours stay within Scope/R2.
R1-AM-004 remains a deferred historical-documentary matter, not a newly accepted technical risk.

Historical pre-sign-off headings in approved AM-001 source documents are retained as history;
current candidate notices and this record identify R2-Rev4's actual approval. No independent status
is rewritten: six blocker findings are submitted for correction assessment, TD-006/007 exit plans
are specified, and only TD-002 carries an already-resolved independent disposition. Barry's R3
decision follows the independent review, not this author submission.

## Changed artifacts and assurance limits

- technical-design.md: new operative ADRs, protocols, budgets and owner/exit contracts.
- plan.md / research.md: concrete mechanism references, unchanged product authority and deployment hypothesis.
- data-model.md: epoch/session guard, bounded view read, keyed/versioned receipt fields and expiring auth counters.
- contracts/http.md / contracts/ui.md: executable-contract detail for authority, cookies, epochs,
  optional read/admission failure, receipt verification and concealed states.
- quickstart.md: planned R3-V01/03/04/05/06/07/08/09 scenarios, without test execution claims.
- this evidence map and r3-rev3-changed-files.sha256: submission inventory and exact file identities.

Scope, independent reports, both reviewer-owned checklists, the R2-Rev4 handoff and earlier
proposals are preserved. Checks performed for this submission are file/hash, link, cross-document
contract and upstream-invariant checks only. No Tasks/Analyze, source implementation, deployment,
load, browser or restore run was performed. R3 remains unapproved; stop for independent R3-Rev3
assessment and explicit human R3 decision before the next authorised phase.

The [hash manifest](r3-rev3-changed-files.sha256) covers all changed/new design documents including
this map, relative to specs/001-personal-task-manager/. It intentionally excludes its own hash to
avoid circular self-reference. Hashes identify this candidate, not independent verification.

**Author static validation:** Scope bytes, approved R2 handoff and both checklists are unchanged;
task fields, TaskListState, task HTTP command shapes, task UI operations and calendar/sort rules
were compared to the input and retained. FD-001 lock and AM-001 silent view behaviour remain
explicit. Every submitted finding has a planned scenario and ownership/exit mapping. Local document
links and diff whitespace were checked. These checks do not establish runtime feasibility or
change any independent finding disposition; R3-Rev3 must inspect the actual corrected files.
