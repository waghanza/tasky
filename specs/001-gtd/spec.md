# GTD Task Assistant

## Clarifications

### Session 2026-09-16

- Q: How should Tasky protect local task content when the device is already unlocked? → A: Let the
  user optionally enable a Tasky PIN or biometric lock. Local task use remains available without an
  account, but the enabled app lock must be satisfied before protected local content is shown.
- Q: When the optional Tasky app lock is enabled, when should it lock again after you leave the app? → A: Lock immediately whenever Tasky goes into the background.
- Q: While Tasky is locked, should completing a task from a reminder require local unlock? → A: Require local unlock for both Open and Complete.
- Q: How should users regain access if they forget their Tasky PIN and cannot use biometrics? → A: Allow the device's PIN, password, or pattern to reset the Tasky lock, preserving local tasks.
- Q: While Tasky is locked, should reminder notifications hide task details even when previews are enabled? → A: Always hide task details while Tasky is locked.

## User Scenarios & Testing

### User Story 1 - Capture and Edit Commitments (Priority: P1)

As a user receiving a request or remembering a commitment, I can capture it immediately as a new task
with a title of at least 20 characters serving as its short description, and add optional details later
so I no longer need to keep it in my head.

**Why this priority**: Trusted, low-friction capture is the foundation of GTD and provides useful personal task management before reminders or structured organization exists.

**Independent Test**: A user can create a task with only a nonblank title of at least 20 characters,
find it in the task list, add details, move it to In Progress, and later view it with all changes preserved.

**Acceptance Scenarios**:

1. **Given** a user viewing any primary screen without signing in, **When** the user captures a task
   with a nonblank title of at least 20 characters and no other fields, **Then** the task is saved
   locally in the New state and is immediately available, including offline.
2. **Given** a new task, **When** the user adds notes, a due date, contexts, and changes its state to In Progress, **Then** the details are preserved and the task appears in the In Progress view.
3. **Given** a task title is blank or contains fewer than 20 characters, **When** the user attempts to
   create or edit the task, **Then** the invalid change is not saved and the user receives a clear
   correction message without losing entered details.
4. **Given** another account has not been granted access, **When** it attempts to fetch the user's remote task data through sync, **Then** neither the data nor its existence is disclosed.
5. **Given** a sync request has a missing, invalid, expired, or revoked JWT, **When** it reaches the remote API, **Then** synchronization is denied without disclosing remote task content, and local task use remains available.
6. **Given** a user has never created an account or signed in, **When** the user launches Tasky offline,
   **Then** the user can create, view, edit, complete, and delete local tasks, and saved changes survive
   closing and reopening the application without account authentication, subject to any enabled local app lock.
7. **Given** an attacker obtains a user's JWT without the device-held private key, **When** the attacker
   presents that JWT from another client, **Then** the APIs reject it without disclosing task content.
8. **Given** a user supplies the correct password and a fresh SMS OTP, **When** a new device submits its
   DPoP public key, **Then** the account API registers that key and records the enrollment event.
9. **Given** an SMS OTP is incorrect, expired, previously used, or exceeds its retry limit, **When** a new
   device attempts enrollment, **Then** key registration is denied and the user receives no account data.
10. **Given** a user confirms account deletion, **When** deletion completes, **Then** the account is
    disabled, its remote task data is soft-deleted, and subsequent synchronization attempts are denied;
    existing local tasks remain available on every device without account authentication, subject to any enabled local app lock.
11. **Given** a user opens any Tasky screen or receives a Tasky notification, **When** Tasky presents
    interface text, **Then** all Tasky-authored text is displayed in French.
12. **Given** a user's JWT expires while task changes remain unsaved, **When** the user attempts another
    local task operation, **Then** Tasky allows the operation and preserves the draft; password
    reauthentication is required only when the user next attempts synchronization.
13. **Given** a user has forgotten the account password, **When** the user requests password recovery,
    **Then** Tasky explains that recovery is unavailable and discloses no remote account or task data;
    the user's existing local tasks remain usable.
14. **Given** a person submits a telephone number, password, and first device public key, **When** Tasky
    accepts the registration request, **Then** it creates a disabled pending account that cannot receive
    a JWT or synchronize remote data; local task use remains available.
15. **Given** a disabled pending account and its valid SMS OTP, **When** verification succeeds, **Then**
    Tasky atomically activates the account and binds its first device key.
16. **Given** an account was deleted on one device while Tasky remains open on another with an
    unexpired JWT, **When** the other device next attempts synchronization, **Then** the remote service
    checks current account status and denies synchronization without disclosing remote task data;
    local tasks and pending local changes are preserved and remain usable.
