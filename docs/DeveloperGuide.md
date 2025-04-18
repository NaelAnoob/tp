---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# BookVault Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

_{ list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well }_

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

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

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

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

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the library user data i.e., all `Person` objects (which are contained in a `UniquePersonList` object) and all `Book` objects.
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* contains logic to track overdue fees and the borrowing status of books.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<box type="info" seamless>

**Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

<puml src="diagrams/UndoRedoState0.puml" alt="UndoRedoState0" />

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

<puml src="diagrams/UndoRedoState1.puml" alt="UndoRedoState1" />

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

<puml src="diagrams/UndoRedoState2.puml" alt="UndoRedoState2" />

<box type="info" seamless>

**Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</box>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

<puml src="diagrams/UndoRedoState3.puml" alt="UndoRedoState3" />


<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</box>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

<puml src="diagrams/UndoSequenceDiagram-Logic.puml" alt="UndoSequenceDiagram-Logic" />

<box type="info" seamless>

**Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</box>

Similarly, how an undo operation goes through the `Model` component is shown below:

<puml src="diagrams/UndoSequenceDiagram-Model.puml" alt="UndoSequenceDiagram-Model" />

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</box>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

<puml src="diagrams/UndoRedoState4.puml" alt="UndoRedoState4" />

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

<puml src="diagrams/UndoRedoState5.puml" alt="UndoRedoState5" />

The following activity diagram summarizes what happens when a user executes a new command:

<puml src="diagrams/CommitActivityDiagram.puml" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

### \[Proposed\] Data archiving

_{Explain here how the data archiving feature will be implemented}_


--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:
* Name: Ryan Tan
* Age 40
* Gender Male
* Job Role: Senior Librarian at a mid-sized public library
* Team Size: 5
* Patron Base: 500-1000 active library users
* Pain Points:
  * Manual record-keeping is inefficient, relying on spreadsheets or paper logs for tracking borrowed books.
  * Difficulties in monitoring overdue books, leading to unreturned books and revenue loss from unpaid fines.
  * Limited visibility into user borrowing history, making it hard to provide services.
  * Membership management is time-consuming, requiring frequent manual updates to renew, activate, or suspend accounts.
  * No centralized way to track book availability, resulting in frequent patron inquiries.
* Technical Comfort
  * Comfortable with using desktop applications.
  * Familiar with basic command-line interfaces.
  * No programming experience.
* Work Style
  * Prefers a straightforward, easy-to-use system.
  * Multitasks daily, handling book check-ins, assisting patrons, and managing overdue accounts.
  * Needs quick access to information during peak hours when handling multiple requests.
* Goals
  * Improve efficiency in managing books.
  * Enhance the patron experience by quickly checking borrowing history and book availability.
  * Streamline membership management by automating activations, renewals, and suspensions.
  * Centralize book inventory and user records for a more organized, real-time system.

### Value Proposition
A streamlined library management system designed for librarians to track book loans,
overdue fines, and membership status in a fast and efficient way. Real-time updates, centralized records, and keyboard-friendly navigation ensure that Emma and her team can manage patrons
and books seamlessly without relying on scattered spreadsheets or outdated systems.


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​   | I want to …​                                                  | So that I can…​                                               |
|--------|-----------|---------------------------------------------------------------|---------------------------------------------------------------|
| `* * *` | new user  | see usage instructions                                        | refer to instructions when I forget how to use the application |
| `* * *` | librarian | add a new library user                                        | register new users and allow them to borrow books             |
| `* * *` | librarian | delete the details of a library users                         | remove users that no longer use the library                   |
| `* * ` | librarian | check the membership status of a user                         | cancel or renew memberships easily                            |
| `* *`  | librarian | issue a certain book to a certain user                        | lend books to users and keep track of them                    |
| `* *`  | librarian | edit/update details of a user                                 | keep the information accurate and up to date                  |
| `* *`  | librarian | mark that a user has returned a certain book                  | keep track of returned books                                  |
| `* *`  | librarian | let users extends their book return dates                     | ensure users have the option of avoiding overdue fees         |
| `* *`  | librarian | list users based on a filter                                  | look at users based on a certain criteria                     |
| `* *`  | librarian | check the due fees of a member                                | ensure members pay the correct fee                            |
| `*`    | librarian | see a list of overdue books and their corresponding users     | monitor the status of borrowed books                          |
| `* *`  | librarian | update book information                                       | ensure that book records remain accurate                      |
| `*`    | librarian | send automated reminders for overdue books or membership fees | keep members updates on overdue books/fees                    |

*{More to be added}*

### Use cases

(For all use cases below, the **BookVault** is the `AddressBook` and the **Actor** is the `user`, unless specified otherwise)

**Use case: Delete a person**

**MSS**

1.  User requests to list persons
2.  AddressBook shows a list of persons
3.  User requests to delete a specific person in the list
4.  AddressBook deletes the person

    Use case ends.

**Extensions**

* 2a. No persons exist in AddressBook.
  * 2a1. AddressBook shows an error message.

    Use case ends.

* 3a. The given index is invalid.

  * 3a1. AddressBook shows an error message.

    Use case resumes at step 2.

**Use case: Add a person**

