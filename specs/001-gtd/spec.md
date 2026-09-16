# Feature Specification: GTD Task Assistant

**Feature Branch**: `gtd`

**Created**: 2026-09-15

**Status**: Draft

**Input**: Build tools based on the GTD philosophy that help Android users to manage their tasks.
Tasks could bge in various context :
+ Professional, a jira task todo / a prospect to call again / a hiring process to continue
+ Family, through the garbage / clean the table / make a laundry
+ Friend, get in touch with john / refund this debt I have from jane /

## Clarifications

### Session 2026-09-15

- Q: For this first feature, where should personal task data live and how should access work? → A:
  Device-local TaskChampion via Hyper API. Updated on 2026-09-16 to also support remote data for
  user-triggered, two-way synchronization across devices.
- Q: How should users unlock access to tasks stored locally on their device? → A: Online Tasky
  account authenticated by the API through a JWT-based security layer.
- Q: How long may a previously authenticated user continue accessing local tasks without contacting
  the account API? → A: Every application launch requires online authentication; the backend is
  stateless and retains no session.
- Q: Which transport-security profile should protect account API calls from man-in-the-middle attacks?
  → A: TLS 1.3 with strict certificate and hostname validation, cleartext disabled, and DPoP-bound JWTs
  signed by a non-exportable device key held in Android Keystore.
- Q: How should a user authorize registration of a new device-held DPoP key during onboarding or device
  replacement? → A: Password plus SMS OTP for this release; authenticator-app TOTP is deferred to the
  next professional-use iteration.
- Q: What reliability target should the account API meet when every application launch depends on it?
  → A: No formal availability or authentication-speed target for the first release.
- Q: How should a task move between Inbox, Active, Waiting, Scheduled, Someday, and Completed states?
  → A: Replace those states with exactly New for a created task, In Progress while it is being done,
  and Done when it is finished. Transitions are only New → In Progress → Done; a Done task cannot be
  reopened but can be copied into a separate New task.
- Q: Which fields should a Done task copy into the new standalone task? → A: Copy every user-editable
  field, including future due dates and reminders; assign a new identity and history, and set only the
  state to New.
- Q: What should happen when a copied task contains a reminder time that has already passed? → A: Past
  due dates and expired reminders are not copied; the user is expected to set fresh scheduling values.
- Q: What should happen to device-local tasks when a user deletes their Tasky account? → A: Delete the
  account and soft-delete its task data. Recovery, sharing, export, retention duration, and permanent
  erasure are deferred to a future professional feature.
- Q: Which accessibility baseline must every primary task flow meet? → A: Formal accessibility
  requirements are deferred to a future release.
- Q: How should the account API respond to repeated failed password sign-in attempts? → A: Password
  attempt throttling is deferred for this release.
- Q: May a configured reminder repeat automatically until its task is completed? → A: A reminder may
  repeat daily until the task is Done or the user stops it.
- Q: When the device's time zone or daylight-saving offset changes, when should a reminder next appear?
  → A: Keep using the time zone in which the reminder was created.
- Q: Which interface languages must the first release support? → A: French only.
- Q: How should the constitution govern password-attempt throttling? → A: Throttling is recommended,
  but each feature may defer it.
- Q: What should happen when the JWT expires while the application remains open? → A: Preserve unsaved
  input, lock task operations, and require the user to enter the account password again.
- Q: How may a user reset a forgotten account password? → A: Forgotten-password recovery is not
  available in the first release.
- Q: What password rules must Tasky enforce when an account is created or its password is changed? → A:
  The first release defines no password policy.
- Q: How should Tasky create an account and register its first device key? → A: Create a disabled account
  before SMS verification, then activate it and bind the first device key after the OTP succeeds.
- Q: What should happen to a disabled pending account whose telephone number is never verified? → A:
  The first release defines no cleanup rule.
- Q: How many reusable contexts may be assigned to one task? → A: A task may have zero or more
  contexts.
- Q: What does a task's due date represent, and when does the task become overdue? → A: A due date has
  an optional local time; a timed task becomes overdue after that time, while a date-only task becomes
  overdue at the start of the following local day.
- Q: Which time zone should determine when a task with a due time becomes overdue? → A: Keep the time
  zone in which the due time was created and follow that zone's current daylight-saving rules.
- Q: How should snoozing affect one-time and daily reminders? → A: Remove snoozing from the first
  release.
