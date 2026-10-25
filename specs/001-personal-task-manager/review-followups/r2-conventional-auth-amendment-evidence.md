# Conventional authentication amendment — decision/evidence map and R2 handoff

**Date:** 2026-10-25. **Status:** targeted R2 proposal, not incorporated or formally approved.
Review [the amendment](r2-conventional-auth-amendment-proposal.md) and
[16 proposed scenarios](r2-conventional-auth-amendment-scenarios.md). Preserve the approved
Scope/R2 baselines and all existing finding statuses. R3 NOT_READY; TD-008/010/011 OPEN.
No Tasks/Analyze/Build, implementation or technical architecture selection.

## Requirement provenance and policy qualification

Original Scope FR-010/AC-010 requires private accounts and account isolation; FR-012/013 defines
login-scoped views; FR-008/009/014 preserves saved tasks and failure protection. The stronger
browser winner, after-arrival fresh-validation/render and Q1 fresh-interaction gates are explicit
later design outcomes in human-approved R2-Rev6, not requirements that Codex may simply dismiss
as reviewer suggestions. R2-NAV-001 is separately approved in R2-Rev5 and remains unchanged.

[Independent R3-Rev9](../../../independent-reviews/3_design-technical/Review_9/taskify_R3-Rev9_technical_design_review.md)
accepted the bounded assessment, retained three blockers, and recommended targeted R2 reassessment.
That recommendation did not amend requirements. Barry's current human direction now supplies the
specified behavioural changes and narrow risk acceptance. The [remediation policy](../../../independent-reviews/review-remediation-policy.md)
stopping/reconsideration rules pause further custom fixes to the disputed stronger guarantees,
without waiving the still-binding approved contract while review is pending.

| Finding / original classification and status | Failure / consequence and separate risk assessment | Baseline, proportionality and recommended action |
|---|---|---|
| FD-007 / related TD-008: original R3 classification Design Gap; Medium/blocking; OPEN. FD-007's R2-Rev6 written correction remains historically RESOLVED_VERIFIED/approved. | Two anonymous valid responses can change selected cookie/account/epoch/view in response order, defeating first-winner continuity. Severity: recoverable selection/view/continuity effects; display confidentiality considered separately below. Likelihood/exposure: requires overlapping anonymous submissions and response timing, unmeasured. Confidence: high in the documented native-cookie counterexample, not universal impossibility. | Native cookie processing does not elect the first browser winner. The extra guarantee drove custom states and availability trade-offs without a completed proof. **RECONSIDER** that requirement; minimum next action: review CA-007 and CA-T01–04, retaining authenticated redirects and all server guards. Human direction supplied; revised-design gate pending. |
| FD-006 / related TD-010: original Design Gap; Medium/blocking; OPEN. FD-006 R2-Rev6 written outcome remains historically approved. | Delayed authority evidence can become stale before render; unobserved prior-authorised content can be visible on a shared screen. Severity: potentially serious confidentiality impact, not merely cosmetic. Likelihood/exposure: cross-tab change plus delayed observation/response; no measured probability. Confidence: high in the specific reasoning, no runtime incident claimed. Detectability/reversibility: detected on later signal/check; clearing cannot undo viewing already exposed content. | Conventional request authentication, owner/intended-login guards and known-stale checks preserve core integrity but not atomic browser rendering. **RECONSIDER** the atomic/fresh-per-interaction guarantee, not API isolation. Minimum next action: review CA-006, explicit RA-FD006-001 and Scope boundary; no further custom fence while pending. Barry accepted the narrow risk; exact written amendment/Scope consistency still need gates. |
| Consequential FD-008 / related TD-011: original Design Gap; Medium/blocking; OPEN, partially specified. FD-008 R2 written outcome historically approved. | Missing notification can delay runnable-tab clearing; late old-login notices can target new state. Severity: stale-display confidentiality exposure and recoverable navigation issue; wrong-login revocation remains prohibited. Likelihood/exposure: delivery/channel/suspension failures, unmeasured. Confidence: high in the defined failure, not a measured rate. | Prompt signalling/receiving-handler clearing plus boundary recovery is appropriate; guaranteed delivery adds obligations not authorised by this new direction. **RECONSIDER** guaranteed-delivery/atomic-clearing interpretations only. Retain confirmed revocation, epoch-scoped notification/recovery and independent devices. Review CA-008/CA-T10–13; no new transport/availability requirement. |

These are Codex action recommendations, separate from original classifications, lifecycle status
and independent verdicts. No blocker becomes NOTE/DEFER/closed automatically. Current binding
requirements remain Scope v1.1 and approved R2-Rev6 until the formal change process completes.

## Established production baseline and evidence limits

