# How to Build "Dash Quest" From Scratch

### A beginner's guide to making a real 2D video game in Godot — even if you've never written a line of code before

---

## Before We Start

You are going to build a real video game: a side-scroller (a game where your character runs left and right across the screen, like an old Mario game) with 5 levels, enemies that can hurt you, extra lives to find, a main menu, a pause button, and a way to start over.

This is a big project. Professional game developers would take this seriously — so don't worry if it takes you a few afternoons instead of one sitting. That's normal. Save your work often (Ctrl+S), take breaks, and don't be afraid of red error text. Every single game programmer sees error messages all day long. They're not a sign you failed — they're the computer trying to help you find a typo.

**Good news:** a finished version of this exact game already lives in this same project folder. If you ever get totally stuck on a step, you're allowed to peek at the real files to see how they look. But try it yourself first — that's how you actually learn it.

### How This Guide Works

Watch for these little boxes as you read:

> **New Word:** explains a term you've never seen before.

> **Try It:** something to actually do right now, not just read.

> **Watch Out:** a mistake almost everyone makes here.

> **Why:** explains *why* we're doing something, not just *how*.

---

## Table of Contents

1. [What Are We Even Doing? (Meet Godot)](#module-1-what-are-we-even-doing-meet-godot)
2. [Installing Godot and Starting Your Project](#module-2-installing-godot-and-starting-your-project)
3. [A Tour of the Godot Editor](#module-3-a-tour-of-the-godot-editor)
4. [Learning to "Talk" to the Computer (GDScript Basics)](#module-4-learning-to-talk-to-the-computer-gdscript-basics)
5. [The Global Notebook (Autoload)](#module-5-the-global-notebook-autoload)
6. [Building Your Hero (The Player)](#module-6-building-your-hero-the-player)
7. [Making a Bad Guy (The Enemy)](#module-7-making-a-bad-guy-the-enemy)
8. [Coins and Flags (Extra Life + Goal)](#module-8-coins-and-flags-extra-life--goal)
9. [The Scoreboard (HUD)](#module-9-the-scoreboard-hud)
10. [Pause, Game Over, and Victory Screens](#module-10-pause-game-over-and-victory-screens)
11. [The Level Machine (loops, lists, and recipes)](#module-11-the-level-machine-loops-lists-and-recipes)
12. [Building All 5 Levels](#module-12-building-all-5-levels)
13. [Main Menu and Level Select](#module-13-main-menu-and-level-select)
14. [Final Wiring and Playtesting](#module-14-final-wiring-and-playtesting)
15. [Glossary (Look Up Any Word)](#glossary-look-up-any-word)
16. [What to Try Next](#what-to-try-next)

---

## Module 1: What Are We Even Doing? (Meet Godot)

> **New Word — Game Engine:** A game engine is a big toolbox that already knows how to do the hard, boring stuff every game needs — drawing pictures on screen, handling gravity, playing sounds, knowing when two things bump into each other. Instead of building all of that yourself, you use the toolbox and just focus on making *your* game. Godot is a free, open-source game engine.

> **New Word — Programming:** Programming means writing very precise, step-by-step instructions for a computer, in a language it understands. Godot's language is called **GDScript**, and it reads a lot like plain English with some extra punctuation rules. You'll learn it as we go — you don't need to know any of it yet.

Our game, **Dash Quest**, will have:

- A player character you control with the arrow keys and spacebar
- Enemies that walk back and forth and hurt you if you touch them (unless you jump on their heads!)
- 3 lives to start, with extra-life gems hidden in each level
- 5 levels that get harder as you go
- A main menu, a level-select screen, a pause menu, a game-over screen, and a victory screen

We won't use any downloaded artwork — everything will be simple colored shapes that you draw yourself right inside Godot. That keeps things simple, and honestly, colored-block games look pretty cool.

---

## Module 2: Installing Godot and Starting Your Project

> **Try It:**
> 1. Go to `godotengine.org` and click **Download**.
> 2. Choose the newest **Godot 4** version for your computer (Windows, Mac, or Linux). Pick the regular **Standard** version, not the ".NET" one — we're using GDScript, not C#.
> 3. Godot doesn't need to be "installed" the normal way — it's just one file you double-click to open.
> 4. When it opens, click **New Project**.
> 5. Give it a name (like "Dash Quest"), choose a folder to save it in, and for **Renderer** pick **Compatibility** (it runs on more computers).
> 6. Click **Create & Edit**.

You now have an empty Godot project! Let's look around before we build anything.

---

## Module 3: A Tour of the Godot Editor

When Godot opens your project, you'll see several boxes on the screen. Here's what each one does:

- **Scene panel** (usually top-left): shows the list of "pieces" that make up whatever you're currently building.
- **FileSystem panel** (bottom-left): shows all the files in your project, like a mini file browser.
- **Viewport** (the big middle area): shows what your game actually looks like, and lets you drag things around visually.
- **Inspector** (right side): shows all the settings (properties) of whatever piece is currently selected.

> **New Word — Node:** A Node is a single building block, like a LEGO brick. There are different kinds — one draws a picture, one plays a sound, one detects collisions. Everything in Godot is made of nodes.

> **New Word — Scene:** A Scene is a group of nodes stuck together to make something useful — like a LEGO brick tower. Your player character will be a scene. Each level will be a scene. Even your main menu is a scene. Scenes get saved as files ending in `.tscn`.

> **New Word — Script:** A script is a file full of code that gets attached to a node to give it a "brain" — instructions for what it should actually do.

Before building anything, let's set up two folders to keep things organized.

> **Try It:** In the **FileSystem** panel, right-click and choose **New Folder**. Make one folder called `scenes` and another called `scripts`. Every scene we build (`.tscn`) will get saved in `scenes`, and every script (`.gd`) will get saved in `scripts`. Godot is picky about capitalization in file names, so type these exactly.

---

## Module 4: Learning to "Talk" to the Computer (GDScript Basics)

Before we build anything, let's learn a handful of words in GDScript. You'll see all of these used later, so this is a cheat-sheet you can flip back to.

> **New Word — Variable:** A variable is a labeled box that holds a piece of information. In GDScript you make one like this:
> ```gdscript
> var lives = 3
> ```
> This creates a box named `lives` and puts the number `3` inside it. Later, code can check what's in the box, or put a new number in it.

**Data types** — the kinds of things that can go in a box:
- **Numbers**, like `3` or `220.0`
- **Strings** (text), always wrapped in quotes, like `"Level 1"`
- **Booleans** (true/false), like `true` or `false`
- **Vector2** — a pair of numbers for a position or direction, like `Vector2(100, 50)` (100 across, 50 down)

> **New Word — Function:** A function is a mini-recipe: a named list of instructions you can run whenever you want by saying its name. It looks like this:
> ```gdscript
> func say_hello():
>     print("Hello!")
> ```
> Whenever some other code writes `say_hello()`, the computer runs everything inside.

> **New Word — If/Else (a decision):** Sometimes code needs to make a choice.
> ```gdscript
> if lives <= 0:
>     print("Game over!")
> else:
>     print("Keep going!")
> ```
> This reads almost like English: *if* the lives box holds 0 or less, print "Game over!" — *otherwise*, print "Keep going!"

> **Watch Out:** GDScript cares a lot about **indentation** (the spaces at the start of a line). Anything "inside" a function or an `if` needs to be indented one extra step (press Tab). If you get an error about unexpected indentation, that's almost always why.

That's genuinely enough to get started. We'll introduce a few more ideas (lists, groups, signals) right when we first need them, so they stick better.

---

## Module 5: The Global Notebook (Autoload)

Every level in our game needs to remember the same things: how many lives you have left, and which level you're on. But each level is its own separate scene — so how do they share information?

> **New Word — Autoload (Singleton):** An Autoload is a script that Godot loads once, right when the game starts, and never throws away — no matter which scene you switch to. Think of it as a notebook that follows you everywhere in the game. We'll make one called `Global`.

> **Try It:**
> 1. In the **FileSystem** panel, right-click your `scripts` folder → **Create New** → **Script**.
> 2. Set the path to `res://scripts/global.gd`. Leave "Inherits" as `Node`. Click **Create**.
> 3. Delete anything Godot put in the file, and type this instead:

```gdscript
extends Node

const START_LIVES := 3
const MAX_LEVEL := 5

var lives: int = START_LIVES
var current_level: int = 1
var unlocked_levels: int = 1

func _ready() -> void:
	_setup_input()

func _setup_input() -> void:
	_add_key_action("move_left", [KEY_A, KEY_LEFT])
	_add_key_action("move_right", [KEY_D, KEY_RIGHT])
	_add_key_action("jump", [KEY_SPACE, KEY_W, KEY_UP])
	_add_key_action("pause", [KEY_ESCAPE, KEY_P])

func _add_key_action(action_name: String, keys: Array) -> void:
	if InputMap.has_action(action_name):
		return
	InputMap.add_action(action_name)
	for k in keys:
		var ev := InputEventKey.new()
		ev.physical_keycode = k
		InputMap.action_add_event(action_name, ev)

func reset_game() -> void:
	lives = START_LIVES
	current_level = 1

func reset_progress() -> void:
	unlocked_levels = 1
	reset_game()

func lose_life() -> bool:
	lives -= 1
	return lives <= 0

func add_life() -> void:
	lives += 1

func unlock_level(n: int) -> void:
	if n > unlocked_levels:
		unlocked_levels = n

func level_scene_path(n: int) -> String:
	return "res://scenes/Level%d.tscn" % n
```

**What does this do?**
- `lives`, `current_level`, and `unlocked_levels` are the "memory" boxes every part of the game can check.
- `_setup_input()` teaches Godot what the arrow keys, WASD, spacebar, and Escape should *mean* in our game (moving, jumping, pausing) — automatically, the very first time the game runs. Normally you'd click through a settings menu to do this; we're having the code do it for us instead.
- `lose_life()` takes away one life and answers `true` if that was your very last one (game over).
- `unlock_level(n)` remembers the furthest level you've reached, so Level Select knows which buttons to allow.

> **Try It:** Save the script (Ctrl+S). Now go to the top menu: **Project → Project Settings → Autoload** tab. Next to "Path," click the folder icon and choose `res://scripts/global.gd`. Make sure the **Name** box says exactly `Global` (capital G — every script later will refer to it that way). Click **Add**.

> **Why:** Now, from *any* script, anywhere in the game, you can write `Global.lives` or `Global.add_life()` and it just works — because Godot keeps this one notebook alive the whole time the game runs.

---

## Module 6: Building Your Hero (The Player)

Time to make something you can actually control.

> **New Word — CharacterBody2D:** A special kind of node built for characters you move with code (instead of letting real physics fling them around). It knows how to slide along the ground and detect walls automatically once you tell it a velocity (a speed and direction).

### Step 1: Build the scene

> **Try It:**
> 1. Click **Scene → New Scene**, then choose **Other Node**, search for `CharacterBody2D`, and click **Create**. Rename it to `Player` (double-click the name in the Scene panel).
> 2. With `Player` selected, add a child node: click the **+** button in the Scene panel, search `CollisionShape2D`, click **Create**.
> 3. Select the new `CollisionShape2D`. In the Inspector, click the **Shape** dropdown → **New RectangleShape2D**. Click the little shape icon to expand it, and set **Size** to `x = 28, y = 40`.

> **New Word — Collision Shape:** An invisible rectangle (or circle) that tells Godot "this is the part of me that can bump into things." It's separate from what you actually *see*, which we add next.

4. Select `Player` again, add another child: search `Polygon2D`, click **Create**, rename it to `Visual`.
5. With `Visual` selected, look at the 2D viewport — Godot is waiting for you to draw a shape. Click 4 points to make a rectangle roughly the same size as your collision box (about 28 wide, 40 tall), then press **Enter** to finish it. In the Inspector, set its **Color** to a nice blue.
   - *(Want to match exactly? Open the Polygon2D's "Polygon" property in the Inspector and type these 4 points: `(-14,-20) (14,-20) (14,20) (-14,20)`.)*
6. Add one more child under `Visual` (select `Visual`, then add child): another `Polygon2D`, named `Eyes`, drawn as a small white rectangle near the top — just for a face. This one is purely decorative; skip it if you want.
7. Select `Player` again, add a `Camera2D` child. In the Inspector, check **Position Smoothing → Enabled**, and set **Speed** to `6`. This is what will scroll the screen as your character runs.
8. Add one more child to `Player`: a `Timer` node, named `InvincibilityTimer`. In the Inspector, set **Wait Time** to `1.2` and check **One Shot**.

> **Why a Timer?** After getting hurt, we want a brief moment where you can't get hurt *again* immediately (so one enemy doesn't delete all 3 lives in one second). The Timer will count down 1.2 seconds and then tell us it's done.

### Step 2: Set collision channels

> **New Word — Collision Layer & Mask:** Imagine every physical object has a walkie-talkie. The **Layer** is which channel it *broadcasts* on. The **Mask** is which channels it *listens* to. If your Mask includes a channel something else is broadcasting on, you'll notice it (bump into it, stand on it, etc).

We'll use these channels throughout the whole project:
- **Channel 1** = ground and platforms
- **Channel 2** = the player
- **Channel 3** = enemy bodies

> **Try It:** Select the root `Player` node. Scroll the Inspector down to **Collision**. Click the **Layer** grid and turn on only square **2**. Click the **Mask** grid and turn on only square **1**. (This means: "I broadcast as the player, and I only care about bumping into the ground.")

### Step 3: Write the script

> **Try It:** Select the `Player` root node, click the script icon (top of Scene panel) → set path to `res://scripts/player.gd` → **Create**. Replace everything with:

```gdscript
extends CharacterBody2D

const SPEED := 220.0
const JUMP_VELOCITY := -420.0
const GRAVITY := 1000.0
const BOUNCE_VELOCITY := -260.0
const FALL_DEATH_Y := 2000.0

var spawn_point: Vector2
var invincible := false

func _ready() -> void:
	spawn_point = global_position
	add_to_group("player")

func _physics_process(delta: float) -> void:
	if not is_on_floor():
		velocity.y += GRAVITY * delta

	if Input.is_action_just_pressed("jump") and is_on_floor():
		velocity.y = JUMP_VELOCITY

	var dir := Input.get_axis("move_left", "move_right")
	velocity.x = dir * SPEED
	if dir != 0:
		$Visual.scale.x = -1.0 if dir < 0 else 1.0

	move_and_slide()

	if global_position.y > FALL_DEATH_Y:
		take_damage()

func take_damage() -> void:
	if invincible:
		return
	invincible = true
	$InvincibilityTimer.start()
	modulate.a = 0.4
	var is_dead := Global.lose_life()
	get_tree().call_group("hud", "refresh")
	if is_dead:
		get_tree().call_group("game_over_screen", "show_screen")
	else:
		respawn()

func bounce() -> void:
	velocity.y = BOUNCE_VELOCITY

func respawn() -> void:
	global_position = spawn_point
	velocity = Vector2.ZERO

func collect_extra_life() -> void:
	Global.add_life()
	get_tree().call_group("hud", "refresh")

func _on_invincibility_timer_timeout() -> void:
	invincible = false
	modulate.a = 1.0
```

**Line-by-line, in plain words:**
- `_ready()` runs once, the moment the player appears. We remember where we started (`spawn_point`), and we tag ourselves with the **group** `"player"`.
- **New Word — Group:** a group is like a name tag you stick on a node. Later, an enemy or a pickup can check `body.is_in_group("player")` to recognize you without needing to know your exact name.
- `_physics_process(delta)` runs about 60 times every single second — it's our main game loop. Each time: gravity pulls us down, we check if jump was pressed, we read the left/right keys, we flip our drawing to face the way we're moving, and `move_and_slide()` actually moves us and stops us at walls/floors.
- `take_damage()` checks if we're currently invincible (skip if so), starts the invincibility timer, tells `Global` we lost a life, and either respawns us or shows the Game Over screen.
- `get_tree().call_group("hud", "refresh")` means: *"everyone wearing the `hud` name tag, please run your `refresh` function right now."* This is how the lives counter on screen updates, without the Player needing to know anything about how the HUD works.

> **Try It:** Now we need to connect that Timer's signal. Select `InvincibilityTimer`. Next to the Inspector tab, click the **Node** tab. Double-click `timeout()`. In the dialog, make sure it's connecting to the `Player` node, and click **Connect**. Godot will jump you into the script with a new function already started — that's `_on_invincibility_timer_timeout()`, which we already wrote above, so just make sure it matches.

> **New Word — Signal:** A signal is a message a node shouts out when something happens ("I just finished counting down!"). Other nodes can "listen" for that shout and react.

> **Try It:** Save the scene as `res://scenes/Player.tscn`.

---

## Module 7: Making a Bad Guy (The Enemy)

> **Try It:**
> 1. New scene → `CharacterBody2D`, rename to `Enemy`.
> 2. Add a `CollisionShape2D` child → New `RectangleShape2D`, size `28 x 28`.
> 3. Add a `Polygon2D` child named `Visual`, draw a red square about that size.
> 4. (Optional) add a small white `Polygon2D` child under `Visual` named `Eyes` for a face.
> 5. Set the root `Enemy` node's **Collision → Layer** to square **3**, and **Mask** to square **1** (walks on the ground, doesn't get shoved around by the player).
> 6. Add one more child to `Enemy`: an `Area2D`, named `HurtBox`. Give it its own `CollisionShape2D` child with a `RectangleShape2D` sized `34 x 34` (a bit bigger than the enemy itself). Set `HurtBox`'s **Layer** to nothing (all off) and **Mask** to square **2** (it's listening for the player).

> **Why a separate Area2D?** `Area2D` nodes don't push or stop anything physically — they just quietly *notice* when something enters them. That makes them perfect for "detector zones" like this one, instead of trying to figure out damage from a physical bump.

Attach a script to the `Enemy` root: `res://scripts/enemy.gd`

```gdscript
extends CharacterBody2D

@export var speed: float = 60.0
@export var patrol_distance: float = 100.0

const GRAVITY := 900.0

var start_x: float
var direction := 1
var dead := false

func _ready() -> void:
	start_x = global_position.x
	add_to_group("enemies")

func _physics_process(delta: float) -> void:
	if dead:
		return
	velocity.y += GRAVITY * delta
	velocity.x = speed * direction
	move_and_slide()
	if abs(global_position.x - start_x) >= patrol_distance:
		direction *= -1
	$Visual.scale.x = -1.0 if direction < 0 else 1.0

func stomp() -> void:
	if dead:
		return
	dead = true
	queue_free()

func _on_hurt_box_body_entered(body: Node2D) -> void:
	if dead:
		return
	if not body.is_in_group("player"):
		return
	if body.global_position.y < global_position.y - 10.0 and body.velocity.y > 0.0:
		stomp()
		if body.has_method("bounce"):
			body.bounce()
	elif body.has_method("take_damage"):
		body.take_damage()
```

**What's new here:**
- `@export var speed` makes `speed` show up as an editable field in the Inspector, so later we can give different enemies different speeds without changing this script.
- The enemy walks (`direction * speed`) until it's gone `patrol_distance` pixels from where it started, then turns around — that's the back-and-forth patrol.
- `_on_hurt_box_body_entered` runs whenever something walks into the `HurtBox`. If it's the player *and* they're above the enemy *and* falling — that's a stomp: kill the enemy, bounce the player up. Otherwise, it's a side hit, so the player takes damage.
- `queue_free()` tells Godot "delete this node soon" (not instantly, but very soon — safely, once it's done being used this frame).

> **Try It:** Select `HurtBox`, click the **Node** tab, double-click `body_entered(body: Node2D)`, connect it to the `Enemy` node. Save the scene as `res://scenes/Enemy.tscn`.

---

## Module 8: Coins and Flags (Extra Life + Goal)

### Extra Life pickup

> **Try It:** New scene → `Area2D`, rename to `ExtraLife`. Add a `CollisionShape2D` child with a `CircleShape2D`, radius `12`. Add a `Polygon2D` child named `Visual`, colored yellow, drawn as a small diamond or star shape (doesn't need to be perfect). Set the root's **Layer** to nothing, **Mask** to square **2**.

Script `res://scripts/extra_life.gd`:

```gdscript
extends Area2D

var base_y: float
var t := 0.0

func _ready() -> void:
	collision_layer = 0
	collision_mask = 2
	base_y = position.y

func _process(delta: float) -> void:
	t += delta
	position.y = base_y + sin(t * 3.0) * 4.0

func _on_body_entered(body: Node2D) -> void:
	if body.is_in_group("player") and body.has_method("collect_extra_life"):
		body.collect_extra_life()
		queue_free()
```

The `sin(t * 3.0) * 4.0` line makes it gently float up and down forever — `sin` is a math function that smoothly rises and falls between -1 and 1, so multiplying it by 4 gives a small, smooth bob.

> **Try It:** Connect `body_entered` (Node tab) to itself. Save as `res://scenes/ExtraLife.tscn`.

### Goal flag

> **Try It:** New scene → `Area2D`, rename to `Goal`. Add a `CollisionShape2D` with a `RectangleShape2D` sized `30 x 150` (tall, so it's easy to walk into). Add two `Polygon2D` children: `Pole` (a thin gray rectangle) and `Flag` (a small green triangle near the top). Set **Layer** to nothing, **Mask** to square **2**.

Script `res://scripts/goal.gd`:

```gdscript
extends Area2D

signal reached

func _ready() -> void:
	collision_layer = 0
	collision_mask = 2

func _on_body_entered(body: Node2D) -> void:
	if body.is_in_group("player"):
		reached.emit()
```

`signal reached` declares a brand-new signal that *this scene* can shout. We're not handling what happens next here — we just announce "the player got here!" and let whatever's using this Goal decide what that means (we'll wire that up in Module 11).

> **Try It:** Connect `body_entered` to itself, save as `res://scenes/Goal.tscn`.

---

## Module 9: The Scoreboard (HUD)

> **New Word — HUD:** "Heads-Up Display" — the score, lives, and buttons drawn on top of the game, which don't move even when the camera scrolls.

> **New Word — CanvasLayer:** A special node that floats above everything in the game world, like a sheet of glass over your TV screen. UI elements go here so they always stay put on screen.

> **Try It:**
> 1. New scene → search `CanvasLayer`, rename to `HUD`.
> 2. Add a `Control` child named `Margin`. With it selected, look at the toolbar above the viewport for a **Layout** button, and choose **Full Rect** — this stretches it to cover the whole screen.
> 3. Add a `Label` child under `Margin` named `LivesLabel`. Set its **Text** to `Lives: 3` and drag it to the top-left corner.
> 4. Add another `Label` named `LevelLabel`, text `Level 1 / 5`, drag it to the top-center.
> 5. Add a `Button` named `PauseButton`, text `II`, drag it to the top-right corner.
>
> Feel free to make the labels bigger or add colors using **Theme Overrides** in the Inspector — this part is all yours to style.

Script `res://scripts/hud.gd`:

```gdscript
extends CanvasLayer

func _ready() -> void:
	add_to_group("hud")
	refresh()

func refresh() -> void:
	$Margin/LivesLabel.text = "Lives: %d" % Global.lives

func set_level_label(n: int) -> void:
	$Margin/LevelLabel.text = "Level %d / %d" % [n, Global.MAX_LEVEL]

func _on_pause_button_pressed() -> void:
	get_tree().call_group("pause_menu", "toggle_pause")
```

`"Lives: %d" % Global.lives` is **string formatting** — it means "take this text, and stick the number from `Global.lives` in where the `%d` is." It's a tidy way to build text out of variables.

> **Try It:** Select `PauseButton`, Node tab, connect `pressed()` to the `HUD` node. Save as `res://scenes/HUD.tscn`.

---

## Module 10: Pause, Game Over, and Victory Screens

These three screens are all built the same way: a `CanvasLayer`, a dim see-through background, and some buttons in the middle — starting off invisible until we need them.

> **New Word — process_mode:** Normally, when we "pause" the game (`get_tree().paused = true`), *every* node stops running its code — including buttons! We need the pause menu itself to be an exception, so its buttons still work while everything else is frozen. Setting `process_mode = Node.PROCESS_MODE_ALWAYS` in code tells Godot "keep this one running, no matter what."

### Pause Menu

> **Try It:**
> 1. New scene → `CanvasLayer`, rename `PauseMenu`.
> 2. Add a `ColorRect` child named `Dim`, Layout → Full Rect, **Color** set to black with the alpha (last slider) turned down to about 60% — this dims the background game.
> 3. Add a `VBoxContainer` child named `Box`, Layout → Center. Inside it, add a `Label` (text `Paused`) and three `Button`s: `Resume`, `Restart`, `MainMenu` (texts "Resume", "Restart Level", "Main Menu").

> **New Word — VBoxContainer:** A container that automatically stacks its children in a vertical column, evenly spaced, so you don't have to position each button by hand.

Script `res://scripts/pause_menu.gd`:

```gdscript
extends CanvasLayer

func _ready() -> void:
	process_mode = Node.PROCESS_MODE_ALWAYS
	add_to_group("pause_menu")
	visible = false

func _unhandled_input(event: InputEvent) -> void:
	if event.is_action_pressed("pause"):
		toggle_pause()
		get_viewport().set_input_as_handled()

func toggle_pause() -> void:
	if get_tree().paused and not visible:
		return
	get_tree().paused = not get_tree().paused
	visible = get_tree().paused

func _on_resume_pressed() -> void:
	toggle_pause()

func _on_restart_pressed() -> void:
	get_tree().paused = false
	visible = false
	get_tree().reload_current_scene()

func _on_main_menu_pressed() -> void:
	get_tree().paused = false
	visible = false
	get_tree().change_scene_to_file("res://scenes/MainMenu.tscn")
```

Connect each button's `pressed()` signal (Node tab) to the matching function on the `PauseMenu` node. Save as `res://scenes/PauseMenu.tscn`.

### Game Over Screen

Same recipe: `CanvasLayer` named `GameOverScreen` → `Dim` (`ColorRect`, Full Rect, black ~75% alpha) → `Box` (`VBoxContainer`, centered) → `Label` ("Game Over") + `Button`s `Retry` ("Retry Level") and `MainMenu` ("Main Menu").

Script `res://scripts/game_over_screen.gd`:

```gdscript
extends CanvasLayer

func _ready() -> void:
	process_mode = Node.PROCESS_MODE_ALWAYS
	add_to_group("game_over_screen")
	visible = false

func show_screen() -> void:
	get_tree().paused = true
	visible = true

func _on_retry_pressed() -> void:
	Global.lives = Global.START_LIVES
	get_tree().paused = false
	visible = false
	get_tree().reload_current_scene()

func _on_main_menu_pressed() -> void:
	get_tree().paused = false
	visible = false
	get_tree().change_scene_to_file("res://scenes/MainMenu.tscn")
```

Connect the two buttons, save as `res://scenes/GameOverScreen.tscn`.

### Victory Screen

Same recipe again, but simpler: `CanvasLayer` named `VictoryScreen` → `Dim` → `Box` → `Label` ("You Win!") + one `Button` named `MainMenu` ("Main Menu").

Script `res://scripts/victory_screen.gd`:

```gdscript
extends CanvasLayer

func _ready() -> void:
	process_mode = Node.PROCESS_MODE_ALWAYS
	add_to_group("victory_screen")
	visible = false

func show_screen() -> void:
	get_tree().paused = true
	visible = true

func _on_main_menu_pressed() -> void:
	get_tree().paused = false
	visible = false
	Global.reset_game()
	get_tree().change_scene_to_file("res://scenes/MainMenu.tscn")
```

Connect the button, save as `res://scenes/VictoryScreen.tscn`.

---

## Module 11: The Level Machine (loops, lists, and recipes)

This is the hardest module in the whole project. Don't worry if it takes a couple of re-reads — even experienced programmers slow down for this kind of thing. Here's the problem we're solving: placing every single platform and enemy by hand, five times, would take forever and be easy to mess up. Instead, we're going to write **one script that knows how to build *any* level**, as long as we hand it a list of ingredients.

> **New Word — Array (a list):** An Array is an ordered list of things, written in square brackets:
> ```gdscript
> var fruits = ["apple", "banana", "pear"]
> ```

> **New Word — Dictionary (a labeled list):** A Dictionary stores information under labels ("keys"), instead of just order:
> ```gdscript
> var player_info = {"name": "Alex", "score": 10}
> print(player_info["score"])   # prints 10
> ```
> We'll use a Dictionary to describe an entire level: its platforms, its enemies, where the goal is, and so on.

> **New Word — For Loop:** A way to repeat an instruction once for every item in an Array:
> ```gdscript
> for fruit in fruits:
>     print(fruit)
> ```
> This prints "apple," then "banana," then "pear" — automatically, without writing `print()` three separate times.

> **New Word — Inheritance (a recipe template):** One script can say "I work just like *this other script*, but I'll fill in my own details." The base script is called a **base class**. Each of our 5 levels will be a tiny script that just fills in the details, while a shared `level_base.gd` script does all the actual building work.

> **New Word — Rect2:** A rectangle described by a position (its top-left corner) and a size (width and height): `Rect2(x, y, width, height)`.

### Building the base

We won't build this one visually in the editor at all — it works entirely in code, which is actually *less* clicking for you.

> **Try It:** Create `res://scripts/level_base.gd`:

```gdscript
extends Node2D

const PLAYER_SCENE := preload("res://scenes/Player.tscn")
const ENEMY_SCENE := preload("res://scenes/Enemy.tscn")
const EXTRA_LIFE_SCENE := preload("res://scenes/ExtraLife.tscn")
const GOAL_SCENE := preload("res://scenes/Goal.tscn")
const HUD_SCENE := preload("res://scenes/HUD.tscn")
const PAUSE_MENU_SCENE := preload("res://scenes/PauseMenu.tscn")
const GAME_OVER_SCENE := preload("res://scenes/GameOverScreen.tscn")
const VICTORY_SCENE := preload("res://scenes/VictoryScreen.tscn")

var data: Dictionary
var next_level_path: String = ""

func _ready() -> void:
	data = _level_data()
	RenderingServer.set_default_clear_color(data.get("bg_color", Color(0.45, 0.65, 0.9)))
	_build_blocks(data.get("ground_segments", []), Color(0.36, 0.24, 0.14))
	_build_blocks(data.get("platforms", []), Color(0.55, 0.38, 0.2))
	_spawn_enemies(data.get("enemies", []))
	_spawn_extra_lives(data.get("extra_lives", []))
	_spawn_goal(data.get("goal_pos", Vector2.ZERO))
	_spawn_player(data.get("player_start", Vector2.ZERO))
	_spawn_ui(data.get("level_number", 1))
	next_level_path = data.get("next_level", "")

func _level_data() -> Dictionary:
	return {}

func _make_block(rect: Rect2, color: Color) -> void:
	var body := StaticBody2D.new()
	body.collision_layer = 1
	body.collision_mask = 0
	body.position = rect.position + rect.size / 2.0

	var shape := CollisionShape2D.new()
	var rect_shape := RectangleShape2D.new()
	rect_shape.size = rect.size
	shape.shape = rect_shape
	body.add_child(shape)

	var hw := rect.size.x / 2.0
	var hh := rect.size.y / 2.0
	var visual := Polygon2D.new()
	visual.color = color
	visual.polygon = PackedVector2Array([
		Vector2(-hw, -hh), Vector2(hw, -hh), Vector2(hw, hh), Vector2(-hw, hh)
	])
	body.add_child(visual)

	add_child(body)

func _build_blocks(rects: Array, color: Color) -> void:
	for r in rects:
		_make_block(r, color)

func _spawn_enemies(enemies: Array) -> void:
	for e in enemies:
		var enemy := ENEMY_SCENE.instantiate()
		add_child(enemy)
		enemy.global_position = e.get("pos", Vector2.ZERO)
		enemy.patrol_distance = e.get("patrol", 100.0)
		enemy.speed = e.get("speed", 60.0)

func _spawn_extra_lives(positions: Array) -> void:
	for pos in positions:
		var life := EXTRA_LIFE_SCENE.instantiate()
		add_child(life)
		life.position = pos

func _spawn_goal(pos: Vector2) -> void:
	var goal := GOAL_SCENE.instantiate()
	add_child(goal)
	goal.position = pos
	goal.reached.connect(_on_goal_reached)

func _spawn_player(pos: Vector2) -> void:
	var player := PLAYER_SCENE.instantiate()
	add_child(player)
	player.global_position = pos
	var cam: Camera2D = player.get_node("Camera2D")
	cam.limit_left = 0
	cam.limit_right = int(data.get("width", 2000))
	cam.limit_top = -2000
	cam.limit_bottom = int(data.get("floor_y", 550)) + 400

func _spawn_ui(level_number: int) -> void:
	var hud := HUD_SCENE.instantiate()
	add_child(hud)
	hud.set_level_label(level_number)
	add_child(PAUSE_MENU_SCENE.instantiate())
	add_child(GAME_OVER_SCENE.instantiate())
	add_child(VICTORY_SCENE.instantiate())

func _on_goal_reached() -> void:
	Global.unlock_level(int(data.get("level_number", 1)) + 1)
	if next_level_path == "":
		get_tree().call_group("victory_screen", "show_screen")
	else:
		get_tree().change_scene_to_file(next_level_path)
```

**Walking through the confusing parts:**
- `preload(...)` loads a scene file into memory when the game starts, so it's instantly ready to copy later. `.instantiate()` makes an actual copy (an "instance") of that scene we can add to the game.
- `_level_data()` returns an *empty* Dictionary here on purpose. Each real level will have its own script that **overrides** this function with real ingredients — that's the "recipe template" idea from before.
- `_make_block()` is a little factory: give it a rectangle and a color, and it builds a solid platform out of raw code — a `StaticBody2D` (something solid that never moves) with a collision shape and a matching colored visual — no manual scene-building required!
- `_on_goal_reached()` is the function we connect to the Goal's `reached` signal from Module 8. It unlocks the next level and either moves on, or — if there's no next level — shows the victory screen.

> **Try It:** Save this file. It doesn't attach to any scene by itself — it's a *template* other scripts will use.

---

## Module 12: Building All 5 Levels

Now for the payoff: each level is just a tiny scene (one empty `Node2D`) plus a tiny script listing its own ingredients.

> **Try It:** New scene → plain `Node2D`, rename `Level1`. Attach a new script `res://scripts/level_1.gd`. **Delete the auto-generated `extends Node2D` line** and replace the whole file with this — notice the first line points at our template file instead of a plain node type:

```gdscript
extends "res://scripts/level_base.gd"

func _level_data() -> Dictionary:
	return {
		"level_number": 1,
		"width": 1800,
		"floor_y": 550,
		"bg_color": Color(0.55, 0.78, 0.95),
		"ground_segments": [
			Rect2(0, 550, 700, 150),
			Rect2(850, 550, 950, 150),
		],
		"platforms": [
			Rect2(300, 420, 120, 20),
			Rect2(950, 420, 150, 20),
			Rect2(1300, 350, 150, 20),
		],
		"enemies": [
			{"pos": Vector2(500, 536), "patrol": 80.0, "speed": 55.0},
			{"pos": Vector2(1100, 536), "patrol": 100.0, "speed": 55.0},
		],
		"extra_lives": [
			Vector2(360, 380),
		],
		"player_start": Vector2(80, 530),
		"goal_pos": Vector2(1750, 475),
		"next_level": "res://scenes/Level2.tscn",
	}
```

Notice the two `ground_segments` rectangles don't touch — there's a gap between x=700 and x=850. That gap is a **pit**. Walk into it and you'll fall forever and lose a life (remember `FALL_DEATH_Y` from the Player script?).

Save this scene as `res://scenes/Level1.tscn`.

> **Try It:** Repeat the exact same two steps (new `Node2D` scene, attach a script extending `level_base.gd`) for Levels 2 through 5. Each one just needs a bigger, trickier Dictionary. Here they are, ready to copy in:

**`res://scripts/level_2.gd`** (save scene as `Level2.tscn`):
```gdscript
extends "res://scripts/level_base.gd"

func _level_data() -> Dictionary:
	return {
		"level_number": 2,
		"width": 2200,
		"floor_y": 550,
		"bg_color": Color(0.95, 0.78, 0.5),
		"ground_segments": [
			Rect2(0, 550, 600, 150),
			Rect2(750, 550, 500, 150),
			Rect2(1400, 550, 800, 150),
		],
		"platforms": [
			Rect2(250, 420, 120, 20),
			Rect2(850, 400, 140, 20),
			Rect2(1100, 320, 140, 20),
			Rect2(1600, 420, 150, 20),
		],
		"enemies": [
			{"pos": Vector2(400, 536), "patrol": 90.0, "speed": 60.0},
			{"pos": Vector2(950, 536), "patrol": 100.0, "speed": 65.0},
			{"pos": Vector2(1650, 536), "patrol": 120.0, "speed": 65.0},
		],
		"extra_lives": [
			Vector2(900, 360),
			Vector2(1650, 380),
		],
		"player_start": Vector2(80, 530),
		"goal_pos": Vector2(2150, 475),
		"next_level": "res://scenes/Level3.tscn",
	}
```

**`res://scripts/level_3.gd`** (save scene as `Level3.tscn`):
```gdscript
extends "res://scripts/level_base.gd"

func _level_data() -> Dictionary:
	return {
		"level_number": 3,
		"width": 2500,
		"floor_y": 550,
		"bg_color": Color(0.65, 0.6, 0.8),
		"ground_segments": [
			Rect2(0, 550, 500, 150),
			Rect2(700, 550, 450, 150),
			Rect2(1300, 550, 500, 150),
			Rect2(1950, 550, 550, 150),
		],
		"platforms": [
			Rect2(200, 420, 120, 20),
			Rect2(600, 380, 120, 20),
			Rect2(950, 420, 140, 20),
			Rect2(1500, 350, 150, 20),
			Rect2(1800, 420, 150, 20),
		],
		"enemies": [
			{"pos": Vector2(350, 536), "patrol": 100.0, "speed": 70.0},
			{"pos": Vector2(850, 536), "patrol": 100.0, "speed": 70.0},
			{"pos": Vector2(1450, 536), "patrol": 120.0, "speed": 80.0},
			{"pos": Vector2(2050, 536), "patrol": 120.0, "speed": 80.0},
		],
		"extra_lives": [
			Vector2(650, 340),
			Vector2(1550, 310),
		],
		"player_start": Vector2(80, 530),
		"goal_pos": Vector2(2450, 475),
		"next_level": "res://scenes/Level4.tscn",
	}
```

**`res://scripts/level_4.gd`** (save scene as `Level4.tscn`):
```gdscript
extends "res://scripts/level_base.gd"

func _level_data() -> Dictionary:
	return {
		"level_number": 4,
		"width": 2800,
		"floor_y": 550,
		"bg_color": Color(0.85, 0.5, 0.35),
		"ground_segments": [
			Rect2(0, 550, 450, 150),
			Rect2(650, 550, 400, 150),
			Rect2(1200, 550, 400, 150),
			Rect2(1750, 550, 450, 150),
			Rect2(2350, 550, 450, 150),
		],
		"platforms": [
			Rect2(180, 420, 120, 20),
			Rect2(550, 380, 120, 20),
			Rect2(850, 420, 140, 20),
			Rect2(1100, 320, 140, 20),
			Rect2(1450, 400, 140, 20),
			Rect2(1700, 320, 140, 20),
			Rect2(2100, 420, 150, 20),
		],
		"enemies": [
			{"pos": Vector2(300, 536), "patrol": 100.0, "speed": 80.0},
			{"pos": Vector2(750, 536), "patrol": 110.0, "speed": 85.0},
			{"pos": Vector2(1300, 536), "patrol": 110.0, "speed": 85.0},
			{"pos": Vector2(1850, 536), "patrol": 130.0, "speed": 90.0},
			{"pos": Vector2(2450, 536), "patrol": 130.0, "speed": 90.0},
		],
		"extra_lives": [
			Vector2(600, 340),
			Vector2(1150, 280),
			Vector2(2150, 380),
		],
		"player_start": Vector2(80, 530),
		"goal_pos": Vector2(2750, 475),
		"next_level": "res://scenes/Level5.tscn",
	}
```

**`res://scripts/level_5.gd`** (save scene as `Level5.tscn`) — the final level, so `next_level` is an empty string, which triggers the victory screen instead of loading another level:
```gdscript
extends "res://scripts/level_base.gd"

func _level_data() -> Dictionary:
	return {
		"level_number": 5,
		"width": 3200,
		"floor_y": 550,
		"bg_color": Color(0.12, 0.12, 0.28),
		"ground_segments": [
			Rect2(0, 550, 400, 150),
			Rect2(600, 550, 350, 150),
			Rect2(1100, 550, 350, 150),
			Rect2(1600, 550, 350, 150),
			Rect2(2100, 550, 350, 150),
			Rect2(2600, 550, 600, 150),
		],
		"platforms": [
			Rect2(150, 420, 120, 20),
			Rect2(500, 380, 120, 20),
			Rect2(800, 420, 140, 20),
			Rect2(1000, 320, 140, 20),
			Rect2(1350, 400, 140, 20),
			Rect2(1550, 300, 140, 20),
			Rect2(1900, 420, 140, 20),
			Rect2(2200, 340, 140, 20),
			Rect2(2500, 420, 150, 20),
			Rect2(2850, 380, 150, 20),
		],
		"enemies": [
			{"pos": Vector2(250, 536), "patrol": 100.0, "speed": 90.0},
			{"pos": Vector2(700, 536), "patrol": 110.0, "speed": 95.0},
			{"pos": Vector2(1200, 536), "patrol": 120.0, "speed": 95.0},
			{"pos": Vector2(1700, 536), "patrol": 130.0, "speed": 100.0},
			{"pos": Vector2(2200, 536), "patrol": 130.0, "speed": 100.0},
			{"pos": Vector2(2800, 536), "patrol": 150.0, "speed": 110.0},
		],
		"extra_lives": [
			Vector2(550, 340),
			Vector2(1400, 280),
			Vector2(2250, 300),
		],
		"player_start": Vector2(80, 530),
		"goal_pos": Vector2(3150, 475),
		"next_level": "",
	}
```

> **Try It:** Open `Level1.tscn` and press the **Play Scene** button (or F6) to test just this level. You should see a blue sky, brown ground with a gap, floating platforms, two red enemies patrolling, a glowing gem, and a green flag at the end.

---

## Module 13: Main Menu and Level Select

### Main Menu

> **Try It:**
> 1. New scene → `Control`, rename `MainMenu`. Layout → Full Rect.
> 2. Add a `ColorRect` child named `Background`, Layout → Full Rect, pick a color.
> 3. Add a `VBoxContainer` named `Box`, Layout → Center.
> 4. Inside `Box`: a `Label` ("Dash Quest," make it big), a smaller `Label` (subtitle, optional), and three `Button`s: `Start` ("Start Game"), `LevelSelect` ("Level Select"), `Quit` ("Quit").

Script `res://scripts/main_menu.gd`:

```gdscript
extends Control

func _on_start_pressed() -> void:
	Global.reset_game()
	get_tree().change_scene_to_file(Global.level_scene_path(1))

func _on_level_select_pressed() -> void:
	get_tree().change_scene_to_file("res://scenes/LevelSelect.tscn")

func _on_quit_pressed() -> void:
	get_tree().quit()
```

Connect all three buttons' `pressed()` signals to the matching functions. Save as `res://scenes/MainMenu.tscn`.

### Level Select

> **Try It:** New scene → `Control`, rename `LevelSelect`, Layout → Full Rect. Add `Background` (`ColorRect`, Full Rect). Add `Box` (`VBoxContainer`, Center) containing: a title `Label`, five `Button`s named exactly `Level1Button` through `Level5Button` (spelling matters!), and two smaller `Button`s: `ResetProgress` and `Back`.

Script `res://scripts/level_select.gd`:

```gdscript
extends Control

func _ready() -> void:
	for n in range(1, Global.MAX_LEVEL + 1):
		var btn: Button = get_node("Box/Level%dButton" % n)
		btn.disabled = n > Global.unlocked_levels

func _select_level(n: int) -> void:
	Global.reset_game()
	Global.current_level = n
	get_tree().change_scene_to_file(Global.level_scene_path(n))

func _on_level1button_pressed() -> void:
	_select_level(1)

func _on_level2button_pressed() -> void:
	_select_level(2)

func _on_level3button_pressed() -> void:
	_select_level(3)

func _on_level4button_pressed() -> void:
	_select_level(4)

func _on_level5button_pressed() -> void:
	_select_level(5)

func _on_reset_progress_pressed() -> void:
	Global.reset_progress()
	_ready()

func _on_back_pressed() -> void:
	get_tree().change_scene_to_file("res://scenes/MainMenu.tscn")
```

`range(1, Global.MAX_LEVEL + 1)` counts `1, 2, 3, 4, 5` — the `for` loop then checks each level button and **disables** it if you haven't unlocked it yet (`n > Global.unlocked_levels`). That's the whole level-locking system in four lines.

> **Try It:** Connect all seven buttons' `pressed()` signals. Save as `res://scenes/LevelSelect.tscn`.

---

## Module 14: Final Wiring and Playtesting

Almost done! A few settings tie the whole game together.

> **Try It:**
> 1. Go to **Project → Project Settings → General → Application → Run**, and set **Main Scene** to `res://scenes/MainMenu.tscn`. (Or just press **Play** in the top-right corner — Godot will ask you to pick a main scene the first time, and you can choose it there.)
> 2. Still in Project Settings, under **Display → Window**, you can set **Viewport Width** to `1152` and **Viewport Height** to `648` for a nice widescreen size — though Godot's default is already close to this.
> 3. Press **Play** (F5). You should land on your main menu!

**Test the whole loop:**
- Start Game → make sure you can move, jump, and the camera follows you.
- Walk into an enemy from the side — you should lose a life and respawn.
- Jump on an enemy's head — it should disappear and bounce you upward.
- Grab an extra-life gem — your lives counter should go up.
- Fall in a pit — you should lose a life.
- Press Escape — the game should pause, with working buttons.
- Lose all 3 lives — the Game Over screen should appear.
- Reach the flag on Level 5 — you should see "You Win!"
- Go back to the main menu and try Level Select — only unlocked levels should be clickable.

> **Watch Out:** The single most common error you'll see is something like `Node not found: "Visual"`. This almost always means a child node's **name** doesn't exactly match what the script expects (`$Visual`, `$InvincibilityTimer`, `$Margin/LivesLabel`, etc). Node names are case-sensitive — `visual` and `Visual` are different to Godot. Click on the node in your Scene panel and check the exact spelling.

> **Watch Out:** If a button doesn't do anything when clicked, you probably forgot to connect its `pressed()` signal in the **Node** tab. Signals don't connect themselves just because a function with a matching name exists — you have to draw that connection once, in the editor.

---

## Glossary (Look Up Any Word)

- **Node** — a single building-block piece in Godot (like a LEGO brick).
- **Scene** — a group of nodes saved together (`.tscn` file).
- **Script** — code (`.gd` file) attached to a node to control what it does.
- **Variable** — a labeled box holding a value.
- **Function** — a named, reusable set of instructions.
- **If/Else** — code that makes a decision.
- **Array** — an ordered list of values, `[like, this]`.
- **Dictionary** — a list of values with labels, `{"like": "this"}`.
- **For loop** — repeats an instruction once per item in a list.
- **Signal** — a message a node shouts when something happens.
- **Group** — a name tag stuck on a node so other code can recognize it.
- **Autoload / Singleton** — a script that loads once and stays alive across every scene.
- **CharacterBody2D** — a node built for characters you move with code.
- **Area2D** — a node that quietly detects when something enters it, without physically blocking it.
- **StaticBody2D** — a solid object that never moves (like a platform).
- **CollisionShape2D** — the invisible shape that defines what a body can physically touch.
- **Collision Layer / Mask** — the "walkie-talkie channel" system deciding what notices what.
- **CanvasLayer** — a UI layer that stays fixed on screen, ignoring the camera.
- **Vector2** — a pair of numbers representing a 2D position or direction.
- **Rect2** — a rectangle: a position plus a width and height.
- **Inheritance / base class** — one script reusing another script's behavior and adding its own details.
- **preload / instantiate** — loading a scene into memory, then making a usable copy of it.

---

## What to Try Next

You built a whole game. Here are some upgrades to try once you're comfortable — each one reuses something you already learned:

- **A score counter** — add a `score` variable to `Global`, and a `Label` on the HUD, similar to how lives work.
- **A double jump** — give the Player a counter for jumps used, and let it jump again in the air, up to 2 times.
- **Moving platforms** — make a new scene with an `AnimatableBody2D` that slides back and forth, similar to how the Enemy patrols.
- **Sound effects** — add an `AudioStreamPlayer` node and play a sound when you jump, get hurt, or grab a gem.
- **Real artwork** — swap the colored Polygon2D shapes for a `Sprite2D` using pictures you draw or download.
- **A 6th level** — copy the pattern from Module 12 one more time. You've already done the hard part four times over.

Nice work. Go show Liam.
