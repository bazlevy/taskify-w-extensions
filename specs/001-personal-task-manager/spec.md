# Feature Specification: Personal Task Manager

**Feature Branch**: `001-personal-task-manager`

**Created**: 2026-10-20

**Status**: Draft

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

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create and View Tasks (Priority: P1)

As an individual, I want to add a task with its essential details and see it in my task list, so I
can maintain a reliable personal record of work to do.

**Why this priority**: Capturing and viewing tasks is the minimum useful experience; the remaining
workflows depend on a task existing.

**Independent Test**: Create a task with a title, description, priority, and due date; verify that
the task appears in the active-task list with all supplied details.

**Acceptance Scenarios**:

1. **Given** an empty task list, **When** the user submits a task with all required fields,
   **Then** the task is added as active and its title, description, priority, and due date are
   displayed.
2. **Given** a task form, **When** the user leaves any required field empty or enters an invalid
   due date, **Then** the task is not created and the relevant field is identified for correction.

---

### User Story 2 - Maintain Task Status and Details (Priority: P2)

As an individual, I want to update, complete, and delete tasks, so my list accurately reflects my
current commitments.

**Why this priority**: Task lists lose value unless people can keep details current and remove
items they no longer need.

**Independent Test**: Create a task, change its details, mark it complete, then delete it; verify
each action produces the stated list and task-detail result.

**Acceptance Scenarios**:

1. **Given** an active task, **When** the user saves edits to its title, description, priority, or
   due date, **Then** the task shows the updated values.
2. **Given** an active task, **When** the user marks it complete, **Then** it is shown as completed
   and is no longer included in the active-task view.
3. **Given** a completed task, **When** the user reopens it, **Then** it returns to active status
   with its title, description, priority, and due date unchanged.
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

**Independent Test**: Create and modify tasks, leave the application, return to it, and verify the
same task details and completion statuses are available.

**Acceptance Scenarios**:

1. **Given** one or more saved tasks, **When** the user starts a new session, **Then** the saved
   tasks, details, and completion statuses are restored.
2. **Given** saved tasks and selected filters, **When** the user returns in a new session,
   **Then** the selected filters are restored, matching saved tasks are displayed, and all saved
   tasks remain available when the filters are changed.

### Edge Cases

- A title consisting only of whitespace is rejected as missing.
- A due date in the past is retained and does not prevent task creation or editing; it is visibly
  identified as overdue only while the task is active.
- A deleted task cannot be recovered through the task manager.
- When no tasks match the selected filters, the user sees an empty-state message rather than stale
  results.
- If saved task data cannot be loaded, the user is informed that their saved tasks are unavailable,
  can retry loading, and cannot change task data until the prior data loads successfully.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST let a user create an active task only after providing a non-empty
  title, description, priority, and valid calendar-date due date.
- **FR-002**: The system MUST support exactly three task priorities: low, medium, and high.
- **FR-003**: The system MUST display each task's title, description, priority, calendar-date due
  date, and completion status in the task list or task detail view.
- **FR-004**: The system MUST let a user edit the title, description, priority, and due date of an
  existing task, subject to the same validation used for creation.
- **FR-005**: The system MUST let a user mark an active task as complete, reopen a completed task
  as active while preserving its details, and distinguish completed tasks from active tasks.
- **FR-006**: The system MUST let a user permanently delete a task only after an explicit
  confirmation; a deleted task cannot be restored.
- **FR-007**: The system MUST let a user filter the displayed task list by status (all, active,
  or completed) and by priority (all, low, medium, or high), applying both filters together.
- **FR-008**: The system MUST preserve created, edited, completed, and deleted task state between
  sessions for the same user on the same device.
- **FR-009**: The system MUST provide clear feedback when a create, edit, completion, deletion, or
  saved-data loading action cannot be completed.
- **FR-010**: The system MUST operate without sign-in, accounts, shared task lists, or any other
  multi-user capability.
- **FR-011**: The system MUST accept a past calendar-date due date and visibly identify an active
  task with that date as overdue; a completed task is not overdue, but becomes overdue if reopened
  while its due date remains past.
- **FR-012**: The system MUST preserve the user's most recently selected status and priority filters
  between sessions and apply both restored filters when displaying the task list.
- **FR-013**: The system MUST display matching tasks by ascending calendar-date due date, then by
  priority from high to low for the same date, and then by earlier creation time for the same date
  and priority.
- **FR-014**: If saved task data cannot be loaded, the system MUST show an error and a retry action,
  prevent all task-data changes until loading succeeds, and never overwrite unreadable saved data.

### Key Entities *(include if feature involves data)*

- **Task**: A personal work item with a title, description, priority, due date, completion status,
  and a stable identity used to update or delete it.
- **Task Filter**: The selected status and priority criteria that determine which tasks are shown;
  it does not alter the underlying tasks.

## Acceptance Criteria

- **AC-001**: Given a completed task with recorded details, when the user reopens it, then it is
  visible in the active-task view with the same title, description, priority, and due date (Round
  1, Q1).
- **AC-002**: Given a task form, when the user enters a calendar date without a time of day and
  saves valid remaining details, then the task is created with that calendar date as its due date
  (Round 2, Q1).
- **AC-003**: Given a task, when the user confirms deletion, then it is absent from every task view
  and no restore action is available (Round 3, Q1).
- **AC-004**: Given an active task whose due date is before the current calendar date, when the user
  views it, then the task remains available and is visibly identified as overdue; after it is
  completed, the overdue label is absent, and it returns if the task is reopened (Rounds 4 and 9,
  Q1).
- **AC-005**: Given selected status and priority filters, when the user starts a new session, then
  the same filters are applied and only matching tasks are displayed (Round 5, Q1).
- **AC-006**: Given matching tasks due on 2026-10-19, 2026-10-21, and 2026-10-30, when the user
  views the task list, then they appear in that earliest-to-latest order (Round 6, Q1).
- **AC-007**: Given two tasks due on 2026-10-21 and both at medium priority, when one was created
  earlier, then it appears first; a high-priority task due on 2026-10-21 appears before both
  medium-priority tasks (Round 7, Q1).
- **AC-008**: Given unreadable saved task data, when the user opens the task manager, then an error
  and retry action are displayed and task-data changes remain unavailable until loading succeeds
  (Round 8, Q1).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can create a fully detailed task in 60 seconds or less in a usability test.
- **SC-002**: In a test list of 1,000 tasks, applying a status or priority filter updates the
  displayed results within 2 seconds for at least 95% of attempts.
- **SC-003**: At least 90% of usability-test participants can create, edit, complete, delete, and
  filter a task without assistance on their first attempt.
- **SC-004**: After ending and resuming a session, 100% of successfully saved task changes in the
  acceptance test set are available with the correct details and status.

## Assumptions

- The first release serves one person using one device and one local task list; task sharing,
  synchronization, and accounts are out of scope.
- Title, description, priority, and due date are all required because each is described as a task
  attribute in the feature request.
- A due date records a calendar date, not a time of day.
- Tasks are initially active; a completed task can be reopened as active without changing its other
  details.
- The task list initially shows all tasks; after the user changes either filter, the selected filters
  are restored in later sessions.
