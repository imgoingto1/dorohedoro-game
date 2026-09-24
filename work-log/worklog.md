# Work Log — Map + Combat project

A running list of everything done on this project, pulled from `PROJECT.md`. Entries are
chronological and kept as history; where a later entry superseded an earlier one, the earlier one
carries an inline **Update** note rather than being deleted. For what the repo itself is for, see
[`CLAUDE.md`](../CLAUDE.md).

## Current state (as of 2026-09-24) — read this first

**Places:** Map + Combat (87872916277829) and Zogan's Academy Grounds (127609270845586), same
universe (GameId 10766477296), shared DataStores. **Back at code parity on 2026-09-24: 525/525 game
scripts identical** (hash-checked). Last `selftest`: Map + Combat 38 passed / 0 failed (combat-depth
pass), Academy 37 passed / 0 failed / 1 warning (the Rat NPC only exists in the Hole). The shared
folders are Roblox Packages whose Map + Combat edits were never published — publishing them would
turn the next sync into one "Update All".

**Built but never play-tested in a real match**
- Progression level-ups ("Level up!" toast, WorldInfo Rank line) from PvP kill / quest.
- Attribute/Smoke balance numbers (Jay: "seem very high" — first thing to dial back).
- `!party` / `!clan` chat commands end to end; Rating moving on a real PvP kill; the kill-confirm visual.
- A real parry into a riposte, and M2 guard-breaking the Ashmask Shieldbearer (scripted input can't
  time these).
- Blue Night contracts with two real players (only the pure-rule SelfTest covers pairing).
- The 2026-09-24 lamp pass *by eye* — logic verified in Play, but the Studio viewport rendered blank.
- Slide after a very brief (~0.05s) movement tap.

**Waiting on a decision from Jay**
- Whether the `Adv4` quest chain continues (`Adv5`+) — story call.
- Zogan's has 0 staffed shops; Academy players have nowhere to spend Yen.
- Tags have almost nothing to buy at high levels (Grave Keeper is the only sink).
- What a Blue Night pact actually does, how long it lasts, whether it can be broken.
- Devil trial stage content (all 4 stages are TODO scaffolds); a real tumor-transplant mechanic.
- Spore Burst: 24 Smoke for 6.8 dmg vs M1's free 10.2 — check vs a group (AoE).
- Holding LeftControl both fires Slide and arms the M1→Uppercut modifier.
- `Lighting.Technology` still can't be read from script; `LightingStyle` is now Realistic.
- Whether to publish the shared Packages from Map + Combat (UI action only).

**Not built / blocked**
- Trading, player reports, mute/chat moderation, duels/matchmaking (and the MemoryStore work that depends on it).
- Sound: no audio assets or sound-trigger system anywhere.
- Item/move icons (`Items` has no `Icon` field), real equipment meshes, real Smoke VFX.
- Lizard/Dinosaur forms still reskin the player's rig (all 16 Smoke moves have a cast animation as of 2026-09-24).
- Grudge Monument — built, Jay wants it removed, not yet removed.
- 13 missing MAP_3 props; unshared asset IDs (e.g. ColorMap 14565342511) causing permission errors.
- `GhoulProgCheckNPC` floating with no floor — likely intentional, unconfirmed.
- Hell and the Sorcerer world as separate places via package links — not started.
- `Controllers.Gui.Inventory` still logs "Infinite yield possible on GameUI" on slow loads.
- Smoke-cast screen tint (optional follow-up); tokenise `POIGuideClient` waypoint colours if more types are added.

**Resolved since the 2026-09-23 snapshot:** M1/fist reach (Fist hitbox (5,6,5), 3/3 hits at 6 studs);
final HUD look (VitalsHud replaced jjk's Player_Display bars); the fight pit was removed; Academy
parity; lamps without a visible source; animations freezing after a perfect dodge.

## Merge (three places into one)
- Merged **combat test** (combat framework) and **jjk combat game** (map + UI) into **Map + Combat** (target place, 87872916277829).
- Moved assets file-by-file via export/import `.rbxm` (Studio clipboard doesn't work cross-process): ServerStorage assets/packages, ReplicatedFirst preloader, Cmdr commands, Services.Studio config, combat workspace folders, all three map chunks (Shibuya, Districts, the rest), jjk NPCs, and StarterGui.
- Parented all jjk map content under `workspace.Map` and NPCs/characters under `workspace.Live` to satisfy the combat framework's raycast filters (map/live/visuals/ignore presets).
- Moved client scripts (`Controllers`, `Loader`, `VioletSkillsClient`) into `StarterPlayerScripts`.

## Error-fix pass
- Rewrote 86 broken map script paths (84 scripts, 85 replacements) after nesting the map under `workspace.Map`.
- Fixed the spawn bug: four `Spawns1` SpawnLocations were floating at y≈505 with no ground; raycast down and repositioned them; disabled the far-off `SpawnBox` spawn; deleted a leftover `Baseplate` slab and its SpawnLocation.
- Restored missing `GameUI` (broke the whole combat UI layer — health bar, hotbar, inventory).
- Restored missing `ReplicatedStorage.Packages.chrono` and `Packages.Visuals.Rocks`.
- Verified: 119-path audit against combat test showed zero missing paths.

## PlayerData collision → PlayerStats bridge
- Resolved the hard `ReplicatedStorage.PlayerData` naming collision (framework needs it as a ModuleScript, jjk UI needs it as a Folder) by adding a separate `ReplicatedStorage.PlayerStats` folder and repointing 8 jjk scripts to it.
- Built `ServerScriptService.PlayerStatsBridge` (creates per-player stats on join, clones AliveData, fires LoadGui, mirrors framework Yen live, cleans up on leave).
- Built `ServerScriptService.WorldInfoServer` (server uptime + player region).
- Added `StarterCharacter.AliveData`, `Remotes.LoadGui`, `DevProductNotif`, `PurchaseSkill`, `Assets.Ui`, `Assets.InteractHiglight`.
- Ported and rewired the WorldInfo HUD (Version / Character Name / ServerUptime / ServerRegion) to real data sources instead of jjk's broken web/RemoteFunction calls.

## Floating models / shops / locations fix
- Root-caused: `Import Roblox Model` lifts imports in the air to avoid overlap. Found and corrected Y-axis shifts on `MAP_3_Rest.rbxm` (+485.635), `LIVE_jjk_NPCs.rbxm` (+614.38), `WS_CombatFolders.rbxm` (left alone, worked out fine).
- Dropped all affected MAP_3 buildings and jjk NPCs back to their original heights (two single-undo-step moves).
- Logged 13 MAP_3 items still missing from the place (kitchen props, 2 loose parts) as a follow-up.
- Logged pre-existing jjk bugs (not caused by the merge): missing `CochleaOutside`, missing `SmallClouds`.

## Ghoul script cleanup
- Deleted two non-functional Ghoul Cafe teleport scripts; reworded a stray comment; left ghoul-named dependencies alone since other code still relies on them.

## World systems foundations (roadmap #4 Open World, #5 NPC dialogue, #7 Random Events)
- Built self-contained systems that don't touch the combat framework: `WorldMarkers` (POIs, danger zones, secrets, event spots), `WorldNPCs`, `WorldEvents`, `ReplicatedStorage.World` (config, dialogue trees, remotes), `WorldSignals`, `ServerScriptService.World` services, `StarterGui.WorldUI`.
- Danger zones with player attributes + banners; POI/secret discovery toasts; tagged dialogue NPCs with server-validated actions; random world events (CursedSurge, SupplyCache, rare Whisper) on a timer with reward hooks.
- Play-tested: zone banners, discovery toasts, dialogue flow, waypoints, scheduled events, reward toasts — all clean.

## No-asset shortlist build (largest single pass)
- **Saved data (F-01):** migrated `Services.Player.Data` to ProfileService; added new PlayerData fields for burial tags, origin, tumor grade, smoke type, stat XP, attribute points, discovered, hell entrances.
- **Smoke system (C-02, P-01, S-10):** grade/regen service, spend/add/reroll API, config, HUD; move-type framework (Split, Gun, Curse, Regeneration/Strength, Mushroom, Lizard, Dinosaur) with per-type handlers and a client controller; damage scaling hook.
- **Economy / NPCs (N-01, N-02, E-12):** shop catalog + buy service, buffs, reward granting (Yen/Tags/Smoke), shop UI; Shopkeeper/Diner/Gyoza Cook/Grave Keeper NPCs; safe zones that block damage.
- **World / events (W-11, W-03, E-01, W-09):** toxic rain / atmosphere effects, ToxicRain and BlueNight (fist-only zombie) events, Hell dimension with 8 tagged entrances and saved per-player progress, lawless zones with pay bonuses, Smoke Doors travel network, Grudge Monument stun mechanic (flagged for removal later).
- Smoke rebalance: flat base + points-scaled max, percentage regen, ranks/XP removed.
- POI guide tool (hold to list, tap to beam-guide/cycle).
- Character attribute tree (Strength/Toughness/Vitality/Smoke points) feeding into combat and Smoke.
- Rumor board + Fortune Teller NPC; Smoke-related combat hooks (M1 refund, M2 silence).
- Zombie job board + hospital director NPC, CleanupDay/GhostNight/RuleOfHour events.
- War meter (Boiling Point) driving timed Riot events; PartyMishap and KillingField world events.
- Split off **Zogan's Academy Grounds** (127609270845586) as a second place from the old JJK School Map, with a gate + travel service linking it to the main place; not yet published with systems. **Update 2026-09-17:** brought to full parity with Map + Combat (see "Zogan's Academy Grounds brought up to parity").

## Open / outstanding items (from PROJECT.md, as of 2026-09-16)
*Snapshot — see "Current state" at the top for what is still open today.*
- 13 missing MAP_3 props (kitchen items, 2 loose parts) not yet restored.
- Unshared asset IDs still causing permission errors (list in PROJECT.md).
- `GhoulProgCheckNPC` floating with no floor nearby — likely an intentional hidden check NPC, unconfirmed.
- ~~`Lab2` teleport script error (`Triggered` not a member of Part).~~ **Resolved** — already fixed 2026-09-16 (confirmed 2026-09-22).
- ~~Missing `ReplicatedStorage.Weather`, `Remotes.Progression`, `Remotes.CombatTag`.~~ **Resolved as harmless** (2026-09-22): `WeatherManager`/`DaylightManager` are disabled, `Remotes.Progression` was only used by the now-disabled jjk `ProgressionService`, and both `CombatTag` users degrade safely.
- GameUI and jjk's Player_Display/Custom Inventory both draw a health bar/hotbar — final HUD look not decided. (Custom Inventory itself was later rebuilt as the single real inventory.)
- Grudge Monument mechanic built but Jay doesn't want it — pending removal.
- ~~Zogan's Academy Grounds place published but has no game systems yet.~~ **Resolved** 2026-09-17 — ported to parity.
- Hell and the Sorcerer world are planned to become separate places sharing data via package links — not started.

## Backend hardening / PvP & Smoke optimization pass (2026-09-17, in progress)

Jay: current focus shifted from front-end/map work to **backend optimization**, specifically
making the **PvP** and **Smoke** systems more optimal. Confirmed in-game by script timestamps
("Jay, 2026-09-17") on the following, all already present in the place as of this session:

- **`ServerScriptService.World.RemoteGuard`** — per-player token-bucket rate limiting for every
  client→server remote (combat inputs M1/M2/Uppercut/Dodge/Sprint/Block/Equip/Feint/Slide, plus
  world remotes like ShopBuy, SmokeCast, DoorTravel, AttributeSpend, etc). Strikes accumulate per
  player/minute; repeat floods get kicked (never in Studio). `Guard.stats()` feeds admin tools.
- **`ServerScriptService.World.DataMigrations`** — versioned save upgrades (`STEPS` table) run
  right after `Profile:Reconcile()`; `sanitize()` repairs NaN/negative/fractional currencies and
  wrong-typed tables so one bad write can't brick a save. Warns on saves over 1MB (DataStore cap
  is 4MB).
- **`ServerScriptService.World.EconomyLog`** — records every Yen/Tags delta by source, forwards
  to Roblox Analytics (`AnalyticsService:LogEconomyEvent`) on live servers only, keeps per-source
  running totals for the `economy` Cmdr command.
- **`ServerScriptService.World.LeaderboardService`** — cross-server top-10 OrderedDataStore boards
  (PitBest, QuestsDone, BurialTags); writes throttled to once per 5 min per player (only on
  change) and on leave/`BindToClose`; reads every 2 min into a `workspace` attribute the client
  leaderboard renderer (`LeaderboardClient`) draws from.
- **`ServerScriptService.World.SettingsService`** — per-player settings (currently just
  `ScreenShake`) mirrored to player attributes; only non-default values are persisted, keeping
  saves small.
- **`ServerScriptService.World.SelfTest`** (+ `SelfTestRunner`) — one-shot automated back-end
  check: every World module loads, every expected remote exists, quest/shop data integrity,
  RemoteGuard actually throttles, DataMigrations actually repairs a broken save, EconomyLog
  actually records, HandlePoseDriver doesn't drift the weapon grip across overlapping swings, and
  (with a live player) settings/quirk/data-version attrs, a quest debug hook, and — in the Hole
  place only — a pit start+quit round-trip that must refund its fee exactly. Triggered via
  `workspace:SetAttribute("RunSelfTest", true)` or the Cmdr `selftest` command.
- **`StarterPlayer...Controllers.StateController.Misc.HitWeight`** (client) — PvP hit-feel pass:
  brief hitstop on landed M1s (freezes only Action-priority animation tracks, not idle/walk),
  camera shake (bigger for the victim, biggest on a finisher), and an FOV kick on finisher hits.
  Purely cosmetic/local — server hitboxes and timing are untouched. Respects the new
  `Setting_ScreenShake` player attribute (shake/FOV off, hitstop always on).

### Not yet found in the place (likely the next steps)
**Update:** addressed by the next entry ("PvP / Smoke optimization pass").

- No script dated 2026-09-17 touches `ServerStorage.Packages.DamageLogic`, `Smoke`,
  `SmokeService`, or `SmokeMoveService` directly yet — those are still at their 2026-09-15/16
  state. If "make Smoke more optimal" means the Smoke *system's* performance/logic (not just the
  HitWeight/RemoteGuard hooks around it), that work hasn't landed in-game yet.
- Worth checking next time before writing new code: `DamageLogic`'s hit-processing path (hot path
  for every PvP hit) and `SmokeService`'s regen loop for any obvious per-hit or per-tick cost that
  could be cut.

## PvP / Smoke optimization pass (2026-09-17, applied live in Studio Edit mode)

Two concrete, behavior-preserving perf fixes made directly to the place (not yet play-tested —
do that before saving, and run the Cmdr `selftest` / `workspace:SetAttribute("RunSelfTest", true)`
check to confirm nothing broke):

1. **`ServerStorage.Packages.DamageLogic` (`Shared.Processed`) — the per-hit PvP hot path.**
   Reordered the early-exit guards. It used to compute `Character:GetPivot()`,
   `Target:GetPivot()`, `TagService:ReturnTags(Target)`, and a trig `FacingAttacker` angle
   **before** checking `TargetHumanoid.Health <= 0`, the SafeZone attribute check, or the
   SpawnGrace attribute check — so every hit against an already-dead target, into/out of a safe
   zone, or during spawn grace paid for pivot math + tag lookups + trig it was about to throw
   away. Now the three cheap attribute-based early returns run first, and the CFrame/Tags/
   FacingAttacker block only runs for hits that are actually going to be processed. No behavior
   change — these are independent early-return conditions, none of the reordered checks or the
   moved-down block depend on each other.

2. **`ServerScriptService.World.SmokeService` and `.SmokeMoveService` — startup catch-up loop.**
   Both had `task.spawn(function() while true do ... task.wait(1) end end)` meant to catch a
   race where a player's profile finishes loading before the script's `ProfileLoaded:Connect`
   fires. That race is startup-only (`ProfileLoaded` already handles every normal join), but the
   loop ran **forever**, every second, for the entire life of every server, iterating
   `Players:GetPlayers()` each time for no reason after the first few seconds. Replaced both with
   a bounded loop: up to 10 attempts, 1s apart, that stops as soon as every currently-known player
   is ready (or after ~10s regardless). Same catch-up behavior, no permanent per-second poll.

### Not done yet
- No changes made to `DamageLogic.Shared.TakeAction`, `Shared.Parry`, `SmokeService`'s regen
  tick body, or `SmokeMoveService`'s cast handler — read through all of them, didn't find another
  change worth making without more direction (e.g. is "optimize Smoke" about raw performance, or
  about rebalancing costs/cooldowns/regen numbers? those are different asks).
- ~~Not yet play-tested. Run a playtest + Cmdr `selftest` before considering this pass done, then
  save the place.~~ **Update:** `selftest` 18/0 in both places from 2026-09-17 onward, and a clean
  live playtest on 2026-09-22.

## Deferred — reminders for later (2026-09-17)

Jay: skip for now, revisit later. Not started. **Update:** most of this list was built later the
same day — status per item below.

- **Combat anti-cheat / server-side movement & hit validation.** `DamageLogic` trusts whatever
  hit lands; nothing checks for impossible speed/teleport/reach. Pairs with RemoteGuard.
  **Done:** reach check (`MAX_HIT_DISTANCE = 45`) and movement-speed check (`MAX_IMPLIED_SPEED = 150`).
- **DataStore write queue / backoff.** Saves, `LeaderboardService` writes, etc. fire directly
  today; no queue to smooth out throttling under load.
  **Done (backoff):** `DataStoreRetry` wraps Leaderboard/Moderation/Clan calls. A write queue was
  deliberately not built — no load problem to justify it.
- **Moderation / report system.** No persisted ban/mute system exists yet.
  **Partly done:** persisted bans (`ModerationService`, Cmdr `ban`/`unban`). Reports and mute not built.
