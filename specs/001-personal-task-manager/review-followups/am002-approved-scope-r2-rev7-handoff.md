# Approved Scope v1.2 / AM-002 — targeted R2-Rev7 re-review handoff

**Date:** 2026-10-25. **Scope gate:** CLOSED — APPROVED by Barry Levy in R1-Rev5 §9.
**Downstream:** R2-Rev7 NOT_READY / FD-009 OPEN; R3 NOT_READY / TD-008/010/011 OPEN.
No Tasks, Analyze, Build, implementation, runtime tests or downstream approval is authorised.

## Exact authority and identities

The updated [R1-Rev5 report](../../../independent-reviews/1_scope/Review_5/Taskify_R1-Rev5_Targeted_Scope_Review_AC010.md)
is already incorporated at the requested Review_5 path and is retained byte-for-byte. Section 7
records incorporation authorisation; §8 independently verifies the candidate; §9 records the
subsequent final human Scope approval. No review text or historical decision is regenerated.
The supplied Archive.zip contains the earlier report without §9 and is not substituted for this
updated authoritative record.

| Item | SHA-256 |
|---|---|
| Independently verified and human-approved Scope v1.2 candidate, identified in §9 | 87bf1574788c46eedf6412782e1b6a60915a4db0d4b643483d9e79736a2c745c |
| Current canonical [spec.md](../spec.md), after approval metadata only | 688b57c11785af800b072843d7763c14875dbc6cbb5b35594201936cee03549b |
| Updated R1-Rev5 record, including §§8–9 | 992ebc81381920a86d94041c6164f48235722c299f11184a9b3183c3a3716dbf |

The differing specification hashes reflect approval metadata only. The approved product-definition
candidate remains identified by its original hash; no new substantive approval is inferred for
altered requirements. Historical v1.1/R1-Rev4 and R2-Rev5/Rev6 approvals remain attached to their
original baselines.

## Metadata change/evidence map

See the [exact candidate-to-approved-metadata diff](am002-scope-approval-metadata.diff).

- Reconciled/status fields now record independent R1-Rev5 §8 verification and final §9 approval.
- Final approval names Barry, date and the exact approved candidate hash; the operative Scope
  baseline is now v1.2 / AM-002. Section 7 authorisation remains separately recorded.
- The AM-002 history row's approval/evidence cell records the completed gates; its product/risk
  summary is unchanged. Earlier history rows, decisions and approval evidence are preserved.
- Pending-gate explanatory document-control prose is updated; no product-definition text changes.

Author static checks confirm the approved source hash before editing, byte-identical text from
Product Boundary through the end of spec.md, all 16 FRs and 12 ACs unchanged, historical decisions
preserved, and a valid exact unified diff with reverse-apply checking. Review/design artifacts,
policy, constitution and Spec Kit files are unchanged. These are integrity checks, not a fresh
independent R2 review or an execution result.

## Narrow independent R2-Rev7 request

Use [the existing R2-Rev7 report](../../../independent-reviews/2_design-functional/Review_7/Taskify_R2-Rev7_Targeted_Functional_Design_Review_NOT_READY.md)
and the unchanged [conventional-auth proposal](r2-conventional-auth-amendment-proposal.md),
[evidence map](r2-conventional-auth-amendment-evidence.md) and
[CA-T01–16 scenarios](r2-conventional-auth-amendment-scenarios.md).

Recheck FD-009's Scope consistency against now-approved AC-010 and the two Story 4 qualifications.
Confirm CA-006/RA-FD006-001 stays within the approved prior-authorised display/in-flight residual:
no maximum observation delay, but no cross-account API access/mutation, old-intent rebinding,
known-mismatch rendering or private-state retention after an observed change. Preserve FR-010
and task/load/view protections. Recheck only consequential CA-007/008, Q1 and R2-NAV-001 alignment;
do not reopen unrelated approved behaviour or introduce implementation-stage proof requirements.

R1 has resolved the Scope-definition question, not FD-009's independent Functional Design
disposition. The reviewer must give their own targeted verdict and update their finding history
as warranted; Barry's separate final R2 approval is still required. Operative R2/R3 contracts are
unchanged pending those gates. Only subsequently may separately authorised R3 reassessment occur;
this handoff neither selects an architecture nor advances R3/Tasks/Analyze/Build.
