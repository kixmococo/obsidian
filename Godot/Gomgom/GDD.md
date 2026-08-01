# Game Design Document
### GomGom — Final Decisions (v1.0)

*This is the clean version: locked decisions only. For the reasoning, pros/cons, and discussion behind these calls, see GDD-Design-Notes.md.*

---

## Progress Tracker

| Section | Status |
|---|---|
| 1. Game Overview | **Locked** |
| 2. Core Pillars | **Locked** |
| 3. Core Gameplay | **Locked** — Base Kit, Combos, Power-up framework, and Core Loop all confirmed |
| 4. Progression & Economy | **Locked** |
| 5. Level & World Design | **Locked** (structural framework + World Bosses; biome names/exact level counts left as open placeholders) |
| 6. Narrative & Story | **Deliberately deferred** — thin placeholder only, by design |
| 7. UI/UX | **Locked** |
| 8. Art Direction | **Locked** |
| 9. Audio Direction | **Locked** |
| 10. Multiplayer | **Locked** (netcode risk flagged for TDD) |
| 11. Monetization | **Locked** |
| 12. Accessibility | **Locked** — no difficulty tiers, single standardized playthrough |
| 13. Technical Summary | **Locked** — engine = Godot, PC-only scope for v1 TDD |
| 14. Open Questions & Risks | **Locked** (ongoing running list — inherently living, not a one-time fill) |
| 15. Appendix | **Locked** |

---

## Version Control

| Version | Date | Author | Summary of Changes |
|---|---|---|---|
| 0.1 | — | u1 | Initial template + core concept |
| 0.2 | 2026-07-20 | u2 | First speculative full draft (superseded) |
| 0.3 | 2026-07-20 | u1 + u2 | Section-by-section pass begins; Sections 1 (partial) and 3 locked via discussion |
| 1.0 | 2026-07-20 | u1 + u2 | Full section-by-section pass complete — Sections 1-15 all locked or intentionally scoped |

---

## 1. Game Overview

- **Working Title:** GOMGOM
- **Genre:** Co-op Physics Platformer
- **Platform(s):** PC (Steam) — v1 scope. Broader porting ambitions (Switch, macOS, iOS, Android, Xbox, PlayStation) exist but are not a current commitment; see Section 11.
- **Target Audience:** Anyone
- **Target Rating (ESRB/PEGI):** E
- **One-Sentence Pitch:** "GOMGOM uses the power of friendship to get out of a... sticky situation."
- **Design Mandate:** Solo and co-op are both first-class experiences. No level or Combo may create a hard progression block for a solo player — every Combo-gated route needs a solo-viable alternative.

### Unique Selling Points
1. **One gummy body, four verbs, endless combos.** Squish, Stretch, Bounce, and Sticky aren't separate abilities — they're one physical system where the same input means different things depending on context (squish onto a wall vs. onto a teammate; stretch to an anchor vs. to an ally).
2. **Every level teaches you something new, once.** No universal power-up loadout — every level's twist is bespoke to that level's puzzle, so nothing goes stale and nothing needs balancing across the whole game.
3. **Co-op Combos are real tech, not convenience.** Gummy Bridge and Roll Dash aren't "easier with friends" — they're capabilities a solo player structurally cannot access, with driver/passenger roles that make multiplayer a coordination puzzle, not just extra hands.
4. **Toy-shelf art direction.** Glossy, translucent, studio-lit vinyl-toy rendering applied to a full game world.
5. **Cosmetic-only progression.** Candy Stars buy cosmetic heads — no power creep, ever.

### Comparable Titles
- **BattleBlock Theater** — co-op physics platforming, cosmetic head economy, silly tone.
- **Gang Beasts / Human Fall Flat** — squishy, soft-body physical comedy as the core feel.
- **Splasher** — a single substance (paint) that changes player capability contextually; still a fair comp for GomGom's per-level gimmick philosophy even without a universal Flavor system.
- **Trine** — asymmetric co-op puzzle-platforming where combining characters' tools solves puzzles solo can't; GomGom aims for that same "we need each other" feel with one shared kit instead of distinct classes.