- Q: What should happen after a task has remained in the recoverable deletion area for 30 days? → A:
  The first release defines no post-30-day disposition behavior.

### Session 2026-09-16

- Q: When a user taps “Complete” on a New task, what should happen? → A: Automatically move through
  In Progress to Done, recording both transitions.

- Q: When someone taps “Complete” on a reminder while Tasky is closed or authentication has expired,
  how should Tasky proceed? → A: Request authentication, then complete automatically after success;
  leave the task unchanged if authentication fails or is cancelled.

- Q: When a user filters tasks by several contexts, should a task match any selected context or all
  selected contexts? → A: Match any selected context.

- Q: What should happen when the user explicitly forces a sync? → A: Exchange and merge changes with
  the user's remote data so multiple devices can sync; all mobile-side data remains stored in a local
  database.

- Q: When two devices change the same task before syncing, how should Tasky resolve conflicting
  changes, including deletion versus editing? → A: Merge changes to different fields; use the most
  recent change for conflicts, including recoverable deletion versus editing. Permanent deletion always
  wins, as clarified in the subsequent pass.

- Q: For conflicting edits during sync, should “most recent” mean when the edit was made or when
  the server received it? → A: Use the recorded edit time; equal timestamps use a fixed, consistent
  tie-breaker. Device clock differences can affect the winner.

- Q: After a reminder has synced to multiple devices, which devices should display its notifications?
  → A: Notify only on the device where the reminder was created.

- Q: If a task is permanently deleted on one device, may a later edit from another device restore it
  during sync? → A: No. Permanent deletion always wins; discard edits to that task when other devices
  next sync. The most-recent-change rule applies only to recoverable deletion.

- Q: Should quiet hours and notification-preview preferences sync across devices or remain separate
  on each device? → A: Keep quiet hours and notification-preview preferences local to each device and
  exclude them from sync.

- Q: If an account is deleted on one device, when must another device with Tasky already open lose
  access to its local tasks? → A: Deny the next task operation after deletion; require an online
  account-status check for every operation and deny access if that check fails.

- Q: When a task is completed on another device, when should the device that created its reminder stop
  notifying? → A: Stop when the originating device receives the completion through successful manual
  sync; notifications may continue until then.

- Q: Should scheduled reminders appear while the device is offline? → A: Show a generic reminder
  without task details; opening or completing the task still requires online authentication and
  an account-status check.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Capture and Edit Commitments (Priority: P1)

As a user receiving a request or remembering a commitment, I can capture it immediately as a New task
and add details later so I no longer need to keep it in my head.

**Why this priority**: Trusted, low-friction capture is the foundation of GTD and provides useful
personal task management before reminders or structured organization exists.

**Independent Test**: A user can create a task with only a title, find it in the task list, add details,
move it to In Progress, and later view it with all changes preserved.

**Acceptance Scenarios**:

1. **Given** an authenticated user viewing any primary screen, **When** the user captures a task with
   a title, **Then** the task is saved to that user's private task list in the New state and is
   immediately available.
2. **Given** a New task, **When** the user adds notes, a due date, contexts, and changes its state to
   In Progress, **Then** the details are preserved and the task appears in the In Progress view.
3. **Given** a task title is blank, **When** the user attempts to save it, **Then** the task is not
   created and the user receives a clear correction message without losing entered details.
4. **Given** another user has not been granted access, **When** that user attempts to locate or open
   the task, **Then** neither the task nor its existence is disclosed.
5. **Given** a request has a missing, invalid, expired, or revoked JWT, **When** it reaches the task API,
   **Then** access is denied without disclosing task content.
6. **Given** the user previously authenticated and then closed the application, **When** the user opens
   it again without connectivity to the account API, **Then** task access remains locked until online
   authentication succeeds.
7. **Given** an attacker obtains a user's JWT without the device-held private key, **When** the attacker
   presents that JWT from another client, **Then** the APIs reject it without disclosing task content.
8. **Given** a user supplies the correct password and a fresh SMS OTP, **When** a new device submits its
   DPoP public key, **Then** the account API registers that key and records the enrollment event.
9. **Given** an SMS OTP is incorrect, expired, previously used, or exceeds its retry limit, **When** a new
   device attempts enrollment, **Then** key registration is denied and the user receives no account data.