- **PvP ranking (ELO/MMR).** `PlayerKilled` fires (WarService, EconomyLog-adjacent) but nothing
  turns kills into a ranking. **Done:** `RankingService` (Elo, K=24) + `Rating` leaderboard.
- **MemoryStoreService for live matchmaking/party state**, if parties/duels get planned — cheaper
  and faster than DataStore for short-lived cross-server state. **Skipped:** parties are
  in-memory per server; no duel/matchmaking feature exists to need it.

## Backend content gaps — placeholder/stub systems still needing real work (2026-09-17)

Found by grepping the codebase for `placeholder` / incomplete comments. These are gameplay
*content* backends (not infra) that already have scaffolding but are running on stand-in data:

**Update:** all resolved by the following entries except Lizard/Dinosaur forms — progression
built, attribute/Smoke numbers tuned, loot table + DevilMask added, and the persistence question
answered (saves do persist via ProfileService; the `PlayerStatsBridge` comment is stale).

- **Character progression (Level, LevelExp, Experiance, Rolls, Points, AuraColor, Title,
  Faction).** `ServerScriptService.PlayerStatsBridge`'s own header says these are jjk-UI concepts
  "with no framework equivalent yet," created with hardcoded defaults purely so the HUD renders
  without erroring. Only `Money` (mirrors the framework's Yen) is real. This is the single biggest
  gap — there is no real leveling/XP/faction/title system behind the HUD that displays them.
- **Attribute tree balance.** `ReplicatedStorage.World.AttributeConfig`: "Numbers are placeholders
  until real balance is done." The character tree (Strength/Toughness/Vitality/Smoke) works
  mechanically but the numbers were never tuned.
- **Smoke-point payouts.** `ReplicatedStorage.World.SmokeConfig`: "What each point in the Smoke
  attribute gives (placeholder numbers until the tree exists)."
- **Lizard / Dinosaur Smoke forms.** `SmokeTypes` and `SmokeMoves.Forms/Lizard/Dinosaur`: explicitly
  "placeholder form until a real model exists" — the transformation logic works but reskins the
  player's own rig instead of using real creature models/animations.
- **Loot system.** `ServerScriptService.World.EventService`: random-event rewards (CursedSurge,
  SupplyCache, etc.) still just show a placeholder toast (`WorldConfig.PlaceholderRewardNotice`)
  instead of granting a real item — `EconomyService` pays Yen/Smoke, but there's no loot/item-drop
  system to hand out yet.
- **Devil cosmetic reward.** `WorldConfig.AllFoundReward` (Q-08, "find every POI/secret") pays
  Tags + Yen as "a placeholder until the devil cosmetic exists" — the actual reward item was never
  made.
- **Open question worth resolving:** `PlayerStatsBridge`'s comment claims the combat framework
  "deliberately disables persistence - every join gets a fresh copy of PlayerData.DEFAULT_PLAYER_DATA."
  That was true pre-merge; F-01 (2026-09-15) later put `Services.Player.Data` on ProfileService, so
  this comment may now be stale. Worth confirming whether Level/Faction/Title/etc. would actually
  survive a rejoin today or still reset every session — decides how urgent the progression-system
  gap above really is.

## Backend content pass #2 — items 5 & 6 done, 1/2/3 need your input before I build them (2026-09-17)

Jay said "work on all of these besides [Lizard/Dinosaur forms]." Two are done; three need a design
call first (per the standing rule: tell you what an idea is before building it) because the
numbers/behavior aren't derivable from the code - they're decisions.

### Done

- **#5 Loot system.** Turned out the "placeholder reward" comment in `EventService` was stale -
  `EconomyService` already pays real Yen/Smoke/Tags for every event reward (`WorldConfig.
  PlaceholderRewardNotice` was already `false`). What was actually missing was real **item**
  drops. Added a 6-entry weighted loot table to `EconomyService` (reusing the existing GasMask /
  BoneMask / CleanerCoat / ScrapVest / SmokeCharm / GraveBell equipment - no new items invented)
  and wired `SupplyBundle` (the SupplyCache event) to roll one on top of its ¥120. Other event
  rewards (SurgeEssence, kill bounties, etc.) are intentionally left currency-only - they're not
  "cache" themed.
- **#6 Devil cosmetic.** `WorldConfig.AllFoundReward` (found all 8 Hell entrances, `HellService`)
  paid Tags+Yen with a comment saying it was "a placeholder until the devil cosmetic exists."
  Added a real item, `Items.Defs.DevilMask` (Head slot, +8% damage, red Neon look, matches the
  existing part-built-look style every other piece of equipment uses - no new asset needed) and
  granted it alongside the existing Tags/Yen payout in `HellService.recordEntrance`.

### Needs your call before I build it
**Update:** both answered — see "Progression system built" and "Attribute tree / Smoke balance pass" below.

- **#1 Progression (Level/XP/Faction/Title).** Bigger finding: the jjk HUD's
  `StarterGui.Hud.ProgressionService` (and its `ProgressionRankRequirements` /
  `ProgressionTitleRequirements` modules) is **dead code** - it requires
  `ReplicatedStorage.Modules.Managers.TitleManager` and `ReplicatedStorage.Remotes.Progression`,
  both of which PROJECT.md already lists as deliberately-skipped jjk dependencies that were never
  ported. It's also **thematically wrong** for this game: it's Tokyo-Ghoul-flavored (Ghoul ranks
  F- through SSS+, a CCG investigator track) left over from jjk, not this game's Sorcerer/Smoke
  theme. Building "real" progression by resurrecting that code would wire the HUD to the wrong
  fiction. Persistence is confirmed fine either way - `Services.Player.Data` really is on
  ProfileService (F-01), so the "framework disables persistence" comment in `PlayerStatsBridge` is
  itself stale. **Need from you:** should Level/Title come from a new, small XP system I design
  (sourced from real signals that already exist - kills, quests done, Pit runs), with
  Sorcerer/Smoke-appropriate rank names instead of Ghoul/CCG - or do you want to keep the
  Ghoul/CCG flavor and I restore/rebuild its two missing dependencies instead?
- **#2 Attribute tree balance / #3 Smoke-point payouts.** Both configs (`AttributeConfig`,
  `SmokeConfig`) say outright "numbers are placeholders until real balance is done" / "until the
  tree exists." This isn't a bug to fix, it's tuning - I don't have a target power curve to hit.
  **Need from you:** roughly how strong should a fully-invested player feel vs. a fresh one (e.g.
  +X% damage / HP / Smoke at max rank), and how many attribute points should a character
  realistically earn by end-game? Once I have a target I can turn the placeholders into real
  numbers and you tune from playtesting.

---
*Generated 2026-09-17 from PROJECT.md. This file summarizes; PROJECT.md remains the authoritative, detailed log.*
*Updated 2026-09-17: added the backend hardening / PvP hit-feel pass, the PvP/Smoke optimization pass, deferred infra reminders, the placeholder/stub content audit, and content pass #2 (loot system + devil cosmetic shipped; progression + balance blocked on design input) — verified directly against the live Studio place (script header timestamps + direct script reads), so this survives if the terminal closes mid-session.*

## Progression system built (2026-09-17)

Jay's call: build new Sorcerer/Smoke-themed ranks (not the old Ghoul/CCG system). Built and wired
in Studio Edit mode - not play-tested yet.

- **Disabled `StarterGui.Hud.ProgressionService`** (`Disabled = true`, reversible, no code deleted).
  It was hard-erroring on every client - `require(ReplicatedStorage.Modules.Managers.TitleManager)`
  where `ReplicatedStorage.Modules` doesn't exist at all in this place. Confirmed nothing else
  reads `LevelExp`/`Experiance`/etc. except that dead subtree, so disabling it is safe.
- **New `ReplicatedStorage.World.ProgressionConfig`** - the XP/level curve as data, not hardcoded
  logic: `Config.XP` (PvPKill=40, QuestComplete=60, PitClear=25), a linear level-cost curve
  (`BaseCost`=60, `CostGrowth`=25, `MaxLevel`=50), and a small Title ladder (Novice -> Initiate ->
  Adept -> Veteran -> Elite -> Ascendant -> Sovereign at levels 1/5/10/16/24/35/50). All five
  numbers are easy to retune in one place once you've played with it.
- **New `ServerScriptService.World.ProgressionService`** - awards XP off signals that already
  fire for other systems (no new trust surface): `WorldSignals.PlayerKilled` for PvP kills,
  `WorldSignals.GrantReward` with `Source == "Quest"` for quest turn-ins, `Source == "Pit"` for
  Pit clears. Level is always *derived* from `PlayerData.XP` (added as a new persisted field,
  default 0 - old saves get it automatically via ProfileService's `Reconcile()`, no migration
  step needed), never stored separately, so there's nothing to desync. Mirrors Level/LevelExp
  (progress fraction)/Title into the existing `PlayerStats[<name>]` values jjk's HUD already
  reads (WorldInfo's Rank line uses Title already). Each level-up also grants 1 AttributePoint,
  so leveling has a real gameplay payoff instead of being cosmetic-only.
- Not touched: `Faction` (still static "Sorcerer" default from `PlayerStatsBridge` - no signal
  exists yet for changing faction, so there was nothing real to wire), `Rolls`/`Points`/
  `AuraColor*` (still inert defaults; nothing in the live UI reads them - confirmed by grep).
- **Not play-tested yet.** Before saving: play test a PvP kill, a quest turn-in, and a Pit clear;
  confirm the "Level up!" toast fires and WorldInfo's Rank line updates.

## Attribute tree / Smoke balance pass (2026-09-17)

Jay's call: a maxed-out character (rank 20, the tree's existing cap) should feel **strong** -
roughly +60-100% power over a fresh one. Turned the "placeholder" configs into real numbers
hitting that target; nothing structural changed, just the tuning constants each system already
reads.

| Attribute | Before (at max rank 20) | After (at max rank 20) |
|---|---|---|
| Strength (`AttributeConfig.PerRank`) | +40% damage | **+70% damage** (0.02 -> 0.035/rank) |
| Toughness (`AttributeConfig.PerRank`) | -26% damage taken (~+35% eHP) | **-43% damage taken (~+76% eHP)** (0.985 -> 0.972/rank) |
| Vitality (`AttributeConfig.PerRank`) | +80 HP (+80% vs 100 base) | unchanged - already in range |
| Smoke max (`SmokeConfig.PerPoint.Max`) | +100 max Smoke (200 -> 300, +50%) | **+140 max Smoke (200 -> 340, +70%)** (5 -> 7/point) |
| Smoke regen (`SmokeConfig.PerPoint.Regen`, compounds with the max-Smoke bump) | 8/s -> 14/s (+75%) | **8/s -> 16/s (+100%)** (0.1 -> 0.12/point) |

Also fixed a latent display bug while in there: `AttributeConfig`'s Strength/Toughness/Smoke
`Bonus` text functions had their own **hardcoded copies** of the old numbers (e.g. Toughness's
tooltip literally had `0.985` baked into the format string, separate from `PerRank`; Smoke's
tooltip had `5` and `0.1` baked in, separate from `SmokeConfig.PerPoint`). Changing only the real
values would have left the in-game tooltips showing the old, wrong numbers. Updated both so they
stay in sync - worth remembering these three numbers live in two places each.

Not play-tested / not balanced against real combat feel - these are the requested target numbers,
not something verified in a fight yet. Play test before treating this as final.

Jay's reaction after seeing the numbers on paper: "they seem very high but for now its fine" -
not asking for a change yet, but flag this before shipping/announcing the balance pass. Likely
first thing to dial back if a playtest confirms it feels too strong.

## Asset/content survey (2026-09-17) - what's next backend-wise, and what needs real assets

### More backend candidates (beyond the already-deferred anti-cheat/infra list above)
- **No sound system exists anywhere.** Checked the whole place for `Sound` instances: there is
  exactly one (`StarterGui.Transition.ClapDoor`) in the entire game. No hit/block/parry SFX, no
  Smoke-cast SFX, no footsteps, no UI clicks, no level-up/reward stingers, nothing. Before any
  audio assets even exist, there's backend-adjacent work worth doing now: a small FX/Sound
  trigger service so every event that *should* make a sound (landed hits, casts, level-ups, shop
  buys, event start/end) already fires a named hook - so dropping in real audio later is just
  filling in asset ids, not re-wiring gameplay code.
- **Faction/Clan has no backend at all.** `PlayerStats.Faction` and `.Clan` are static defaults
  ("Sorcerer" / "Clanless") set once by `PlayerStatsBridge` - nothing anywhere ever changes them.
  `Player_Display.Handler` already branches its whole UI on Faction (Sorcerer/Human/Bounty
  Hunter/Criminal vs Curse), so the client is ready for a real faction system; there's just no
  server logic that assigns or changes one. Worth deciding if this is meant to be real
  gameplay (a Sorcerer vs Curse PvP split) or just flavor text. **Update:** Jay declined — not needed.
- The infra list from earlier today (anti-cheat, DataStore write queue, moderation, PvP ranking,
  MemoryStore matchmaking) is all still open and untouched. **Update:** built in the next entries —
  see the status notes on the "Deferred" list above.

### Everything that needs a real asset (grouped)

**VFX**
- All 16 Smoke moves (Split: SplitCut/Unravel; Gun: SmokeShot/ChargedBlast; Curse: Hex/Wither;
  Regeneration: Mend/Surge; Mushroom: SporeBurst/SporeTrap/Cling; Lizard: ScaleForm/TailSweep;
  Dinosaur: BeastForm/Stomp/Bite) currently render through two generic procedural helpers,
  `Kit.slash` and `Kit.burst`, tinted by the type's plain `Color3` - no real particle textures,
  beams, meshes, or trails. Confirmed by reading `SmokeMoves.Split` directly.
- Hell (the mud-plain dimension) and the Grudge Monument are "part-built" per PROJECT.md - no
  real environment art.

**Animations**
- Good news: weapon combat animations are actually complete. All 7 weapons (Katana, Sword, Axe,
  Dagger, Trident, SwordShield, Fist) have full M1/M2/AerialM1/AerialM2/Block/Equip/Unequip/
  RunningM1/ParryAttempt/Movement sets already in `ReplicatedStorage.Assets.Animations.Weapons`.
  Nothing to source there.
- Lizard and Dinosaur Smoke forms (skipped this round, per your earlier call) still have no real
  creature models or animations - they reskin the player's own rig.
- World NPCs (Street Informant, Shopkeeper, Trainer Goro, Diner/Gyoza Cook/Grave Keeper,
  Fortune Teller, Hospital Director) are grey R6 rigs in tinted clothes - no real outfits or idle
  animations.

**Sounds** (see backend note above - there's nowhere for these to plug in yet)
- Combat: hit/block/parry/posture-break, per weapon if you want variety.
- Smoke: a cast sound per move (16), silence/dry-out stingers.
- UI: level-up, shop buy, quest complete, notification pop, menu clicks.
- World: footsteps, ambient loops (city, Hell, rain), event start/end stingers, door travel.

**Icons**
- `ReplicatedStorage.World.Items` (every weapon/equipment/consumable) has zero `Icon` fields -
  the inventory, shop, and hotbar UIs have no item art to show at all right now, just text.
- Same gap for the 7 Smoke types / 16 moves if the Smoke UI is meant to show move icons.

**Cosmetics / equipment models**
- Every equipment item (GasMask, BoneMask, CleanerCoat, ScrapVest, SmokeCharm, GraveBell, and the
  new DevilMask) is a single re-sized/re-colored Part, by design (`Look = small part-built
  visual, no models yet`). Real accessory meshes would upgrade the whole equipment system at
  once, not just one item.

## Bag/Inventory K keybind + backend candidates pass (2026-09-17)

Jay's call: wire the real bag to K, skip Faction work (not needed) and sound (can't get assets
right now), do the rest of the deferred backend list. Applied live in Studio Edit mode - not
play-tested yet.

### Bag <-> K keybind
There were two competing inventory UIs: `WorldClient.BagClient` (the real one - reads
`InventoryJSON`/`EquippedJSON` off `InventoryService`, calls `Remotes.ItemAction`), bound to **B**;
and jjk's `Custom Inventory.playerManager.inventoryHandler` (a generic Tool-drag hotbar, not
backed by `Items`/`InventoryService` at all), bound to **K**. Moved `BagClient`'s open key from B
to K, and disabled Custom Inventory's own K-open behavior using its own built-in off switch
(`SETTINGS.INVENTORY_KEYBIND = nil`, documented in its own comment as the supported way to turn
it off) so the two don't fight over the same key. Nothing else about Custom Inventory was touched.

### Declined (per Jay)
- **Faction system** - confirmed nothing dynamically sets `Faction`/`Clan` anywhere in the
  codebase (grepped for assignments); nothing to disable, there was never anything running. Not
  building it.
- **Sound system** - skipped, no audio assets to hook up yet.
- **MemoryStoreService matchmaking** (from the original deferred list) - skipped for a different
  reason: there's no party/duel/matchmaking *feature* anywhere in the codebase for it to serve.
  Building the plumbing with no caller would be speculative infrastructure - revisit once there's
  an actual feature that needs live cross-server state.

### Built
- **`ServerScriptService.World.DataStoreRetry`** - small exponential-backoff wrapper
  (`DataStoreRetry.call(fn, ...)`, 4 attempts, doubling delay) for the one place in the codebase
  that made raw DataStore calls without ProfileService's own retry handling:
  `LeaderboardService`'s `SetAsync`/`GetSortedAsync`. Both now go through it.
- **Anti-cheat: reach check in `DamageLogic.Shared.Processed`.** Added `MAX_HIT_DISTANCE = 45`
  and an early return if attacker-target distance exceeds it, placed right after the CFrames are
  computed (still after the cheap Health/SafeZone/SpawnGrace exits from the earlier perf pass).
  Deliberately generous - this only catches blatant teleport/reach exploits, it's not range
  balance. First pass only; full movement-speed/velocity validation is still open.