---

## 2. Core Pillars

- **Malleable Movement** — GomGom is never one shape for long. Every traversal problem should have a squish/stretch/bounce/stick answer before it has any other kind of answer.
- **Better Together** — Co-op isn't a difficulty modifier, it's a toolkit expansion. Gummy Bridge and Roll Dash are capabilities a solo player structurally cannot access — playing with friends changes what's *possible*, not just what's *easier*.
- **Sweet Surprise** — Every level's twist is a treat you unwrap once. Power-ups exist to delight in the moment, not to be collected, optimized, or ground toward — nothing recurs long enough to become a chore.
- **Cozy Mischief** — Tone sits between "adorable" and "a little bit of a menace." Cute doesn't mean toothless — hazards, rival gummies, and physical slapstick are all fair game.
- **Shelf-Life Charm** — Every asset should look like it belongs in toy-photography key art. If a level prop or UI element would look out of place next to a vinyl-figure render, it's wrong for GomGom.

---

## 3. Core Gameplay

### 3.1 Core Loop

**Core Loop Summary:**
1. From the world map, select a level in the current world (World 0 first; later worlds unlock per Section 4).
2. Enter the level solo or with 1-3 co-op partners.
3. Traverse using the Base Kit (Squish, Stretch, Bounce, Sticky) and, in co-op, Combo interactions (Gummy Bridge, Roll Dash) that aren't available solo.
4. Encounter and use that level's one-off power-up(s), which recontextualize the Base Kit for its specific puzzles — not expected to reappear elsewhere.
5. Collect Candy scattered through the level along the way.
6. Reach the level's goal to earn up to 3 Stars (completion / time bonus / optional in-level challenge). The first time each Star is earned, it also grants a one-time Candy bonus.
7. Return to the world map; spend accumulated Candy on cosmetic heads (and other cosmetics) between levels.
8. Complete World 0 in its entirety to unlock World 1; from World 1 onward, accumulate enough Stars to hit each world's unlock threshold (exact numbers TBD) — a stuck level can be skipped and returned to later.
9. Progress through Worlds 1-5 to the Finale Gauntlet (World 6).

### 3.2 Controls & Camera
- **Input Method(s):** Gamepad-first, full keyboard remap support
- **Camera Perspective:** 2.5D side-scroll

### 3.3 Key Mechanics

The full moveset is organized into two tiers: a **Base Kit** available to every player at all times, and a **Combo tier** that only exists when 2+ players interact. There is no universal power-up system — all power-ups are one-off, level/environment-specific (see below).

#### Tier 1 — Base Kit

**Squish**
- Crouch and Squish are the same input/action.
- Shrinks the player to roughly half height.
- While squished, the player becomes a valid **Stretch target** for another player — they can be grabbed and launched.
- If launched into a wall, the squished player sticks into it on impact.
- Getting free from a wall-stick: self-free — any input pops the player out after a short beat.

**Stretch**
- Targets either a tagged environmental grip-point, or another player.
- Target = squished player → grab-and-launch (this covers what was previously scoped as a separate "Slingshot Launch" move; no separate system needed).
- Target = standing (non-squished) player → forms a link between the two players. This link is the basis of Gummy Bridge (see Combo tier).

**Bounce**
- Charging a bigger bounce costs air control — committing to a charged bounce means committing to the jump, no mid-air correction.

**Sticky**
- Cling to walls/ceilings.
- Unlimited stamina — no time limit on clinging. (Difficulty-tier stamina limiting was considered and explicitly rejected — see Section 12.)

#### Tier 2 — Combo Moves (2+ players only)

**Gummy Bridge**
- Formed when a player Stretches and links to another standing (non-squished) player at a tagged grip point.
- The resulting link behaves as a single elastic object, with outcome determined by what interacts with it:
  - No velocity (walking across) → static walkway.
  - Velocity + non-squished body lands on it → trampoline, launches the incoming player.
  - Velocity + squished body lands on it → absorbs/catches the incoming player.