10. **Given** a user confirms account deletion, **When** deletion completes, **Then** the account is
    disabled, its task data is soft-deleted, and every subsequent access attempt is denied.
11. **Given** a user opens any Tasky screen or receives a Tasky notification, **When** Tasky presents
    interface text, **Then** all Tasky-authored text is displayed in French.
12. **Given** a user's JWT expires while task changes remain unsaved, **When** the user attempts another
    task operation, **Then** Tasky preserves the draft, locks task operations, and requires the account
    password before issuing a replacement JWT and resuming access.
13. **Given** a user has forgotten the account password, **When** the user requests password recovery,
    **Then** Tasky explains that recovery is unavailable and discloses no account or task data.
14. **Given** a person submits a telephone number, password, and first device public key, **When** Tasky
    accepts the registration request, **Then** it creates a disabled pending account that cannot receive
    a JWT or access tasks.
15. **Given** a disabled pending account and its valid SMS OTP, **When** verification succeeds, **Then**
    Tasky atomically activates the account and binds its first device key.
16. **Given** an account was deleted on one device while Tasky remains open on another with an
    unexpired JWT, **When** the other device attempts its next task operation, **Then** an online
    account-status check denies access without disclosing task data.
17. **Given** Tasky remains open with an unexpired JWT, **When** a task operation cannot complete its
    online account-status check, **Then** the operation is denied, unsaved input is preserved, and
    no task data is disclosed.

---

### User Story 2 - Receive Kind Reminders and Complete Work (Priority: P2)

As a user, I can choose when and how a task reminds me, respond without guilt or pressure, and mark
the task complete when the commitment is fulfilled.

**Why this priority**: Reminders turn stored commitments into timely assistance and directly address
the risk of forgetting tasks amid competing demands.

**Independent Test**: A user can schedule a reminder, receive it at the intended time, open or complete
the task from it, reschedule the reminder from the task, and verify that obsolete reminders stop.

**Acceptance Scenarios**:

1. **Given** a New or In Progress task with an enabled reminder, **When** its scheduled time arrives outside
   quiet hours, **Then** the user receives one respectful reminder with actions to open or complete the
   task.
2. **Given** a reminder arrives during configured quiet hours, **When** quiet hours end, **Then** the
   reminder is delivered without sending repeated notifications for the same occurrence.
3. **Given** a New or In Progress task, **When** the user marks it Done, **Then** its completion is recorded
   and future reminders on that device are cancelled; reminders on other originating devices stop when
   those devices receive the completion through manual sync. A New task automatically moves through
   In Progress to Done in that single action, with both transitions recorded in its history.
4. **Given** a Done task, **When** the user copies it, **Then** a separate New task is created without
   changing the Done task or creating a relationship between them; the copy has a new identity and
   history, retains future scheduling fields, omits past due dates and expired reminders, and prompts the
   user to set fresh values for omitted scheduling fields.
5. **Given** reminder delivery is unavailable, **When** the user next opens the application, **Then**
   the user sees the missed reminder and a clear explanation of the delivery limitation.
6. **Given** a New or In Progress task has a daily reminder, **When** each scheduled daily time arrives,
   **Then** the user receives at most one notification for that occurrence until the reminder is stopped
   or the originating device records or receives the task's completion.
7. **Given** a reminder was created in one time zone, **When** the device moves to another time zone or
   the original zone changes its daylight-saving offset, **Then** the reminder retains its selected local
   clock time under the original zone's current rules.
8. **Given** Tasky is closed or authentication has expired, **When** the user taps Complete on a
   reminder, **Then** Tasky requests authentication and automatically completes the task only after
   authentication succeeds; failed or cancelled authentication leaves the task unchanged.
9. **Given** a reminder created on one device has synced to another device, **When** its scheduled
   occurrence arrives, **Then** only the originating device may display the notification; the other
   device does not display it, even if the originating device is unavailable.
10. **Given** a user has different quiet hours or notification-preview preferences on two devices,
    **When** the user changes these preferences on one device and syncs both devices, **Then** each
    device retains its own settings, and reminders follow the originating device's local preferences.
11. **Given** a task is completed on another device, **When** its reminder's originating device has not
    received the completion through manual sync, **Then** scheduled notifications may continue there;
    once successful manual sync receives the completion, all future notifications for that task stop.
12. **Given** an enabled scheduled reminder on an offline originating device, **When** its delivery time
    arrives outside quiet hours, **Then** a generic notification appears without task details, regardless
    of notification-preview preferences.
