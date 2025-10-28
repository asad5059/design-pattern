## Definition
Command is a behavioral design pattern that turns a request into a stand-alone object that contains all information about the request. This transformation lets you pass requests as a method arguments, delay or queue a request’s execution, and support undoable operations.

## Problem
Imagine that you’re working on a new text-editor app. Your current task is to create a toolbar with a bunch of buttons for various operations of the editor. You created a very neat `Button` class that can be used for buttons on the toolbar, as well as for generic buttons in various dialogs.

<img width="230" height="260" alt="image" src="https://github.com/user-attachments/assets/28db0540-6bfb-4308-b45b-d1798ef3e498" />

While all of these buttons look similar, they’re all supposed to do different things. Where would you put the code for the various click handlers of these buttons? The simplest solution is to create tons of subclasses for each place where the button is used. These subclasses would contain the code that would have to be executed on a button click.
<img width="400" height="190" alt="image" src="https://github.com/user-attachments/assets/dceb6136-87fb-4224-b0d2-ef58111cc97d" />

Before long, you realize that this approach is deeply flawed. First, you have an enormous number of subclasses, and that would be okay if you weren’t risking breaking the code in these subclasses each time you modify the base `Button` class. Put simply, your GUI code has become awkwardly dependent on the volatile code of the business logic.

<img width="480" height="110" alt="image" src="https://github.com/user-attachments/assets/c7fc9359-d65d-4be7-98c1-d9b239180fd7" />


And here’s the ugliest part. Some operations, such as copying/pasting text, would need to be invoked from multiple places. For example, a user could click a small “Copy” button on the toolbar, or copy something via the context menu, or just hit `Ctrl+C` on the keyboard.

Initially, when our app only had the toolbar, it was okay to place the implementation of various operations into the button subclasses. In other words, having the code for copying text inside the CopyButton subclass was fine. But then, when you implement context menus, shortcuts, and other stuff, you have to either duplicate the operation’s code in many classes or make menus dependent on buttons, which is an even worse option.

## Solution

In the code it might look like this: a GUI object calls a method of a business logic object, passing it some arguments. This process is usually described as one object sending another a request.

<img width="470" height="190" alt="image" src="https://github.com/user-attachments/assets/1f9ce728-b6d1-4948-9f24-31aec8a85510" />

The Command pattern suggests that GUI objects shouldn’t send these requests directly. Instead, you should extract all of the request details, such as the object being called, the name of the method and the list of arguments into a separate command class with a single method that triggers this request.

Command objects serve as links between various GUI and business logic objects. From now on, the GUI object doesn’t need to know what business logic object will receive the request and how it’ll be processed. The GUI object just triggers the command, which handles all the details.

<img width="550" height="190" alt="image" src="https://github.com/user-attachments/assets/a3ab93ad-daa1-4cd8-b8ad-830f86559032" />

The next step is to make your commands implement the same interface. Usually it has just a single execution method that takes no parameters. This interface lets you use various commands with the same request sender, without coupling it to concrete classes of commands. As a bonus, now you can switch command objects linked to the sender, effectively changing the sender’s behavior at runtime.

You might have noticed one missing piece of the puzzle, which is the request parameters. A GUI object might have supplied the business-layer object with some parameters. Since the command execution method doesn’t have any parameters, how would we pass the request details to the receiver? It turns out the command should be either pre-configured with this data, or capable of getting it on its own.


<img width="440" height="240" alt="image" src="https://github.com/user-attachments/assets/4f31306d-3f3d-49c6-9742-e1c9b3a9cea6" />

Let’s get back to our text editor. After we apply the Command pattern, we no longer need all those button subclasses to implement various click behaviors. It’s enough to put a single field into the base Button class that stores a reference to a command object and make the button execute that command on a click.

You’ll implement a bunch of command classes for every possible operation and link them with particular buttons, depending on the buttons’ intended behavior.

