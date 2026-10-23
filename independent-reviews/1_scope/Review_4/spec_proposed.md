# Feature Specification: Personal Task Manager

> **INDEPENDENT R1 REVIEWER PROPOSAL — NOT CANONICAL OR FORMALLY APPROVED**
>
> Prepared for Taskify Scope Amendment AM-001 (R1-Rev4), 2026-10-23. Codex owns the canonical `specs/001-personal-task-manager/spec.md`. This file is a proposed replacement only. The approved Scope remains the unchanged canonical file (SHA-256 `4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`) until reconciliation, independent verification and explicit human approval.

**Feature Branch**: `001-personal-task-manager`

**Created**: 2026-10-20

**Reconciled**: 2026-10-22 (R1-011 approved; R1-006 wording corrected; independently verified)

**Status**: **REVIEWER PROPOSAL — AM-001 NOT APPROVED**; preceding Scope baseline remains approved

**Approved Baseline**: 2026-10-22; metadata-corrected SHA-256 `4ef233db3b4dfdbcbd50b4225e1613ef30fedfabf19b618dc17fd5bb0d2c74ae`

**Proposed Revision**: 1.1 / AM-001, 2026-10-23; formal human amendment approval PENDING

**Review Reference**: `independent-reviews/1_scope/Review_4/scope_review_report.md`

**Original Approval Reference**: Historical R1 report Sections 12 and 13; latest Section 13 copy to be verified in the authoritative history

**Input**: User description: "Build a simple personal task manager."

## Scope Version and Amendment Register (reviewer proposal)

| Scope version | Date | Change ID | Summary and rationale | Human approval |
|---|---|---|---|---|
| 1.0 (reference label) | 2026-10-22 | Baseline | Task manager Scope with documented downstream deferrals; a subsequent metadata-only correction was separately confirmed without changing requirements. | **APPROVED** |
| 1.1 (proposed) | 2026-10-23 | **AM-001** | Narrow filter/sort view restoration to each authenticated browser login, add sort restoration, allow silent defaults on view-state failures. Prompted by R3-Rev1 TD-002 and Addendum A, which exposed preference-read failure blocking otherwise available tasks. Changes FR-012, FR-013, AC-005 and related scenarios. | **PENDING — NOT APPROVED** |

**AM-001 clarification decisions recorded 2026-10-23 (not formal amendment approval):**

- **DEC-AM-001-01 / R1-AM-001:** View selections apply immediately. If remembering them fails or is unconfirmed, keep the user's working view without a save/restoration warning; later refresh/new tabs use available valid remembered settings or complete defaults. View-state failure never blocks otherwise loadable tasks.
- **DEC-AM-001-02 / R1-AM-002:** Each authenticated browser/profile/device session maintains its own view while valid; other sessions for the same account do not import it. Saved tasks still synchronise.
- **DEC-AM-001-03 / R1-AM-003:** Treat the four settings as one view. If any remembered component is invalid, missing or unreadable, use all four defaults without warning; genuine first use also defaults normally.
- **R1-AM-004:** Documentary follow-up: the supplied historical R1 report lacks the later Section 13 metadata-correction approval entry. Preserve both approvals and obtain the latest authoritative record on incorporation; this is not a new product decision.

Detailed evidence, alternatives, regressions and review dispositions are recorded in Review 4's independent report. Git/byte hashes may support exact verification but are not the human-facing version label. Nothing above records formal approval of AM-001.

## Product Boundary

The first release is a personal task-management **web application for desktop and mobile browsers**. Each first-time user may self-register a private account without an invitation or administrator provisioning. Users sign in to their private accounts to access one personal task list each. Successfully saved tasks and their changes persist between sessions and synchronise across browsers and devices signed in to the same account. Tasks belonging to one account must not be accessible through another account.

Native desktop/mobile applications, task sharing and collaboration are outside the first release. No offline-operation guarantee is established. Detailed authentication, synchronisation mechanisms, recovery interactions, and browser/version compatibility belong to downstream Design and Test within these boundaries.