13. **Given** an offline reminder notification, **When** the user selects Open or Complete, **Then**
    task details remain hidden and the task remains unchanged until connectivity, valid authentication,
    and a successful online account-status check permit the action.

---

### User Story 3 - Organize and Review Work (Priority: P3)

As a user, I can organize standalone tasks into GTD views and regularly review my commitments so I
can decide what deserves attention now.

**Why this priority**: Organization and review make the captured list trustworthy as it grows and
prevent the task list from becoming another source of overload.

**Independent Test**: A user can update a mixed task list, filter standalone tasks by context, and
complete a guided review of outstanding work.

**Acceptance Scenarios**:

1. **Given** tasks in different states, **When** the user opens a GTD view, **Then** only tasks matching
   New, In Progress, or Done are shown.
2. **Given** two tasks describe parts of the same outcome, **When** the user edits or completes either
   task, **Then** the other task remains unchanged and no relationship is inferred between them.
3. **Given** a large task collection, **When** the user searches or filters by text, one or more
   contexts, state,
   or due state, **Then** matching tasks are shown and the filter can be cleared; selecting multiple
   contexts includes a task when it has at least one selected context.
4. **Given** the user starts a review, **When** the user processes each prompted category, **Then** the
   review records progress and identifies overdue, New, and In Progress tasks.
5. **Given** one task is due today without a time and another has a due time today, **When** the timed
   task's due time passes, **Then** only the timed task is overdue until the following local day begins.
6. **Given** a task has a due time created in one time zone, **When** the device moves to another time
   zone, **Then** the overdue instant remains determined by the originating zone's current rules.
7. **Given** the same user has changed tasks on two registered devices, **When** the user explicitly
   forces sync on each device with connectivity available, **Then** local changes are uploaded and
   remote changes are downloaded and merged into each device's local database; non-conflicting changes
   are preserved and neither device receives another user's data.
8. **Given** a sync fails or is interrupted, **When** the user retries it, **Then** local data and pending
   changes are preserved and the retry does not duplicate tasks or lose non-conflicting saved changes.
9. **Given** two devices changed different fields of the same task before sync, **When** their changes
   are synchronized, **Then** both field changes are retained.
10. **Given** two devices changed the same task field before sync, **When** their changes are
    synchronized, **Then** the most recent change to that field wins without asking the user to resolve
    the conflict.
11. **Given** one device recoverably deleted a task and another edited it before sync, **When** their changes are
    synchronized, **Then** the more recent change wins: a later deletion leaves the task deleted, while
    a later edit leaves it undeleted, subject to account access and task lifecycle requirements.
12. **Given** two conflicting edits have different recorded edit times, **When** the older edit arrives
    at the server after the newer edit, **Then** the edit with the later recorded edit time wins.
13. **Given** conflicting edits have equal recorded edit times, **When** devices sync in different
    orders or retry synchronization, **Then** the fixed tie-breaker selects the same winner on all devices.
14. **Given** device clocks differ, **When** their edits conflict during sync, **Then** the winner is
    determined by recorded edit times even if these differ from the edits' actual chronological order.
15. **Given** a task was permanently deleted on one device and edited later on another, **When** the
    devices next sync, **Then** permanent deletion wins, the edits are discarded, and the task cannot
    be restored by that sync or a retry.

### Edge Cases

- A due date passes while the device is powered off or unable to present reminders.
- A time-zone database rule changes after a reminder has been scheduled.
- A task is marked Done, copied, edited, or deleted close to its reminder time.
- A task is completed on another device before its reminder's originating device receives the
  completion through manual sync; notifications may continue on the originating device until then.
- A copied task contains a mixture of future and already-passed due dates or reminders.
- Search and review operate over thousands of New, In Progress, and Done tasks.
- Notification previews could expose sensitive task text on a locked device.
- A scheduled reminder becomes due offline: only generic wording is displayed, and its actions remain
  subject to online authentication and account-status checks.
- Password reauthentication fails or is cancelled after a JWT expires while a task draft is pending.
- A user forgets the account password and no recovery path is available.
- A deletion fails partway through and is retried.
- Two devices edit the same task before a manual sync: changes to different fields merge, and the
  most recent change wins when the same field differs.