- This single system covers what were previously three separate concepts (Bridge, Net, Trampoline) — no separate Gummy Net mechanic exists; it's a state of Bridge.

**Player Stacking & Squishing Onto Players**
- Players can physically stand/walk on top of one another (e.g., for extra Bounce height).
- Squish is a toggle: press to squish down, press again to un-squish and stand back up.
- A squished player is a valid Stretch-launch target, and can also simply be walked/bumped into by another player, exactly like sticking to a wall.
- If the arriving player is not yet squished, they stick to the squished ally only once they press Squish themselves (this press is what forms the attachment).
- If two already-squished players make contact without a fresh Squish press at that moment (e.g., during a Bridge catch), they do not attach — normal passive contact only.
- A squished-and-attached player separates by pressing Squish again (un-squish/stand).

**Roll Dash**
- Triggered by the *squish key press itself* while in contact with an already-squished player — not merely by both being in a squished state.
- This means: unsquished player touches/launches into a squished ally, then presses Squish → attaches AND activates Roll Dash.
- Two already-squished players simply colliding (no fresh press) does NOT trigger it — this is what prevents Gummy Bridge's catch state from accidentally forming a ball.
- 3rd/4th players join the same way: attach to the ball, then press Squish to join and grow it.
- The player whose press most recently triggered/grew the ball controls its rolling movement; all other players in the ball retain Stretch, letting them reach out and grab things while rolling.
- Un-forming: pressing Squish again (un-squish) separates that player from the ball.

#### Cut / Folded-In / Shelved
- **Slingshot Launch** — folded into Stretch + Squish (grab-and-launch is the same interaction, not a separate move).
- **Gummy Net** — folded into Gummy Bridge's catch state.
- **Shield Bubble** — shelved. Candidate idea (3+ players forming a closed loop with the same link logic as Bridge) noted for revisit after Bridge is prototyped — not designed or committed.

#### Power-ups: Environmental, Not Universal
There is no universal/reusable Flavor Mode system. All power-ups (including former concepts like Vacuum Gulp, Healing Goo, Glue Trap) are **one-off, level/environment-specific** — each modifies or adds to the Base Kit for the context of a single level or puzzle, and is not expected to recur elsewhere in the game. Concept art referencing a branded 7-8 flavor system should be treated as inspiration, not a locked spec.

---

## 4. Progression & Economy

### 4.1 Player Progression
**Progression Type:** Linear, world-by-world. No lives system — each level is pass/fail (die → retry the level, no shared life pool).

Worlds are completed in a fixed sequence. **World 0 must be completed in its entirety** (every level, not just a star threshold) — it exists to demonstrate the full Base Kit and most power-ups that will recur or be built on in later worlds (a small number of power-ups may be held back for later worlds specifically). From World 1 onward, the next world unlocks via a **Star threshold**, not full completion — letting a player skip a level they're stuck on and come back later. (Exact threshold numbers are TBD pending level-count planning — see Open Questions.)

Each level awards up to **3 Stars**: one for completion, one for a time bonus, one for an optional in-level challenge. The first time a Star is earned (not on replay), it converts into a set amount of Candy — so Stars are the skill/completion signal, and Candy is the spendable currency, but the two are connected rather than fully separate systems.

### 4.2 Economy / Resources

| Resource | Earned Via | Spent On | Notes |
|---|---|---|---|
| Candy | Collected in-level, plus a one-time bonus the first time each Star is earned | Cosmetic heads (and other cosmetics) | Single currency — no separate Coin/Gem economy; simplified from the original 3-currency concept for scope |
| Star | Level completion / time bonus / optional in-level challenge (3 max per level) | Nothing directly — gates world unlocks (from World 1 onward) and grants a one-time Candy bonus | Not spendable; a progression/skill marker, not a currency |

---

## 5. Level & World Design

### 5.1 World Structure
**Structure:** Linear levels, grouped into worlds, with single-player, 2-player, and 3-4-player variants/paths available within the same world set (per the solo/co-op parity design mandate from Section 1).

