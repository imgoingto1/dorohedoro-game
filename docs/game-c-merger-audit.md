# Game C — Merger Audit & Design

2026-09-24 · Jay · Live version: the "Game C — Merger Audit & Design" Claude Doc. This file is a snapshot of it.

## Overview

This doc audits two Roblox codebases and will turn them into one design, Game C. It covers phases 1–2 (audit and comparison) first, then the interview, then the design.

| | Game A | Game B |
| --- | --- | --- |
| Studio place | "Map + Combat" (placeId 87872916277829) | "animation farm" (placeId 72078340292677) → `ServerStorage.CC` |
| Theme | Dorohedoro-inspired (your project) | Bleach-inspired (packed game, `v_Aug_17`, package README dated 3/27/2024) |
| Live source | ~340k lines (includes ~8 MB vendored ReactLua) | ~32 MB of source in 1,899 scripts |
| State | Active development, backups dated 2026-09-23/24 | Complete packaged game, unpacks on load |

**Method:** scripts were read directly from Studio through the MCP bridge. Vendored libraries (React, Packages, Cmdr) are noted but not audited line by line. Backup and archive folders are skipped unless live code depends on them.

**Status:** Phases 1–2 done; interview round 1 answered; Game C design comes after the interview.

## Game A — "The Hole" (Map + Combat)

Game A is an open-city Smoke-sorcerer brawler. Its combat and back end are deep, but its progression and endgame are thin. The strongest code is in combat (ActionManager → DamageLogic), and the world runs on about 50 small data-driven services. It began as a JJK base kit (`ProgressionConfig` still names "jjk's leftover" systems), and most of that kit has since been replaced.

### Core gameplay loop

Spawn in the Hole → walk or Smoke-door to a quest giver or job board → fight zombies, thieves or militia, or run errands → earn Yen and Burial Tags → buy food buffs and gear → random world events every 4–8 minutes pull players together (Blue Night, Field Boss, Killing Field) → PvP kills fill the server-wide War meter → at 100 a Riot doubles rewards and pays for kills.

### Progression

| Track | How it grows | Cap | What it gives |
| --- | --- | --- | --- |
| Level (derived from XP) | 40 XP per PvP kill, 60 per quest turn-in | 50 (32,340 XP) | Title + 1 attribute point per level |
| Attributes (T menu) | Points from levels, quests and the Grave Keeper (10 tags each) | 20 ranks × 4 (Smoke, Strength, Toughness, Vitality) | Rank 20 ≈ +70% of that stat. Respec: first free, then ¥250 with a 10-minute cooldown |
| Use-based stats (P-06) | Training at the Fight Club, or use; hourly caps | 50 | Only Endurance does anything (+3 HP per level). Weapon and Durability are saved but unused |
| Smoke type | Rolled once, weighted: Split 20, Gun 20, Regen 18, Mushroom 18, Curse 12, Lizard 8, Dinosaur 4 | — | 2–3 moves on Z/X/C. No reroll |
| Quirks | Rolled once: 1 good + 1 bad | — | ±5–15% to HP, Smoke, speed, damage; food-related quirks |
| Gear | Bought: 6 weapons, 6 equipment pieces, 1 quest mask | — | Flat ±5–20% modifiers |
| Gang rep | Killing Ashmask raiders | 100, decays 5/day after 24 h | Unlocks gang dialogue (10) and premium shop rows (30) |

At 100 XP per quest, level 50 takes about 540 quest completions. Maxing all attributes needs 80 points, but levels give only 49. The gap is filled with tag purchases, which makes Burial Tags the real long-term currency.

### Combat

The server is authoritative. The client sends input packets (buffer-encoded `Packet` library), the server runs action modules, and state is replicated back as VFX events.

```mermaid
flowchart LR
  In[Client input<br/>Packets] --> G[RemoteGuard<br/>rate limit]
  G --> AM[ActionManager<br/>priority, buffer, cancel]
  AM --> Mods[M1 / M2 / Uppercut<br/>Dodge / Block / Slide]
  Mods --> HB[HitBox]
  HB --> DL[DamageLogic.Processed]
  DL --> TA[TakeAction<br/>stun, knockback, stagger]
  TA --> Rep[Packets.Replicate<br/>client VFX]
```

