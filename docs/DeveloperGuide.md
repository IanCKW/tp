---
layout: page
title: Developer Guide
---
* Table of Contents
{:toc}

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<img src="images/ArchitectureDiagram.png" width="450" />

The ***Architecture Diagram*** given above explains the high-level design of the App. Given below is a quick overview of each component.

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams in this document can be found in the [diagrams](https://github.com/se-edu/addressbook-level3/tree/master/docs/diagrams/) folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.

</div>

**`Main`** has two classes called [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java). It is responsible for,
* At app launch: Initializes the components in the correct sequence, and connects them up with each other.
* At shut down: Shuts down the components and invokes cleanup methods where necessary.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

The rest of the App consists of four components.

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

Each of the four components,

* defines its *API* in an `interface` with the same name as the Component.
* exposes its functionality using a concrete `{Component Name}Manager` class (which implements the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component (see the class diagram given below) defines its API in the `Logic.java` interface and exposes its functionality using the `LogicManager.java` class which implements the `Logic` interface.

![Class Diagram of the Logic Component](images/LogicClassDiagram.png)

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

The sections below give more details of each component.

### UI component

![Structure of the UI Component](images/UiClassDiagram.png)

**API** :
[`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class.

The `UI` component uses JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* Executes user commands using the `Logic` component.
* Listens for changes to `Model` data so that the UI can be updated with the modified data.

### Logic component

![Structure of the Logic Component](images/LogicClassDiagram.png)

**API** :
[`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

1. `Logic` uses the `AddressBookParser` class to parse the user command.
1. This results in a `Command` object which is executed by the `LogicManager`.
1. The command execution can affect the `Model` (e.g. adding a person).
1. The result of the command execution is encapsulated as a `CommandResult` object which is passed back to the `Ui`.
1. In addition, the `CommandResult` object can also instruct the `Ui` to perform certain actions, such as displaying help to the user.

Given below is the Sequence Diagram for interactions within the `Logic` component for the `execute("delete 1")` API call.

![Interactions Inside the Logic Component for the `delete 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.
</div>

### Model component

![Structure of the Model Component](images/ModelClassDiagram.png)

**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

The `Model`,

* stores a `UserPref` object that represents the user’s preferences.
* stores the address book data.
* exposes an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* does not depend on any of the other three components.


<div markdown="span" class="alert alert-info">:information_source: **Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique `Tag`, instead of each `Person` needing their own `Tag` object.<br>
![BetterModelClassDiagram](images/BetterModelClassDiagram.png)

</div>


### Storage component

![Structure of the Storage Component](images/StorageClassDiagram.png)

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

The `Storage` component,
* can save `UserPref` objects in json format and read it back.
* can save the address book data in json format and read it back.

### Common classes

Classes used by multiple components are in the `seedu.addressbook.commons` package.

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

![UndoRedoState0](images/UndoRedoState0.png)

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

![UndoRedoState1](images/UndoRedoState1.png)

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

![UndoRedoState2](images/UndoRedoState2.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</div>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

![UndoRedoState3](images/UndoRedoState3.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</div>

The following sequence diagram shows how the undo operation works:

![UndoSequenceDiagram](images/UndoSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</div>

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</div>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

![UndoRedoState4](images/UndoRedoState4.png)

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

![UndoRedoState5](images/UndoRedoState5.png)

The following activity diagram summarizes what happens when a user executes a new command:

![CommitActivityDiagram](images/CommitActivityDiagram.png)

#### Design consideration:

##### Aspect: How undo & redo executes

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

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:

* has a need to manage a significant number of contacts, orders, menu items and inventory
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**: manage contacts, orders, menu items and inventory faster than a typical mouse/GUI driven app


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a...                        | I want to...                                               | So that I can...                                            |
| -------- | ------------------------------ | ---------------------------------------------------------- | ----------------------------------------------------------- |
| `* * *`  | new user                       | see usage instructions                                     | refer to instructions when I forget how to use the App      |
| `* * *`  | fast typer                     | be able to input by CLI                                    | key in commands faster                                      |
| `* * *`  | restaurant owner               | add a customer's contact                                   | keep track of each customer's details                       |
| `* * *`  | restaurant owner               | remove a customer's contact                                | remove customers who no longer patronize the restaurant     |
| `* * *`  | restaurant owner               | add dishes to the menu                                     | keep track of dishes being offered                          |
| `* * *`  | restaurant owner               | remove dishes from the menu                                | remove dishes that are not being offered anymore            |
| `* * *`  | restaurant owner               | add food orders to the order list                          | keep track of the food I need to prepare                    |
| `* * *`  | restaurant owner               | remove food orders from the order list                     | remove the order if my customers changed their minds        |
| `* * *`  | restaurant owner               | add the ingredients that I have restocked to the inventory | know which ingredients I have in stock                      |
| `* * *`  | restaurant owner               | remove ingredients from the food inventory                 | remove an ingredient I have just used                       |
| `* *`    | restaurant owner               | edit a customer's contact                                  | rectify typos for customer errors                           |
| `* *`    | user with many contacts        | find a customer's contact                                  | quickly locate the contact details of a particular customer |
| `* *`    | owner with a large menu        | find a dish on the menu                                    | quickly locate details of a dish on the menu                |
| `* *`    | owner of a busy restaurant     | find a food order from the order list                      | quickly locate the details of an order I'm working on       |
| `* *`    | owner with a complex inventory | find the quantity of an ingredient in the food inventory   | quickly check how much of a certain ingredient I have left  |

### Use cases

(For all use cases below, the **System** is the `AddressBook` and the **Actor** is the `user`, unless specified otherwise)



**Use case: Request help**

**MSS**

1.  User requests help
2.  JJIMY displays a list of commands

    Use case ends.

**Use case: Exit**

**MSS**

1.  User requests to exit
2.  JJIMY exits

    Use case ends.

**Use case: Add a contact**

**MSS**

1.  User requests to add a contact
2.  JJIMY adds the contact

**Extensions**

*1a. JIMMY detects duplicate
 *1a1. JIMMY shows an error message
	Use case ends

    Use case ends.

**Use case: List all contacts**

**MSS**

1.  User requests to list contacts
2.  JJIMY shows a list of contacts

    Use case ends.


**Use case: Delete a contact**

**MSS**

1.  User requests to list contacts
2.  JJIMY shows a list of contacts
3.  User requests to delete a specific contact in the list
4.  JJIMY deletes the contact

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. JJIMY shows an error message.

      Use case resumes at step 2.



**Use case: Find a contact**

1. User requests to list contacts
2. JJIMY shows a list of contacts
3. User requests to find contacts based on keywords.
4. JJIMY returns a list of matching contacts for the keywords.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given keywords do not match any contacts.

    * 3a1. JJIMY shows an error message.

      Use case resumes at step 2.


**Use case: Add a menu item**

**MSS**

1.  User requests to add a menu item
2.  JJIMY adds the menu item

**Extensinos**

*1a. JIMMY detects duplicate
 *1a1. JIMMY shows an error message
	Use case ends

    Use case ends.

**Use case: List all contacts**

**MSS**

1.  User requests to list menu items
2.  JJIMY shows a list of menu items

    Use case ends.


**Use case: Delete a contact**

**MSS**

1.  User requests to list menu items
2.  JJIMY shows a list of menu items
3.  User requests to delete a specific menu item in the list
4.  JJIMY deletes the menu item

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. JJIMY shows an error message.

      Use case resumes at step 2.



**Use case: Find a menu item**

1. User requests to list menu items
2. JJIMY shows a list of menu items
3. User requests to find menu items based on keywords.
4. JJIMY returns a list of matching menu items for the keywords.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given keywords do not match any menu item.

    * 3a1. JJIMY shows an error message.

      Use case resumes at step 2.



**Use case: Add an order**

**MSS**

1.  User requests to add an order
2.  JJIMY adds the order

**Extensions**

*1a. JIMMY detects duplicate
 *1a1. JIMMY shows an error message
	Use case ends

    Use case ends.

**Use case: List all orders**

**MSS**

1.  User requests to list orders
2.  JJIMY shows a list of orders

    Use case ends.


**Use case: Delete an order**

**MSS**

1.  User requests to list orders
2.  JJIMY shows a list of orders
3.  User requests to delete a specific order in the list
4.  JJIMY deletes the order

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. JJIMY shows an error message.

      Use case resumes at step 2.



**Use case: Find a order**

1. User requests to list orders
2. JJIMY shows a list of orders
3. User requests to find orders based on keywords.
4. JJIMY returns a list of matching orders for the keywords.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given keywords do not match any order.

    * 3a1. JJIMY shows an error message.

      Use case resumes at step 2.



**Use case: Add an inventory item**

**MSS**

1.  User requests to add an inventory item
2.  JJIMY adds the inventory item

**Extensions**

*1a. JIMMY detects duplicate
 *1a1. JIMMY shows an error message
	Use case ends

    Use case ends.

**Use case: List all inventory items**

**MSS**

1.  User requests to list all inventory items
2.  JJIMY shows a list of all inventory items

    Use case ends.


**Use case: Delete an order**

**MSS**

1.  User requests to list all inventory items
2.  JJIMY shows a list of all inventory items
3.  User requests to delete a specific inventory item in the list
4.  JJIMY deletes the inventory item

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. JJIMY shows an error message.

      Use case resumes at step 2.



**Use case: Find a inventory item**

1. User requests to list all inventory items
2. JJIMY shows a list of all inventory items
3. User requests to find inventory items based on keywords.
4. JJIMY returns a list of matching inventory items for the keywords.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given keywords do not match any inventory item.

    * 3a1. JJIMY shows an error message.

      Use case resumes at step 2.


### Non-Functional Requirements

1.  Should work on any _mainstream OS_ as long as it has Java `11` or above installed.
2.  Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.

*{More to be added}*

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, OS-X
* **Private contact detail**: A contact detail that is not meant to be shared with others