Other GUI elements, such as menus, shortcuts or entire dialogs, can be implemented in the same way. They’ll be linked to a command which gets executed when a user interacts with the GUI element. As you’ve probably guessed by now, the elements related to the same operations will be linked to the same commands, preventing any code duplication.

As a result, commands become a convenient middle layer that reduces coupling between the GUI and business logic layers. And that’s only a fraction of the benefits that the Command pattern can offer!

<img width="440" height="240" alt="image" src="https://github.com/user-attachments/assets/9ca1a889-20c7-46c3-b3c3-3b50e344b6eb" />

et’s get back to our text editor. After we apply the Command pattern, we no longer need all those button subclasses to implement various click behaviors. It’s enough to put a single field into the base Button class that stores a reference to a command object and make the button execute that command on a click.

You’ll implement a bunch of command classes for every possible operation and link them with particular buttons, depending on the buttons’ intended behavior.

Other GUI elements, such as menus, shortcuts or entire dialogs, can be implemented in the same way. They’ll be linked to a command which gets executed when a user interacts with the GUI element. As you’ve probably guessed by now, the elements related to the same operations will be linked to the same commands, preventing any code duplication.

As a result, commands become a convenient middle layer that reduces coupling between the GUI and business logic layers. And that’s only a fraction of the benefits that the Command pattern can offer!

## Example

