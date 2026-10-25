# R3-Rev7 P1 — Anonymous-login continuity proposal (NOT APPROVED)

**Date:** 2026-10-24. **Status:** Proposal for Barry's decision and targeted R2/R3 review.
This is not an operative browser/storage requirement, risk acceptance or a resolved finding.

## Problem and proposed invariant

R3-Rev6 shows that a valid live A cookie cannot be the destination for an unrelated provisional
B response. Maintenance needs no credential issuance at all. Anonymous contenders need one
canonical pre-auth browser context before credentials are committed; absence of an active cookie
does not establish that no other anonymous attempt is already in flight.

P1 proposes a same-origin IndexedDB read/write transaction that atomically gets-or-creates one
random coordination ID for the browser storage partition. Store no tasks, view preferences,
passwords, session tokens or private-account identity. Release the transaction before any network
work; no long-lived Web Lock, visible-page lease or wait for another tab.

The ID is only a namespace, not authentication or a continuity credential. A server-owned context
record binds that namespace and owns the generation, pending candidates and current session.
Bind enrolment to same-origin CSRF and server-issued opaque proof; knowing/choosing an ID alone
cannot join an existing login, revoke it, recover its credentials or read its account data.
Detailed enrolment/key lifecycle and storage-clearing behaviour require R3 verification.

All initial contenders register under this SAME context before activation. Existing live Session
credentials can identify their context without IndexedDB, so ordinary task use and authoritative
logout must not depend on IndexedDB availability.

## Credential separation, conditional on approval

1. Keep candidate preparation/continuation proof out of the active Session credential. A candidate
   may prove its own CSRF-bound transition, never authorize private reads or replace A's credential.
   Prefer no per-attempt transport cookie when an opaque preparation challenge suffices.
2. ACTIVATE locks the shared context, rechecks generation/current-login/transition/deadline and
   commits at most one live login. Competing B is rejected after A wins, even if B's request was
   dispatched while anonymous. Authenticated page/POST navigation still preserves A (R2-NAV-001).
3. Deliver a confirmed credential in its own login-specific HttpOnly cookie name after commit.
   Server selection is the canonical context's current login, not newest/oldest surviving Cookie
   header entry. Missing required current proof fails closed, never chooses an older account.
4. A delayed cancelled candidate response cannot write any confirmed credential slot. A delayed
   confirmed response for retired A cannot overwrite current B's distinct cookie name, and the
   context rejects A after logout. Recovery can re-deliver only the actual current login's cookie;
   it cannot disclose tasks or restore another account from a provisional proof.
5. Cookie cleanup expires only explicitly retired names, never a shared name reused by a later
   login. Names, absolute expiry, retries, current-proof loss, header/cookie quotas and cleanup
   need concrete bounded contracts plus adversarial browser validation before P1 is recommended
   technically ready. This proposal is not itself a completed proof of all those details.
6. Logout locks the context, advances generation, revokes the current Session and cancels every
   pre-registered contender regardless of which proof cookies are presented. Other devices have
   different contexts. No old response or candidate can reactivate the signed-out generation.

This avoids the reviewer counterexamples by combining shared pre-auth ordering with separate
credential destinations. Multiple anonymous independent families plus a single overwritable
cookie do not meet that invariant; adding a recovery label to a cancelled cookie does not fix it.

## Specific human decision required — P1-A

May Taskify require atomic IndexedDB enrolment **for a fresh browser with neither an established
context nor a valid existing Session**, showing technical Retry when that storage is unavailable?
Established valid sessions must retain tasks/navigation/logout without IndexedDB.

All currently targeted browser brands must still be tested; no unsupported-browser exclusion is
approved. Requiring this additional storage capability where cookies/JavaScript otherwise work
can change approved availability. Barry previously required an alternative preserving current
availability, so that dependency cannot be silently incorporated under another API name.

Decision options:
- Reject the additional dependency; continue seeking a cookie-only or other availability-
  preserving mechanism. P1 remains unapproved and TD-008 stays open.
- Authorise preparation of a targeted R2 availability/storage amendment for P1-A, with any necessary
  Scope assessment and explicit human approval before incorporation. This does not approve P1's
  technical correctness, browser tests, cookie limits or R3 gate.

No elapsed time/no response constitutes permission. No forced re-login, ineffective logout or
finite cross-account exposure is offered as an accepted fallback. No claim of impossibility of
all other designs is made: an adequate cookie-only mechanism has not been established here.

## Required evidence before adoption

R7-T02/03 must hold and reverse candidate/confirmed headers around A becoming live and logout,
keep B's captured request proofs valid, lose broadcasts, exercise simultaneous anonymous forms
with no pre-existing Session, and show A remains usable until authoritative logout.
Also test atomic first enrolment in several windows, terminated transactions, private browsing,
blocked/deleted storage, cookie omission/eviction/header limits, repeated recovery and old-name
cleanup. Confirm no private data/credential in IndexedDB or logs and no coupling across devices.
These are planned scenarios, not executed tests.

Primary basis: [IndexedDB transaction model](https://www.w3.org/TR/IndexedDB-3/) supplies atomic
read/write transaction scheduling; [Django CSRF](https://docs.djangoproject.com/en/5.2/ref/csrf/)
supplies request-origin/token checks. Neither specification supplies the proposed auth protocol.
