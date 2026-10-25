# R2-Rev8 approved incorporation — change/evidence map

**Date:** 2026-10-25. **Human Functional Design gate:** R2-Rev8 CLOSED — APPROVED by Barry Levy.
**FD-009:** RESOLVED_VERIFIED, from independent R2-Rev8 §2; not an author-invented disposition.
R3 NOT_READY; TD-008/010/011 OPEN. P1/IndexedDB paused. No Tasks, Analyze, Build or implementation.

## Authority and reviewed identities

[R2-Rev8 approval](../../../independent-reviews/2_design-functional/Review_8/Taskify_R2-Rev8_Targeted_Functional_Design_Review_APPROVED.md)
§1 identifies the exact reviewed documents, §2 verifies the targeted outcomes/FD-009, and §4
records Barry's explicit approval of the R2-Rev8 identity and unchanged amendment content.
The report was already supplied in Review_8 and is retained byte-for-byte. No earlier approval
is relabelled: the R2-Rev7 NOT_READY report remains unchanged as history.

| Preserved input | Verified SHA-256 |
|---|---|
| Approved Scope v1.2, current metadata-only spec.md | 688b57c11785af800b072843d7763c14875dbc6cbb5b35594201936cee03549b |
| R1-Rev5 decision/verification record, §§8–9 | 992ebc81381920a86d94041c6164f48235722c299f11184a9b3183c3a3716dbf |
| R2-Rev8 approval record | 15e93e9da14b4ce7badd9ddf6e557cb7961ae5a6d01b0ea950102d997fb627f1 |
| Approved conventional-auth proposal | 8199decf14eb62f063bab7db16bb4d98a54686783584b4cd1dc51aeb3370385e |
| Approved supporting evidence map | 5237fa167a79923b17589c34560ab70d38d754d985f531035126e57e6c00709a |
| Approved CA-T01–16 scenarios | 009f33f8cfedd227a7342b079feb1705f8b6154e01b5b8ed13535b20e3207a15 |

## Finding / decision incorporation

| Finding / authority | Operative change and retained protections | Planned evidence |
|---|---|---|
| FD-007 / CA-007 | Genuine overlapping anonymous responses follow actual valid-cookie selection, including same-account epochs. Remove first-browser/submission/server winner and mandatory competitor conflict. Preserve authenticated GET 302 / POST 303 before replacement credential evaluation, CSRF 403, stale-form revalidation and normal deliberate logout-first switching. | CA-T01–04/13; actual header vs server/callback order, current-owner/epoch/view outcomes |
| FD-006 / CA-006 and aligned Q1 | Server auth/owner/intended-login checks remain absolute. Known stale/mismatched replies are discarded and observed changes clear prior-login state. Boundary/uncertainty revalidation retained; no fresh auth trip per local action/normal response or atomic render fence. Preserve FD-001 and exact receipt/version retry. | CA-T05–09/15; wrong-owner and old-intent rejection, known vs unobserved state, task failure vs auth uncertainty |
| RA-FD006-001 / Scope AC-010 | Explicit prior-authorised display/in-flight residual while same-profile change is unobserved, with no maximum delay. No cross-account API access/mutation, old-intent rebinding, known-mismatch rendering or post-observation retention. | CA-T06–08; AM-002 exact risk boundary. Potential confidentiality consequence remains HIGH; exposure UNKNOWN; no runtime safety/frequency claim |
| FD-008 / CA-008 | Confirm intended shared revocation before success; promptly dispatch notifications, receiving runnable tabs clear/navigate. No guaranteed delivery/tab ACK/atomic clear. Preserve loss/suspension/late-message recovery, newer-login protection, independent devices, uncertain ACK and bounded CSRF recovery. | CA-T10–13/15; no new guaranteed fallback transport, extra polling or browser dependency |
| FD-009 | Record RESOLVED_VERIFIED with independent §2 provenance in current contracts, plan/research/model/guide and this handoff. Scope v1.2 supplies the AC-010 authority formerly absent at R2-Rev7. | Author source/identity checks; no retrospective revision of R2-Rev7 or automatic TD closure |

Historical R2-Rev5/R2-NAV-001 and R2-Rev6 approvals remain attached to their exact baselines;
only the approved targeted outcomes supersede prior clauses. FD-003/004 risk records remain in
their original bounds. No Scope product text, constitution or Spec Kit operation is changed.

## Changed operative artifacts

| Artifact | Targeted reconciliation |
|---|---|
| contracts/ui.md | Approved metadata/status; conventional anonymous ordering; local response checks/boundary revalidation; accepted display risk; aligned Q1; prompt but non-guaranteed notification and recovery. |
| contracts/http.md | Login/redirect/CSRF/epoch outcomes; remove winner-enforcement 409 obligation only, retain task conflicts and security. Old family/phase/rotation interfaces no longer operative requirements. No new endpoint prescribed. |
| data-model.md | Current logical login/view/authority states. Physical field tables preserved; obsolete custom auth tables clearly isolated as historical unapproved snapshots, not required planned schema. |
| research.md / plan.md | Current Scope/R2 authority, FD-009 and supersession; remove obsolete functional proof obligations. Existing stack, historical Plan check and unrelated technical decisions retained; no Plan rerun. |
| quickstart.md | Copy the 16 approved CA rows verbatim. Preserve R2-6R and earlier technical scenarios as history, update affected NAV/R3 privacy expectations to approved known-state/residual boundaries. No future commands executed. |
| technical-design.md / auth-transition-protocol.md | Current governing functional baseline and requalified R3 dependencies. Earlier auth mechanism retained only as historical snapshot; no architecture chosen or previous TD automatically closed. Technical-design §§3–8 unchanged. |
| checklists/feature.md | Append unchecked CHK061–068; identify superseded readings of CHK053–060, preserve prior text/checkmarks. No independent technical/runtime check is marked passed. |

New deliverables are this map, the [R3 handoff](r2-rev8-r3-handoff.md) and the
[SHA-256 manifest](r2-rev8-incorporation.sha256). The manifest uses repository-root-relative paths
for affected artifacts and unchanged authoritative inputs; it excludes itself.

## Validation / remaining work

Static author checks cover approval identities, exact CA table transfer, requirement/decision
trace, protected-file and unrelated-section preservation, local links, tables, whitespace and
hashes. These are not independent verification or executed acceptance tests.

Results: all 28 independent review Markdown records and canonical Scope are byte-identical
to the starting workspace. The approved amendment/evidence/scenarios retain their reviewed
hashes. The 16 CA rows match exactly; targeted comparisons preserve unrelated contracts,
physical field tables, task-state rules and historical checklist/scenario text. Markdown
links, table structure, fenced blocks and whitespace checks pass. An unrelated Finder
`.DS_Store` metadata update was observed and left untouched; it is not a review-record change.

No additional product decision is identified. The [R3 handoff](r2-rev8-r3-handoff.md) requires
technical mechanisms to be assessed proportionately against the retained contract under the
review-remediation policy; execution belongs to later authorised Build/Test. R3 remains NOT_READY
and requires independent assessment plus explicit human approval. No phase gate advances here.
