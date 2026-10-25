# Taskify — Independent R3-Rev10 Technical Design Review

**Review date:** 2026-10-25 (Australia/Melbourne)  
**Independent reviewer verdict:** **NOT_READY**  
**Human R3 gate at this review:** **OPEN — NOT APPROVED**  
**Reviewed submission:** Approved R2-Rev8 Functional Design incorporation and associated targeted R3 handoff, *before* the subsequent ADR-AS-004 technical remediation.  
**Provenance:** This Markdown report was reconstructed from the contemporaneous independent R3-Rev10 assessment in the conversation after the user requested a standalone copy. It was **not** present in the reviewed archive at that time. Historical verdict and evidence have not been retrospectively altered by the later R3-Rev11 approval.

## 1. Executive verdict

**The approved R2-Rev8 behavioural changes are coherently incorporated, but the technical architecture is not yet complete.** The former first-browser-established-login-winner and browser-wide atomic validation-to-render guarantees have been formally superseded under Scope v1.2 / AM-002 and R2-Rev8. Their old counterexamples are therefore not current grounds for blocking R3.

However, the submitted `technical-design.md` and `auth-transition-protocol.md` explicitly classify their older authentication state machine as a historical, unapproved candidate. No operative, integrated technical replacement has yet been selected and substantiated for the retained session lifecycle, protected operation ordering, known-stale client-state handling and confirmed cross-tab logout behaviour.

**Independent verdict: NOT_READY.** Retain **TD-008, TD-010 and TD-011 OPEN / BLOCKING** pending a coherent, proportionate design and targeted independent verification. TD-006 stays OPEN as a nonblocking Operations/R5 dependency. No further product/Scope decision is identified at this stage.

## 2. Baseline and static evidence

| Input / check | Independent result |
|---|---|
| Reviewed `Archive.zip` | SHA-256 `80247b22cd50cffa07f34268a4c34e162ca35319a0e69477b752202dd302eae8` |
| Approved Scope v1.2 / AM-002 `spec.md` | SHA-256 `688b57c11785af800b072843d7763c14875dbc6cbb5b35594201936cee03549b` |
| R1-Rev5 Scope independent/human approval | Present; approved AC-010 residual established |
| R2-Rev8 Functional Design | Independent READY followed by Barry Levy's explicit human APPROVED decision in R2-Rev8 §4 |
| Historical R2-Rev7 | NOT_READY preserved, not renamed or silently approved |
| R2-Rev8 incorporation manifest | **18/18 SHA-256 entries independently verified** |
| CA-T01–16 acceptance scenarios | **16/16 rows exactly match** the approved scenario document |
| Operative UI/HTTP/data-model/Plan/Technical Design | Reviewed for the targeted R2-Rev8 changes and R3 dependencies |
| Prior R3-Rev9 report | Historical assessment under subsequently superseded stricter requirements |
| Tests / implementation | No execution; written schedules are plans only |

The attached archive supported the claimed reconciliation and preserved approvals. At this review, no immediately preceding full workspace archive was attached for an independent binary before/after comparison of all 28 independent review records; the manifest and source identities were verified within the supplied archive. That limitation does not create an additional material R3 blocker.

## 3. Findings

### TD-008 — Conventional session lifecycle and protected commit ordering

**Original classification:** Design Gap. **Severity:** Medium. **Likelihood/exposure:** Unknown. **Confidence:** High that an operative technical mechanism is absent; no executed defect observed. **Action:** **BLOCK**. **Lifecycle status:** **OPEN**.

**Requirement provenance:** FR-010/AC-010, R2-NAV-001, CA-007/CA-008 and CA-T01–05/10/13–15. Genuine anonymous login responses may follow ordinary browser cookie application order. A first-winner election or competitor 409 is *not* binding. However, authenticated navigation preserves the current login, server operations must validate owner and intended login, and confirmed logout must revoke precisely its intended login. No command under revoked authority may commit after logout confirmation.