## Clarifications

### Session 2026-10-20 — surviving outcomes confirmed 2026-10-22

The outcomes below are retained from the draft insofar as consistent with the expressly authorised 2026-10-22 decisions. Their original respondent and approval provenance were not independently verified.

- A completed task can be reopened as active without changing its recorded task details.
- Due dates are **calendar dates**, without a time-of-day component.
- A user confirms deletion before a task is permanently removed; there is no in-application restore action.
- A past calendar date is a valid due date. An active task with a past due date is visibly overdue; a completed task is not overdue, but becomes overdue if reopened while its due date remains past.
- **Historical clarification (to be superseded upon formal AM-001 approval):** The most recently selected status and priority filters were restored between sessions. AM-001 proposes restoration of all four view settings only within a still-valid authenticated browser login, with defaults for each new login.
- The initial/default sort is by **earliest due date first**. **When sorting by Due Date**, tasks sharing the same due date are ordered by higher priority first; among tasks sharing a due date and priority, earlier-created tasks precede later-created tasks. Other tie cases belong to Functional Design.
- When previously saved tasks cannot load, show an error and allow retry; do not allow task-data changes that could overwrite or destroy existing saved data while loading remains unresolved. Persistent failures must direct the user to support.

### Session 2026-10-22 — authorised Scope decisions

- **DEC-R1-001; DEC-R1-001-P1:** Only the title is a mandatory user-entered field. Description, priority and due date are optional. If priority is omitted during creation or cleared during editing, its effective value becomes **Medium**.
- **DEC-R1-001-P2:** When sorting by due date, tasks with no due date appear last in **either direction**; no due date is invented for them.
- **DEC-R1-002; DEC-R1-003; DEC-R1-005:** The product is a web app for desktop and mobile browsers, using private accounts and sign-in; each account's personal tasks are accessible and synchronised across devices. Remote storage and transmission of those tasks are permitted; access by a different account is prohibited. Sharing/collaboration remain out of scope.
- **DEC-R1-011:** First-time users can self-register private accounts without requiring an invitation or administrator provisioning. The registration method and interaction details belong to Design.
- **DEC-R1-004-FINAL (supersedes DEC-R1-004):** On failure to load previously saved tasks, notify the user, offer retry, direct persistent failures to support and protect existing saved data from destructive/overwriting changes. No self-service saved-data reset is authorised.
- **DEC-R1-006; DEC-R1-006-P1/P2/P3 (historical baseline):** User-selectable sorting supports exactly **Due Date, Priority, Date Created, and Title**, with both directions for each. First-use default is **Due Date — earliest first**. Secondary ordering outside the retained same-due-date rule and sorting-preference restoration were left to Functional Design. AM-001 proposes to supersede **only** that restoration delegation: sort restoration follows FR-012; other tie cases stay with Functional Design.
- **DEC-R1-007-FINAL:** Remove unapproved numeric success thresholds SC-001–SC-003; retain the successfully-saved data persistence/cross-device availability guarantee in SC-004 without the original unsupported numeric claim.
- **DEC-R1-PHASE-001; DEC-R1-008/009/010 (historical baseline):** Detailed sorting tie cases, preference synchronisation, failed-write UI behaviour, overdue time-zone interpretation and test methodologies were assigned downstream. AM-001 proposes only the product-level filter/sort session lifetime and silent view-state failure outcome; concurrency, acknowledgement mechanics, implementation and test methodologies remain downstream.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create and View Tasks (Priority: P1)

As a signed-in individual, I want to add a task with only a title if I choose, or with additional details, and see it in my task list, so I can maintain a reliable personal record of work to do.

**Why this priority**: Capturing and viewing tasks is the minimum useful experience; the remaining workflows depend on a task existing.

