# Game Design Document — DESIGN NOTES
### GomGom — Annotated Draft (v0.3)

*This version keeps every discussion, pros/con, and rejected idea alongside the decisions, so we can see WHY later. For the clean reference doc, see GDD.md.*

---

## 1. Game Overview

- **Working Title:** GOMGOM
- **Genre:** Co-op Physics Platformer
- **Platform(s):** PC (Steam)
- **Target Audience:** Anyone
- **Target Rating (ESRB/PEGI):** E
- **One-Sentence Pitch:** "GOMGOM uses the power of friendship to get out of a... sticky situation."

> **Discussion — Solo vs. co-op priority:** u1 chose "truly equal — both must be first-class" over co-op-primary or solo-primary. u2's honest flag: this is the harder path. It means every Combo-gated route (things requiring 2+ players) needs a genuine solo-viable alternative — not "solo players skip this," but an actual different solution. More design/QA surface than co-op-primary, but it's the right call if GomGom needs to be a complete game for a solo player, not a "come back with friends" tease.
>
> **Discussion — genre label:** "Multiplayer Platformer" (original) vs. "Co-op Physics Platformer" (u2 suggestion, emphasizes the squish/stretch feel as the selling point, not just headcount). u1 chose Co-op Physics Platformer.
>
> **Discussion — target audience:** u2 offered to sharpen "Anyone" into something more specific (e.g., "kids + families" or "all-ages but tuned like BattleBlock"). u1 chose to keep it broad. Worth remembering this might get revisited once tone/difficulty decisions (see Sticky stamina discussion below) make the audience more concrete in practice even if the doc stays broad on paper.

### Unique Selling Points (u2 draft, NOT yet confirmed by u1 — conversation moved into a mechanics detour before this was locked)
1. One gummy body, four verbs, infinite combos — no ability-unlock grind.
2. [NEEDS REWRITE] Originally "Flavor Modes reskin your whole kit" — invalid now that universal Flavor Modes were cut in favor of one-off environmental power-ups. Needs new wording once Section 3 language is finalized.
3. Co-op Combos are real tech, not convenience.
4. Toy-shelf art direction.
5. Cosmetic-only progression.

### Comparable Titles (u2 draft, NOT yet confirmed)
BattleBlock Theater, Gang Beasts/Human Fall Flat, Splasher, Trine — u2's reasoning for each is in the prior draft. Not yet reviewed by u1.

---

## 2. Core Pillars — full discussion trail

u2 checked whether the four unaffected pillars (Malleable Movement, Better Together, Cozy Mischief, Shelf-Life Charm) still held up given everything decided since they were first drafted. u1 confirmed all four still fit as-is — no changes needed.

**Pillar 3 rework:** the original "Tasty Transformation" was written around universal Flavor Modes (collect and master a recurring toolkit). u2's framing: the new philosophy isn't a reworded version of the old one, it's a genuinely different design ethos — surprise/novelty per-level instead of a masterable system. u2 offered three naming directions ("One Good Trick," "Sweet Surprise," "No Two Bites Alike"). u1 chose **Sweet Surprise**.

**Section 2 status: fully locked.**

---

## 3. Core Gameplay — full discussion trail

### Why we detoured here first
u1 wanted to lock core abilities/movement before continuing section-by-section, specifically wanting pros/cons/drawbacks discussed for each move and how each relates to solo vs. multiplayer play.

### Squish
**u2's initial pros/cons:**
- Pro: clear silhouette change, sets up physical comedy, pairs naturally with dissolve/burst-style effects.
- Con (the main worry): "duck to slide under things" is one of the most generic verbs in platforming — risk of feeling like a reskinned crouch with no gummy-specific identity. u2 proposed alternatives: fall-damage absorption, or brief wall/ceiling stick on press.

**u1's actual direction (better than either option offered):** crouch and Squish are literally the same input. Squish shrinks the player to half height AND turns them into a valid **Stretch target** for another player — they can be grabbed and launched, and if launched into a wall, they stick into it. Getting free: self-free, any input pops them out after a short beat (u1 explicitly ruled out timer-based or teammate-rescue options).

