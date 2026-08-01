# GDScript in Modules: Beginner → Novice, Step by Step
*Same structure as the Python curriculum — work top to bottom, don't skip. Each module ends with a checkpoint. Where GDScript genuinely works differently from Python (and it does, in some important places), that's called out explicitly rather than glossed over.*

---

## How to use this
- Open Godot (4.7.1 is current stable), create a new project, and just work inside it — attach scripts to nodes as you go rather than writing loose files.
- Type every example yourself.
- Keep the built-in **Output** panel visible at the bottom of the editor — `print()` writes there, and it's your main feedback loop.
- This assumes Godot is already installed, since you're already running it for your chess project.

---

## Module 0 — Orientation
**Goal:** understand what a GDScript file actually is inside Godot.

- Godot programs are built from **nodes** arranged in a **scene tree**. A script is attached to a node and gives that node behavior.
- Two lifecycle methods you'll see constantly:
```gdscript
extends Node

func _ready():
    print("this node just entered the scene tree")

func _process(delta):
    pass  # runs every rendered frame; delta = seconds since last frame
```
- `extends` at the top of a script says what built-in node type this script is attached to / extending — every script's capabilities start from whatever it extends.
- Unlike a plain Python script you run from a terminal, a GDScript file doesn't "run" on its own — it runs because it's attached to a node that exists in a scene, and that scene is running.

**Checkpoint:** create a new empty scene with a `Node2D` root, attach a script, and get `_ready()` to print something. Run the scene (F6) and confirm it shows in the Output panel.

---

## Module 1 — Variables & Types
**Concepts:** variables, dynamic *and* optional static typing.

```gdscript
var player_name = "u1"       # inferred as String
var level = 12                 # inferred as int
var height = 1.8                # inferred as float
var is_dev = true                 # bool
var nothing = null                 # like Python's None

print(typeof(level))    # prints a type-code int; better: print(level) and eyeball it
```
GDScript can be dynamic like Python (above), **or** statically typed — this is the biggest early difference from Python, and it's used constantly in real Godot code:
```gdscript
var player_name: String = "u1"
var level: int = 12
var height: float = 1.8
var is_dev: bool = true
```
Static typing here isn't optional boilerplate — it gets you editor autocomplete and catches type mistakes before you even run the scene. Prefer typed variables once you're past pure experimentation.

**Practice:** declare typed variables for a game character: `name` (String), `level` (int), `hp_percent` (float), `is_alive` (bool).

**Checkpoint:** what's the actual benefit of writing `var level: int = 12` instead of just `var level = 12`? (Think about what happens if you later accidentally assign `level = "twelve"`.)

---

## Module 2 — Operators & Expressions
```gdscript
7 + 3, 7 - 3, 7 * 3, 7 / 3     # int / int truncates like C#, NOT like Python's true division
7.0 / 3                         # use a float to get a float result: 2.333...
7 % 3                            # modulo: 1
pow(2, 5)                          # 32 — no ** operator, use pow()

5 == 5, 5 != 3, 5 > 3, 5 <= 5       # comparisons -> bool
true and false, true or false, not true

var x = 5
x += 1
x *= 2
```
**Important difference from Python:** `7 / 3` in GDScript with two ints gives `2` (integer division), not `2.333`. Python's `/` always gives a float; GDScript's doesn't unless at least one operand is a float. This trips people up constantly coming from Python.

**Practice:** given `var hp = 73`, write an expression that evaluates to `true` if hp is above 50% of a max of 100.

**Checkpoint:** what does `5 / 2` give you in GDScript, and what do you need to change to get `2.5`?

---

## Module 3 — Control Flow
```gdscript
var hp = 40

if hp <= 0:
    print("dead")
elif hp < 50:
    print("low health")
else:
    print("healthy")
```
Just like Python: **indentation defines the block**, no braces, no `;`. This is one of the more comfortable overlaps coming from Python.

**Truthy/falsy** works differently than Python — GDScript is stricter. `0`, `""`, empty arrays are generally still falsy in boolean contexts, but GDScript is more type-strict overall, so lean on explicit comparisons (`if hp > 0:`) rather than relying on implicit truthiness, especially once variables are typed.

**Practice:** write a script that takes a `score` variable and prints a letter grade using if/elif/else.

**Checkpoint:** rewrite one `if` check you wrote above to be explicit (`if health_potions > 0:` instead of `if health_potions:`) — get in the habit now, it'll save you debugging time later.

---