**Independent Test**: Create a task using only a valid title, confirm it is active with Medium priority and no invented description or due date; separately create a task with optional details and verify that all supplied details appear.

**Acceptance Scenarios**:

1. **Given** an empty task list, **When** the signed-in user creates a task with a valid title alone, **Then** it is added as active with Medium priority, no description and no due date.
2. **Given** a task form, **When** the user provides a blank/whitespace-only title or an invalid non-empty due date, **Then** the task is not created and the relevant field is identified for correction.
3. **Given** a valid title and valid optional description, priority and/or calendar due date, **When** the user saves, **Then** the task retains and displays its supplied details and effective priority.

---

### User Story 2 - Maintain Task Status and Details (Priority: P2)

As an individual, I want to update, complete, reopen and delete my tasks, so my list accurately reflects my current commitments.

**Why this priority**: Task lists lose value unless people can keep details current and remove items they no longer need.

**Independent Test**: Create a task, change its details, mark it complete, reopen it, then delete it; verify each action produces the stated task-detail and list result.

**Acceptance Scenarios**:

1. **Given** an existing task, **When** the user saves valid edits to its title, optional description, priority or due date, **Then** the task displays the updated values; clearing priority makes it Medium, and clearing description or due date leaves those attributes absent.
2. **Given** an active task, **When** the user marks it complete, **Then** it is shown as completed and is not included in a view filtered to active tasks.
3. **Given** a completed task, **When** the user reopens it, **Then** it returns to active status with its task details unchanged.
4. **Given** a task, **When** the user confirms deletion, **Then** it is permanently removed from the task list and no restore action is offered.

---

### User Story 3 - Filter and Sort Tasks (Priority: P3)

As an individual, I want to filter and sort my tasks, so I can focus on relevant work and choose a useful ordering.

**Why this priority**: Filtering and sorting make a growing personal task list practical without changing the core ability to manage tasks.

**Independent Test**: Create tasks spanning the supported statuses, priorities, due dates and titles; verify individual and combined filters, all four sorting methods in both directions, initial default sorting and undated-last handling. Select all four view settings and refresh/open a new tab within the same valid login; verify restoration. With incomplete/unavailable view state, verify silent complete defaults without restricting available tasks.

**Acceptance Scenarios**:

1. **Given** tasks with active and completed statuses, **When** the user filters by a status, **Then** only tasks with that status are displayed.
2. **Given** tasks with Low, Medium and High priorities, **When** the user filters by priority, **Then** only tasks matching the selected priority are displayed.
3. **Given** active and completed tasks across multiple priorities, **When** the user applies both filters, **Then** only tasks matching both values are displayed.
4. **Given** a task list, **When** the user selects Due Date, Priority, Date Created or Title as the sort method and chooses a direction, **Then** matching tasks are displayed in that requested primary order.
5. **Given** tasks with and without due dates, **When** the user sorts by Due Date in either direction, **Then** undated tasks follow dated tasks.

---

### User Story 4 - Resume a Private Task List Across Devices (Priority: P1)

As an individual, I want to register my own private account and access my personal task list when I return or sign in from another device, so I can get started and continue working without recreating my tasks.

**Why this priority**: Private account access, cross-device synchronisation and persistence are approved product guarantees and essential to trust in the task manager.

**Independent Test**: Register a first-time private account without an invitation or administrator action, sign in to that account in desktop and mobile browsers, create and modify tasks, and verify that successfully saved task data is available on both after synchronisation. Verify each distinct authenticated session begins with its own default view and remembers only its own later selections. Sign in to a different account and verify the first user's tasks and settings are inaccessible.

**Acceptance Scenarios**:

1. **Given** successfully saved tasks, **When** their account owner signs in during a later session, **Then** the saved tasks, details and completion statuses are available.
2. **Given** successfully saved task changes in one signed-in browser, **When** the same account is used on another device and synchronisation completes, **Then** the changes are available there with the correct details and status.
3. **Given** a different signed-in account, **When** that user accesses the task manager, **Then** the first account's task list is not accessible.
4. **Given** a complete, valid four-setting view remembered in a still-valid authenticated browser login, **When** the user refreshes or opens another tab in that login, **Then** the status/priority filters and sort method/direction are restored and matching saved tasks appear in that order. A new login or distinct browser/profile/device session starts with all four defaults and remembers only its own later choices; saved tasks still synchronise by account. Already-open tabs need not update their view live.
5. **Given** a persistent failure to load existing saved tasks, **When** retry does not resolve it, **Then** the user is directed to support, and the failure does not permit changes that could overwrite or destroy the existing saved task data.
6. **Given** a first-time user without an account, **When** they register, **Then** they can obtain a private account and sign in without requiring an invitation or administrator provisioning.

### Edge Cases

- A title containing only whitespace is rejected as missing.
- An omitted description or due date remains absent; an omitted or cleared priority defaults to Medium.
- A supplied invalid calendar due date is rejected; a valid past date is retained and is overdue only while the task is active.
- A deleted task cannot be restored through the task manager.
- When no tasks match the selected filters, the user sees an empty-state message rather than stale results.
- Undated tasks sort last when Due Date is selected, whether earliest-first or latest-first.
- If view settings are absent, incomplete, invalid or unreadable, show the **complete** default view (Status All, Priority All, Due Date earliest first) without a notification, provided authentication and task loading succeed. A failed/unconfirmed attempt to remember a new view leaves the current tab's selection usable without warning.
- If previously saved task data cannot be loaded, the user is told the task list is unavailable, can retry, is directed to support for persistent failures and cannot make task-data changes that could overwrite or destroy existing saved data while loading remains unresolved. No self-service data reset is provided.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow a signed-in user to create an active task with a non-empty, non-whitespace title; description, priority and calendar-date due date MUST be optional. A supplied due date MUST be a valid calendar date, with no time component.
- **FR-002**: The system MUST support exactly three effective task priorities (Low, Medium and High) and MUST assign Medium when priority is omitted on creation or cleared during editing.
- **FR-003**: The system MUST display each task's title, effective priority and completion status, and display its description and calendar-date due date when provided, in the task list or task detail view. It MUST NOT fabricate a description or due date when absent.
- **FR-004**: The system MUST permit editing an existing task's title, description, priority and due date using the creation validation rules; the description and due date may be cleared, and clearing priority MUST restore the Medium default.
- **FR-005**: The system MUST let a user mark an active task as complete, reopen a completed task as active while preserving its task details, and distinguish completed tasks from active tasks.
- **FR-006**: The system MUST permanently delete a task only after the user explicitly confirms deletion; a deleted task cannot be restored through the task manager.
- **FR-007**: The system MUST support combined filtering of the displayed task list by status (All, Active or Completed) and priority (All, Low, Medium or High).
- **FR-008**: The system MUST preserve successfully saved task creation, edits, status changes and deletions across sessions and synchronise the same account's task data across desktop and mobile browsers/devices. Successfully saved changes MUST become available to that account on other devices after synchronisation.
- **FR-009**: The system MUST provide clear feedback when task creation, editing, completion/reopening, deletion, synchronisation or saved-data loading cannot be completed.
- **FR-010**: The system MUST provide private user accounts and sign-in. Signing in to the same account on different browsers or devices MUST provide access to that account's personal task list; task data MUST NOT be accessible through another account. Task-list sharing and collaboration are out of scope.
- **FR-011**: The system MUST accept valid past calendar-date due dates and visibly identify active tasks with dates before the current calendar date as overdue; completed tasks MUST NOT appear overdue, but MUST become overdue when reopened if their due dates remain past. Tasks without a due date MUST NOT appear overdue.
- **FR-012**: The system MUST apply the user's selected status filter, priority filter, sort method and direction to the displayed task list, and MUST remember this complete four-setting view within the current authenticated browser login when view-state persistence succeeds. Refreshing or opening a new tab during that still-valid login MUST restore the latest complete, valid view remembered in that session; already-open tabs need not update live. Each new login, including after logout/expiry or in another browser, browser profile or device, MUST start with Status All, Priority All and Due Date earliest first. Another still-valid session MUST retain only its own remembered view and never inherit another session's settings. If remembering settings fails or cannot be confirmed, the current tab's selected view remains usable without a view-setting warning; a later load uses any complete valid remembered view available or, otherwise, all four defaults. If any remembered component is absent, incomplete, invalid or unreadable, the entire view MUST default silently. View-state failure alone MUST NOT block otherwise successfully loaded tasks or task actions. Authentication/ownership constraints remain mandatory, and failure to load task data MUST continue to obey FR-014.
- **FR-013**: The system MUST let users sort the filtered task list by **Due Date, Priority, Date Created or Title**, each in both directions (Due Date: earliest/latest; Priority: High–Low/Low–High; Date Created: newest/oldest; Title: A–Z/Z–A). On first use of each newly authenticated session, the default MUST be Due Date, earliest first. In either Due Date direction, tasks with no due date MUST appear last. **When the selected sort method is Due Date**, tasks sharing a due date MUST be ordered by higher priority first, then earlier-created tasks for identical priority; remaining tie cases are owned by Functional Design. Sort method/direction restoration follows FR-012.
- **FR-014**: If previously saved tasks cannot be loaded, the system MUST inform the user, provide a retry opportunity, direct the user to support if the failure persists and prevent task-data changes that could overwrite or destroy existing saved task data while the loading failure remains unresolved. A user-facing saved-data reset feature is not included.
- **FR-015**: The first release MUST be accessible as a web application through desktop and mobile browsers. Native desktop and mobile applications are excluded from this release; offline operation is not a required guarantee.
- **FR-016**: The system MUST permit first-time users to self-register a private account without invitation or administrator provisioning, so they can sign in and access their personal task list.

