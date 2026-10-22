# Feature Specification: Personal Task Manager

**Feature Branch**: `001-personal-task-manager`

**Created**: 2026-10-20

**Status**: Draft — latest Scope change reconciled; pending final human Scope sign-off

**Input**: User description: "Build a simple personal task manager."

## Clarifications

### Session 2026-10-20

- Q: After a task is marked complete, can the user return it to active status? → A: A completed
  task can be reopened and returned to active status while preserving its details.
- Q: Does a task’s due date represent only a calendar day, or must it support a specific time of
  day as well? → A: A task due date is a calendar date only; it has no time-of-day component.
- Q: What should happen when the user deletes a task? → A: The user confirms deletion, after which
  the task is permanently removed and cannot be restored.
- Q: May a user create or edit a task with a due date that has already passed? → A: Past calendar
  dates are valid; the task remains available and is visibly identified as overdue.
- Q: When the user returns in a new session, should their most recently selected status and priority
  filters be restored? → A: Restore the last selected status and priority filters.
- Q: In the absence of a user-selected sort option, how should matching tasks be ordered? → A:
  Display matching tasks by earliest calendar-date due date first.
- Q: When two or more matching tasks have the same due date, which order should the list use between
  them? → A: Order same-date tasks by higher priority first; for the same priority, order the
  earlier-created task first.
- Q: If the task manager cannot read previously saved tasks when a session begins, what may the user
  do next? → A: Show an error with a retry option; do not allow task-data changes or overwrite
  unreadable saved data until the prior data loads successfully.
- Q: If a task was completed after its due date, should it still be visibly labelled overdue? → A:
  Only active tasks with past due dates are overdue; reopening one makes it overdue again if its due
  date remains past.

### Session 2026-10-21 — Independent Scope Review Decisions

The following decisions were expressly approved by the product owner during independent Scope review; they supersede any inconsistent earlier assumptions or draft requirements. Review finding IDs are retained for traceability.

- **REV-001 (approved 2026-10-21; partially superseded by REV-014)**: Originally required title, description, and priority; due date optional. REV-014 subsequently makes description optional. Title and priority remain mandatory.
- **REV-002 (approved)**: Successfully saved task data persists across sessions only within the same browser profile on the same device. Other browser profiles and browsers have separate task lists; cross-device synchronization is out of scope.
- **REV-006 (approved)**: Task contents remain local to the browser profile and are not transmitted to external services.
- **REV-010 (approved)**: Dated tasks appear before undated tasks. Dated tasks are ordered by earliest due date; ties use higher priority and then earlier creation. Undated tasks are ordered by higher priority, then earlier creation.
- **REV-005 (approved; interpreted with REV-014)**: Title must contain at least one non-whitespace character. An optional description may be omitted or empty; if supplied as nonempty text, it must contain at least one non-whitespace character. Whitespace-only descriptions remain invalid.
- **REV-007 (approved)**: An active task is overdue only when its due date is earlier than the device's current local calendar date.
- **REV-011 (approved; ID assigned during reconciliation)**: Failed task changes must not corrupt previously saved task data or be reported as successfully saved. Detailed recovery interaction and implementation are Design concerns.

### Session 2026-10-21 — Subsequent Scope Change

- **REV-014 (approved)**: Description is optional on task creation and editing. Title and priority remain mandatory; due date remains optional. This supersedes only the description-required portion of REV-001. The REV-005 non-whitespace rule remains applicable to mandatory titles and to descriptions containing entered text; an omitted or empty description is valid.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create and View Tasks (Priority: P1)

As an individual, I want to add a task with its essential details and see it in my task list, so I
can maintain a reliable personal record of work to do.

**Why this priority**: Capturing and viewing tasks is the minimum useful experience; the remaining
workflows depend on a task existing.

**Independent Test**: Create a task with a valid title and priority but no description or due date;
create another with optional description and due date. Verify that both appear in the active-task
list and that optional details are displayed only when provided.

**Acceptance Scenarios**:

1. **Given** an empty task list, **When** the user submits a valid title and priority without a
   description or due date, **Then** the task is added as active with those required details.