- One device recoverably deletes a task while another edits it: the more recent change determines
  whether the task remains deleted or undeleted, subject to account access and task lifecycle requirements.
- One device permanently deletes a task while another edits it: permanent deletion always wins and
  edits to that task are discarded when devices next sync.
- Conflicting changes have identical recorded edit times: a fixed, deterministic tie-breaker MUST
  select the same winner on every device regardless of sync arrival order or retries.
- Devices have different clock settings: conflict resolution uses recorded edit times even when clock
  differences cause a change made earlier in real time to win.
- A manual sync loses connectivity after only some changes have been exchanged.
- A task reaches the end of its 30-day restoration window without being restored or manually deleted.
- The device-local task API is temporarily unavailable while the user views or edits a task.
- A JWT expires or is revoked while the device cannot contact the account API; every task operation
  fails closed when its online account-status check cannot succeed.
- The account API is unavailable when the application launches.
- A valid JWT is replayed without the DPoP proof, with a reused proof, or from a different device key.
- An SMS OTP is delayed, intercepted, reused, guessed repeatedly, or redirected after a SIM swap or
  telephone-number port.
- An initial-registration SMS OTP expires while its disabled pending account still exists.
- A user changes the verified telephone number and immediately attempts to register a new device key.
- A deleted account's old JWT or registered device key is used to request soft-deleted task data.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The account API MUST authenticate each Tasky account through a JWT-based security layer;
  each application launch MUST obtain a JWT through online authentication, every task operation MUST
  present a valid JWT, and each API MUST reject missing, invalid, expired, or revoked credentials. The
  backend MUST remain stateless and MUST NOT retain server-side session state between requests. When a
  JWT expires while the application remains open, Tasky MUST preserve unsaved input, lock all task
  operations, and require the account password before obtaining a replacement JWT and resuming access.
  Before every task operation, including reads, writes, and sync, Tasky MUST obtain an online check of
  current account status. A deleted or disabled account, or a failed or unavailable check, MUST deny the
  operation even when the presented JWT has not expired. Cached account status MUST NOT authorize an
  operation; failed checks MUST preserve unsaved input without exposing task data.
  Displaying a previously scheduled generic offline reminder under FR-013 is exempt from the online
  check; this exemption MUST NOT authorize task-content access or task changes.
- **FR-002**: The system MUST provide every user with a personal task space that is private by default,
  store all mobile-side data in a local database on that user's device through TaskChampion, and expose
  task operations through a device-local Hyper-based API. Remote account-scoped data MUST support
  user-triggered, two-way synchronization across the user's registered devices.
- **FR-003**: Users MUST be able to capture a task with only a non-blank title from any primary task view.
- **FR-004**: Every newly created task MUST start in the New state.
- **FR-005**: Users MUST be able to edit a task's title, notes, state, due date, reminder schedule,
  and contexts.
- **FR-006**: The system MUST provide exactly the task states New, In Progress, and Done. It MUST allow
  only New → In Progress and In Progress → Done transitions and MUST preserve every transition in task
  history. Completing a New task MUST automatically apply New → In Progress → Done in a single user
  action and record both transitions without requiring a separate start action.
- **FR-007**: Every task MUST be standalone; creating, editing, completing, copying, or deleting one task
  MUST NOT create a relationship with or change another task.
- **FR-008**: Users MUST be able to create reusable contexts that describe where or how a task can be
  performed and assign zero or more contexts to each task. A context MAY be assigned to multiple tasks.
- **FR-009**: Users MUST be able to search and filter their tasks by text, state, one or more contexts,
  and due state. When multiple contexts are selected, a task MUST match the context filter if it has
  at least one selected context.
- **FR-010**: The system MUST provide a guided review covering New, In Progress, overdue, and Done tasks
  and MUST preserve review progress if interrupted.
- **FR-011**: Users MUST be able to schedule one or more reminders for a New or In Progress task, including
  a date and local time. Each reminder MUST support either a one-time occurrence or daily repetition;
  a daily reminder MUST repeat until the originating device records the reminder being stopped or
  records or receives the task's completion. Completion on another device cancels future notifications
  only when the originating device receives it through successful manual sync. Every reminder MUST retain
  the time zone in which it was created and use that zone's current daylight-saving rules even if the
  device later changes time zone. Every reminder MUST retain the identity of the device on which it
  was created. Only that device MAY display notifications for the reminder; synchronization MUST NOT
  transfer notification delivery to another device, including when the originating device is unavailable.