## Module 4 — Loops
```gdscript
for i in range(5):          # 0,1,2,3,4
    print(i)

for i in range(2, 10, 2):    # start, stop, step -> 2,4,6,8
    print(i)

var n = 3
while n > 0:
    print(n)
    n -= 1

for i in range(20):
    if i == 5:
        break
    if i % 2 == 0:
        continue
    print(i)
```
Directly parallel to Python's `for`/`while`/`break`/`continue` — this module should feel almost identical to what you already know.

**Godot-specific extra:** you'll very often loop over a node's children:
```gdscript
for child in get_children():
    print(child.name)
```

**Practice:** print all multiples of 3 between 1 and 50. Then loop over `get_children()` of any node with a few children and print each child's name.

**Checkpoint:** what does `get_children()` return, and why does looping over it matter so much more in GDScript than looping over arbitrary lists mattered in Python?

---

## Module 5 — Strings in Depth
```gdscript
var s = "Hello, u1"
s[0]                  # "H"
s.to_lower()
s.to_upper()
s.strip_edges()          # like Python's .strip()
s.split(", ")              # ["Hello", "u1"]
s.replace("u1", "u2")

var name = "u1"
var level = 12
print("%s is level %d" % [name, level])          # older-style formatting
print(str(name, " is level ", level))               # concatenation via str()

# GDScript 2.0+ also supports Python-style f-strings-like formatting via %:
print("%s: %d%%" % ["HP", 73])
```
**Difference from Python:** GDScript doesn't have f-strings in the Python sense — the idiomatic approach is `%`-style formatting (`"%s is level %d" % [name, level]`) or building with `str()`. It reads a bit more like older Python string formatting.

**Practice:** given `var sentence = "the quick brown fox"`, split it into words and print them joined with `"-"` instead of spaces (`.split()` + `"-".join(words)` — `join` exists on `String` in GDScript too, called as `"-".join(array)`).

**Checkpoint:** what's the GDScript equivalent of Python's `f"{name} is level {level}"`, and why might it feel a bit clunkier at first?

---

## Module 6 — Arrays
```gdscript
var scores = [10, 20, 30]
scores.append(40)
scores.insert(0, 5)
scores.erase(20)
scores.pop_back()
scores.sort()
scores.reverse()
scores.size()               # like Python's len()
30 in scores                  # membership test

scores.slice(1, 3)              # [20, 30]

# Typed arrays — used constantly in real Godot code
var names: Array[String] = ["a", "b", "c"]
```
An `Array` is GDScript's list — same mutable, ordered role as Python's `list`. GDScript's typed arrays (`Array[String]`) are worth adopting early; they catch mistakes the editor can flag before you even run anything.

**Practice:** build a typed array of 5 enemy names (`Array[String]`), add one, remove one, sort it, print the third element by index.

**Checkpoint:** what does declaring `Array[String]` buy you over a plain untyped `Array`?

---

## Module 7 — Dictionaries
```gdscript
var player = {"name": "u1", "level": 12, "hp": 80}
player["level"]
player["mana"] = 50
player.get("stamina", 0)         # safe lookup with default
player.has("hp")                   # membership check
player.erase("mana")

for key in player:
    print(key, player[key])
```
Directly parallel to Python's `dict` — same "key → value" role, same `.get()` safe-lookup pattern. One difference: iterating `for key in player:` gives you keys directly (similar to Python, but GDScript doesn't have a `.items()` you loop with two variables — you fetch the value yourself inside the loop with `player[key]`).

**Practice:** build a dict representing an inventory (item name → quantity). Add an item, increase a quantity, print every item and count.

**Checkpoint:** how would you print both the key and value together in one loop, given GDScript's dict iteration only hands you the key?

---

## Module 8 — Functions
```gdscript
func greet(name, greeting = "Hello"):
    return "%s, %s!" % [greeting, name]

greet("u1")
greet("u1", "Hey")

# typed version — the idiomatic Godot style
func greet_typed(name: String, greeting: String = "Hello") -> String:
    return "%s, %s!" % [greeting, name]

func total(numbers: Array) -> int:
    var sum = 0
    for n in numbers:
        sum += n
    return sum

var square = func(x): return x * x     # anonymous lambda function
```
No `*args`/`**kwargs` equivalent in GDScript — functions take a fixed parameter list (with optional defaults). If you need variable argument counts, you pass an `Array` explicitly. `-> String`/`-> int` after the parameter list is the return type hint — same spirit as Python's `-> str` hints, but GDScript checks it more meaningfully at compile time when used consistently.

**Practice:** write a typed function `calculate_damage(base: int, multiplier: float = 1.0, is_critical: bool = false) -> int` that returns `base * multiplier * (2 if is_critical else 1)`.

