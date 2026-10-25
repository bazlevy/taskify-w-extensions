# Taskify — R2-Rev7 Independent Targeted Functional Design Review — NOT_READY

**Review ID:** R2-Rev7  
**Review boundary:** FD-006, FD-007, consequential FD-008, Q1, AC-010 and related regressions  
**Review date:** 2026-10-25 (Australia/Melbourne)  
**Independent reviewer verdict:** **NOT_READY — targeted R1 Scope/AC-010 assessment required**  
**Human R2-Rev7 gate:** **OPEN — NOT APPROVED**  
**R3:** NOT_READY; TD-008, TD-010, TD-011 OPEN. No Tasks, Analyze or Build authority.

## Reviewed evidence

- Scope v1.1/AM-001, R1-Rev4 human approved: `specs/001-personal-task-manager/spec.md`, SHA-256 `b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca`.
- Approved R2-Rev5 navigation and R2-Rev6 FD-006/007/008/Q1 records remain unchanged. R2-Rev6 review record SHA-256 `bf5655e36c120dd79b35542430915de56817f4587245774abc50cad2ffafbf6f`.
- Submitted proposal `r2-conventional-auth-amendment-proposal.md` SHA-256 `8199decf14eb62f063bab7db16bb4d98a54686783584b4cd1dc51aeb3370385e`; evidence map `5237fa167a79923b17589c34560ab70d38d754d985f531035126e57e6c00709a`; 16 scenarios `009f33f8cfedd227a7342b079feb1705f8b6154e01b5b8ed13535b20e3207a15`. All three independently match their supplied SHA-256 manifest.
- This is a written-artifact review, not a runtime test or proof of R3 implementation feasibility. Files in the supplied archive were not modified.

## Targeted assessment

| Item | Finding | R2 judgment |
|---|---|---|
| **FD-007** | Genuine overlapping anonymous login responses follow ordinary browser cookie selection; no first-browser-winner promise. Authenticated GET/POST, stale-form CSRF handling, logout-first account switching and per-login view lifetime retained. | **Functionally acceptable** replacement; technical ordering/revocation mechanics remain R3. |
| **FD-006** | Server ownership/intended-login guards remain absolute. UI discards known stale replies, clears on observed change and revalidates on relevant lifecycle/uncertainty boundaries; no atomic render fence. Previously authorised content may briefly remain visible before change is observed. | **Functionally acceptable subject to AC-010 authority** and explicit bounded accepted risk. |
| **FD-008** | Confirmed shared-login revocation, prompt notification dispatch/receiving-tab clearing, resume/focus checks, newer-login protection and no false logout ACK remain. No guaranteed notification delivery or atomic clearing. | **Functionally acceptable** consequential alignment. TD-011 mechanism still needs R3 assessment. |
| **Q1 / FD-001 / FR-014** | Distinguishes server-protected operations from local cached UI interactions; no unnecessary auth round trip per click. Task load failure still blocks mutations, view-state failure stays silent, pending task identities/receipts protected. | **No substantiated regression**. |
| **CA-T01–16** | Cover cookie/header order, authenticated redirects, old-intent ownership, known/unobserved stale replies, logout notification/loss, CSRF, task integrity, view/session lifetime, regression. | **Sufficient written scenarios** for this targeted change; planned, not executed. |

## FD-009 — AC-010 wording / stale display

**Type:** SCOPE_ESCALATION. **Action:** BLOCK final R2 gate pending R1 authority. **Status:** OPEN.  
**Evidence:** Scope FR-010/AC-010 versus proposed amendment §§3 and 5 and CA-T07.  
**Scenario:** A's previously authorised task content can remain visible or complete an in-flight render on stale Tab A while same-profile cookie/session selection has silently switched to B.  
**Consequence:** Potential serious confidentiality disclosure to an observer of that shared browser; neither cross-account API authorisation nor wrong-account mutation is permitted.  
**Severity:** HIGH (potential consequence). **Likelihood:** UNKNOWN (unusual overlap/observation failure; no measured rate). **Confidence:** HIGH in the textual tension/counterexample; not a measured incident.  
**Minimum action:** Independent R1 determines whether the wording is already consistent or requires a narrowly scoped clarification/amendment and human Scope approval. The exact proposal text is in amendment §5; no additional architectural remediation should begin to settle a Scope-language question.

**Minor nonblocking editorial note:** Evidence-map language suggesting FD-008 itself requires RECONSIDER is better presented as **DEFER** of TD-011's proportionate notification mechanism. R2 never required guaranteed notification delivery or simultaneous tab clearing; preserve the original FD-008 history.

## Gate and next action

The replacement FD-006/007/008 behaviours match Barry's current direction and are materially complete at the functional-contract level, but **R2-Rev7 is NOT_READY** while AC-010's authoritative interpretation is unresolved. Current R2-Rev6 approval remains historically binding; the proposal does not silently supersede it.

1. Independent targeted R1 assessment of AC-010 using `taskify-r1-ac010-handoff.md`.
2. If necessary, exact Scope wording and Barry's human R1 amendment approval; otherwise document authoritative R1 no-amendment rationale.
3. Return R1 disposition to independent R2-Rev7 for a narrow final consistency/gate review; then Barry may approve the amended R2 contract.
4. Only after applicable approvals, Codex reconciles operative contracts and independent R3 reassesses TD-008/010/011. No automatic finding closure or implementation authorisation.
