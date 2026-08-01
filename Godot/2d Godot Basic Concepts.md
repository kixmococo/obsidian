# Learning Godot From Zero: Building a Mini Action-Platformer

A complete beginner-to-finished-game guide, built step by step with u1 and u2.

---

## Roadmap

1. Foundations
2. The Godot Editor
3. Player Scene
4. Sprite & Collision
5. Your First Script (Movement)
6. Custom Input (WASD + Arrow Keys)
7. Walls and Collision
8. Camera
9. Bigger World / Platformer Level
10. Enemies (Patrol AI)
11. Combat — Player Takes Damage
12. Combat — Attacking & Enemy Health
13. Items & Inventory (Action Bar)
14. UI — Heart Health Bar & Game Over
15. Polish (Juice) & Export
16. Where to go next

---

## Lesson 1: What Is a Computer Program?

A **computer** is a machine that follows instructions, very fast, very literally. It has no common sense — it does *exactly* what you tell it, nothing more.

A **program** is a list of instructions written in a language the computer can understand. For example: "when the player presses the right arrow key, move the character 5 pixels to the right." The computer runs that instruction thousands of times per second, which is why it looks smooth.

**Programming languages** let humans write instructions in a way that's readable to us but can be translated into what the computer needs. Godot uses **GDScript** — a language designed specifically for making games, and one of the easier languages to learn since it reads almost like plain English.

**Godot** is a **game engine** — a pre-built toolkit that handles the boring, hard stuff (drawing images, playing sounds, detecting collisions, reading keyboard input) so you only write the instructions specific to *your* game.

---

## Lesson 2: Installing Godot