17. **Given** the remote account service is unavailable, **When** the user attempts synchronization,
    **Then** sync fails without losing local data or pending changes, and local task operations remain
    available without an account-status check.
18. **Given** a new task draft with no optional details, **When** its nonblank title contains exactly
    19 characters, **Then** saving is rejected; after the title reaches 20 characters, saving succeeds
    without a separate description field.
19. **Given** a user has enabled a Tasky PIN or biometric lock, **When** the user opens Tasky or
    returns to it after any time in the background, **Then** Tasky requires that local unlock before
    showing protected task content; synchronization authentication remains separate.
20. **Given** the user cannot unlock Tasky with their Tasky PIN or biometrics, **When** they
    successfully verify the device's PIN, password, or pattern through local lock recovery,
    **Then** they can reset the Tasky lock and regain access without losing local tasks,
    requiring connectivity, or authenticating a synchronization account. Failed, cancelled, or
    unavailable device verification leaves the Tasky lock and local data unchanged.

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
   those devices receive the completion through manual sync. A new task automatically moves through
   In Progress to Done in that single action, with both transitions recorded in its history.
4. **Given** a Done task, **When** the user copies it, **Then** a separate new task is created without
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
8. **Given** Tasky is closed or account authentication has expired, **When** the user taps Complete on a
   reminder, **Then** Tasky completes the task locally without account authentication or connectivity
   after successful local unlock if Tasky is locked, and retains the change for a later user-triggered sync.
   Open likewise requires local unlock before displaying task content. If local unlock fails, is
   cancelled, or is unavailable, neither action reveals task content or completes the task.
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
    Tasky opens the local task or completes it locally without account sign-in or an online account-status
    check, after successful local unlock if Tasky is locked.
14. **Given** Tasky is locked and notification previews are enabled, **When** a reminder is displayed,
    **Then** its notification contains only generic wording without task details, whether online or offline.
    Any notification already showing task details must hide those details when Tasky locks.

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
16. **Given** a local task collection has synchronized with one account, **When** synchronization is
    attempted using a different account, **Then** the attempt is rejected before any collection data
    is exchanged, and local tasks and pending changes remain usable and unchanged.
17. **Given** a collection's original synchronization account has been deleted, **When** the user tries
    to synchronize that collection with a new account, **Then** the attempt is rejected while local
    task use remains available; account deletion does not enable collection sharing or transfer.

### Edge Cases

- A due date passes while the device is powered off or unable to present reminders.
- A time-zone database rule changes after a reminder has been scheduled.
- A task is marked Done, copied, edited, or deleted close to its reminder time.
- A task is completed on another device before its reminder's originating device receives the
  completion through manual sync; notifications may continue on the originating device until then.
- A copied task contains a mixture of future and already-passed due dates or reminders.
- Search and review operate over thousands of New, In Progress, and Done tasks.
- A task title contains 19 or exactly 20 characters; the minimum also applies when editing a title.
- Notification previews could expose sensitive task text on a locked device.
- Tasky locks while a reminder notification is already visible: task details are hidden immediately,
  regardless of preview preferences or connectivity.
- A scheduled reminder becomes due offline: only generic wording is displayed, and its actions remain
  available locally without account authentication.
- Password reauthentication for sync fails or is cancelled while a task draft is pending; the draft,
  local task access, and pending sync changes remain available.
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
- The local Realm database is temporarily unavailable while the user views or edits a task.
- A JWT expires or is revoked while the device cannot contact the account API; sync is blocked while
  local task operations continue.
- The account API is unavailable when the application launches; local task use remains available.
- A valid JWT is replayed without the DPoP proof, with a reused proof, or from a different device key.
- An SMS OTP is delayed, intercepted, reused, guessed repeatedly, or redirected after a SIM swap or
  telephone-number port.
- An initial-registration SMS OTP expires while its disabled pending account still exists.
- A user changes the verified telephone number and immediately attempts to register a new device key.
- A deleted account's old JWT or registered device key is used to request soft-deleted task data.
- A synchronization account is deleted while another device is offline: its local tasks remain usable,
  and the next sync attempt is denied without deleting or locking local data.
- A different account is used to synchronize an existing collection, including after deletion of its
  original account: reject the sync without exchanging data or changing local tasks.
- An optional local Tasky PIN or biometric lock is enabled, unavailable, cancelled, or fails while
  the device itself remains unlocked; protected task content remains hidden until local unlock succeeds.
  Reminder Open and Complete actions require successful local unlock while Tasky is locked; failed,
  cancelled, or unavailable unlock leaves the task unchanged and its content hidden.