**u2's reaction:** this elegantly solves the "give Squish identity" problem by making it the *object* half of a two-verb interaction (Stretch = verb, Squish = target/noun) rather than inventing a new isolated mechanic. It also auto-solves solo/co-op parity for this interaction specifically — solo players still Stretch to anchor-points, they just can't target an ally, so nothing is lost, only added in co-op.

### Stretch
**u2's initial pros/cons:**
- Pro: most versatile verb — movement AND interaction (levers, rescue, pulling objects).
- Con (main worry): if Stretch could anchor to ANY surface, it becomes a Spider-Man grapple hook that trivializes platforming gaps — designers would be fighting the player's ability to skip challenges. u2's recommendation was anchor-point-only (tagged grip surfaces), not free-aim.
- Secondary con raised: precise aim-and-release on a gamepad, in local co-op chaos, could be fiddly — especially for a broad "Anyone" audience that may include younger players.

**u1's decision:** anchor-points AND grab other players (not free-aim-anywhere). This directly matches u2's anchor-point recommendation while adding the player-targeting behavior that became the basis for the whole Squish/Bridge system above.

### Bounce
**u2's initial pros/cons:**
- Pro: satisfying charge-jump, natural stomp-attack tie-in to combat/enemy-defeat HUD counter seen in concept art.
- Con: charge-jump feel lives or dies on tuning — if charging costs air control, it can feel stiff; if it doesn't, chaining bounces might trivialize vertical challenges. u2 flagged this needs physical prototyping more than paper design.
- Co-op idea u2 raised (not in original concept art): bouncing off a teammate's head for extra height, "gummy tower" style. Not yet confirmed/rejected by u1 — still an open idea, not committed to the framework.

**u1's decision:** charging costs air control — full commitment to a charged bounce, no mid-air correction. u1 explicitly wanted the "commit to the jump" version over the free-control or ground-only-charge options.

### Sticky
**u2's initial pros/cons:**
- Pro: opens vertical space in an otherwise flat 2.5D game; ceiling traversal is a fun visual gag.
- Con: stamina-limited wall-cling is an extremely well-worn trope (every 3D Zelda since Breath of the Wild) — risk of feeling like a meter to manage rather than a move to enjoy. Also flagged a genuine readability concern: ceiling traversal in a side-view camera can be disorienting and needs early testing, not just paper assumption.

**u1's decision — and this became a bigger idea than just Sticky:** rather than picking stamina-limited OR stamina-free universally, u1 proposed this could be a **difficulty control** — Hard mode has the stamina meter, Normal mode has unlimited stamina.

**u2's reaction:** flagged this as a genuinely good idea because it resolves a tension u2 had already raised back in the original Section 12 draft — color-coded power-ups + no accessibility options was a real risk for an all-ages title. Difficulty-tiering built INTO core verbs (rather than a bolted-on separate "assist mode" system) is cheaper to build and more elegant. u2 asked whether this should become a FORMAL difficulty system across other mechanics (hazard damage, Bounce charge risk, etc.) — **u1 said not sure yet, revisit at Section 12 once the full kit is known. This is bookmarked, not decided.**

### Combo tier discussion

**Slingshot Launch:** u2 noticed this is now redundant — grabbing a squished ally and flinging them IS Slingshot Launch, per the Squish/Stretch decisions above. Recommended cutting it as a separate concept rather than designing it twice. Not explicitly confirmed/denied by u1 in isolation, but implicitly accepted as the conversation moved forward treating it as folded in.

**Gummy Bridge / Gummy Net merge:** u2 proposed that Stretch-linking to a standing (non-squished) ally forms a Bridge, and suggested Gummy Net could just be "what happens if, while linked, one player squishes mid-link" (the link goes net-shaped). 

**u1's actual answer (more elegant than u2's proposal):** players Stretch to a tagged grip point (including targeting squished players); the resulting stretched-arm link IS the bridge. The behavior is unified by physics state rather than a special "squish mid-link" trigger:
- No velocity crossing it → walk across normally.
- Landing with velocity, non-squished → bounced (trampoline).
- Landing with velocity, squished → caught (net/absorb).

u1 explicitly asked u2 for more thoughts here. **u2's reaction:** reflected this back as "one elastic object with a single physics rule, not three mechanics" — cheaper to build (one state check) and more coherent than u2's own original proposal. u2 confirmed u1's version was better than what u2 had suggested.