1. Go to **godotengine.org**
2. Click **Download**
3. Choose **Godot 4.x, Standard version** (not ".NET" — that's for C#; GDScript needs Standard)
4. Pick your OS
5. Unzip and run the single executable — no installer needed
6. Keep the folder somewhere permanent (e.g. `Documents/Godot`)
7. Double-click to launch → you should see the **Project Manager**

---

## Lesson 3: Creating Your Project

1. In the Project Manager, click **"+ Create New Project"**
2. **Project Name**: e.g. `MiniZelda`
3. **Project Path**: pick a folder (e.g. `Documents/Godot/MiniZelda`)
4. **Renderer**: choose **Compatibility** (runs on any computer; Forward+ is for high-end 3D)
5. Click **Create & Edit**

### Core concept: Scenes and Nodes

- A **Node** is the basic building block of everything in Godot — a sprite, a sound player, a collision shape, a timer, a UI button. Each node does one job.
- A **Scene** is a group of nodes arranged in a tree (parent → children), saved together as a reusable unit.

Think nesting dolls: small scenes (player, enemy, sword) combine into bigger scenes (a room), which combine into the biggest scene of all (your game).

---

## Lesson 4: Creating the Player Scene

1. `Scene → New Scene` → choose **2D Scene** (creates a `Node2D` root)
2. Delete that root — we actually want `CharacterBody2D` as root, since it comes with built-in movement/collision helpers
3. Add a new root node → search `CharacterBody2D` → Create
4. Rename it to `Player` (double-click the name)
5. Save: `Ctrl+S` → `scenes/player.tscn` (create a `scenes` subfolder to stay organized)

---

## Lesson 5: Sprite and Collision

A `CharacterBody2D` alone is invisible and shapeless. It needs two children: something to draw (visual) and something to define its physical shape (collision).

**Visual (placeholder art):**
1. Right-click `Player` → Add Child Node → `ColorRect` → Create
2. Inspector: **Size** `32x32`, pick any **Color**, **Position** `-16, -16` (centers it on the origin)

**Collision shape:**
1. Right-click `Player` → Add Child Node → `CollisionShape2D` → Create
2. Inspector → **Shape** → New **RectangleShape2D** → set **Size** `32x32`

Resulting tree:
```
Player (CharacterBody2D)
 ├─ ColorRect
 └─ CollisionShape2D
```

---

## Lesson 6: Your First Script (Movement)

**Concept:** A script is a file of instructions attached to a node, written in GDScript, telling that node how to behave.

1. Select `Player` → right-click → **Attach Script** → defaults → **Create** (`player.gd`)

Godot's starter code:
```gdscript
extends CharacterBody2D

func _ready():
	pass

func _physics_process(delta):
	pass
```

- `extends CharacterBody2D` — inherits built-in movement/collision abilities
- `_ready()` — runs once, when the node enters the game
- `_physics_process(delta)` — runs automatically ~60x/sec; `delta` is the tiny fraction of a second since the last call, used for frame-rate-independent movement
- `pass` — placeholder meaning "do nothing yet"

**Movement code (original, arrow-key version):**
```gdscript
extends CharacterBody2D

var speed = 200

func _physics_process(delta):
	var direction = Vector2.ZERO

	if Input.is_action_pressed("ui_right"):
		direction.x += 1
	if Input.is_action_pressed("ui_left"):
		direction.x -= 1
	if Input.is_action_pressed("ui_down"):
		direction.y += 1
	if Input.is_action_pressed("ui_up"):
		direction.y -= 1

	velocity = direction.normalized() * speed
	move_and_slide()
```

- `var speed = 200` — a **variable**: a named box holding a value (pixels/second here)
- `Vector2` — a pair of numbers (X, Y) used for position, direction, speed
- `Input.is_action_pressed("ui_right")` — checks if a key/action is currently held
- `direction.normalized()` — makes the direction's length exactly 1 so diagonal movement isn't faster than straight movement
- `velocity` — built-in property on `CharacterBody2D` representing speed + direction
- `move_and_slide()` — built-in function that actually moves the body and handles sliding along walls

Run with **F6** (Play Current Scene) or the "Play Current Scene" icon.

> **Chromebook note:** No physical F-keys? Use the on-screen "Play Current Scene" icon (top-right, play icon with curved arrow) instead of F6, or hold `Search` + the number key (Search+6 = F6).

**Common gotcha:** an unused function parameter like `delta` can trigger a warning/error if never referenced in the code. Prefix it with an underscore — `_delta` — to tell Godot "I know, I'm intentionally not using this."

---

## Lesson 7: Custom Input (WASD + Arrow Keys)

**Concept: The Input Map** — Godot maps abstract **actions** (like `move_right`) to physical keys. Your script checks actions, not raw keys, so you can rewire input without touching code.

1. `Project → Project Settings → Input Map`
2. Add actions `move_right`, `move_left`, `move_up`, `move_down`, each bound to **D, A, W, S** respectively (type name → Add → click **+** on that row → press the key → OK)
3. **To also support arrow keys:** add a second binding to each of the same four actions (Right, Left, Up, Down arrows) — an action can hold multiple keys; `is_action_pressed` returns true if *any* bound key is held.

Updated script:
```gdscript
extends CharacterBody2D

var speed = 200

func _physics_process(delta):
	var direction = Vector2.ZERO

	if Input.is_action_pressed("move_right"):
		direction.x += 1
	if Input.is_action_pressed("move_left"):
		direction.x -= 1
	if Input.is_action_pressed("move_down"):
		direction.y += 1
	if Input.is_action_pressed("move_up"):
		direction.y -= 1

	velocity = direction.normalized() * speed
	move_and_slide()
```

**Debugging silent failures:** if an action name in code doesn't exactly match one created in the Input Map, nothing crashes — it just silently does nothing. Always double check spelling/existence first.

---

## Lesson 8: Walls and Collision

**Concept:** The Player scene should stay just the player. A separate **Level** scene holds room layout and *instances* the player scene inside it. This keeps things reusable (many rooms can reuse one player) and separates concerns (movement bugs → check player.gd; layout bugs → check the level).

1. `Scene → New Scene` → **2D Scene** → rename root `Level` → save as `scenes/level.tscn`
2. Select `Level` → click the chain-link **"Instantiate Child Scene"** icon → choose `player.tscn` (more reliable than drag-and-drop, especially on trackpads)

**Building a wall:**
1. Right-click `Level` → Add Child Node → `StaticBody2D` → Create → rename `Wall`
   - `StaticBody2D` = never moves but blocks movement (walls, floors). `CharacterBody2D` = for things that move (player, enemies).
2. Add child `ColorRect` to `Wall` — size `200x32`, pick a color
3. Add child `CollisionShape2D` to `Wall` — New RectangleShape2D, size `200x32` (must match the ColorRect)
4. Position the `Wall` node where you want it

**Duplicating walls to build a room:**
- Select the wall's root node (the StaticBody2D) → `Ctrl+D` to duplicate → reposition each copy
- **Watch out:** dragging a duplicate in the viewport while another wall is selected in the Scene panel can accidentally *reparent* it as a child of that wall instead of a sibling. All walls should be siblings, directly under `Level`, at the same indentation level. Fix accidental nesting by dragging the node in the **Scene panel** (not viewport) onto the `Level` root.

Save, run, and confirm the player stops at walls instead of passing through.

---

## Lesson 9: Camera

**Concept:** A `Camera2D` node controls what part of the 2D world is drawn to the screen, and can follow a target automatically.

1. Open `player.tscn` → right-click `Player` → Add Child Node → `Camera2D` → Create (no extra setup needed for basic following)
2. Save, test — camera should stay centered on the player as it moves

**Useful Camera2D properties:**
- **Zoom** (Vector2, default `1,1`) — higher = zoomed in, lower = zoomed out
- **Limit** (Left/Right/Top/Bottom) — caps how far the camera scrolls, so it stops at the room's edge instead of showing empty space beyond the walls. Set these to match your level's actual pixel boundaries (check wall Position values in the Inspector to find the range).

---

## Lesson 10: Side-Scroller Movement (Gravity & Jump)

*(Project pivoted from top-down to a side-scrolling platformer here.)*

Final player movement script with gravity, jump, and double-jump:

```gdscript
extends CharacterBody2D

const SPEED = 300.0
const JUMP_VELOCITY = -400.0
const MAX_JUMPS = 2
var jumps_left = MAX_JUMPS

func _physics_process(delta: float) -> void:
	# Add the gravity.
	if not is_on_floor():
		velocity += get_gravity() * delta
	else:
		jumps_left = MAX_JUMPS

	# Handle jump.
	if Input.is_action_just_pressed("spacebar") and jumps_left > 0:
		velocity.y = JUMP_VELOCITY
		jumps_left -= 1

	# Get the input direction and handle the movement/deceleration.
	var direction := Input.get_axis("move_left", "move_right")
	if direction:
		velocity.x = direction * SPEED
	else:
		velocity.x = move_toward(velocity.x, 0, SPEED)
	move_and_slide()
```

Key concepts:
- `const` — like `var`, but the value can never change after being set. Good for fixed values like speed.
- `JUMP_VELOCITY` is **negative** because Y increases *downward* in Godot — negative Y velocity = moving up.
- `is_on_floor()` — built-in check on `CharacterBody2D`, true when resting on solid ground.
- `get_gravity()` — pulls the project's global gravity setting (`Project Settings → Physics → 2D → Default Gravity`), applied every frame scaled by `delta` for consistent feel regardless of frame rate.
- `is_action_just_pressed` (vs `is_action_pressed`) — fires only on the exact frame a key goes down, not continuously — correct for jump so holding the key doesn't repeatedly launch you.
- `Input.get_axis(negative_action, positive_action)` — returns -1, 0, or 1; a compact way to handle a single left/right axis.
- `:=` — type inference; Godot figures out the variable's type from what's assigned.
- `move_toward(current, target, max_step)` — smoothly slides a value toward a target instead of snapping, giving a bit of deceleration "slide" feel.

**Required Input Map action:** `spacebar` → bound to the Space key.

### Rebuilding the level for platforming

1. Delete/repurpose old walls: keep two, turn one into `Ground` (long ColorRect + matching CollisionShape2D, e.g. `2000x64`, positioned along the bottom)
2. Turn the other into `Platform1` (e.g. `200x32`), positioned up and to the side
3. Duplicate (`Ctrl+D`) to create `Platform2`, `Platform3` at varying heights for a jump-between layout
4. Place `Player` standing on the Ground
5. Update Camera2D **Limit** values to match the new horizontal layout (wide X range, moderate Y range)

---

## Lesson 11: Enemies (Patrol AI)

**Concept:** An enemy is structurally similar to the player (visual + collision + script), but controlled by code instead of input — a simple form of game AI (rules a computer follows to act like it's deciding something, unrelated to machine learning).

1. `Scene → New Scene` → **2D Scene** → delete root, add `CharacterBody2D` instead → rename `Enemy` → save as `scenes/enemy.tscn`
2. Add child `ColorRect` (32x32, red, position `-16,-16`)
3. Add child `CollisionShape2D` (RectangleShape2D, 32x32)
4. Add child `Timer` — **Wait Time**: `2`, **Autostart**: ON

Script (`enemy.gd`):
```gdscript
extends CharacterBody2D

const SPEED = 100.0
var direction = 1

func _physics_process(delta):
	if not is_on_floor():
		velocity += get_gravity() * delta

	velocity.x = direction * SPEED
	move_and_slide()

func _on_timer_timeout():
	direction *= -1
```

- `direction = 1` or `-1` acts as a left/right multiplier
- `_on_timer_timeout()` — a **signal handler**: a function that runs automatically when a specific event (signal) fires, rather than every frame

**Connecting a signal:** select the `Timer` node → **Node** tab (next to Inspector) → find `timeout()` → double-click → confirm it connects to `Enemy` / `_on_timer_timeout` → Connect. A chain-link icon appears next to the Timer once connected.

Instance `enemy.tscn` into `level.tscn` via the chain-link **Instantiate Child Scene** icon, same as the player.

**Common error:** *"Script inherits from native type 'CharacterBody2D', so it can't be assigned to an object of type 'Node2D'"* — means the scene's root node type doesn't actually match what the script `extends`. Fix via right-click the root → **Change Type** → select the correct type (keeps children and script intact). If the error persists after fixing the source scene, the *instance* in the level may be stale — delete it, re-save the source scene, and re-instance it fresh.

---

## Lesson 12: Combat — Player Takes Damage

**Concept:** `Area2D` detects overlap without physically blocking movement — different from the `StaticBody2D`/`CharacterBody2D` collision used for walls and characters. Perfect for "did the player touch danger" without physics side effects.

**Groups** let code ask "is this thing an enemy?" without caring about its exact name.

1. In `enemy.tscn`, select `Enemy` root → **Groups** tab (near Node/signals) → type `enemies` → **Add**
2. In `player.tscn`, add child `Area2D` to `Player` → rename `Hurtbox` → add child `CollisionShape2D` (RectangleShape2D, ~32x32)
3. Add child `Timer` to `Player` → rename `HurtTimer` → **Wait Time**: `1`, **One Shot**: ON → connect `timeout()` → `_on_hurt_timer_timeout`

`player.gd` additions:
```gdscript
var max_health = 5
var health = max_health
var can_take_damage = true

func take_damage(amount):
	if not can_take_damage:
		return
	health -= amount
	print("Player health: ", health)
	can_take_damage = false
	$HurtTimer.start()
	if health <= 0:
		die()

func die():
	print("Player died")
	queue_free()

func _on_hurtbox_body_entered(body):
	if body.is_in_group("enemies"):
		take_damage(1)

func _on_hurt_timer_timeout():
	can_take_damage = true
```

- `can_take_damage` — an invincibility flag preventing a single overlap from draining health across dozens of physics frames
- `take_damage(amount)` — a function with a **parameter**, called manually (not automatically like `_ready`)
- `print(...)` — outputs to the Output panel; the main debugging tool for inspecting variable values while the game runs
- `$HurtTimer` — the `$` symbol is shorthand for "get a child node by name"
- `queue_free()` — built-in function that safely removes a node from the game

Connect `Hurtbox`'s `body_entered(body)` signal to `_on_hurtbox_body_entered`.

---

## Lesson 13: Combat — Attacking & Enemy Health

**Concept:** Same Area2D pattern as Hurtbox, reversed — a **Hitbox** that deals damage rather than receives it, only active during a brief attack window.

1. Input Map: add `attack` action, bind to **J** (or mouse click)
2. `player.tscn`: add child `Area2D` to `Player` → rename `Hitbox` → add `CollisionShape2D` (~32x32) → position it slightly in front of the player, e.g. `20, 0`
3. Add child `Timer` → rename `AttackTimer` → **Wait Time**: `0.2`, **One Shot**: ON → connect `timeout()` → `_on_attack_timer_timeout`

`player.gd` additions:
```gdscript
var is_attacking = false

func _physics_process(delta: float) -> void:
	# ...existing movement/jump code...
	if Input.is_action_just_pressed("attack") and not is_attacking:
		attack()
	# ...

func attack():
	is_attacking = true
	$AttackTimer.start()

func _on_attack_timer_timeout():
	is_attacking = false

func _on_hitbox_body_entered(body):
	if is_attacking and body.is_in_group("enemies"):
		body.take_damage(1)
```

Connect `Hitbox`'s `body_entered` signal to `_on_hitbox_body_entered`.

`enemy.gd` additions:
```gdscript
var health = 3

func take_damage(amount):
	health -= amount
	print("Enemy health: ", health)
	if health <= 0:
		queue_free()
```

---

## Lesson 14: Items & Inventory (Action Bar)

**Concept: Arrays** — a variable holding a *list* of values, indexed from `0`. Used here to represent 4 inventory slots.

### Potion pickup

1. `Scene → New Scene` → **2D Scene** → delete root, add `Area2D` instead → rename `Potion`
2. Add `ColorRect` (24x24, position `-12,-12`) and `CollisionShape2D` (24x24) → save as `scenes/potion.tscn`

Script (`potion.gd`):
```gdscript
extends Area2D

func _on_body_entered(body):
	if body.has_method("add_to_inventory"):
		body.add_to_inventory("potion")
		queue_free()
```

- `body.has_method("...")` — checks if the colliding object has a specific function, an alternative to groups for "can this thing hold items?"

Connect `body_entered` → `_on_body_entered`.

### Inventory array on the player

```gdscript
var inventory = ["", "", "", ""]
var selected_slot = 0

func add_to_inventory(item_name):
	for i in range(inventory.size()):
		if inventory[i] == "":
			inventory[i] = item_name
			print("Picked up: ", item_name, " → slot ", i + 1)
			return
	print("Inventory full!")
```

- `["", "", "", ""]` — array literal syntax; four empty slots
- `for i in range(inventory.size())` — a **for loop**, repeating for each index `0` through `3`
- `inventory[i]` — square brackets look up a specific slot by index

### Action bar UI

**Concept:** UI stays fixed on screen regardless of camera position. `CanvasLayer` draws its contents directly to the screen, ignoring the camera. `Control` is the base type for UI elements.

1. `player.tscn`: right-click `Player` → Add Child Node → `CanvasLayer` → rename `UI`
2. Add child `HBoxContainer` to `UI` → rename `ActionBar` (auto-arranges children horizontally, evenly spaced) → position near the bottom of the screen
3. Add child `Panel` to `ActionBar` → rename `Slot1` → **Custom Minimum Size** `48x48`
4. Add child `Label` to `Slot1` (displays item name)
5. Duplicate `Slot1` three times (`Ctrl+D`) → rename `Slot2`, `Slot3`, `Slot4`

### Input actions for slots and use

`use_item` → **F**; `slot_1`–`slot_4` → **1**, **2**, **3**, **4**

### Wiring code to UI

```gdscript
func _ready():
	update_action_bar()

func _process(_delta):
	if Input.is_action_just_pressed("slot_1"):
		selected_slot = 0
	elif Input.is_action_just_pressed("slot_2"):
		selected_slot = 1
	elif Input.is_action_just_pressed("slot_3"):
		selected_slot = 2
	elif Input.is_action_just_pressed("slot_4"):
		selected_slot = 3

	if Input.is_action_just_pressed("use_item"):
		use_item()

	update_action_bar()

func use_item():
	var item = inventory[selected_slot]
	if item == "":
		return
	if item == "potion":
		health = max_health
		print("Used potion. Health restored to ", health)
		inventory[selected_slot] = ""
		update_action_bar()

func update_action_bar():
	var slots = [$UI/ActionBar/Slot1, $UI/ActionBar/Slot2, $UI/ActionBar/Slot3, $UI/ActionBar/Slot4]
	for i in range(slots.size()):
		var label = slots[i].get_node("Label")
		label.text = inventory[i]
		if i == selected_slot:
			slots[i].modulate = Color(1, 1, 0)
		else:
			slots[i].modulate = Color(1, 1, 1)
```

- `_process(_delta)` — runs every rendered frame (vs. `_physics_process` for physics-step-based logic); convention for input/UI checks
- `modulate` — tints a node's color; `Color(1,1,0)` = yellow highlight for the selected slot, `Color(1,1,1)` = neutral/white

(Remember to also call `update_action_bar()` inside `add_to_inventory()` after picking something up.)

---

## Lesson 15: UI — Heart Health Bar & Game Over

### Hearts

1. `player.tscn`: add child `HBoxContainer` to `UI` → rename `HeartsBar` → position top-left
2. Add child `ColorRect` → rename `Heart1` (24x24, red)
3. Duplicate four times → `Heart2`–`Heart5`

```gdscript
func update_hearts():
	var hearts = [$UI/HeartsBar/Heart1, $UI/HeartsBar/Heart2, $UI/HeartsBar/Heart3, $UI/HeartsBar/Heart4, $UI/HeartsBar/Heart5]
	for i in range(hearts.size()):
		hearts[i].visible = i < health
```

- `i < health` — for `health = 3`, indices 0/1/2 are `true` (visible), 3/4 are `false` (hidden) — hearts drain from the right as health decreases
- `.visible` — built-in property on every node; hides without deleting, so it can reappear later

Call `update_hearts()` in `_ready()`, `take_damage()`, and `use_item()` (anywhere health changes).

### Game Over screen & restart

1. `level.tscn`: right-click `Level` → Add Child Node → `CanvasLayer` → rename `GameOverScreen`
2. Add child `ColorRect` → rename `Background` (black, alpha ~200/255, sized to cover the screen)
3. Add child `Label` → rename `GameOverLabel` → text `Game Over\nPress R to Restart`, larger font size, centered
4. Select `GameOverScreen` → uncheck **Visible** (hidden until death)
5. Input Map: add `restart` → bind to **R**

`level.gd` (attach script to `Level` root):
```gdscript
extends Node2D

func _process(_delta):
	if $GameOverScreen.visible and Input.is_action_just_pressed("restart"):
		get_tree().reload_current_scene()

func show_game_over():
	$GameOverScreen.visible = true
```

- `get_tree()` — accesses the SceneTree, Godot's global manager for the running game
- `reload_current_scene()` — discards current scene state and reloads it fresh from disk, resetting everything

`player.gd`'s `die()`, updated:
```gdscript
func die():
	print("Player died")
	get_parent().show_game_over()
	set_physics_process(false)
	visible = false
```

- `get_parent()` — the direct parent node (here, `Level`)
- `set_physics_process(false)` — stops `_physics_process()` from running on this node, freezing a dead player in place
- We no longer `queue_free()` the player, since `reload_current_scene()` wipes everything clean on restart anyway, and keeping the node around avoids null-reference issues with `get_parent()`

---

## Lesson 16: Polish (Juice) & Export

### Hit flash

`enemy.gd`:
```gdscript
func take_damage(amount):
	health -= amount
	print("Enemy health: ", health)
	flash_white()
	if health <= 0:
		queue_free()

func flash_white():
	$ColorRect.modulate = Color(5, 5, 5)
	await get_tree().create_timer(0.1).timeout
	$ColorRect.modulate = Color(1, 1, 1)
```

- `await` — pauses just *this function* until the thing on the right finishes, without freezing the rest of the game
- `get_tree().create_timer(0.1).timeout` — a lightweight, one-off timer signal, an alternative to adding a full Timer node for quick delays
- `Color(5,5,5)` — values above 1 push a color into overexposed/glowing white for a hit-flash look

Same pattern added to `player.gd`'s `take_damage()`.

### Screen shake

`player.gd`:
```gdscript
func shake_camera():
	var original_pos = $Camera2D.offset
	for i in range(6):
		$Camera2D.offset = original_pos + Vector2(randf_range(-4, 4), randf_range(-4, 4))
		await get_tree().create_timer(0.03).timeout
	$Camera2D.offset = original_pos
```

- `randf_range(-4, 4)` — random decimal between -4 and 4, used to jitter the camera's `offset` briefly before snapping back

Call `shake_camera()` alongside `flash_white()` in `take_damage()`.

### Sound effect

1. Import a short `.wav`/`.ogg` hit sound (e.g. from freesound.org) into a `sounds` folder in FileSystem
2. Add child `AudioStreamPlayer` to `Player` → rename `HitSound` → set **Stream** to the imported file
3. In `take_damage()`: add `$HitSound.play()`

### Exporting a standalone build

1. `Editor → Manage Export Templates` → **Download and Install**
2. `Project → Export` → **Add...** → choose a platform (Windows Desktop is the safest first target)
3. Defaults are fine for most options
4. Click **Export Project** → choose a save location/filename (e.g. `MiniZelda.exe`) → leave "Export With Debug" unchecked for a final build → **Save**
5. Keep the resulting `.exe` and `.pck` files together in the same folder
6. Double-click the `.exe` directly (outside the editor) to confirm it runs standalone

---

## What's Next

**Content expansion (same skills, more of them):**
- More enemy types — copy `enemy.gd`, adjust speed/health/damage, maybe add a ranged attack
- More items — keys/doors, a coin/currency counter, armor that reduces damage
- More rooms — scene transitions via `get_tree().change_scene_to_file("res://scenes/level2.tscn")` triggered from an Area2D at a level's edge

**New systems (new concepts, same overall pattern):**
- **AnimatedSprite2D** — swap ColorRects for real animated sprites once art is available
- **TileMap** — paint levels visually instead of hand-placing individual wall nodes; a big time-saver for bigger worlds
- **Save/load** — using `FileAccess` to write and read health, inventory, and position
- **State machines** — give enemies layered behavior (idle/chase/attack) without the code turning to spaghetti

**Art & audio:**
- Free asset sources: **itch.io** ("free 2D asset pack"), **Kenney.nl**, **OpenGameArt.org**
- **AnimationPlayer** node for animations beyond basic movement, like an attack swing

**Recommended next step:** TileMap — hand-placing walls gets old fast once you want a bigger world, and it directly unblocks building a real dungeon instead of one test room.