2. **Given** an empty task list, **When** the user provides a valid title and priority with a
   description and/or a valid calendar-date due date, **Then** the task is added as active and the
   supplied optional details are available.
3. **Given** a task form, **When** the user omits a mandatory field, enters a whitespace-only
   title or nonempty whitespace-only description, or supplies an invalid due date, **Then** the
   task is not created and the relevant field is identified for correction.

---

### User Story 2 - Maintain Task Status and Details (Priority: P2)

As an individual, I want to update, complete, and delete tasks, so my list accurately reflects my
current commitments.

**Why this priority**: Task lists lose value unless people can keep details current and remove
items they no longer need.

**Independent Test**: Create a task, change its details, mark it complete, then delete it; verify
each action produces the stated list and task-detail result.

**Acceptance Scenarios**:

1. **Given** an existing task, **When** the user saves valid edits to its title or priority,
   adds, changes, or removes an optional description, or adds, changes, or removes an optional
   due date, **Then** the task shows the updated values.
2. **Given** an active task, **When** the user marks it complete, **Then** it is shown as completed
   and is no longer included in the active-task view.
3. **Given** a completed task, **When** the user reopens it, **Then** it returns to active status
   with its title, description, priority, and optional due date unchanged.
4. **Given** a task, **When** the user confirms deletion, **Then** it is permanently removed and no
   longer appears in any task view.

---

### User Story 3 - Filter Task List (Priority: P3)

As an individual, I want to filter tasks by completion status and priority, so I can focus on the
most relevant work.

**Why this priority**: Filtering makes a growing personal task list practical without changing the
core ability to manage tasks.

**Independent Test**: Create tasks covering each status and priority, apply each filter and a
combined status-and-priority filter, and verify only matching tasks are displayed.

**Acceptance Scenarios**:

1. **Given** tasks with active and completed statuses, **When** the user filters by a status,
   **Then** only tasks with that status are displayed.
2. **Given** tasks with low, medium, and high priorities, **When** the user filters by a priority,
   **Then** only tasks with that priority are displayed.
3. **Given** active and completed tasks across multiple priorities, **When** the user applies both
   filters, **Then** only tasks matching both selected values are displayed.

---

### User Story 4 - Resume a Saved Task List (Priority: P1)

As an individual, I want my task list to remain available after I leave and return, so I do not
need to recreate my work.

**Why this priority**: Persistence is an explicit requirement and essential to trust in the task
manager.

**Independent Test**: Create and modify tasks, leave the application, return to it in the same
browser profile on the same device, and verify task details and completion statuses are preserved.
Open another browser profile and verify that it has a separate task list.

**Acceptance Scenarios**:

1. **Given** one or more saved tasks, **When** the user starts a new session in the same browser
   profile on the same device, **Then** those tasks, details, and completion statuses are restored.
2. **Given** saved tasks and selected filters, **When** the user returns in that browser profile,
   **Then** the selected filters are restored, matching saved tasks are displayed, and all saved
   tasks remain available when the filters are changed.
3. **Given** tasks saved in one browser profile, **When** the user opens the application in another
   browser or browser profile, **Then** the first profile's tasks are not available there and the
   task lists are separate.

### Edge Cases

- A title must contain at least one non-whitespace character. Description may be absent or empty;
  a nonempty description consisting solely of whitespace is invalid.
- Omitting a due date is valid; a task without a due date is never overdue.
- A supplied due date in the past is retained and does not prevent task creation or editing; it
  is visibly identified as overdue only while the task is active, relative to the device's current
  local calendar date.
- A deleted task cannot be recovered through the task manager.
- When no tasks match the selected filters, the user sees an empty-state message rather than stale
  results.
- If saved task data cannot be loaded, the user is informed that their saved tasks are unavailable,
  can retry loading, and cannot change task data until the prior data loads successfully.
- A failed task-data change must not corrupt previously saved task data or be reported as saved.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST let a user create an active task after providing a title containing
  at least one non-whitespace character and a valid priority. Description and due date are
  optional. An omitted or empty description is valid; a nonempty description MUST contain at
  least one non-whitespace character. A supplied due date MUST be a valid calendar date without
  a time-of-day component.
