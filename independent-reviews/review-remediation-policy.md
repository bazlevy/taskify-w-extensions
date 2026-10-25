# Review Remediation Policy

## A. PURPOSE AND APPLICABILITY

Apply this policy when Codex is explicitly tasked with evaluating, planning or remediating
independent R2 Functional Design or R3 Technical Design review findings. It supplements the
existing development workflow; it is not a general policy for unrelated work.

This policy does not modify Spec Kit operation, approved product requirements, existing review
gates, reviewer independence or human approval authority. It creates no development phase and
does not authorise application implementation, product remediation or workflow execution by
itself. Do not alter Spec Kit commands, scripts, templates, extensions, workflows, configuration
or the project constitution to apply it.

## B. DEFAULT ENGINEERING STANDARD

Default to conventional production-grade software. Prefer established, mature, well-supported
approaches appropriate to the actual product requirements and risks.

Production-grade does not imply eliminating every theoretical failure, perfect behaviour under
exceptional timing conditions, absolute guarantees beyond approved requirements, or custom
architecture where established patterns suffice.

Preserve appropriate security, privacy, authorisation, data integrity and essential reliability.
Distinguish material security or integrity failures from recoverable usability, presentation and
availability issues. Classify the actual consequence rather than the technical label: an
availability or presentation failure can still cause serious harm. Low likelihood alone does
not excuse credible serious harm.

## C. FINDING QUALIFICATION

Before accepting a finding as blocking or recommending remediation, assess all five areas below.
Identify missing evidence and uncertainty explicitly; do not invent facts to complete the assessment.

1. **Requirement provenance:** Identify the exact approved requirement or obligation and its
   source/approval. Distinguish original Scope from later design clarification. Establish whether
   the guarantee was introduced during earlier remediation and whether it was approved. A later
   approved guarantee remains binding until formally changed; its origin alone does not waive it.
2. **Concrete failure:** State triggering conditions, observable consequence, affected users and
   systems, and supporting evidence. Distinguish a demonstrated execution, a reasoned design
   counterexample and an unsupported hypothesis.
3. **Risk:** Assess severity of consequence, likelihood/exposure and confidence separately.
   Include detectability and reversibility where relevant. Do not invent probabilities or treat
   high confidence as high severity. Lack of likelihood data is uncertainty, not proof of rarity
   or safety.
4. **Established production baseline:** Examine relevant framework-native capabilities,
   comparable established production practices, their actual guarantees and limitations, and
   suitability for the approved requirements. Use documentation or reliable evidence where
   practical. Label unsupported comparisons as assumptions; popularity alone is not assurance.
5. **Proportionality:** Compare the risk reduction with implementation and maintenance cost,
   added states, dependencies and failure modes, and simpler established alternatives.

Do not classify a finding as blocking solely because a failure is technically possible.
Conversely, do not dismiss a substantiated material failure because its trigger is unusual.

## D. FINDING ACTIONS

Use these four action classifications separately from each finding's existing lifecycle status:

| Action | Meaning |
|---|---|
| **BLOCK** | A material, substantiated defect requiring resolution under the currently approved requirements. |
| **RECONSIDER** | An existing or proposed requirement appears stronger than necessary or disproportionate to credible risk. |
| **DEFER** | The design is credible, but defined verification belongs to an appropriate downstream phase. |
| **NOTE** | A nonblocking concern, acceptable minor issue or optional improvement. |

Preserve finding IDs, original classifications, lifecycle statuses and reviewer history. Record
the recommended action alongside them, not in their place. Do not silently change earlier
review verdicts. An action recommendation cannot override an existing blocking verdict, close
a finding, accept a material risk or authorise progression through a human gate. NOTE is not
authority to accept a previously unaccepted material risk.

## E. REMEDIATION RULES

### BLOCK

Validate the finding and its consequence. Preserve binding requirements and security protections.
Prefer established, supported solutions and recommend the minimum proportionate correction.
Evaluate its complexity and new failure modes before expanding the design. Do not automatically
adopt the reviewer's suggested architecture or implementation: the required outcome and a
proposed means of achieving it are distinct.

### RECONSIDER

Immediately pause remediation of the disputed requirement when it is driving material complexity.
Do not introduce further custom mechanisms solely to satisfy it while human reconsideration is
pending. Present:

- The original Scope requirement and the additional or disputed guarantee, with provenance.
- The realistic failure and consequence, compared with conventional production behaviour.
- Trade-offs, residual risk and the precise human decision required.

Preserve the existing approved contract and gate status. RECONSIDER does not authorise Codex to
waive or change a binding requirement. An unmet approved requirement remains unresolved until
it is satisfied or formally changed. Unrelated, independently actionable work may continue
within its existing authorisation.

