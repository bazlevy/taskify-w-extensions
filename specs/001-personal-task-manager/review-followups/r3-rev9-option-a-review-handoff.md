# R3-Rev9 assessment handoff — Option A preserved

**Date:** 2026-10-24. **Status:** assessment submission, not a complete corrected architecture.
R3 NOT_READY; TD-008/010/011 OPEN. Scope and R2-Rev5/Rev6 remain approved. P1 paused.
No Tasks, Analyze, Build, implementation or runtime testing.

Read [ADR-AS-003](r3-rev9-option-a-architecture-assessment.md) first. Barry's latest Option A
decision excludes mandatory Web Locks, IndexedDB or another capability at this stage and directs
an assessment stop if compliance needs disproportionate expansion. No dependency is adopted.

## Finding and evidence map

| Finding | Assessment / concrete evidence | Disposition and remaining proof |
|---|---|---|
| TD-008 / FD-007, S2 | ADR §2 identifies both cookie-header orders, server-order independence, shared anonymous-bootstrap race and loss of active logout proof. Server locking/409 alone does not prevent a later cookie write. | OPEN. No availability-preserving complete selection mechanism established. S2 is preserved, not used to permit pre-logout replacement. |
| TD-010 / FD-006, Q1 | ADR §3 gives an after-arrival validation reply that becomes stale across legitimate A logout/B login before reveal. Extra probes and local generations do not fence that interval. | OPEN. No new private-reveal exposure is accepted. Server owner/epoch guards still protect mutations; that alone is not DOM proof. |
| TD-011 / FD-008 | ADR §4 specifies channel/message schema, ACK trigger, epoch-scoped clearing/navigation, duplicate/late/lost delivery, suspension and uncertain outcomes. | Concrete proposed notification design; OPEN pending independent review and integration with a proved reveal/action gate. Broadcasts are not revocation authority. |
| Prior resolved TDs / FD risks | No supporting mechanism is removed and no unrelated finding disposition is rewritten. Barry retains TD-006/R5 and TD-007/R4 ownership/exits. | Existing history unchanged. FD-003/004 do not accept any race above. Capacity/throttle/receipt/view-isolation mechanisms are not expanded. |

## Deterministic technical scenarios — planned only

These supplement, not replace or weaken, approved R2-6R-T01–16. Diagnostic counterexamples are
not passing acceptance results. No code or browser fixture was executed during assessment.

| ID / approved trace | Schedule and required evidence |
|---|---|
| A9-01 / T01/02 | Dispatch two anonymous native logins; persist both sessions, deliver A's valid cookie first then B's. Record authentication/account/epoch/view after each header, not just callbacks. Reverse header order and server completion; repeat same account. Native overwrite is a blocker reproducer, not acceptable final selection. |
| A9-02 / T01/02/04 | Add server serialization or a loser 409; hold any competing cookie response already sent. A conflict with a later replacing cookie still fails. If an approach emits only one valid cookie, prove initial shared admission itself under A9-03, rather than presume a shared browser root. |
| A9-03 / T01/02 | Hold anonymous bootstrap X's cookie response; bootstrap Y and establish A; release X. Observe whether A's usable cookie/logout proof is replaced. Include separate fresh tabs, no existing Session and omitted/rejected cookies. No IP/UA fingerprint merges independent browsers/devices. |
| A9-04 / T03/04/16 | Confirm original L revocation, deliver a distinct still-valid late login; L never revives. Then establish C before delayed B and assert C protection. Terminally rejected attempts cannot revive. This permitted S2 branch does not excuse A9-01. |
| A9-05 / T07/08 | Hold A-private payload, let it arrive, then dispatch fresh authority validation. Server validates A; hold that reply. Another tab revokes A and establishes B; drop notices. Release A's identity reply and attempt reveal. Newly exposed A data is a TD-010 failure. Repeat with same-account new epoch. |
| A9-06 / T07–10 | Move legitimate auth change to immediately after the last successful validation and before scheduled DOM/accessibility commit; delay callbacks and requestAnimationFrame. Check generations, query and revision. Matching old values are insufficient if no shared fence observes the change. Failed required authority checks conceal; ordinary task-refresh errors retain stale rows/FD-001 locks. |
| A9-07 / T05/12 | Confirm logout L; observe immediate ACK-handler broadcast and runnable matching tabs' synchronous clear/generation invalidation/login navigation before any poll. Deliver duplicate and late messages after N is active: no N revocation/view reset or old-intent replay; login URL obeys R2-NAV-001. |
| A9-08 / T06/11/15 | Fail channel construction/posting, drop delivery, suspend/restore and use history/bfcache. Initial/resumed/private-reveal gates stay concealed; message absence cannot imply authenticated access. Other devices stay live. Lost ACK/401/403 never emits false confirmed message or success. Gate correctness still requires TD-010 proof. |
| A9-09 / conditional alternative | If ever separately authorised, test missing/denied Web Locks, frozen holder, normal release, abort-before/after headers and document destruction. Force-release/timeout must not permit a late old operation. Optional native fallback must pass A9-01/05/06 independently; API presence alone is not a result. No such dependency is adopted now. |

## Reviewer request and exact trade-offs

Assess the incompatibility analysis and the proportionality stop, not an assertion that every
conceivable no-new-capability design is impossible. A contrary technical solution should provide
one integrated issuance/selection/revocation/reveal proof for the schedules above, with safe
bootstrap, termination and bounded resource behaviour; naming a framework or transport is not proof.

The simplest identified prospective alternative is mandatory shared/exclusive browser fencing
around native Django auth and reveal. Its specific trade-off is fail-closed loss/delay of affected
operations when coordination is unavailable or held by a stalled context. Optional fencing leaves
the unsupported path unresolved. Barry has **not** approved that availability departure; his
latest decision preserves Option A. Do not reopen the question automatically or infer acceptance.

A native-only fallback would instead need explicit changes to first-browser-winner FD-007 and
validation-to-reveal FD-006; privacy/Scope implications require the appropriate R2/R1 gates. No
functional amendment, residual exposure or broader account-wide logout is accepted here.

## Changed artifacts and static preservation evidence

| Artifact | Change |
|---|---|
| technical-design.md | Metadata only: add TD-010/011, current R3-Rev8 input and Option A assessment/stop references. Body from §0 onward unchanged. |
| plan.md | Metadata only: current blocker IDs, Option A and assessment links. Feature/design body unchanged; no Plan rerun. |
| r3-rev9-option-a-architecture-assessment.md | New ADR-AS-003: native baseline, incompatibilities, notification protocol, alternatives and exact trade-offs; no dependency adoption. |
| This handoff | New finding/scenario/evidence map and bounded reviewer request; nine planned diagnostic/regression scenarios, no execution. |
| r3-rev9-option-a.sha256 | New repository-root-relative affected-file and authoritative-input manifest, omitting itself. |

Of 72 pre-existing files under specs/ and independent-reviews/, 70 are byte-identical; only the
two metadata updates above changed. All 29 independent-review files, approved operative UI/HTTP,
Scope, R2 proposals/approval records, previous handoffs and old manifests are preserved.
The technical-design and Plan body comparisons pass; the nine new scenario IDs are unique and
complete. Local links, tables, newlines, whitespace and git diff --check pass static author checks.
Manifest checksums identify this assessment, not an independently verified technical correction.

The revised handoff therefore cannot recommend READY or close TD-008/010/011. Further architecture
expansion stops as directed. Human R3 approval remains separate. See the
[evidence/changed-file manifest](r3-rev9-option-a.sha256); all scenarios remain planned.