- **`ServerScriptService.World.RankingService`** - PvP Elo rating. New `PlayerData.Rating`
  (default 1000), updated on every `WorldSignals.PlayerKilled` with a standard Elo formula
  (K=24), mirrored to a player attribute and added as a 4th `LeaderboardService` board (`Rating`).
  Reuses the same kill signal ProgressionService/WarService already listen to.
- **`ServerScriptService.World.ModerationService`** - persisted bans. DataStore `Moderation_v1`,
  keyed by UserId, record = `{ Reason, By, At, Until }` (`Until = nil` is permanent). Banned
  players are kicked right after joining (Roblox can't block a join before it happens without a
  proxy, so kick-on-join is the standard pattern) - Studio only warns, never kicks, matching
  `RemoteGuard`'s existing convention. Added Cmdr commands `ban <player> <reason?> <minutes?>` and
  `unban <userId>` in a new `Cmdr.Commands.Moderation` folder (auto-discovered by Cmdr's existing
  recursive `RegisterCommandsIn` - no registration wiring needed).
  **Not built:** player-facing reporting - there's no UI to send a report from yet, so a remote
  with no caller would be speculative. Also no mute system - this game uses default Roblox chat,
  there's no custom chat pipeline to enforce a mute flag against.

### Not play-tested yet
Before saving: press K in-game and confirm the real Bag opens (not the old Custom Inventory
window); get a PvP kill and check `Rating` moves on both players and shows on the leaderboard;
try `ban`/`unban` from Cmdr on an alt/friend if possible.

## Follow-up: closing gaps in today's own new systems (2026-09-17)

Jay said "keep building backend systems" without a specific target. Rather than start a new
player-facing feature blind (trading and parties/duels don't exist anywhere in the codebase - I
checked - and both carry real exploit risk, e.g. item duping, that deserve a design check-in
before building, not a guess), I closed two gaps in the systems built earlier today:

- **`DataMigrations.sanitize`** didn't know about the two PlayerData fields added today (`XP`,
  `Rating`) - a corrupted/NaN value in either would have slipped through the save-integrity pass.
  Added both (`XP` floors to 0, `Rating` floors to the 1000 default) alongside the existing
  Yen/BurialTags/AttributePoints guards.
