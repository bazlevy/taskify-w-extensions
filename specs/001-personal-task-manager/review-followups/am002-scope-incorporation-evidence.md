# AM-002 Scope incorporation — R1-Rev5 verification handoff

**Date:** 2026-10-25. **Candidate:** canonical Scope v1.2 / AM-002, PENDING FINAL SCOPE APPROVAL.
Barry authorised incorporation under DEC-AM-002-01; this is not final Scope approval or an
independent post-incorporation verification. Scope v1.1 remains the operative approved baseline.
R1-AC010-001 remains pending independent disposition; R2-Rev7 / FD-009 and R3 remain NOT_READY.
R3 TD-008/010/011 remain OPEN. No Tasks, Analyze, Build, implementation or runtime tests.

## Authority and exact identities

[R1-Rev5 report](../../../independent-reviews/1_scope/Review_5/Taskify_R1-Rev5_Targeted_Scope_Review_AC010.md)
§4 supplies the complete AC-010 replacement and minimum conforming edits; §7 records Barry
Levy's 2026-10-25 authorisation, with verification and final human approval still pending.
The report is preserved unchanged; SHA-256
`d84a7f0580af2597641eac9d114527e969e7caf03eac13ad9325c0f964d52bb0`.

| Artifact | SHA-256 / identity |
|---|---|
| Approved Scope v1.1 source, captured before this reconciliation | b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca |
| Updated canonical [spec.md](../spec.md), Scope v1.2 / AM-002 candidate | 87bf1574788c46eedf6412782e1b6a60915a4db0d4b643483d9e79736a2c745c |
| Exact comparison | [Unified diff against the approved v1.1 bytes](am002-scope-v1.1-to-v1.2.diff), not against an assumed Git HEAD version |

## Targeted change / evidence map

| Source → decision → canonical change | Author validation |
|---|---|
| R1-AC010-001 / FD-009 → §4 AC-010 and DEC-AM-002-01 → complete AC-010 replacement | Text exactly matches §4, with only canonical list/ID formatting retained. Covers prior-authorised display/in-flight completion while a same-profile change is unobserved, no maximum delay, clearing on observation, known-mismatch rejection and unchanged cross-account operation prohibition. |
| §4 / §7 → Story 4 independent test | Only the final tasks/settings isolation sentence qualified: no access through the new account's authenticated operations, subject only to AC-010's display residual. |
| §4 / §7 → Story 4 scenario 3 | Only the isolation result qualified: inaccessible through the new account's authenticated requests, with the narrow display case referred to AC-010. |
| §4 amendment register / §7 authorisation → document control and new v1.2 history row | Records authorisation separately from PENDING final approval, names the exact reviewed v1.1 hash, preserves the earlier approvals/history and makes no self-verification or gate-approval claim. |

No other product changes: all 16 FRs, including FR-010, are byte-for-byte unchanged; all other
11 ACs and all requirement/acceptance IDs are unchanged. Reversing only the three authorised
product-text edits reproduces the prior Product Boundary-through-end text exactly. Earlier
version rows and historical clarification/approval decisions are preserved. The old v1.1
approval-metadata statement is put in historical tense so it does not misdescribe AM-002.

## Static validation and next gate

The approved source hash was checked before editing. The exact unified diff was regenerated
from that source and the actual amended file; it matches the supplied diff artifact, and
`git apply --reverse --check` passes without changing files. Author checks also cover exact
authorised AC-010 text, unchanged requirements, Story 4-only product conformance, pending status,
links and Markdown formatting. These are static author results, not independent R1 verification.

Only spec.md and these two new handoff/diff artifacts are authored in the repository. Historical
review reports, R2/R3 operative contracts, prior proposals, policy, constitution and Spec Kit
files are not modified; no Spec Kit command is run.

Return the actual canonical file, diff and hash to independent R1-Rev5 under §7.2. The reviewer
must issue their own post-incorporation verification/readiness recommendation and disposition
R1-AC010-001 as warranted. Then Barry must explicitly approve the exact verified Scope v1.2
version/hash under §7.3 before its final approval metadata is changed. Only after that Scope gate
may the separate targeted R2-Rev7 FD-009 consistency/gate review proceed. No downstream gate is
advanced by this handoff.