**Concrete failure concern:** A task command validates session L, pauses before committing, while a second tab revokes L and receives success. If the pending command later commits without ordering against revocation, protected integrity is violated. A separate delayed logout/401 response might delete a newer N cookie if native middleware response writers are not controlled. The former first-winner cookie race, by contrast, is explicitly permitted for genuinely anonymous logins.

**Established baseline:** Django's supported login, logout, PostgreSQL-backed sessions, secure session cookies and CSRF are the proportionate foundation. Native framework naming does not establish task/logout transaction ordering, response-cookie continuity or exact logout-acknowledgement truthfulness. See [Django auth](https://docs.djangoproject.com/en/5.2/topics/auth/default/), [Django sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/) and [SessionMiddleware source](https://github.com/django/django/blob/stable/5.2.x/django/contrib/sessions/middleware.py).

**Minimum remedy / exit:** Codex should select and document one cohesive conventional session issuance, validation, revocation and failure mechanism. Show deterministic command-first/logout-first effects, stale-intent rejection including same-account different epochs, authenticated GET/POST navigation, native fixation/CSRF protection, old-session replay failure and response cookie handling. Reuse mature Django/PostgreSQL patterns; justify any added custom state or adapter. Do not recreate the superseded browser-winner machinery.

### TD-010 — Known-stale response and browser private-state handling

**Original classification:** Design Gap. **Severity:** Medium. **Likelihood/exposure:** Unknown. **Confidence:** High in the missing integrated client/server design; no observed incident. **Action:** **BLOCK**. **Lifecycle status:** **OPEN**.

**Requirement provenance:** CA-006, approved AC-010/RA-FD006-001, accepted Q1 and CA-T05–09/15.

**Concrete failure concern:** A delayed response belonging to login A is applied after the client has *observed* selection of login B because local identity/generation state has not been coherently invalidated. This can reveal known-stale private content or permit old-intent rebinding. Previously authorised A content finishing before the change becomes known is the specific **approved residual** and must not be reclassified as a new defect.

**Established baseline/proportionality:** Server authentication and owner/intended-epoch guards, request identity/generation checks and lifecycle revalidation are normal proportional approaches. Repeated independent auth round trips cannot guarantee browser-wide atomic painting and are not required. Unrelated task refresh errors must retain FD-001 stale-row/write-lock rules; optional view-state failure must remain silent.

**Minimum remedy / exit:** Specify the client controller's known identity, generation, query/revision checks, observed-change clearing, initial/focus/resume/history concealment and uncertainty recovery, integrated with the selected server session guard. Demonstrate CA-T05–09/15 known-versus-unobserved schedules without adding an atomic-render obligation.

### TD-011 — Confirmed same-profile logout notification

**Original classification:** Design Gap. **Severity:** Medium. **Likelihood/exposure:** Unknown. **Confidence:** High in missing operative integration; no runtime delivery evidence. **Action:** **BLOCK**. **Lifecycle status:** **OPEN**.

**Requirement provenance:** CA-008 and CA-T10–14. Confirmed intended logout requires prompt notification dispatch; receiving runnable same-login tabs clear and navigate without waiting for a poll/action. Delivery is *not guaranteed*, and suspended/late/missed events recover at existing lifecycle boundaries.

**Concrete failure concern:** A runnable tab receives a confirmed logout signal but has no authoritative handler to clear private state; a late L notification may clear a newer N view; an unknown logout outcome may be misreported as confirmed. Failure to handle a received message can have serious shared-device confidentiality consequences. Lack of guaranteed transport delivery alone is not a contract violation under R2-Rev8.

**Established baseline:** BroadcastChannel is a plausible lightweight same-origin notification primitive, not authentication authority or durable guaranteed messaging. An earlier ADR-AS-003 proposal supplied credible partial protocol details, but remained historical and unadopted. See [BroadcastChannel](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API).

