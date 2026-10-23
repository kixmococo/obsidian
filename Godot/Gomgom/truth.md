for claude: this is the click-by-click completion guide for everything built in `c/` to
finish out `tasklist.md`, written for someone who has never programmed, never opened a
game engine, and doesn't know what any of this is. Every module below explains itself
twice: once fast, once painfully slow.

# Truth: How Every Remaining Ticket Got Built (and How to See It Working)

## Read this part first

- **Everything in this guide only exists inside the `c/` folder.** `project-gomgom/` (the
  original project folder, a sibling of `c/`) was left completely untouched, on purpose.
  Every time this guide says "open the project," it means: open the copy that lives in
  `c/`, not `project-gomgom/`. If you ever see two folders that look almost identical and
  you're not sure which one to open — it's `c/`.
- **This was all built by hand-writing files, without ever running the game engine to
  double-check them.** The machine this work was done on doesn't have the Godot game
  engine installed on it. Everything was copied very carefully from patterns that were
  already tested and working (the original Main Menu), but you should treat the
  "verify it" steps in each module below as genuinely necessary, not optional — they're
  the first time any of this gets checked by an actual working game engine.
- **Every module below assumes you've already done the one-time setup right below this.**
  Do that once, then come back here and pick whichever module you want.
- Ticket numbers in parentheses (like `#22`) refer to `tasklist.md` — that's the original
  wishlist this whole guide is working through.

---

## One-time setup: opening the project

You only need to do this once. After this, every module just says "open the project" and
means these same steps.

**Quick version:**
1. Install Godot 4.7.x if you don't have it (godotengine.org → Download → pick the
   "Standard" version, not .NET).
2. Open Godot. Click "Import." Select the `project.godot` file inside `c/`. Click "Import
   & Edit."

**Detailed version:**
1. Godot is the free program that runs and edits this whole game. If it's not already on
   your computer, go to `godotengine.org` in a web browser, click the big "Download"
   button, and get the version that matches what this project needs — you can check by
   opening the file `c/project.godot` in any plain text app and looking for the line that
   says `config/features=PackedStringArray("4.7", ...)` — that "4.7" means you want
   Godot version 4.7 (any small update number after that, like 4.7.1, is fine).
2. Once Godot is installed, open it. The first screen you see is called the **Project
   Manager** — it's a list of every Godot project you've opened before (probably empty
   for you).
3. In the top-right area of that window, click the button labeled **Import**.
4. A file picker window pops up. Navigate to the `c/` folder (the one this whole guide
   keeps mentioning), and inside it, click on the file named exactly `project.godot`.
   Click the **Open** button (or double-click the file).
5. You'll see one more small confirmation window with the project's name and path on it.
   Click **Import & Edit**.
6. Godot will now open the actual editor — a much busier-looking window with several
   panels. This takes a little while the first time (Godot has to process every art
   asset). Just wait for it to finish; you'll know it's done when the loading bar/spinner
   at the bottom goes away and the screen looks calm.
7. You're now looking at the Godot editor with the `c/` copy of the project open. Leave
   this window open — every module below picks up from right here.

A couple of landmarks you'll keep needing, all inside this same editor window:
- **FileSystem panel** — bottom-left by default. It's a folder-tree view of every file in
  the project, exactly like `c/`'s folders on your actual hard drive.
- **The Play button** — top-right corner, a triangle (▶) icon. Clicking it runs the whole
  game, starting from the Main Menu, exactly like a player would experience it.
- **The GUT panel** — a tab at the very bottom edge of the editor window labeled `GUT`.
  This is the automated test runner mentioned throughout this guide — it's a robot that
  double-checks code by actually running little pieces of it and confirming they behave
  correctly, much faster (and more reliably) than clicking through the game by hand every
  time. If you don't see a `GUT` tab at the bottom, click **Project → Project Settings →
  Plugins** in the top menu and confirm "Gut" has a checkmark/is enabled — it should
  already be on, since this project ships with it enabled.

