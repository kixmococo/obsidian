# Learning Godot in 3D: Building a Mini Action-Adventure

A companion guide to the 2D Mini-Zelda tutorial — same journey, same concepts, but in three dimensions. Every major section calls out **"2D vs 3D"** so you can see exactly what changes and what stays the same when you move from a flat world to a full one.

---

## Roadmap

1. Foundations (unchanged from 2D)
2. The Godot Editor in 3D
3. Player Scene
4. Mesh & Collision
5. Your First Script (Movement in 3D space)
6. Custom Input
7. Ground, Walls, and Collision
8. Camera (this is the big one)
9. Gravity & Jumping
10. Enemies (Patrol AI)
11. Combat — Player Takes Damage
12. Combat — Attacking & Enemy Health
13. Items & Inventory (UI is still 2D!)
14. UI — Heart Health Bar & Game Over
15. Polish & Export
16. Where to go next

---

## Lesson 1: What Is a Computer Program? (unchanged)

Same as 2D — a computer follows instructions exactly, a program is a list of those instructions, GDScript is the language, Godot is the engine that handles rendering/physics/input so you just write game-specific logic. Nothing about this changes in 3D. If you've read the 2D guide's Lesson 1, skip ahead.

---

## Lesson 2: Installing Godot (unchanged)

Identical process — download from godotengine.org, Standard (non-.NET) version, unzip, run. One thing worth knowing: Godot is *one engine* for both 2D and 3D — you're not installing anything different. You choose 2D or 3D per-scene, and can even mix both in the same project (e.g. a 3D game with a 2D UI overlay, which is actually exactly what we'll do later).

---

## Lesson 3: Creating Your Project

Same steps as 2D (Create New Project → name → path → Create & Edit). One setting matters more here:

- **Renderer**: for 3D, **Forward+** is the modern default and looks best if your computer can handle it. **Mobile** is a lighter-weight option for lower-end hardware. **Compatibility** (what we used for 2D) still works for simple 3D but has fewer visual features. Start with **Forward+** unless your machine struggles, then drop to Mobile.

### Core concept: Scenes and Nodes (unchanged)

Same idea as 2D — Nodes are building blocks, Scenes are trees of nodes saved as reusable units, small scenes nest into bigger ones. This concept is 100% identical in 3D.

> **🔑 2D vs 3D:** The Scene/Node *system* doesn't change at all. What changes is which specific node *types* you use — everything with a `2D` suffix (`Node2D`, `CharacterBody2D`, `CollisionShape2D`) has a `3D` counterpart (`Node3D`, `CharacterBody3D`, `CollisionShape3D`) with an extra dimension of position, rotation, and scale. The naming pattern is consistent enough that once you know 2D, 3D node names are mostly predictable.

---

## Lesson 4: Creating the Player Scene

1. `Scene → New Scene` → choose **3D Scene** (not 2D Scene!) — this creates a `Node3D` root instead of `Node2D`
2. Delete that root, add `CharacterBody3D` instead (search for it same as before)
3. Rename to `Player`
4. Save as `scenes/player.tscn`

> **🔑 2D vs 3D:** `CharacterBody2D` → `CharacterBody3D`. Same job (built-in movement + collision helpers, `move_and_slide()`, `velocity`, `is_on_floor()`), just operating in 3D space. The biggest practical difference: **position and movement now use three numbers (X, Y, Z) instead of two (X, Y).** In 2D, Y was up/down on your screen. In 3D, **Y is up/down in the world (height/gravity), and X/Z form the ground plane** (X = left/right, Z = forward/back). This trips up almost everyone at first — in 2D "up" meant negative Y; in 3D "up" means *positive* Y, and gravity pulls toward *negative* Y.

---

## Lesson 5: Mesh & Collision

**Concept:** Just like a `CharacterBody2D` needed a `ColorRect` (visual) and `CollisionShape2D` (physical shape), a `CharacterBody3D` needs the 3D equivalents.

**Visual (placeholder "art"):**
1. Right-click `Player` → Add Child Node → `MeshInstance3D` → Create
2. In the Inspector, find **Mesh** → click the dropdown → choose a primitive shape, e.g. **New CapsuleMesh** (a good default humanoid placeholder — capsules are the classic "programmer character" shape in 3D, the equivalent of your colored square in 2D)
3. Optionally set a **Material** (right-click the mesh in Inspector → New StandardMaterial3D) and pick an **Albedo Color** to give it some color, same spirit as picking a ColorRect color