- **`SelfTest`** didn't cover any of today's new services. Added three checks: `ProgressionConfig`
  level-curve math (0 XP = level 1, one level's cost = level 2), `ModerationService` ban/unban
  round-trip (bans a fake UserId, confirms it reads back, unbans, confirms it's gone), and a
  `RankingService` boot check (confirms the script loaded and created its `PlayerKilled` signal).
  Not covered: RankingService's actual Elo math - it's a plain `Script` with no public API to call
  into from a test, and re-deriving the same formula inline would just be testing itself, not the
  real code. Converting it to a ModuleScript+Script pair purely to make it testable felt like the
  wrong trade for now.

Next unclaimed backend items, for when there's a specific target: player trading, party/duel
system (which is also the blocker for the MemoryStoreService matchmaking item from earlier),
player-facing reports, mute/chat moderation. All need a design decision before building, not just
more code.

## Inventory unification (2026-09-17, corrected)

Jay clarified: the earlier "just move the keybind" fix wasn't what he wanted - he wanted ONE real
inventory system total, not two systems with one of them silenced. Rebuilt properly.

**Decision (Jay's call):** keep the existing Custom Inventory look/window (the one players
already know), rewired to real data - not BagClient's simpler list UI.

### What was actually wrong
`Custom Inventory` (K) was a generic Backpack/`Tool` drag-and-drop hotbar system
(`playerManager` + `inventoryHandler`, ~670 lines total) with its own data model - numbered
hotbar slots holding arbitrary `Tool` instances. It had **no connection at all** to
`ReplicatedStorage.World.Items` or `InventoryService` - completely different concepts (e.g.
`GasMask` armor isn't a `Tool` and was never going to show up in it). `BagClient` (B) was the one
actually wired to real data. Rebinding B to K would have just swapped which of the two
mismatched systems was visible, not merged them - correctly called out.

### What was built
New **`StarterGui.Custom Inventory.InventoryClient`** (LocalScript) - reuses the existing GUI
(the `hotBar` strip, the `Inventory` grid window, its `SearchBox`, and the `toolSlot` template
clone-per-item) but is driven entirely by the real backend: player attrs `InventoryJSON` /
`EquippedJSON` (`InventoryService`), actions through `Remotes.ItemAction`, same pattern
`BagClient` already proved out. Bound to **K**. Search box filters the grid by item name.
`hotBar` now shows the up-to-4 currently-equipped items (Weapon/Head/Body/Charm) as a persistent
strip instead of a draggable numbered Tool bar.

**Retired (disabled, not deleted - reversible):**
- `Custom Inventory.playerManager` (and its child `inventoryHandler`) - the old Tool-drag system.
  Its `StarterGui:SetCoreGuiEnabled(Backpack, false)` call is preserved in the new script.
- `WorldClient.BagClient` - the separate B-bound window. No longer needed; its logic pattern
  (decode JSON attrs, `Items.get`, `ItemAction:InvokeServer`) is what `InventoryClient` reused.

Now there is exactly one inventory system, bound to K, backed by real data.

### Known limitation carried over
Slots show item name text, not icon art - `Items` still has no `Icon` field (logged earlier in
the asset survey). `toolIcon` is present in the template and wired to hide/show correctly, so
dropping in real icons later is a one-line change (`frame.toolIcon.Visible = true` +
`frame.toolIcon.Image = def.Icon`), not another rewrite.

### Not play-tested yet
Press K, confirm the grid shows real inventory items (not the old Tool hotbar), equip/unequip a
weapon and equipment piece, confirm the hotBar strip updates, try the search box, confirm the old
Custom Inventory drag behavior is gone and nothing else broke (e.g. default Backpack GUI should
stay hidden).

## Party + Clan + Smoke moves HUD (2026-09-17)

Jay's instructions were terse ("skip 1, create a party system, skip 2, skip 4, add 5, skip 6") -
my read: skip trading (1), build the *party* half of item 2 but not dueling/matchmaking (hence
"skip 2" right after - the party/duel item was bundled, he wants the party half only), skip mute
(4), build Clans (5), skip the weapon-shop check (6), plus the explicit new ask: a Smoke
moves+cooldowns HUD, bottom-right. **Flagging this interpretation in case it's wrong** - easy to
adjust either way.

### PartyService (`ServerScriptService.World.PartyService`)
In-memory only - parties don't need to survive a server restart, and there's no cross-server
matchmaking feature yet for them to need to. No UI (none was requested for this one) - chat
commands: `!party invite <player>`, `!party accept <leader>`, `!party leave`,
`!party kick <player>` (leader-only), `!party who`. Max 4 members, invites expire after 2 minutes,
leadership auto-transfers if the leader leaves. Mirrors membership to player attributes
`PartyLeader` / `PartyMembersJSON` so a future UI can read it without touching this file.
**Deliberately doesn't change combat rules** - party members can still hit each other; there's no
friendly-fire system anywhere in this codebase yet to hook a "don't hit your party" rule into.

### ClanService (`ServerScriptService.World.ClanService`)
Persistent - `PlayerData.Clan` (new field) + a DataStore `Clans_v1` roster (key = clan name
lowercased -> `{ Owner, Members, CreatedAt }`). No new UI needed - `PlayerStats.Clan` already
renders on WorldInfo's HUD ("FirstName Clan", "Clanless" if none), so joining a clan is
immediately visible. Chat commands: `!clan create <name>`, `!clan invite <player>`,
`!clan accept <name>`, `!clan leave`, `!clan kick <player>` (owner-only),
`!clan disband` (owner-only), `!clan who`. Ownership passes to the next member if the owner
leaves; the clan is deleted from the DataStore if the last member leaves.
**Known limitation:** clan creation is a plain Get-then-Set (not a DataStore transaction), so two
players creating the same name at the exact same moment could both succeed - noted in the script
comments. Matches this codebase's existing DataStore rigor elsewhere (e.g. `LeaderboardService`
does the same); would need `UpdateAsync` to be airtight, which felt like overkill for a first pass.

### SmokeMovesHud (`StarterPlayer.StarterPlayerScripts.WorldClient.SmokeMovesHud`)
Bottom-right list of the player's current Smoke type's moves (2-3, from
`ReplicatedStorage.World.SmokeTypes`), each showing its name, Smoke cost, and a live cooldown
countdown. Reuses BagClient's exact color/font scheme so it looks native. No new remote needed -
`SmokeMoveService` already stamps a `ReadyAt` attribute on each move's Tool when cast, and already
sets the player's `SmokeType` attribute; this HUD just reads both. Rebuilds when `SmokeType`
changes or the character respawns; cooldown text updates every frame via `RunService.Heartbeat`.
Known gap carried over: no move icons (same `Items`-has-no-`Icon` gap as the inventory) - each
slot uses a colored swatch (the Smoke type's own color) instead.

### Not play-tested yet
Try `!party invite`/`!party accept` between two accounts (or Studio's multi-player test), same for
`!clan create`/`!clan invite`/`!clan accept`, confirm WorldInfo's HUD shows the real clan name
after joining, and confirm the Smoke moves HUD appears bottom-right with counting-down cooldowns
after casting a move.

## Smoke move keybinds (Z/X/C) + Slide moved to Control (2026-09-17)

Jay: bind Smoke moves to Z/X/C directly, move Slide off C onto Control. Applied live in Studio
Edit mode to **both** connected places (Map + Combat and Zogan's Academy Grounds - confirmed both
were byte-identical on the touched files before editing, so the same two edits were mirrored to
both, then re-verified identical after).

- **`ReplicatedStorage.Packages.Bind.Keybinds.Config`**: `Slide` moved from `` `C` `` to
  `` `LeftControl` ``, freeing C for the third Smoke move.
- **`StarterPlayer...WorldClient.SmokeMovesClient`**: move 1/2/3 of the player's current Smoke
  type (ordered list in `SmokeTypes.Types[id].Moves`) are now directly keybound to Z/X/C -
  pressing the key finds the matching move Tool (in Backpack or already-equipped), equips it if
  needed via `Humanoid:EquipTool`, and fires `SmokeCast` the same way a manual click always did.
  Types with only 2 moves (most of them) just leave C unused. Tool-click casting still works too,
  unchanged.
- **Known interaction, not a bug**: `LeftControl` was already used as a held modifier elsewhere
  (`InputController.Combat.M1` checks `IsKeyDown(LeftControl)` to turn an M1 click into an
  Uppercut). Holding Ctrl now both arms that modifier and fires Slide at the same time - flagged
  for Jay to feel out in a playtest, not changed further without his call.
- Not play-tested yet in either place.

## Zogan's Academy Grounds brought up to parity with Map + Combat (2026-09-17)

Jay: while touching Zogan's for the keybind change above, asked to verify the *entire* set of
2026-09-17 backend/system work (everything from "Backend hardening" through "Party + Clan + Smoke
moves HUD" above) actually exists and matches in Zogan's too, since it's a separate place that
had drifted. Audited first (read-only, both places), found Zogan's was stuck at roughly the
2026-09-16 end-of-day state, then ported everything on Jay's "port everything" call. Map + Combat
was the read-only source of truth throughout; nothing there was touched by this pass.

### Audit findings (before porting)
Missing entirely in Zogan's: `DataStoreRetry`, `RankingService`, `ModerationService`,
`PartyService`, `ClanService`, `ProgressionConfig`/`ProgressionService`, `SmokeMovesHud`,
`Items.Defs.DevilMask` + its `HellService` grant. Present but stale: the old
two-competing-inventories bug was back (K-keybind never disabled on the old Custom Inventory,
`BagClient` still live on B), `DamageLogic` had neither the perf reorder nor the
`MAX_HIT_DISTANCE` anti-cheat check, `SmokeService`/`SmokeMoveService` still ran the unbounded
startup loop, `EconomyService` had no loot table (`SupplyBundle` was Yen-only), and
`AttributeConfig`/`SmokeConfig` were still on the old pre-balance-pass numbers (including the old
hardcoded tooltip text).

### Ported / fixed
- **New, byte-identical to Map + Combat**: `DataStoreRetry`, `RankingService`, `ModerationService`
  (+ Cmdr `ban`/`banServer`/`unban`/`unbanServer` under `Cmdr.Commands.Moderation`),
  `PartyService`, `ClanService`, `ProgressionConfig` + `ProgressionService`, `SmokeMovesHud`,
  `Items.Defs.DevilMask` + the `HellService.recordEntrance` grant.
- **`ReplicatedStorage.PlayerData`**: added the `XP`, `Rating`, `Clan` fields (type + defaults) so
  the new systems above have somewhere to persist to. ProfileService's `Reconcile()` back-fills
  existing saves automatically - no manual migration needed.
- **Brought current in place**: `DamageLogic` (guard reorder + `MAX_HIT_DISTANCE = 45`),
  `SmokeService`/`SmokeMoveService` (bounded startup loop), `EconomyService` (6-entry loot table
  wired to `SupplyBundle`), `AttributeConfig`/`SmokeConfig` (tuned balance numbers + matching
  tooltip text), `LeaderboardService` (now uses `DataStoreRetry`, added the 4th `Rating` board),
  `DataMigrations` (added `XP`/`Rating` sanitize guards), `SelfTest` (added the
  ProgressionConfig/ModerationService/RankingService checks).
- **Inventory unification, mirrored**: new `StarterGui.Custom Inventory.InventoryClient`
  (K-bound, real `InventoryJSON`/`EquippedJSON` data) enabled; old `Custom Inventory.playerManager`
  had its K-open switched off the same way (`inventoryHandler.SETTINGS.INVENTORY_KEYBIND = nil`);
  `WorldClient.BagClient` retired to match Map + Combat's disabled state.
- Every dependency these systems needed (`WorldSignals`, `HellService`, `WarService`,
  `PitService`, `ReplicatedStorage.PlayerStats`, Cmdr's command-folder structure) already existed
  in Zogan's, so nothing had to be skipped as not-applicable.

### Not play-tested yet
Same checklist as the original Map + Combat pass, run once per place: press K (only the real Bag
should open), get a PvP kill (Rating/XP/leveling all move), try `!party`/`!clan` commands, try
Cmdr `ban`/`unban`, run the Cmdr `selftest` command. Worth spot-confirming on an existing (not
fresh) test save that the new `XP`/`Rating`/`Clan` fields actually appear after a rejoin.

## Play-mode smoke test + the two remaining deferred backend items (2026-09-17)

Confirmed persistence needs no fix: both places share the same Roblox universe (`game.GameId` =
10766477296 for both, confirmed via `execute_luau`), so `DataStoreService` is already shared
automatically - it's universe-scoped, not place-scoped. Loaded the same real test account
(`cooljayzfalt`) in Play mode in both places back-to-back and got byte-identical `PlayerData`
(Yen, Inventory, Quests, AttributePoints, the works) both times. The two places also already used
the same ProfileService store name (`PlayerData_v1`) and key format (`Player_<UserId>`), which is
the only thing that could have broken this - they didn't, so nothing needed changing. One real
gotcha found in the process: **never run Play mode in both places at once** - they fight over the
same account's ProfileService session lock and you get a spurious empty profile / "no profile" in
whichever session loses, which looks exactly like a data-loss bug but isn't one.

Ran the Cmdr `selftest` in both places after the port from the previous entry: **Zogan's Academy
Grounds: 18 passed, 0 failed.** Map + Combat's first run showed 3 fails (`player data loaded`,
`player attrs mirrored`, `quest debug hook`) - all from the profile-lock collision above (Zogan's
Play session was still running when Map + Combat's started); stopping Zogan's and isolating Map +
Combat should clear it (see "Known issue" below - a stuck Studio window prevented re-confirming
this run live, but the failure signature matches the lock-collision exactly and none of it touches
code this session touched).

### Two remaining deferred backend items, closed out
Both applied to **Map + Combat first, then mirrored identically to Zogan's Academy Grounds** (kept
at parity all session).

1. **Movement-speed anti-cheat**, `ServerStorage.Packages.DamageLogic` (`Shared.Processed`),
   alongside the existing `MAX_HIT_DISTANCE` reach check from earlier today. Tracks each attacking
   player's last hit position + server time (`lastAttackSample`, cleaned up on
   `Players.PlayerRemoving`) and rejects a hit (soft `return false`, same style as the reach
   check - no kick, no strike) only when the implied speed between two samples exceeds
   `MAX_IMPLIED_SPEED = 150` studs/sec **and** the samples are at least
   `MIN_SPEED_CHECK_INTERVAL = 0.5`s apart. That second condition is the load-bearing one: this
   game has 50+ map scripts (`Workspace.Map...Teleport*.ProximityPrompt.Script`) that instantly
   set `HumanoidRootPart.CFrame` for interior/exterior transitions, plus Smoke Doors and Hell
   entrances - all of those look identical to a hack teleport in a two-sample speed check (huge
   distance, ~zero elapsed time). Rather than touch 50+ unrelated map scripts to tag every
   legitimate teleport, the check only evaluates samples spaced out enough that no single teleport
   could explain the gap - only genuinely sustained fast movement can. Checked every real way to
   move fast first (Sprint tops out at 40.5 WalkSpeed, Slide is 65 for 0.45s, Dodge's dash burst is
   40 for well under a second, Smoke forms/Lizard/Dinosaur don't touch speed at all) - 150 studs/
   sec over a minimum half-second is >2x the fastest legitimate value with real headroom. Player-
   only (`AttackerPlayer`), never runs against NPC/AI attackers. Same "deliberately generous, not
   balance" philosophy as the reach check.
2. **DataStore retry coverage.** `ServerScriptService.World.DataStoreRetry` (built earlier today)
   was already used by `LeaderboardService` but `ModerationService` (`Moderation_v1`) and
   `ClanService` (`Clans_v1`) still called `store:GetAsync`/`SetAsync`/`RemoveAsync` directly
   through bare `pcall`. Routed all of it through `DataStoreRetry.call(...)` instead - same
   exponential-backoff wrapper, no new abstraction. Deliberately did **not** build a generic
   write-queue/coalescing system on top of this: `ProfileService` already retries player saves
   internally, `LeaderboardService` is already throttled to once/5min, and `ClanService`'s own
   comment already flags its Get-then-Set race as "acceptable for a first pass, not for a bank" -
   a speculative queue with no actual load problem to justify it would be over-engineering past
   that same bar. Wrapping the two direct-call sites in the retry helper that already exists is the
   right-sized version of this deferred item.

### Test results
- **Zogan's Academy Grounds**: Play mode, `selftest` -> 18 passed / 0 failed / 0 warnings, console
  clean (no new errors/warnings from either change, `ModerationService ban/unban round-trip` self-
  test exercises the new `DataStoreRetry`-wrapped ban/unban path directly and passed).
- **Map + Combat**: code confirmed identical to Zogan's (read back after editing), but this
  session's Studio window got stuck mid Play-mode toggle (`start_stop_play` kept returning "hasn't
  finished yet" and the viewport froze on a static frame - `ServerUptime` never ticked, screenshot
  didn't change across multiple captures) and never recovered enough to re-run `selftest` live
  there. This looks like a Studio-side hang, not a code issue - Edit-mode operations kept working
  the whole time (read the edited script back successfully after the hang started). **Known issue:
  restart the Map + Combat Studio window and re-run Cmdr `selftest` there to get a clean live
  confirmation** - the code is identical to what already passed cleanly in Zogan's, but it hasn't
  been independently exercised live in Map + Combat this session. **Update:** resolved — the
  hang cleared and Map + Combat ran `selftest` 18/0 live during the atmosphere pass the same day.
- ClanService's `!clan create`/etc. chat commands specifically weren't triggered live in either
  place (no easy way to simulate a real chat message through this session's tools) - confidence
  there comes from the module-load self-test check passing (no syntax/require errors) plus the
  wrapping pattern being identical to the already-proven `LeaderboardService`/`ModerationService`
  usage, not from an end-to-end command test. Worth an actual `!clan create` in a real playtest.

## Slide PvP patch: must be moving to Slide, Stop instead if standing still (2026-09-17)

Jay: pressing Slide from a dead stop should give a "Stop" in place, not a slide - player has to
have been moving for a little first. Applied to **both** places (Map + Combat first, confirmed via
read-back, then mirrored to Zogan's Academy Grounds and verified live there since Map + Combat's
Studio window was still stuck on the Play-mode hang from the previous entry).

- **`ServerScriptService.Services.Player.ActionManager`**: `Manager.new` now tracks each
  character's `MovingSince` (a timestamp, set the moment `Humanoid.MoveDirection.Magnitude` goes
  from 0 to > 0 via `GetPropertyChangedSignal`, cleared back to `nil` the moment it returns to 0;
  connection lives in the same `Trove` the manager already cleans up on character removal). New
  `Manager:HasMovedFor(Duration)` returns whether the character has been continuously moving for
  at least `Duration` seconds. The `UnPackets.Slide` handler now calls `HasMovedFor(0.15)` before
  deciding what to activate: `Slide` if true, `Stop` if false. 0.15s is deliberately short - "a
  little", not a real windup - just enough that a literal standing-start tap can't get the slide.
- **New `ServerScriptService.Services.Player.ActionManager.Movement.Stop`** (sibling module to
  `Slide`, same `Activate`/`Conditions`/`Class` shape every action module in this folder uses):
  zeroes the character's horizontal `AssemblyLinearVelocity` (kills any residual momentum) and
  holds a brief `Stopping` state for 0.15s. No dash, no `LinearVelocity` burst, no i-frames, no
  hitbox change, no VFX - deliberately inert, so standing-still Slide-spam can't be used as a free
  reposition/duck in a fight. Gates on the same `InAir`/stun-state `Conditions` Slide already used,
  so behavior while airborne or stunned is unchanged from before this patch (Slide already did
  nothing there; now Stop also does nothing there, for the same reason).
- **Verified live** in Zogan's Academy Grounds (Map + Combat's Studio window needs the restart
  flagged in the previous entry before it can be re-confirmed there; code is identical, applied and
  read back the same way as every other mirrored change this session):
  - Character standing still (`MoveDirection` confirmed `(0,0)`), pressed the real Slide key
    (`LeftControl`) via simulated input -> `HumanoidRootPart.Position` measured before and ~0.7s
    after: **zero displacement** (identical position to the studs). Stop fired, no slide.
  - Held `W` for 400ms (well past the 0.15s floor), then pressed `LeftControl` while still holding
    `W` -> measured **~32.7 studs of forward displacement** over the same window - a real Slide
    burst. Confirms the gate isn't blocking legitimate slides, only standing-start ones.
  - Console clean both times, no new errors/warnings. Cmdr `selftest` re-run after: still 18
    passed / 0 failed / 0 warnings.
- Not tested: sliding immediately after a very brief tap of a movement key (e.g. 0.05s) - expected
  to fall through to Stop per the 0.15s floor, matches the "moving for a little" ask, but wasn't
  independently verified as its own case (the standing-still test above is functionally the same
  code path, just at the 0s extreme).

## Economy/quest depth pass (2026-09-17)

Jay's call: "depth on existing systems" - richer content within what's already there, not new
mechanics. Applied to both places (`ReplicatedStorage.World.Shops` and `.Quests` are byte-identical
shared data files across both places, confirmed via read-back before editing - both self-tests
report the same "32 quest definitions" count, so this isn't a per-place mirror exercise, it's
literally the same two files copied into each place's ReplicatedStorage).

Skipped Play-mode verification for this pass - three other parallel forks (atmosphere, screen
effects, NPC AI) were actively using both Studio instances' Play mode at the same time, and this
session already confirmed running Play in both places at once causes spurious ProfileService
lock-collision failures. Both changes are pure data-table edits (a price number, a rewards table)
with no logic changes, verified correct via read-back in both places instead; low enough risk that
skipping a live self-test run felt like the right call rather than fighting the other forks for
Play mode.

### Fixed
- **`Shops.Diner.Items.GyozaPlate` price mismatch.** The exact same buff (`Id = "Meal"`, +25 max HP,
  180s, identical `Desc`) was priced 110 Yen at the Diner but 120 Yen at the dedicated `Gyoza` stand
  selling the literal same plate. Aligned the Diner's price up to 120 to match the specialty vendor.
- **`Quests.Defs.Rematch` (Academy) reward.** Was the worst Yen-per-cooldown-second of any repeatable
  in the game (150 Yen / 1800s = 0.083/s) despite being the only repeatable that risks a full boss
  fight (420 HP Proctor with weapon AI, 240s time limit) instead of a farm/grind. Every other
  boss/danger-tier repeatable (`SpecimenSweep`, `NightWatch`, `HoleCleanup`) already pays Tags
  alongside Yen; `Rematch` paid none. Bumped `Rewards` from `{ Yen = 150, Points = 1 }` to
  `{ Yen = 190, Tags = 1, Points = 1 }` - brings it in line with that established pattern without
  making it the best-paying repeatable either.

### Investigated, no change made
- Scanned every shop's price-per-effect and every quest chain's `Requires` links for other
  inconsistencies or broken references - found none. All `Requires` ids resolve to real quests, no
  orphaned chains besides the one flagged below.
- Checked whether Tags (burial tags) run out of things worth buying at higher levels
  (`GraveKeeper`'s shop is the only Tags sink: cash-in, one buff, attribute points, one charm). This
  looks like a real future gap for a high-Tags player, but fixing it means inventing a new item,
  which the "depth, not new mechanics" scope explicitly rules out - flagging for a design call
  instead of guessing at a new item.

### Flagged, not built - needs Jay's call
- **The `Adv4` quest chain (Academy) has no `Next` and nothing continues after it.** Its closing line
  ("The Academy will remember your name.") reads like a deliberate arc conclusion, not an oversight -
  and Jay's own account is *currently playing through Adv4 right now* (`Active = Adv4, Step 2` in the
  live save). Whether to add an `Adv5`+ continuation is a real story/plot decision (what comes after
  graduation-tier content - a new antagonist, a new arc, a title/rank change) per this project's
  standing rule to ask before inventing narrative direction, not something to bolt on unasked.
- **Zogan's Academy Grounds has zero shops staffed** (self-test: "0 shops staffed here" there vs "10"
  in Map + Combat). Every entry in `Shops.lua` is Hole-themed (Grave Keeper, Diner, Arms Dealer,
  alley stalls, etc.) - none map to Academy NPCs (Headmaster, Kessa, Bram, Nurse). Academy students
  currently have nowhere to spend Yen at all. Fixing this needs either new Academy-flavored shop
  entries (new items - out of this pass's scope) or physically staffing an existing shop on an
  Academy NPC (touches `workspace` NPC instances, which is the parallel NPC-AI fork's lane this
  session, not mine) - noted for whoever picks up Academy economy work next, not built here.

## NPC AI depth pass (2026-09-17)

Jay's call: "depth on existing systems" for backend, applied to the combat NPC AI specifically.
The existing framework (`ServerScriptService.Services.AI.AI.NPCController`/`NPCBehaviors`/
`NPCMovement`/`NPCActions`) turned out to already be quite deep - threat tables with decay and
target-switching, weighted strafe movement, hit/miss-streak-scaled attack cooldowns and
feint/dodge/parry chances, sprint and jump logic, agro claim/release with fade distance, position
prediction for aim-leading. Read all four scripts plus `WorldConfig.BlueNight.ZombieAI` and
`ReplicatedStorage.Config.NPC` before touching anything, to find genuine gaps rather than
duplicating what already exists. No idle-dialogue/"bark" infrastructure exists for combat NPCs
(only tagged interactive NPCs like shopkeepers have dialogue) - building that from scratch was
explicitly out of scope, so idle/patrol variety was skipped rather than half-built.

Applied to **both** places (Map + Combat first, confirmed via read-back byte-identical to Zogan's
before editing, then mirrored). Live self-test (`selftest`) verified clean in Zogan's after (18
passed / 0 failed / 0 warnings, no new console errors) - Map + Combat's Studio window was stuck on
the same Play-mode hang flagged earlier this session (`start_stop_play` still returning "hasn't
finished yet"), so that side rests on the code read-back match plus Zogan's live result, same as
the Slide-patch entry above. Coordinated with three other parallel forks (atmosphere, screen
effects, economy) sharing both Studio instances this session - checked `get_studio_state` before
every Play-mode call and kept Play sessions short to avoid blocking them.

### Built
1. **Low-health self-preservation** (`NPCBehaviors.GetDesperationFactor`, new). Returns `1` (no
   change) above 35% health; below that, scales up to `1.6x` at 0 health. Multiplies into
   `ShouldDodge`'s and `ShouldHoldBlock`'s existing chance calculations (capped at 0.9 / 0.85 so a
   dying NPC still isn't unhittable), so a wounded enemy gets visibly more careful - more dodges,
   more blocks - without any new state, mechanic, or behavior branch. Zero effect on healthy NPCs.
2. **Shambling movement for simple mobs** (`NPCController.Local.HandleM1Only`, the `M1Only` chase-
   and-punch path shared by Blue Night/Cleanup Day zombies, `Mobs.lua`, and `QuestService`'s zombie
   spawns - all clone `WorldConfig.BlueNight.ZombieAI`). Their straight-line beeline to the target
   now has a small sinusoidal side-to-side drift (`math.sin(tick() * 2.2 + npc.UpdateOffset * tau)
   * 0.35`, blended into the move direction) instead of walking a perfect line - reuses the
   per-NPC `UpdateOffset` phase value that already existed for stagger, so no new per-NPC state.
   Purely cosmetic - doesn't touch `AttackCooldown`, damage, or the state machine. Gated on
   `Config.M1Only` only, so the full combat AI (weapon-carrying humanoid enemies) is untouched.

### Not built
- Idle/patrol "look around" or ambient bark variety - would require new dialogue/bark
  infrastructure for combat NPCs that doesn't exist yet; out of scope for a depth pass on the
  existing framework.
- No change to `ShouldParry`/`ShouldEvasive`/`ShouldUppercut`/`ShouldFeint` or any attack-side
  behavior - kept the change surface to the two self-preservation (defensive) behaviors so this
  stays "richer," not "rebalanced."

## Environmental atmosphere pass (2026-09-17)

Jay's call: "dive fully into" atmosphere/visuals. This pass covers passive world atmosphere only -
weather/lighting/ambient life, not combat/UI screen effects, economy, or NPC AI (three sibling
forks this session covered those lanes on the same two Studio instances). No new art assets used
anywhere - the whole project has none - everything here is built from Roblox's built-in
Lighting/Atmosphere services and generic `rbxasset://` particle textures, matching how every other
effect in this game (Smoke moves, rain, petals) is already built.

Read `StarterPlayer...WorldClient.AtmosphereClient`, `ServerScriptService.World.AtmosphereService`,
and `ReplicatedStorage.World.WorldConfig` first: per-zone mood (`Config.ZoneMood` by `DangerLevel`),
ToxicRain/BlueNight/GhostNight/Hell event overrides, and an Academy-specific warm daytime mood with
falling petals already existed and are solid - didn't duplicate or fight any of it. The one real gap
was that `ClockTime`/`Ambient`/`OutdoorAmbient`/`Brightness` were all static snapshots taken once at
script start - no passive time passage, and no ambient "life" particle layer outside of rain/petals.

### Built, in `StarterPlayer...WorldClient.AtmosphereClient` only
1. **Passive day-night cycle.** `computeCycle()` drives `Lighting.ClockTime` through a smooth
   30-minute real-time sine wave, place-specific range: Map + Combat ("Hole") swings dusk->night
   (16->21, matching the slum's original static 17.9 baseline so the "normal" look didn't change,
   just started drifting around it); Zogan's Academy swings within a bright school-day band (9->15).
   The same cycle phase (`nightAmount`, 0..1) blends `OutdoorAmbient`/`Ambient`/`Brightness` from
   their captured baselines toward a darker, cooler variant at the "night" end via two small helper
   functions (`scaleColor`/`tintColor` - Color3 has no built-in scalar multiply in Luau, so these are
   hand-rolled) - subtle by design (up to -45%/-40%/-25% respectively at the far end), so the
   deliberately gritty/dark baseline mood isn't fought, just given quiet variation over a session.
   A `task.spawn` loop ticks every 2s (not `Heartbeat` - this is a slow drift, doesn't need per-frame
   precision) and only advances/reapplies while no Hell/BlueNight/GhostNight override is active;
   those event overrides (unchanged) still take full priority and the cycle simply pauses under them
   rather than fighting for control of `Lighting.ClockTime`.
2. **Ambient drifting ash/wisps.** A new low-rate (`Rate = 5`, vs. rain's 900 and petals' 40 -
   deliberately sparse, ambient texture not weather) camera-follow `ParticleEmitter` using the same
   `smoke_main.dds` texture already used for rain mist, tinted pale grey, long lifetime (10-16s) so
   it reads as a constant faint drift rather than a burst. Skipped entirely in Academy - the existing
   petal effect already fills the "ambient life" role there and stacking both would look cluttered.

### Verified live (both places, Play mode + `screen_capture` + console + Cmdr `selftest`)
Coordinated Play-mode access with the three sibling forks sharing both Studio instances - checked
`get_studio_state` before every `start_stop_play` call, did edit-mode work and retried rather than
interrupting an active session, never ran Play in both places at once (this session's own earlier
finding: concurrent Play sessions fight over one shared ProfileService-locked account and produce
spurious failures).
- **Zogan's Academy Grounds**: screenshot shows a bright, sunlit field with a faint drifting
  particle visible - matches the intended school-day mood. Console clean, no new errors/warnings.
  Cmdr `selftest`: **18 passed, 0 failed, 0 warnings**.
- **Map + Combat**: screenshot shows the intended dark, moody slum lighting (zone mood already
  applies heavy density/haze/negative-saturation here) with no rendering glitches. This session's
  earlier Studio-window Play-mode hang (flagged in two prior entries above) had cleared by the time
  this pass tested it - `start_stop_play` worked normally. Console clean. Cmdr `selftest`: **18
  passed, 0 failed, 0 warnings** (34 modules / 10 shops staffed here, matching this place's known
  larger scope vs. Zogan's 32 modules / 0 shops).
- Both `AtmosphereClient` copies confirmed byte-identical via read-back after editing.

### Not done
- No per-danger-zone variation to the new day-night/ash layer beyond what `ZoneMood` already
  provides - the cycle and ash particles are place-level, not zone-level, since the existing
  zone-mood system already handles danger-tier atmosphere and doubling up on that axis felt like
  scope creep past "add passive life," not a request to retune zone danger feel.
- No sound/audio tied to any of this - this project has no audio assets or sound-trigger
  infrastructure at all yet (a pre-existing, previously-logged gap), out of reach for a visual-only
  pass with no new assets.

## Combat & UI screen-effects pass (2026-09-17)

Jay's call: "dive fully into" combat/UI screen effects. This pass covers reactive screen/camera
feedback only - not environmental atmosphere, economy, or NPC AI (three sibling forks this session
covered those on the same two Studio instances). No new art assets - everything built from Frames,
UIGradient, and TweenService, matching how `HitWeight` (the existing hit-feel system) already works.

Read `StarterPlayer...StateController.Misc.HitWeight` and the `Setting_ScreenShake` attribute
convention first, plus `Packets.Replicate`'s dispatch mechanism (`StateController`'s
`OnClientEvent` handler resolves `Category/Effect` to a `StateController.<Category>.<Effect>`
ModuleScript and calls it with `(Character, Parameters)` - confirmed `FireClient` works for a
single targeted player via the existing `Feint`/`Feint M2` pattern, not just broadcast).

### Built
1. **Damage-taken flash + low-health pulse**, new `StarterPlayer...WorldClient.ScreenEffectsClient`
   (self-initializing LocalScript, not routed through `Packets.Replicate` - pure client-local
   reaction to the player's own `Humanoid.HealthChanged`, works for any damage source, no server
   round-trip needed). A 4-edge vignette (`Frame` + `UIGradient`, transparency fading edge-to-center)
   tinted red. One `RunService.Heartbeat` loop drives both effects off a single combined alpha so
   they never fight: `flashAlpha` spikes on any health decrease (scaled by damage as a % of max HP,
   decays at `FLASH_DECAY_PER_SEC = 3.2`) and `pulseAlpha` sine-oscillates between `0.12` and `0.4`
   alpha on a 1.4s period whenever health is below 25%, silent otherwise. Neither respects
   `Setting_ScreenShake` (pure color/vignette, not shake/FOV, per the same convention `HitWeight`
   already draws that line on).
2. **Kill-confirm flourish**: new `ServerScriptService.World.KillConfirmEffect` (tiny listener,
   same `signal()`-helper idiom `RankingService`/`ProgressionService` already use) fires
   `Packets.Replicate:FireClient(killer, `Misc`, `KillConfirm`, {})` off the existing
   `WorldSignals.PlayerKilled` - no new trust surface, same signal three other systems already
   consume. New `StateController.Misc.KillConfirm` (client) plays a white flash + a two-piece
   X-shaped "hitmarker" pop, both tweened out and destroyed within 0.3s - snappy, no slow-mo,
   doesn't disrupt competitive feel.
3. **Skipped**: the optional Smoke-cast screen tint (per-type color flash on casting a move) -
   the three required effects above were the real ask; adding a fourth cosmetic layer felt like
   scope creep for a first pass. Easy to add later reusing `SmokeTypes.Types[id].Color`.

### Verified live (both places)
Coordinated Play-mode access with three sibling forks sharing both Studio instances - checked
`get_studio_state` before every `start_stop_play` call, backed off and did Edit-mode work instead
when another session had Play open rather than interrupting it (lost one `KillConfirmEffect`/
`KillConfirm` write in Zogan's this way mid-session - a Play-mode transition appears to have
discarded an Edit-mode write that raced it; caught by post-write read-back, rewritten once Zogan's
was free again, confirmed correct on a second read-back).
- **Damage flash**: forced 30% max-HP damage via `Humanoid.Health -=`, sampled the vignette's
  `BackgroundTransparency` every 0.05s. **Zogan's**: 0.554 -> 0.712 -> 0.878 -> 1.000 (peaks visible
  then fully decays in ~0.2s). **Map + Combat**: 0.547 -> 0.746 -> 0.941 -> 1.000, same shape.
  Matches the tuned decay rate exactly.
- **Low-health pulse** (Zogan's only, health forced to 15%): 20 samples over 2s show a clean
  sinusoid between 0.601 and 0.878 transparency (= alpha 0.399 down to 0.122, matching the
  configured `0.12`-`0.4` range) on a ~1.4s period. Healed back to full: 5 samples all `1.000` -
  pulse stops cleanly, no lingering flicker.
- **Kill-confirm**: not end-to-end fired - the only trigger path (`WorldSignals.PlayerKilled`,
  a protected `BindableEvent`) can't be invoked from injected test code in this session (same
  "additional Capabilities" sandbox restriction that's blocked `require`/`:Fire()` on protected
  instances all session), and there's only one test account in Studio Play to be both killer and
  victim. Confidence instead comes from: the server half uses the exact same `signal()` idiom
  already proven live by `RankingService` (which really does update `Rating` on real kills), and
  the client half uses the exact same `Packets.Replicate:FireClient` + `StateController.Misc.*`
  dispatch path `HitWeight` already uses successfully in production. Worth an actual PvP kill in a
  real playtest to confirm the visual, but the wiring isn't a guess.
- Console clean in both places across all tests. Cmdr `selftest`: **18 passed / 0 failed / 0
  warnings** in both (Map + Combat's Studio-window Play-mode hang from earlier in this session
  cleared by the time this pass ran it - `start_stop_play` and both `Client`/`Server` bridges
  worked normally here, unlike the two entries above that had to fall back to Zogan's-only
  verification).
- All three new scripts confirmed byte-identical between places via read-back after mirroring.

### Not done
- No screen-shake/FOV component added to any of these three effects - deliberately kept them as
  pure color/vignette/flash, distinct from `HitWeight`'s shake layer, so `Setting_ScreenShake`
  stays a meaningful toggle rather than needing to also gate these.
- Smoke-cast tint (see above) - flagged as easy follow-up, not built this pass.

## NPC posture-break tuning: block over parry/dodge (2026-09-17)

Jay: NPCs should parry/dodge less and lean on Block instead, so the Posture mechanic (weapon
configs already deal `Posture` damage per hit - `ReplicatedStorage.Config.Weapons` - building
toward a guard-break stun, per the "Posture Break" client effect/UI bar) can actually come into
play. Applied to `ServerScriptService.Services.AI.AI.NPCBehaviors` in both places (Map + Combat
first, confirmed via read-back, then mirrored to Zogan's Academy Grounds).

Root cause found by reading the decision order in `NPCController.HandleChase`: `ShouldHoldBlock`
is checked *first*, before the Dodge/Parry fallback - but it required `ConsecutiveHits >= 3`
(three hits already landed) before it would even roll, so in practice it almost never fired early
in a fight and NPCs fell straight through to the Dodge/Parry coin flip every time. Fixed by
inverting the balance rather than touching the decision order itself:

- **`ShouldHoldBlock`**: removed the `ConsecutiveHits < 3` gate entirely - Block is now considered
  on every incoming attack, not just as a late panic reaction. Chance raised from a flat
  `35% * desperation` (capped 85%) to an escalating `65% -> 75% (1 hit) -> 85% (2+ hits)`, times
  desperation, capped 95% - mirrors the escalation shape `ShouldParry` already used.
- **`ShouldParry`**: base chances cut roughly 3x (`55%->15%`, `65%->22%`, `75%->30%`,
  `90%->35%` for M2/Uppercut). The separate "reactive parry while already stunned" branch (a
  near-guaranteed parry any time a stunned NPC gets attacked close-up) is no longer unconditional -
  now rolls `35%` first.
- **`ShouldDodge`**: the unconditional early-out (`math.random(1,5) < 2`, a flat 40% dodge on any
  incoming attack or while stunned, checked before distance/anything else) cut to `math.random(1,10)
  < 2` (20%). `BaseDodgeChance` roughly halved (`18%/30%/45% -> 8%/14%/20%`), the desperation-scaled
  cap lowered (`90% -> 60%` at max desperation) so even a near-dead NPC isn't dodging nearly every
  hit, and the sprint-punish dodge halved (`15% -> 8%`).
- **Not touched**: `ShouldEvasive` (the stun-recovery reposition move, gated on `Distance >
  AttackRange` - a different branch than the in-range block/dodge/parry decision, so it doesn't
  interfere with posture building up at melee range) and the low-health desperation multiplier
  system itself (kept as a multiplier on top of the new lower bases, same as before). `M1Only`
  mobs (zombies etc.) were already exempt from this whole decision tree and remain so.

### Test results
Cmdr `selftest` clean in both places after (**18 passed / 0 failed / 0 warnings**), no new console
errors. This is a probability/priority-order change only - no state machine or new mechanic
touched, so self-test coverage (module load, no syntax/require errors) is the right bar here, same
as the balance-only economy/quest fixes earlier in this session. Not separately play-tested against
a live NPC fight to eyeball the new block-heavy behavior and confirm posture actually breaks -
worth doing before calling this final.

## Smoke move animations ported from Hollow Lineage (2026-09-17)

Jay: reuse real animations/VFX from his other Roblox project, Hollow Lineage (placeId
95339301338452, a separate Roblox universe/experience), to make the 16 existing Smoke moves feel
better - explicitly no new moves/mechanics, presentation only. Hollow Lineage was read-only
throughout; nothing there was touched.

**Mechanism**: an `Animation` instance's `AnimationId` is just a reference to a published Roblox
asset - since Jay owns both experiences, the same ID works in either place with no file transfer
needed. Read each source `AnimationId` from Hollow Lineage via `inspect_instance`, created a
matching `Animation` instance in `ReplicatedStorage.Assets.Animations.Smoke.<TypeId>.<MoveId>` in
Dorohedoro with the same ID (via `execute_luau` - `multi_edit` only handles script text, not
generic instances), then wired a `TrackService.Play(...)` call into each move handler at the same
point it already calls its `Kit.*` VFX helper (`ServerScriptService.World.SmokeMoves.Kit` - the
existing flat-color procedural VFX were left completely alone, this only adds a real animation
layer on top). Zero Smoke move previously played any animation at all - casting just triggered
numbers and a colored flash while the character stood still.

**13 of 16 moves ported** (both places, byte-identical):

| Dorohedoro move | Hollow Lineage source | AnimationId |
|---|---|---|
| Split.SplitCut | Elegant Slash | rbxassetid://130781506918816 |
| Split.Unravel | Triple Slash | rbxassetid://102476477167032 |
| Gun.SmokeShot | Needle's Eye | rbxassetid://80043743224718 |
| Gun.ChargedBlast | Dark Flame Burst | rbxassetid://96216989359164 |
| Curse.Hex | The Shadow | rbxassetid://79545818095671 |
| Curse.Wither | Dark Eruption | rbxassetid://81904091476065 |
| Regeneration.Mend | Sweet Soothing | rbxassetid://88333784028455 |
| Regeneration.Surge | Hyper Body | rbxassetid://98773183894553 |
| Mushroom.SporeBurst | Dagger Throw | rbxassetid://86021617571891 |
| Mushroom.Cling | Skewer | rbxassetid://71115530944694 |
| Lizard.TailSweep | Spin Kick | rbxassetid://128906002364308 |
| Dinosaur.Stomp | Axe Kick | rbxassetid://130518204548540 |
| Dinosaur.Bite | Serpent Strike | rbxassetid://96782185266262 |

**3 skipped, on purpose:**
- `Mushroom.SporeTrap` - no fitting "place an object on the ground" animation found in Hollow
  Lineage's catalog (it's an anime-combat game, not a trap/engineering-themed one); the candidate
  tried first (`Pebble`) turned out to be a plain `Script`-based Tool with no `Animation` instance
  at all. Left as VFX-only, matching its prior state.
- `Lizard.ScaleForm` / `Dinosaur.BeastForm` - these are the form-toggle moves (reskin the player's
  own rig), lower priority per the task framing since a one-off transform-in animation is a
  different, smaller payoff than every repeatable attack getting one. Not ported this pass.
- Originally picked `Shadow Tendrils` for `Curse.Hex` (thematically the best name match) but its
  `Animation` instance had an empty `AnimationId` (`Content{SourceType=None}`) in the source place -
  substituted `The Shadow` instead once that was caught.

**Verified live, not just read-back**: in both places, Cmdr `selftest` stayed **18 passed / 0
failed / 0 warnings** after, console fully clean, and `|Animations| Loaded (183/183)!` confirms
every ported `AnimationId` preloaded successfully with no ownership/moderation rejection (170 base
+ 13 new). In Map + Combat, actually pressed the real Z keybind (the direct-cast keybind from
earlier today) to cast `Mend` live and read the character's `Animator:GetPlayingAnimationTracks()`
straight from the server - confirmed a track named `Mend` playing `rbxassetid://88333784028455`
(Sweet Soothing's exact ID), proving the whole pipeline end-to-end on a real cast, not just that
the asset exists.

### Not done
- VFX porting (beyond animations) - the task explicitly said "animations mainly", and reproducing
  Hollow Lineage's particle-hierarchy VFX (some over 100 descendants) property-by-property without
  a direct cross-Studio instance-copy tool would have meant hand-rebuilding each one; prioritized
  getting every portable move a real animation (13/16, breadth) over polishing VFX on a handful
  (depth), per the task's own stated priority. The existing `Kit.slash`/`Kit.burst` flat-color VFX
  are unchanged and still fire alongside every new animation.
- No sound - ImportedSFX/Sound folders exist in Hollow Lineage but weren't touched; Dorohedoro still
  has no audio infrastructure at all (a pre-existing, separately-logged gap from earlier in this
  session), and wiring sound in without a wider sound-trigger system felt like scope creep for a
  "no new mechanics" pass.
- Cast animations don't yet respect weapon-equip visuals or interrupt combos - they play at
  `AnimationPriority.Action` exactly like `Slide` already does, no further tuning attempted.

## Release-readiness pass: playtest, bug hunt, optimization audit (2026-09-22)

Jay: "optimize the game, self-playtest, search for bugs, find ways to make it releasable."
Worked against Map + Combat (placeId 87872916277829). Two code fixes applied and verified by
read-back; one apparent bug chased down and proven to be an artifact of my own test method;
three audits came back clean.

### Fixed

1. **PoliceStation Teleport4 was a dead teleport that left players stuck.**
   `Workspace.Map...PoliceStation.Parts.Teleport4.ComponentPrimaryPart.ProximityPrompt.Script`
   pointed at `game.workspace.CochleaOutside`, which does not exist in this place (it never came
   across from jjk - PROJECT.md already flagged it as a pre-existing jjk bug, but as "not caused by
   the merge" rather than as something live). Triggering the "Outside" prompt threw
   `CochleaOutside is not a valid member of Workspace` and the player did not move.
   The state of that whole interior loop is worth knowing:

   | Prompt | Destination | Enabled |
   |---|---|---|
   | Koban -> PoliceStation ("Police Station") | Teleport3 | **false** |
   | Teleport3 -> outside ("Outside") | PoliceKoban.Model.Teleport - **valid** | **false** |
   | Teleport4 -> outside ("Outside") | `workspace.CochleaOutside` - **missing** | **true** |

   So the working exit was switched off and only the broken one was live. Repointed Teleport4 at
   the same Koban street spot Teleport3 uses, resolved defensively through `FindFirstChild` so a
   missing destination can never throw again, and made the character lookup nil-safe (it was
   `Player.Character.HumanoidRootPart` with no guard). Left both prompts' Enabled flags alone -
   the entrance being disabled looks deliberate, so this is a safety fix, not a re-opening of the
   interior.

2. **`Services.Player.Data.GetProfile` returned zero values instead of `nil`.**
   The function fell off the end when a player had no profile, so it returned *no values at all*,
   not `nil`. That is invisible to `local data = GetProfile(p)` (the common case, and why nothing
   had broken yet) but breaks any caller that forwards the result: `tostring(GetProfile(p))` raises
   `missing argument #1`, `f(GetProfile(p))` passes no argument, `{GetProfile(p)}` builds an empty
   table. Found it by accident when a probe calling `tostring()` on it threw. Added an explicit
   `return nil` and marked the return type `PlayerData?`.

### Resolved - the SelfTest "failure" was my own measurement artifact

During this pass `SelfTest` reported **16 passed / 1 failed** - test 14 "player data loaded +
versioned" failing with `DataVersion nil, 2 bytes`, and the run dropping from 18 checks to 17
(the pit test is gated on `if HERE == "Hole" and data then`, so a nil `data` silently skips it).

**The game is fine. The measurement was wrong.** Proven, not assumed:

1. **The data layer demonstrably works.** A real client-side `ShopBuy` bought Mushroom Tea at the
   Diner counter: Yen 8265 -> 8215, Smoke restored. That handler returns
   `"Your data is still loading."` when `GetProfile` is nil. `WorldSignals.SpendYen:Invoke(player, 0)`
   - which runs EconomyService's own closure - also returned `true`, which likewise requires a
   non-nil profile.

2. **`execute_luau` has a module cache separate from real server scripts.** Injected
   `require(Services.Player.Data)` returns the same table across separate `execute_luau` calls
   (verified by tagging it), but it is *not* the instance the game uses: calling
   `UpdateState(player, {UpdateYen, +7})` through it was a no-op on `leaderstats.Yen`, while real
   purchases move that same value. So the injected copy's `Profiles` table is permanently empty -
   `Shared.OnStart()` never ran in that context - and `GetProfile` there always returns nil.

3. **Injected connections persist across calls and win the race.** `_G` survives between
   `execute_luau` calls and a connection registered in one call still fires in a later one
   (verified). So the moment I did `require(SelfTest)` from injected code, a *second* live SelfTest
   instance existed, bound to `workspace:GetAttributeChangedSignal("RunSelfTest")` alongside the
   real one that `SelfTestRunner` binds at boot.

4. **Roblox fires the most recently connected handler first, and the guard lets only one run.**
   The handler does `if attr == true then attr = false; SelfTest.run() end` - the flag is cleared
   synchronously before `run()` yields, so whichever connection fires first is the only one that
   ever executes. A direct reproduction of that exact pattern in Edit mode, 4 runs across 2 and 3
   connections, had the **last-connected** handler win every time (`ran: 1` each trial).

   Therefore my injected copy - connected last - won every `RunSelfTest` trigger this session, and
   every selftest result observed came from the copy with the empty `Profiles` table. The real
   SelfTest never ran.

**Gotcha worth remembering:** never `require` a module from `execute_luau` if that module registers
event connections at its top level. The injected copy is a *separate instance* with uninitialised
state, it stays connected for the rest of the session, and because Roblox dispatches
last-connected-first it will shadow the real handler on any guarded/one-shot signal. To test a real
server module, drive it through an entry point the real context owns (a `WorldSignals` Bindable, a
remote invoked from the Client datamodel, or Cmdr) rather than requiring it.

**Still worth doing:** an actual clean `selftest` run (Cmdr `selftest`, or the workspace attribute
in a session where SelfTest was never required from injected code) to confirm it is back to 18/18.
Blocked today only by the Studio Play hang below. **Update:** done — 18/0, see "Clean selftest" below.

### Audits - all clean, no action taken

- **Remote/exploit surface.** 27 remotes; 22 scripts hold server-side handlers. All 10 World
  services that expose a remote call `Guard.allow` (RemoteGuard). `VioletSkillsServer.Cast` - the
  one big hand-written handler - validates properly: tool type, tool parent, config/name match,
  R6 rig, blocked states, and a server-side cooldown keyed on `workspace:GetServerTimeNow()`.
  `Packets.Replicate.OnServerEvent` (a client-driven effect relay, an obvious spoof vector) is
  connected only under `RunService:IsStudio()`. The known real gap is still the deferred one:
  no server-side movement/hit validation in `DamageLogic`.
- **Per-player memory hygiene.** 22 World services write per-player state; 21 have
  `PlayerRemoving` cleanup. `EventService` is the only one without - checked it, and both tables
  (`broken`, `served`) are event-scoped locals discarded when the event ends, not module state.
  No leak.
- **Workspace performance.** 714,455 instances total, 517,167 in Workspace; 355,192 BaseParts.
  StreamingEnabled is **on**. Only 163 unanchored parts (no stray physics load). 6,998 lights with
  `Shadows = true` on **zero** of them - already the cheap configuration. 732 ParticleEmitters,
  837 Beams. Collision fidelity is sane: 403 PreciseConvexDecomposition against 25,349 Box and
  5,013 Hull. The one number worth revisiting if client FPS becomes a problem is **103,040 parts
  with CastShadow = true** - but that is a look change, not a free win, so it is a decision rather
  than a fix.

### Stale items in this log / PROJECT.md - corrected

- `Lab2.EnterPart/ExitPart` "`Triggered` is not a member of Part" - **already fixed 2026-09-16**;
  both scripts now use the child prompt and a CFrame target.
- `ReplicatedStorage.Weather` / `workspace.SmallClouds` missing (WeatherManager) - real, but
  **`WeatherManager` and its `DaylightManager` are both `Disabled = true`**, so neither runs and
  neither errors. Atmosphere is handled by `AtmosphereService` / `AtmosphereClient` instead.
- `Remotes.CombatTag` missing - both referencing scripts already degrade safely
  (`WaitForChild(..., 10)` + warn, and a `FindFirstChild` with a 5s fallback).

### Still true / not addressed
- The asset-permission errors remain (console shows ColorMap `14565342511` on
  `Underground Lab.Model.B2.SurfaceAppearance` rejected). Not fixable from Studio - needs sharing
  from each asset's page.
- ~~A full player profile table is printed to the server console at boot~~ - RESOLVED below: it is
  Studio plugin output, not game code, so it never reaches live servers. Originally not found in any
  game container; a later full-tree scan confirmed no script prints it.

### Studio incident
`start_stop_play` wedged partway through a restart: the game stopped, the next start timed out
after 120s, and every later `start_stop_play` call returned `Start play hasn't finished yet` while
`get_studio_state` kept reporting Edit and `execute_luau` kept working normally against the Edit
datamodel. Same class of hang logged on 2026-09-17, which cleared on its own. Needs Studio's
Stop/Play pressed by hand to clear. All Edit-mode work above was completed and read back while wedged.

## Live playtest (2026-09-22, same pass) - Studio Play hang worked around

### Working around the wedged Play toggle
`start_stop_play` stayed wedged (`Start play hasn't finished yet`) for ~30 minutes. `game:SavePlace()`
is not callable from Edit ("can only be called from a server script"), so the place could not be saved
programmatically either. What finally worked was sending **F5 at the OS level**:

- `WScript.Shell.SendKeys` - no effect on Studio's Qt UI.
- `keybd_event` alone - **wrong window**. `SetForegroundWindow` is refused when called from a
  background process, so the keystroke went to whatever had focus.
- **`AttachThreadInput` + `SetForegroundWindow` + `keybd_event`** - works. Attach our thread's input
  queue to the current foreground window's thread first, which grants foreground rights. Always
  verify `GetForegroundWindow() == Studio hwnd` **before** sending any key, and target the right PID
  (two Studio instances are usually open - Map + Combat and Zogan's Academy Grounds).

After the session, `start_stop_play(false)` worked again - the wedge cleared once Play had actually run.

### Clean selftest: 18 passed, 0 failed, 0 warnings
Triggered via `workspace:SetAttribute("RunSelfTest", true)` in a fresh session where SelfTest was
never required from injected code. Test 14 now reads `DataVersion 1, 2015 bytes`, and the pit test is
back (`fee 40 refunded, Yen 8215 -> 8215`). Confirms the earlier 16/1 was purely the injected-copy
artifact described above.

### What was actually played
Clean boot: no errors, no warnings, `|Animations| Loaded (183/183)!`. Console stayed clean through
the entire session - the only errors in the log are my own `AssistantCommand` probes.

| System | Result |
|---|---|
| M1 melee | Works. `Combo` advances 1->3, attack animation plays, **10.2 damage** to a Rig dummy. Hitbox reach is between ~4.3 and 6 studs - at 6.0 studs, dead centre and facing the target at 0 degrees, **nothing lands**; at 4.3 studs it does. Not a bug, but the reach is tight. |
| Smoke move (Z / Spore Burst) | Works end-to-end. Continuous sampling caught **Smoke 328 -> 304.7 (exactly the 24 cost)** at t=4.73s and **6.8 damage** landing at t=5.85s, then steady regen back up. A single before/after read misses it - there is ~4s of input latency through the MCP bridge, so sample in a loop. |
| Bag / inventory | Opens on **K**, not B. Shows the saved inventory (Axe, Black Smoke x2, Bone Mask, Cleaner's Coat, Dagger, Gas Mask, Grave Bell, Katana, Scrap Vest, Sword & Shield) with correct equipped-slot labels. |
| Random events | Fired unprompted during play: "Ashmask Sweep - The Ashmasks finish their sweep and melt back into the fog." |
| HUD | WorldInfo (name, `V. dev`, uptime, region US), quest tracker, smoke bar `SMOKE · MUSHROOM 328/328`, move list with costs, equipment bar, burial tags, Yen - all rendering. |
| Client performance | **83.8 FPS**, only **924 parts** streamed within 200 studs. StreamingEnabled is doing its job. |

### Fixed: HUD z-order collision
The bag panel and the jjk health/smoke bars are drawn on top of each other. Measured at runtime:

- `Custom Inventory/Inventory`  x 398-887, y **166-277**
- `Player_Display/Player` (health + smoke bars)  x 222-1063, y **48-320**

The bag sits entirely inside the HUD bars' rect, and the smoke bar text bleeds through the item grid.
Root cause is not the layout but the draw order: **`Hud`, `Player_Display` and `Custom Inventory`
were all `DisplayOrder = 0`**, and between separate ScreenGuis with equal DisplayOrder Roblox does
not guarantee which draws on top. Set `StarterGui["Custom Inventory"].DisplayOrder = 10` - above
`Hud`/`Player_Display` (0) and `SmokeGui` (5), below the `WorldUI`/`ShopGui`/`DoorGui` layer (20).
Read back and confirmed. This only makes the bag draw *deterministically* on top; it does not settle
the still-open question of the overlapping health bars themselves.

### Flagged, not changed
- **`Transition` is `DisplayOrder = 0`.** It is the fullscreen fade (measured 2572x724, covering the
  whole screen) used for door travel, so it shares the same ambiguity as above and can end up drawn
  *under* the HUD mid-fade. Left alone because raising it changes how transitions look, which is
  a call rather than a fix.
- **M1 reach.** Whiffing at 6 studs while perfectly aimed may feel bad in PvP. Worth a deliberate
  decision rather than a silent tweak.
- **Spore Burst costs 24 Smoke for 6.8 damage vs M1's 10.2 for free** against a single target. It may
  be balanced by AoE, which a one-dummy test cannot show - worth checking against a group.

### Corrected: the boot-time profile dump is NOT game code
Earlier in this pass I flagged the full player profile printed at boot as log spam to remove before
release. It is **not** produced by the place: a whole-source scan of **all 1,071 `LuaSourceContainer`s
in the live server tree** (0 unreadable) found exactly one quote-free `print` -
`ReplicatedStorage.VioletSkills.ForgeEmit.obj.Promise: print(text)`, where `text` is a string. The
dump appears interleaved with plugin output ("Loaded Cajuns Animation Spoofer v2", "Animation Spoofer
cannot run while game is running"), so it comes from a Studio plugin and **will not appear on live
servers**. No action needed.

## Optimization pass - measured with SceneAnalysisService (2026-09-22)

Jay: "just work on optimization for now." Everything below is **measured**, not guessed, using
`SceneAnalysisService` in Play mode with a locked camera for each A/B. **No optimization was
applied** - every candidate was tested first and three of four were rejected because the measurement
said they would not help. The scene is unchanged apart from the earlier `DisplayOrder` fix.

**Caveat on absolute numbers:** these FPS figures come from Studio Play with a second Studio
instance open and the Edit datamodel resident (~11.4 GB). A real client will be faster. The
*relative* deltas are the meaningful part; the absolute FPS is not.

### The one real finding: the shadow pass is ~94% of rendered geometry

Street-level view, streaming intact, character at the Spawns1 spawn:

| Pass | Triangles |
|---|---|
| **Shadows** | **1,738,584** |
| Opaque | 115,157 |
| Particles | 860 |
| UI | 762 |
| Transparent | 562 |
| **Total** | **1,855,925** (1,719 draw calls, 61.8 FPS) |

The shadow map renders **15x more geometry than the visible scene**. This is the sun/global shadow,
not point lights - the earlier audit found 6,998 lights with `Shadows = true` on **zero** of them.

`Lighting` carries the attribute `RBX_LightingTechnologyUnifiedMigration = true`, which points at
**Unified** lighting - the most expensive technology. `Lighting.Technology` cannot be read from
injected Luau (`lacking capability RobloxScript`), so treat that as strong evidence, not confirmed;
check it in the Lighting properties panel.

### A/B: GlobalShadows off (locked camera, same view)

| | GlobalShadows on | off |
|---|---|---|
| Draw calls | 1,495 | **487** |
| Triangles | 1,364,303 | **100,358** |
| FPS | 59.3 | **73.4** |

**-67% draw calls, -93% triangles, +24% FPS.** Draw calls drop below the ~1,000 mark that matters
for mobile. This is the only large lever found, and it is a **look decision, not a free win** - the
gritty slum lighting leans on those shadows. Not applied; Jay's call.

### Three candidates tested and rejected

1. **Disable `CastShadow` on underground interiors.** 62,126 shadow-casting parts (60.3% of all
   102,956) sit below y = -50. Looked like a large, invisible win. **Measured it first: it gains
   nothing.** Standing inside the densest underground cluster (12,086 parts within 150 studs), the
   render was **17 draw calls / 20,184 triangles / 110 FPS with no Shadows pass at all** - Roblox
   already culls the shadow cascade underground. Would have been a 62k-part mutation for zero gain.
2. **`RenderFidelity: Precise -> Automatic`.** 28,889 MeshParts (58%) are `Precise`. Switching the
   4,481 streamed ones cut triangles 8.6% and draw calls 4.1% but **FPS was flat (-0.3, inside
   noise)**. Triangle count is not the bottleneck; the GPU eats 1.7M fine. The shadow *pass* costs
   because it is a whole extra pass, not because of its triangle count. Reverted.
3. **Disable `CastShadow` on small props.** Only **253 parts map-wide (0.2%)** have a largest
   dimension under 4 studs. 70,189 (68%) are 8+ studs - the shadow load is building geometry, which
   cannot be hidden without changing the look. No headroom.

### Confirmed healthy - no action needed
- **No memory leaks.** `GetUnparentedInstancesAsync` reports **53** unparented instances total
  across 20 host scripts; the largest is Roblox's own `AnimationLoader.Rig.Animate` holding 15
  preloaded `Animation` objects. Several others are Roblox's `PlayerModule`. This is a normal
  baseline, and it matches the static finding that 21/22 World services clean up on `PlayerRemoving`.
- **Streaming works well.** Only ~1,877 parts streamed within 200 studs of the player out of 355,127
  in the workspace. Underground interiors render at 110 FPS.
- `GetScriptMemoryAsync` is unavailable in this Studio build (`requires STUDIOPLAT37936 flag`), so
  per-script Luau heap was not measured.

### A process note (two invalidated measurements)
Two mid-pass A/B results were wrong and had to be redone: setting `plr.ReplicationFocus` to an
underground probe part to force streaming there, and then never resetting it, meant the city was
no longer streamed to the client. The "dense city view" afterwards reported 48 draw calls / 119k
triangles - obviously wrong for a street view, which is what caught it. **Reset `ReplicationFocus`
to `nil` and destroy any probe part before measuring anything else**, and sanity-check draw-call
counts against the view before trusting a delta.

### Verified clean afterwards
`GlobalShadows = true`, `__ClaudeFocus` destroyed, all 28,889 `Precise` meshes still `Precise`,
`ReplicationFocus` nil. The only surviving change from this session is
`StarterGui["Custom Inventory"].DisplayOrder = 10`.

### Recommendation
There is no free performance win left in the scene - it is already well configured (streaming on,
no shadow-casting lights, negligible unanchored physics, no leaks). The remaining decisions are
both look-vs-speed and both Jay's:
1. **Check `Lighting.Technology`.** If it is Unified, dropping to ShadowMap is likely the single
   biggest gain for the smallest visual change. I cannot read or set it from script.
2. **`GlobalShadows = false`** is worth +24% FPS and gets draw calls into mobile range, at the cost
   of the shadowed look. A middle path is to keep shadows on desktop and expose this as a quality
   setting - `SettingsService` and the `Setting_ScreenShake` attribute convention already exist to
   hang that off.

## UI consistency pass - everything now reads like StarterGui.Hud (2026-09-22)

Jay: "make the ui all replicate / look similar to the hud ui", and when asked, picked **Hud as the
reference** (over the newer World UI style) and **fonts + colours + panel chrome** (over fonts and
colours only).

I flagged that `Hud` is not really a designed system - 3 font families across only 7 labels, pure
white text, no palette - and that matching it means restyling the newer World UIs *away* from their
gold/dark token set. Jay chose it anyway, so that is what was built.

### The problem, measured
Before this pass the UI used **10 font families**: SourceSansPro x135, Balthazar x41, Ubuntu x36,
GothamSSm x28, AccanthisADFStd x23, LegacyArial x12, Fondamento x4, TitilliumWeb x4, Inconsolata x2,
Guru x1. Every imported GUI brought its own look.

A structural note that shaped the approach: **`WorldUI`, `SmokeGui`, `ShopGui` and `DoorGui` are
empty in StarterGui** - they are built entirely at runtime by the `WorldClient` LocalScripts. So a
static edit of StarterGui could never have reached them.

### What was built (2 new scripts, no instance mutation)

**`ReplicatedStorage.World.UITheme`** (ModuleScript) - the token set, every value lifted off `Hud`
rather than invented:

| Token | Value | Source in Hud |
|---|---|---|
| Body / Title font | `Fondamento` | BuyUI Title / Price / Yes / No |
| Small font (<14px) | `TitilliumWeb` | CooldownUI SkillName |
| Text | `rgb(255,255,255)` | BuyUI labels |
| TextDim | `rgb(221,221,221)` | DevProduct |
| Panel | `rgb(26,26,26)` | BuyUI.TextTab background |
| BarFill / BarTrack | `rgb(100,100,100)` / `rgb(0,0,0)` @0.5 | CooldownUI Bar / BG |
| Ornament | `rgb(95,99,125)` @0.7 | BuyUI.TextTab corner images |
| Outline | `rbxassetid://11200533465`, Slice, `Rect(6,6,8,8)`, -2px inset, ZIndex 20 | same |
| Corners | 4 sliced images, `Rect(0,0,32,32)`, 4px inset | same |

Hud's corner ornaments are sized in *scale* (3.9% x 4.4%), which collapses to roughly 11x3 px on a
small panel - so the theme uses a fixed 16x16 px instead, which is the same art reading correctly at
any panel size. Hud's `Background` image (11250398782) was skipped: its position/size are hand-tuned
for BuyUI's exact layout and it is not a generic panel backdrop.

**`StarterPlayer.StarterPlayerScripts.WorldClient.UIThemeClient`** (LocalScript) - applies the theme
to the whole `PlayerGui` at runtime, with repeat sweeps over the first ~3s (the code-built GUIs
populate late, and `AbsoluteSize` is meaningless until a frame is laid out) plus a `DescendantAdded`
hook for things created later (toasts, dialogue rows, shop rows, quest cards).

**Nothing is baked into the place.** Every change is made to the live PlayerGui copy, so deleting
`UIThemeClient` reverts the entire look in one step, and the player attribute `Setting_UITheme = false`
disables it without deleting anything.

### Deliberately not themed
- **`Hud`** - it is the reference; restyling it would be circular.
- **`Player_Display`** - health / stamina / posture bars. Their colours are functional, not
  decorative, and it has 0 text objects anyway.
- **`Vignette`, `Transition`, `ScreenEffects`** - full-screen overlays; chrome would be wrong.
- **`Cmdr`, `Freecam`** - third-party / Roblox built-ins with their own design.
- **`WorldInfo.LeekPrevention`** - the 36 anti-leak watermark labels at 93% transparency. Restyling
  them would only make them more visible.
- **Any frame whose name looks like a meter** (`bar`, `fill`, `track`, `health`, `stamina`,
  `posture`, `progress`, `slider`, `meter`, `gradient`) never gets a dark body or chrome.
- A frame only counts as a panel if it is visible, at least 120x40 px, **and** contains text - so
  decorative sub-frames keep their own backgrounds.

### Verified live
`[UITheme] applied Hud look - 185 text objects, 19 panels`, console otherwise clean (only the
pre-existing SurfaceAppearance asset-permission errors). After the sweep, across the whole PlayerGui:

| Font | Count | |
|---|---|---|
| Fondamento | 199 | themed |
| TitilliumWeb | 43 | themed |
| Ubuntu | 36 | WorldInfo.LeekPrevention - intentionally skipped |
| SourceSansPro | 3 | Cmdr - intentionally skipped |
| Inconsolata | 2 | Cmdr - intentionally skipped |
| Guru | 1 | Hud.DevProduct - the reference GUI itself |

**10 font families down to 2**, and every remaining exception is on the skip list by design.
Screenshots confirm the quest card, Rule-of-the-Hour banner, smoke-move list, bottom-left buttons,
WorldInfo top bar and the bag all now carry Fondamento text inside Hud's outlined chrome, while the
health/stamina bars below are untouched.

### One bug found and fixed on the way
First run threw `Guru is not a valid member of "Enum.Font"` - Guru is FontFace-only and has no
`Enum.Font` member, so the module failed to load and nothing themed. The token was unused anyway
(Hud is skipped), so it was removed with a comment explaining why there is deliberately no Notif token.

### Follow-ups worth considering
- The `WorldClient` scripts still hard-code `Balthazar` / `SourceSansPro` and the old gold palette
  when they build their UI; the theme overwrites it a frame later. That works, but it means two
  sources of truth. Porting those scripts to require `UITheme` directly would remove the flicker
  risk and the duplication.
- `Transition` is still `DisplayOrder = 0` (flagged in the playtest section) - unrelated to this
  pass but still worth a decision.

### Regression found and fixed while capturing screenshots (same pass)

The first live pass broke the **Quest Log**: it rendered as an empty bordered box. The rows were all
present and `Visible` (`Completed (17)`, every Enrollment entry) but invisible on screen.

Cause: `ThemeOutline` was given `ZIndex = 20`, copied straight off Hud, while the quest rows sit at
ZIndex 8 - so the outline's fill painted over the panel's own contents. It works in Hud only because
BuyUI's content sits higher still; applied generically it hides everything.

Two fixes:
1. **Chrome moved behind content** - `OutlineZIndex` and `CornerZIndex` are now `0`, which still
   draws above the frame's own background but below every real child. Chrome is decoration; it
   should never be able to occlude content.
2. **No nested chrome** - `isPanel` now rejects any frame with a themed ancestor
   (`hasThemedAncestor`). Panels were being stacked inside panels inside scrolling frames, which
   both caused the occlusion and read as clutter. Themed panel count dropped 19 -> 15, and the
   Character panel went from a box around every attribute row to one clean outer panel.

Re-verified live: Quest Log shows the active quest with Travel/Abandon buttons and the full
Completed (17) list; bag, character panel and default HUD all render correctly.

### Captures
Saved to `captures/2026-09-22/` (viewport-cropped, 1285x419):
`01-hud-default.png`, `02-bag-inventory.png`, `03-character-panel.png`, `04-quest-log.png`.

Method note: MCP `screen_capture` returns image data inline and writes no file, so the captures are
taken with PowerShell `CopyFromScreen`. Two things that matter for repeating this:
- The game viewport had to be located inside the Studio window. Done by briefly showing a
  full-screen magenta `ScreenGui`, capturing, and scanning for the magenta bounding box - which
  gave viewport at (254,176) 1285x419 inside a 1918x1030 window, matching `Camera.ViewportSize`.
- `CopyFromScreen` grabs whatever is physically on screen, so the first capture included an
  overlapping terminal window. The capture script now raises Studio via `AttachThreadInput` +
  `SetForegroundWindow` and **aborts rather than capturing** if focus cannot be confirmed.
Helper script kept at `scratchpad/capture.ps1` for the session; viewport rect cached in
`%TEMP%\claude_viewport.txt`.

## UI follow-ups closed out (2026-09-23)

Both items flagged at the end of the theming pass, plus one bug the first fix exposed.

### 1. The font flash - first fix was wrong, second one is right

New UI (toasts, dialogue lines, shop rows, quest cards) rendered one frame in the WorldClient
scripts' own Balthazar/gold before `UIThemeClient` flipped it to Fondamento.

**First attempt: style text synchronously inside `playerGui.DescendantAdded`.** It does not work.
Measured it with a probe label: parent a `TextLabel` with Balthazar set, read `FontFace` back with
no yield -> still `Balthazar`; only after a frame -> `TitilliumWeb`. **`DescendantAdded` fires
deferred in modern Roblox**, so no handler on it can ever be same-frame. Any runtime restyle is
inherently at least one frame late. The handler improvement was kept anyway (it now also clears the
tag and re-asserts on the deferred pass, so a script that sets its font *after* parenting no longer
wins), but it cannot fix the flash.

**Real fix: the WorldClient scripts now source their style from `UITheme` directly**, so the wrong
font is never rendered at all. They all declared their palette as module-level constants, so this
was a declaration change, not a rewrite of every call site:

| Script | Now uses |
|---|---|
| `SmokeMovesHud` | Panel / Text / TextDim / FontFace.Title / FontFace.Body |
| `BagClient` | + PanelRaised, Accent |
| `QuestClient` | + State.Good, State.Away, FontFace.Bold; marker `crate` colour -> Accent |
| `CharacterTreeClient` | + State.Good, State.Bad |
| `PitClient` | + State.Danger |
| `LeaderboardClient` | inverted onto the dark palette (see below) |
| `RuleBannerClient` | font + neutral body text only; reds kept |
| `WarMeterClient` | font + neutral label text only; meter fills kept |
| `AtmosphereClient` | rain-warning font only; hazard green kept |

`LeaderboardClient` was the most off-brand UI in the game - a light parchment panel in **Bangers**.
Its light/dark roles invert cleanly onto the Hud tokens: `PAPER` was the surface so it became the
dark panel, `INK` was the text on it so it became the light text.

New tokens added to `UITheme`:
- `UITheme.FontFace.{Title,Body,Bold,Small}` - `Font.new` equivalents, because these scripts assign
  `.FontFace` rather than `.Font`.
- `Color.PanelRaised` `rgb(38,38,38)` - one step up from Panel for rows on a panel. Hud has no
  second surface of its own (BuyUI is a single TextTab), so this is **derived, not sampled**.
- `Color.Accent` `rgb(214,170,90)` - **the one token that is not Hud's.** Hud never styles an
  interactive element (its Yes/No are plain white text), so there was nothing to copy. The old gold
  is kept so buttons still read as buttons. Hud's only non-greyscale value is
  `Ornament rgb(95,99,125)`; switching `Accent` to that is a one-line change if a strictly
  Hud-only palette is wanted.
- `UITheme.State.{Good,Bad,Danger,Away}` - semantic colours kept as-is. These encode meaning
  (good/bad/danger/elsewhere), so flattening them to greyscale would lose information.

Verified: console clean, no require or syntax errors, font distribution unchanged
(Fondamento 199 / TitilliumWeb 43 + the three intentional exceptions), quest log / bag / character
panel all still render correctly.

### 2. `Transition` could not actually black out the screen

`StarterGui.Transition` is the fullscreen fade used by `InteractUI.InteractableFrame.LocalScript`
for teleports (with the ClapDoor sound). It sat at **`DisplayOrder = 0`** - the same layer as `Hud`,
`Player_Display` and `InteractUI` - so during a fade the HUD could draw straight through it. A
blackout that does not black out is a bug, not a style choice.

Set `DisplayOrder = 100` (highest game layer was 20) and `IgnoreGuiInset = true` so the fade also
covers the topbar strip. **Verified by measurement**, not by eye: forced `Blackout.BackgroundTransparency = 0`,
captured the viewport, and sampled 33,810 pixels - **0.2% non-black**, and those are Roblox's own
CoreGui topbar buttons, which no `ScreenGui` can cover. Restored afterwards.

Resulting layer order: Hud/Vignette/InteractUI/Player_Display (0) -> WorldInfo (2) -> SmokeGui (5)
-> Custom Inventory (10) -> WorldUI/ShopGui/DoorGui (20) -> **Transition (100)**.

### Captures refreshed
`captures/2026-09-22/` re-taken after the refactor: `01-hud-default.png`, `02-bag-inventory.png`,
`03-character-panel.png`, `04-quest-log.png`.

### Still open
- The boot-time profile dump did not appear in this session's console at all, which is further
  evidence it is plugin output rather than game code (already concluded in the playtest section).
- `POIGuideClient`'s waypoint colours (`POI` gold, `Safe` green) are still inline rather than
  tokenised. They are semantic categories, so they were left alone - tokenise them if more waypoint
  types get added.

## Graphics / performance settings (2026-09-23)

Jay: "implement more settings for optimization, shadows fps tracker etc". Built on the existing
`SettingsService` convention rather than a parallel system: keys live in one `SPEC` table, are saved
in `PlayerData.Settings`, mirrored to player attrs `Setting_<key>`, and changed by the client
through `Remotes.SetSetting`. Only non-default values are stored, so the save stays small.

### Four new keys

| Key | Default | What it does | Why |
|---|---|---|---|
| `Shadows` | true | `Lighting.GlobalShadows` | **The only big lever.** Measured on the Shibuya street view: shadows were 1.74M of 1.86M rendered triangles (~94%). Off: draw calls 1495 -> 487, triangles 1.36M -> 100k, **+24% FPS**, and draw calls drop under the ~1000 mark that matters on mobile. |
| `PostFX` | true | Bloom / SunRays / DepthOfField `.Enabled` | Full-screen passes, cheap to skip on a weak GPU. |
| `AmbientFX` | true | every `ParticleEmitter` under `workspace.CurrentCamera` | That is exactly AtmosphereClient's weather layer - rain streaks (Rate 900), mist, petals, ash. Combat VFX live under `workspace.Visuals` and are never touched, so turning weather off can never hide a move's effects. |
| `FpsCounter` | false | the perf HUD | So a player can tell whether any of the above actually helped. |

### `StarterPlayerScripts.WorldClient.PerformanceClient` (new)

Applies the four settings and draws the counter. Notes on the two non-obvious bits:

- **PostFX only ever forces effects off.** When the setting is on it restores whatever it previously
  suppressed and then stops touching them, so AtmosphereClient stays free to enable/disable its own
  effects for Hell / BlueNight / rain without this script fighting it every frame. Verified:
  `DepthOfField` was `false` at boot, and toggling PostFX off and back on left it `false` rather
  than force-enabling it.
- **AmbientFX watches `CurrentCamera.DescendantAdded`**, because AtmosphereClient creates its
  emitters lazily (rain only exists once it starts raining). It sets `.Enabled`, not `.Rate` -
  AtmosphereClient drives `Rate` on a loop and would overwrite a rate change, but never touches
  `Enabled`.
- The settings arrive with the profile, which can land after the script starts, so it re-applies on
  a short loop for the first ~5s instead of trusting a single pass.

The counter sits top-right (`DisplayOrder = 30`, above the world UI at 20, below Transition at 100),
themed from `UITheme`, showing FPS plus a "low" figure (slowest single frame in the window) and ping
where available. The number is colour-coded: green >= 50, accent gold >= 30, red below.

### Settings UI
`CharacterTreeClient`'s settings section was a hand-built row for ScreenShake. Replaced with a
`SETTING_ROWS` spec + `settingRow()` helper, so adding a setting is now one line here and one line
in `SPEC`. Five rows render under Character (T) -> Stats -> Settings.

### Verified live, end to end
Drove every toggle through the real `SetSetting` remote, exactly as the UI does:

```
BEFORE  shadows=true   postfx[DepthOfField=false SunRays=true  Bloom=true ]  emitters[3 on / 0 off]  perfhud=false
AFTER   shadows=false  postfx[DepthOfField=false SunRays=false Bloom=false]  emitters[0 on / 3 off]  perfhud=true
RESTORE shadows=true   postfx[DepthOfField=false SunRays=true  Bloom=true ]  emitters[3 on / 0 off]
```

All five attributes mirror correctly, the FPS counter renders ("71 FPS / low 60" green, later
"35 FPS / low 22" amber - confirming the colour thresholds), and the console stayed clean.

### Capture incident worth remembering
The PowerShell capture helper silently captured **the wrong monitor**. The Map + Combat Studio
window had been dragged to a second display (window rect went from `L=1` to `L=-1919`), while
`capture.ps1` was still using an absolute screen rect cached from the earlier session - so it
faithfully captured whatever now sat at the old coordinates (a different place entirely, a combat
test baseplate with practice dummies). Caught only because the image obviously was not Dorohedoro.

Fixed: `capture.ps1` now resolves the window origin **live** on every call and caches only the
viewport *offset* + size, plus the expected window dimensions - and throws rather than capturing if
the window has been resized, so a stale offset can never silently produce a wrong image.

Second gotcha: the re-derive step then failed with "could not focus Studio". `SetForegroundWindow`
is refused while the user is actively working in another window, which is the correct outcome - the
script aborts instead of capturing. **Do not take OS screenshots while Jay is at the machine**; use
`mcp screen_capture` instead, which renders through the plugin and needs no focus (it just cannot
write a file). The settings panel and FPS counter were verified that way.

### Accent switched to Hud's own palette (2026-09-23)

Jay: "switch to hud's pallete". `UITheme.Color.Accent` was the one token not sampled from
`StarterGui.Hud` - the gold `rgb(214,170,90)` carried over from the old World UI palette, kept
because Hud never styles an interactive element (its Yes/No buttons are plain white text) so there
was nothing to copy.

Now `Accent = rgb(95,99,125)` - Hud's only non-greyscale value, the blue-grey used on the BuyUI
corner ornaments. It is now identical to `UITheme.Color.Ornament` (asserted at runtime), so the
whole UI is strictly Hud greyscale plus that one tone.

**One line changed, nothing else.** Every accent-coloured element already sourced its colour from
`UITheme`, which is the payoff from centralising the palette in the previous pass. Verified live -
8 accent elements across `PitGui` (next), `QuestGui` (quest card + Travel button), and
`CharacterGui` (PointsDot, the four attribute `+` buttons, Respec, the active tab) all picked it up
with no per-script edits.

**Not changed:** `UITheme.State` (Good / Bad / Danger / Away) and the attribute bars, war-meter
fills, rain-warning green and rule-banner reds. Those encode meaning rather than brand, so
flattening them into the Hud palette would lose information. Screenshots confirm the attribute bars
still read purple / orange / blue / green against the new accent.

**Trade-off worth noting:** blue-grey buttons are more muted than the gold was, so interactive
elements are slightly less eye-catching. That is inherent to a strictly-Hud palette, since Hud has
no call-to-action colour of its own. Reverting is the same one line in `UITheme`.

## Catch-up: 2026-09-23 / 2026-09-24 passes recorded only in PROJECT.md

Added to the log on 2026-09-24 so the repo has them; `PROJECT.md` has the full code map for each.

### AI and combat flow (2026-09-23)
Pre-change copies: `ServerStorage.Backup_AI_2026-09-23`, `ServerStorage.Backup_Combat_2026-09-23`.
- **AI crowds** (`Services.AI.AI.NPCSwarm`): separation steering (NPC<->NPC collision stays off),
  surround slots (attack ring 4 studs, slowly orbiting waiting ring 10+), rotating attack tokens (2 per
  target for full AI, 3 for `M1Only`), 0.35 s swing spacing. Replaced the "Agro" TagService tag, which
  leaked a never-expiring tag per NPC per frame.
- **AI fairness** (`NPCBehaviors.PerceivedAction`): 0.18-0.3 s reaction time, guard budget of 3
  block/parry/dodge/evasive (+1 per 1.6 s), Evasive only after 3 hits in a combo. Wind-up tells
  (`Indicators.WindupIndicator`) before mob punches, full-AI string openers and uppercuts; a stun cancels
  them. `NPCNavigation` pathfinds when line of sight is blocked, the height gap is > 6 studs, or stuck.
- **Combat flow:** 0.6 s input buffer (`ActionManager:Request`); Dodge/Block can cancel an attack's
  recovery after its hit frame; Block/Sprint remotes only accept Activate/Release/Cancel; posture drains
  12/s after 2 s; perfect dodge now fires its effect, skips Dodge's cooldown and makes the next hit a
  x1.3 counter; soft lock-on (`Functions.SoftLock`, setting `SoftLock`); combo counter (`Misc.ComboHit`).
- **Smoke moves** live in `player.SmokeMoves`, cast on Z/X/C without equipping; a cast pressed mid-swing
  waits for the swing (`SWING_WAIT` 1 s) and drops queued M1s. Hotbar keys 1-9 owned by Custom Inventory.
- **Swing prediction** (`Functions.SwingPredict`): your ground M1 plays locally on click and hands over
  to the server's swing; tested at 0 and 150 ms lag. Server states mirrored to clients as `State_<Name>`
  character attributes; M1 animation lists sorted by numeric name.
- **Lighting:** `AtmosphereClient` is the only ClockTime writer (the jjk "Day/Night Cycle" fought it);
  Hole runs one 24 h day per 35 min off the server clock. Academy keeps 9-15.
- **Ghost toasts:** UIThemeClient no longer puts chrome on self-sizing / laid-out frames.

### Overnight build (2026-09-24)
Pre-change copies: `ServerStorage.Backup_Overnight_2026-09-24`. SelfTest 35/35 at the time.
- **En's Gang rep** (`World.Reputation`, `ReputationService`, `ReputationConfig`): earned from Ashmask
  raiders (+2), captains (+5), breaking the sweep (+4); no decay for 24 h after the last gain, then
  -5/day, floor 0. Admin `rep`.
- **Rep gates:** Rat (BackAlley) needs rep 10, else a brush-off tree with a lock badge; his Black Smoke x3
  needs rep 30 and still costs ¥330.
- **Tumor debuff** (`TumorService`, `TumorConfig`): Artificial = Smoke regen -40%, fading to a permanent
  -10% over 10 h of playtime; purple head tag. No transplant mechanic yet — set with admin `tumor`.
- **Blue Night contracts** (`ContractService`, `Contracts`): "The Broker" appears only during BlueNight;
  signing pairs you with a waiting signer you've never been bound to. The pact has no effect yet.
- **Devil trial** (`DevilTrial`, `DevilTrialService`, `DevilTrialConfig`): 4-stage scaffold entered via
  Madame Ise; dying mid-trial resets to stage 1. Stage content is TODO.
- Save changes are additive (defaults + `Profile:Reconcile()`, `DataMigrations.sanitize`).

### Day pass (2026-09-24) — Jay's picks "1-3 8-10"
Pre-change copies: `ServerStorage.Backup_HUD_2026-09-24`; archived objects in
`ServerStorage.Archive_2026-09-24` (each has `ArchivedFrom` / `ArchivedReason`).
- **Cleanup:** archived `Map.WeatherManager`, the jjk "Day/Night Cycle" and `Hud.ProgressionService`;
  `Visuals.Rocks` crater no longer errors without `Assets.Effects.Slash2.Slam`.
- **Slow-spawn warnings:** `Gui.Camera`, `Gui.Hotbar`, `Gui.HealthBar` wait with timeouts.
- **Fist reach:** Fist hitbox (4,5,4) -> (5,6,5); 3/3 hits at 6.0 studs.
- **Performance:** `CastShadow = false` on 19,418 map parts under 1 cubic stud (tag `PerfNoShadow`).
- **New HUD:** `WorldClient.VitalsHud` draws HEALTH / GUARD (and later STAGGER) under the Smoke bar;
  `Player_Display` stays enabled for the number-key hotbar but its jjk bars are hidden.

### Combat depth (2026-09-24)
Pre-change copies: `ServerStorage.Backup_CombatDepth_2026-09-24`. SelfTest 38/38, clean console.
- **Parry payoff:** a clean parry gives a 1 s Riposte (next hit x1.25, counts as a counter, adds
  stagger); the parrier's own stun drops 0.4 -> 0.2 s.
- **Stagger meter** (Humanoid attr `Stagger`): built by being parried (+40), eating a counter (+30), an
  unblocked heavy (+20), or being hit out of a swing/wind-up (+12); drains 15/s after 2.5 s. At 100: the
  guard-break stun plus a Finisher window (next hit x1.5 heavy / x1.25 other).
- **Enemy archetypes** (`Config.NPCArchetypes`): Shield, Rusher, Thrower (`NPCRanged` stone), Flanker.
  The Ashmask Sweep's grunts now use them. Verified live except a real parry into a riposte and M2
  guard-breaking the Shieldbearer.

## Lamp lighting, animation breakage, Academy sync (2026-09-24)

Jay: "fix the lighting thats emitted from light sources (makes the game seem flat since the lamps are
emitting light straight down with no source), animations break out of nowhere for npcs and players
randomly, apply past changes / updates to Zogan's Academy Place". Pre-change copies:
`ServerStorage.Backup_AnimLight_2026-09-24` (Map + Combat) and `ServerStorage.Backup_Sync_2026-09-24`
(Academy, the 60 scripts overwritten by the sync).

### Lighting (Map + Combat)
Cause: lamp bulbs were plain Plastic parts (no visible source), all 6,998 lights had `Shadows = false`
(light passed through the lamp itself, a flat disc on the road), `Lighting.LightingStyle` was `Soft`,
and the Hole's night ambient only dropped 45%, so lamp pools had nothing to stand out against.
- 473 outdoor lamp lights (models named *lamp* / High_Mast, above y = -50, Range >= 12) tagged
  `StreetLamp`, `Shadows = true`; their bulbs (<= 8 studs) tagged `StreetLampBulb`, set Neon + light colour.
- 484 other small (<= 4 studs, opaque) light-source parts tagged `LampGlow`, set Neon permanently.
- Originals in attrs `LampOrigMaterial` / `LampOrigColor`; revert snippet in workspace attr `LampPassNote`.
- New `WorldClient.LampClient`: street lamps on (Neon bulb, light on, shadows if `Setting_Shadows` isn't
  false) from ClockTime 17.6 to 6.4, off with the original bulb by day - so the shadow cost only
  applies at night. Streamed-in lamps handled through tag signals.
- `LightingStyle` Soft -> Realistic (old value kept in Lighting attr `PreLampPassLightingStyle`).
- `AtmosphereClient`: Hole-only night fill 0.45/0.4 -> 0.65/0.6 (Academy's 9-15 cycle unaffected).
- Verified in Play: 97 streamed lamps, forced night -> 97 on / 97 shadows / 97 neon bulbs, forced
  day -> 0 / 0 / 0. Not verified by eye: the Studio viewports rendered blank all session, so the
  look (brightness, bloom on the bulbs, how dark night is) still needs Jay's eyes.

### Animations
Two real bugs and one cleanup:
1. `StateController.Misc.Perfect Dodge` stopped **every** playing track on the dodger, idle/walk
   included. `AnimationController.PlayTrack` only acts on a pose *change*, so the loop stayed dead -
   the character slid or T-posed until it changed pose. NPC perfect dodges are broadcast to all
   clients, so NPCs froze on every screen the same way. Now only Action-layer tracks stop.
2. `AnimationController`: `PlayTrack` restarts the current pose's track if something stopped it, plus a
   0.3 s watchdog that restarts a stopped looping pose. Verified: stopping the idle track by hand ->
   restarted after 0.10-0.17 s, 3/3 trials.
3. `HitWeight` hit-stop froze walk cycles too: `Enum.AnimationPriority.Core` has value 1000, so the
   `>= Action` check caught Core-priority movement tracks. Core is now excluded explicitly (same fix
   in Perfect Dodge).
4. Cleanup, **not proven to be a cause**: `TrackService.Play` loaded a new AnimationTrack per call
   (every swing, every hit react) and kept it in its store forever (`CleanOnStop` was never used);
   `SwingPredict` leaked one per click; `GetKeyframeTime` one per call. Tracks are now released on
   `Ended`. I first blamed Roblox's 256-tracks-per-Animator cap, but 300 loaded tracks held alive
   played fine on both server and client with no warning - so that cap is not what breaks animations.
   Verified: 320 plays through TrackService -> 0 failures, 1 wrapper left in the store (was 320).

### Zogan's Academy Grounds brought up to date
Compared every script (hash per file) in both places. Academy was missing everything since roughly
2026-09-22: 60 scripts differed and 30 were missing. All copied (line-level hunks for changed files,
whole files for new ones), each verified by hash. Final check: **525/525 game scripts identical** in
both places. Every Academy-only line that was dropped was fight-pit code, which Map + Combat removed on
2026-09-23.
- Archived to Academy `ServerStorage.Archive_2026-09-24`: `World.PitService`, `WorldClient.PitClient`,
  `World.Remotes.PitAction`, `StarterGui.Hud.ProgressionService` (all with ArchivedFrom/ArchivedReason).
- GUI: `Player_Display.Player` hidden, `Custom Inventory` DisplayOrder 10, `Transition` DisplayOrder
  100 + IgnoreGuiInset. `LightingStyle` -> Realistic (Academy's own bright Lighting values kept).
- Map: 67 candle flames ("Bougie") set Neon (`LampGlow`); 4,148 parts < 1 stud^3 `CastShadow = false`
  (`PerfNoShadow`, same revert note). Academy has no street lamps, so LampClient has nothing to switch.
- Academy Play test: clean boot, `Animations Loaded (183/183)`, SelfTest **37 passed / 0 failed /
  1 warning** (Rat NPC only exists in the Hole). One error in the log,
  `LinearVelocity:31 ... LinearStore`, comes from SelfTest's unregistered dummy rigs in an unchanged
  file - pre-existing, not from this pass.
- Left alone on purpose: Academy's Lighting values, `FallenPartsDestroyHeight` (-2000 vs -500),
  `ServerStorage.DevRigs`.

### Still open
- The 8 shared folders are Roblox Packages (`ServerScriptService.World`, `ReplicatedStorage.World`,
  `WorldClient`, `PlayerData`, 4 GUIs) at the same published version in both places, but Map + Combat's
  edits were never published. Publishing them (right-click -> Publish to Package) would make future
  syncs one "Update All" instead of a copy - it can't be done from script.
- `Controllers.Gui.Inventory` still logs "Infinite yield possible on GameUI" on slow loads (the
  2026-09-24 timeout pass covered Camera/Hotbar/HealthBar but not this one).
- Neither place is saved by this pass (`SavePlace` isn't callable from Edit); Team Create syncs edits.

## Last 3 Smoke move animations (2026-09-24)

Jay: "check repo and do smoke move animations". The repo listed `SporeTrap`, `ScaleForm` and
`BeastForm` as the only Smoke moves with no animation (the 2026-09-17 port covered 13/16). This pass
fills those three the same way: an `Animation` in `ReplicatedStorage.Assets.Animations.Smoke.<Type>`
plus a `TrackService.Play(..., "SmokeCast")` at Action priority in the move handler.

| Move | Source (the "animation farm" place, 72078340292677) | AnimationId | Length |
|---|---|---|---|
| Mushroom.SporeTrap | `VollstandigAnimations.Power2.Downslam` | rbxassetid://133736526054969 | 0.82 s |
| Lizard.ScaleForm | `SkillAnimations.Healing.SkillOvercharge` | rbxassetid://123765930470609 | 1.25 s |
| Dinosaur.BeastForm | `SkillAnimations.Hakuda.SkillBeastialClaws` | rbxassetid://118739676730078 | 1.53 s |

- Picked only full R6 rigs with no prop joints. Rejected `Ink.HandSlam` (numbered VFX-rig joints) and
  `Horse.Transformation` (9.2 s, with a camera track).
- Forms play the animation only on toggle-**on** and only if `Forms.enter` succeeded; toggling off
  stays silent. `ScaleForm`/`BeastForm` now return the `Forms.enter` result through a local.
- **Source note:** the animation farm looks like a copy of another game (`ServerStorage.CC.GameData`,
  "spoofanimations" folders). All three IDs loaded in Play (`Loaded (186/186)`, track lengths read
  back on the server), but Studio can be more lenient than a live server - check once on a live server
  that they play.

**Tested (Map + Combat, Play):** real `SmokeCast` remote casts. SporeTrap: -40 Smoke, trap placed,
track `133736526054969` playing. ScaleForm: form Lizard, track `123765930470609`; a 2nd cast at 2.8 s
was correctly refused by the 3 s cooldown, then it toggled off. BeastForm: scale 0.90 -> 1.62, -40
Smoke, track `118739676730078`, finished within 1.9 s with idle still running; toggle-off restored
0.90. SelfTest **38 passed / 0 failed / 0 warnings**. Jay's Smoke type restored to Mushroom.
**Academy:** mirrored; the 3 scripts and all 16 Smoke animation IDs hash-identical to Map + Combat;
Play boot `Loaded (186/186)`, the 3 tracks load, SelfTest **37 / 0 / 1 warning** (Rat, expected).
**Not tested:** how they look by eye; a live server; interaction with a mid-swing M1.

## React HUD prototype in a scratch place (2026-09-24)

Jay sent a reference screenshot and asked for a HUD built with the react-lua model in the open
Studio. The open Studio was a **blank `Place1`** (not Map + Combat or Academy), holding only the
`Roblox/react-lua` package, so this is a standalone prototype. Nothing was touched in either game place.

**What was built (Place1):**
- `ReplicatedStorage.ReactLua` is the package, moved from Workspace and renamed.
- `ReplicatedStorage.HudUI`: `Theme` (colours/fonts), `HudState` (store: `set`, `subscribe`,
  `onAction`, `action`), `Primitives` (diamond caps, ornate pill bar with smooth + damage-lag fill,
  ornate button, fade line), `Widgets`, `App`.
- `StarterPlayerScripts.HudClient` mounts it with `ReactRoblox.createRoot`. Health and the player
  list are live (Humanoid, Players/Teams); everything else is demo data behind `DEMO = true`.
- Widgets: Upper Buttons (settings/menu/leave), In Combat (warning, countdown), Boss HP Bar, Quest
  UI (objectives, strike-through when done, Abandon), Party List (members with HP bars), Search
  player / Leave / Disband, Player List (team groups, scrollable), Invite (Accept/Reject), Parry,
  Health and Stamina bars, Toolbar (3 slots, keys 1-3 select). Icons come from the package's
  `BuilderIcons` font.

**Package fix:** the published react-lua package (asset 15621638430 v5) is **broken**: it is
missing 9 luau-polyfill modules (Boolean, Console, ES7Types, InstanceOf, Math, Number, String,
Symbol, Timers), and their link stubs returned the ModuleScript instead of requiring it. A fresh
insert has the same gap. Fix: removed the `PackageLink` (the linked copy auto-reverts local edits),
added minimal implementations of those 9 under `_Index`, and changed 14 link stubs to `require()`.
React and ReactRoblox now load.

**Tested (Place1, Play):** HUD mounts with no console errors; every widget renders (screen
captures); the left column (quest, party, controls) no longer overlaps at a 665 px-tall viewport.
**Also tested:** clicking Accept (invite closed, inviter added to the party) and key 2 (slot 2 selected).
**Not tested:** the other buttons,
other resolutions, and mobile. The rebuilt polyfills cover only what React calls.
**Not done:** porting into Map + Combat / Academy, or wiring the widgets to the real game systems
(VitalsHud, party attributes, quests). Waiting on Jay.

### React HUD art pass (2026-09-24)

Jay: the first version looked like "a way worse version" of the reference. It was all flat
frames. This pass added real art and rebuilt the widgets around it.

- **Art:** 14 PNGs drawn procedurally with C#/GDI+ (`tools/hud-art/` in the local project folder, not tracked by the repo; there's no Python on the
  station). They're uploaded to Jay's account as image assets; the ids are in `HudUI.Assets`:
  an ornate health-bar frame (filigree horns, jewel caps, chevrons, crests), a 9-slice pill frame
  for the small bars and buttons, a grey marble fill and a highlight layer (tinted per bar), the
  crowned, horned skull, ink smoke, the diamond menu button, the spiked ring emblem, the toolbar
  slot (chamfered frame, red wing sigil), dividers, glow and streak sprites.
- **Code:** `Primitives.Bar` puts the marble inside a clip frame so the texture doesn't squash as
  the fill shrinks. It has a lag layer, a hot glow at the fill edge and flickering embers above
  the frame. All pulsing glows share one clock binding. `OrnateButton` uses the marble fill and
  the pill frame. The font is Garamond, which Roblox resolves to the `Guru` family
  (`Garamond.json` doesn't exist; the first try at a bold variant failed to load).
- **Gotcha:** with `ZIndexBehavior.Sibling`, children of a low-ZIndex frame always draw under a
  higher sibling. The embers sat inside the fill region, so the metal frame hid them. They're now
  in an overlay frame above it.

**Tested (Place1, Play):** no console errors; all 14 images load (`PreloadAsync` Success);
full-screen and zoomed captures of every widget.
**Not tested:** mobile or very wide screens, and a published server (the images may need
moderation there).

### React HUD grit pass (2026-09-24)

Jay: the reference is "way more gritty"; ours looked cartoonish, flat and plasticky, and needed
textured squares poking in and out of the frames.

- **Generator** (`tools/hud-art`, local): a `Grunge` pass on every frame, skull and divider
  (blotches, grain, scratches, pits, chipped soft edges). The chrome is now darker worn iron. The
  fill is rebuilt with fibres, blotches and cracks, and the highlight layer is now hot cracks and
  square specks instead of gloss. New images: `flecks_a`/`flecks_b` (square and diamond flecks
  scattered around a line) and `ember` (a square spark). The health frame has square studs biting
  into and poking out of its rails. The skull is darker, with cracks and square debris. The smoke
  has fibres and ink spatter. 15 new assets uploaded; ids in `HudUI.Assets`. The old ids are unused.
- **Code:** fills, buttons and panels **tile** their texture at its drawn aspect (`P.tile`) instead
  of stretching, so the grit reads the same on every bar. Bars get two flickering fleck layers that
  spill above and below the frame over the filled part, and square embers instead of round glows.
  Flecks were also added along the player-list lines, under player rows, around "In Combat", under
  the invite, and on the Leave/Disband/Abandon/Accept/Reject buttons. Colours are a little less
  saturated.

**Tested (Place1, Play):** no console errors; all new assets `PreloadAsync` Success; full-screen
and zoomed captures (vitals, In Combat, left column, player list).
**Not tested:** mobile, other resolutions, a published server.

### React HUD back-to-basics pass (2026-09-24)

Jay sent a close crop of the reference's vitals and toolbar: better, but go back to the basics of
the reference. The UI should be **square, not rounded**, and the horns have to go (nothing in
the reference has them).

- **Generator:** new rectangular frames: `health_frame` (1024x96), `slim_frame` (9-slice, for
  parry/stamina/party bars) and `button_frame` (9-slice, with a taller fill for text). Each has
  compact iron **end caps** (a plate with a gem, knobs, tight curls and a short point) instead of
  horns, square **teeth** alternately biting into and poking out of the rails, chevrons, and centre
  knots. `center_knot` splits the parry bar and `sub_ornament` is the scrolled line hanging under the
  health bar. `slot_frame` is square with corner brackets, mid-edge notches, a smoky red well and a
  faint sigil. `badge` is the square key tab. The fill is now smoky and mottled and **tiles without
  seams** (periodic noise). The skull's horns are removed. The old pill/ornament assets are unused.
- **Code:** every `UICorner` removed. Bars use `style = "health"` or the slim default. `OrnateButton`
  uses the button frame. Vitals are laid out like the reference (parry + knot, health, hanger,
  stamina); slots are 70x70 with the square tab.

**Tested (Place1, Play):** no console errors; all assets `PreloadAsync` Success; a zoomed capture of
vitals + toolbar compared against Jay's crop, and a full-screen capture.
**Not tested:** mobile, other resolutions, a published server.
