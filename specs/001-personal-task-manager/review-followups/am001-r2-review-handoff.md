# AM-001 — Targeted R2 amendment review handoff

**Prepared:** 2026-10-23. **Scope:** APPROVED. **Revised Functional/Technical Design:** NOT APPROVED.
**Authoring status:** Spec Kit Plan research/design reconciliation and Checklist rerun completed;
independent targeted R2 assessment and explicit human sign-off pending. No code/tests/Tasks run.

## Authority and Scope metadata check

Barry Levy approved Scope 1.1 / AM-001 on 2026-10-23 (Australia/Melbourne) in
[R1-Rev4 Section 8](../../../independent-reviews/1_scope/Review_4/scope_review_report.md#8-final-human-scope-amendment-approval--2026-10-23-australiamelbourne),
following independent verification in Section 7. Exact approved candidate:
`61062311e5d1423f1d13f485c555158964e09f0548d13263bfefa0f048447fd4`.

Canonical spec received only approval/document-control metadata changes: approved header/date/
Barry reference, Section 8 link, approved 1.1 history entry, clarification/approval provenance and
R1-AM-004 DEFERRED documentary status. Its entire text from `## Product Boundary` onward remains
byte-identical to that candidate, including FRs, ACs, scenarios, historical clarifications and
assumptions. Historical pending-supersession phrases in those sections are retained verbatim;
the new document-control approval governs their status without rewriting product text.

Exact metadata-updated canonical SHA-256:
`b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`.
The original Section 12 approval and its exact hash remain recorded. R1-AM-004 remains DEFERRED
as directed in Section 8; no missing Section 13 was fabricated or risk-accepted.

## Changed Design artifacts and traceability

| Artifact | Amendment / evidence for R2 |
|---|---|
| plan.md | Rerun against approved Scope; unchanged stack/layers; session-bound view storage and explicit downstream gate HOLD |
| research.md §7 | Current-login four-field view; silent fallback/save failure; last-commit concurrency; session namespace versus small session-row evaluation |
| data-model.md | Replace account TaskFilterPreference with SessionViewState tied to existing Django session; add sort/direction; remove preference conflict version; task entities/versions/receipts unchanged |
| contracts/http.md | Direct task bootstrap chooses complete valid session view or defaults; replace account preference GET/PUT with authenticated CSRF-protected POST /api/session/view/; preserve GET safety and task locks |
| contracts/ui.md | Same-login restoration including sorting, independent valid sessions, new-login defaults; no view failure warnings or task-access block; current view retained on failed remembering |
| quickstart.md | AM001-V01–10 covers approved policy, save uncertainty, multiple tabs/sessions, privacy and technical races; retain R2-T05/07/12 regression cases |
| checklists/feature.md | Append CHK037–052; retain original 36 prompts/markers with explicit amendment routing |
| checklists/requirements.md | Reconsider existing 16 Scope-quality criteria, record unchanged 16/16 state without claiming Design approval |

DEC-AM-001-01 → HTTP/UI silent remembering/fallback → AM001-V05/06/10, CHK041–043.
DEC-AM-001-02 → login-scoped entities/HTTP/UI → AM001-V01–04/09, CHK038–040/047.
DEC-AM-001-03 → whole-tuple validation/defaults → AM001-V05/08, CHK037/041.
FR-013 / AC-006/007/011 → preserved domain sorting → quickstart scenario 5, CHK044.
FR-008/010/014 / SC-004 → unchanged tasks/privacy/load protection → AM001-V03/09/10 and retained
quickstart scenarios 6–9; no convenience failure authorises a task-data reset or write retry.

## Author decisions requiring targeted Design review

The Scope policy is settled. R2 must assess the following proposed mechanics, not reapprove Scope:

- Concurrent explicit tab selections use last successful server commit; existing tab controls stay
  independent. One save per tab and latest queued tuple reduce local response disorder.
- A lost/invalid save reply is internally unconfirmed, with no view toast, false success claim or
  special Retry flow; no automatic replay of an uncertain old choice. A newer explicit choice may
  still attempt remembering. Delayed commits may affect the shared view seen by future loads.
- Parameterless task bootstrap uses session settings/defaults; later polls carry the current tab's
  explicit full tuple and do not overwrite shared settings. Failed auth/task reads remain errors.
- A small session-bound view row replaces the account row to avoid stock whole-session middleware
  overwrite races. R3 must verify session-first lock order, login/expiry/rotation invalidation,
  independent transaction failures and no stale auth/session resurrection. No runtime proof exists.

Earlier unapproved proposals under review-followups remain audit artifacts. Their view notices,
blocking startup, optional retry prompts and account guarantees do not override approved AM-001.
The reviewed new handoff/current contracts are the requested amendment package. Independent R1/R2/
R3 reports were not edited. R2-Rev3 approval of its original package remains intact; it does not
approve these changed documents. Previous FD-002/005 verification is historical, with intentional
Scope-authorised changes to its persistence/conflict behaviour; FD-001 and FD-003/004 remain intact.

## Residual risks and exit conditions

| Item | Owner / next gate | Evidence still required |
|---|---|---|
| Silent resets, different login views, competing tabs, unconfirmed view remembering | Functional Design owner / targeted R2 reviewer | Assess AM001-V01–10 and CHK037–052; explicit human R2 approval of resulting Design, not author self-certification |
| TD-001 cross-tab private-content remanence | UI/security + Technical Design / R3-Rev2 | Conceal/invalidate before revalidation, logout/account-change signal/fallback and multi-tab A→logout→B scenario; not fixed by view reset |
| TD-002 previous preference-bootstrap failure | Functional Design then Technical Design / R2 then R3-Rev2 | Verify removal of blocking preference bootstrap and silent complete defaults without bypassing task/auth failures; independently determine disposition, still OPEN here |
| TD-003 full snapshot/receipt capacity | Performance/operations + Technical Design / R3 | Workload envelope, resource/protection argument and subsequent load evidence; no product task cap/SLA invented |
| TD-004 receipt digest privacy | Security/privacy + Technical Design / R3 | Keyed/unkeyed decision, rotation/replay and residual guessing risk; this view amendment preserves existing receipt definition pending that correction |
| TD-005 authentication throttling | Deployment/security + Technical Design / R3 | Enforcement point, forwarded-source trust, shared limits/failure mode and abuse scenarios |
| TD-006 diagnostics, backup and migration recovery | Operations lead, Technical Design handoff / R5 | Named owner, redacted reference tracing, session/view handling on restore, consistent task/revision/receipt backup and disposable restore/migration drill; no RPO/RTO invented |
| TD-007 canonicalization compatibility | Maintenance + Technical Design / R4 and upgrades | Versioned canonical format/old replay fixture and migration/rollback compatibility |
| FD-003 interruption; FD-004 no credential recovery | Product owner accepted risks, Design/Test retain | Preserve memory-only interruption disclosures and no-reset/transfer notices; no claim of risk elimination |
| R1-AM-004 archival evidence | Scope/document-control owner / historical follow-up | Retrieve authoritative Section 13 if available; DEFERRED, no condition on new approved Scope and no fabricated record |

All TD findings remain OPEN. No R3 solution or finding closure is claimed by completing this Plan.
After independent targeted R2 assessment obtain explicit human R2 approval, then reconcile remaining
technical corrections and seek R3-Rev2 plus human R3 approval in the prescribed sequence. Stop
before R2 sign-off now; no Tasks, Analyze, R4/R5 execution, Build or deployment is authorised.