**Minimum remedy / exit:** Adopt or proportionately revise the existing epoch-scoped notification proposal in a coherent operative lifecycle: only committed logout ACK triggers dispatch; receiving L tabs clear/navigate promptly; duplicate/late L notices do not affect N; unknown outcomes never produce false confirmation; missing/unavailable delivery and suspension follow approved recovery. No new guaranteed transport or browser capability requirement.

## 4. Inherited finding register and regressions

| Finding | Status at R3-Rev10 | Requirement to preserve |
|---|---|---|
| TD-001 | Previously Resolved | Reverify account-isolation substance when old family/claim mechanism is replaced |
| TD-002 | Resolved | No material regression evidenced |
| TD-003 | Resolved at design level | Capacity hypotheses and future measurements retained |
| TD-004 | Resolved at design level | Keyed receipt privacy/integrity and historical replay retained |
| TD-005 | Resolved at design level | Shared authentication throttling and proxy boundary retained |
| TD-006 | **OPEN, nonblocking** | Barry owns Operations/R5 restore, backups, diagnostics and evidence acceptance |
| TD-007 | Resolved at design level | Barry owns R4/future upgrade compatibility and fixture follow-up |
| TD-008 | **OPEN, blocking** | Session/epoch/transaction/logout mechanism missing |
| TD-009 | Resolved at design level | Bounded isolated optional-view reader retained |
| TD-010 | **OPEN, blocking** | Integrated known-stale client-state mechanism missing |
| TD-011 | **OPEN, blocking** | Confirmed notification mechanism not operatively adopted |

FD-003/004's original human-accepted risks and RA-FD006-001's approved unobserved-display residual are unchanged. No new risk is silently accepted or waived by R2 incorporation.

## 5. Proportionality, remedial direction and stopping rule

The repeated historical remediation cycles substantially expanded custom authentication machinery to satisfy unusually strict first-winner, render-fence and notification guarantees. Barry subsequently authorised narrow R1/R2 amendments to remove those exceptional obligations while retaining substantive account security. The R3 stopping-rule reassessment has therefore already occurred; no renewed product trade-off is currently needed.

**Recommendation:** Codex should start from Django-native sessions and PostgreSQL, justify only those integrations needed for retained security/integrity, and prepare a **single cohesive technical candidate** for TD-008/010/011. Do not extend the historical `AuthorityFamily`/`AuthorityHandle`/`AuthTransition`, PREPARE/ISSUE/ACTIVATE, maintenance rotation or mandatory Web Locks/IndexedDB solely because earlier reviews discussed them. Technical review should evaluate the candidate's actual limits and failure modes, not turn reviewer suggestions into new binding requirements.

The required output is an integrated architecture/rationale, affected artifact reconciliation, deterministic CA-T01–16 and targeted regression plan, integrity manifest and independent R3 handoff. No code, software tests, Tasks, Analyze, Build or deployment are authorised by this NOT_READY verdict.

## 6. Independent verdict and gate conditions

**Independent R3-Rev10 verdict: NOT_READY. Human R3 gate: OPEN — NOT APPROVED.** TD-008/010/011 remain OPEN/BLOCKING. TD-006 remains OPEN/nonblocking with Operations/R5 exit. No additional Scope/Functional Design decision is required to prepare a proportionate technical remediation.

The next appropriate step at the time of this review was Codex's integrated technical candidate and fresh independent assessment. Human approval could occur only after a favourable independent design verdict; application/runtime validation remains separate.

### Historical chronology note — not part of this verdict

Subsequently, Codex supplied the **R3-Rev10 / ADR-AS-004 technical candidate** in a new archive. It was reviewed independently under **R3-Rev11**, which reached **READY FOR HUMAN APPROVAL** and was then expressly approved by Barry Levy on 2026-10-25. That later disposition is documented in `Taskify_R3-Rev11_Technical_Design_Review_APPROVED.md`. It does **not** retroactively change this R3-Rev10 NOT_READY report, the evidence available at this review or the status of TD-008/010/011 at that earlier point.