- A user forgets the Tasky PIN and cannot use biometrics: successful verification of the device's
  PIN, password, or pattern permits local lock reset without data loss. If device verification is
  unavailable, fails, or is cancelled, protected content remains hidden and local data is preserved.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Local task use MUST NOT require a Tasky account, sign-in, JWT, connectivity, or online
  account-status check. Application launch, task creation, reading, editing, completion, copying,
  deletion, search, review, and reminder actions MUST work offline. Account authentication MUST apply
  only to synchronization and management of the synchronization account. Remote synchronization MUST
  require a valid JWT and verification of current account status; missing, invalid, expired, or revoked
  credentials, a deleted or disabled account, or an unavailable status check MUST deny synchronization.
  When a JWT expires, password reauthentication MUST be required on the next sync attempt, without
  blocking local operations or losing drafts and pending changes. The backend MUST remain stateless
  and MUST NOT retain server-side session state between requests.
- **FR-002**: The system MUST provide every user with a personal task space that is private by default,
  store all mobile-side task data in a local Realm database, and persist local changes independently
  of synchronization. Remote account-scoped data MUST support user-triggered, two-way synchronization
  across the user's registered devices; local task use MUST remain available without configuring sync.
- **FR-003**: Users MUST be able to capture a task from any primary task view using only a nonblank
  title of at least 20 characters. The title MUST serve as the task's short description; a separate
  description MUST NOT be required. Creating or editing a task with a blank title or one shorter than
  20 characters MUST be rejected without losing entered details. Notes and other details remain optional.
- **FR-004**: Every newly created task MUST start in the New state.
- **FR-005**: Users MUST be able to edit a task's title, notes, state, due date, reminder schedule,
  and contexts.
- **FR-006**: The system MUST provide exactly the task states New, In Progress, and Done. It MUST allow
  only New → In Progress and In Progress → Done transitions and MUST preserve every transition in task
  history. Completing a new task MUST automatically apply New → In Progress → Done in a single user
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
  While Tasky is locked, all reminder notifications MUST hide task details regardless of connectivity
  or preview preferences. This MUST also apply to notifications already visible when Tasky locks.
- **FR-012a**: Users MAY enable a separate Tasky PIN or biometric lock for local task content. When
  enabled, Tasky MUST lock immediately whenever it enters the background, with no grace period.
  Local unlock MUST be required before protected local task content is shown on launch or on return
  from the background, including when the device itself is unlocked. A failed, cancelled, or unavailable local
  unlock MUST leave protected content hidden without blocking synchronization-account authentication
  or deleting local data. The local lock MUST remain independent of synchronization credentials.
  Users MUST be able to reset the Tasky lock after successful verification of the device's PIN,
  password, or pattern, preserving all local tasks. This recovery MUST work offline without
  synchronization-account authentication. Failed, cancelled, or unavailable device verification
  MUST NOT reset the Tasky lock, reveal protected content, or delete local data.
- **FR-013**: Each reminder MUST use neutral, respectful wording and MUST offer open and complete actions
  without shame, threat, or repeated pressure. The first release MUST NOT offer a snooze action.
  Opening or completing a task from a reminder MUST operate locally without account authentication,
  including while Tasky is closed, the device is offline, or sync credentials have expired.
  While Tasky is locked, both Open and Complete MUST require successful local unlock before showing
  task content or completing the task. Failed, cancelled, or unavailable local unlock MUST leave
  task content hidden and MUST NOT complete the task.
  Previously scheduled reminders MUST still appear offline on their originating device, subject to
  quiet hours, notification permission, and device availability, using generic French wording without
  task details. Completion MUST be saved locally and remain available for a later user-triggered sync.
- **FR-014**: Marking a task Done MUST record the completion time and MUST cancel all future
  reminders for that task on the device recording the completion. Other originating devices MUST cancel
  future reminders when they receive the completion through successful manual sync; until then,
  notifications MAY continue. Completing a task MUST NOT trigger automatic cross-device cancellation.
- **FR-015**: A Done task MUST NOT be reopened or moved to an earlier state. Users MUST be able to copy a
  Done task into a distinct new task while the original remains unchanged and no relationship is created.
  The copy MUST retain every user-editable field and all future due dates and reminders, but MUST omit
  past due dates and expired reminders and prompt the user to set fresh scheduling values. It MUST receive
  a new identity, creation time, and history.
- **FR-016**: Users MUST be able to move tasks to a recoverable deletion area, restore them for 30 days,
  and permanently delete content they own. After the 30-day restoration window ends, this feature
  defines no automatic deletion, retention, or other disposition behavior. Permanent deletion MUST
  take precedence over edits during sync regardless of their timestamps; other devices MUST discard
  edits to that task when they next sync and MUST NOT restore it through sync or retries.
