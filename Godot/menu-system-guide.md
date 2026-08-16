# Menu System Guide: Pause Menu, Settings, and the Start Menu

A from-scratch, line-by-line walkthrough of how `ezd`'s menu system was
built: the main/title menu, the in-game pause menu, the settings panel
(mouse sensitivity + fullscreen), the keybindings remap panel, and how
pausing actually freezes the game while the menu itself keeps running.

This is a **guide to the existing implementation**, not a design spec. Where
the implementation made a call that the GDD/TDD leave `[TBD]`, that's called
out explicitly — see [Open decisions carried by this system](#open-decisions-carried-by-this-system).

---

## Table of contents

1. [The big picture](#1-the-big-picture)
2. [How pausing actually works in Godot](#2-how-pausing-actually-works-in-godot)
3. [Build order: how this was actually assembled, commit by commit](#3-build-order-how-this-was-actually-assembled-commit-by-commit)
4. [The Input Map layer (`default_bindings.gd`)](#4-the-input-map-layer-default_bindingsgd)
5. [`GameSettings`: the cross-scene settings store](#5-gamesettings-the-cross-scene-settings-store)
6. [`PauseMenu` walkthrough, line by line](#6-pausemenu-walkthrough-line-by-line)
7. [`pause_menu.tscn` walkthrough, node by node](#7-pause_menutscn-walkthrough-node-by-node)
8. [`KeybindingsPanel` walkthrough, line by line](#8-keybindingspanel-walkthrough-line-by-line)
9. [`MainMenu` walkthrough, line by line](#9-mainmenu-walkthrough-line-by-line)
10. [How `PauseMenu` gets attached to the player](#10-how-pausemenu-gets-attached-to-the-player)
11. [`CharacterMenu`: the second, non-pause menu](#11-charactermenu-the-second-non-pause-menu)
12. [How the menus were tested](#12-how-the-menus-were-tested)
13. [Open decisions carried by this system](#13-open-decisions-carried-by-this-system)
14. [How to extend this system](#14-how-to-extend-this-system)

---

## 1. The big picture

There are actually **three** menu-shaped things in this codebase, and it's
worth telling them apart before diving into code:

| Scene | Type | When it's active | Freezes gameplay? |
|---|---|---|---|
| `scenes/ui/main_menu.tscn` | `Control`, the game's **boot scene** (`run/main_scene` in `project.godot`) | Before a game session exists at all — no `Player` node exists yet | N/A — there's no gameplay running yet |
| `scenes/ui/pause_menu.tscn` | `CanvasLayer`, instanced as a **child of `Player`** | During gameplay, opened with `Esc` | Yes, via `SceneTree.paused` |
| `scenes/ui/character_menu.tscn` | `CanvasLayer`, instanced as a **child of `Player`** | During gameplay, opened with `B`/`I`/`M` | Yes, via `SceneTree.paused` |

`PauseMenu` and `MainMenu` are deliberately near-identical in structure —
both have a `MainPanel` → `SettingsPanel` → `KeybindingsPanel` chain, swapped
by hiding/showing sibling `Control` nodes rather than by changing scenes.
`MainMenu` doesn't reuse `PauseMenu`'s scene file, though; the
`KeybindingsPanel` node tree is hand-duplicated into both `.tscn` files
because both were authored as inline node trees rather than as reusable
sub-scenes (see `main_menu.gd`'s header comment). The `KeybindingsPanel`
**script** (`scripts/ui/keybindings_panel.gd`) is shared — only the scene
node tree is duplicated, not the logic.

`CharacterMenu` is a separate system covered briefly in
[Section 11](#11-charactermenu-the-second-non-pause-menu) — it also pauses
the game but has no settings/keybindings panels of its own.

---

## 2. How pausing actually works in Godot

This is the mechanism the whole system leans on, so it's worth explaining
before any script code. Every `Node` in Godot has a `process_mode` property
with these relevant values:

- `PROCESS_MODE_PAUSABLE` (the default, value `0`) — the node's `_process`,
  `_physics_process`, and input callbacks **stop running** when
  `SceneTree.paused` is `true`.
- `PROCESS_MODE_ALWAYS` (value `3`) — the node keeps running **regardless**
  of `SceneTree.paused`.

Setting `get_tree().paused = true` is a single global switch. Godot doesn't
require you to manually pause every gameplay node — the *default* mode
(`PAUSABLE`) already freezes them for free. The only thing that needs
special handling is **the menu itself**, since a menu that also froze when
`paused = true` would be unable to un-pause itself (its own `Resume` button
wouldn't receive clicks, its own `Esc` handler wouldn't fire).

That's why `PauseMenu` and `CharacterMenu` both set `process_mode = 3`
(`PROCESS_MODE_ALWAYS`) on their scene's root `CanvasLayer` node — visible
as `process_mode = 3` in `pause_menu.tscn` line 7 and `character_menu.tscn`
line 7. Every other node in the game (`Player`, enemies, `CameraRig`,
projectiles, etc.) keeps the `PAUSABLE` default and needs **zero** extra
code to freeze correctly when the menu sets `get_tree().paused = true`.

This is the entire trick. No per-system "am I paused?" checks scattered
through the combat/movement code — Godot's pause propagation does that
uniformly, and the menu opts itself out of it.

---

## 3. Build order: how this was actually assembled, commit by commit

The menu system wasn't built in one pass. Understanding the order clarifies
*why* the code is shaped the way it is (e.g. why `KeybindingsPanel` is a
separate script from day one, or why `MainMenu` came so much later than
`PauseMenu`).

### Commit `fb579a2` — "Add pause/settings/debug menu"

The first version. Added:
- `scripts/ui/pause_menu.gd` + `scenes/ui/pause_menu.tscn`
- Three panels: **Main** (Resume/Settings/Debug/Quit), **Settings** (mouse
  sensitivity slider + fullscreen checkbox — the only two settings wired to
  anything real at the time), **Debug** (Reset Dummy/Reset Level), gated
  behind a `DEBUG_MENU_ENABLED` constant per an explicit user request to be
  able to disable the Debug menu later without deleting the code.
- `CameraRig`'s own `Esc`-handling code was **removed** in this commit,
  because it fought with `PauseMenu` over mouse mode (both were trying to
  toggle `Input.mouse_mode` on the same keypress).

This commit also caught a real, pre-existing engine-config bug while
building the "Reset Dummy" debug button: in `.tscn` files, a node's
`groups=[...]` membership must be part of the `[node ...]` header line —
putting it on a separate property line below silently does nothing. This
meant `LockableDummy` had never actually been in the `"lockable"` group at
runtime since the very first bootstrap commit — a strong candidate for the
root cause of earlier "nothing to lock onto" reports. Fixed in
`test_level.tscn` as a side effect of building the debug menu.

A GUT test written for the menu's panel-switching caught a real bug before
it shipped: `_show_settings_panel()`/`_show_debug_panel()` were only hiding
`_main_panel`, not hiding *each other* — meaning Settings and Debug could
end up stacked on top of one another. Fixed by having every `_show_*_panel`
method explicitly set the visibility of **all** panels, not just the two it
cares about (see [Section 6](#6-pausemenu-walkthrough-line-by-line)).

### Commit `386f2d6` — "Add jump, auto-unlock on target death, and a keybindings remap UI"

Added `scripts/ui/keybindings_panel.gd` as `PauseMenu`'s
Settings → Keybindings sub-panel: lists every input action with its current
key/mouse binding and a live "Rebind" button. Verified against a real
headless Godot 4.7 run of the GUT suite (43/43 passing at the time), not
just read-through — noted explicitly in the commit message because a
UI-input-capture feature like this is easy to get subtly wrong without
actually running it.

### Commit `e5a8ddb` — "Add modular Enemy AI (Disposition/aggro) and menu/inventory system"

Added `CharacterMenu` (`scripts/ui/character_menu.gd` +
`scenes/ui/character_menu.tscn`) as a **second**, separate pause-capable
menu — deliberately not folded into `PauseMenu`, since the GDD's menu-flow
section is itself `[TBD]` and this was a placeholder answer for "where does
inventory/skills/map live," not a finalized design. It follows the same
`CanvasLayer`-child-of-`Player`, `process_mode = ALWAYS` pattern `PauseMenu`
already established.

### Commit `6456da9` — "Add Loop/Time System, World Event scaffold, and a start menu"

Added `MainMenu` (`scripts/ui/main_menu.gd` + `scenes/ui/main_menu.tscn`) as
the game's actual boot scene, replacing whatever `run/main_scene` pointed at
before, and set `run/main_scene="res://scenes/ui/main_menu.tscn"` in
`project.godot`. This is the commit that introduced `GameSettings`
(`scripts/core/game_settings.gd`) as a new autoload — see
[Section 5](#5-gamesettings-the-cross-scene-settings-store) for why it had
to exist: `MainMenu`'s sensitivity slider needs somewhere to write a value
*before* a `Player` node exists to receive it.

---

## 4. The Input Map layer (`default_bindings.gd`)

Before any menu code, it's worth understanding where key bindings actually
live, since `KeybindingsPanel` edits this same system at runtime.

Godot's **Input Map** is a global table: named "actions" (e.g. `"jump"`,
`"attack"`) each mapped to a list of `InputEvent` resources (key, mouse
button, joypad button, joypad axis). Scripts check `Input.is_action_pressed("jump")`
rather than a raw key — this is what `project.godot`'s `[input]` section
registers action *names* for (all with empty `"events": []` — see the file
listing in [Section 1](#1-the-big-picture)'s intro).

`scripts/core/default_bindings.gd` is an **autoload** (registered in
`project.godot [autoload]`, meaning it's instantiated once at game start and
lives for the whole process) that fills in the actual default events at
runtime, rather than hand-authoring binary `InputEvent` resource literals
directly in `project.godot`. The header comment explains why: those literals
are "fragile to edit by hand," so binding via named engine constants
(`KEY_W`, `JOY_AXIS_LEFT_Y`, etc.) in a script is equivalent and much less
error-prone.

Line by line:

```gdscript
const _DEFAULTS := {
	"move_forward": [[KEY_W], [], [JOY_AXIS_LEFT_Y, -1.0]],
	...
}
```

Each entry is `action_name: [keyboard_keys, mouse_buttons, joy_axis_spec, joy_button_index]`.
`keyboard_keys` and `mouse_buttons` are **arrays** (an action can have
multiple keys bound); `joy_axis_spec` is either `null` or a 2-element
`[axis_enum, axis_value]` pair; the trailing `joy_button_index` (when
present) is a single `JoyButton` enum value. The array is *ragged* —
`"jump"`'s entry has no axis spec (index 2 doesn't exist for the "no analog
axis" case) — which is why `_bind_defaults` guards with `spec.size() > 2`
and `spec.size() > 3` before touching those indices.

```gdscript
func _ready() -> void:
	for action_name: String in _DEFAULTS:
		if not InputMap.has_action(action_name):
			continue
		if not InputMap.action_get_events(action_name).is_empty():
			continue
		_bind_defaults(action_name, _DEFAULTS[action_name])
	_bind_lock_on_cancel()
```

- `InputMap.has_action(action_name)` — skip anything in `_DEFAULTS` that
  isn't actually registered in `project.godot`'s `[input]` section (defends
  against the two files drifting out of sync).
- `InputMap.action_get_events(action_name).is_empty()` — **only** binds
  actions that currently have zero events. This is deliberate: it means a
  future save-based rebinding/config system could persist and re-apply
  custom bindings *before* this autoload runs, and this script would then
  see non-empty event lists and back off, never stomping on a saved
  preference. (No such persistence layer exists yet — see
  [Section 13](#13-open-decisions-carried-by-this-system).)

```gdscript
func _bind_defaults(action_name: String, spec: Array) -> void:
	for keycode: Key in spec[0]:
		var event := InputEventKey.new()
		event.physical_keycode = keycode
		InputMap.action_add_event(action_name, event)
	...
```

Builds one `InputEvent` resource per binding and calls
`InputMap.action_add_event()` to attach it to the action. Note
`event.physical_keycode`, not `event.keycode` — this matters later:
`physical_keycode` identifies the *physical* key position (layout-independent,
what you'd want for WASD-style bindings), while `keycode` is the
layout-translated character. `KeybindingsPanel` has to stay consistent with
this choice or its "what key is this bound to" display breaks (see
[Section 8](#8-keybindingspanel-walkthrough-line-by-line)).

```gdscript
func _bind_lock_on_cancel() -> void:
	const ACTION := "lock_on_cancel"
	...
	var key_event := InputEventKey.new()
	key_event.physical_keycode = KEY_TAB
	key_event.shift_pressed = true
	InputMap.action_add_event(ACTION, key_event)
```

A separate, dedicated method for `Shift+Tab` ("force-unlock", skipping the
rest of the lock-on cycle) because the `_DEFAULTS` dictionary format has no
way to express a *required modifier key* — only bare keycodes. Rather than
teach the whole table a new shape for a single caller, this one action gets
its own small binder function. This is a good example of the project's
general "don't over-generalize for one caller" bias.

---

## 5. `GameSettings`: the cross-scene settings store

```gdscript
extends Node
@export var mouse_sensitivity := 0.0025
```

That's the entire file. It's a tiny autoload (registered in
`project.godot [autoload]` as `GameSettings`) whose only job is holding
`mouse_sensitivity` somewhere that exists independent of any particular
scene.

**Why this had to exist:** `PauseMenu`'s sensitivity slider writes directly
to `_player.camera_rig.mouse_sensitivity` — a value that only exists once a
`Player` node is instantiated. But once `MainMenu` became the boot scene
(commit `6456da9`), there's a point in the game's lifecycle — the title
screen — where the Settings panel needs to show and edit sensitivity and
**no `Player` exists yet**. `GameSettings` is the shared value both
`MainMenu` (pre-game) and `PauseMenu` (in-game, via
`_player.camera_rig.mouse_sensitivity = value; GameSettings.mouse_sensitivity = value`)
read from and write to, so the value set on the title screen actually
carries into the game once a `Player` spawns.

Fullscreen doesn't need an equivalent store: `DisplayServer.window_set_mode()`
is already global engine state (not per-`Player`), so both menus read/write
it directly via `DisplayServer.window_get_mode()`/`window_set_mode()`
without needing a settings object in between.

Note what this is *not*: there's no disk persistence here. `GameSettings`
holds the value in memory for the current process only — closing and
reopening the game resets `mouse_sensitivity` to `0.0025`. See
[Section 13](#13-open-decisions-carried-by-this-system).

---

## 6. `PauseMenu` walkthrough, line by line

File: `scripts/ui/pause_menu.gd`.

```gdscript
class_name PauseMenu
extends CanvasLayer
```

`CanvasLayer`, not `Control`, as the script's base/root node type. A
`CanvasLayer` renders its children in **screen space**, above the 3D
viewport, unaffected by any in-world camera — exactly what's wanted for a
full-screen UI overlay. (Contrast with `MainMenu`, which extends `Control`
directly — see [Section 9](#9-mainmenu-walkthrough-line-by-line) for why
that's fine there: there's no 3D scene underneath it to layer over.)

```gdscript
const DEBUG_MENU_ENABLED := true
```

A single boolean gate for the whole Debug panel and its button, per an
explicit user request ("which we will later disable") to be able to hide
debug tooling without deleting the code that backs it. Flipping this to
`false` hides `_debug_button` (see `_ready()` below) — the panel's contents
still exist in the scene tree, just unreachable.

```gdscript
@onready var _main_panel: Control = $MainPanel
@onready var _settings_panel: Control = $SettingsPanel
@onready var _debug_panel: Control = $DebugPanel
@onready var _keybindings_panel: KeybindingsPanel = $KeybindingsPanel
```

`@onready` defers evaluation of `$MainPanel` (Godot's shorthand for
`get_node("MainPanel")`) until the node is actually inside the scene tree,
which is required — you can't call `get_node()` before `_ready()`. These
four are the four mutually-exclusive "screens" the menu can show. Note
`_keybindings_panel` is typed as `KeybindingsPanel` (the `class_name` from
that script), not generic `Control` — this is what lets
`_keybindings_panel.refresh_labels()` type-check later without a cast.

```gdscript
@onready var _resume_button: Button = $MainPanel/CenterContainer/VBoxContainer/ResumeButton
...
@onready var _player: Player = get_parent()
```

Every interactive control gets its own typed `@onready` reference — this is
plain, explicit "cache my node paths" boilerplate, but it's what makes the
rest of the file read as flat, flag-free logic instead of `get_node()` calls
scattered through every method. `_player: Player = get_parent()` is the load-
bearing line for [Section 10](#10-how-pausemenu-gets-attached-to-the-player):
it assumes `PauseMenu`'s *direct parent* in the scene tree is a `Player`
node, which is only true because of how the scene is instanced there.

```gdscript
func _ready() -> void:
	visible = false
	_debug_button.visible = DEBUG_MENU_ENABLED

	_sensitivity_slider.min_value = 0.0005
	_sensitivity_slider.max_value = 0.01
	_sensitivity_slider.step = 0.0005

	_resume_button.pressed.connect(_close)
	_settings_button.pressed.connect(_show_settings_panel)
	_debug_button.pressed.connect(_show_debug_panel)
	_quit_button.pressed.connect(get_tree().quit)
	...
```

- `visible = false` — the menu starts hidden; gameplay is the default state.
- Slider bounds (`0.0005`–`0.01`, step `0.0005`) are set here in code rather
  than in the `.tscn` — keeps the numeric tuning in one place next to the
  logic that uses it.
- `.pressed.connect(...)` wires each `Button`'s `pressed` signal to a
  handler using Godot 4's callable-based signal syntax. `get_tree().quit`
  passed directly (no `()`- it's a `Callable` reference, not a call) means
  clicking Quit calls `SceneTree.quit()` with zero glue code.

```gdscript
func _unhandled_input(event: InputEvent) -> void:
	if event.is_action_pressed("ui_cancel"):
		_close() if visible else _open()
```

This is the `Esc` handler. `"ui_cancel"` is one of Godot's **built-in**
input actions (bound to `Esc` by default project-wide, no entry needed in
`default_bindings.gd`). `_unhandled_input` (not `_input`) means this only
fires for events that no `Control` further up the input-handling chain
already consumed — relevant because `CharacterMenu` deliberately intercepts
`ui_cancel` earlier, in its own `_input()`, to close *itself* first rather
than letting this handler open `PauseMenu` on top of it (see
[Section 11](#11-charactermenu-the-second-non-pause-menu)). The ternary is a
toggle: closed → open, open → close, with no other state to consider.

```gdscript
func _open() -> void:
	visible = true
	_show_main_panel()
	get_tree().paused = true
	Input.set_mouse_mode(Input.MOUSE_MODE_VISIBLE)
	_sensitivity_slider.value = _player.camera_rig.mouse_sensitivity
	_fullscreen_button.button_pressed = DisplayServer.window_get_mode() == DisplayServer.WINDOW_MODE_FULLSCREEN
```

Opening does five things in order:
1. Makes the `CanvasLayer` visible.
2. Resets which sub-panel is showing back to Main (so re-opening the menu
   never resumes on whatever panel it was last closed from).
3. **The actual pause**: `get_tree().paused = true` freezes every
   `PROCESS_MODE_PAUSABLE` node in the tree, per
   [Section 2](#2-how-pausing-actually-works-in-godot).
4. Un-captures the mouse cursor so it can click UI buttons (gameplay
   normally runs with the mouse captured/hidden for camera look).
5. **Syncs the UI to current state** — the sensitivity slider and
   fullscreen checkbox are set to reflect the *actual* current values every
   time the menu opens, rather than trusting stale widget state from last
   time. This matters because sensitivity/fullscreen can also be changed
   from `MainMenu` before the `Player` even existed.

```gdscript
func _close() -> void:
	visible = false
	get_tree().paused = false
	Input.set_mouse_mode(Input.MOUSE_MODE_CAPTURED)
```

The exact inverse: hide, un-pause, re-capture the mouse for gameplay look
controls.

```gdscript
func _show_main_panel() -> void:
	_main_panel.visible = true
	_settings_panel.visible = false
	_debug_panel.visible = false
	_keybindings_panel.visible = false
```

...and three near-identical siblings (`_show_settings_panel`,
`_show_debug_panel`, `_show_keybindings_panel`) each setting **all four**
panels' visibility explicitly, not just the one or two that differ from the
previous call. This shape exists specifically because of the bug caught in
commit `fb579a2` (see [Section 3](#3-build-order-how-this-was-actually-assembled-commit-by-commit)):
an earlier version only toggled the panel being shown/hidden relative to
`_main_panel`, which let two non-Main panels end up visible simultaneously.
Setting every panel's state on every call makes "exactly one panel visible"
an invariant that's true by construction, not by careful sequencing.

`_show_keybindings_panel()` has one extra line beyond the visibility swap:

```gdscript
	_keybindings_panel.visible = true
	_keybindings_panel.refresh_labels()
```

`refresh_labels()` re-reads every action's current binding text right
before the panel is shown — necessary because bindings can change (via
rebinding) while the panel isn't visible, and the cached `Label` text
wouldn't otherwise know to update.

```gdscript
func _on_sensitivity_changed(value: float) -> void:
	_player.camera_rig.mouse_sensitivity = value
	GameSettings.mouse_sensitivity = value
```

Every slider drag writes to **two** places: the live `CameraRig` instance
(so the sensitivity change is felt immediately, mid-game) and
`GameSettings` (so it's remembered if the player later dies/reloads to a
scene where a new `Player`/`CameraRig` gets instantiated — that new
instance needs to pick up the last-set value, not the `0.0025` code
default).

```gdscript
func _on_fullscreen_toggled(pressed: bool) -> void:
	DisplayServer.window_set_mode(DisplayServer.WINDOW_MODE_FULLSCREEN if pressed else DisplayServer.WINDOW_MODE_WINDOWED)
```

Directly toggles the OS window mode — no intermediate state to keep in
sync, since `DisplayServer` itself is the single source of truth (see
[Section 5](#5-gamesettings-the-cross-scene-settings-store)'s note on why
fullscreen skips `GameSettings`).

```gdscript
func _on_reset_dummy_pressed() -> void:
	for dummy: TestDummy in get_tree().get_nodes_in_group("test_dummy"):
		dummy.reset()
```

Debug-panel-only. `get_nodes_in_group("test_dummy")` — note this is the
**permanent** `"test_dummy"` group (added in the same commit that built this
button), distinct from the `"lockable"` group that a dummy loses on death.
Using the permanent group means Reset Dummy can find and revive a dummy even
after it's died and dropped out of `"lockable"`.

```gdscript
func _on_reset_level_pressed() -> void:
	get_tree().paused = false
	get_tree().reload_current_scene()
```

Unpausing **before** reloading matters: `reload_current_scene()` tears down
and reinstantiates the whole current scene tree, and if `paused` were still
`true` when the new tree spawns, the new `Player` would start life already
frozen (or at minimum, the previous pause state would leak across a reload
that's supposed to be a fresh start).

---

## 7. `pause_menu.tscn` walkthrough, node by node

The scene file mirrors the script's `@onready` references exactly — every
`$Path/To/Node` in the script corresponds to a `[node name="..." parent="..."]`
block here. Structure:

```
PauseMenu (CanvasLayer, process_mode=3, script=pause_menu.gd)
├── MainPanel (Control)
│   ├── Dim (ColorRect — 60%-black full-screen overlay)
│   └── CenterContainer
│       └── VBoxContainer
│           ├── TitleLabel  ("Paused")
│           ├── ResumeButton
│           ├── SettingsButton
│           ├── DebugButton
│           └── QuitButton
├── SettingsPanel (Control, starts visible=false)
│   ├── Dim
│   └── CenterContainer → VBoxContainer
│       ├── TitleLabel  ("Settings")
│       ├── SensitivityRow (HBoxContainer) → Label + HSlider
│       ├── FullscreenRow (HBoxContainer) → Label + CheckButton
│       ├── KeybindingsButton
│       └── BackButton
├── KeybindingsPanel (Control, starts visible=false, script=keybindings_panel.gd)
│   ├── Dim
│   └── CenterContainer → VBoxContainer
│       ├── TitleLabel  ("Keybindings")
│       ├── ScrollContainer → RowsContainer (VBoxContainer, populated at runtime)
│       └── BackButton
└── DebugPanel (Control, starts visible=false)
    ├── Dim
    └── CenterContainer → VBoxContainer
        ├── TitleLabel  ("Debug Menu")
        ├── ResetDummyButton
        ├── ResetLevelButton
        ├── SkipToNextDayButton
        └── BackButton
```

A few details worth calling out explicitly:

- **`process_mode = 3` is set once, on the root `PauseMenu` node.** Godot's
  `process_mode` is inherited down the tree by default (`PROCESS_MODE_INHERIT`
  for children unless overridden), so every child — `MainPanel`, every
  `Button`, every `Label` — inherits `ALWAYS` from the root without needing
  the property set individually on each of them.
- **Each panel has its own `Dim` `ColorRect`**, not one shared dimmer behind
  all four panels. Since only one panel is visible at a time (enforced in
  script, [Section 6](#6-pausemenu-walkthrough-line-by-line)), this is
  simpler than trying to share one `Dim` node across panel swaps, at the
  cost of four near-identical `ColorRect` nodes. A reasonable target for
  future de-duplication if the panel count grows.
- **`anchor_right = 1.0` / `anchor_bottom = 1.0`** on `Dim` and the panel
  `Control`s stretches them to fill the full viewport regardless of window
  size, since anchors in Godot's UI system are fractional (`1.0` = 100% of
  the parent's size), not pixel offsets.
- The `SkipToNextDayButton` node exists in the `.tscn` (added later, in
  commit `6456da9`, alongside the Loop/Time System) but wasn't itself shown
  in this walkthrough's earlier commit description — it's wired in the
  script via `_skip_to_next_day_button.pressed.connect(LoopManager.skip_to_next_day)`,
  connecting straight to the `LoopManager` autoload rather than routing
  through a `PauseMenu` method, since there's no `PauseMenu`-local state
  involved.

---

## 8. `KeybindingsPanel` walkthrough, line by line

File: `scripts/ui/keybindings_panel.gd`. This is the most mechanically
interesting piece of the menu system — it does **runtime input remapping**,
not just menu navigation.

**Scope, stated up front in the header comment:** only keyboard/mouse
bindings are shown or editable. Gamepad bindings are left untouched
entirely — rebinding a keyboard key doesn't erase or affect the joypad
button/axis already bound to the same action, so a controller stays fully
functional even after the keyboard side has been remapped away from
defaults.

```gdscript
const _ACTIONS := [
	["move_forward", "Move Forward"],
	...
]
```

A flat list of `[action_name, display_label]` pairs — this is both the
**source of truth for row order** (top to bottom, as declared) and the
mapping from Godot's internal action names to human-readable labels. The
header comment notes it's kept in sync with `default_bindings.gd`'s
`_DEFAULTS` keys by hand — "matches ... exactly (that's the full set of
actions the game defines)." There's no automated check that the two lists
stay in sync; adding a new action means updating both files.

```gdscript
@onready var _rows_container: VBoxContainer = $CenterContainer/VBoxContainer/ScrollContainer/RowsContainer
```

Unlike `PauseMenu`, this panel's rows aren't hand-authored in the `.tscn` —
only the empty `RowsContainer` exists in the scene file. Rows are built
dynamically in `_ready()`:

```gdscript
func _ready() -> void:
	for entry: Array in _ACTIONS:
		_add_row(entry[0], entry[1])
```

```gdscript
func _add_row(action_name: String, display_name: String) -> void:
	var row := HBoxContainer.new()
	_rows_container.add_child(row)

	var name_label := Label.new()
	name_label.custom_minimum_size = Vector2(180, 0)
	name_label.text = display_name
	row.add_child(name_label)

	var key_label := Label.new()
	key_label.custom_minimum_size = Vector2(140, 0)
	key_label.text = _current_binding_text(action_name)
	row.add_child(key_label)
	_key_labels[action_name] = key_label

	var rebind_button := Button.new()
	rebind_button.custom_minimum_size = Vector2(100, 32)
	rebind_button.text = "Rebind"
	rebind_button.pressed.connect(_on_rebind_pressed.bind(action_name))
	row.add_child(rebind_button)
	_rebind_buttons[action_name] = rebind_button
```

For each action, this constructs one `HBoxContainer` row with three
children built entirely from code (`Node.new()`, not scene instancing):
an action-name `Label`, a current-binding `Label`, and a "Rebind" `Button`.
Two dictionaries — `_key_labels` and `_rebind_buttons` — keep a handle to
each row's dynamically-created `Label`/`Button` keyed by action name, since
there's no `@onready $Path` shortcut available for nodes that don't exist
until runtime.

`.pressed.connect(_on_rebind_pressed.bind(action_name))` — `Callable.bind()`
partially applies the `action_name` argument, so every button's `pressed`
signal (which itself carries no arguments) still ends up calling
`_on_rebind_pressed(action_name)` with the *right* action baked in per-row,
without needing 18 separate handler methods.

```gdscript
func _current_binding_text(action_name: String) -> String:
	for event in InputMap.action_get_events(action_name):
		if event is InputEventKey:
			return event.as_text_physical_keycode()
		if event is InputEventMouseButton:
			return event.as_text()
	return "Unbound"
```

Reads back whatever `InputMap` currently has bound and renders it as
display text, returning on the **first** matching key or mouse event found
(so if an action somehow had two keyboard bindings, only the first is
shown — not a concern in practice since rebinding always erases existing
bindings first, see `_apply_rebind` below). `"Unbound"` is the fallback
when the loop finds only joypad events (or nothing at all).

The comment above `event.as_text_physical_keycode()` documents a real,
specific gotcha: every `InputEventKey` this file (and `default_bindings.gd`)
constructs sets `physical_keycode` but leaves `keycode` at its default `0`.
Godot's generic `as_text()` resolves `keycode` **first**, so calling it on
one of these events reads the empty/wrong field and produces blank or
garbled text — `as_text_physical_keycode()` is the accessor that actually
reads the field these events populate. This is exactly the kind of thing
that looks correct in a quick read but silently breaks at runtime, which is
why it's called out with an inline comment rather than left implicit.

```gdscript
var _rebind_action := ""
var _key_labels := {}
var _rebind_buttons := {}

func _on_rebind_pressed(action_name: String) -> void:
	if _rebind_action != "":
		return
	_rebind_action = action_name
	_rebind_buttons[action_name].text = "Press a key..."
```

`_rebind_action` is the panel's "am I currently capturing a key" flag —
empty string means idle. `_on_rebind_pressed` is a guard against starting a
**second** rebind capture while one is already in progress (clicking a
different Rebind button mid-capture is a no-op until the first finishes or
is cancelled), then flips the clicked button's own label to
`"Press a key..."` as the visual cue for which action is currently listening.

```gdscript
func _input(event: InputEvent) -> void:
	if _rebind_action == "":
		return

	if event is InputEventKey and event.pressed and not event.echo:
		if event.physical_keycode == KEY_ESCAPE:
			_cancel_rebind()
		else:
			_apply_rebind(event)
		get_viewport().set_input_as_handled()
	elif event is InputEventMouseButton and event.pressed:
		_apply_rebind(event)
		get_viewport().set_input_as_handled()
```

The header comment explains the choice of `_input` over `_unhandled_input`
explicitly: `Control` nodes normally consume mouse clicks before they'd ever
propagate down to `_unhandled_input`, which would make capturing a
**mouse-button** rebind unreliable (the click that's supposed to *become*
the new binding would get eaten by whatever button is visually under the
cursor first). `_input()` runs earlier in Godot's event-dispatch order, before
GUI processing claims the event, so it reliably sees the raw click.

Inside: only runs its capture logic while `_rebind_action` is non-empty
(otherwise this method would intercept every keypress in the entire game,
including ones meant for other UI). `not event.echo` skips key-repeat
events from a held-down key. `KEY_ESCAPE` is special-cased as "cancel," not
a bindable key — pressing Escape while capturing backs out instead of
binding Escape itself to the action. Any other key or a mouse click gets
passed to `_apply_rebind`. `get_viewport().set_input_as_handled()` marks the
event consumed so it doesn't also reach `PauseMenu`'s own `_unhandled_input`
`Esc`-toggle (which would otherwise close the whole pause menu out from
under an in-progress rebind capture).

```gdscript
func _apply_rebind(event: InputEvent) -> void:
	var action_name := _rebind_action

	for existing in InputMap.action_get_events(action_name):
		if existing is InputEventKey or existing is InputEventMouseButton:
			InputMap.action_erase_event(action_name, existing)

	var new_event: InputEvent
	if event is InputEventKey:
		new_event = InputEventKey.new()
		new_event.physical_keycode = event.physical_keycode
	else:
		new_event = InputEventMouseButton.new()
		new_event.button_index = event.button_index
	InputMap.action_add_event(action_name, new_event)

	_key_labels[action_name].text = _current_binding_text(action_name)
	_rebind_buttons[action_name].text = "Rebind"
	_rebind_action = ""
```

This is the actual remap:
1. **Erase existing bindings first** — loops over every currently-bound
   event for the action and removes any `InputEventKey`/`InputEventMouseButton`
   (explicitly *not* touching `InputEventJoypadButton`/`InputEventJoypadMotion`,
   which is what keeps gamepad bindings alive through a keyboard rebind).
   This guarantees an action never ends up with two conflicting
   keyboard/mouse bindings stacked on top of each other.
2. **Build a fresh event.** Notice it doesn't reuse the incoming `event`
   object directly — it constructs a **new** `InputEventKey`/`InputEventMouseButton`
   and copies over just `physical_keycode` (again, physical not logical, to
   stay consistent with `default_bindings.gd`) or `button_index`. This
   strips out anything else riding along on the captured event (like a
   pressed-state or timestamp) that shouldn't be baked into a persistent
   binding.
3. `InputMap.action_add_event()` — registers the new binding.
4. Refreshes the row's label text and the button's caption back to
   `"Rebind"`, and clears `_rebind_action` back to idle.

```gdscript
func _cancel_rebind() -> void:
	_rebind_buttons[_rebind_action].text = "Rebind"
	_rebind_action = ""


func refresh_labels() -> void:
	for entry: Array in _ACTIONS:
		var action_name: String = entry[0]
		_key_labels[action_name].text = _current_binding_text(action_name)
```

`_cancel_rebind()` just resets the UI without touching `InputMap`.
`refresh_labels()` (called by `PauseMenu`/`MainMenu` every time this panel
is about to become visible) re-reads every action's current text — needed
because rebinding could have happened, then the panel closed and reopened,
and the cached `Label` text needs to catch up to whatever `InputMap`
actually holds now.

**Persistence note:** none of this writes to disk. Rebinding changes
`InputMap` — global engine state — for the running session only. Restarting
the game reloads `project.godot`'s (empty) default events plus whatever
`default_bindings.gd` fills in, discarding any in-session rebinds. Same
caveat as `GameSettings` in [Section 5](#5-gamesettings-the-cross-scene-settings-store).

---

## 9. `MainMenu` walkthrough, line by line

File: `scripts/ui/main_menu.gd`. This is the game's boot scene
(`run/main_scene` in `project.godot`), and structurally it's `PauseMenu`
with the pause-specific pieces removed and a `Start` button added in their
place.

```gdscript
class_name MainMenu
extends Control
```

`Control`, not `CanvasLayer` (contrast `PauseMenu` in
[Section 6](#6-pausemenu-walkthrough-line-by-line)). There's no 3D viewport
running underneath the title screen that this needs to layer *above* — it
**is** the entire screen — so plain `Control` is sufficient and there's no
need for `CanvasLayer`'s screen-space-overlay behavior.

```gdscript
const TEST_LEVEL_SCENE := "res://scenes/test_level/test_level.tscn"
```

Hardcoded path to the scene `Start` loads into. A stand-in for a real
level-select/save-slot flow, which doesn't exist yet — this project only has
one playable level (`test_level.tscn`) at time of writing.

```gdscript
func _ready() -> void:
	Input.set_mouse_mode(Input.MOUSE_MODE_VISIBLE)
	_show_main_panel()

	_sensitivity_slider.min_value = 0.0005
	_sensitivity_slider.max_value = 0.01
	_sensitivity_slider.step = 0.0005
	_sensitivity_slider.value = GameSettings.mouse_sensitivity
	...
```

Unlike `PauseMenu`, the mouse mode is set unconditionally to `VISIBLE` in
`_ready()` rather than only on open/close transitions — the title screen
never needs the mouse captured for camera-look, since there's no camera yet.
Slider bounds match `PauseMenu`'s exactly (kept in sync by hand — same
caveat as `KeybindingsPanel`'s `_ACTIONS` list, no shared constant backs
both). Critically, `_sensitivity_slider.value = GameSettings.mouse_sensitivity`
initializes the slider **from the shared store** — this is the read side of
the `GameSettings` round-trip described in
[Section 5](#5-gamesettings-the-cross-scene-settings-store): if the player
adjusted sensitivity, quit to the title, and came back to Settings, they'd
see their own last value, not the code default.

```gdscript
func _on_start_pressed() -> void:
	get_tree().change_scene_to_file(TEST_LEVEL_SCENE)
```

`change_scene_to_file`, not `paused = false` + hide (there was never a pause
to undo) — this fully unloads `MainMenu`'s scene tree and loads
`test_level.tscn` fresh, which is what actually spawns the first `Player`
node of the session.

```gdscript
func _on_sensitivity_changed(value: float) -> void:
	GameSettings.mouse_sensitivity = value
```

Only writes to `GameSettings` — there's no `_player.camera_rig` to also
update yet (contrast `PauseMenu._on_sensitivity_changed`, which writes to
both). Once `Start` is pressed and a real `Player`/`CameraRig` spawns, that
`CameraRig` needs to read its initial sensitivity from `GameSettings` itself
— that wiring lives in the player/camera-rig code, outside this file's
scope.

The rest of `MainMenu` (`_show_main_panel`, `_show_settings_panel`,
`_show_keybindings_panel`, `_on_fullscreen_toggled`) is structurally
identical to `PauseMenu`'s equivalents minus the Debug panel — see
[Section 6](#6-pausemenu-walkthrough-line-by-line) for the line-by-line
explanation, it isn't repeated here.

---

## 10. How `PauseMenu` gets attached to the player

`PauseMenu` (and `CharacterMenu`) aren't stand-alone scenes loaded
independently — they're **instanced as children directly inside
`player.tscn`**:

```
[node name="PauseMenu" parent="." instance=ExtResource("8")]
[node name="CharacterMenu" parent="." instance=ExtResource("9")]
```

This is why `pause_menu.gd`'s `@onready var _player: Player = get_parent()`
(Section 6) works without any manual wiring — Godot's scene-instancing
system means every time a `Player` scene is instantiated (once per game
session, from `test_level.tscn`), a fresh `PauseMenu` and `CharacterMenu`
come along as children automatically, each with their parent already set to
that exact `Player` instance. There's no singleton/autoload menu — pausing
is scoped per-`Player`, which only matters in a single-player game insofar
as it means the menu's lifetime is tied to the level, not the whole process
(reloading the level via `_on_reset_level_pressed()`, Section 6, naturally
gets a fresh `PauseMenu` for free as part of the new `Player` instance).

`PlayerHud` and `DebugOverlay` (`scripts/ui/player_hud.gd`,
`scripts/debug/debug_overlay.gd`) follow the identical
child-`CanvasLayer`-of-`Player` pattern, though they aren't covered in this
guide since they're HUD elements, not menus.

---

## 11. `CharacterMenu`: the second, non-pause menu

`CharacterMenu` (`scripts/ui/character_menu.gd` +
`scenes/ui/character_menu.tscn`) is worth a brief look since it shares the
pause mechanism but isn't part of the Pause/Settings/Keybindings flow this
guide otherwise focuses on.

- Opened by three dedicated hotkeys — `B` (inventory), `I` (skills &
  talents), `M` (map) — each toggling straight to that specific tab, rather
  than one generic "open menu" key. Pressing the hotkey for the
  already-open tab **closes** the menu (`_toggle_tab`), so `B` alone is a
  full open/close toggle for Inventory.
- Same `process_mode = ALWAYS` / `get_tree().paused = true` /
  `Input.set_mouse_mode(MOUSE_MODE_VISIBLE)` triple as `PauseMenu._open()`,
  for the identical reason (Section 2).
- **The one subtlety worth internalizing:** it overrides `_input()`, not
  `_unhandled_input()`, specifically so it can consume `"ui_cancel"` *before*
  `PauseMenu`'s own `_unhandled_input` handler ever sees it. Since `_input()`
  always runs earlier in Godot's per-frame event dispatch than
  `_unhandled_input()` — regardless of which node is higher in the scene
  tree — this guarantees that pressing `Esc` while `CharacterMenu` is open
  closes *it*, rather than closing nothing (the event still being
  "unhandled") and then also opening `PauseMenu` on top of it. This lets
  `PauseMenu` stay completely unaware that `CharacterMenu` exists — no
  cross-referencing between the two scripts required.

No Settings/Keybindings panels live under `CharacterMenu` — those stay
exclusively under `PauseMenu` (and their `MainMenu` duplicate).

---

## 12. How the menus were tested

All of the menu logic is covered by GUT (Godot Unit Test) suites under
`tests/unit/`, run headless against the real engine rather than only
read-through-verified. Relevant files:

- `tests/unit/test_pause_menu.gd`
- `tests/unit/test_main_menu.gd`
- `tests/unit/test_keybindings_panel.gd`

A pattern worth noting from `test_pause_menu.gd`'s header comment: GUT's own
test runner shares the *same* `SceneTree` as the code under test, so a test
that sets `get_tree().paused = true` and doesn't clean up would leak a
paused tree into whatever test runs next. Every pause-menu test file has an
`after_each()` that unconditionally sets `get_tree().paused = false`,
regardless of whether the test passed or failed, specifically to prevent
that leakage.

Representative tests (see the files directly for the full suites):

- `test_open_pauses_the_tree_and_shows_main_panel` — calls `pause_menu._open()`
  directly (not by simulating an `Esc` keypress) and asserts
  `get_tree().paused`, `pause_menu.visible`, and that exactly `_main_panel`
  is visible among the four panels.
- `test_settings_and_debug_panels_are_mutually_exclusive_with_main` — walks
  Settings → Debug → Main and asserts at each step that only the current
  panel is visible. This is the regression test for the exact bug caught in
  commit `fb579a2` (Section 3).
- `test_reset_dummy_button_resets_the_real_dummy` — notably does **not**
  mock `TestDummy`. It instantiates the *full* `test_level.tscn`, damages
  the real `LockableDummy` node to death, calls the real
  `_on_reset_dummy_pressed()`, and asserts the dummy is alive again and back
  in the `"lockable"` group — testing the actual `get_nodes_in_group()`
  broadcast path, not a stand-in for it.
- `test_sensitivity_slider_initializes_from_game_settings` (in
  `test_main_menu.gd`) — sets `GameSettings.mouse_sensitivity = 0.004`
  *before* instantiating a fresh `MainMenu`, then asserts the new menu's
  slider already shows `0.004`. `0.004` is called out in an inline comment
  as deliberately chosen to be a multiple of the slider's own `0.0005` step
  — using a non-aligned value would get silently snapped by `HSlider`
  itself, which would make the test assert the wrong thing for the wrong
  reason.

---

## 13. Open decisions carried by this system

Per `CLAUDE.md`'s instruction to flag implementation decisions made to
unblock progress rather than treat them as final, here's what this system
currently assumes that hasn't been decided at the design-doc level (GDD
Section 7.1 "Menu Flow" is itself marked `[TBD]`):

| Decision | Where | Why it was needed | Status |
|---|---|---|---|
| No settings persistence to disk | `GameSettings`, `KeybindingsPanel` rebinding | Needed *some* concrete behavior for sensitivity/fullscreen/keybind changes to be testable and usable within a session | Session-only by design for now; a real save-format decision (also `[TBD]`, see `Docs/updates-log.md`) would need to land first |
| `PauseMenu` and `CharacterMenu` as two separate menus, not one | `scripts/ui/pause_menu.gd`, `scripts/ui/character_menu.gd` | GDD's menu flow is undecided; this was a placeholder split (pause/settings/debug vs. character/inventory/skills/map) rather than a finalized IA | Open — could be merged or restructured once GDD 7.1 is resolved |
| `KeybindingsPanel` scene tree duplicated between `pause_menu.tscn` and `main_menu.tscn` instead of extracted into its own `.tscn` | Both scene files | Avoided touching the already-working `PauseMenu` scene while building the unrelated `MainMenu` feature | Open, flagged directly in `main_menu.gd`'s header comment as a reasonable future de-duplication |
| Gamepad bindings are not user-remappable, only keyboard/mouse | `scripts/ui/keybindings_panel.gd` | Scoped down deliberately to ship something working; axis/button remap UI is a larger feature | Open |
| Debug panel ships enabled by default (`DEBUG_MENU_ENABLED = true`) | `scripts/ui/pause_menu.gd` | Explicit user request to keep it toggleable-but-present during active development | Expected to flip to `false` before a real release build |

---

## 14. How to extend this system

A few concrete "if you need to..." pointers, derived from the patterns
above:

- **Add a new rebindable action:** add the action to `project.godot`
  `[input]`, add its default binding(s) to `_DEFAULTS` in
  `scripts/core/default_bindings.gd`, and add a matching
  `["action_name", "Display Name"]` entry to `_ACTIONS` in
  `scripts/ui/keybindings_panel.gd`. All three must be kept in sync by hand
  — there's no single source of truth today (Section 8).
- **Add a new pause-menu setting:** follow the `mouse_sensitivity` pattern —
  add a field to `GameSettings`, add the control (`Slider`/`CheckButton`/etc.)
  to *both* `pause_menu.tscn`'s `SettingsPanel` and `main_menu.tscn`'s
  `SettingsPanel` (they're independent node trees, Section 1), wire
  `@onready` references and a `_on_x_changed` handler in both `.gd` files
  the same way sensitivity is wired.
- **Add a new pause-menu sub-panel** (beyond Main/Settings/Debug/Keybindings):
  add a new `Control` node as a sibling of the existing four panels in
  `pause_menu.tscn`, give it a `_show_<name>_panel()` method that
  explicitly sets *all* panels' visibility (not just the new one) — see
  Section 6's explanation of why that's load-bearing, not stylistic.
- **Persist settings/keybindings across restarts:** this is the biggest gap
  flagged above (Section 13). It would need a real save-format decision
  first (see `Docs/updates-log.md`'s save-format entry), then
  `GameSettings`/`KeybindingsPanel` would read from that store at startup
  instead of always starting from code defaults — `default_bindings.gd`'s
  "only bind if currently empty" check (Section 4) already anticipated this
  and would need no changes to cooperate with a future loader that runs
  before it.
