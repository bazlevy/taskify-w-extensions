# Targeted R3 handoff — incorporated approved R2-Rev8

**Date:** 2026-10-25. **Inputs:** Scope v1.2 / AM-002 and R2-Rev8 APPROVED.
**FD-009:** RESOLVED_VERIFIED per independent R2-Rev8 §2, accepted by Barry under §4.
**Technical gate:** R3 NOT_READY; TD-008/010/011 OPEN. P1/IndexedDB paused.
No Tasks, Analyze, Build, implementation or runtime execution is authorised.

## Approval and review boundary

[R1-Rev5 §§8–9](../../../independent-reviews/1_scope/Review_5/Taskify_R1-Rev5_Targeted_Scope_Review_AC010.md)
supplies independent AM-002 verification and final Scope approval. Current canonical Scope
SHA-256 is `688b57c11785af800b072843d7763c14875dbc6cbb5b35594201936cee03549b`;
its human-approved product-definition candidate was
`87bf1574788c46eedf6412782e1b6a60915a4db0d4b643483d9e79736a2c745c`.

[R2-Rev8 approval §§1–4](../../../independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md)
verifies and human-approves the exact conventional-auth amendment/scenarios. The incorporation
map/manifest provides identities. R2-Rev7 NOT_READY remains the historical Scope-referral verdict;
it has not been renamed, overwritten or silently approved. R2-Rev5 navigation and unaffected
R2-Rev6 decisions remain historical approvals.

Review the current operative [UI](../contracts/ui.md), [HTTP](../contracts/http.md),
[logical states](../data-model.md) and [CA-T01–16](../quickstart.md). Prior R2-6R/R7/A9 fixtures
remain historical stimuli, not permission to reinsert first-winner, atomic-render or guaranteed
delivery requirements. R3-Rev9's strict-contract assessment is history under the amended baseline.

## Requalified technical obligations

| Finding (still OPEN) | Retained outcome / required R3 assessment | Superseded demand / evidence trace |
|---|---|---|
| TD-008 | Specify coherent conventional session issuance/selection, native security, authenticated R2-NAV-001 redirects, intended logout truthfulness, valid/invalid/revoked lifecycle, CSRF and login/view continuity. Distinguish anonymous in-flight selection from authenticated credential replacement. | No first-browser/server/submission winner, mandatory competitor 409 or coordination solely to preserve a winner. CA-T01–05/10/13–15. Anonymous response selection alone is no longer a contract defect. |
| TD-010 | Protected APIs authenticate/owner/intended-login guard. UI rejects known stale identity/query/generation/revision, clears on observed change and revalidates appropriate lifecycle/uncertainty. Preserve FD-001 and silent view failure distinctions. | No browser-wide check-to-render fence or independent auth trip for each local interaction/normal reply. CA-T05–09/15. Unobserved prior-authorised display/in-flight completion is explicitly allowed by AC-010, not a counterexample requiring renewed custom fencing. |
| TD-011 | Specify prompt logout notification dispatch and receiving runnable tabs' clear/navigation, plus missed/late/duplicate/unavailable delivery recovery, suspension/history, newer-login nonrevocation, independent devices and no false ACK. | No guaranteed transport delivery, tab-ACK wait, atomic clear, numerical observation deadline, new browser capability or faster-poll obligation. CA-T10–14. Do not reclassify the approved delivery limitation as an unmet absolute requirement. |

R3 must still demonstrate credible mechanisms for these retained requirements. None of the TDs
is closed by this handoff; native framework naming alone is not complete proof. Use established
Django/browser capabilities and justify every remaining custom state, dependency or safeguard.
The historical AuthorityFamily/Handle/transition/maintenance snapshots do not select the next
architecture. Reuse/remove/rejustify them through a coherent technical revision, preserving the
substance of earlier TD-001 account-isolation resolution and unchanged unrelated guarantees.

## Risk, evidence and phase limits

RA-FD006-001 is the approved narrow risk of prior-authorised content remaining visible/finishing
rendering during an unobserved same-profile change, with no maximum delay. Potential confidentiality
consequence is serious; likelihood unmeasured. It never permits cross-account API reads/mutations,
old-intent rebinding, known mismatches or private-state retention after observation. Neither
FD-003 nor FD-004 is broadened by this risk. Wrong-owner/old-intent access remains a material defect.

Apply [Review Remediation Policy](../../../independent-reviews/review-remediation-policy.md).
Qualify actual requirements, consequence, likelihood/confidence and proportionality; keep action
recommendations separate from finding lifecycle status. Do not restart incremental custom fixes
for superseded guarantees or turn reviewer architectural suggestions into new requirements.
An actual new product/browser/risk departure must be specifically escalated; no such additional
human choice is identified by this incorporation.

CA-T01–16 are the approved written tests, copied unchanged, not executed results. R3 supplies
design evidence and a deterministic plan for retained request/response/logout/receipt races.
Once a credible design is approved, Build/Test executes under separate authority. Do not demand
runtime proof in R2/R3 without a specific justified feasibility need; do not defer an unresolved
design gap as though execution were the only missing item.

Technical-design §§3–8 retain capacity/resource, receipt HMAC/compatibility, throttle, optional-view
and Operations/R5 rules unchanged. Barry remains accountable for TD-006 Operations/R5 and TD-007
Maintenance/R4 exits; Codex prepares designs/validation/handoff. No ownership or approval is risk
acceptance beyond the exact accepted records.

Next is targeted independent R3 assessment of a coherent updated technical candidate and affected
regressions, followed only by Barry's explicit R3 gate decision if warranted. This functional
incorporation is not that candidate's completed technical remediation, a READY recommendation
or permission to generate Tasks, run Analyze or advance Build. See
[evidence map](r2-rev8-incorporation-evidence.md) / [manifest](r2-rev8-incorporation.sha256).