- **FR-002**: The system MUST support exactly three task priorities: low, medium, and high.
- **FR-003**: The system MUST display each task's title, priority, completion status, and any
  supplied description or due date in the task list or task detail view.
- **FR-004**: The system MUST let a user edit the title and priority of an existing task and
  add, change, or remove its optional description or due date, subject to the same validation
  used for creation.
- **FR-005**: The system MUST let a user mark an active task as complete, reopen a completed task
  as active while preserving its details, and distinguish completed tasks from active tasks.
- **FR-006**: The system MUST let a user permanently delete a task only after an explicit
  confirmation; a deleted task cannot be restored.
- **FR-007**: The system MUST let a user filter the displayed task list by status (all, active,
  or completed) and by priority (all, low, medium, or high), applying both filters together.
- **FR-008**: The system MUST preserve all successfully saved task changes, including creations,
  edits, status changes, and deletions, across sessions within the same browser profile on the
  same device. Different browsers and browser profiles MUST maintain separate task lists;
  cross-browser-profile and cross-device synchronization are out of scope.
- **FR-009**: The system MUST provide clear feedback when a create, edit, completion, deletion, or
  saved-data loading action cannot be completed.
- **FR-010**: The system MUST operate without sign-in, accounts, shared task lists, or any other
  multi-user capability. Task-list access and persistence are bounded by the local browser
  profile as specified in FR-008.
- **FR-011**: The system MUST accept a past calendar-date due date. An active task MUST be visibly
  identified as overdue only when its due date is earlier than the device's current local
  calendar date. A completed task and a task without a due date MUST NOT be labelled overdue;
  a completed task that is reopened becomes overdue again if its due date remains past.
- **FR-012**: The system MUST preserve the most recently selected status and priority filters
  between sessions within the same browser profile and apply both restored filters when
  displaying that profile's task list.
- **FR-013**: The system MUST display matching tasks with due dates first, by ascending
  calendar-date due date, then by priority from high to low for the same date, then by earlier
  creation time for the same date and priority. Tasks without due dates MUST follow all dated
  tasks, ordered by priority from high to low, then by earlier creation time for equal priority.
- **FR-014**: If saved task data cannot be loaded, the system MUST show an error and a retry action,
  prevent all task-data changes until loading succeeds, and never overwrite unreadable saved data.
- **FR-015**: The system MUST keep task data, including task contents and details, local to the
  browser profile. Task data MUST NOT be transmitted to external services.
- **FR-016**: If any task-data change fails, the system MUST NOT corrupt previously saved task data
  and MUST NOT report the failed change as successfully saved.

### Key Entities *(include if feature involves data)*

- **Task**: A personal work item with a required title and priority, optional description and
  calendar-date due date, completion status, and a stable identity used to update or delete it.
- **Task Filter**: The selected status and priority criteria that determine which tasks are shown;
  it does not alter the underlying tasks.

## Acceptance Criteria

- **AC-001**: Given a completed task with recorded details, when the user reopens it, then it is
  visible in the active-task view with the same title, description, priority, and due date (if
  any) (Round 1, Q1).
- **AC-002**: Given a task form, when the user enters a calendar date without a time of day and
  saves valid remaining details, then the task is created with that calendar date as its due date
  (Round 2, Q1).
- **AC-003**: Given a task, when the user confirms deletion, then it is absent from every task view
  and no restore action is available (Round 3, Q1).
- **AC-004**: Given an active task whose due date is earlier than the device's current local
  calendar date, when the user views it, then the task remains available and is visibly identified
  as overdue; after completion the overdue label is absent, and it returns if the task is reopened.
  A task without a due date is never overdue (Rounds 4 and 9, Q1; REV-007).
- **AC-005**: Given selected status and priority filters, when the user starts a new session in
  the same browser profile, then the same filters are applied and only matching tasks are
  displayed (Round 5, Q1; REV-002).
- **AC-006**: Given matching tasks due on 2026-10-19, 2026-10-21, and 2026-10-30, when the user
  views the task list, then they appear in that earliest-to-latest order (Round 6, Q1).