Every hit passes through `DamageLogic.Processed`, which checks these in order: safe zone → spawn grace → 45-stud reach check → speed-hack check → multiplier stack → i-frames (perfect dodge) → parry → ragdoll → block and posture → stagger gain → world hooks.

- **M1:** 4–5 hit combos per weapon, plus running and aerial variants. Air combat has an air-weld juggle and a downslam finisher.
- **M2 (heavy):** separate module per weapon. A landed M2 silences Smoke users for 2.5 s.
- **Block, parry and posture:** block opens with a 0.25 s parry window. A clean parry gives the defender a 1 s Riposte (+25% damage). Blocked hits add posture (damage × 1.7) and deal 15% chip damage. At 100 posture the guard breaks and the player is Paralyzed. Posture drains after 2 s without a hit.
- **Stagger meter:** built only by skilled play (being parried +40, eating a counter +30, unblocked heavy +20, being interrupted +12). At 100 it opens a Finisher window: the next M2 deals ×1.5.
- **Dodge:** has i-frames. A perfect dodge (hit lands inside the i-frames) resets the Dodge cooldown and makes the next hit ×1.3.
- **Feint:** cancels an M1 into a feint on a 3 s cooldown.
- **Input buffer:** 0.6 s. Dodge and Block may cancel an attack's recovery, but only after its hitbox has fired.
- **Smoke moves:** cost Smoke from a flat 200 bar. Landed M1s refund 1.5% of the bar. Types range from DoT (Curse) and traps (Mushroom) to transformation forms (Lizard, Dinosaur).
- **Weapons:** Katana, Fist, Axe, Trident, Dagger, Sword and SwordShield. Only Katana and Fist have published animations; the other five use `rbxassetid://0`, borrowed Fist clips or placeholders.

### Character development

There is one origin, Sorcerer. Human is coded, but its roll chance is 0. The build space is: Smoke type (random) × weapon × attribute spread × 3 gear slots × quirks. Vows (Fists, Flames) exist as modules but are switched off. A Devil Awakening trial is scaffolded in 4 stages with fail-on-death, but its content is still TODO.

### Content

