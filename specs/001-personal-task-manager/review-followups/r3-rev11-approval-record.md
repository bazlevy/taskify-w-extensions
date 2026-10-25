# R3-Rev11 human approval record — R3-Rev10 baseline

**Decision ID:** DEC-R3-REV11-20261025  
**Decision maker:** Barry Levy, human product owner  
**Decision date:** 2026-10-25 (Australia/Melbourne); time of day not asserted  
**Independent verdict:** R3-Rev11 READY — report wording: READY FOR HUMAN APPROVAL  
**Human R3 gate:** CLOSED — APPROVED  
**Approved baseline:** R3-Rev10 / ADR-AS-004 native-session technical candidate

## Authority and exact approval boundary

Barry explicitly directed that this approval be recorded in the conversation. The existing
[independent R3-Rev11 report](../../../independent-reviews/3_design-technical/Review_11/Taskify_R3-Rev11_Technical_Design_Review_APPROVED.md)
§§1–7 supplies independent design verification; §8 separately records Barry's subsequent
“approved” declaration and human gate. This record transcribes that decision; it does not create
a new independent review, amend a prior verdict or claim runtime verification.

Report SHA-256:
`239a8a67ec0879794634f4de5c4ade993030c5d72e14389a3d0e6a6e2d8e5438`.

Approval is bound to the exact reviewed artifacts in the unchanged
[R3-Rev10 manifest](r3-rev10-changed-files.sha256), SHA-256:
`f59f91ef30738671c8e7319e8d0f5aeacc73225412546485306d177c62c93602`.
All 21 entries were rehashed and matched when recording this decision. The report's archive
identity is `Archive(1).zip`, SHA-256
`d582d23efb676ef7df77f07353c584a36d7359f3333bf3c08c90fc904415e1a4`;
that identity is transcribed from report §2, not claimed as a separately inspected archive here.

No reviewed file is edited. Pre-approval OPEN/NOT_READY/candidate wording inside those exact
files remains historical submission metadata. This decision and R3-Rev11 supply the current
gate/dispositions for this baseline; historical R3-Rev9/Rev10 and earlier review verdicts remain
attached to their own reviews, not retroactively approved or overwritten.

## Finding dispositions — design level only

| Finding | Recorded disposition | Authority / downstream verification |
|---|---|---|
| TD-008 | RESOLVED_VERIFIED at design level | R3-Rev11 §3/§6/§8; native session lifecycle, intended-login/task/logout ordering, cookie-response adapter and logout recovery; execute CA and R10-X fixtures later |
| TD-010 | RESOLVED_VERIFIED at design level | R3-Rev11 §4/§6/§8; known-stale rejection, observed-change clearing and lifecycle recovery; client-state/runtime regressions retained |
| TD-011 | RESOLVED_VERIFIED at design level | R3-Rev11 §5/§6/§8; confirmed logout notification, receiving-tab clearing and loss/suspension recovery; supported-browser verification retained |

Zero open R3 blockers per independent R3-Rev11. Original Design Gap / Medium / blocking
classifications remain review history. The report's current action is NOTE for downstream
verification, separate from the RESOLVED_VERIFIED lifecycle status. Approval is of the written
design and validation plan, not implementation success or executed test results.

## Preserved approvals, risks and obligations

- Scope v1.2 / AM-002 and R2-Rev8 remain APPROVED; prior Scope/R2 approvals, FD-009 resolution
  and R2-Rev7 NOT_READY history remain unchanged.
- FD-001 task-write/retry protection, FD-003/004 accepted limitations and RA-FD006-001 remain
  within their exact approved boundaries. No new product guarantee, residual-risk acceptance,
  first-winner, atomic-render, guaranteed-delivery or mandatory browser capability is introduced.
- TD-001/002/003/004/005/007/009 dispositions and required regressions remain as recorded in
  R3-Rev11 §6. Written design approval does not waive those tests.
- TD-006 remains OPEN, nonblocking: Barry is accountable for Operations/R5 backup, restore and
  diagnostics, including logout-receipt clearing. Codex prepares designs, plans and handoffs;
  actual evidence and the R5 human gate remain required.
- TD-007 remains resolved with Barry accountable for R4/future upgrade compatibility. Native
  middleware/response-adapter/read-only facade patch compatibility, rekey preflight and command
  receipt compatibility fixtures remain binding downstream work.