- **FR-017**: Users MUST be able to delete their account. Deletion MUST disable the account immediately,
  revoke its registered device keys, and mark its remote task data as soft-deleted and inaccessible
  through synchronization.
  Account deletion MUST preserve existing local tasks on every device, including the requesting device,
  and MUST NOT delete, soft-delete, or lock local data. Local task operations and reminders MUST remain
  available without account authentication, subject to any enabled local app lock; subsequent sync attempts for the deleted account MUST be denied.
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
  held in Android Keystore. Each protected remote request MUST carry a fresh proof bound to its HTTP method,
  target URI, issued JWT, and device key; invalid or replayed proofs MUST be rejected.
- **FR-023**: Registering a new device-held DPoP public key MUST require the account password and a
  single-use SMS OTP sent to the account's verified telephone number. The OTP MUST expire after five
  minutes, allow no more than five failed attempts, and become invalid immediately after successful use.
- **FR-024**: Changing the verified telephone number MUST require account reauthentication, notify the
  previously verified contact channel, and block new device-key registration for 24 hours.
- **FR-025**: The first release MUST present all Tasky-authored interface text, reminder content,
  validation messages, authentication messages, and error messages in French.
- **FR-026**: The first release MUST NOT provide forgotten-account-password recovery or account password reset without
  the current password. A failed recovery request MUST NOT disclose whether an account exists or expose
  remote account or task data; failed recovery MUST NOT block local task use.
- **FR-027**: Self-registration MUST accept a telephone number, password, and first device public key and
  create a disabled pending account before SMS verification. A pending account MUST NOT receive a JWT or
  synchronize remote task data. Pending account verification MUST NOT block local task use.
  Successful verification with the SMS OTP defined by FR-023 MUST atomically activate
  the account and bind the submitted first device public key. This feature defines no expiry, deletion,
  or telephone-number release rule for an unverified pending account.
- **FR-028**: A task due date MUST be a calendar date with an optional local time. A task with a due time
  MUST become overdue immediately after that time; a date-only task MUST become overdue at the start of
  the following local calendar day. A due time MUST retain the time zone in which it was created and use
  that zone's current daylight-saving rules even if the device later changes time zone.

- **FR-029**: Users MUST be able to explicitly force a two-way sync that uploads local changes and
  downloads and merges remote changes for their own account across registered devices. All mobile-side
  data MUST remain stored in the local Realm database. Sync MUST require valid authentication and connectivity,
  preserve non-conflicting changes and task identities, and prevent access to another account's data.
  Failed or interrupted sync MUST preserve local data and pending changes and support retry without
  duplicating tasks or losing non-conflicting saved changes. This feature MUST NOT synchronize
  automatically. Quiet hours and notification-preview preferences MUST be excluded from sync.
- **FR-030**: Sync MUST merge changes to different fields of the same task. When changes conflict on
  the same field, the most recent change MUST win without prompting the user. For deletion versus
  editing, the more recent change MUST determine whether the task remains deleted or undeleted only
  for recoverable deletion. Permanent deletion MUST always win, as required by FR-016.
  Conflict resolution MUST preserve account access restrictions and the permitted task lifecycle;
  it MUST NOT reopen a Done task or restore remote access to a deleted account's data. “Most recent” MUST
  mean the recorded time of the edit, not the time the server receives it. Equal edit timestamps MUST
  use a fixed, deterministic tie-breaker that yields the same winner on every device regardless of sync
  arrival order or retries. Recorded edit times MUST remain unchanged during sync and retries; device
  clock differences MAY therefore affect which change wins.
- **FR-031**: A synchronized task collection MUST remain associated with its original synchronization
  account in this release. Attempts to synchronize it with another account MUST be rejected before
  any collection data is exchanged, without modifying local tasks or pending changes. Signing out or
  deleting the original account MUST NOT remove this restriction or prevent local task use.
  Synchronization of a shared collection across accounts belongs to a separate future sharing feature;
  this release MUST NOT offer collection sharing or transfer between accounts.

### Key Entities

- **User**: A person using local tasks without a required account. An optional synchronization account
  has a lifecycle status, authenticated identity, verified telephone number, and registered device public
  keys. The user may also have an optional local Tasky PIN or biometric lock. Quiet hours and notification-preview
  preferences belong to each device and are not synchronized. A new account is disabled pending
  SMS verification and becomes Active only when its first device key is bound; account deletion disables
  the account and soft-deletes its remote data while preserving usable local tasks on every device.