- **FR-012**: Users MUST be able to configure quiet hours and whether sensitive task text appears in a
  reminder preview; previews MUST hide task text by default on a locked device. Quiet hours and
  notification-preview preferences MUST be stored locally per device and MUST NOT be synchronized;
  changing either preference on one device MUST NOT affect another device's settings.
  Offline notifications MUST hide all task details regardless of the device's preview preference.
- **FR-013**: Each reminder MUST use neutral, respectful wording and MUST offer open and complete actions
  without shame, threat, or repeated pressure. The first release MUST NOT offer a snooze action. If the
  user selects Complete while Tasky is closed or authentication has expired, Tasky MUST request the
  authentication required by FR-001 and automatically complete the task after successful authentication.
  Failed or cancelled authentication MUST leave the task unchanged.
  Previously scheduled reminders MUST still appear offline on their originating device, subject to
  quiet hours, notification permission, and device availability, using generic French wording without
  task details. Opening or completing a task from such a notification MUST require connectivity,
  valid authentication, and a successful online account-status check under FR-001; until these succeed,
  task details MUST remain hidden and the task MUST remain unchanged.
- **FR-014**: Marking a task Done MUST record the completion time and MUST cancel all future
  reminders for that task on the device recording the completion. Other originating devices MUST cancel
  future reminders when they receive the completion through successful manual sync; until then,
  notifications MAY continue. Completing a task MUST NOT trigger automatic cross-device cancellation.
- **FR-015**: A Done task MUST NOT be reopened or moved to an earlier state. Users MUST be able to copy a
  Done task into a distinct New task while the original remains unchanged and no relationship is created.
  The copy MUST retain every user-editable field and all future due dates and reminders, but MUST omit
  past due dates and expired reminders and prompt the user to set fresh scheduling values. It MUST receive
  a new identity, creation time, and history.
- **FR-016**: Users MUST be able to move tasks to a recoverable deletion area, restore them for 30 days,
  and permanently delete content they own. After the 30-day restoration window ends, this feature
  defines no automatic deletion, retention, or other disposition behavior. Permanent deletion MUST
  take precedence over edits during sync regardless of their timestamps; other devices MUST discard
  edits to that task when they next sync and MUST NOT restore it through sync or retries.
- **FR-017**: Users MUST be able to delete their account. Deletion MUST disable the account immediately,
  revoke its registered device keys, and mark all associated task data as soft-deleted and inaccessible.
  This feature MUST NOT recover, share, export, or permanently erase soft-deleted account data, and it
  defines no retention duration.
- **FR-018**: The system MUST protect task content, account data, reminder details, and soft-deleted data
  from unauthorized disclosure or modification during storage, transmission, and reminder display.
- **FR-019**: Security-relevant events, including authentication failure, access denial, device-key
  changes, and deletion requests, MUST be auditable by an authorized operator without recording
  secret credentials or unnecessary task content.
- **FR-020**: Recoverable errors MUST explain what happened, retain unsaved user input, and provide a safe
  retry path without duplicating or corrupting tasks.
- **FR-021**: All remote API traffic MUST use TLS 1.3 with strict hostname and certificate validation;
  cleartext traffic and permissive certificate validation MUST be rejected.
- **FR-022**: Every issued JWT MUST be sender-constrained with DPoP to a non-exportable asymmetric key
  held in Android Keystore. Each protected request MUST carry a fresh proof bound to its HTTP method,
  target URI, issued JWT, and device key; invalid or replayed proofs MUST be rejected.
- **FR-023**: Registering a new device-held DPoP public key MUST require the account password and a
  single-use SMS OTP sent to the account's verified telephone number. The OTP MUST expire after five
  minutes, allow no more than five failed attempts, and become invalid immediately after successful use.
- **FR-024**: Changing the verified telephone number MUST require account reauthentication, notify the
  previously verified contact channel, and block new device-key registration for 24 hours.
- **FR-025**: The first release MUST present all Tasky-authored interface text, reminder content,
  validation messages, authentication messages, and error messages in French.
- **FR-026**: The first release MUST NOT provide forgotten-password recovery or password reset without
  the current password. A failed recovery request MUST NOT disclose whether an account exists or expose
  account or task data.