- **AC-007**: Given two tasks due on 2026-10-21 and both at medium priority, when one was created
  earlier, then it appears first; a high-priority task due on 2026-10-21 appears before both
  medium-priority tasks (Round 7, Q1).
- **AC-008**: Given unreadable saved task data, when the user opens the task manager, then an error
  and retry action are displayed and task-data changes remain unavailable until loading succeeds
  (Round 8, Q1).
- **AC-009**: Given a valid title and priority, when the user creates a task without a
  description or due date, then creation succeeds and the task is active. Given an existing
  task with a due date, when the user removes the date and saves valid edits, then the task
  has no due date (REV-001; REV-014).
- **AC-010**: Given a creation or editing form, when title contains only whitespace, or an
  entered nonempty description contains only whitespace, then saving is rejected and the
  relevant field is identified. An omitted or empty description is valid (REV-005; REV-014).
- **AC-011**: Given saved task data in one browser profile, when the user returns in the same
  profile on the same device, then it remains available; when the application is opened in a
  different browser or profile, the first profile's tasks are not accessible there (REV-002).
- **AC-012**: Given tasks with and without due dates, when the list is displayed, then every dated
  task appears before every undated task; dated tasks follow due date, priority, and creation
  order, while undated tasks follow priority and creation order (REV-010).
- **AC-013**: Given a task with a title, description, priority, or optional due date, when the user
  uses the application, task data remains local to that browser profile and is not transmitted
  to external services (REV-006).
- **AC-014**: Given previously saved tasks and a subsequent task-data change that fails, when
  the system reports the outcome and the saved tasks are subsequently read, then the failed
  change was not presented as successfully saved and the previously saved task data is not
  corrupted (REV-011).
- **AC-015**: Given a task with a description, when the user removes its description and saves
  the task with a valid title and priority, then the change succeeds and the task has no
  description. Given a task without a description, when the user adds valid descriptive text
  and saves, that description is retained (REV-014).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can create a fully detailed task in 60 seconds or less in a usability test.
- **SC-002**: In a test list of 1,000 tasks, applying a status or priority filter updates the
  displayed results within 2 seconds for at least 95% of attempts.
- **SC-003**: At least 90% of usability-test participants can create, edit, complete, delete, and
  filter a task without assistance on their first attempt.
- **SC-004**: After ending and resuming a session within the same browser profile on the same
  device, 100% of successfully saved task changes in the acceptance test set are available with
  the correct details and status.

## Provenance and Assumptions

### Approved decisions and constraints (not assumptions)

- **REV-001 / REV-005 / REV-014**: Title and priority are mandatory; description and due date
  are optional. Title must contain at least one non-whitespace character; an omitted or empty
  description is valid, but a nonempty description must contain at least one non-whitespace
  character. REV-014 supersedes REV-001 only as to description being mandatory.
- **REV-002 / REV-006**: Task data remains local to each browser profile. Browsers and profiles
  have separate task lists; cross-device synchronization and external task-data transmission
  are excluded. No accounts or sharing are included in the draft product scope.
- **REV-007**: Overdue status is based on the device's current local calendar date.
- **REV-010**: Dated tasks precede undated tasks, with the defined priority/creation tie-breaks.
- **REV-011**: Failed task-data changes must not corrupt previously saved task data or be
  reported as successfully saved.
- **2026-10-20 clarifications**: Supplied due dates represent calendar days, completed tasks
  may be reopened preserving details, and confirmed deletion is irreversible.

### Working assumptions and draft baseline (subject to Scope sign-off)

- The task list initially shows all tasks; after either filter is changed, selections are
  restored in later sessions within that browser profile.
- Initial task status, three priority levels, baseline feature selection, and measurable
  success targets are specified in the draft requirements. Where they do not trace to an
  explicit clarification, their acceptance remains subject to final human Scope sign-off.

Detailed failure/recovery interactions and implementation choices remain for Design. The
outstanding downstream review findings and resolution gates are tracked in the separate
Scope Review Report. Human approval of this reconciled specification has not yet been given.
