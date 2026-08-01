In **Godot**, a **signal** is a messaging system that lets nodes **send out notifications** that other nodes can listen for and respond to. It's like saying:

> "Hey! Something just happened!"

### Example:

Let’s say you have a button, and you want something to happen when it's pressed. You don’t constantly check if it was clicked — instead, the button **emits a signal**, and your script listens for it.

---

### How it works:

- **Emit signal**: A node triggers an event (like `pressed`, `body_entered`, etc.).
    
- **Connect signal**: Another node connects to that signal and defines what to do when it happens.
    

---

### Built-in signals:

Most nodes come with built-in signals.  
Examples:

- `Button`: `pressed`
    
- `Area2D`: `body_entered`, `area_entered`
    
- `Timer`: `timeout`
    

---

### Custom signals:

You can also make your own:

gdscript

CopyEdit

`signal picked_up_loot  func _on_player_loot_item():     emit_signal("picked_up_loot")`

Another node can then **connect** to `picked_up_loot` and react to it.

---

### Connecting signals (two ways):

**1. In the editor (easy for UI):**

- Select the node.
    
- Go to the **Node** tab next to the Inspector.
    
- Double-click a signal.
    
- Choose the target node + method to connect.
    

**2. In code:**

gdscript

CopyEdit

`button.connect("pressed", self, "_on_button_pressed")`

---

### TL;DR:

**Signals = event system.**  
They help different parts of your game talk to each other without being tightly linked. Super useful for clean and flexible code.

Want a real example based on your roguelike game?