**7 worlds, loosely structured (count may change):**
- **World 0 (Tutorial/Home)** — demonstrates the full Base Kit and Combo tier, and showcases the majority of power-ups at least once (a small number may be deliberately held back for later worlds). Must be completed in its entirety to unlock World 1.
- **Worlds 1-5 (Focus Worlds)** — each is a distinct candy biome (setting/aesthetic TBD) with a loose thematic and mechanical emphasis driven primarily by that world's unique one-off power-ups, not a rigid "one verb per world" formula — keeps worlds feeling bespoke rather than formulaic, consistent with the Sweet Surprise pillar. Unlock via Star threshold (exact numbers TBD).
- **World 6 (Finale Gauntlet)** — combines mechanics and power-up ideas from across the whole game for a final challenge run.

### 5.2 Level List
*Biome names/settings and exact level counts per world are placeholders — to be brainstormed in a future pass.*

| World | Focus | Key Content | Status |
|---|---|---|---|
| World 0 — [Tutorial biome TBD] | Full Base Kit + Combo demonstration | Introduces Squish, Stretch, Bounce, Sticky, Gummy Bridge, Roll Dash; showcases majority of power-ups | Not Started |
| World 1 — [biome TBD] | Loose, powerup-driven | TBD | Not Started |
| World 2 — [biome TBD] | Loose, powerup-driven | TBD | Not Started |
| World 3 — [biome TBD] | Loose, powerup-driven | TBD | Not Started |
| World 4 — [biome TBD] | Loose, powerup-driven | TBD | Not Started |
| World 5 — [biome TBD] | Loose, powerup-driven | TBD | Not Started |
| World 6 — Finale Gauntlet | Combines mechanics/power-ups from all prior worlds | TBD | Not Started |

### 5.3 Enemies
- **Rival Gummies** — small aggressive blob enemies (see the angry pink and spiky purple gummies in the concept sheets), likely a "Sour Gum" antagonist faction (see Section 6, Narrative).
- Primarily Mario-style hazards scattered through levels, meant to be **avoided** rather than hunted. Stomp and stretch-grab-and-drag exist as tools for dealing with them when needed, but there's no defeat-tracking or combat-scoring system — enemies are level obstacles, not a scored objective.

### 5.4 World Bosses
Every world, **including World 0**, ends with a boss fight capping that world's content (World 0's boss is expected to be easier, matching its tutorial role).

