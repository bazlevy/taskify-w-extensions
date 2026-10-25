# R3-Rev7 — TD-008 correction / evidence map and decision HOLD

**Prepared:** 2026-10-24. **Status:** Author correction package; NOT READY / NOT APPROVED.
TD-008 remains open. Rotation and CSRF are corrected at author-design level; the anonymous
continuity mechanism is not complete. This is not independent verification or risk acceptance.

## Controlled authority

Input: [R3-Rev6 independent report](../../../independent-reviews/3_design-technical/Review_6/taskify_R3-Rev6_technical_design_review.md).
It retains TD-001/002/003/004/005/007/009 resolved at written-design level, TD-006 nonblocking
with R5 evidence due, and TD-008 open as the sole technical blocker.

[Human R2-Rev5 §7](../../../independent-reviews/2_design-functional/Review_5/Taskify_R2-Rev5_Targeted_Functional_Design_Review.md)
APPROVED by Barry on 2026-10-24 includes R2-NAV-001. Source metadata now reflects that actual
decision; old review text/approvals are not rewritten. Scope v1.1 remains byte-identical:
`b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`.
The prior nine navigation-manifest entries match this turn's input; fresh technical files do not
inherit a new R2/R3 approval. All earlier reports/maps/manifests/checklists/handoffs remain unchanged.

## Targeted correction / evidence

| TD-008 component | Author correction / evidence | Planned validation / status |
|---|---|---|
| Maintenance rotation overwrites live A | protocol §2; technical-design §2; HTTP POST /api/auth/rotation/; UI continuity; data-model maintenance rule. One family-locked transaction moves only Django key/view, same browser handle/account/epoch/original expiry; no provisional ISSUE/ACTIVATE/CSRF or authority-cookie write. Retired captured inner key resolves through the current same-epoch family, never spurious 401/re-login. Logout retains valid browser proof | R7-T01: both claim/rotation/logout orders, delayed response, old-key/view races, zero authority-cookie writes and actual family revocation. Author design corrected, runtime/independent verification outstanding |
| Anonymous B overwrites A after A becomes usable | Old one-slot provisional writer and forced-relogin fallback explicitly withdrawn in protocol §4, HTTP/UI/data model, Plan/research. The rotation fix is not falsely extended to separate anonymous roots. Continuity/logout outcome remains mandatory | R7-T02/03 are exact required outcomes, but mechanism remains HOLD. P1 is a proposal, not an adopted implementation or resolved finding |
| Related cross-tab CSRF handoff | protocol §3; HTTP GET /api/csrf/; UI recovery. Obtain current no-store masked token, no Session/auth/task/view writes. Trusted pre-dispatch csrf_failed allows one bounded housekeeping/logout retry only after fresh token and same intended epoch; changed identity never receives old logout intent. No false logout ACK | R7-T04–06: pre-login tab, actual login CSRF-secret rotation, first/second 403, changed identity during refresh, lookup outage and proxy/unknown responses. Author design corrected, runtime/independent evidence outstanding |
| FD-001 / view convenience regression | Task mutations do not automatically replay after CSRF rejection; exact ID/payload and valid saved-task snapshot/explicit retry remain. View remembering stays silent, fresh token only for a later explicit selection; no old-tuple replay or new task lock | R7-T07 plus R2-T05/07/12 and AM001-V01–10; no accepted-risk expansion |
| R2-NAV-001 / existing resolved TDs | R2-Rev5 approval correctly referenced. Authenticated navigation preserves account/epoch/view, logout-first switching unchanged. Technical-design §§3–8 retained verbatim; TD-001 privacy/family fence retained. Maintenance's stable same-account handle removes a source of inconsistency rather than permitting account replacement | NAV-01–08 / R7-T09; independent dispositions quoted only, not newly closed by author |

## Exact outstanding decision — P1-A, not adopted

[Concrete unapproved P1](r3-rev7-browser-context-proposal.md) proposes atomic first-browser
context enrolment and separates candidate delivery from confirmed credentials. It records the
anonymous ordering invariant, enrolment/proof isolation, cookie/cleanup work still required and
adversarial scenarios. It does not pretend IndexedDB itself proves the complete auth protocol.

Its specific new dependency is IndexedDB for a **fresh profile with neither established context
nor valid existing Session**; a blocked/unavailable store could prevent fresh sign-in even though
cookies/JavaScript work. Existing sessions' task/navigation/logout must remain independent of it.
Barry previously required an availability-preserving alternative, so this dependency is NOT
silently incorporated. No new browser exclusion, forced re-login or ineffective logout is accepted.

Barry has been asked whether to:
1. Reject that dependency and continue seeking an availability-preserving alternative; or
2. Authorise preparation of P1-A for targeted R2/human consideration, without adopting it or
   granting technical approval. Any eventual incorporation needs the appropriate explicit decision.

No answer/elapsed time is approval. An adequate cookie-only alternative has not been established
here; that is not a proof that none exists. This package cannot be recommended ready or claim the
requested full TD-008 resolution while anonymous continuity and its mechanism remain open.
R7-T08 is conditional evidence, not an approved supported-client test exclusion.

## Changed artifacts and checks

Eight source design files changed: technical-design.md, auth-transition-protocol.md,
contracts/http.md, contracts/ui.md, data-model.md, plan.md, research.md and quickstart.md.
The new P1 proposal and this evidence map are included in the hash manifest.

Static comparisons: Scope/review/approval/checklist/history bytes unchanged; technical-design
§§3–8 and HTTP snapshot/view/task mutation/failure contracts unchanged; task forms/actions/view
remembering and User/TaskListState/Task/MutationReceipt/AuthThrottleBucket/sort sections unchanged.
All AM001-V01–10 and unrelated R3-V03/04/05/06/07/09 scenario rows retained; NAV-08 changes only
to reference the actual R2-Rev5 approval. Prior R5 rotation/delivery fixtures are marked historical
where superseded; current R7-T01–09 state the mandatory outcomes and explicit HOLD.
Local links, scenario trace, whitespace and SHA-256 identities are author document checks only.

Barry remains TD-006 Operations/R5 and TD-007 Maintenance/R4 accountable; Codex prepares designs,
validation/handoffs. No operational policy, restore/upgrade drill or runtime test is claimed.
No application code, Tasks, Analyze or Build. R3 remains unapproved; no finding independently
verified or newly risk-accepted by this author correction.

[SHA-256 manifest](r3-rev7-changed-files.sha256) covers ten changed/new Markdown artifacts,
relative to specs/001-personal-task-manager/, omitting its own hash to avoid circular identity.
It identifies this correction/proposal package, not a complete approved TD-008 solution.