**Shield Bubble:** originally Sticky+Bounce combo from concept art. u2 flagged early this didn't derive cleanly from the verb logic the way other combos did, and it doesn't fit into the physical model in the same way. u1 initially said cut it. Later, once Bridge's elastic-link logic was defined, u2 proposed reviving it as "3+ players forming a closed loop using the same link logic as Bridge" (open loop = Bridge, closed loop = dome/shield). **u1's decision: interesting, but shelve it — revisit after Bridge is actually prototyped.** Not designed further; noted as a candidate idea only.

**Roll Dash:** u1 confirmed keeping this — two or more squished players fuse into a rollable ball — but explicitly wanted a **separate, dedicated input** to trigger it, not an automatic trigger from squished players touching/overlapping. This was a deliberate choice to avoid unwanted/accidental fusion during normal squished-player proximity (e.g., during a Bridge catch).

### The Vacuum Gulp / Healing Goo / Glue Trap question — and the bigger Flavor Modes decision

These three moves from the original concept sheet never fit cleanly into Base Kit or Combo tier. u2 asked whether to resolve them now or park them; u1 chose to resolve now.

**u1's framing:** call all of these "power-ups or upgrades that will be level and environment specific" — they modify or add an ability, contextually.

**u2's first read:** this maps onto the existing Tier 2 (Flavor Modes) concept from the original speculative draft — proposed splitting into two categories: universal/reusable **Flavor Modes** (the branded 7-8 flavor system from concept art) alongside one-off **Environmental Power-ups** (Vacuum Gulp etc.) as a separate, second category.

**u1 pushed back and asked u2's honest opinion** on fully merging everything into one-off, level-specific gimmicks instead of keeping any universal/reusable flavor system.

**u2's honest take (given in full):**
- Argued FOR keeping something universal: the concept art explicitly brands "7 FLAVOR MODES — POWERED BY TASTY TWISTS" as a headline, trading-card-styled feature. u2's read was that this looked like a deliberate collectible/toy-line hook (Kirby copy abilities, Pokémon types), and cutting it entirely risks losing marketable identity and a mastery arc (tools that recur and combine with new later-game hazards).
- Argued full one-off is NOT inherently "too gimmicky" as a pattern — cited Mario Odyssey's capture system as proof one-off, per-level powers work great as a design pattern on their own merits.
- u2's actual recommendation was a middle path: keep ONE Flavor system, but let individual flavors be either common/reusable (the marketed set) or rare/secret/one-off (narrative or level-specific surprises) — same collectible framing, but folds in the one-off gimmick need without a second parallel system.

**u1's final decision: rejected the middle path. Fully merge — ditch the universal/reusable flavor concept entirely.** All power-ups are one-off, level/environment-specific, full stop. u1 also clarified the concept art (including the flavor card sheets) should be read as inspiration/idea-building material, not a literal spec to preserve.

**u2's acknowledgment:** this was u1's call to make after hearing the trade-offs, and u2 flagged the two direct ripple effects it creates:
1. Pillar 3 ("Tasty Transformation") needs a reword — it was written around universal flavors.
2. The speculative Section 5 world structure (worlds organized by flavor) is now invalid and needs a new organizing principle when we get there.

Both are logged as open items rather than fixed preemptively, per u1's wish to go section-by-section in order.

---

### Section 1 closed out — USPs and Comparable Titles

After Section 3 was locked, u2 returned to the unresolved USPs/comps and rewrote them from scratch rather than patching — the old #2 ("Flavor Modes reskin your kit") was the headline USP in the original speculative draft and had no patch-fix once universal flavors were cut. New #2 ("every level teaches you something new, once") replaces it, built around the one-off-per-level-gimmick philosophy instead.

u1 approved the full revised list without changes.