- **FR-027**: Self-registration MUST accept a telephone number, password, and first device public key and
  create a disabled pending account before SMS verification. A pending account MUST NOT receive a JWT or
  access task data. Successful verification with the SMS OTP defined by FR-023 MUST atomically activate
  the account and bind the submitted first device public key. This feature defines no expiry, deletion,
  or telephone-number release rule for an unverified pending account.
- **FR-028**: A task due date MUST be a calendar date with an optional local time. A task with a due time
  MUST become overdue immediately after that time; a date-only task MUST become overdue at the start of
  the following local calendar day. A due time MUST retain the time zone in which it was created and use
  that zone's current daylight-saving rules even if the device later changes time zone.

- **FR-029**: Users MUST be able to explicitly force a two-way sync that uploads local changes and
  downloads and merges remote changes for their own account across registered devices. All mobile-side
  data MUST remain stored in the local database. Sync MUST require valid authentication and connectivity,
  preserve non-conflicting changes and task identities, and prevent access to another account's data.
  Failed or interrupted sync MUST preserve local data and pending changes and support retry without
  duplicating tasks or losing non-conflicting saved changes. This feature MUST NOT synchronize
  automatically. Quiet hours and notification-preview preferences MUST be excluded from sync.
- **FR-030**: Sync MUST merge changes to different fields of the same task. When changes conflict on
  the same field, the most recent change MUST win without prompting the user. For deletion versus
  editing, the more recent change MUST determine whether the task remains deleted or undeleted only
  for recoverable deletion. Permanent deletion MUST always win, as required by FR-016.
  Conflict resolution MUST preserve account access restrictions and the permitted task lifecycle;
  it MUST NOT reopen a Done task or restore access to a deleted account's data. “Most recent” MUST
  mean the recorded time of the edit, not the time the server receives it. Equal edit timestamps MUST
  use a fixed, deterministic tie-breaker that yields the same winner on every device regardless of sync
  arrival order or retries. Recorded edit times MUST remain unchanged during sync and retries; device
  clock differences MAY therefore affect which change wins.

### Key Entities

- **User**: A person with a Tasky account, lifecycle status, authenticated identity, verified telephone
  number, registered device public keys, and preferences. Quiet hours and notification-preview
  preferences belong to each device and are not synchronized. A new account is disabled pending
  SMS verification and becomes Active only when its first device key is bound; account deletion disables
  the account and soft-deletes its data.
- **Task Collection**: A user's private boundary containing their tasks, contexts, reminders,
  and review history, stored locally on each device and synchronized with account-scoped remote data
  when the user explicitly forces sync.
- **Task**: A standalone commitment with a title, optional notes, state, calendar due date, optional due
  time and its originating time zone, reminders, contexts, history, and completion information; it
  follows New → In Progress → Done, cannot transition backward, and has no relationship to another task
  in this feature. A copy receives a new identity and history while retaining all user-editable fields
  and only future scheduling values from its source.
- **Context**: A reusable label describing a setting, resource, energy level, or situation; it may be
  assigned to multiple tasks, and each task may reference zero or more contexts.
- **Reminder**: A user-selected prompt associated with a New or In Progress task, scheduled time,
  originating time zone, originating device identity, one-time or daily recurrence mode, active state,
  and delivery state. Only its originating device displays its notifications.
- **Review Session**: A user's progress through a structured examination of New, In Progress, overdue,
  and Done tasks.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 90% of first-time test participants can capture a task in 15 seconds or less
  without guidance.
- **SC-002**: At least 90% of test participants can add details and move a New task to In Progress in
  60 seconds or less without guidance.
- **SC-003**: At least 95% of eligible reminders become visible to the user within 60 seconds of the
  configured time.
- **SC-004**: At least 90% of test participants can open or complete a task directly from a reminder on
  their first attempt.
- **SC-005**: In authorization testing, 100% of requests from another account or with a missing, invalid,
  expired, or revoked JWT, including credentials from a deleted account, are denied without disclosing
  task content or existence.
- **SC-006**: At least 85% of test participants can identify In Progress tasks matching a chosen context
  and overdue tasks within 30 seconds.
- **SC-007**: Users can search, filter, and review a collection of 10,000 tasks without any tested action
  taking longer than two seconds to show a usable result.
- **SC-008**: At least 80% of pilot users rate reminder wording as respectful and helpful after four weeks
  of regular use.
