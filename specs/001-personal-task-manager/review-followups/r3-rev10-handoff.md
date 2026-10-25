# R3-Rev10 targeted independent technical review handoff

**Date:** 2026-10-25. **Submission:** ADR-AS-004/native-session technical candidate.
**Binding inputs:** Scope v1.2 / AM-002 and R2-Rev8 APPROVED; FD-009 RESOLVED_VERIFIED.
**Unchanged gate:** R3 NOT_READY; TD-008/010/011 OPEN. P1/IndexedDB remains paused.
No implementation, application tests, Tasks, Analyze or Build has been authorised/performed.

## Review packet

- [Integrated technical design](../technical-design.md) §§0–2 and unchanged §§3–8.
- [Authentication lifecycle / ADR-AS-004](../auth-transition-protocol.md), normative technical
  candidate §§1–8.
- [Data model](../data-model.md), native Session payload, guard and confirmation receipt.
- [Finding/proportionality evidence map](r3-rev10-evidence.md).
- [CA-T01–16 deterministic schedules and prior-test reconciliation](r3-rev10-validation.md).
- [Changed-file and authoritative-input SHA-256 manifest](r3-rev10-changed-files.sha256).
- [Historical protocol snapshot](r3-pre-rev10-auth-protocol-snapshot.md), not operative instructions.

## Requested independent assessment

1. TD-008: native fresh login/cookie selection, authenticated redirects and native security;
   intended-login guard before owner/receipt/task/view work; Session-first command/logout
   commit ordering; response adapter preventing delayed auth-cookie deletion/re-save; exact
   logout receipt confirmation, duplicate/CSRF/unknown outcome recovery and S2 continuity.
2. TD-010: current-generation/identity/query/revision checks, observed-change clearing, concealed
   lifecycle/recovery discovery, no old-intent rebinding, correct task/view failure distinction.
3. TD-011: after-commit ACK-triggered epoch-only notification, prompt receiving-tab clear/navigation,
   loss/unavailable/suspended fallback, no false ACK or newer-login effects.
4. Regression: CA-T01–16, X01–03 and retained prior fixtures, optional-view isolation/capacity/
   command HMAC/privacy/throttle/upgrade/restore mechanisms and existing owners/exits.

Requalify the original R3-Rev9 Medium blocking Design Gaps against the now-approved CA-006/007/008.
Independent report history remains exact. First-browser winner, atomic rendering and guaranteed
delivery are superseded requirements; their possibility alone is not evidence against Rev10.
Conversely, server cross-account access/mutations, old-intent replay, known-stale rendering,
post-observation private retention and false logout confirmation remain defects.

The design retains only three server integration safeguards beyond native authentication:
epoch/Session guard, compact logout-confirmation receipt and restricted response save/cookie
adapter. Review their necessity, failure modes and patch/upgrade compatibility. No authority-family
registry, multistage login, periodic rekey, browser lock or durable browser storage remains.
"Confirmed notification" means the logout was committed, not message delivery assurance.

## Remaining risk and exit boundaries

No new product-policy or risk-acceptance decision identified. RA-FD006-001, FD-003/004 and approved
functional limits remain exact. Transport loss and unobserved authorised display are handled at
their approved boundaries, without a zero-exposure/maximum-delay promise.

Barry remains accountable for TD-006 Operations/R5 and TD-007 Maintenance/R4. Add logout-receipt
restore clearing to R5 evidence and session adapter/rekey preflight to R4 compatibility fixtures;
Codex prepares documentation/plans, later authorised Build/Test supplies execution. No assignment
or this submission is risk acceptance.

Author static document/hash checks are distinct from independent design verification and runtime
tests. Please return a targeted disposition/evidence assessment; the author does not pre-close
findings or declare readiness. Even a favourable independent review requires Barry's explicit R3
human gate before any subsequent Tasks/Analyze/Build authorisation.
