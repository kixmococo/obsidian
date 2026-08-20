# How-To: Build GomGom's Menu System — Exact Steps Against the Real `project-gomgom` Checkout

**v2 of this guide**, replacing the first draft. That draft worked from the tickets and
[[menu-system-guide]] alone, without looking at actual project code. This version is grounded in a
full read-through of `~/Documents/project-gomgom` — real file paths, real function names, real
gaps. Every step below is meant to be directly actionable, not adapted.

**Before you start:** this checkout has no `.git` directory. Confirm this is actually your tracked
clone of `team-gomgom/project-gomgom` (`git remote -v`) before following the commit steps in
Section 5 — if it's a fresh export, re-clone from GitLab first so the work below lands somewhere
real.

---

## 0. What's actually already true in the repo (read this before writing any code)

This is the single most important section — several tickets are further along than their GitLab
checkboxes suggest, and the codebase already made one architectural choice for you.

| Ticket | What's already real in the repo | The actual gap |
|---|---|---|
| **#18** Main Menu | `ui/menus/main_menu/main_menu.tscn` + `.gd` — fully built. `RootView` holds `MainView` (Play/Multiplayer/Settings) and `MultiplayerView` (Host/Join/Back), swapped by visibility via `show_main_view()`/`show_multiplayer_view()`. Settings + Host + Join currently open a shared `ComingSoonOverlay` (`show_coming_soon(msg)`). Fully covered by `test/unit/ui/main_menu.gd`. | None — ticket is correctly closed. The mocked buttons get de-mocked as a *side effect* of #26 and (out of scope, see §6) real multiplayer wiring, not as further #18 work. |
| **#22** Audio Manager | `autoload/audio_manager.gd` exists and is registered in `project.godot [autoload]` — but it's a thin `AudioStreamPlayer` wrapper (`play_music`/`stop_music`/`play_sfx`). It holds **no volume state, no setter, no signal.** | The ticket's actual asks (dummy master/music/sfx volume, an example setter, an example changed-signal) don't exist yet. "The file already exists" ≠ "the ticket is done." |
| **#23** In-Game Menu | `ui/menus/pause_menu/` is an empty folder (`.gitkeep` only). Worse: `autoload/game_manager.gd`'s `_input()` currently does `if is_in_level and event.is_action_pressed("escape"): return_to_main_menu()` — **Esc already quits straight to the main menu today, with no menu shown at all.** `pause_game()`/`resume_game()` exist and are correct (`get_tree().paused` + an `is_paused` flag) but nothing calls them yet. | Build the actual menu, and replace that `_input()` line — it's the reason nothing appears today. |
| **#26** Main Settings Menu | Doesn't exist — `main_menu.gd`'s Settings button just shows the mock overlay. | Needs a real settings view. |
| **#28** Sound Settings Window | Doesn't exist. | Needs rows wired to #22's (not-yet-built) volume state. |
| **#29** Keybinding Window | `project.godot [input]` currently defines exactly four actions: `move_left`, `move_right`, `jump`, `escape`. **No `crouch` action exists** despite the ticket listing it as an example row. `core/input/` is an empty scaffold folder. | Build the panel against the three real actions; flag `crouch` as not-yet-a-real-action rather than inventing one. |
| **#32** FPS Counter | Doesn't exist. `ui/hud/` is an empty scaffold folder. | Straightforward, zero dependencies. |
| **#34** In-Game Settings Menu | Doesn't exist. | Likely resolves to *filtering*, not new UI — see §4's entry, the ticket's own closing note turns out to be correct here. |

**The one thing the codebase already got right, that you should build on rather than replace:**
`main_menu.gd`'s `RootView → MainView / MultiplayerView` pattern — one scene, sibling `Control`
views swapped by an explicit `show_*_view()` call — is exactly the "one menu, not three" idea from
this guide's first version. Don't invent a new `MenuRoot` framework; extend this existing pattern.

