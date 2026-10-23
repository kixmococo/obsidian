# First Login & GM Command Reference — Singleplayer WoW Classic (CMaNGOS)

Two command surfaces exist and they are not the same:

- **Server console** — `docker attach wow-mangosd`. No dot prefix. Detach with
  **Ctrl-P, Ctrl-Q** (Ctrl-C shuts the server down).
- **In-game chat** — commands start with a dot (`.additem`), require GM level on
  your account.

Command sets drift slightly between core revisions. The ground truth on YOUR build
is always `.commands` (list everything available to your GM level) and
`.help <command>` (syntax for one command). If anything below errors, check those
first before assuming breakage.

---

## 1. One-time first login setup

Console (`docker attach wow-mangosd`):

```
account create brad <password>
account set gmlevel brad 3
```

GM levels: 0 = player, 1 = moderator, 2 = gamemaster, 3 = administrator.
Level 3 unlocks everything below.

Then log in with the client, create a character, and in chat:

```
.gm on            -- enable GM mode (with GM_LOGIN_STATE=2 this persists)
.gm visible off   -- optional: invisible to bots
```

Sanity checks that the world is alive:

```
/who              -- bot population (ramps over ~10 min after server start)
.server info      -- uptime, connection count, revision
```

---

## 2. Items and gold

```
.lookup item <name>        -- find item IDs by name, e.g. .lookup item thunderfury
.additem <itemId> [count]  -- give item to selected target (yourself if none)
.additemset <setId>        -- give an entire armor set
.repairitems               -- repair everything
```

**Gold**: `.modify money` works in **copper**. 1g = 10,000 copper.

```
.modify money 10000000     -- +1,000 gold
.modify money -500000      -- -50 gold (negative works)
```

Examples:

```
.lookup item arcanite bar
.additem 12360 20          -- 20 Arcanite Bars
```

---

## 3. Leveling and character state

```
.levelup [levels]           -- level selected target (self if none); .levelup 10
.character level <name> <n> -- set a character to exact level n
.maxskill                   -- max out weapon skills for current level
.setskill <skillId> <value> [max]
.learn <spellId>            -- teach one spell
.learn all_myclass          -- every trainer spell for your class/level
.learn all_crafts           -- all professions + recipes
.learn all_lang             -- all languages
.revive                     -- resurrect selected/self
.die                        -- kill selected target
.cooldown                   -- clear all cooldowns
```

Leveling a bot: target it (or use `.character level <botname> <n>`). Note the
random-bot system manages its own population's levels; manual changes to random
bots may be re-randomized by the module over time. Bots on YOUR account keep
what you give them.

---

## 4. Movement and teleportation

```
.lookup tele <name>       -- search teleport destinations, e.g. .lookup tele orgr
.tele <location>          -- teleport yourself, e.g. .tele orgrimmar
.tele name <player> <loc> -- teleport another character (works on bots)
.appear <name>            -- go to a player/bot
.summon <name>            -- bring a player/bot to you
.recall                   -- return to pre-teleport position
.gps                      -- current map/zone/coords (useful when reporting bugs)
.modify speed 3           -- run speed multiplier (1 = normal, max 10)
.modify aspeed 3          -- all speeds (run/swim/fly-fall)
.cheat taxi on            -- all flight paths known (older cores: .taxicheat on)
.explorecheat 1           -- reveal the whole map
```

---

## 5. God mode and combat cheats

```
.cheat god on             -- invulnerability
.cheat power on           -- infinite mana/rage/energy
.cheat casttime on        -- instant casts
.cheat cooldown on        -- no cooldowns
.cheat status             -- show which cheats are active
.modify hp <n>            -- set max health
.modify scale 2           -- character size (cosmetic fun)
.modify mount <mountId>   -- force a mount
.aura <spellId>           -- apply any aura to yourself
.morph <displayId>        -- change model
```

If `.cheat` subcommands don't exist on your revision, the older flat forms
(`.god on`, etc.) are the fallback — `.commands` will show which family you have.

---

## 6. Quests

```
.lookup quest <name>
.quest add <questId>       -- put quest in your log
.quest complete <questId>  -- flag it complete (turn in at the NPC)
.quest remove <questId>
```

---

## 7. World manipulation (flavor)

```
.wchange 1 0.5            -- weather: type 1 rain, density 0.5 (0=fine, 2=snow, 3=storm)
.npc add <entry>          -- permanently spawn a creature at your position
.npc delete               -- remove selected creature spawn
.announce <text>          -- server-wide message
.send message <name> <text>
```

---

## 8. Playerbots — party and control

Two distinct bot populations:

**Random bots** (the 250 filling the world): not yours to command. Interact like
players — `/invite BotName`, they accept (config-dependent), assign a role, play.
They also initiate: dungeon whispers, trades, duels.

**Your bots** (alts on your own account, addable by name):

```
.bot add <name>           -- log one of your alts in as a bot
.bot remove <name>
```

**Whisper/party-chat commands to bots in your group** (ike3 lineage — full
reference: https://ike3.github.io/mangosbot-docs/):

```
follow / stay / flee      -- positioning
attack                    -- attack your target
grind                     -- freeform grinding nearby
summon                    -- bot comes to you
reset                     -- reset bot AI state
ll                        -- list bot's inventory
e <itemlink>              -- equip item
autogear                  -- self-equip best available
talents                   -- show/manage spec
co ?  /  nc ?             -- list combat / non-combat strategies
co +heal                  -- e.g. add heal strategy in combat
```

Admin-side random bot management (GM chat or console, syntax varies by
revision — check `.help bot`):

```
.bot reset                -- reset the random bot system (after big conf changes)
```

---

## 9. Server console operations (docker attach wow-mangosd)

```
server info
server shutdown 30        -- clean shutdown in 30s (world saves)
saveall                   -- force-save all characters
account onlinelist
send message <name> <text>
```

Host-side (from the repo directory):

```
docker compose down            -- clean stop (90s grace, world saves)
docker compose up -d           -- start
docker compose logs -f mangosd -- watch the world tick
docker compose restart mangosd -- apply .env bot/AH changes
./scripts/backup.sh            -- dump all DBs to ./backups/
```

---

## 10. Quality-of-life examples

```
.modify money 100000000        -- 10,000g: fund your AH habit
.levelup 59                    -- 1 -> 60 in one command
.learn all_myclass && .maxskill
.tele ironforge
.additem 19019                 -- Thunderfury, because you can
```

One honest note: heavy GM use bleeds the fun out of exactly the thing this
server was built for — a world that feels earned and alive. The commands are
here for testing, unsticking yourself, and the occasional indulgence; the
magic you described ("the AH had new listings and nobody was online") only
works if your character mostly lives by the world's rules.