**Comparable titles — Splasher re-check:** originally comped because "one substance changes your ability contextually" mirrored Flavor Modes. u2 flagged this needed re-justifying now that Flavor Modes are gone, and asked whether it should be dropped/replaced. u1's call: still fits — Splasher's single-substance-changes-capability idea maps well enough onto the new per-level gimmick philosophy (each level's power-up recontextualizes the same base tools) even without a persistent flavor system. Kept as-is.

**Section 1 status: fully locked.**

---

## 4. Progression & Economy — full discussion trail

u2 recommended proceeding to Section 4 before Section 5, since Section 4 had no unresolved dependencies while Section 3.1 (Core Loop) was intentionally left unfinished pending Section 5's world structure.

**Currency simplification:** the original speculative draft proposed 3 currencies (Candy Star, Coin, Gem) mirroring the sidescroller concept art HUD, with Coin/Gem purposes left undefined. u2 flagged this as scope risk rather than just re-proposing a fix, and asked whether u1 actually wanted the full 3-currency system. u1 chose to simplify to a single currency, explicitly noting the HUD art (coin/gem counters) was "concept flavor," not a locked spec.

**Star-earning structure:** u1 wanted all three options u2 offered (pure completion / hidden objective / time-skill bonus) combined into one 3-Star-per-level system — u1 asked for clarification on what "HUD art" referred to, which u2 clarified was the sidescroller gameplay mockup's on-screen timer, the origin of the time-bonus idea.

**Currency vs. progression marker split:** u1 proposed that in-level collectible "coins/candies" directly buy cosmetics, while a separate marker tracks per-level progression/skill (the 3-star system), with the first-time earning of that marker granting a one-time bonus of the spendable currency. u2 flagged this as a good decoupling — the collectible (found in the level) and the rating (earned by playing well) do different jobs instead of being conflated into one system like the original "Candy Star" concept.

**Naming the progression marker:** u1 asked for a candy-themed alternative to "Star," specifically because "Candy" was already the currency name and wanted to avoid overlap. u2 proposed Hearts (favored, ties to GomGom's signature heart-antenna brand identity), Sprinkles, and Jellybeans. u1 initially considered Hearts but reversed after realizing it could be misread as a lives/health system — **u1 explicitly clarified the game has no lives system; each level is pass/fail (die = retry that level, no shared life pool)**, which is a notable standalone design decision beyond just naming. Final call: keep the name **Stars** (reverting to the original template's language) rather than risk that confusion.

**World-gating:** World 1 requires 100% completion (all levels) specifically to demonstrate the full Base Kit and most power-ups that recur/build on in later worlds — u1 noted a small number of power-ups may be deliberately held back for later worlds. From World 2 onward, unlocking is via a Star threshold rather than full completion, explicitly to let a stuck player skip and return later. Exact threshold numbers are open, pending level-count planning.

**Section 4 status: fully locked**, aside from the exact numeric Star thresholds (logged as an open item, dependent on future level-count planning).

---

## 5. Level & World Design — full discussion trail

**Foundational question:** with flavor-based world organization dead, what ties a world together? u2 offered three axes (candy biome/setting, mechanical focus, narrative arc) as separate options plus a combination option. u1 chose combination — "somewhat of all of these."

**First mapping attempt (revised):** u2's initial proposal was a rigid 1-mechanic-per-world grid: World 0 = tutorial (teaches everything, already locked from Section 4), then five dedicated focus worlds each spotlighting one of the six remaining systems (Stretch, Squish, Bounce, Sticky, Bridge, Roll Dash), with one pair combined to fit five worlds. u1 gave an example of a pairing they liked (Squish + Bridge, since Squish is thematically the "object" both Bridge's catch-state and the launch interaction revolve around). u2 built a full mapping around that: Squish+Bridge / Stretch / Sticky / Bounce / Roll Dash across 5 worlds, plus a Finale Gauntlet (World 6) = 7 worlds total. u1 confirmed the 7-world count but explicitly rejected the rigid mapping.

**u1's actual framing (better than the rigid grid):** the mechanical identity of each world should come primarily from that world's **unique one-off power-ups**, not from a fixed Base-Kit-verb assignment. World 0 demos the majority of power-ups (not necessarily all — some deliberately held back for later, consistent with the earlier Section 4 decision) plus the full Base Kit/Combo tier. Worlds 1-5 have a "larger focus or theme involving specific core mechanics and powerups" but this is explicitly loose and arbitrary rather than locked to a grid.

**u2's reaction:** flagged that this is actually more consistent with the Sweet Surprise pillar (Section 2) than the rigid grid was — a strict "world = mechanic" formula would work against the "every level's twist is bespoke, nothing recurs as a system" philosophy. Agreed the looser framing was the better call.