**MSS**
1.  User requests to add a person with specific details
2.  AddressBook adds the person to the list

    Use case ends.

**Extensions**
* 1a. The provided details are incomplete or invalid.
    * 1a1. AddressBook shows an error message. Use case resumes at step 1.

* 1b. The person already exists in the AddressBook.
    * 1b1. AddressBook shows an error message. Use case resumes at step 1.

**Use case: Add a book**

**MSS**
1.  User requests to add a book with specific details
2.  AddressBook adds a book to the list

    Use case ends.

**Extensions**
* 1a. The provided details are incomplete or invalid.
    * 1a1. AddressBook shows an error message.

      Use case resumes at step 1.

* 1b. The book already exists in the AddressBook.
    * 1b1. AddressBook shows an error message.

      Use case resumes at step 1.



**Use case: Lend a book to a person**

**MSS**
1. User requests to list all persons
2. AddressBook shows list of all persons
3. User requests to list all books
4. AddressBook shows list of all books
5. User requests to lend a specific book to a specific person
6. AddressBook marks book as borrowed
7. AddressBook marks person as having borrowed the specific book

   Use case ends.

**Extensions**

* 2a. No persons exist in the AddressBook.
  * 2a1. AddressBook shows an error message.

    Use case ends.

* 4a. No books exist in AddressBook.
  * 4a1. AddressBook shows an error message.

    Use case ends.

* 5a. Given index for person is invalid.

  * 5a1. AddressBook shows an error message.

    Use case resumes at step 2.

* 5b. Given index for book is invalid.

    * 5b1. AddressBook shows an error message.

      Use case resumes at step 4.



*{More to be added}*
### Non-Functional Requirements

#### Performance:
1.  Should be able to hold up to 1000 persons and 10,000 book entries without a noticeable sluggishness in performance for typical usage.
2.  Should be able to handle up to 1000 commands per hour without noticeable sluggishness in performance for typical usage.
3.  Search operations (e.g., searching for books, members) should return results within 2 second for up to 1000 results.

#### Usability:
1.  Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
3.  The app should have an intuitive UI so that new librarians can start using it within 30 minutes.
4. The system should provide clear error messages and tooltips to guide users.

### Glossary

* **Library Management System (LMS)**: A software solution designed to help librarians manage book loans, overdue fines, and membership status efficiently.  
* **Patron**: A registered library user who can borrow books and use library services.  
* **Membership Status**: The current state of a patron's library account, which can be active, expired, or suspended.
* **Book Loan**: The process of borrowing a book from the library for a specified duration.
* **Overdue Book**: A book that has not been returned by its due date.  
* **Overdue Fine**: A penalty charged to a patron for not returning a book on time.  
* **Check-in**: The process of returning a borrowed book to the library.  
* **Check-out**: The process of lending a book to a patron.  
* **Book Availability**: The current status of a book, indicating whether it is available for borrowing or checked out by a patron.  
* **Borrowing History**: A record of all books a patron has borrowed in the past.  
* **Renewal**: Extending the borrowing period of a book before the due date.  
* **Suspension**: Temporarily restricting a patron’s ability to borrow books due to unpaid fines or violations of library policies.  
 

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**
Given below are instructions to test the app manually.
<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</box>

### Launch and shutdown
1. Initial launch
   1. Download the jar file and copy into an empty folder

   2. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

2. Saving window preferences
   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   2. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

3. _{ more test cases …​ }_

### Deleting a person

1. Deleting a person while all persons are being shown

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   2. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   3. Test case: `delete 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   4. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

2. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

2. _{ more test cases …​ }_

## **Appendix: Effort**

This project involved moderate-to-high difficulty due to managing multiple entity types (books, users, loans), compared to AB3's single entity (Person). Team members contributed to custom parsing logic, dual-entity linking (user↔book), and membership tracking.

### Challenges:
- Designing flexible command parsers for new command types.
- Ensuring data consistency across complex operations like `issue`, `return`.
- Handling JSON serialization of nested data.

### Reused Components:
- JSON storage structure from AB3, extended for new entity classes.
- UI base layout and controller patterns from AB3.

### Effort Highlights:
- Book and user command logic: ~40%
- Command parsing: ~20%
- Testing: ~5%
- Model/storage integration: ~20%
- UI enhancements: ~15%

## **Appendix: Planned Enhancements**

**Team size: 5**
1. **Improve quality of output messages example dates and fines**
   *Current:* Dates are printed numerically `20-04-2025` and fines are printed without currency example`15`  
   *Planned:* Use more user-friendly format such as `20th April, 2025` and `owes 15$`

2. **Implement command to fetch user index**
   *Current:* Delete command uses `index` to delete user.   
   *Planned:* Implement command to fetch user `index` or modify existing command to use `email`.

3. **Implement membership fees and tracking**  
   *Current:* Membership status is hard coded during insertion 
   *Planned:* Track membership status using dates and implement renew membership command.

4. **Warn before deleting user with loans**  
   *Current:* Deletes silently  
   *Planned:* Prompt: `User still has borrowed books. Confirm delete?`

5. **Implement feature for automated reminders**  
    *Current:* Users not notified of overdue books/ outstanding fines.
    *Planned:* Notify users by sending an email.



