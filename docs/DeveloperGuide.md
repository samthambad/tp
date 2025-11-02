---
layout: page
title: Developer Guide
---
* Table of Contents
{:toc}

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

AI was used throughout the development of this project:
- GitHub Copilot was used for auto-completing code snippets.
- Claude Sonnet 4.5 was used to generate the unit tests.
- Claude Haiku 4.5 was used to review long documents and tool-use to ensure document consistency.
* This is a brownfield project based on the [AddressBook-Level3](https://se-education.org/addressbook-level3/) project developed by SE-EDU.

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams are in this document `docs/diagrams` folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.
</div>

### Architecture

<img src="images/ArchitectureDiagram.png" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<img src="images/ComponentManagers.png" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

![Structure of the UI Component](images/UiClassDiagram.png)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<img src="images/LogicClassDiagram.png" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

![Interactions Inside the Logic Component for the `delete 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</div>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<img src="images/ParserClasses.png" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<img src="images/ModelClassDiagram.png" width="450" />


The `Model` component,

* stores the application data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user's preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

#### Field Validation Constraints

Each `Person` object contains validated fields with the following constraints to prevent UI overflow and ensure data integrity:

| Field | Validation Rules | Max Length |
|-------|-----------------|------------|
| `Name` | Alphanumeric characters and spaces only. Cannot be blank. | 50 characters |
| `Phone` | Numeric digits only. Minimum 3 digits. | 20 digits |
| `Email` | Valid email format (local-part@domain). | 50 characters |
| `Address` | Any non-blank value. | No limit |
| `Tag` | Alphanumeric characters only. | 15 characters per tag |

These constraints are enforced at the model level in their respective classes (`Name`, `Phone`, `Email`, `Tag`) through the `isValid*()` methods.

<div markdown="span" class="alert alert-info">:information_source: **Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<img src="images/BetterModelClassDiagram.png" width="450" />

</div>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<img src="images/StorageClassDiagram.png" width="550" />

The `Storage` component,
* can save both application data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product Scope

**Target User Profile**:

* Needs to manage a large number of student contacts
* Prefers desktop applications over mobile/web
* Types quickly and efficiently
* Prefers keyboard input over mouse-driven interactions
* Comfortable using command-line style applications

**Value Proposition**:
Enable tutors and teachers to manage student contacts, notes, and tuition records **faster and more efficiently** than traditional GUI-based apps.

### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​ | I want to …​                                                         | So that I can…​                     |
|----------|---------|----------------------------------------------------------------------|-------------------------------------|
| `* * *`  | tutor   | add a new student with details (name, contact, subject, hourly rate) | manage them in the system           |
| `* * *`  | tutor   | delete a student permanently                                         | reduce clutter                      |
| `* * *`  | tutor   | add a lesson with date, time, and location                           | track when and where I teach        |
| `* * *`  | tutor   | record fee payments                                                  | know who has paid                   |
| `* * *`  | tutor   | see outstanding payments                                             | follow up with students/parents     |
| `* * *`  | tutor   | search by student name                                               | quickly find their record           |
| `* *`    | tutor   | mark attendance for a lesson                                         | know if the student showed up       |
| `* *`    | tutor   | add tuition fees per lesson or per month                             | track income                        |
| `* *`    | tutor   | see a weekly schedule                                                | plan my teaching                    |
| `* *`    | tutor   | view a student’s details                                             | reach them easily                   |
| `*`      | tutor   | import student data from a spreadsheet                               | save time entering existing records |
| `*`      | tutor   | create groups of students                                            | manage group lessons                |
| `*`      | tutor   | assign different hourly rates to different students                  | reflect actual arrangements         |
| `*`      | tutor   | export a fee report for a particular time period eg. a month         | manage my finances better.          |

### Use cases

(For all use cases below, the **System** is the `StudentConnect` and the **Actor** is the `user`, unless specified otherwise)

**UC1: Add New Student**

**MSS**

1. Actor chooses to add a new student.
2. System requests student details (name, contact, subject, hourly rate).
3. Actor enters the details.
4. System saves the details and displays a success message.

    Use case ends.

**Extensions**

* 3a. System detects missing or invalid details.

    * 3a1. System requests correct details.

    * 3a2. Actor re-enters details.

      Use case resumes at step 4.

* *a. At any time, Actor cancels the operation.

    * *a1. System requests confirmation.

    * *a2. Actor confirms.

      Use case ends.

**UC2: Delete Student**

**MSS**

1. Actor selects a student to delete.
2. System requests confirmation.
3. Actor confirms deletion.
4. System deletes the student and shows a success message.

    Use case ends.

**Extensions**

* 2a. Actor cancels deletion.

    * 2a1. System closes deletion process.

      Use case ends.

**UC3: Add Lesson**

**MSS**

1. Actor chooses to add a new lesson.
2. System requests details (date, time, location, student).
3. Actor enters lesson details.
4. System saves the lesson and confirms creation.

    Use case ends.

**Extensions**

* 3a. System detects invalid/missing lesson details.

    * 3a1. System requests correction.

      Use case resumes at step 3.

**UC4: Mark Attendance**

**MSS**

1. Actor selects a lesson.
2. System displays the lesson and attendance options.
3. Actor marks student as present or absent.
4. System saves attendance record.

    Use case ends.

**Extensions**

* 3a. Actor changes attendance after marking.

    * 3a1. System updates the record accordingly.

      Use case resumes at step 4.

**UC5: Add Tuition Fees**

**MSS**

1. Actor selects student and chooses to add tuition fees.
2. System requests fee details (per lesson or per month).
3. Actor enters fee details.
4. System saves fee record.

    Use case ends.

**Extensions**

* 3a. Invalid or incomplete fee details.

    * 3a1. System prompts for correction.

      Use case resumes at step 3.

**UC6: Record Fee Payment**

**MSS**

1. System shows students with outstanding fees.
2. Actor selects a student with outstanding fees.
3. Actor enters payment details (amount, date, method).
4. System records payment and updates balance.

    Use case ends.

**Extensions**

* 2a. Payment is made for student who does not owe any fees.

    * 2a1. System displays error, showing that selected student does not owe.

* 3a. Payment details invalid.

    * 3a1. System prompts for correction.

      Use case resumes at step 3.

**UC7: View Outstanding Payments**

**MSS**

1. Actor chooses to view outstanding payments.
2. System retrieves and displays a list of unpaid fees by student.

    Use case ends.

**Extensions**

* 2a. No outstanding payments.

    * 2a1. System displays “No pending fees.”

      Use case ends.

**UC8: View Schedule**

**MSS**

1. Actor chooses to view schedule.
2. System displays daily/weekly schedule of lessons.

    Use case ends.

**Extensions**

* 2a. No lessons scheduled.

    * 2a1. System displays “No lessons scheduled.”

      Use case ends.

**UC9: Search Student**

**MSS**

1. Actor enters student name into search bar.
2. System searches for matching student records.
3. System displays results.

    Use case ends.

**Extensions**

* 2a. No matching student found.

    * 2a1. System displays “No student found.”

      Use case ends.

**UC10: View Student Details**

**MSS**

1. Actor selects a student.
2. System displays student details (name, contact, subject, fees, history).

    Use case ends.

**Extensions**

* 2a. Student record is missing/corrupted.

    * 2a1. System displays error message.

      Use case ends.

**UC11: View Payment History**

**MSS**

1. Actor chooses to view all recorded payments.
2. System gets payment entries from all students.
3. System sorts the payments by date, newest first.
4. System displays the payment group by dates and shows a total paid amount.

Use case ends.

**Extensions**

* 2a. No Payment record exist.

    * 2a1. System displays “No payments recorded.”
  
      Use case ends.

* 2b. Some entries are linked to students that were deleted or corrupted.

    * 2b1. System skips invalid entries and logs a warning.
    * 2b2. System displays remaining valid entries.
  
      Use case ends.

### Non-Functional Requirements

1.  Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2.  Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
4. The interface should follow familiar patterns (e.g., similar to phone contacts apps) with clear navigation, searchable fields, and minimal clicks to add/view a student. Aim for a learnability time of under 5 minutes for new users.
5. Graceful handling of failures (e.g., invalid email entry) with user-friendly messages, plus backend logs for quick debugging.
6. The data should be stored locally and should be in a human editable text file.
7. Package everything into a single JAR file.

### Glossary

* **Student**: A person receiving tuition from the tutor.
* **Lesson**: A scheduled teaching session between tutor and one or more students.
* **Attendance**: A record of whether a student was present or absent for a lesson.
* **Progress Tracking**: A feature that allows tutors to note student learning milestones and monitor improvement.
* **Outstanding Fees**: Tuition fees owed by a student that are not yet paid.
* **MVP (Minimum Viable Product)**: The smallest functional version of StudentConnect that delivers core value to users.

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<div markdown="span" class="alert alert-info">:information_source: **Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</div>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

### Deleting a person

1. Deleting a person while all persons are being shown

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

### Adding a person
1. Adding a person with all fields specified

   1. Test case: `add n/John Doe p/98765432 e/johnd@example.com addr/John street, block 123, #01-01`
      Expected: Person is added to the list.
    
   2. Test case: `add n/Bob Dylan p/99765432 e/bobd@example.com addr/Bo street, block 13, #01-31 tag/friend`
      Expected: Person is added to the list with a tag.
    