---

## Module 1 — Audio Manager gets real (dummy) volume settings (ticket `#22`)

**Before:** The game had a sound system (`AudioManager`) that could play music and sound
effects, but had no concept of "volume settings" at all — no master volume, no music
volume, nothing a player could ever turn up or down.

**After:** `AudioManager` now holds three volume knobs (Master, Music, SFX), a way to
change any one of them, and it announces (in engine terms: "emits a signal") every time a
change happens, so any other part of the game can react. Music and sound effects that get
played now actually get quieter/louder based on these settings.

This module has no on-screen button of its own yet — it's the plumbing other modules
(3, 5, and 7 below) plug sliders into. You verify this one with the robot test runner
instead of by clicking around.

**Quick version:**
1. Open the project (see setup above).
2. Open the `GUT` panel at the bottom.
3. Find and run the test script at `test/unit/audio/audio_manager.gd`.
4. Confirm every test shows green/passed.

**Detailed version:**
1. With the project open in Godot, click the `GUT` tab at the very bottom of the window.
   A panel opens showing a tree of every test file in the project.
2. In that tree, look for `test` → `unit` → `audio`, and inside it, a file called
   `audio_manager.gd`. This is a script that was written specifically to poke at the new
   volume-settings code and check it behaves correctly.
3. You can either check the box next to just that one file (to run only this test) or
   leave everything checked to run the whole project's test suite (also fine, just
   slower).
4. Click the **Run** button in the GUT panel (usually a ▶-style button near the top of
   that panel).
5. Watch the output area. Each test prints its own name followed by pass/fail. You're
   looking for lines like `test_default_settings_are_full_volume` followed by a green
   checkmark or "passed" — not red/"failed." There are 6 tests in this file; all 6 should
   pass.
6. If you want to see the actual code (not required, just for curiosity): in the
   FileSystem panel, open `autoload` → `audio_manager.gd`. Near the top you'll see
   `var settings := { "master_volume": 1.0, ... }` — that dictionary is the three volume
   knobs. Further down, `func set_setting(...)` is the "turn a knob" function, and
   `signal audio_setting_changed(...)` is the "announcement" other code can listen for.

---

## Module 2 — A debug FPS counter (ticket `#32`)

**Before:** No way to see how fast (or slow) the game is running on screen.

**After:** A small green "FPS: 60" (or whatever number) label appears in the top-left
corner of the screen automatically, any time you run the game from inside the Godot
editor. It does **not** appear in a real exported/released build of the game — only while
testing. You can also press the `F3` key to hide or re-show it on demand.

**Quick version:**
1. Open the project.
2. Press the Play button (▶, top-right) to run the game.
3. Look at the top-left corner of the game window for the green "FPS: ..." text.
4. Press `F3` — it disappears. Press `F3` again — it comes back.
5. Close the game window (or press the red stop button in the Godot editor) when done.

**Detailed version:**
1. Open the project in Godot, as in the one-time setup.
2. In the very top-right corner of the Godot editor window, there's a row of small
   buttons. The leftmost one is a triangle/play icon (▶) — this is the **Play** button
   for the whole project. Click it.
3. A new window opens — this is the actual game running, starting on the Main Menu (the
   same one from `#18`, already built).
4. Look at the very top-left corner of that new game window. You should see small green
   text that says something like `FPS: 60` (the number will bounce around depending on
   your computer). That's the new FPS counter.
5. With that game window focused (click on it once so your keyboard is "talking" to it,
   not the editor), press the `F3` key on your keyboard once. The green text should
   disappear.
6. Press `F3` again. It should reappear.
7. Try clicking "Play" on the main menu to actually enter the level — the FPS counter
   should keep showing in the corner the whole time, since it's meant to help while
   testing anything in the game, not just the menu.
8. When you're done looking, either close the separate game window directly, or go back
   to the Godot editor and click the stop button (a square icon, right next to the Play
   triangle you clicked in step 2).