- **Task Collection**: A user's private boundary containing their tasks, contexts, reminders,
  and review history, stored locally in Realm and usable without account authentication, subject to any enabled local app lock. Account-scoped remote
  data is exchanged only when the user authenticates and explicitly forces sync. A collection can exist
  without an account; once synchronized, it retains its original synchronization-account identity and
  cannot synchronize with a different account in this release.
- **Task**: A standalone commitment with a nonblank title of at least 20 characters serving as its short
  description, optional notes, state, calendar due date, optional due
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
- **SC-002**: At least 90% of test participants can add details and move a new task to In Progress in
  60 seconds or less without guidance.
- **SC-003**: At least 95% of eligible reminders become visible to the user within 60 seconds of the
  configured time.
- **SC-004**: At least 90% of test participants can open or complete a task directly from a reminder on
  their first attempt.
- **SC-005**: In synchronization authorization testing, 100% of remote requests from another account or with a missing, invalid,
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
- **SC-012**: In offline acceptance tests without a Tasky account or valid sync credentials, 100% of
  local capture, read, edit, complete, copy, delete, search, review, and reminder-action scenarios
  succeed without account authentication or network requests after satisfying any enabled local app lock,
  and saved task changes survive an app restart.
- **SC-013**: In local-lock acceptance tests, 100% of background transitions lock Tasky immediately;
  locked notifications reveal no task details, and reminder Open and Complete actions require local
  unlock. Successful device-credential recovery preserves all local tasks without network access;
  failed or cancelled unlock or recovery reveals no protected content and changes no tasks.

## Assumptions

- The first release supports Android 10 and later and does not include an iOS, desktop, or web client.
- French is the only supported interface language in the first release.
- Users can use local tasks without an account; an individual account is required only for synchronization.
- Users who forget their account password cannot recover synchronization access in the first release;
  their existing local task data remains usable.
- Tasks have no parent, child, predecessor, successor, dependency, grouping, or inferred relationship in
  this feature.
- Reminder delivery depends on user-granted notification permission and device availability; missed
  reminders remain visible inside the application. Previously scheduled reminders can appear offline
  with generic wording only; opening or completing their tasks works locally without account authentication,
  after successful local unlock if Tasky is locked.
- A stable source of current time is available for reminder scheduling and task history.
- Application launch and local task operations do not contact the account API or require a JWT.
  Only synchronization requires connectivity, valid credentials, and current account-status validation.
  Expired credentials require password reauthentication when sync is next requested; local use continues.
- The backend retains no server-side session; each request is authorized solely from its presented
  credentials and current account data.
- The first release has no formal availability or authentication-latency objective for the account API;
  outages fail closed for synchronization while local task use remains available.
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
- Account deletion soft-deletes remote task data without a defined retention period. Recovery, sharing, export,
  retention duration, and permanent erasure are deferred to a future professional feature.
  Existing local tasks remain usable on every device after account deletion; local task deletion is a
  separate user action governed by FR-016.

### Scope Boundaries

- Included: offline personal task management without sign-in, GTD classification and review, task reminders, task history,
  recoverable deletion, account privacy, and user-triggered two-way synchronization across the user's
  registered devices.
- Excluded from this feature: calendar or email integrations, file attachments, chat, organizational
  administration, projects, task grouping, task dependencies, task sequencing, other task relationships,
  shared task spaces, task collection sharing or transfer between accounts, task assignment,
  enterprise single sign-on, billing,
  location-triggered reminders, automated task creation from third-party services, and
  artificial-intelligence task decisions. Formal accessibility conformance requirements and acceptance
  testing, password-attempt throttling, password-policy hardening, forgotten-account-password recovery,
  unverified-account cleanup, post-restoration-window task disposition, reminder snoozing, and
  localization beyond French are deferred to a future release. Relationships between tasks belong to a
  separate future feature; authenticator-app TOTP and soft-deleted data recovery, sharing, export,
  retention, and erasure belong to a future professional-use iteration.
  Synchronizing a shared collection across accounts belongs to a separate future sharing feature.

### Dependencies

- Users must permit notifications to receive reminders outside the application.
- The device must provide a trustworthy local date and time for time-sensitive behavior.
- Accessing and changing tasks depends on the local Realm database, independently of the account API.
- Mobile storage follows the local-first Realm requirement in RULES.md.
- Signing in and renewing JWT credentials depends on connectivity to the Tasky account API.
- Manual two-way synchronization depends on connectivity to an authenticated remote service that stores
  the user's account-scoped synchronized data.
- New-device enrollment depends on delivery to the account's verified SMS-capable telephone number.
- Initial account activation depends on successful SMS verification and atomic first-device-key binding.