**Biome naming:** u2 offered to brainstorm actual biome names/settings now vs. later. u1 chose to defer this (left as an open placeholder) and keep moving through the rest of Section 5.

**Section 5 status: structurally locked** (world count, tiered structure, unlock logic, loose theming philosophy). Biome names, exact per-world level counts, and Star threshold numbers remain open placeholders for a future pass.

---

## 3.1 Core Loop — closed out

Written directly from everything already locked in Sections 1, 3, 4, and 5 — no new decisions needed, just assembling the pieces (world map → level → Base Kit/Combos → level-specific power-up → Candy collection → Stars → world unlock progression). No discussion trail beyond this; it was mechanical assembly, not a new design question.

## 5.4 World Bosses — full discussion trail

u1 introduced this after Section 7 was already locked, flagging it as narrative-adjacent: every world ends with a boss, damaged through combo/core-skill synergy puzzles rather than straightforward combat.

**World 0 boss:** u2 asked whether the tutorial world also gets a boss or if bosses start at World 1. u1 confirmed World 0 also ends with a boss, expected to be easier given its tutorial role.

**Damage mechanic source:** u2 asked whether boss mechanics reuse existing Combos (Bridge/Roll Dash) or are bespoke per boss like one-off power-ups. u1's answer: a mix, varies per boss — not a single rule.

**Solo/co-op parity — the important nuance:** u2 asked directly whether solo players need a genuine way to damage combo-gated bosses, given the Section 1 mandate. u1's answer was more precise than a yes/no: **the same boss appears in both modes, but the specific puzzle/mechanic to damage it is tailored per player count** — not "solo gets an alternate route" (the original Section 1 phrasing) but "content itself adapts to player count." u1 clarified this isn't boss-specific — many other levels work the same way.

**Consequence:** u2 flagged that this meant the Section 1 design mandate as originally written was too narrow (it only mentioned Combo-gated *routes* needing alternatives) and rewrote it to describe the broader adaptive-content principle, rather than bolting a boss-specific exception onto the old wording. u1 didn't explicitly review this rewording — logged here in case it needs a second look.

**Section 5.4 status: locked.** Section 1's design mandate wording was updated to match — worth u1 double-checking that edit specifically, since it wasn't separately confirmed.

### Correction — scope walked back, u2's reservations addressed

u2 was asked directly for questions/reservations before moving on. Honest answer given: tailoring genuine solo AND co-op versions of every boss doubles design/build effort per boss — a real production cost worth being deliberate about rather than committing to uniformly across all 7 bosses at once, consistent with the playbook's vertical-slice-protection advice. u2 recommended World 0's boss get full parity treatment first as the proof-of-concept, with the rest TBD rather than a blanket commitment.

u1 agreed to scope the adaptive-content principle back down to boss fights specifically — **not** the generalized "content adapts to player count throughout" wording u2 had written into Section 1 without direct confirmation. Section 1's mandate was reverted to its original, narrower Combo-route phrasing; the boss-specific adaptive principle now lives only in Section 5.4, explicitly scoped and flagged as not-yet-generalized to regular levels.

**Solo boss mechanic pattern:** u2 asked whether a solo boss version should reuse the existing environmental-anchor-point substitute pattern (established in Section 3 for Combo-gated routes) or be fully bespoke per boss. u1 chose to reuse the anchor-point pattern where possible, rather than defaulting to bespoke solo mechanics.