### Key Entities *(include if feature involves data)*

- **User Account**: A private identity that a first-time user can self-register without invitation or administrator provisioning, used for sign-in, owning one personal task list and authorising access to that list across browsers and devices.
- **Task**: A personal work item belonging to one user account, with a stable identity, required title, optional description and calendar-date due date, one effective priority (Low/Medium/High), completion status and recorded creation order/date for sorting.
- **Task Filter**: Selected status and priority criteria determining which account-owned tasks appear; filtering does not change the tasks. Selection is part of an authenticated-session-scoped view, not an account-persistent task record.
- **Task Sorting**: The selected sorting method and direction for displaying filtered tasks; sorting does not change the tasks. Selection shares the same session-scoped view lifetime and complete-default fallback as filters.

## Acceptance Criteria

- **AC-001**: Given a completed task, when the user reopens it, then it appears as active with its title and any provided description, priority and due date unchanged.
- **AC-002**: Given a valid title and no other values, when the signed-in user saves, then an active task is created with Medium priority and no due date or description; if the user supplies a valid calendar due date without a time of day, that date is retained.
- **AC-003**: Given a task, when the user confirms deletion, then it disappears from every task view and no restore action is available.
- **AC-004**: Given an active task whose due date precedes the current calendar date, when the user views it, then it is visibly overdue; once completed, the overdue label is absent, and it returns if the task is reopened while the date remains past. A task with no due date is not overdue.
- **AC-005**: Given a complete valid four-setting view remembered in the current authenticated browser login, when the user refreshes or opens another tab in that still-valid login, then the status filter, priority filter, sort method and direction are restored and matching tasks appear in that order. Already-open tabs need not update live. Given a new login or a distinct browser/profile/device session, the initial view is All/All/Due Date earliest first, independently of other sessions' remembered views. Given absent, incomplete, invalid or unreadable remembered settings while authentication and task loading succeed, all four settings reset to defaults without warning and the task list and controls remain usable. Given an unconfirmed view save, the current tab may retain its selected view without warning, but that unconfirmed choice is not guaranteed to restore. Task-data load failure retains AC-008 protection.
- **AC-006**: Given tasks due on 2026-10-19, 2026-10-21, and 2026-10-30, when first-use default sorting is applied, then they appear in earliest-to-latest order; undated tasks appear afterwards. When Due Date sorting is reversed, the dated tasks reverse but undated tasks remain last.
- **AC-007**: Given **Due Date** sorting is selected and multiple tasks share a due date, a higher-priority task appears before lower-priority tasks; among tasks sharing that date and priority, the earlier-created task appears first. Other tied-value behaviour is defined during Functional Design.
- **AC-008**: Given saved tasks that fail to load, when the account owner opens the app, then an error and retry opportunity are presented; if failure persists, support guidance is available and changes capable of overwriting or destroying existing saved data are blocked. No self-service reset is offered.
- **AC-009**: Given an account that has saved tasks on a desktop browser, when the same user signs in on a mobile browser and synchronisation completes, then those tasks and their successfully saved changes are available with correct details and status.
- **AC-010**: Given two distinct signed-in accounts, when either user accesses the app, then neither can view or modify the other's personal tasks.
- **AC-011**: Given multiple tasks, when a user selects any of the four supported sorting methods in either direction, then the resulting primary order matches that selection, subject to the approved due-date handling.
- **AC-012**: Given a first-time user without an account, when they register, then they can obtain a private personal account and sign in without an invitation or administrator provisioning.

