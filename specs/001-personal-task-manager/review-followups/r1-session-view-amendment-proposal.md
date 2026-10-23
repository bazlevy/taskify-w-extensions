# R1 proposal — Authenticated-session view settings

**Prepared:** 2026-10-23. **Status:** PROPOSED; independent R1 review and formal human approval pending.
**Source:** [R3-Rev1 Addendum A](../../../independent-reviews/3_design-technical/Review_1/taskify_R3-Rev1_technical_design_review.md).
The user endorsed the intended behaviour, not a replacement R1/R2/R3 gate approval.

## Baseline and amendment boundary

The approved [specification](../spec.md) remains unchanged at SHA-256
`4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`.
This proposal supersedes only cross-session view restoration and Design's delegation of sort
restoration. Task persistence/synchronisation (FR-008/010, SC-004), privacy, task-load protection
(FR-014), four sort methods, defaults, and existing ordering/tie rules remain in force.

“Authenticated browser session” means the current login shared by tabs using the same session
cookie. Refresh or opening a tab is not a new login. Logout, expiry, or a new login starts a new
view-settings lifetime; another browser/profile/device has its own session, even for the same
account. Closing a tab alone does not end a still-valid login. Browser restart follows validity of
that login; this amendment introduces no new authentication expiry or remember-me policy.

## Proposed replacement requirement text

**FR-012 (replace):** The system MUST remember the selected status filter, priority filter, sort
method and direction within the current authenticated browser session. Refreshing or opening a new
tab in that session MUST restore its latest remembered view. Live changes to already-open tabs are
not required. Logout followed by login, a new authenticated session, and another browser, profile
or device MUST start with defaults: Status All, Priority All, Due Date earliest first. When view
settings cannot be restored but saved tasks can load, the system MUST show tasks using defaults,
give a brief nonblocking notification, and allow the user to reapply filters and sorting. Failure
to load task data MUST continue to obey FR-014.

**FR-013 (targeted change):** Retain all existing sort methods, directions, undated-last handling
and same-date tie rules. Replace “sorting-preference restoration [is] owned by Functional Design”
with “sort method/direction restoration follows FR-012; remaining tie cases are owned by Functional
Design.” No change to initial Due Date earliest-first sorting.

**AC-005 (replace):** Given a selected view remembered in the current authenticated browser session,
when the user refreshes or opens another tab in that session, then all four settings are restored
and the matching tasks appear in that order. Existing open tabs need not update live. After logout
and login, or in a distinct browser/profile/device session, the view starts with defaults. If view
restoration fails but task loading succeeds, tasks appear in the default view with a brief
nonblocking notice and usable view controls; task-load failure retains AC-008 protection.

## Other Scope text affected on later incorporation

| Location in spec.md | Targeted amendment |
|---|---|
| Clarifications: surviving filter-restoration outcome | Annotate as superseded by the new approved decision; retain historical decision provenance |
| Authorised Scope decisions / downstream preference delegation | Append a newly approved decision reference/date when actually supplied; identify the narrower session lifetime and inclusion of sort |
| User Story 3 independent test | Add same-login refresh/new-tab view restoration and nonblocking defaults fallback |
| User Story 4 scenario 4 | Replace later-session filter restoration with current-login refresh/new-tab restoration of all four view settings; keep task cross-device scenarios unchanged |
| Edge Cases | Add unavailable view settings → default view/notice; distinguish task data unavailable → FR-014 |
| Task Filter / Task Sorting entities | Clarify session-scoped display settings, separate from account-persistent task records |
| Assumptions and Exclusions: filter and sort bullets | Replace later-session restoration/delegation with this lifetime and defaults; exclude cross-device view persistence and required live cross-tab updates |

## Proposed acceptance examples

| ID | Scenario and required result |
|---|---|
| R1-VIEW-01 | Choose Completed/High/Title descending; refresh the same valid login → restore all four, without changing task data |
| R1-VIEW-02 | Remember that view, open a new tab → restore it; an already-open tab may keep its old view until refreshed/reopened |
| R1-VIEW-03 | Logout and sign in again as the same account → All/All/Due Date ascending; saved tasks remain intact |
| R1-VIEW-04 | Sign in to the same account in a distinct device/browser/profile → defaults, with account-synchronised tasks still available |
| R1-VIEW-05 | No stored settings → defaults without an error; unreadable/invalid settings with successfully loaded tasks → defaults and a nonblocking notice |
| R1-VIEW-06 | Task loading fails regardless of settings → task error/Retry/support and blocked task writes, including uncertain replay under FD-001 |
| R1-VIEW-07 | Account A logs out and B logs in in the shared profile → B receives defaults and no A settings/content; TD-001 concealment remains separately required |

The R2 proposal specifies concurrency, save acknowledgement, and degraded-save feedback. These
details must not be interpreted as already approved by this R1 proposal.

## Precise approval requested

After independent R1 review, the human Scope owner must explicitly approve replacement FR-012,
targeted FR-013/AC-005 and related text; accept loss of view restoration across later logins and
devices; and approve remembering sorting within the current login plus the nonblocking fallback.
Record the resulting decision against a concrete reviewed amendment revision. No Scope text is
authorised to change by this document alone. Then review and approve the dependent
[R2 proposal](r2-session-view-amendment-proposal.md).

**R1 reviewer verdict:** PENDING. **Formal human R1 decision:** PENDING.
**TD-002:** OPEN. Existing Scope approval/history remain valid for their original baseline.
