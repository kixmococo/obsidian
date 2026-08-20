for claude: this is how we initially made the main menu for the gomgom project

# Main Menu Feature — Task Summary

Source ticket: `tickets.md` ("Create a main menu to start the game off for player to see
and have buttons and stuff to use"). Multiplayer wiring was explicitly out of scope per
your instruction — the multiplayer button is present but mocked, not connected to a real
lobby.

## What was built

**New scene:** `project-gomgom/ui/menus/main_menu/main_menu.tscn`
**New script:** `project-gomgom/ui/menus/main_menu/main_menu.gd`
**New test:** `project-gomgom/test/unit/ui/main_menu.gd`
**Edited:** `project-gomgom/project.godot` (`run/main_scene` now points at the main menu
instead of straight into the level)

The main menu has three top-level buttons:

- **Play** — real, working. Calls the existing `GameManager.change_level()` autoload
  method to load `levels/level_01_forest/level_01.tscn`, which already existed in the
  project. This was the one ticket item ("connect the already-existing main level to the
  main game button") that had something real to connect to.
- **Multiplayer** — opens a sub-view with **Host Game** / **Join Game** / **Back**
  buttons, per the ticket's ask for host/join buttons. Since no lobby scene or
  networking exists yet, Host and Join are mocked: pressing either just pops a "still
  cooking, check back soon" overlay instead of navigating anywhere.
- **Settings** — mocked the same way (no settings widget exists in the project yet), pops
  the same "coming soon" overlay.

The "coming soon" overlay is a small reusable panel (dim background + card + message +
OK button) shared by Settings, Host, and Join, so mocking a not-yet-built feature doesn't
require a new panel every time — it's how a Godot UI would typically stub this out.

## How it was done

1. **Read the ticket and the actual project** before writing anything. `tickets.md` is a
   GitLab-issue export with a lot of checklist-UI cruft mixed into the text, so I read
   past that to the actual asks: a main menu scene, a Play button (connect if the level
   already exists), multiplayer host/join buttons (mocked), and a settings button
   (mocked, since no settings widget exists yet).
2. **Checked what already existed** in the repo: `ui/menus/main_menu/` was an empty
   scaffold folder (just a `.gitkeep`), `levels/level_01_forest/level_01.tscn` already
   existed and was already `project.godot`'s `run/main_scene`, and
   `autoload/game_manager.gd` already had a `change_level(path)` helper — so wiring Play
   was just calling that.
3. **Read the design docs** (`docs/GDD.md`, `docs/TDD.md`) for naming/tone conventions —
   e.g. multiplayer is explicitly "client-hosted" (GDD Section 10), which is why the
   buttons are named "Host Game" / "Join Game" rather than something matchmaking-based.
   Reused the character's signature pink (`Color(0.972549, 0.494, 0.674, 1)`, same value
   used on the Player mesh) for menu titles, per the GDD's note that GomGom's pink stays
   the visual identity anchor across the game.
4. **Followed existing code conventions**: checked `entities/player/player.gd` for
   GDScript style (typed vars, tabs, `@onready`) and `gdlintrc`/`gdformatrc` for the
   project's lint/format rules, and `test/unit/version/version.gd` for the GUT test
   style already in use.
5. **Wrote the scene by hand** as a `.tscn` text file (Control root, a `RootView` holding
   `MainView` and `MultiplayerView` as togglable `CenterContainer`s, plus a
   `ComingSoonOverlay`), and the controller script (`main_menu.gd`) that wires button
   `pressed` signals to view-switching functions.
6. **Verified all of it against the real engine**, not just by reading the file back.
   Found a working Godot 4.7.1 binary at `~/Downloads/Godot_v4.7.1-stable_linux.x86_64`
   and used it to:
   - Run `godot --headless --import` to confirm the project (including the new scene)
     imports cleanly with no errors.
   - Run a small throwaway `SceneTree` script to instantiate the menu headlessly, fire
     each button's `pressed` signal, and print/assert the resulting view state —
     confirmed Play actually swaps the running scene to `Level01` via `GameManager`, and
     confirmed the multiplayer/settings/coming-soon panels show and hide correctly.
   - Ran the project's actual GDScript linter/formatter (`gdformat --diff`, `gdlint`) —
     clean, no changes needed, no lint warnings.
   - Ran the full GUT suite the same way GitLab CI does
     (`godot --headless -d -s addons/gut/gut_cmdln.gd -gconfig .testconfig`) — all 10
     tests pass (3 pre-existing + 7 new).
   - Rendered the actual scene with a real GPU/display (there was a live `DISPLAY` and an
     NVIDIA GPU available) and took screenshots of all three menu states. The first pass
     showed the "coming soon" popup was too small — the wrapped message text overlapped
     the OK button and there was no visible card background. Fixed by giving the popup
     panel an explicit `StyleBoxFlat` background and more room, then re-rendered to
     confirm the fix, and re-ran lint/format/tests again to make sure the fix didn't
     break anything.
7. **Added a GUT test file** (`test/unit/ui/main_menu.gd`) covering: initial state (main
   view visible, others hidden), Multiplayer → sub-view, Back → main view, and that
   Settings/Host/Join all trigger the mocked "coming soon" overlay. This matches the
   project's existing testing convention (GUT, `test/unit/`) and TDD.md's stated policy
   of unit-testing UI/system logic rather than relying only on manual verification.
8. Deleted the now-unnecessary `ui/menus/main_menu/.gitkeep` since the folder is no
   longer empty (matches how the rest of the repo only keeps `.gitkeep` in genuinely
   empty scaffold folders).

## What was intentionally left out

- No real multiplayer lobby — per your instruction, ignored ticket item "#10 connect the
  already-existing multiplayer lobby," since no lobby exists and no multiplayer
  networking is set up. Host/Join are mocked, not stubbed-and-broken.
- No real settings widget — same situation (`ui/components/` is empty), so Settings is
  mocked rather than pointing at nothing.
- No pause menu work — out of scope for this ticket (`ui/menus/pause_menu/` is a
  separate, still-empty scaffold folder, untouched).

## Files changed

```
modified:   project-gomgom/project.godot                 (run/main_scene -> main menu)
deleted:    project-gomgom/ui/menus/main_menu/.gitkeep
new file:   project-gomgom/ui/menus/main_menu/main_menu.tscn
new file:   project-gomgom/ui/menus/main_menu/main_menu.gd
new file:   project-gomgom/ui/menus/main_menu/main_menu.gd.uid   (auto-generated by Godot)
new file:   project-gomgom/test/unit/ui/main_menu.gd
```

Nothing has been committed — this is all sitting as working-tree changes in the
`project-gomgom` repo (branch `main`), same as everything was found. Let me know if
you'd like it committed.