9. Optional robot check: open the `GUT` panel at the bottom, find
   `test/unit/hud/fps_counter.gd`, and run it the same way as Module 1's test — all tests
   in that file should pass. One of the tests (the `F3` toggle one) is written to
   politely skip itself if it's not running in a debug context, which is normal and not a
   failure.

---

## Module 3 — The Sound Settings piece (ticket `#28`)

**Before:** No screen anywhere in the game let a player change volume.

**After:** A small, reusable "Sound" screen exists — three sliders (Master/Music/SFX) and
a Back button — that reads and writes the volume knobs from Module 1. It was built once,
as its own file, specifically so it could be reused in two different places later
(Module 5's Settings menu, and Module 6/7's in-game pause menu) instead of building it
twice.

Because this piece isn't reachable by clicking anything yet on its own (it only becomes
reachable once Module 5 wires a button to it), you verify this one with the robot test
runner too. You'll get to actually click the sliders in Module 5.

**Quick version:**
1. Open the project.
2. Open GUT, find `test/unit/ui/sound_settings_panel.gd`, run it.
3. Confirm all tests pass.

**Detailed version:**
1. Open the project in Godot.
2. Click the `GUT` tab at the bottom.
3. In the test tree, navigate `test` → `unit` → `ui`, and find `sound_settings_panel.gd`.
4. Run just that file (checkbox + Run, same as Module 1).
5. All 5 tests should pass — they check that moving a slider actually updates
   `AudioManager`'s settings from Module 1, and that the Back button announces itself
   correctly (an "announcement," a.k.a. signal, called `back_requested`) so whatever menu
   is showing this piece knows to switch back to its own previous screen.
6. If you're curious what this looks like visually before Module 5 wires it up anywhere:
   in the FileSystem panel, go to `ui` → `components`, and double-click
   `sound_settings_panel.tscn`. This opens it in Godot's scene editor (the main middle
   area of the screen) so you can see the three sliders and Back button laid out, even
   though nothing points at this file yet.

---

## Module 4 — The Controls (rebinding) piece (ticket `#29`)

**Before:** No way for a player to change which keyboard key does what — Move Left, Move
Right, and Jump were permanently stuck to A, D, and Space.

**After:** A small, reusable "Controls" screen exists, listing those three actions, each
with a button labeled "Rebind." Clicking Rebind, then pressing any key (or clicking a
mouse button), changes that action to the new key on the spot. Like Module 3, this was
built as its own reusable file for the same two-places-need-it reason.

**Quick version:**
1. Open the project.
2. Open GUT, find `test/unit/ui/keybindings_panel.gd`, run it.
3. Confirm all tests pass.

**Detailed version:**
1. Open the project in Godot.
2. Click the `GUT` tab at the bottom.
3. Navigate `test` → `unit` → `ui`, find `keybindings_panel.gd`, run it (same
   checkbox-and-Run process as before).
4. All 5 tests should pass. They check: three rows get built (one per action), clicking
   "Rebind" puts that row into a "waiting for a key press" state, pressing a key actually
   changes the binding, pressing Escape while waiting cancels instead of binding Escape
   itself, and the Back button announces itself the same way Module 3's does.
5. One thing worth understanding, since it's a deliberate choice: the game's own "pause /
   go back" key (Escape) is **not** in this rebindable list — only Move Left, Move Right,
   and Jump are. That's on purpose: Escape is also the key used to cancel a rebind that's
   in progress (see the test in step 4 about pressing Escape), so letting a player
   reassign Escape itself would make that cancel button unreliable.
6. To see it laid out visually (same idea as Module 3): FileSystem panel → `ui` →
   `components` → double-click `keybindings_panel.tscn`.

---

## Module 5 — The real Settings menu, reachable from the Main Menu (ticket `#26`)

**Before:** The Main Menu's "Settings" button showed a "still cooking, check back soon!"
placeholder popup — clicking it did nothing real.