- Bosses are damaged through **puzzle-like maneuvers**, not straightforward attacking — players must synergize their Combos and core verbs to find and execute the boss's weak point, closer to a Zelda dungeon boss than a health-bar slog.
- Damage mechanics are a **mix**: some bosses reuse existing Combos (Bridge, Roll Dash) applied to the fight; others use a bespoke, boss-specific mechanic in the same spirit as one-off level power-ups.
- Per the Section 1 design mandate, **the same boss appears in both solo and co-op**, but the specific puzzle/mechanic used to damage it is tailored per player count — a solo player gets a genuine, intended solo method, not a scaled-down or missing version of the co-op fight. **This adaptive-content principle is currently scoped to boss fights specifically** — not generalized across all regular levels — to keep production cost bounded; extending it further is a future decision, not a current commitment.
- **Solo boss mechanic:** reuses the existing environmental-anchor-point substitute pattern where possible (the same approach already established for Combo-gated routes in Section 3 — an anchor point standing in for what a second player's Bridge/Roll Dash would normally provide), rather than inventing a fully bespoke solo-only mechanic per boss by default.
- **Production note:** building genuine solo + co-op parity for every boss doubles design/build effort per boss. World 0's boss should be the first to get full parity treatment, as the proof-of-concept; the remaining 6 bosses carry the same principle but execution/priority is TBD, not a blanket commitment yet.

---

## 6. Narrative & Story

*Deliberately deferred — u1's call, no objection from u2. GomGom's draw is the physical co-op toolkit and toy-shelf aesthetic, not plot, so a thin/arbitrary story frame is fine for now (BattleBlock Theater, Overcooked). Placeholder framing only:*

GomGom's candy world is threatened by an invading faction of "Sour Gums" (the rival gummy enemies from Section 5.3); GomGom and friends push back the sourness across the world's biomes. Each world's boss is the local Sour Gum leader/champion for that biome — giving each world a concrete story capstone even while the connective narrative between worlds stays thin/arbitrary. No further detail locked — revisit if/when it actually matters to production.

---

## 7. UI/UX

### 7.1 Menu Flow
**Main Menu → Gameplay Flow Summary:** Main Menu → World Map (worlds unlock per Section 4) → Level Select within a world → pre-level lobby (co-op players join here) → Level → Results screen (Star breakdown + Candy earned) → back to Level Select.

### 7.2 HUD
Kept deliberately minimal during play:
- **Candy counter** — running total, always visible.
- **Timer** — runs during the level, supports the time-bonus Star.
- No lives/health bar (no lives system, per Section 4).
- No defeat counter at all — enemies aren't a scored/tracked system, just hazards to avoid (see Section 5.3).
- No live Star indicators during play — revealed on the **Results screen** instead (completion / time bonus / optional challenge), keeping the in-level HUD uncluttered and avoiding spoiling a level's optional challenge before the player finds it naturally.

### 7.3 Onboarding / FTUE
Each move (Squish, Stretch, Bounce, Sticky, Gummy Bridge, Roll Dash) gets a **light UI prompt** — a brief on-screen button icon the first time that move becomes usable in World 0, then never shown again. No forced tutorial popups or text boxes. Enemy weaknesses (e.g., stomp vs. stretch-grab-and-drag) are not explicitly explained — players are expected to discover them independently, Mario-style, through experimentation.

---

## 8. Art Direction

- **Visual Style Reference:** Glossy, translucent, studio-lit vinyl-toy photography — GomGom should read as an object you could physically pick up. World backgrounds extend this into full "toy diorama" sets (candy-cane fences, gingerbread-textured platforms, gumdrop terrain), per the concept art (treated as inspiration, not a literal spec — see Section 3's power-up note).
- **Color Palette Philosophy:** Each world's biome shifts to its own distinct palette — helps world identity and wayfinding, and gives the "Sweet Surprise" pillar a visual dimension (new world, new look, not just new mechanics). GomGom's own core pink family stays consistent as the character's identity anchor regardless of biome.
- **Player Differentiation:** All players share the same uniform pink GomGom body — cosmetic heads (Section 4's Candy economy) are the sole differentiation method, no per-player color/pattern variants. This is a deliberate choice, not just a scope-saver: identical bodies support the Cozy Mischief pillar by making "who's who" mix-ups (especially inside a fused Roll Dash ball) a source of comedy rather than a coordination problem — the mechanics themselves (e.g., Roll Dash's driver logic) don't actually depend on visually telling players apart, so the ambiguity is low-stakes and Gang Beasts-style charming rather than frustrating.

*Guidance: link the full Art Bible here once it exists as a separate doc — this section should stay a summary.*

---

## 9. Audio Direction

- **Music Style/References:** Bouncy, candy-pop instrumentation — playful major-key melodies, toy-percussion textures (music-box/bell layers over upbeat pop structure). Tempo and instrumentation shift subtly per world's biome/palette (see Section 8), reinforcing world identity musically as well as visually.
- **SFX Philosophy:** Stylized and squishy — every core verb (Squish, Stretch, Bounce, Sticky) needs a distinct, satisfying, cartoon-physical sound. This is a "feel" game; SFX carries as much game-feel weight as the animation itself, arguably more for the Base Kit specifically.
- **VO Scope:** Light, non-verbal "babble" VO (Animal Crossing / Banjo-Kazooie style vocal mumbles), tied to GomGom's expression system (Happy/Curious/Excited/Cheeky/Focused/Surprised). Matches the toy-mascot identity without the cost or localization burden of full voice acting.

---

## 10. Multiplayer / Online Features

- **Mode(s):** Single-player and co-op — co-op is not competitive; friendly-fire-free by design, consistent with the E rating and Cozy Mischief tone.
- **Player Count:** 1-4.
- **Matchmaking Approach:** Client-hosted. Local (couch) and online co-op are both in scope for v1, treated identically in design — no local-only or online-only feature gap.
- **Netcode Requirements:** Flagged as a significant technical risk for the TDD — Combo mechanics (Gummy Bridge's elastic link, Roll Dash's fusion, Squish-launch trajectories) are tightly-coupled, physics-driven interactions between players. Getting these to feel good over real network latency (not just locally) is a harder problem than typical platformer netcode and should be prototyped early, not assumed to "just work" once the local version feels right.

---

## 11. Monetization

- **Model:** Premium (one-time purchase) for the base game. Open to a premium (paid) expansion later — not free content drops, consistent with "no F2P systems ever" — but this is a future possibility, not a v1 commitment.
- **Monetized Systems:** Steam storefront only for v1 (Windows + Linux) — no in-game purchases. Open to porting to additional platforms later, including Nintendo Switch, macOS, iOS, Android, Xbox, and PlayStation — v1's core focus remains PC/Steam; porting is a future ambition, not a current scope commitment (see Section 1 note).
- **Fairness/Ethics Guardrails:** Online interactions not rated for a separate age tier. All Candy/Star economies are cosmetic-only by design — no mechanical advantage is ever purchasable or grindable-around, consistent across both the base game and any future premium expansion.

---

## 12. Accessibility

- **Difficulty options:** None. A formal difficulty-tier system was considered (Sticky's stamina as a Normal/Hard toggle, potentially expanded to other mechanics) and explicitly rejected — GomGom ships as **one standardized playthrough**, no difficulty modifiers.
- **Colorblind modes:** Not planned — with universal Flavor Modes cut in favor of one-off, level-specific power-ups, the original concern (color-coded flavor identity) no longer applies.
- **Remappable controls:** Full keyboard and gamepad remapping (already established in Section 3.2).
- **Subtitles/captioning:** Captioning applies to sound cues rather than dialogue, since narrative is deliberately thin and VO is non-verbal babble (Section 9) — no spoken dialogue exists to subtitle in the traditional sense.
- **Other:** None planned.

---

## 13. Technical Summary

*High-level only — full detail belongs in the TDD (next document).*

- **Target Engine:** Godot
- **Target Platforms & Min Specs:** PC (Windows + Linux) for v1 — specs TBD in the TDD. Broader porting ambitions exist (see Section 11) but are out of scope for the v1 technical plan.
- **Link to TDD:** [to be added once TDD.md exists]

---

## 14. Open Questions & Risks (running list)

| Question/Risk | Status |
|---|---|
| Solo-equivalent routes for every Combo-gated puzzle | Design mandate locked; specific solutions TBD per-level |
| Formal difficulty-tier system (Sticky stamina, and possibly others) | **Resolved** — rejected; single standardized playthrough, Sticky is unlimited stamina |
| Shield Bubble as closed-loop Bridge | Shelved — revisit after Bridge prototype |
| World/level organizing principle (flavor-based structure is now invalid) | **Resolved** — biome + loose powerup-driven theme + narrative beat, see Section 5 |
| Player-stacking for Bounce height bonus | **Resolved** — players can stand on each other; folded into Roll Dash/Squish-onto-player system |
| Star threshold numbers for World 1+ unlocks | Open — depends on total level-count planning, not yet known |
| Candy biome names/settings for Worlds 0-6, and exact level count per world | Open — deliberately left as a placeholder pass, per u1 |
| Netcode for physics-driven Combos (Bridge, Roll Dash, Squish-launch) over real network latency | Flagged risk — needs early TDD prototyping, not assumed to work by default |

---

## 15. Appendix

- **References:** GomGom character/mechanic concept sheets (7 pieces — official character turnaround, two mechanics-sheet variants, sidescroller gameplay mockup). Treated as inspiration/idea-building material throughout this document, not a literal spec — several mechanics diverged deliberately from what's shown in the art.
- **Related Documents:** Technical Design Document (next), Enterprise Game Development Playbook (production process reference), Art Bible (not yet started — see Section 8).