Official [Django authentication](https://docs.djangoproject.com/en/5.2/topics/auth/default/)
documents session login/logout and authenticated access checks; [session documentation](https://docs.djangoproject.com/en/5.2/topics/http/sessions/)
documents server-backed state and login fixation protection. [RFC 6265 §4.1.1/§4.1.2](https://www.rfc-editor.org/rfc/rfc6265.html#section-4.1.1)
documents response-cookie races and replacement semantics. Checked 2026-10-25.
They support a conventional foundation, not a claim that stock Django alone implements every
Taskify requirement. Object ownership, intended-login protection, the exact authenticated
redirects and UI response/recovery rules still require design attention. No unsupported assertion
that all production systems share a particular risk tolerance or failure rate is used.

The counterexamples are design reasoning in ADR-AS-003 and independent R3-Rev9, not executed
tests. Risk acceptance comes from Barry's request, not the framework documentation or Codex.

## Prospective artifact changes — none applied now

| Artifact | Minimum eventual change / preserved obligation |
|---|---|
| contracts/ui.md | Replace FD-007 winner/conflict guarantee; replace FD-006/Q1 universal fresh/atomic gates with known-state checks and boundary revalidation; record risk; align notification delivery language only. Keep normal forms, task failure states, privacy clearing and R2-NAV-001. |
| contracts/http.md | Anonymous races may create/select valid sessions normally; remove winner-enforcement 409 obligation only. Keep authenticated GET 302/POST 303, CSRF 403, API auth/intended-epoch 401, owner checks, task conflict codes, versions and receipts. No new route prescribed. |
| data-model.md | Logical selection/view/response states follow the revised distinction; fresh login defaults and valid-login tuple retained. No physical schema/key/store choice at R2. |
| research.md / plan.md | Record new decision/risk provenance; drop unjustified functional obligations after approval. Retain stack/history and no new browser capability. No Plan command run now. |
| quickstart.md / checklists/feature.md | Proposed CA-T01–16 map old expectations explicitly; after approval reconcile affected scenarios/questions, preserving historical rows and unaffected regressions. No checks are pre-marked passed. |
| technical-design.md / auth-transition-protocol.md | After R2/Scope gates, R3 reassesses what remains necessary; no first-browser election, browser-wide render fence or guaranteed notification obligation is inherited from superseded clauses. Custom mechanisms may be retired only with a coherent replacement/regression assessment. No architecture chosen here. |
| Scope spec.md / independent reports / approvals / prior proposals and manifests | Unchanged. Proposal §5 provides exact optional canonical clarification text and required R1 decision; no Scope authority is inferred from R2 risk acceptance. |

## R3 dependencies and phase boundaries

After approval, reassess TD-008 against native ordering, valid/invalid session lifecycle,
R2-NAV-001, intended logout and S2, not the superseded first-winner property. Reassess TD-010
against server owner/epoch guards, known-stale clearing and appropriate lifecycle/uncertainty
checks, not an atomic render proof. Reassess TD-011 against prompt dispatch/receiving behaviour,
lost/late notification recovery and no false ACK, not guaranteed transport delivery.

R3 must still show a credible mechanism for the retained contract; removing a stronger requirement
does not prove remaining security/continuity correct. Preserve TD-001 privacy history, optional
view isolation, receipt/version/task ordering, CSRF/native fixation, browser coverage and Barry's
TD-006/007 owners/exits. P1 stays paused; no Web Locks or new storage dependency is introduced.

Once a credible technical design is specified, execution evidence belongs to Build/Test under
existing authority; defer only that execution, with owners/scenarios/gates, not unresolved design
feasibility. Independent R3 disposition and explicit human R3 approval remain necessary later.

## Targeted independent R2 handoff / remaining decisions

Review only CA-006/007, RA-FD006-001 and consequential CA-008, plus their Scope/session-view/task
protection traces. Verify that stale forms/authenticated requests are not confused with genuinely
anonymous in-flight requests, and that the narrow risk is not presented as an API access waiver.
Review the disclosed completion of already-authorised in-flight content within that risk boundary.

The precise remaining consistency item is AC-010's broad “view” wording: proposal §5 supplies
the recommended targeted clarification. Obtain authoritative R1 assessment; a canonical addition
requires Barry's Scope approval through the existing process. If R1 confirms that existing Scope
already covers this interpretation, record that instead of creating a needless version change.
No new behavioural choice about first-winner guarantees or mandatory coordination is requested.
Formal approval of the independently reviewed Functional Design is still pending; no gate is closed.

Only this map, the proposal, scenarios and [proposal SHA-256 manifest](r2-conventional-auth-amendment.sha256)
are created. Existing operative contracts, independent reviews and approvals remain unchanged.
Static preservation, links, document structure, IDs and hash checks are author checks, not review
approval or runtime verification. No product remediation or downstream phase is performed.

## Source identities and static validation

| Preserved source | SHA-256 |
|---|---|
| Scope v1.1 spec.md | b1eb181074f746f355b80a22d67d9ccc5f1f5d6b94dcea64e320610479339fca |
| R2-Rev6 approval record | bf5655e36c120dd79b35542430915de56817f4587245774abc50cad2ffafbf6f |
| Independent R3-Rev9 report | b2ce2271525696eaad7d5850f36d7b23d3cdbeb573bebce59e047bc1942cd8a9 |

All 122 pre-existing files still present match their start-of-task hashes, including the operative
contracts, Scope, review records, remediation policy, AGENTS.md and Spec Kit files. Archive.zip,
present at the initial snapshot, was independently removed during the session; this task did not
write/delete it or recreate it, and it is not used as amendment approval evidence. Only the three
new Markdown proposals and their new manifest are authored here. Diff/whitespace/links/tables and
the complete unique CA-T01–16 identities are checked; the proposed-artifact manifest omits itself.