| Type | What exists |
| --- | --- |
| Map | Hole city: Shibuya-style streets, SecondWard, Brick Estate, Lab, Warehouse, Food Alley. 29 POIs, 3 danger zones (Kanon Park D1, Old Town Backstreets D2, Port Island D3), 3 secrets, 2 safe zones (The Diner, Hell), 34 event spots |
| Second place | Zogan's Academy Grounds, a separate place used by the tutorial questline |
| NPCs | 15 dialogue NPCs: Liaison, Madame Ise, the Grave Keeper, the arms dealer, 4 food-alley vendors, the Trainer, Dr. Mizoguchi and others |
| Quests | 28: a 9-step Enrollment chain, a 4-step Advanced chain and 15 repeatables (cooldowns of 8–30 minutes) |
| Events | 12 random events: Smoke Surge, Supply Cache, Whisper, Toxic Rain, Blue Night, Cleanup Day, Ghost Night, Rule of the Hour, Party Mishap, Killing Field, Ashmask Sweep, Field Boss (The Skinner) |
| Enemies | Zombie, Specimen, Night Stalker, Thief, Ashmask grunt and captain, Proctor Dunmore (420 HP), The Skinner (800 HP, scales with players). Archetypes: Shield, Rusher, Thrower, Flanker |
| Exploration | 7 Smoke doors (toll fast travel), 8 hidden Hell entrances (the reward for finding all is the Devil's Mask), Grudge Monument, rumor board, discovery toasts |

### Economy

- **Currencies:** Yen and Burial Tags. Tags cash out at ¥30–35 each and also buy attribute points.
- **Faucets:** quests (¥40–300), jobs (¥80 + 2 tags), events (¥15–150), Riot kills (¥25), Killing Field kills (¥20).
- **Sinks:** food buffs (¥35–250), gear (¥250–500 each, about ¥4,300 for everything), door tolls (¥30/60), respec (¥250), Black Smoke (¥120).
- **Guardrails:** Yen can't go negative or fractional, save data is sanitised on load, and every change is logged to `EconomyLog` and Roblox Analytics.
- **No trading.**

### PvP

PvP is open everywhere except safe zones. PvP kills give XP (40), Elo rating (K = 24, with a leaderboard), War meter progress and bonus Yen during a Riot or Killing Field. Party members can still hit each other; there is no friendly-fire rule.

### PvE

PvE comes from quest mobs, job-board zombie hunts, events and one field boss. The AI stack is solid: NPCController (37k chars), a swarm token system (only N attackers at once), navigation, ranged throwers and archetypes. Mobs and bosses gain +35% health and damage per extra nearby player, capped at ×2.5. BossService supports phases at health thresholds.

### Death

There is almost no penalty: a 3 s respawn followed by 3 s of spawn grace. Nothing is lost — no items, Yen or XP. The only death rules are that Hex hits harder if the caster dies and that dying mid-trial resets the Devil Trial.

### Endgame

There effectively isn't one. The reachable goals are level 50, 80 attribute ranks, Elo ladder rank, all Hell entrances and the Devil Trial (unfinished).

### Social

- **Parties:** in-server only, via chat commands and a React panel.
- **Clans:** saved in a DataStore. Created with chat commands and not race-safe. They have no gameplay effect.
- **Contracts:** Blue Night partner contracts record a pair but have no effect yet.
- **Leaderboards:** QuestsDone, BurialTags and Rating.

### Technical architecture

- **Two layers:** a Services layer (inherited kit: ActionManager, EntityLoader, AI, Data) and a World layer. The World layer is about 50 Scripts and Modules that talk through `ServerStorage.WorldSignals` Bindables and player attributes.
- **Data:** ProfileService with session locking, versioned migrations and a sanitiser. One store, `PlayerData_v1`, is shared across places.
- **Networking:** buffer packets for combat and plain Remotes for the World layer, rate-limited by `RemoteGuard` (token buckets, kick on abuse).
- **UI:** mixed. A React HUD (`ReactHudClient`, HudUI widgets), older ScreenGui scripts, and leftovers from the kit.
- **Tooling:** Cmdr admin commands and a 36k-char `SelfTest` harness for Studio.
- **Dead code:** `StarterGui.SkillTree` (Portuguese, needs a missing `Modules.SkillsReqs`), the `PurchaseSkill` and `DevProductNotif` remotes, a disabled Ghoul/CCG ProgressionService, and 7 dated backup folders in ServerStorage.

### Strengths

1. The combat has real depth: parry → riposte, perfect dodge → counter, stagger → finisher, posture, feints, air juggles and an input buffer. Skill wins fights.
2. It is secure by default: every remote is rate-limited, hits are checked for reach and speed server-side, and actions are allowlisted.
3. The world has personality. Events like Rule of the Hour, Ghost Night and Blue Night, plus hidden Hell entrances and the Grudge Monument, make the city feel alive.
4. The code is data-driven. Quests, events, shops, Smoke moves and archetypes are config tables, so content is cheap to add.
5. The save layer is production quality: ProfileService, migrations, sanitising and an economy log.

### Weaknesses

1. **No stakes.** Free death, no loot drops and no item loss mean PvP has no tension beyond Elo.
2. **Thin power curve.** Level 50 takes about 540 quests, but each level only gives 1 point and a title. The numbers are small, and the gear ceiling is reached in a few hours.
3. **Economy dead-ends.** Once the 12 gear items are bought, Yen only goes to food and tolls.
4. **One-shot randomness with no path forward.** Smoke type and quirks roll once, with no reroll or pity. A 4% Dinosaur roll is permanent.
5. **Stubbed systems.** Contracts, Devil Trial stages, clans and Vows exist but do nothing yet.
6. **Content is placeholder.** Five of seven weapons have no animations, and equipment uses part-built visuals.
7. **Hard to keep consistent.** There are about 50 services and several UI generations, and many systems communicate only through attributes.

## Game B — Soul-realm faction MMO (CC package)

Game B is a large Bleach-style faction MMO. Its biggest draws are **race transformation** and a **social hierarchy**. It gets there with heavy RNG, long timers and code that can't be maintained.

**Provenance: read this first.** Dozens of scripts carry the header `Saved by UniversalSynSaveInstance (Join to Copy Games)`, and much of the code is decompiled output (`v0`, `v1`… variable names). This package was ripped from someone else's live game with an exploit tool. Place names, the `TYPETEST` flags and the database name point to the Type://Soul family of games. None of its code, models, animations, VFX or maps can ship in a game you publish. **Game B is a design reference only.**

**Security findings (do not bring these files into Game A):**

- **Bytecode VM:** `NPCBoss.Data.Main` contains `FiOne`, a Lua bytecode interpreter that runs code through `setfenv`. That is the standard way to hide a backdoor.
- **Asset loading:** the `Shinigami` state machine pulls in `InsertService`.
- **Live secrets:** Discord webhook URLs are hard-coded in `Webhooks` and `FactionManager`. They are left out of this doc on purpose.
- **Outbound calls:** `Teleports` calls an external IP-geolocation API, and `FirebaseService` posts to an outside database.
- **Game A is clean.** A scan of the "Map + Combat" place found none of these markers.

### Core gameplay loop

Spawn as a Lost Soul → pick a path (Shinigami, Hollow, later Quincy) → grind EXP on NPCs, missions and PvP → rank up through grades (each rank has an EXP target, a minimum time at rank and, at higher ranks, PvP "grips") → unlock a release (Shikai, Res or Vollstandig) → join a faction's division and raids → chase Bankai or its equivalent and rare rolls.

### Progression

| Track | How it works |
| --- | --- |
| Race path | Lost Soul → Shinigami or Hollow. Hollows evolve Frisker → Fishbone → Menos → Adjuchas → Vasto Lorde, dying and respawning at each stage. Late hybrids: Arrancar, Vastocar, Visored. Quincy is its own faction |
| Rank grades | 14 grades from Trainee through Grade 5–1 and Special Grade to Semi-Elite and Elite. Each needs 20 EXP (×4 multiplier, ×0.7 for Shinigami) plus a minimum time at rank. Top ranks also need PvP grips, raid grips and division EXP |
| Shikai personality | Rolled: Kind Hearted, Squeamish, Vengeful, Hateful, Chaotic or Pure. Decides which activities feed release EXP (Hateful needs grips; Pure needs Hollow kills) |
| Stats | SP spent on Hakuda, Kendo, Kido, Speed, Healing and Bladedancer. Each scales a different system (Kendo scales posture damage; Speed scales flashstep) |
| Skills | Unlocked by SP thresholds, skill boxes (loot) and essences. There are 366 server skill modules |
| Releases | Rolled from rarity pools: common, rare, legendary, mythical (about 30 Shikai, 23 Res, 17 Vollstandig). Bankai needs a fight, time at rank and kill counts. True Bankai has a 12-hour cooldown |
| Identity rolls | Weapon type (1% rare), zanpakuto name and callout (10% rare), hilt and colours, eyes, 10% marking chance, clan (common, then legendary 3%, mythical 1%, "Hell" 0.5%) |
| Rerolls | Items, capped at 500–1,500 per type, sold through developer products |

### Combat

Combat runs on a state machine per entity type. Its core pieces:

- **Light attacks and posture:** block adds posture (base 50, scaled by Kendo). A posture break opens the defender up.
- **Deflect:** parrying stuns the attacker for `ParryStunTime`. There is a 0.4 s auto-parry frame.
- **Other defence:** counters on a 20 s cooldown, hyperarmor, and weapon clashes (`ClashSystem`).
- **Movement and pressure:** flashstep scales with Speed, and critical strikes are tied to the release.
- **Artifacts:** they change the defensive rules, for example extra block chip, a parry that absorbs Reiatsu, or reduced posture damage.
- **Resources:** skills cost Reiatsu. Releases add meter modes with regen buffs (Res +30%, Vollstandig +35%).

### Content

- **Places:** 9+ places — Main Menu, Karakura Town (shared hub), Soul Society, Rukon District, Hueco Mundo, Las Noches, Wandenreich City, a Clan War lobby and the Dangai travel corridor. Gates (Senkaimon, Garganta) can be broken, with 45 s cooldowns.
- **Enemies:** about 50 NPC and boss state machines: Lost Soul, Frisker, Fishbone, Menos, Adjuchas, Vasto Lorde, Jidanbo, Nozarashi, Visored, and more than 30 named bosses.
- **Activities:** boss raids with voting, party missions through queues (rank-gated), bounty board, Stalker missions, a timed AFK world, codes and weekend loot events.
- **Game modes:** Gladiator (lives), Art of Soul, King's Gambit, Soul Shuffle, Clan War, Championship and ranked arena.

### Economy

- **Currency:** Kan.
- **Items:** over 500 tradeables across accessories (10 slots), items, artifacts and clan items. Rarity counts: Common 16, Uncommon 5, Rare 20, Legendary 79, Mythical 103, Limited 12, plus 150 marked Unobtainable (developer-only).
- **Prices:** sell values from 150 (Common) to 4,000 (Legendary). Legendary accessories cost 40,000.
- **Faucets:** world lootboxes, raids, missions and codes, plus weekend multipliers of ×10 and ×20.
- **Trading:** full player trading (`TradeManager`).
- **Monetization:** developer products for rerolls and boxes, with a purchase-history receipt ledger.

### PvP

Full-loot-lite PvP built on **knock → carry → grip (execute)**. At 0 HP a player is knocked out for 12 s. The attacker can carry them, execute them (which counts toward rank-up) or let them get up. A 60 s combat tag punishes logging out mid-fight. Bounties go on repeat killers, jail has bail, and ranked Elo has a soft cap. Faction wars include Karakura raids, invasions, Quincy hunts, capture items and flags.

### PvE

The AI is state-machine driven. Each boss is a separate 170k-character copy of the same template, about 8 MB in total. Raids track damage contribution, scale respawn timers (30 s, +10 per death) and grant faction-wide raid buffs.

### Death

Death has real stakes:

- **Base respawn:** 3 s. In raids it is 30 s (+10 per death); in hunts it is 60 s (+30 per death).
- **Executions:** being gripped counts for the killer's progression.
- **Losses:** Hollows lose 5 of the 18 mask cracks they need. Release EXP can drop 10%. Held capture items, flags and dropped items fall to the ground, and Kan can be dropped.
- **Combat logging:** leaving mid-fight is recorded against the player.

### Endgame

Top division positions (Captain, Espada, Sternritter, King), True Bankai and the late forms, the ranked ladder and championship titles, mythical rolls, clan wars and rare cosmetics.

### Social

- **Factions:** Shinigami, Arrancar and Quincy.
- **Divisions:** 13 squads with ranked positions.
- **Clans:** DataStore rosters and clan wars.
- **Other:** parties with a mission queue, trading, titles, and cross-server messaging for admin actions.

### Technical architecture

- **Server layout:** Managers (Data, Faction, Division, Rank, Combat, Trade, AntiCheat) plus `EntityStateMachines`. The `Shinigami` state machine alone is 841k characters.
- **Client layout:** skill, release and VFX modules, about 5.4 MB.
- **Data:** ProfileService with rollback and version queries. Admin IDs are hard-coded.
- **Anti-cheat:** position, flight and noclip checks, remote history, and auto-ban on malformed input.
- **Duplication:** 7+ OLD or BACKUP copies of live managers. `FactionManager` has three near-identical variants of about 300k characters each.

### Strengths

1. **Transformation fantasy.** Evolving from Lost Soul to Hollow to Vasto Lorde, or Shinigami to Bankai, gives obvious, visible power spikes to chase.
2. **Personality-gated progression.** The same goal is reached through different activities, which nudges players into different playstyles.
3. **Knock → grip → execute.** Kills are a choice with consequences: mercy, execution, bounties, jail.
4. **Social hierarchy.** Division seats, faction raids and clan wars give players status to fight over.
5. **Distinct realms.** Each realm has its own identity, and travel between them (Dangai, gates) feels like a journey.

### Weaknesses

1. **Gacha everywhere.** Identity, weapon, release and clan are rolls; rerolls are paid; mythical pools and developer-only items exist. That creates pay-to-win pressure and frustrates players who roll badly.
2. **Time gates.** Minimum time per rank, a 12-hour True Bankai cooldown and AFK worlds stretch play instead of deepening it.
3. **Forced PvP.** Rank-ups need player grips, which locks PvE-focused players out.
4. **Inflation.** Weekend ×10 to ×20 multipliers and code handouts devalue the loot chase.
5. **Overwhelming for new players.** Nine places, six stats, dozens of forms and more than eight game modes.
6. **Code can't be maintained.** It is copy-paste bosses, decompiled output, hard-coded secrets and a probable backdoor, on top of the IP problem.

## System-by-system comparison

Game A supplies the engine: combat, data, world services and security. Game B supplies ideas for stakes, identity and long-term goals. Every B concept is rebuilt inside A's code. These verdicts are provisional until the interview confirms direction.

| System | Game A | Game B | Verdict | Why |
| --- | --- | --- | --- | --- |
| Combat | ActionManager + DamageLogic, server-authoritative | Per-entity state machines, deflect, counter, clash | **Keep A**, merge clash and hyperarmor | A is cleaner and already secured; clash and hyperarmor fill gaps for bosses and heavies |
| Movement | Sprint, slide, dodge, aerial, feint | Flashstep scaled by the Speed stat | **Keep A**, add mobility as a build option | A's feel is tuned; B's idea of a stat that shapes mobility adds build variety |
| Blocking | Posture 0–100, 15% chip, drains after 2 s | Posture scaled by Kendo, artifact modifiers | **Keep A** | Expose posture multipliers as stat and gear hooks |
| Parrying | 0.25 s window → Riposte, auto-parry follow-up | Deflect + parry stun, counter on a 20 s cooldown | **Keep A** | A already rewards the parry with Riposte and stagger |
| Dodging | I-frames, perfect dodge → counter | Flashstep only | **Keep A** | B has no equivalent |
| Abilities | Smoke type (2–3 moves), rolled once | 366 skills, releases rolled from rarity pools | **Rebuild** | Keep A's type identity; add earned unlocks and staged releases instead of one roll |
| Weapons | 7 buyable; 5 have no animations | Rolled weapon type with rarity | **Keep A** | Weapons are bought or earned, never rolled; the gap is animation work |
| Classes | None | Class field (9 modules) | **Remove** | Smoke type and weapon already define a role; a class would overlap |
| Races | Sorcerer only (Human coded at 0%) | Lost Soul → branching races and evolutions | **Merge** | A already has Human, Sorcerer and a Devil trial scaffold; B's transformation arc makes them a path |
| Stats | 4 attributes + 3 unused use-stats | 6 SP stats, each scaling a system | **Rebuild** | One stat system; drop the unused use-stats; each stat should change how you play, not just a number |
| Leveling | Level 1–50 from quest and PvP XP | Rank grades with EXP + time + grips | **Rebuild** | Named ranks with activity requirements, no wall-clock timers |
| Talents | Quirks (random good + bad) | Shikai personality gates EXP sources | **Merge** | Quirks become "temperament": it picks which activities speed your growth, not a flat stat |
| Skills | Smoke moves only | Skill boxes, SP-threshold trees | **Rebuild** | A small earned skill pool per Smoke type |
| Quests | 28 data-driven, 7 step types | Queue missions with rank gates | **Keep A** | A's quest engine is better; add rank gates |
| NPCs | 15 dialogue NPCs, actions, rep gates | Large NPC handler | **Keep A** | A's dialogue system is data-driven |
| Bosses | Proctor, The Skinner; BossService phases | 30+ copy-pasted bosses | **Keep A** framework | Add bosses as config + phase functions, never copies |
| Dungeons | None | Boss raids with voting, hunts | **New** (B-inspired) | Build on A's Survive/Kill steps, Mobs and BossService |
| World | One city + Academy place | 9+ realm places with gates | **Keep A**, grow slowly | One dense city plus a few gated places, not nine |
| Exploration | Doors, Hell entrances, secrets, rumors | Realm gates, Dangai | **Keep A** | A's discovery loop is its best non-combat feature |
| Loot | Mobs drop nothing | Lootboxes, rarity tiers | **New** | Game C needs drop tables |
| Inventory | 4 slots + consumables | 10 accessory slots + hotbar | **Keep A**, expand | 5–6 slots is enough |
| Economy | Yen + Burial Tags | Kan + rarity + paid rerolls | **Keep A** currencies, add sinks | No paid rerolls of power |
| Trading | None | Full trading | **Rebuild later** | Only once the loot table exists, with anti-dupe on UpdateAsync |
| Death | Nothing lost | Knock → carry → grip, drops, losses | **Merge** | A needs stakes; B's knock-down gives choice instead of instant loss |
| Respawning | 3 s + 3 s grace | 3 / 30 / 60 s, scaling per death | **Keep A**, add scaling in raids | |
| PvP | Open, Elo, War meter, Riots | Faction wars, bounties, jail | **Merge** | A's War meter plus B's bounty and jail give PvP consequences |
| PvE | Events, jobs, one field boss | Raids, hunts, bosses | **Keep A** + raids | |
| Factions | En's Gang rep (1 gang) | 3 factions with division seats | **Merge** | A's gang rep becomes factions with ranks and seats |
| Guilds | Clans (no effect, race-unsafe) | Clans + clan wars | **Rebuild** | After factions; decide whether clans or factions are the social unit |
| Parties | In-server, React panel | Parties + mission queue | **Keep A** | Add party-based credit sharing |
| Events | 12 random world events | Weekend loot ×10–×20, raids | **Keep A** | Drop reward multipliers — they inflate the economy |
| Crafting | None | Essences (partial) | **Optional** | Only if loot needs a second sink |
| Shops | 8 shops, rep-gated rows | Market rotation | **Keep A** | Add rotation later |
| Progression | Thin (1 point per level) | Deep but time-gated | **Rebuild** | The core problem to solve in Game C |
| Endgame | None | Seats, True Bankai, ranked, mythic rolls | **New** (B-inspired) | Seats + late transformations + ranked |
| UI | React HUD + legacy ScreenGuis | Many ScreenGuis | **Keep A** React HUD | Remove legacy UI |
| Data saving | ProfileService + migrations + sanitiser | ProfileService + rollback | **Keep A** | Add a rollback admin command (concept only) |
| Anti-exploit | RemoteGuard, reach/speed checks | Movement, flight and noclip checks | **Keep A** + movement checks | |

## Code triage

Only Game A's code is a candidate for reuse, and most of it is. Game B contributes design concepts, which get reimplemented from scratch in A's architecture.

### Reusable (Game A, keep as-is or with light edits)

| System | Main scripts | Depends on | Why keep |
| --- | --- | --- | --- |
| Combat core | `Services.Player.ActionManager` + action modules, `ServerStorage.Packages.DamageLogic`, `HitBox`, `AirCombat` | StateManager, TagService, TrackService, Packets, EntityLoader, RemoteGuard | Deep, tuned, server-authoritative, with reach and speed checks |
| Networking | `Remotes.Packets` / `UnPackets`, `World.RemoteGuard` | Packet library (buffers) | Compact, rate-limited, allowlisted actions |
| Save layer | `Services.Player.Data`, `World.DataMigrations`, `EconomyLog`, `DataStoreRetry` | ProfileService | Session locking, versioned migrations, sanitiser, analytics |
| AI | `Services.AI.AI` (NPCController, Swarm, Navigation, Ranged), `Config.NPCArchetypes` | ActionManager, DamageLogic, SimplePath | Archetypes and attack tokens make readable PvE |
| Mobs and bosses | `World.Mobs`, `MobScaling`, `BossService` | AI | Spawn, scale and phase from config |
| Quests | `World.QuestService`, `World.Quests` | WorldSignals, Mobs, NPCService | 7 step types, cross-place, repeatables |
| World layer | `WorldService`, `WorldQuery`, `POIIndexService`, `DoorService`, `HellService`, `RumorService`, `EventService` | WorldMarkers, CollectionService tags | Discovery and events are A's identity |
| NPCs and shops | `NPCService`, `Dialogues`, `EconomyService`, `Shops` | Guard, Reputation | Data-driven dialogue, rep gates |
| Items and buffs | `InventoryService`, `Items`, `Buffs` | Attributes read by DamageLogic | Simple and working |
| Ops | `ModerationService`, `SettingsService`, Cmdr, `SelfTest` | DataStoreRetry | Bans, per-player settings, admin commands, test harness |
| UI | `ReactHudClient`, `HudUI` | ReactLua | Newest UI generation; standardise on it |

### Rebuild (keep the concept, rewrite the code)

| System | Problem | Rebuild as |
| --- | --- | --- |
| `ProgressionService` + `ProgressionConfig` | 50 levels that each give only 1 point | Named ranks with activity requirements (B's grades, without timers) |
| `Stats` + `StatConfig` | Weapon and Durability XP are saved but do nothing | Fold into one stat system, or delete |
| `SmokeMoveService` roll | One permanent roll, no path forward | Rolled starting type + earned unlocks, a pity or reroll quest, staged forms |
| `ClanService` | Chat commands only; get-then-set races on the roster | UpdateAsync roster with React UI, after the faction decision |
| `Contracts` / `DevilTrial` | Scaffolding with no effect or content | Real stages hooked into the transformation path |
| Weapons 3–7 | `rbxassetid://0`, borrowed Fist clips | Animate or cut; ship 3–4 finished weapons rather than 7 half-done |

### Remove

- `StarterGui.SkillTree` (dead JJK kit UI, needs a missing `Modules.SkillsReqs`) and the `PurchaseSkill` and `DevProductNotif` remotes.
- The disabled Ghoul/CCG ProgressionService and the legacy `TumorGrade` save field (by migration).
- Seven `Backup_*` / `Archive_*` folders in ServerStorage. Move them to a local file, because they bloat the place and confuse search.
- About 90 loose parts and models in the root of `Workspace.Map`. Move them into region folders.

### Merge (Game B concept → Game A code)

| Concept from B | Where it lands in A |
| --- | --- |
| Knock → carry → grip (execute) | New `Downed` state in StateManager; a DamageLogic.TakeAction hook replaces instant death in PvP |
| Personality-gated growth | QuirkService → temperament; ProgressionService reads which activity feeds rank EXP |
| Staged transformations | SmokeMoveService forms + the DevilTrial pipeline |
| Faction seats | Reputation module generalised to several factions; seats as a ranked list per faction |
| Bounty and jail | WarService + WorldSignals.PlayerKilled |
| Raid contribution credit | Mobs `CreditRange` → damage-share tracking in BossService |
| Clash and hyperarmor | DamageLogic.Processed (hyperarmor = ignore stun) and a clash branch when two M2s meet |

### Optional (later, not in the first build)

Trading, crafting and essences, arena game modes (Gladiator, King's Gambit), ranked seasons, and more realm places.

### Game B migration verdict

**Nothing moves.** Every B system above is rebuilt from its idea, not its code. There are three reasons:

1. **Ownership:** B was ripped from another developer's game.
2. **Safety:** it contains a bytecode VM and live secrets.
3. **Maintainability:** decompiled, copy-pasted code can't be maintained.

Keep the `animation farm` place away from your published place, and never paste CC scripts or models into it.

## Interview log

Round 1 covers the decisions everything else depends on. Answers get recorded here as they come in; the newest answer wins if two conflict.

### Round 1 — identity and stakes (asked 2026-09-24)

| # | Question | Answer |
| --- | --- | --- |
| 1 | PvP, PvE or a mix — and where is PvP allowed? | **Mixed, zoned.** PvE progression everywhere. PvP is open in danger zones and during events (Riot, Killing Field). Safe zones stay safe. |
| 2 | What should death cost? | **Knock-down + soft loss.** At 0 HP you are downed; the attacker can spare, carry or execute. An execution drops a share of carried Yen and Tags. Gear and progression are never lost. |
| 3 | How is a character's power identity decided? | **Rolled and permanent.** Smoke type stays a one-time roll. Rerolls exist only as rare drops, so the roll is a long-term chase. |
| 4 | What goes in the public GitHub repo? | **Everything**, including the provenance and security sections. |

World size moves to round 2.

### Next rounds (planned)

- **Progression:** named ranks vs levels, how grindy, whether transformations (Sorcerer → Devil) are the endgame.
- **Combat:** how important weapons are vs Smoke, whether downed players can be carried, what bosses should test.
- **Social:** factions vs clans as the main group, whether seats are competitive, trading or not.
- **Monetization:** what, if anything, is sold.

## Game C design

Filled in after the interview.