## Success Criteria *(mandatory)*

### Observable Outcomes

- **SC-004**: Successfully saved task changes remain available with their correct details and statuses across later sessions and, after synchronisation, across browsers and devices signed in to the same account.

The draft numerical targets previously labelled SC-001 (60-second task creation), SC-002 (2-second filtering for 95% of attempts) and SC-003 (90% unaided usability) were **not approved** and are removed from the active success criteria under DEC-R1-007-FINAL. Their history is retained in `scope_review_report.md`. Detailed verification methods are owned by Test.

## Assumptions and Exclusions

- First-time users can self-register private accounts without invitations or administrator provisioning. Each user signs in to access their personal task list; account-specific tasks persist and synchronise across signed-in browsers and devices. Remote storage and transmission are permitted, but cross-user access and task sharing/collaboration are excluded.
- This release supports desktop and mobile browsers as a web application, not native desktop or mobile apps. No offline-operation guarantee is specified.
- The title is the only mandatory user-entered task attribute. An absent description or due date stays absent; an omitted or cleared priority defaults to Medium.
- Due dates are calendar dates with no time of day. Past due dates are allowed, and only active tasks with passed due dates are overdue.
- Newly created tasks are active; completed tasks may be reopened without changing their details. Deletion requires confirmation and offers no in-application restore.
- Each new authenticated browser login begins with Status All and Priority All filters; valid settings chosen later in that login are remembered across refresh/new tabs. Separate valid sessions (including another browser/profile/device) keep independent views; no cross-device view-preference persistence or live update of already-open tabs is promised.
- An authenticated browser login may be shared by tabs using the same session cookie. Refreshing a tab, opening or closing a tab, or restarting the browser does not itself define a new login; the view's lifetime follows the validity of authentication. No new login-expiry or remember-me policy is introduced.
- The initial sort for each new authenticated session is Due Date (earliest first), with undated tasks last for either due-date direction. Users may choose the four approved sorting methods and reverse each; method/direction are remembered with filters under FR-012, and additional tie-breaking remains with Functional Design. Failed/unavailable view-state persistence silently falls back to a valid remembered view or complete defaults on the next load without affecting saved tasks.
- If previously saved tasks fail to load, the app provides error/retry/support guidance while protecting saved data against overwriting or destruction. The earlier proposed self-service data reset is superseded and excluded.