**The one thing the empty folders tell you:** `ui/components/`, `ui/hud/`, and `core/input/` are
scaffolded and empty. That's not incidental — it's the project's own folder layout already
anticipating a shared-component settings panel, a HUD overlay, and an input/keybinding system.
This guide fills those in rather than picking new locations.

---

## 1. Architecture (adjusted for what's actually in this codebase)

### 1.1 Don't build a new CanvasLayer wrapper for the main menu

[[menu-system-guide]] §1/§9 explains why `ezd`'s `PauseMenu` is a `CanvasLayer` (renders over a live
3D viewport) while `MainMenu` is a plain `Control` (nothing underneath it to layer over). The exact
same split applies here, and `main_menu.gd` already made the right call — it `extends Control`.
Leave it alone.

### 1.2 One new shared component: `ui/components/settings_panel/`

This is the concrete fix for the exact tech debt [[menu-system-guide]] §13 flags in `ezd`
(`KeybindingsPanel`'s node tree hand-duplicated between `pause_menu.tscn` and `main_menu.tscn`
because nobody wanted to touch the working scene). You're not in that position — build it once,
correctly, from the start:

```
ui/components/settings_panel/
├── settings_panel.tscn   (Control, tabs or sectioned VBoxContainer: Audio, Controls)
└── settings_panel.gd
```

```gdscript
# ui/components/settings_panel/settings_panel.gd
class_name SettingsPanel
extends Control

signal back_pressed

@export var in_game: bool = false   # true = subset view (ticket #34), false = full (ticket #26)

@onready var _video_section: Control = $VBoxContainer/VideoSection
@onready var _back_button: Button = $VBoxContainer/BackButton


func _ready() -> void:
    _back_button.pressed.connect(func(): back_pressed.emit())
    _apply_mode()


func _apply_mode() -> void:
    # Nothing today is genuinely restart-required (no real Video/graphics
    # settings exist yet — see ticket #34's note in the guide). This is the
    # one row that *would* need gating once a real one exists.
    _video_section.visible = not in_game
```