**After:** Clicking "Settings" opens an actual screen: a Fullscreen on/off switch, plus
"Sound" and "Controls" buttons that open Module 3's and Module 4's pieces, plus a "Back"
button that returns to the main menu. The Multiplayer screen's "Host"/"Join" buttons still
show the same "still cooking" popup as before (those are still out of scope, per the
original ticket) — but that popup itself got tidied up behind the scenes into its own
reusable file too, the same way the Sound and Controls pieces were, so it's ready to be
reused again later without copy-pasting it a second time.

**Quick version:**
1. Open the project, press Play.
2. On the Main Menu, click **Settings**.
3. Toggle **Fullscreen**, confirm the window actually goes fullscreen/windowed.
4. Click **Sound** → move a slider → click **Back**.
5. Click **Controls** → click **Rebind** on any row → press a new key → confirm the label
   updates → click **Back**.
6. Click **Back** again to return to the main menu.
7. Optional: click **Multiplayer** → **Host Game**, confirm the "still cooking" popup
   still appears like before.

**Detailed version:**
1. Open the project in Godot (one-time setup above, if you haven't already this session).
2. Click the ▶ Play button, top-right of the editor. A new window opens showing the Main
   Menu — the same GomGom title screen from before, with **Play**, **Multiplayer**, and
   **Settings** buttons stacked in the middle.
3. Click the **Settings** button with your mouse.
4. You should now see a new screen titled **Settings**, with, from top to bottom: a row
   that says "Fullscreen" with a switch/checkbox next to it, a **Sound** button, a
   **Controls** button, and a **Back** button.
5. Click the switch next to "Fullscreen." The whole game window should immediately expand
   to fill your entire screen (or, if it was already fullscreen, shrink back down to a
   normal window). Click it again to flip it back. This is a real, live setting — not a
   mock.
6. Click the **Sound** button. The screen changes to one titled **Sound**, with three
   horizontal sliders labeled "Master Volume," "Music Volume," and "SFX Volume," plus a
   **Back** button underneath.
7. Click and drag any of the three sliders left or right. There's no music currently
   playing to hear the difference by ear in this exact spot, but the value is being saved
   live — you already confirmed the underlying plumbing works in Module 1's robot test.
8. Click **Back** (the one on this Sound screen). You should land back on the **Settings**
   screen from step 4 — not the main title screen. This "Back goes one level up, not all
   the way out" behavior is intentional.
9. Click the **Controls** button instead. The screen changes to one titled **Controls**,
   listing three rows: "Move Left," "Move Right," and "Jump," each showing its current key
   (e.g., "A", "D", "Space") and a **Rebind** button next to it.
10. Click **Rebind** next to "Jump." Its button text changes to "Press a key...". Now
    press any key on your keyboard, for example `K`. The row's key label should instantly
    update to show `K`, and the button goes back to saying "Rebind."
11. Go actually play the level (see Module 6 below, or just click Back twice to reach the
    main menu, then click **Play**) and confirm the new key actually makes the character
    jump — that's the real proof the rebind took effect, not just a label change.
12. Back on the Controls screen, click **Back** — you should land on the **Settings**
    screen again, same rule as step 8.
13. Click **Back** once more on the Settings screen — you should now be all the way back
    on the original main title screen (Play / Multiplayer / Settings buttons).
14. Optional, to confirm nothing about the still-mocked pieces broke: click
    **Multiplayer**, then click **Host Game**. You should see the same "This feature is
    still cooking — check back soon!" popup that existed before any of this work started.
    Click its **OK** button to dismiss it, then click **Back** to return to the main menu.

---

## Module 6 — The in-game Pause Menu, and its own Settings screen (tickets `#23` and `#34`)

These two tickets were built together on purpose, because ticket `#23` itself asks for a
Settings button inside the pause menu, and ticket `#34` is exactly what that button should
open — so there was never a version of `#23` worth building without `#34` sitting right
next to it. Splitting them into two separate click-throughs below would just mean doing
the exact same clicks twice, so this module covers both.

**Before:** Pressing Escape during a level immediately, silently kicked you all the way
back to the main menu — no confirmation, no pause, no way to just glance at settings and
keep playing.

**After:** Pressing Escape during a level opens a pause popup with **Resume**,
**Settings**, **Main Menu**, and **Close Game** buttons. The game world visibly freezes
behind it (nothing moves), the popup's own buttons still work while frozen, and pressing
Escape again (or clicking Resume) un-freezes everything and closes the popup. Its
**Settings** button opens the exact same Sound and Controls pieces from Modules 3 and 4 —
built once, reused here instead of rebuilt.

One thing worth understanding, since it was a deliberate requirement: **this whole pause
feature is single-player only.** If the game were ever in an active multiplayer session,
pressing Escape wouldn't pause anything — because pausing works by freezing the *entire*
game world, which in multiplayer would freeze it for every connected player, not just the
one who pressed Escape. Since Host/Join are still just mocked placeholder buttons (see
Module 5, step 14) and don't actually start a real multiplayer session yet, you won't be
able to see this "does nothing in multiplayer" behavior for yourself right now — every
real play session today is single-player by default, so Escape will always work exactly as
described above. This is just future-proofing that's already in place and ready for
whenever real multiplayer gets built.

**Quick version:**
1. Open the project, press Play.
2. Click **Play** on the main menu to enter the level.
3. Press `Esc`. Confirm the world freezes and the Pause popup appears.
4. Click **Settings** → **Sound**/**Controls**, confirm they're the same screens as
   Module 5, and confirm changes here affect real gameplay (e.g., rebind Jump, close the
   menu, test the new key).
5. Press `Esc` again (or click **Resume**) — confirm the world un-freezes.
6. Open the pause menu again, click **Main Menu** — confirm you land back on the title
   screen.

**Detailed version:**
1. Open the project in Godot, click ▶ Play.
2. On the Main Menu, click **Play**. The screen changes to the actual 3D level — you
   should see the pink GomGom character and be able to move it left/right with `A`/`D`
   and jump with `Space` (or whatever you rebound Jump to in Module 5).
3. Move the character around for a second to confirm the level is really running (not
   frozen).
4. Press the `Esc` key on your keyboard.
5. Everything in the level should freeze exactly where it is — try pressing a movement
   key; the character should not move at all. At the same time, a dark, dimmed popup
   appears on top of the frozen level, titled **Paused**, with four buttons stacked
   underneath: **Resume**, **Settings**, **Main Menu**, and then, with a small gap after
   it, **Close Game** — that gap is intentional, so "Main Menu" and "Close Game" aren't
   sitting right next to each other where you could misclick one for the other.
6. Click **Settings**. The popup's contents change to a small **Settings** screen with
   **Sound** and **Controls** buttons and a **Back** button — no Fullscreen switch here
   (that one lives only in the main menu's Settings, since it's more of a "before you
   start playing" kind of setting).
7. Click **Sound**. You'll see the exact same three sliders (Master/Music/SFX Volume) as
   Module 5 — this is the literal same reusable piece, just opened from inside the level
   instead of from the main menu. Drag a slider, then click **Back** — you land back on
   this pause menu's own Settings screen (step 6), not the main menu's.
8. Click **Controls** instead. Same idea — the identical Rebind rows from Module 5/Module
   4. Click **Rebind** next to "Jump," press a different key than whatever it's currently
   set to (for example, if it's currently `K` from Module 5's walkthrough, try `L`
   instead), confirm the label updates, then click **Back**.
9. Click **Back** again on the Settings screen — you're back on the main **Paused** popup
   from step 5.
10. Click **Resume**. The popup disappears and the level un-freezes — the character should
    respond to movement again immediately.
11. Try the key you just rebound in step 8 (e.g., `L`) — the character should jump. This
    confirms the rebind you did *while paused, mid-level* took effect on the actual
    running game, not just on a label.
12. Press `Esc` again to reopen the pause popup, just to confirm the toggle direction
    works both ways (Resume via button in step 10, now via key press again in step 12).
13. Press `Esc` one more time to close it again (confirming the key itself also toggles
    closed, not just open).
14. Reopen the pause popup (`Esc`) one last time and click **Main Menu**. You should be
    taken all the way back to the title screen — Play / Multiplayer / Settings — exactly
    like clicking "Main Menu" was always meant to feel like a deliberate, considered way
    to leave, not an accident.
15. (Optional, and only do this last since it closes the whole game): from inside a level,
    open the pause popup and click **Close Game**. The entire game window should close.
    This is the correct, intended behavior for that button — there's nothing to undo
    afterward, so only click it when you're actually done testing.
16. Optional robot check: open the `GUT` panel, find `test/unit/ui/pause_menu.gd` under
    `test` → `unit` → `ui`, and run it. All tests should pass, including two specifically
    about the single-player-only rule described above (one confirms Escape pauses when
    `NetManager` has no active connection, the other confirms it does nothing when
    `NetManager` does have one).

---

## Final checkpoint: every ticket in `tasklist.md`, confirmed

Here's the full list from `tasklist.md`, and where each one landed:

| Ticket | What it needed | Where it lives now | Module above |
|---|---|---|---|
| `#18` Main Menu | Already done before this session started | `ui/menus/main_menu/` | — (pre-existing) |
| `#22` Audio Manager Setup | Dummy audio settings + example method + signal | `autoload/audio_manager.gd` | Module 1 |
| `#32` FPS Counter | On-screen FPS, debug-only | `autoload/fps_counter.gd` | Module 2 |
| `#28` Sound Settings Window | Adjustable sound settings, connected to Audio Manager | `ui/components/sound_settings_panel.tscn` | Module 3 (built), Module 5 & 6 (used) |
| `#29` Keyboard/Controller Mapping Window | Adjustable key bindings | `ui/components/keybindings_panel.tscn` | Module 4 (built), Module 5 & 6 (used) |
| `#26` Main Settings Menu | Reachable from main menu, has Back, mocked settings | `ui/menus/main_menu/main_menu.tscn` (`SettingsView`) | Module 5 |
| `#23` In-Game Menu | Esc opens/closes it, Resume/Main Menu/Close Game/Settings buttons | `ui/menus/pause_menu/pause_menu.tscn` | Module 6 |
| `#34` In-Game Settings Menu | Reachable from the pause menu, mocked settings | `ui/menus/pause_menu/pause_menu.tscn` (`SettingsView`) | Module 6 |

**One continuous playtest that touches every single item on this list, start to finish:**

1. Open the project, click ▶ Play.
2. On the title screen, notice the small green FPS counter in the corner (`#32`) —
   press `F3` to confirm it can hide/reappear.
3. Click **Settings** (`#26`), flip the Fullscreen switch once, open **Sound** (`#28`,
   `#22`) and nudge a slider, click Back, open **Controls** (`#29`) and rebind Jump to a
   new key, click Back twice to return to the title screen.
4. Click **Play** to enter the level.
5. Move around and jump using your newly rebound key, confirming Module 5's rebind
   carried into the real level.
6. Press `Esc` (`#23`) — confirm the world freezes and the pause popup appears with
   Resume / Settings / Main Menu / Close Game.
7. Click **Settings** inside the pause popup (`#34`), open **Sound**, nudge a slider,
   Back, open **Controls**, rebind a different action, Back, Back again to the main pause
   screen.
8. Click **Resume** — confirm the world un-freezes and your latest rebind works in
   gameplay.
9. Press `Esc`, then click **Main Menu** — confirm you land back on the title screen
   cleanly.
10. Every ticket in `tasklist.md` has now been clicked through, in one sitting, in the
    actual running game.

If every step above behaved the way it's described, `c/` now has every ticket from
`tasklist.md` implemented and demonstrated end to end — `project-gomgom/` is still exactly
as it was before this work started, untouched, ready for someone to decide whether to
carry these same changes over there.