### DEFER

Specify the validation required and the appropriate downstream phase. Identify its owner,
observable exit evidence and the gate before which it must be completed. Do not require
implementation-stage proof during Functional or Technical Design without justification. A
credible design still needs suitable design evidence; DEFER must not conceal an unresolved
feasibility gap or bypass an existing blocking verdict.

### NOTE

Record the concern without blocking progress. Preserve any existing obligations or human gates;
do not convert an optional improvement into a new requirement.

## F. REQUIREMENT ESCALATION CONTROL

Never automatically convert reviewer recommendations, defensive mitigations, proposed technical
safeguards or hypothetical edge-case handling into new binding functional requirements.

For a material proposed requirement change, document:

- Original approved requirement and proposed change.
- Provenance, justification and expected behavioural consequences.
- Conventional alternative and residual risk.
- Required human approval through the applicable existing process.

No material change to approved Scope or Functional Design may occur without the appropriate
human decision. Neither reviewer advice nor Codex's remediation proposal substitutes for that
decision. Routine technical choices within approved behaviour do not automatically require
product escalation; identify the actual material departure before requesting a human decision.

## G. SIMPLICITY AND PROBLEM REFRAMING

For significant problems:

1. **Challenge necessity:** Is the problem genuinely material and worth solving? Can simpler
   product behaviour or architecture avoid it? A change to approved behaviour is a proposal
   requiring its existing approval process, not a unilateral simplification.
2. **Prefer established solutions:** Determine how mature frameworks and comparable production
   systems address the problem, including their limitations and fit to the approved needs.
3. **Use first principles only when necessary:** If suitable established approaches cannot meet
   approved needs, justify a custom solution direction and its added obligations.

Recommend the minimum required outcome or direction, not unnecessary implementation detail.

## H. REMEDIATION STOPPING RULE

After two substantial remediation/re-review cycles on the same underlying issue, pause additional
incremental remediation. Renaming the finding or changing the proposed mechanism does not reset
the underlying issue. This is a stopping trigger, not a requirement to attempt two fixes before
questioning disproportionate complexity.

Reassess the original requirement, actual likelihood and consequence, established production
approaches, complexity introduced by previous fixes, and new failure modes and maintenance burden.

Present options to simplify, accept a justified residual risk, or continue with a proportionate
remedy. Escalate material product trade-offs to the human owner. Risk acceptance and requirement
changes remain human decisions; presenting an option does not approve it. Preserve binding
obligations and existing gate status while the disputed remediation is paused. Unrelated,
independently actionable work may continue within its authorisation.

## I. PHASE BOUNDARIES

| Existing phase | Assessment or work boundary |
|---|---|
| **R2 Functional Design** | Assess necessary observable behaviours and outcomes. |
| **R3 Technical Design** | Assess feasible, proportionate technical mechanisms. |
| **Build/Test** | Implement and verify approved behaviour. |

Do not require runtime proof prematurely. If execution evidence is needed to establish design
feasibility, explain why and seek only the authority required by the existing workflow; this
policy grants none. Do not confuse design evidence with execution evidence, a test plan with
executed tests, an implementation detail with a product requirement, or a theoretical edge case
with a material defect.

## J. EVIDENCE, STATUS AND HUMAN GATES

Preserve finding IDs and classifications, approved baselines, human decisions, review iteration
history, accepted risk records and remediation evidence. Record new assessments with clear
provenance rather than overwriting earlier independent conclusions.

Never claim unperformed verification. Identify evidence as design reasoning, a planned validation
or an executed result, as appropriate. Only authorised humans may approve requirement changes,
material residual risks and formal review gates. Preserve reviewer independence: Codex's
recommendation or static check is not independent verification.

This policy does not supersede existing reviewer verdicts or human approvals, and does not
reclassify or resolve existing findings merely by being introduced.

## K. CODEX REMEDIATION OUTPUT

When processing material review findings, report concisely and with supporting evidence:

- Finding ID and original classification.
- Requirement provenance.
- Failure scenario and consequence.
- Severity, likelihood/exposure and confidence, assessed separately.
- Established production baseline and its limitations.
- Proportionality assessment.
- Recommended action: **BLOCK / RECONSIDER / DEFER / NOTE**, separate from lifecycle status.
- Minimum justified next action.
- Required human decision, if any; otherwise state that none is needed.
- Current binding requirement and gate status.

Scale detail to the material issue. Use existing review and remediation records; this policy
does not require a new directory hierarchy, general standards framework or reporting system.