- **SC-009**: In transport and token-replay testing, 100% of cleartext connections, invalid server
  identities, unbound JWTs, mismatched DPoP proofs, and replayed proofs are rejected.
- **SC-010**: In device-enrollment testing, 100% of incorrect, expired, reused, or retry-exhausted SMS
  OTPs and all enrollment attempts during the telephone-number-change hold are rejected.
- **SC-011**: In synchronization acceptance tests, 100% of non-conflicting field changes are preserved,
  conflicting changes resolve according to FR-030, and registered devices converge after successful
  manual syncs with no further edits. Interrupted-sync retries create no duplicate tasks, and no test
  exposes another account's data.

## Assumptions

- The first release supports Android 10 and later and does not include an iOS, desktop, or web client.
- French is the only supported interface language in the first release.
- Users create individual accounts before accessing personal task data.
- Users who forget their account password cannot recover access in the first release.
- Tasks have no parent, child, predecessor, successor, dependency, grouping, or inferred relationship in
  this feature.
- Reminder delivery depends on user-granted notification permission and device availability; missed
  reminders remain visible inside the application after authorized access succeeds. Previously scheduled
  reminders can appear offline with generic wording only; opening or completing their tasks requires
  online access under FR-001.
- A stable source of current time is available for reminder scheduling and task history.
- Every application launch requires connectivity to the account API. A JWT obtained during that launch
  authorizes local task operations only while the application remains open, the JWT remains valid,
  and an online account-status check succeeds for each operation. Loss of connectivity blocks task
  operations even before JWT expiry. Expiry requires password reauthentication; Tasky does not renew
  the JWT silently.
- The backend retains no server-side session; each request is authorized solely from its presented
  credentials and current account data.
- The first release has no formal availability or authentication-latency objective for the account API;
  outages still fail closed and prevent task access.
- The first release defines no rate limit, delay, or account lockout for repeated failed password
  authentication attempts. This accepts increased exposure to online password guessing to keep the
  first-release authentication scope limited. A future security iteration is expected to add and test
  password-attempt throttling before professional use. This remains subject to the security release gate.
- The first release defines no password length, composition, compromised-password screening, or
  expiration rules. Password-policy hardening is deferred and remains subject to the security release
  gate.
- The first release retains unverified pending accounts without a defined cleanup time or
  telephone-number reuse rule.
- The first release defines no disposition for task content after its 30-day restoration window ends.
- Each Android installation generates its DPoP key locally; private key material never leaves Android
  Keystore, and the account API stores only the corresponding public key and registration metadata.
- SMS is an interim second factor for consumer onboarding and device replacement. Authenticator-app TOTP
  is deferred to the next iteration aimed at professional use.
- Account deletion soft-deletes task data without a defined retention period. Recovery, sharing, export,
  retention duration, and permanent erasure are deferred to a future professional feature.

### Scope Boundaries

- Included: personal task management, GTD classification and review, task reminders, task history,
  recoverable deletion, account privacy, and user-triggered two-way synchronization across the user's
  registered devices.
- Excluded from this feature: calendar or email integrations, file attachments, chat, organizational
  administration, projects, task grouping, task dependencies, task sequencing, other task relationships,
  shared task spaces, task assignment, offline task access, enterprise single sign-on, billing,
  location-triggered reminders, automated task creation from third-party services, and
  artificial-intelligence task decisions. Formal accessibility conformance requirements and acceptance
  testing, password-attempt throttling, password-policy hardening, forgotten-password recovery,
  unverified-account cleanup, post-restoration-window task disposition, reminder snoozing, and
  localization beyond French are deferred to a future release. Relationships between tasks belong to a
  separate future feature; authenticator-app TOTP and soft-deleted data recovery, sharing, export,
  retention, and erasure belong to a future professional-use iteration.

### Dependencies

- Users must permit notifications to receive reminders outside the application.
- The device must provide a trustworthy local date and time for time-sensitive behavior.
- Accessing and changing tasks depends on the device-local TaskChampion store and Hyper-based API,
  plus a successful online account-status check for every task operation.
- Signing in and renewing JWT credentials depends on connectivity to the Tasky account API.
- Manual two-way synchronization depends on connectivity to an authenticated remote service that stores
  the user's account-scoped synchronized data.
- New-device enrollment depends on delivery to the account's verified SMS-capable telephone number.
- Initial account activation depends on successful SMS verification and atomic first-device-key binding.