**Section 5.4 status: re-confirmed locked**, now correctly scoped (boss-only, not a general level principle) and with an explicit production-priority note (World 0's boss first, rest TBD).

---

## 6. Narrative & Story — deliberately deferred

u1 chose to skip real narrative work for now, explicitly asking u2 to flag if there were reservations. u2's honest answer: none — GomGom's draw is the physical co-op toolkit and toy-shelf aesthetic, not plot, and comparable titles (BattleBlock Theater, Overcooked) prove a thin/arbitrary story frame is sufficient for this kind of game. A minimal placeholder (Sour Gums as antagonist faction, tied to the Section 5.3 enemy note) was left in the clean doc so Section 6 isn't a total blank, but nothing here is locked or meant to be treated as final. Post-World-Bosses update: each world's boss is now framed as that biome's local Sour Gum leader/champion, giving each world a concrete story capstone even while the connective narrative stays thin.

---

## 7. UI/UX — full discussion trail

Two foundational questions before writing anything: how World 0 teaches its moves, and whether combat is core or optional.

**FTUE delivery:** u2 offered environmental/show-don't-tell (Mario-style, no popups), light UI prompts (icon shown once, first use only), or traditional tutorial text. u1 chose light UI prompts.

**Combat scope:** u2 asked whether combat is core-required, optional/avoidable, or purely cosmetic. u1's answer: genuinely varies per level — could be any of the three depending on the level — with Mario-style enemy-specific weaknesses (jump on head, stretch-grab-and-drag off an edge) that players discover independently rather than being told.

**HUD design consequence (u2's addition, not directly asked for):** u2 flagged that if combat's role (required/optional/comedic) varies per level and some levels' hidden Star objectives are combat-related, a persistent live defeat-counter during play would spoil which levels care about combat before the player discovers it naturally. Proposed keeping the in-level HUD minimal (Candy counter + Timer only) and moving Star breakdown + defeat count to the Results screen instead. u1 didn't explicitly confirm/reject this specific HUD proposal in a follow-up round — it was written directly into the doc as a reasoned consequence of the combat answer, following the same "ask u2 for the honest take, then move" pattern established earlier in the conversation. Worth double-checking with u1 if this feels off in review.

**Section 7 status: locked** (with the caveat above that the HUD minimalism call was u2's extrapolation, not a directly confirmed u1 decision).

### Correction — enemies reframed as avoidance hazards, not a scored system

u1 followed up to reframe the combat answer: Rival Gummies are, similar to Mario, primarily **hazards scattered through levels meant to be avoided**, not a hunted/scored combat system. Stomp and stretch-grab-drag remain available as tools when needed, but there's no defeat-tracking. u1 also clarified Stars are purely completion / time / optional in-level challenge — removing the "hidden objective" wording that had an implied combat connection.

This walked back u2's earlier HUD reasoning (the flagged, unconfirmed extrapolation from before) — the "don't spoil combat-related hidden objectives" rationale for hiding the defeat counter no longer applies, because there's no defeat counter at all now; enemies simply aren't a tracked/scored system. Section 5.3, Section 4's Star description, the Core Loop, and Section 7's HUD were all corrected to match. u2's underlying HUD-minimalism instinct (keep the live HUD to Candy + Timer, reveal Star breakdown on Results) still holds, just for a cleaner reason: uncluttered screen, not spoiler-avoidance.

---

## 9. Audio Direction — full discussion trail

Quick pass — u2 re-offered the babble-VO idea from the original speculative draft (never formally re-confirmed since). u1 confirmed both babble VO and the music/SFX direction as-is, no changes. **Section 9 status: locked.**

## 10. Multiplayer — full discussion trail

u2 surfaced an existing open item: whether online (non-couch) co-op is in scope for v1. u1's answer went further than the options offered — not just "yes, in scope," but explicitly **no distinction between local and online**, both treated identically in design from day one.

u2 flagged this as a real technical risk rather than just writing it in cleanly: the Combo tier (Bridge's elastic link, Roll Dash's fusion, Squish-launch trajectories) are tightly-coupled physics interactions between players, and getting these to feel good over real network latency is harder than typical platformer netcode. Recommended early TDD prototyping rather than assuming it "just works" once local feels right. Logged as a flagged risk in the Open Questions table, not a blocker to locking the section.

**Section 10 status: locked**, with the netcode risk carried forward into Open Questions for TDD attention.

---

## 11. Monetization — full discussion trail

u1 confirmed the existing premium/Steam/cosmetic-only model, and added two forward-looking notes: openness to a future premium (paid) expansion, and openness to porting beyond PC/Steam to Switch, macOS, iOS, Android, Xbox, and PlayStation. Both explicitly framed as future possibilities, not v1 commitments. u2 added a light cross-reference back in Section 1's Platform field so v1 scope stays unambiguous while capturing the broader ambition.

**Section 11 status: locked.**

---

## 12. Accessibility — full discussion trail

Resolved the two long-bookmarked items from way back in the Bounce/Sticky mechanics discussion.

**Difficulty tiers:** u2 asked whether to formalize Sticky's Normal/Hard stamina toggle into a broader difficulty system now that the full kit was visible. u1 reversed course entirely — recanted the difficulty idea, choosing **one standardized playthrough with no difficulty modifiers at all**. This reopened Sticky's stamina question (previously deferred to "decide via difficulty tier"), which u1 resolved as **unlimited stamina**, the original "Normal mode" version. Section 3.3's Sticky entry and the Open Questions table were both corrected to remove the now-dead difficulty-tier reference.

**Colorblind accessibility:** u2 asked whether this was still worth addressing given each world now has its own palette (Section 8). u1's call: no — the original concern was specifically about color-coded universal Flavor Mode identity, which no longer exists since power-ups are one-off/environmental. Not planned.

Remappable controls and sound-cue captioning (adapted from the original "subtitles" since narrative/VO is thin and non-verbal) carried forward from the original template unchanged.

**Section 12 status: locked.**

---

## 13. Technical Summary — full discussion trail

u2 asked whether Section 11's porting ambitions (Switch, macOS, iOS, Android, Xbox, PlayStation) should be reflected in the technical target platforms too. u1 chose to keep this section scoped to PC only — the porting ambition stays a Section 11 monetization/business note, not a v1 technical commitment. **Section 13 status: locked.**

## 14. Open Questions & Risks — no separate discussion

This section is the running table that's been maintained live throughout the whole conversation — it was never a one-time fill-in, so there's no separate discussion trail. Treated as locked/complete in the sense that it's structurally in place and current as of this pass; it will keep accumulating new items as work continues (TDD, level design, etc.).

## 15. Appendix — no separate discussion

Straightforward wrap-up — concept art references and related-document links. No design decisions involved, just documentation housekeeping.

---

## GDD Section-by-Section Pass: Complete

As of this point, Sections 1 through 15 of the clean GDD.md are all locked or intentionally scoped (Section 6's narrative remains a deliberate thin placeholder by design, not an oversight). Remaining open items are tracked in GDD.md's Section 14 table and will carry forward into the TDD and future level-design passes as appropriate.

Not yet discussed. All prior speculative content for these sections (art/audio direction, accessibility, etc.) from the v0.2 draft should be treated as **superseded** — much of it was built on the now-cut universal Flavor Mode system and will need to be reworked, not just copy-pasted forward, when we reach each section.

### Roll Dash trigger — reversed and refined (post-Bounce-bookmark discussion)

u2 flagged an unresolved bookmark: the "bounce off a teammate's head" co-op idea from the Bounce discussion was never explicitly confirmed or rejected. u1 confirmed it and expanded it: general player-stacking (standing on each other) should exist, AND squishing onto another player should work like squishing into a wall, AND this should double as the Roll Dash trigger — with the explicit goal of requiring no extra dedicated input (reversing the earlier "separate explicit input" decision from the initial Combo-tier lock).

**u2's reaction:** flagged this as a deliberate reversal of an earlier decision, noted the trade-off (losing the accidental-trigger safety net a dedicated input provided), but agreed it's consistent with how clean Bridge turned out (one input, meaning determined by context/state).

**Refining how it avoids accidental triggers (the important part):** u2 asked whether squishing-onto-players requires the same Stretch-launch as walls, and specifically whether Bridge's catch state (which already involves a squished player landing on/near another) could accidentally trigger Roll Dash.

u1's answer, worked through over two exchanges: both walking-into and launching-into work for attaching to another player, but attachment (and Roll Dash activation) specifically requires an **unsquished player to press Squish while in contact with an already-squished ally** — that key-press is the activating event. If two players are *already* squished when they make contact (e.g., during a Bridge catch, where the falling player is already squished before landing), no fresh press occurs at the moment of contact, so they simply stick together passively — no ball forms. This means the safeguard u2 was probing for falls out naturally from the input-edge-trigger logic, without needing a special-case exception for Bridge specifically.

**Un-forming:** Squish is a toggle — press to squish, press again to un-squish and stand, which also separates a player from a stick/ball.

**3+ player scaling:** u1 confirmed the same attach-then-squish rule applies for a 3rd/4th player joining an existing ball — no separate rule needed.

**Driver/passenger dynamic (u1's addition, not previously discussed):** the player whose press most recently triggered or grew the ball controls its rolling movement. All other players inside the ball keep access to Stretch, letting them reach out and grab things while the ball rolls. u2's reaction: this turns Roll Dash from "a faster way to move" into a genuine coordination mechanic — one player steers, others interact with the environment mid-roll — which is a nice bit of depth that wasn't in the original concept art at all.

**Status: locked**, not just bookmarked-and-resolved — full trigger logic, attach/detach behavior, and multi-player scaling are all specified above in the clean GDD.md version.

---

## Bookmarked / Open Items Log

1. **Difficulty-tier system** — Sticky's stamina-on-Hard/unlimited-on-Normal idea; u1 wants to decide whether this becomes a formal cross-mechanic system once the full kit is visible. Revisit at Section 12.
2. **Shield Bubble as closed-loop Bridge (3+ players)** — shelved candidate idea, not designed. Revisit after Bridge is prototyped.
3. ~~Bounce co-op idea (bounce off a teammate's head)~~ — **Resolved.** Folded into general player-stacking + the Roll Dash/squish-onto-player system above.
4. ~~USPs and Comparable Titles~~ — **Resolved.** Rewritten post-Flavor-Mode-cut; Splasher comp re-justified and kept.
5. ~~Pillar 3 rewrite~~ — **Resolved.** Renamed "Sweet Surprise" in Section 2.
6. ~~Section 5 world-organizing principle~~ — **Resolved.** Biome + loose powerup-driven theme + narrative beat.
7. **Star threshold numbers for World 1+ unlocks** — depends on total level-count planning, not yet known.
8. **Candy biome names/settings for Worlds 0-6, and exact level count per world** — deliberately deferred, per u1.

---

## 8. Art Direction — full discussion trail

**Palette:** u1 confirmed each world/biome gets its own distinct palette (wayfinding + world identity), with GomGom's core pink staying constant across biomes as the character anchor.

**Player differentiation:** u1 chose cosmetic heads only, uniform pink bodies — no per-player color variants. u2 flagged a genuine readability concern: Squish and Roll Dash both obscure the head, exactly when player-tracking matters most in co-op chaos, and offered a lightweight color-cue as a possible fix. u1 pushed back with a design instinct: embrace the confusion as comedy rather than fix it, consistent with Cozy Mischief.

u2's honest reassessment after thinking it through: agreed with u1, reversing the initial concern — Roll Dash's driver logic is determined mechanically (whoever's press triggered/grew the ball), not by player recognition, so passengers don't actually need to visually distinguish teammates to coordinate. Squish likely still leaves the heart-antenna visible even compressed, so most states keep identity readable anyway. The remaining ambiguity is functionally low-stakes and matches genre precedent (Gang Beasts, Human Fall Flat both use identical-looking characters for comedic confusion). No supplementary cue added.

**Section 8 status: locked.**

---

u1 asked for final reservations on Sections 1-6. While reviewing, u2 found two actual document inconsistencies (not soft opinions):
1. Section 4.1 still referred to "World 1 must be completed... World 2 onward unlocks via Star threshold" — stale from before Section 5 renamed the tutorial world to World 0. Fixed to World 0 / World 1 throughout Section 4 and the Open Questions table.
2. Section 4.2's economy table still said Stars are earned via "completion / hidden objective / time bonus" — stale wording from before the enemy/combat correction. Fixed to match "completion / time bonus / optional in-level challenge."

Both fixed directly in GDD.md; straightforward consistency errors, no discussion needed.

### Softer reservations raised (not yet decided, flagged for u1's awareness)
1. **Roll Dash driver handoff** — unspecified what happens to ball control if the current "driver" (most recent Squish-press to join/trigger) un-squishes mid-roll. Does control pass to another squished member automatically, or does the ball just stop?
2. **Fail-state granularity** — "no lives, pass/fail per level, retry the level" is locked, but the checkpoint granularity within a level isn't specified. This matters most for boss fights: a full level-reset after one mistake on a long co-op puzzle boss could feel punishing. Worth deciding before this hits the TDD, since it affects checkpoint/save-state architecture.
3. **Accessibility still bookmarked** — Section 12 remains open, and the mechanical depth we've built (4 verbs, 2 combos, boss puzzles) is real complexity sitting under a deliberately broad "Anyone" audience target from Section 1. Not a contradiction, just a reminder that Section 12 carries real weight when we get there.