**Checkpoint:** why does GDScript not have a `*args`-style catch-all, and what would you do instead if you genuinely wanted a variable number of arguments?

---

## Module 9 — Functional-Style Array Methods
**Concepts:** the closest GDScript gets to Python's comprehensions.

```gdscript
var numbers = range(10)
var squares = numbers.map(func(x): return x * x)
var evens = numbers.filter(func(x): return x % 2 == 0)
var total = numbers.reduce(func(acc, x): return acc + x, 0)
```
GDScript has no comprehension syntax like `[x**2 for x in range(10)]` — instead you use `.map()`, `.filter()`, `.reduce()` with lambda functions (Godot 4+). Functionally equivalent, different shape. For anything non-trivial, a plain `for` loop is often more readable in GDScript than chaining these — don't force it just because Python trained you to reach for comprehensions.

**Practice:** given an array of words, use `.filter()` and `.map()` to get only words longer than 4 characters, uppercased.

**Checkpoint:** rewrite your `.filter()`/`.map()` chain above as a plain `for` loop — which one do you find more readable, and why might that differ from your instinct in Python?

---

## Module 10 — Error Handling
**This is the module with the biggest real difference from Python — read carefully.**

GDScript has **no `try`/`except`, no exceptions, no `raise`.** This is a deliberate design choice, not a missing feature — Godot favors explicit error checking over exception-based control flow.