Instance this **once** inside `main_menu.tscn` (as a new `SettingsView` sibling of `MainView`/
`MultiplayerView` under `RootView`, `in_game = false`) and **once** inside the new
`pause_menu.tscn` (`in_game = true`). Same script, same `.tscn` — instanced twice, never
copy-pasted. Audio rows (#28) and the keybindings panel (#29) live inside this component, so they
automatically appear in both contexts for free.

### 1.3 New scene: `ui/menus/pause_menu/pause_menu.gd` + `.tscn`

`CanvasLayer`, `process_mode = PROCESS_MODE_ALWAYS` — same reasoning as guide §2/§6: it has to
keep receiving input to un-pause itself once `get_tree().paused = true`. Structurally: `MainPanel`
(Resume / Settings / Main Menu / Close Game — Close Game in its own row, away from Main Menu, per
the ticket's explicit anti-misclick request) and one instance of `SettingsPanel` from §1.2.

**What this project does *not* need that `ezd`'s guide spends real space on:** mouse-mode
capture/release (`Input.set_mouse_mode`). Check `entities/player/player.gd` and
`player_camera_rig.gd` — this is a side-scroller-constrained 3D platformer; the camera never
captures the mouse (only mouse-wheel zoom, handled in `_unhandled_input`). There is no
`MOUSE_MODE_CAPTURED` anywhere in this project to release. Skip that part of the reference guide —
porting it in would be dead code.

### 1.4 Ownership: `GameManager`, not `Player` — and here's exactly why

`ezd`'s `PauseMenu` is a child of `Player` (guide §10) because `ezd` is single-player — there's
only ever one `Player`, so "whose pause menu" has one answer. GomGom's GDD/TDD lock **local
co-op up to 4 players AND online co-op with explicit parity** (TDD §1/§4/§5.4) — a per-`Player`
pause menu breaks immediately (whose Esc press wins?).

`GameManager` (`autoload/game_manager.gd`) is already the right owner — it's alive for the whole
process, unaffected by `change_scene_to_file()` (autoloads sit outside the swapped "current scene"
subtree), and it already tracks `is_in_level`/`is_paused`. Concretely:

```gdscript
# autoload/game_manager.gd — add near the top, with the other @onready-style state
const PAUSE_MENU_SCENE := preload("res://ui/menus/pause_menu/pause_menu.tscn")
var _pause_menu: PauseMenu
```

```gdscript
func _ready() -> void:
    _pause_menu = PAUSE_MENU_SCENE.instantiate()
    add_child(_pause_menu)
```

And **replace** the existing escape handler:

```gdscript
# BEFORE (current code — this is why no menu appears today):
func _input(event: InputEvent) -> void:
    if is_in_level and event.is_action_pressed("escape"):
        get_viewport().set_input_as_handled()
        return_to_main_menu()

# AFTER:
func _input(event: InputEvent) -> void:
    if is_in_level and event.is_action_pressed("escape"):
        get_viewport().set_input_as_handled()
        _pause_menu.toggle()
```

`pause_menu.gd`'s Main Menu button calls `GameManager.return_to_main_menu()` — already correct
today (`resume_game()` runs *before* `change_scene_to_file()`, matching guide §6's "unpause before
teardown" lesson exactly). Nothing to fix there, just wire the button to it.

While you're in `game_manager.gd`: `pause_game()`/`resume_game()` should emit `EventBus`'s
already-declared-but-unused `game_paused`/`game_resumed` signals (`autoload/event_bus.gd` — these
exist and nothing fires them yet):

```gdscript
func pause_game() -> void:
    is_paused = true
    get_tree().paused = true
    EventBus.game_paused.emit()


func resume_game() -> void:
    is_paused = false
    get_tree().paused = false
    EventBus.game_resumed.emit()
```

### 1.5 The multiplayer-context flag — be honest about what's actually built

The first version of this guide raised "don't let a solo pause freeze a networked session" as an
open question. Having now read `autoload/net_manager.gd`, `networking/lobby.gd`, and
`entities/player/player.gd`, here's the concrete answer: **there is currently no in-level
multiplayer sync code at all.** `NetManager` only handles the lobby stage (host/connect/player
list). `Player` has zero multiplayer awareness — no `MultiplayerSynchronizer`, no RPCs, no
authority checks. "Multiplayer" today ends at `networking/lobby.tscn`; nothing keeps two players'
game state in sync once a level actually loads.

So: build the `context` distinction into `pause_menu.gd` now (so the API shape is right — e.g. a
`multiplayer_active: bool` the menu can read from `NetManager.peer != null`), but its *behavior*
can be identical to solo for now — both call `GameManager.pause_game()`. Leave an inline comment
and a line in `docs/TDD.md`'s Open Questions table (§15) flagging that this needs revisiting once
real in-level netcode (TDD §5.4/§14's highest-risk module) exists — don't half-build a
network-safe pause for a sync system that isn't there yet, and don't silently forget the question
either.

---

## 2. Build order

Two tracks can run in parallel once §1's shell (pause_menu scene + settings_panel component +
GameManager wiring) exists.

```
Section 1 (shell: pause_menu.tscn, settings_panel.tscn, GameManager wiring)
   │
   ├── Track A — Menu content
   │     #26 (settings_panel in main_menu)  →  #23 (pause_menu)  →  #34 (filter check)
   │
   ├── Track B — Settings data sources
   │     #22 (AudioManager volume state)  →  #28 (sound rows)
   │     #29 (keybindings panel, independent)
   │
   └── Track C — independent, any time
         #32 (FPS counter)
```

| Ticket | Depends on | Why |
|---|---|---|
| #18 | — (closed) | No action needed, see §0 |
| #32 | — | Zero coupling to the menu work |
| #22 | — | `AudioManager` needs extending before anything can read from it |
| #29 | — | Self-contained; only needs `InputMap`, already populated by `project.godot` |
| #26 | §1's `settings_panel` component | Settings button in `main_menu.gd` needs somewhere real to point |
| #23 | §1's `pause_menu.tscn` + `GameManager` wiring | This *is* most of §1 |
| #28 | #22 | Sliders need `AudioManager`'s volume fields to read/write |
| #34 | #23, #26 | Needs both contexts' settings panel instances to compare against |

---

## 3. Ticket-by-ticket exact steps

### #22 — Audio Manager Setup

Edit `autoload/audio_manager.gd` (don't create a new file — extend the existing one):

```gdscript
extends Node

signal audio_settings_changed(bus: String, value: float)

var master_volume := 1.0
var music_volume := 1.0
var sfx_volume := 1.0

var music_player: AudioStreamPlayer
var sfx_player: AudioStreamPlayer


func _ready() -> void:
    music_player = AudioStreamPlayer.new()
    sfx_player = AudioStreamPlayer.new()
    add_child(music_player)
    add_child(sfx_player)


func set_volume(bus: String, value: float) -> void:
    match bus:
        "master": master_volume = value
        "music": music_volume = value
        "sfx": sfx_volume = value
    audio_settings_changed.emit(bus, value)

# play_music / stop_music / play_sfx unchanged below
```

**Test** — `test/unit/audio_manager.gd`, matching `test/unit/ui/main_menu.gd`'s exact shape:

```gdscript
extends GutTest


func test_set_volume_updates_field_and_emits_signal():
    watch_signals(AudioManager)
    AudioManager.set_volume("music", 0.5)

    assert_eq(AudioManager.music_volume, 0.5)
    assert_signal_emitted_with_parameters(
        AudioManager, "audio_settings_changed", ["music", 0.5]
    )
```

**Commit:** `Add dummy volume state and changed-signal to AudioManager (closes #22)`

---

### #29 — Keyboard/Controller Mapping Window

New `ui/components/keybindings_panel/keybindings_panel.gd` + `.tscn`, following
[[menu-system-guide]] §8's blueprint closely — that section is close to a drop-in reference here,
down to the specific gotcha:

- Row list is exactly the three real actions in `project.godot [input]` today:
  `move_left`, `move_right`, `jump`. **Do not add a `crouch` row** — that action doesn't exist in
  the project yet (the ticket lists it only as an example). Note this explicitly in the MR
  description so it isn't mistaken for an oversight.
- `_input()` (not `_unhandled_input()`) for capture, exactly per guide §8 — `Control` nodes would
  otherwise eat a mouse-button rebind before it reaches this handler.
- `event.physical_keycode`, not `event.keycode`, both when reading (`as_text_physical_keycode()`)
  and when constructing new bind events — guide §8 documents this as looking correct and silently
  breaking at runtime; `project.godot`'s existing bindings already use `physical_keycode` (visible
  in the raw `[input]` section), so staying consistent matters here specifically.
- Erase existing keyboard/mouse events before adding a new one (`InputMap.action_erase_event`),
  same as guide §8's `_apply_rebind`.
- **File the follow-up ticket the ticket text explicitly asks for**: "a separate ticket should be
  made for creating a mapping manager autoload... will continually persist (and be saved to disk if
  #27 is done)." Do this now. `core/input/` is already an empty scaffold folder waiting for exactly
  this — don't build persistence yet (there's no save system either — `core/save_system/` is also
  empty, and TDD §5.2's JSON save format isn't implemented), just file the ticket and leave the
  folder as the agreed landing spot.
- Instance this panel inside `SettingsPanel`'s "Controls" section (§1.2).

**Test:** `test/unit/ui/keybindings_panel.gd` — port guide §12's rebind test pattern: rebind
`jump`, assert the old key event is erased and the new one is present via
`InputMap.action_get_events("jump")`.

**Commit:** `Add keyboard rebinding panel for move/jump actions (closes #29)`

---

### #26 — Main Settings Menu

1. Build `SettingsPanel` per §1.2 if not already done as part of the shell.
2. In `main_menu.tscn`, add a `SettingsView` `Control` under `RootView`, sibling to `MainView`/
   `MultiplayerView`, containing one instance of `SettingsPanel` (`in_game = false`).
3. In `main_menu.gd`, add `show_settings_view()` following the exact existing pattern:

```gdscript
@onready var settings_view: Control = $RootView/SettingsView
@onready var settings_panel: SettingsPanel = $RootView/SettingsView/SettingsPanel


func show_settings_view() -> void:
    main_view.hide()
    multiplayer_view.hide()
    settings_view.show()
    coming_soon_overlay.hide()
```

4. Update `show_main_view()`/`show_multiplayer_view()` to also `settings_view.hide()`, matching
   guide §6's "every `_show_*` sets every panel's visibility explicitly" rule — this project's
   current `show_main_view()`/`show_multiplayer_view()` don't yet anticipate a third view, so this
   is a required edit, not optional cleanup.
5. Replace `_on_settings_pressed()`'s body: `show_coming_soon(MOCK_SETTINGS_MESSAGE)` →
   `show_settings_view()`. Connect `settings_panel.back_pressed` to `show_main_view`.
6. Delete `MOCK_SETTINGS_MESSAGE` once nothing references it — don't leave dead mock code sitting
   next to the real implementation.

**Test:** extend `test/unit/ui/main_menu.gd` (same file, matching its existing style) — replace
`test_settings_button_shows_mocked_coming_soon_overlay` with an assertion that `settings_view` is
shown and `main_view`/`multiplayer_view` are hidden.

**Commit:** `Wire Settings button to a real Settings view (closes #26)`

---

### #23 — In-Game Menu

1. Build `pause_menu.tscn`/`.gd` per §1.3, owned by `GameManager` per §1.4.
2. `MainPanel`: Resume, Settings, Main Menu, Close Game — Close Game visually separated (its own
   `HBoxContainer` row or an explicit spacer control) from Main Menu.
3. `toggle()`/`open()`/`close()`:

```gdscript
class_name PauseMenu
extends CanvasLayer

@onready var _main_panel: Control = $MainPanel
@onready var _settings_panel: SettingsPanel = $SettingsPanel


func _ready() -> void:
    process_mode = Node.PROCESS_MODE_ALWAYS
    visible = false


func toggle() -> void:
    close() if visible else open()


func open() -> void:
    visible = true
    _show_main_panel()
    GameManager.pause_game()   # see §1.5 — same call path regardless of multiplayer, for now


func close() -> void:
    visible = false
    GameManager.resume_game()


func _show_main_panel() -> void:
    _main_panel.visible = true
    _settings_panel.visible = false


func _show_settings_panel() -> void:
    _main_panel.visible = false
    _settings_panel.visible = true
```

4. Button wiring: Resume → `close()`. Settings → `_show_settings_panel()`.
   Main Menu → `GameManager.return_to_main_menu()`. Close Game → `get_tree().quit()`.
5. `_settings_panel.in_game = true` (set in the `.tscn` via `@export`, per §1.2).
6. Do the `GameManager` edits from §1.4 (instance `pause_menu`, replace the `_input()` body, add
   the `EventBus` emits).

**Test:** `test/unit/ui/pause_menu.gd` — mirror guide §12's
`test_open_pauses_the_tree_and_shows_main_panel`:

```gdscript
extends GutTest

const PAUSE_MENU_SCENE := preload("res://ui/menus/pause_menu/pause_menu.tscn")

var menu: PauseMenu


func before_each():
    menu = add_child_autofree(PAUSE_MENU_SCENE.instantiate())


func after_each():
    get_tree().paused = false   # guide §12's leak-prevention rule — same shared SceneTree risk


func test_open_pauses_and_shows_main_panel():
    menu.open()
    assert_true(get_tree().paused)
    assert_true(menu.visible)


func test_close_unpauses():
    menu.open()
    menu.close()
    assert_false(get_tree().paused)
```

**Commit:** `Add in-game pause menu, replace escape-to-main-menu shortcut (closes #23)`

---

### #28 — Sound Settings Window

Rows inside `SettingsPanel`'s Audio section — three sliders (Master/Music/SFX):

```gdscript
func _on_music_slider_changed(value: float) -> void:
    AudioManager.set_volume("music", value)
```

Sync on show, same rule as guide §6 step 5 ("sync UI to current state every time the panel
opens"):

```gdscript
func _sync_audio_rows() -> void:
    _music_slider.value = AudioManager.music_volume
    _sfx_slider.value = AudioManager.sfx_volume
    _master_slider.value = AudioManager.master_volume
```

Call `_sync_audio_rows()` from `SettingsPanel._ready()` and again whenever `_apply_mode()`/a
`show`-equivalent runs, so reopening from either `main_menu` or `pause_menu` never shows stale
slider positions.

**Test:** extend `test/unit/ui/settings_panel.gd` — set `AudioManager.music_volume = 0.5` before
instantiating, assert the slider shows `0.5` (mirrors guide §12's
`test_sensitivity_slider_initializes_from_game_settings`).

**Commit:** `Wire Sound Settings sliders to AudioManager (closes #28)`

---

### #34 — In-Game Settings Menu

With `SettingsPanel` already built as one component with an `in_game` flag (§1.2), check what
`_apply_mode()` actually has to hide. Right now: **nothing restart-required exists in this project**
— Audio (#28) and Controls (#29) are both safely changeable mid-level; the only stubbed row
(`_video_section` in §1.2's skeleton) isn't backed by any real graphics setting yet.

- [ ] Confirm this is still true once #26/#28/#29 land (re-check for any new restart-required row).
- [ ] If it's still true: comment on #34 in GitLab explaining the finding — "resolved by #26's
      shared `SettingsPanel` + `in_game` flag, no exclusive main-menu-only settings exist today" —
      and close it without a separate MR. This matches the ticket's own stated exit condition
      ("if there are no settings that are exclusive to the main menu, this ticket can be removed").
- [ ] If a real restart-required setting gets added later (e.g. a resolution/quality option), that
      future ticket sets `_video_section.visible = not in_game` to actually do something — the
      hook already exists, so no rework needed then either.

**Commit:** none required if closing on the finding above; otherwise
`Filter restart-required settings from in-game Settings view (closes #34)`.

---

### #32 — FPS Counter

New `ui/hud/fps_counter/fps_counter.tscn` + `.gd` — fills the empty `ui/hud/` scaffold:

```gdscript
class_name FpsCounter
extends CanvasLayer

@onready var _label: Label = $Label


func _ready() -> void:
    visible = OS.is_debug_build()


func _process(_delta: float) -> void:
    if visible:
        _label.text = "%d FPS" % Engine.get_frames_per_second()
```

Note: no `process_mode = ALWAYS` here — an FPS counter should freeze with the rest of gameplay
when paused, unlike the menu itself. Instance it once from `GameManager._ready()` alongside
`pause_menu`, same persistence rationale as §1.4.

**Test:** `test/unit/hud/fps_counter.gd` — instantiate, assert `visible == OS.is_debug_build()`.

**Commit:** `Add debug-build FPS counter overlay (closes #32)`

---

## 4. Pitfalls checklist (repo-specific, not generic)

- [ ] **Don't set `Input.mouse_mode` anywhere.** This project never captures the mouse — porting
      `ezd`'s capture/release dance from the reference guide would be dead code here (§1.3).
- [ ] **Don't attach `pause_menu` to `Player`.** GDD/TDD lock up to 4-player co-op — `GameManager`
      is the only correct owner (§1.4).
- [ ] **Don't build real network-safe pause behavior for multiplayer yet.** No in-level sync code
      exists (`Player`/`NetManager` confirmed — §1.5). Build the `context` hook, not the full
      solution.
- [ ] **Every `show_*_view()`/`_show_*_panel()` in `main_menu.gd` and `pause_menu.gd` must set every
      sibling view's visibility explicitly**, not just the one changing — guide §3/§6 documents
      this exact bug shipping in `ezd`'s first menu commit. `main_menu.gd`'s current two `show_*`
      methods need this fix the moment `SettingsView` is added (§3, #26 step 4).
- [ ] **`GameManager.pause_game()`/`resume_game()` stay the single source of truth** for
      `get_tree().paused`/`is_paused` — `pause_menu.gd` calls them, never sets `get_tree().paused`
      directly.
- [ ] **One `SettingsPanel` component, instanced twice, never copy-pasted** — this is the concrete
      fix for the tech debt `ezd`'s guide flags in §13.
- [ ] **`physical_keycode`, not `keycode`**, everywhere in the keybindings panel — see #29's entry.
- [ ] **Don't add a `crouch` keybinding row** — the action doesn't exist in `project.godot` yet.
- [ ] **`after_each()` must reset `get_tree().paused = false`** in any GUT test that opens
      `pause_menu` — same shared-`SceneTree` leak risk guide §12 documents.
- [ ] **Run `gdformat` and `gdlint` on every new file** before opening an MR — both are configured
      (`gdformatrc`, `gdlintrc`) and installed in the project's own `Dockerfile` (`gdtoolkit`) for
      exactly this.
- [ ] **Follow `gdlintrc`'s `class-definitions-order`** in new scripts (signals → consts → vars →
      `@onready` vars → functions) — matches the ordering already used in `game_manager.gd` and
      `net_manager.gd`.

---

## 5. Testing & committing, concretely

- Local headless test run (matches `.testconfig`'s configured dirs):
  `godot --headless -s addons/gut/gut_cmdln.gd -gdir=res://test/unit -gexit`
- New test files go under `test/unit/`, mirroring the existing flattened convention (
  `test/unit/ui/main_menu.gd` for `ui/menus/main_menu/main_menu.gd` — drop the `menus/<name>/`
  nesting, keep the category). So: `test/unit/ui/pause_menu.gd`, `test/unit/ui/settings_panel.gd`,
  `test/unit/ui/keybindings_panel.gd`, `test/unit/hud/fps_counter.gd`, `test/unit/audio_manager.gd`.
- Branch naming per `docs/TDD.md` §8's locked GitLab Flow: `feature/<ticket>-<short-name>`, e.g.
  `feature/23-in-game-menu`.
- Commit message style: imperative, one line, `(closes #N)` — this repo has no prior git history
  to mirror locally, so this follows the same convention [[menu-system-guide]] §3 documents from
  `ezd`'s real commit history.
- Open the MR against `main`, targeting GitLab CI once it exists (`docs/TDD.md` §8 flags CI/CD
  tooling as still pending full team sign-off — don't block your MR on CI that isn't wired up yet).
- Tick only the GitLab checkboxes actually satisfied; for #34 specifically, a "closed with no code"
  outcome is the *correct* result if §3's finding holds — don't build UI just to tick a box.

---

## 6. Explicitly out of scope for these 8 tickets (don't build this now)

`autoload/net_manager.gd` and `networking/lobby.gd`/`.tscn` already implement real ENet
host/join/player-list logic — significantly more than "mocked." `main_menu.gd`'s Host/Join buttons
currently point at `show_coming_soon()` instead of this real system. Wiring them together would be
a genuinely good next step, but **no ticket in this batch asks for it** (#18 is already closed
against "(mocked)" acceptance criteria). Flag it to the team as a follow-up ticket rather than
folding it into #18/#23's scope — matches this guide's own rule from v1: finish what's asked, flag
what's adjacent.