- R3-Rev11 §7's LogoutReceipt proportionality NOTE is retained without changing the approved
  design or promoting exact recovery into a new minimum product requirement.
- Build/Test must execute the planned CA-T01–16, R10-X01–03 and affected prior regressions under
  separate appropriate authorisation. R4/R5 and human Build approval are not granted here.
  P1/IndexedDB remains paused.

This action records approval only. No Tasks, Analyze, implementation, application tests,
deployment or downstream gate work is performed.

## Exact reviewed artifact set

Repository-root-relative paths, copied unchanged from the reviewed manifest; this new decision
record is not itself a member of the previously reviewed candidate.

```text
63e216a5b74582dabb4509fce0316db3801519c099fd96fd6f9addb8c84b9ca3  specs/001-personal-task-manager/quickstart.md
229dba1ecbbd09d868c6c15069284a350087b98ae2c776a600fd032176413201  specs/001-personal-task-manager/auth-transition-protocol.md
0667d5cca398c4142c823b16b22e63476ae63efc23bf05874967a47a80f114cb  specs/001-personal-task-manager/research.md
7f5c56a3ec0ecdf52726b97bac41e5a2762340112e2b7559025e7d62e9b63120  specs/001-personal-task-manager/data-model.md
8b5fda2f122a5834df0993524343e5198a2bd7b36363340a56d5a8eac5cb6c9b  specs/001-personal-task-manager/plan.md
41ce09515e7ef3054a8aac3606e8991576f4e4b59d36b5eaab82aa27ef4413c2  specs/001-personal-task-manager/technical-design.md
71d4ceea413339f4a473508301557161c8a94d8a210bc18569f88de41f37757e  specs/001-personal-task-manager/checklists/feature.md
8ea9edb7b913e349fdebe478ab728f3c19b06569bcba718f2904ba3a09e64970  specs/001-personal-task-manager/review-followups/r3-pre-rev10-auth-protocol-snapshot.md
25b7f5834b223c4ab497b5868e8249f997da45c1d6e6b9e3c4e77326144cb9ad  specs/001-personal-task-manager/review-followups/r3-rev10-handoff.md
ecb87cc9af7e2425da25b8a1c6fb6e9640ef90704815a6f2cc242d9f323ebad6  specs/001-personal-task-manager/review-followups/r3-rev10-validation.md
2d43e287ecd74f9d608708e0a72943c3422e5e9464104afb66dd6aad04b76625  specs/001-personal-task-manager/review-followups/r3-rev10-evidence.md
688b57c11785af800b072843d7763c14875dbc6cbb5b35594201936cee03549b  specs/001-personal-task-manager/spec.md
da3fabe5cb3b403b05e466a5ea0401e724b8116a1fcf04d5544deed1f43dc39d  specs/001-personal-task-manager/contracts/ui.md
ce2c48e635207bed5a9a79aca9222819c380426a0f0802b9e9795ad1b5035637  specs/001-personal-task-manager/contracts/http.md
67162dc03401aed5b061b9c547d337a0ea75e3e0ddedf2741eac0935fbbf48c8  independent-reviews/review-remediation-policy.md
15e93e9da14b4ce7badd9ddf6e557cb7961ae5a6d01b0ea950102d997fb627f1  independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md
c6c6bd435cfe234239bd7b0e03f45e9aa751689baed7edca36b8e3cdbfb82650  independent-reviews/2_design-functional/Review_7/Taskify_R2-Rev7_Targeted_Functional_Design_Review_NOT_READY.md
b2ce2271525696eaad7d5850f36d7b23d3cdbeb573bebce59e047bc1942cd8a9  independent-reviews/3_design-technical/Review_9/taskify_R3-Rev9_technical_design_review.md
8199decf14eb62f063bab7db16bb4d98a54686783584b4cd1dc51aeb3370385e  specs/001-personal-task-manager/review-followups/r2-conventional-auth-amendment-proposal.md
5237fa167a79923b17589c34560ab70d38d754d985f531035126e57e6c00709a  specs/001-personal-task-manager/review-followups/r2-conventional-auth-amendment-evidence.md
009f33f8cfedd227a7342b079feb1705f8b6154e01b5b8ed13535b20e3207a15  specs/001-personal-task-manager/review-followups/r2-conventional-auth-amendment-scenarios.md
```