```gdscript
func divide(a: float, b: float) -> float:
    if b == 0:
        push_error("Division by zero attempted")
        return 0.0
    return a / b

# Many built-in Godot functions return an error code instead of throwing:
var err = get_tree().change_scene_to_file("res://level2.tscn")
if err != OK:
    print("Scene change failed: ", err)

# assert() halts execution in debug builds if a condition fails — a dev-time sanity check,
# not a runtime error-handling tool (it's stripped out of release exports)
assert(hp >= 0, "HP should never go negative")
```
`push_error()` / `push_warning()` log to the Output panel (and the editor's debugger) without stopping execution — you check return values and handle problems with plain `if` statements, the same way C does. This is the opposite instinct from Python's "ask forgiveness" exception style — GDScript wants you to check first.

**Practice:** rewrite a function that divides two numbers, but instead of dividing by zero, use `push_error()` and return `0.0`, following the pattern above.

**Checkpoint:** in one sentence, why might a game engine prefer explicit error codes over exceptions during a 60-times-per-second game loop? (Hint: think about what happens to a frame if an unhandled exception propagates up mid-`_process()`.)

---

## Module 11 — Files & I/O
```gdscript
var file = FileAccess.open("user://save.txt", FileAccess.WRITE)
file.store_string("level=12\n")
file.close()

var read_file = FileAccess.open("user://save.txt", FileAccess.READ)
while not read_file.eof_reached():
    print(read_file.get_line())
read_file.close()
```
`FileAccess` is GDScript's file-handling class — no `with` context manager, you call `.close()` explicitly (though Godot does clean up automatically when the `FileAccess` object goes out of scope, calling `.close()` yourself is still the clear, intentional habit). `res://` refers to files bundled with your project; `user://` is the writable location for save data on the player's machine — this distinction matters a lot and has no real Python parallel.

```gdscript
var data = {"name": "u1", "level": 12}
var json_string = JSON.stringify(data)
var save_file = FileAccess.open("user://save.json", FileAccess.WRITE)
save_file.store_string(json_string)
save_file.close()

var load_file = FileAccess.open("user://save.json", FileAccess.READ)
var loaded = JSON.parse_string(load_file.get_as_text())
load_file.close()
```

**Practice:** save a dict of player stats to `user://save.json`, then read it back and print each stat.

**Checkpoint:** what's the difference between `res://` and `user://`, and why does a shipped game need both?

---

## Module 12 — Organizing Code: Scripts, class_name, Autoloads
```gdscript
# combat.gd
class_name Combat

static func calculate_damage(base: int, multiplier: float = 1.0) -> int:
    return int(base * multiplier)
```
```gdscript
# main.gd
func _ready():
    var dmg = Combat.calculate_damage(10, 2.0)
```
`class_name` makes a script globally accessible by that name, project-wide, without needing an explicit `import`/`preload` — this is a real difference from Python's `import`. For a script you *don't* want globally registered, you'd use `preload("res://combat.gd")` or `load(...)` instead, closer in spirit to Python's import.

**Autoloads** (Project Settings → Autoload) are Godot's singleton pattern — a script that's always loaded and globally accessible, commonly used for game state, save systems, or global signals — roughly the role a module-level global or a config singleton plays in a Python app.

**Practice:** split your damage-calculation function (Module 8) out into its own script with `class_name Combat`, and call it from another script.

**Checkpoint:** when would you reach for `class_name` versus `preload()`? What's the tradeoff of making everything globally named?

---

## Module 13 — Classes & Objects
```gdscript
class_name Character
extends Node

var character_name: String
var hp: int = 100

func _init(name: String, starting_hp: int = 100):
    character_name = name
    hp = starting_hp

func take_damage(amount: int) -> int:
    hp -= amount
    return hp

func _to_string() -> String:
    return "Character(%s, hp=%d)" % [character_name, hp]
```
`_init()` is GDScript's constructor — the direct equivalent of Python's `__init__`. Note that here `Character extends Node`, meaning it's meant to actually live in the scene tree (unlike a plain Python class with no such requirement). GDScript classes don't *have* to extend a Node — you can write a plain data class extending `RefCounted` if you just want an object with no scene presence:
```gdscript
class_name SaveData
extends RefCounted

var level: int
var hp: int
```

**Practice:** build a `Player` class (`class_name Player`, extending `RefCounted` if you don't need it in the scene tree) with `name`, `hp`, `level`, and a `level_up()` method.

**Checkpoint:** when would a class need to `extend Node` versus `extend RefCounted`? What does living in the scene tree actually get you?

---

## Module 14 — Inheritance & Polymorphism
```gdscript
class_name Mage
extends Character

var mana: int = 50

func _init(name: String, starting_mana: int = 50):
    super._init(name)          # calls Character._init — this is GDScript's super()
    mana = starting_mana

func cast(spell: String) -> String:
    mana -= 10
    return "%s casts %s" % [character_name, spell]

func take_damage(amount: int) -> int:      # override
    amount = amount / 2                      # mages take half damage
    return super.take_damage(amount)          # call the parent's version
```
`super.method_name()` is GDScript's equivalent of Python's `super().method_name()` — same purpose (call the parent class's version), slightly different syntax (no parentheses after `super`).

**Practice:** build a `Warrior` class extending your `Player`, add an `armor` attribute, override `level_up()` to also increase armor.

**Checkpoint:** what's the syntax difference between calling the parent constructor in GDScript (`super._init(...)`) versus Python (`super().__init__(...)`)?

---

## Module 15 — Signals (Godot's Core Communication Pattern)
**Concepts:** the idiomatic Godot way for objects to talk to each other without being tightly coupled — this has no direct Python-standard-library equivalent (closest conceptual cousin is C#'s events, if that helps from your other guide).

```gdscript
class_name Character
extends Node

signal took_damage(amount: int)
signal died

var hp: int = 100

func take_damage(amount: int):
    hp -= amount
    took_damage.emit(amount)
    if hp <= 0:
        died.emit()
```
Somewhere else — another node connects to it:
```gdscript
func _ready():
    var enemy = get_node("Enemy")
    enemy.took_damage.connect(_on_enemy_took_damage)
    enemy.died.connect(_on_enemy_died)

func _on_enemy_took_damage(amount: int):
    print("Enemy took %d damage" % amount)

func _on_enemy_died():
    print("Enemy defeated")
```
This is the pattern that lets a UI health bar, a sound effect, and a quest tracker all react to "character took damage" without any of them needing to know about each other — they just connect to the signal. This is arguably the single most important idiomatic-Godot concept in this whole curriculum, and it's the piece with zero Python-standard-library parallel.

**Practice:** add a `signal leveled_up(new_level)` to your `Player` class, emit it inside `level_up()`, and connect a print statement to it from another script.

**Checkpoint:** why is emitting a signal generally better here than having `Character` directly call methods on the health bar, sound player, and quest tracker itself?

---

## Module 16 — Nodes, Scenes & Composition
**Concepts:** the structural concept that replaces a chunk of what OOP composition does in plain Python.

```gdscript
# accessing other nodes
@onready var sprite = $Sprite2D          # $ is shorthand for get_node()
@onready var health_bar = $UI/HealthBar

func _ready():
    sprite.modulate = Color.RED
```
`@onready` defers the variable's assignment until the node is actually in the scene tree — needed because node references aren't valid until then. A scene (`.tscn`) is a saved tree of nodes, and it can be **instanced** like a template:
```gdscript
var enemy_scene = preload("res://enemy.tscn")
var enemy_instance = enemy_scene.instantiate()
get_tree().current_scene.add_child(enemy_instance)
```
This is roughly the closest thing to "creating an object from a class" you did in Module 13, but for whole prefab game objects rather than plain data classes — it's Godot's version of a factory/spawn pattern.

**Practice:** create a simple scene with two child nodes, reference both with `@onready` and `$`, and instance a second copy of the whole scene from a script at runtime.

**Checkpoint:** why does `@onready` matter — what would go wrong if you tried to access `$Sprite2D` in a plain `var sprite = $Sprite2D` declared outside `_ready()`?

---

## Module 17 — Built-in Types & Utility Functions
```gdscript
randi() % 6 + 1               # dice roll (1-6)
randf()                         # random float 0.0-1.0
randf_range(1.5, 4.5)
randomize()                      # seed from OS entropy — call once at game start

Vector2(3, 4)                     # built-in 2D vector type — .x, .y, .length(), .normalized()
Vector3(1, 2, 3)

Time.get_ticks_msec()               # milliseconds since engine start
str(level)                            # convert to String, like Python's str()
int("42")                              # parse a String to int
```
GDScript's "standard library" is really the whole Godot API surface — `Vector2`/`Vector3` especially are things you'll use constantly and have no real Python-standard-library parallel (you'd reach for a third-party library like `pygame.Vector2` for the closest equivalent).

**Practice:** write a script that rolls a d20 ten times and prints how many times each value 1-20 came up (a `Dictionary` works well as your tally).

**Checkpoint:** what's `randomize()` for, and what would happen across multiple runs of your game if you never called it?

---

## Module 18 — Testing & Debugging
GDScript has no `pytest`-equivalent built in — the community-standard addon is **GUT (Godot Unit Test)**, installed via the AssetLib panel inside the editor.
```gdscript
extends GutTest

func test_calculate_damage():
    assert_eq(Combat.calculate_damage(10, 2.0), 20)
```
**Debugging habits, Godot-specific:**
- `print()` still works exactly like you'd expect, shown in the Output panel.
- The built-in **Debugger** panel gives you real breakpoints — click the gutter next to a line number, run the scene, and execution pauses there with full variable inspection. This is a step up from anything you'd casually set up in a plain Python script.
- `push_warning()` / `push_error()` show up in the Debugger's Errors tab with a stack trace, distinct from `print()`.

**Practice:** install GUT, and write two test cases for your `Combat.calculate_damage()` function covering a normal case and a critical-hit case.

**Checkpoint:** what's the practical advantage of the Debugger's clickable breakpoints over scattering `print()` statements everywhere, especially in a `_process()` loop running 60 times a second?

---

## Module 19 — Static Typing, for Real This Time
You saw typed variables in Module 1 — now the full picture, since this matters more in GDScript than type hints do in Python:
```gdscript
func calculate_damage(base: int, multiplier: float = 1.0) -> int:
    return int(base * multiplier)

var enemies: Array[Character] = []
var stats: Dictionary = {}
```
Unlike Python (where type hints are pure documentation, unchecked at runtime), GDScript's static typing is **checked by the editor and at compile time** — a typed function called with the wrong argument type is flagged before you even hit Run, not discovered three functions deep in a traceback. This is a meaningfully bigger deal in GDScript than it was in your Python guide.

**Practice:** go back through your Module 13–15 classes and add full type hints to every variable and function signature.

**Checkpoint:** try deliberately passing a `String` where a typed function expects an `int` — when does Godot catch it, and how is that different from what happened when you tried the equivalent in Python?

---

## Module 20 — Capstone: Put It All Together
**Goal:** prove novice level by building something that touches most of the modules above — and that plugs directly into the kind of project you're already building.

Build a **small combat encounter scene**:
- A `Character` base class + a `Mage`/`Warrior` subclass (Modules 13–14), fully typed (Module 19)
- Signals for `took_damage` and `died` (Module 15), connected to a simple health-bar UI node
- A scene with the player and an enemy as separate instanced nodes (Module 16)
- Damage calculation using `push_error()`-style guarding instead of exceptions (Module 10)
- Save/load the player's stats to `user://save.json` (Module 11)
- Split into at least two scripts using `class_name` (Module 12)
- At least one GUT test for your damage function (Module 18)

This is deliberately shaped as a slice of the chess-boss-fight project you're already building — if you can build this without re-reading earlier modules, you're at novice, and the pattern scales directly into what you're working on.

---

## What "novice" unlocks next
Coroutines and `await` for async-style waiting (`await get_tree().create_timer(1.0).timeout`), the `AnimationPlayer`/`Tween` system for procedural motion, custom `Resource` types for structured save/config data, and shaders. Left out here on purpose — they build on this foundation rather than being part of it, same reasoning as the Python guide's "next steps" section.