**Collision shape:**
1. Right-click `Player` → Add Child Node → `CollisionShape3D` → Create
2. Inspector → **Shape** → **New CapsuleShape3D** → adjust **Radius**/**Height** to roughly match your mesh

Resulting tree:
```
Player (CharacterBody3D)
 ├─ MeshInstance3D
 └─ CollisionShape3D
```

> **🔑 2D vs 3D:** `ColorRect` → `MeshInstance3D` (holds an actual 3D shape/model, not just a flat colored rectangle). `CollisionShape2D` → `CollisionShape3D`, and shapes go from 2D (`RectangleShape2D`, `CircleShape2D`) to 3D (`BoxShape3D`, `CapsuleShape3D`, `SphereShape3D`). In 2D, "placeholder art" was a flat colored square. In 3D, it's a simple 3D primitive (capsule, box, sphere) — same philosophy of "don't wait for real art to start building," just one dimension deeper. **Always double check your mesh size and your collision shape size roughly match**, exactly like you did with ColorRect vs CollisionShape2D — this is a universal 2D/3D habit.

---

## Lesson 6: Your First Script (Movement in 3D space)

Attach a script to `Player` (same process as 2D: right-click → Attach Script → defaults → Create).

```gdscript
extends CharacterBody3D

var speed = 5.0

func _physics_process(delta):
	var input_dir = Vector2.ZERO

	if Input.is_action_pressed("move_right"):
		input_dir.x += 1
	if Input.is_action_pressed("move_left"):
		input_dir.x -= 1
	if Input.is_action_pressed("move_back"):
		input_dir.y += 1
	if Input.is_action_pressed("move_forward"):
		input_dir.y -= 1

	var direction = (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()

	velocity.x = direction.x * speed
	velocity.z = direction.z * speed

	move_and_slide()
```

**Line-by-line, with 2D comparisons baked in:**

- `var speed = 5.0` — same idea as 2D's `speed`, just a much smaller number. **Units in 3D are meters, not pixels** — a speed of `200` (reasonable for 2D pixels/sec) would be absurdly fast for meters/sec in 3D. Expect 3D numbers across the board (speed, jump height, positions) to look tiny compared to their 2D counterparts.
- `Vector2 input_dir` — we still collect raw left/right/forward/back input as a 2D pair (nothing here needs a third dimension — you're not pressing a key for "up/down" movement, that's what jumping and gravity are for)
- `transform.basis * Vector3(...)` — this is new and 3D-specific. It converts your flat 2D input into a 3D direction *relative to which way the object is facing*, using the node's own orientation (`basis` describes rotation). In 2D, direction and screen orientation were basically the same thing; in 3D, an object can face any direction, so input needs to be translated into world-space movement.
- `Vector3` — the 3D equivalent of `Vector2`, holding three numbers (X, Y, Z) instead of two
- `velocity.x` and `velocity.z` — notice we're **not** touching `velocity.y` here at all — that axis is reserved for gravity/jumping (Lesson 9), completely separate from left/right/forward/back movement
- `move_and_slide()` — identical function, identical job, just now operating in 3D

> **🔑 2D vs 3D:** `Vector2` → `Vector3`. The extra axis (Z in this case, since Y is reserved for height) is the core adjustment. Also new: **`transform.basis`**, which has no 2D equivalent in this context — 2D rotation was a single angle number; 3D rotation/orientation is enough more complex that Godot gives you this `basis` tool to correctly convert local input into world-facing direction. Don't worry about deeply understanding basis/rotation math yet — using it exactly as shown above is a standard, safe pattern you'll reuse often.

---

## Lesson 7: Custom Input

Same Input Map system and workflow as 2D — nothing conceptually changes here.

1. `Project → Project Settings → Input Map`
2. Add actions: `move_forward` (W), `move_back` (S), `move_left` (A), `move_right` (D) — same keys as 2D, just renamed from up/down to forward/back since we're not on a flat screen plane anymore
3. Add `jump` → bind to **Space**

> **🔑 2D vs 3D:** No difference at all in *how* the Input Map works. The only change is naming convention — "up/down" made sense for a 2D top-down or platformer game; "forward/back" reads more naturally for 3D movement, though you could name them anything. The underlying mechanism (`Input.is_action_pressed("...")`) is identical code in both.

---

## Lesson 8: Ground, Walls, and Collision

Same "separate Level scene, instance the Player into it" pattern as 2D.

1. `Scene → New Scene` → **3D Scene** → rename root `Level` → save as `scenes/level.tscn`
2. Use the chain-link **Instantiate Child Scene** icon to bring in `player.tscn`

**Building the ground:**
1. Right-click `Level` → Add Child Node → `StaticBody3D` → Create → rename `Ground`
2. Add child `MeshInstance3D` → Mesh → **New PlaneMesh** (a flat horizontal surface — the 3D equivalent of a long floor ColorRect) → set **Size** to something large, e.g. `50 x 50`
3. Add child `CollisionShape3D` → Shape → **New BoxShape3D** (a very flat, wide box works better than a true plane for reliable collision) → set size to match, with a small height like `0.1`

**Building a wall:**
1. Right-click `Level` → Add Child Node → `StaticBody3D` → Create → rename `Wall`
2. Add child `MeshInstance3D` → Mesh → **New BoxMesh** → set **Size**, e.g. `1 x 3 x 10` (width, height, depth)
3. Add child `CollisionShape3D` → Shape → **New BoxShape3D** → match the size

> **🔑 2D vs 3D:** `StaticBody2D` → `StaticBody3D`, same job (solid, immovable, blocks movement — walls, floors). The big new idea: **3D objects have volume in three directions, so every size now has a width, height, AND depth**, versus 2D's width/height only. A 2D wall was a flat rectangle; a 3D wall is an actual box with thickness. `ColorRect` → `PlaneMesh`/`BoxMesh` depending on whether you're making a flat floor or a solid wall — 2D never needed this distinction since everything was already flat.

---

## Lesson 9: Gravity & Jumping

```gdscript
extends CharacterBody3D

var speed = 5.0
var jump_velocity = 4.5

func _physics_process(delta):
	if not is_on_floor():
		velocity += get_gravity() * delta

	if Input.is_action_just_pressed("jump") and is_on_floor():
		velocity.y = jump_velocity

	var input_dir = Vector2.ZERO
	if Input.is_action_pressed("move_right"):
		input_dir.x += 1
	if Input.is_action_pressed("move_left"):
		input_dir.x -= 1
	if Input.is_action_pressed("move_back"):
		input_dir.y += 1
	if Input.is_action_pressed("move_forward"):
		input_dir.y -= 1

	var direction = (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()

	if direction:
		velocity.x = direction.x * speed
		velocity.z = direction.z * speed
	else:
		velocity.x = move_toward(velocity.x, 0, speed)
		velocity.z = move_toward(velocity.z, 0, speed)

	move_and_slide()
```

> **🔑 2D vs 3D:** This is almost identical logic to the 2D platformer's gravity/jump code (`is_on_floor()`, `get_gravity()`, `is_action_just_pressed`, `move_toward` — all the same functions, same behavior). The only real difference: in 2D, gravity affected `velocity.y` while your only horizontal axis was `velocity.x`. In 3D, gravity still affects `velocity.y` (height is always Y in Godot 3D), but now you have **two** horizontal axes to manage (`velocity.x` and `velocity.z`) instead of one. Everything else — the concept of gravity accumulating over time, jump being a one-frame velocity kick, floor detection resetting your jump — carries over exactly.

---

## Lesson 10: Camera (the biggest 2D → 3D difference)

**Concept:** In 2D, the camera was almost an afterthought — attach a `Camera2D`, done, it just follows. In 3D, camera setup is a genuinely bigger decision, because you're choosing *how the player sees the game* — behind the character (third-person), inside the character's head (first-person), or fixed at an angle (isometric/top-down 3D). We'll build a simple third-person follow camera, the most common choice for an action-adventure.

1. Open `player.tscn`, right-click `Player` → Add Child Node → `Node3D` → Create → rename `CameraRig` (a helper container so we can position and rotate the camera without touching the player mesh directly)
2. Right-click `CameraRig` → Add Child Node → `Camera3D` → Create
3. Position the `Camera3D` behind and above the player — e.g. `Position: (0, 2, 5)` (2 units up, 5 units back), and use the **Rotation** field or click-drag in the 3D viewport to angle it slightly downward toward the player

**Basic follow behavior** happens automatically since the camera is a child of the Player (same principle as 2D — child nodes move with their parent). For smoother, more film-like following, a common upgrade is a `SpringArm3D` node instead of a plain `Node3D` — it automatically pulls the camera closer if something (like a wall) gets between it and the player, preventing the camera from clipping through geometry. Worth knowing this exists; not required for your first working camera.

> **🔑 2D vs 3D:** `Camera2D` → `Camera3D`, but the *jump in complexity* here is real, unlike most other 2D→3D swaps in this guide. A 2D camera only ever had one meaningful choice: how far to zoom. A 3D camera has an entire **perspective** to decide — where it sits relative to the player (behind? above? inside their head?), which angle it looks at, and how it handles obstacles getting in the way. There's no 2D equivalent to `SpringArm3D` at all — 2D cameras never have anything physically "in front of" them the way a 3D wall can block your view. Expect to spend more iteration time getting a 3D camera to feel good than you did in 2D.

**Camera limits:** 2D's `Camera2D.Limit` (Left/Right/Top/Bottom) has no direct 3D equivalent — in an open 3D world, you generally don't "wall off" the camera the same way; instead, level geometry itself (walls, cliffs) naturally constrains where the player can go, and the camera just follows.

---

## Lesson 11: Enemies (Patrol AI)

Same structure as 2D: a `CharacterBody3D` with a mesh, collision shape, and a Timer-driven direction flip.

```gdscript
extends CharacterBody3D

var speed = 2.0
var direction = 1

func _physics_process(delta):
	if not is_on_floor():
		velocity += get_gravity() * delta

	velocity.x = direction * speed
	move_and_slide()

func _on_timer_timeout():
	direction *= -1
```

Add to a `enemies` group exactly as in 2D (**Groups** tab on the root node).

> **🔑 2D vs 3D:** Functionally identical to the 2D enemy script — same Timer signal pattern, same direction-flip trick, same gravity handling. The only change is which axis it patrols along (`velocity.x` here; could just as easily be `velocity.z` depending on your level layout) and, as always, smaller speed numbers since we're in meters. This is a good example of how much of your 2D knowledge transfers directly — patrol AI logic doesn't actually care how many dimensions the world has.

---

## Lesson 12: Combat — Player Takes Damage

Same `Area2D`-as-Hurtbox concept, just in 3D.

1. `enemy.tscn`: add to **Groups** → `enemies` (identical to 2D)
2. `player.tscn`: add child `Area3D` to `Player` → rename `Hurtbox` → add child `CollisionShape3D` (CapsuleShape3D or BoxShape3D, roughly matching the player)

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

> **🔑 2D vs 3D:** `Area2D` → `Area3D`, `CollisionShape2D` → `CollisionShape3D`. Every line of GDScript logic here — the invincibility flag, `take_damage()`, `is_in_group()`, `queue_free()`, `$HurtTimer` — is **completely identical to the 2D version**. Signals, groups, and functions don't know or care about dimensions; only the node types (Area2D vs Area3D) and shapes (RectangleShape2D vs CapsuleShape3D) change. This is genuinely one of the most reassuring things about learning Godot: your scripting knowledge is almost entirely dimension-agnostic.

---

## Lesson 13: Combat — Attacking & Enemy Health

Identical pattern to 2D, just in 3D nodes.

1. Input Map: add `attack` → bind to a key or mouse click
2. `player.tscn`: add child `Area3D` to `Player` → rename `Hitbox` → add `CollisionShape3D`, positioned slightly in front of the player (e.g. `Position: (0, 0, -1)` — negative Z is "forward" by Godot's convention)
3. Add `AttackTimer` (Timer, Wait Time `0.2`, One Shot ON) exactly as in 2D

```gdscript
var is_attacking = false

func attack():
	is_attacking = true
	$AttackTimer.start()

func _on_attack_timer_timeout():
	is_attacking = false

func _on_hitbox_body_entered(body):
	if is_attacking and body.is_in_group("enemies"):
		body.take_damage(1)
```

`enemy.gd`:
```gdscript
var health = 3

func take_damage(amount):
	health -= amount
	print("Enemy health: ", health)
	if health <= 0:
		queue_free()
```

> **🔑 2D vs 3D:** Again, the *logic* is unchanged from 2D — only the node types and the fact that "in front of the player" now needs a Z-axis offset instead of an X-axis offset (since forward/back is Z, not X, once you're using the `transform.basis`-based movement from Lesson 6). Worth noting: a real 3D game usually rotates the Hitbox to always point wherever the player is currently facing (using that same `transform.basis` concept from movement) rather than a fixed offset — a nice upgrade for later, not required for a first working version.

---

## Lesson 14: Items & Inventory (UI is still 2D!)

**Concept:** The inventory *array* and picking-up logic works identically to 2D. The pickup item itself becomes a 3D Area — but the **UI stays completely 2D**, because screen interfaces are still flat rectangles even in a 3D game.

### Potion pickup (3D)

1. `Scene → New Scene` → **3D Scene** → delete root, add `Area3D` instead → rename `Potion`
2. Add child `MeshInstance3D` (e.g. New SphereMesh, small) and `CollisionShape3D` (SphereShape3D, matching size)

```gdscript
extends Area3D

func _on_body_entered(body):
	if body.has_method("add_to_inventory"):
		body.add_to_inventory("potion")
		queue_free()
```

### Inventory array (identical to 2D)

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

### Action bar UI

1. `player.tscn`: right-click `Player` → Add Child Node → `CanvasLayer` → rename `UI` — **exactly the same as 2D**
2. Everything inside it — `HBoxContainer`, `Panel`, `Label` — is built exactly as in the 2D guide's Lesson 14

> **🔑 2D vs 3D:** This is an important and slightly surprising rule: **`CanvasLayer` and all `Control`-based UI nodes (`HBoxContainer`, `Panel`, `Label`, `ProgressBar`, etc.) work identically whether your game is 2D or 3D.** A `CanvasLayer` always draws flat on top of the screen, ignoring whether the camera behind it is a `Camera2D` or `Camera3D`. This is why almost every 3D game still has a completely flat, 2D-style HUD (health bars, inventory, menus) — you're not missing a "3D UI" system, you're supposed to keep using the 2D one. The only genuinely 3D UI concept — floating labels/health bars that exist *in* the 3D world and get smaller with distance (like a nameplate over an enemy's head) — uses a different, more advanced node (`Label3D` or a `SubViewport`-based world-space UI) that's worth knowing exists but isn't needed for a first game.

---

## Lesson 15: UI — Heart Health Bar & Game Over

**Entirely identical to the 2D guide.** Hearts (`HBoxContainer` of `ColorRect`s), the `update_hearts()` function, the Game Over `CanvasLayer`/`ColorRect`/`Label`, and `get_tree().reload_current_scene()` all work exactly the same way, because — as established in Lesson 14 — UI doesn't care whether the world behind it is 2D or 3D.

```gdscript
func update_hearts():
	var hearts = [$UI/HeartsBar/Heart1, $UI/HeartsBar/Heart2, $UI/HeartsBar/Heart3, $UI/HeartsBar/Heart4, $UI/HeartsBar/Heart5]
	for i in range(hearts.size()):
		hearts[i].visible = i < health
```

`level.gd` (attach to `Level` root):
```gdscript
extends Node3D

func _process(_delta):
	if $GameOverScreen.visible and Input.is_action_just_pressed("restart"):
		get_tree().reload_current_scene()

func show_game_over():
	$GameOverScreen.visible = true
```

> **🔑 2D vs 3D:** The only change worth flagging: `extends Node2D` becomes `extends Node3D` for the Level root script — everything else, letter for letter, is the same as the 2D version.

---

## Lesson 16: Polish & Export

### Hit flash

In 3D, `modulate` (which worked on 2D ColorRects) doesn't directly apply to a 3D mesh's *material* the same simple way. Instead:

```gdscript
func flash_white():
	var mat = $MeshInstance3D.get_surface_override_material(0)
	if mat == null:
		mat = StandardMaterial3D.new()
		$MeshInstance3D.set_surface_override_material(0, mat)
	mat.albedo_color = Color(5, 5, 5)
	await get_tree().create_timer(0.1).timeout
	mat.albedo_color = Color(1, 1, 1)
```

> **🔑 2D vs 3D:** This is one of the few spots where 3D genuinely needs a different approach, not just a renamed node. 2D's `modulate` property works on anything derived from `CanvasItem` (which includes `ColorRect`) and tints the whole node instantly. 3D visuals are driven by **materials** (`StandardMaterial3D`) attached to a mesh's surface, which control color, shininess, texture, and more — there's more setup, but also far more visual control once you get into real art. The `await get_tree().create_timer(...)` pattern, though, is identical to 2D — that part is dimension-agnostic.

### Screen shake

```gdscript
func shake_camera():
	var original_pos = $CameraRig/Camera3D.position
	for i in range(6):
		$CameraRig/Camera3D.position = original_pos + Vector3(randf_range(-0.1, 0.1), randf_range(-0.1, 0.1), 0)
		await get_tree().create_timer(0.03).timeout
	$CameraRig/Camera3D.position = original_pos
```

> **🔑 2D vs 3D:** Same shaking technique as 2D (`randf_range`, a short loop, snap back to original), just using `Vector3` and much smaller offset numbers (0.1 meters vs 4 pixels) since we're in 3D units. `Camera2D.offset` becomes just nudging the `Camera3D` node's regular `position` directly.

### Sound effect

Identical to 2D: `AudioStreamPlayer` node, set a **Stream**, call `.play()`. (For sounds that should get quieter with distance in a 3D world, Godot has `AudioStreamPlayer3D` — worth knowing it exists, not required for a UI/hit-sound which should always play at full volume regardless of camera distance.)

### Exporting a standalone build

**Entirely identical process to 2D** — Manage Export Templates → Project → Export → Add platform → Export Project. 3D builds are simply larger in file size and more demanding on the player's graphics hardware; the export *workflow* itself doesn't change at all.

---

## Quick-Reference: 2D → 3D Node Translation Table

| 2D | 3D | Notes |
|---|---|---|
| `Node2D` | `Node3D` | Base spatial node |
| `CharacterBody2D` | `CharacterBody3D` | Player/enemy movement |
| `StaticBody2D` | `StaticBody3D` | Walls, floors |
| `Area2D` | `Area3D` | Hurtbox/Hitbox/pickups |
| `CollisionShape2D` | `CollisionShape3D` | Physical shape |
| `RectangleShape2D`/`CircleShape2D` | `BoxShape3D`/`CapsuleShape3D`/`SphereShape3D` | Collision shape types |
| `ColorRect` | `MeshInstance3D` + Mesh (Box/Capsule/Sphere/Plane) | Visual representation |
| `Camera2D` | `Camera3D` (often + `SpringArm3D`) | Viewport control |
| `Vector2` | `Vector3` | Position/direction math |
| `CanvasLayer`, `Control` nodes (`HBoxContainer`, `Panel`, `Label`, etc.) | **Unchanged** | UI stays 2D even in 3D games |
| `modulate` (color tint) | `StandardMaterial3D.albedo_color` | Color/material control |
| GDScript logic (`if`, `for`, functions, signals, groups, arrays) | **Unchanged** | Scripting concepts are dimension-agnostic |

---

## What's Next

- **Lighting** — 3D introduces an entirely new concept 2D never needed: `DirectionalLight3D` (sun-like), `OmniLight3D` (point light), `SpotLight3D`, plus `WorldEnvironment` for sky/fog/ambient light. A flat-lit 3D scene looks noticeably worse than one with basic lighting set up — worth tackling early.
- **Navigation & pathfinding** — `NavigationRegion3D` + `NavigationAgent3D` let enemies path around obstacles intelligently, well beyond the simple back-and-forth Timer patrol.
- **Animation** — `AnimationPlayer` (shared with 2D) plus `AnimationTree` for blending between movement animations (idle/walk/run) smoothly — more commonly needed in 3D since characters are viewed from every angle.
- **Importing real 3D models** — `.glb`/`.gltf` files from Blender, Mixamo (free rigged/animated humanoid models), or asset stores, dropped directly into your FileSystem panel.
- **Free 3D asset sources:** **Kenney.nl** (has 3D packs too), **Sketchfab** (filter by downloadable/free license), **Mixamo** (free character animations), **Poly Haven** (free environment textures/HDRIs for lighting).

**Recommended next step:** basic lighting setup (`DirectionalLight3D` + `WorldEnvironment`) — it's a five-minute addition that makes every single thing you've built so far look dramatically better, before you invest more time in mechanics.
