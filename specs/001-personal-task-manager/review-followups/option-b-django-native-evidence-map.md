# Option B — targeted evidence/change map

**Date:** 2026-10-24. **Status:** Proposal only; R3 NOT_READY, TD-008 OPEN.
Preferred candidate does not mean approved architecture. P1/IndexedDB remains paused.
The current user instruction authorises this Django-native design follow-up; it does not resume
P1, adopt a risk, or authorise implementation/Tasks/Analyze/Build.

**Review target:** [ADR-AS-002 / concrete Option B](option-b-django-native-design-proposal.md),
developing [ADR-AS-001](adr-auth-simplicity-assessment.md) and its original artifact map.
Scope v1.1 and R2-Rev5 §7/R2-NAV-001 are unchanged. Earlier assessments/reviews remain history.

## Precise proposal delta and rationale

| Change | Requirement / reason | Proposal / evidence |
|---|---|---|
| Remove AuthorityFamily/Handle/Transition, login phases and custom maintenance endpoints | These are not necessary for ordinary FR-010 sign-in and private task ownership; removing them is a proposal, not transfer of their old safety proof | §§1/5/6; no new auth table, alternate credential or storage dependency. S2/S4 still block full sufficiency |
| Keep stock PostgreSQL SessionStore, login/flush and fixation protection | FR-010/015/016 private accounts, existing browser coverage/self-registration and independent devices; no duplicate custom SessionStore | §§1/2; preserve registration/support/no-recovery disclosures and limiter; inspect pinned Django source and regress, do not assume framework orders browser headers |
| Thin GET 302 / authenticated POST 303 fixed redirect adapter | Approved R2-NAV-001 is more specific than stock LoginView's default redirect | §2 / B-01; stale forms never replace observed live login; genuine CSRF 403 stays intact |
| One public epoch in Django Session + existing view keyed by that epoch/owner | FR-010 stale-intent guard; FR-012/AC-005 continuity through key changes. Neither header nor view row grants authentication | §3 / B-05/07; remove physical-key cascade dependency, derive view identity from verified Session, conservative liveness-based cleanup |
| Session-row guard before account/receipt/task or view locks | FR-008/010/014: private work uses compatible FOR SHARE; logout FOR UPDATE fences it. FR-012: a view-only stall cannot block tasks on an exclusive auth lock | §3 / B-06/07/11; small parameterized PostgreSQL guard, no lock upgrades; inner account→receipt→task order/budgets retained |
| Narrow empty-session cookie-deletion suppression | S3 old delete response cannot erase newer cookie; revoke server Session instead. No provisional cookie machinery | §4 / B-04/11; verify retained-cookie hygiene, Vary/no-store/CSRF and deletion replay; does NOT solve late-login S2 |
| Existing CSRF recovery and UI generations | FR-009/014/AC-008: Retry/support and saved-data protection; login rotates token, stale pages need bounded safe recovery, epoch/response guards and concealment before revisit; retain FD-001/003/004 | §6 / B-08/09/12; no password/task-command auto-replay, no false logout ACK, no view-error warning |
| Default no app maintenance/live fallback rekey | No periodic key-roll requirement; use standard defaults while retaining login key rotation. View identity survives native rekey, delivery/logout extension is separately gated | §5 / B-07/10; no operational key change authorised, no silent weakening of fixation protection |

## S1–S5 and residuals

| Schedule | Current proposal result | Required review treatment |
|---|---|---|
| S1 / anonymous race | Natural response-order selection specified, no exact winner invented. Correct API owner/epoch checks protect commands; final selection may be another fresh login | Engineering choice, not an automatic D1 human gate. Its S2/S4 effects are not excused by this definition |
| S2 / late response after logout | Native flush revokes the identified Session but cannot find unseen earlier anonymous Sessions. Same-account and different-account counterexamples remain | OPEN technical blocker under unchanged approved logout; B-03 is a reproducer, not a passing guarantee |
| S3 / delayed deletion | Explicit response policy removes the deletion header; newer cookie/session remains usable | Proposed control with exact B-04 assertions, not runtime verification |
| S4 / already-focused private DOM | Epoch/owner checks stop wrong-account writes; existing concealment protects normal revisit, not every later cookie change with lost notifications/reads | OPEN privacy proof gap. B-09 records failure if observed; no tolerated duration |
| S5 / framework rekey | View key is stable across standard cycle_key; default profile does not enable live fallback rekey. Full late-key/logout behaviour remains unproved | Pre-enablement technical gate B-10; if required by deployment, proposal incomplete for it. No operator/global logout policy silently adopted |

Low ordinary likelihood is qualitative and unmeasured, not risk acceptance. Shared-device
post-logout access and old-account private display can have significant confidentiality impact.
No unaffected TD resolution is rewritten; removing TD-001's previous proof needs independent
regression assessment before adoption. Existing TD-006/007 owners/exit evidence remains due.

## Actual R2/human departures — none adopted/requested

D1 needs no approval just to specify an otherwise unspecified initial-login winner.
D2/S2 and D3/S4 are **technical shortfalls**, not an instruction that Barry must redefine logout
or privacy to accommodate the design. The current proposal requests no such weakening.

Only an explicit future variant that allows post-logout authentication by old requests, accepts
a stale-private-display interval, changes valid-login view retention, or replaces the approved
ordinary task-error stale-row rule would need targeted R2/human consideration. Do not bundle
those changes as a precondition of this review. The reviewer can reject the candidate as
incomplete without a product decision; P1 does not resume automatically.

## Affected artifacts — prospective only

| Artifact | If this candidate is later authorised |
|---|---|
| technical-design.md / auth-transition-protocol.md | Replace only affected auth sections with Option B; preserve prior proof/history and §§3–8, independently recheck changed assumptions |
| contracts/http.md / contracts/ui.md | Native page/session/token routes, exact redirect/epoch/CSRF/response policy; remove custom claims/phases/rotation endpoints; keep task payloads, receipts, view semantics and safeguards |
| data-model.md | Drop custom authority/transition entities; key SessionViewState by verified login epoch/owner. Safe migration/expiry cleanup must be reviewed if deployed data ever exists; no silent view reset assumed |
| plan.md / research.md | Record standard Django profile and each justified thin adapter; measured Session-lock/cleanup costs within existing budgets. No actual Plan rerun in this assessment |
| quickstart.md | Add B-01–12 schedules with S2/S4 failures explicitly blocking; retain NAV/AM001/FD/owner/task/HMAC/throttle/optional-view/restore/upgrade regressions |
| Scope, approved checklists and all prior reports/maps/P1 | No edits; approval and risk history untouched. P1 remains paused |

Only this map and the new Option B proposal are added. Hash preservation, local links, scenario
identities and whitespace are checked; no runtime application/browser/DB/restore/load validation.
New proposal identities are listed in the companion [SHA-256 manifest](option-b-proposal.sha256),
which omits itself. This is a reviewable incomplete candidate, not an adopted technical baseline.

Static preservation check: all 58 pre-existing files under specs/ and independent-reviews/ match
their start-of-follow-up hashes. Scope SHA-256 remains
`b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`;
R2-Rev5 approval-record SHA-256 remains
`f08218d808638187bb5227a386e5b8c91913e481308a62a2f62b9016559eca90`.
B-01–12 are complete and unique; local document links and whitespace pass static checks.