```python
from abc import ABC, abstractmethod

# 1. The Receiver
# This is the object that will do the actual work.
class Editor:
    """
    The Receiver class. It has the actual business logic to perform
    on the text.
    """
    def __init__(self):
        self.text = "Hello World"
        # We simulate a "selection" for the demo
        self.selection_start = 6
        self.selection_end = 11  # This selects "World"

    def get_selection(self) -> str:
        """Gets the currently selected text."""
        return self.text[self.selection_start:self.selection_end]

    def delete_selection(self):
        """Deletes the selected text."""
        prefix = self.text[:self.selection_start]
        suffix = self.text[self.selection_end:]
        self.text = prefix + suffix
        # Move cursor to where selection was
        self.selection_end = self.selection_start
        print(f"Editor: Deleted selection. Text is now: '{self.text}'")

    def replace_selection(self, text_to_paste: str):
        """Replaces the selection with text from the clipboard."""
        prefix = self.text[:self.selection_start]
        suffix = self.text[self.selection_end:]
        self.text = prefix + text_to_paste + suffix
        # Move cursor to the end of the pasted text
        self.selection_start = self.selection_start + len(text_to_paste)
        self.selection_end = self.selection_start
        print(f"Editor: Pasted '{text_to_paste}'. Text is now: '{self.text}'")


# 2. The Command Interface and Base Class
class Command(ABC):
    """
    The base Command class defines the common interface and
    stores the app, editor, and backup state.
    """
    def __init__(self, app: 'Application', editor: Editor):
        self.app = app
        self.editor = editor
        self.backup = ""

    def save_backup(self):
        """Make a backup of the editor's state."""
        self.backup = self.editor.text

    def undo(self):
        """Restore the editor's state from backup."""
        self.editor.text = self.backup

    @abstractmethod
    def execute(self) -> bool:
        """
        The main method of the command.
        It must return True or False to indicate if the command
        should be saved in the history.
        """
        pass


# 3. Concrete Commands
class CopyCommand(Command):
    """
    Copy command doesn't change the editor's state,
    so it won't be saved to history.
    """
    def execute(self) -> bool:
        self.app.clipboard = self.editor.get_selection()
        print("Command: Copied to clipboard.")
        return False  # Return False: don't save to history

class CutCommand(Command):
    """
    Cut command changes the editor's state,
    so it must be saved to history.
    """
    def execute(self) -> bool:
        self.save_backup()  # Save state before changing it
        self.app.clipboard = self.editor.get_selection()
        self.editor.delete_selection()
        print("Command: Cut to clipboard.")
        return True  # Return True: save to history

class PasteCommand(Command):
    """
    Paste command also changes the editor's state.
    """
    def execute(self) -> bool:
        self.save_backup()  # Save state before changing it
        self.editor.replace_selection(self.app.clipboard)
        print("Command: Pasted from clipboard.")
        return True  # Return True: save to history

class UndoCommand(Command):
    """
    The Undo operation is also a command, triggered by the user.
    It tells the app to run its own undo logic.
    """
    def execute(self) -> bool:
        self.app.undo()
        print("Command: Executed Undo.")
        return False  # The undo action itself isn't saved to history


# 4. The Invoker (Sender)
class Application:
    """
    The Application acts as the Sender/Invoker. It creates commands
    and executes them. It also holds the command history.
    """
    def __init__(self):
        self.clipboard = ""
        self.editor = Editor()
        # We use a simple list as our command history stack
        self.history: list[Command] = []

    def execute_command(self, command: Command):
        """
A button click or shortcut would call this method.
        """
        print(f"\n--- Executing {type(command).__name__} ---")
        # The command returns True if it should be saved
        if command.execute():
            self.history.append(command)
            print("App: Saved command to history.")
        print(f"Current Text: '{self.editor.text}'")

    def undo(self):
        """
        Takes the most recent command from history and runs its
        undo method.
        """
        if not self.history:
            print("App: History is empty. Nothing to undo.")
            return
        
        # Pop the last command
        command = self.history.pop()
        
        if command:
            print(f"--- Undoing {type(command).__name__} ---")
            command.undo()
            print(f"Current Text: '{self.editor.text}'")


# 5. The Client (How it's all used)
if __name__ == "__main__":
    # Setup
    app = Application()

    print(f"Initial Text: '{app.editor.text}'")
    print(f"(Current 'selection' is '{app.editor.get_selection()}')")

    # 1. User cuts "World"
    cut = CutCommand(app, app.editor)
    app.execute_command(cut)
    # Text is now: "Hello "
    # Clipboard is: "World"
    # History has: [CutCommand]

    # 2. User pastes "World"
    paste = PasteCommand(app, app.editor)
    app.execute_command(paste)
    # Text is now: "Hello World"
    # History has: [CutCommand, PasteCommand]

    # 3. User copies (selection is now empty)
    copy = CopyCommand(app, app.editor)
    app.execute_command(copy)
    # Text is unchanged
    # Clipboard is: ""
    # History is unchanged: [CutCommand, PasteCommand]

    # 4. User presses the "Undo" button
    undo = UndoCommand(app, app.editor)
    app.execute_command(undo)
    # This undoes the PasteCommand
    # Text is now: "Hello "
    # History has: [CutCommand]

    # 5. User presses "Undo" again
    app.execute_command(undo)
    # This undoes the CutCommand
    # Text is now: "Hello World"
    # History is empty
    
    # 6. User presses "Undo" again
    app.execute_command(undo)
    # Prints: "History is empty. Nothing to undo."
```

## Pros
1. Decoupling: It decouples the invoker from the receiver. The invoker doesn't need to know anything about the concrete operations or the objects performing them.
2. Undo/Redo: This is one of its most powerful benefits. By storing commands in a history, and by each command knowing how to reverse itself, undo/redo functionality becomes relatively straightforward.
3. Queueing & Logging: Commands can be queued (for asynchronous execution or batch processing) or logged (for persistence or recovery) without affecting the invoker or receiver.
4. Extensibility: We can add new commands without changing existing invoker or receiver classes.

Cons
1. Increased Complexity: For very simple operations, introducing command objects can add unnecessary layers of abstraction and boilerplate code.
2. More Classes: Every distinct action typically requires its own concrete command class, potentially leading to a large number of classes in a complex system.
3. State Management for Undo: Implementing a robust undo() method for every command can be challenging, especially when commands affect multiple parts of a system or involve complex state changes. Each command needs to store sufficient information to revert its action.
