# Map + Combat — merge project notes

Last updated: 2026-09-15 · Merge complete, error-fix pass done, PlayerStats bridge + WorldInfo HUD wired, floating-map fix done (unsaved until you save the place).

## What this project is

Merging three Roblox Studio places into one:

| Place | placeId | Role |
|---|---|---|
| **Map + Combat** | 87872916277829 | **Target.** The merged place. |
| combat test | 97112397628420 | Source of the **combat framework**. |
| jjk combat game | 73903928733822 | Source of the **map** and **UI**. |

Owner group: Guppy's Shop. Studio account: cooljayzfalt.

## How to move things between places

Studio's clipboard does **not** work across separate Studio processes — Paste is greyed
out. The working method is file-based:

1. In the SOURCE place, set the Explorer selection (fastest via Luau:
   `game:GetService("Selection"):Set({...})`).
2. Right-click a selected row **in empty space to the right of the label**
   (around x=1360 in a maximized 1456-wide window). Right-clicking directly on
   the label/icon often fails to open the menu.
3. `Save / Export` → `Save to File...` → type the full path into the File name box.
4. In the TARGET place, right-click the destination parent →
   `Insert` → `Import Roblox Model` → pick the .rbxm.

Window switching: hover the Roblox Studio taskbar icon, then click the thumbnail
for the place you want.

Gotcha: Studio's file dialog is sometimes spawned as a separate `nativedialog.exe`
process. When that happens, automation can't type into it — reopen the dialog and it
usually comes back in-process.

The `.rbxm` files in this folder are reusable — re-import any of them to redo a step.

| File | Contents | Destination |
|---|---|---|
| `SS_ServerStorage.rbxm` | Assets (M2, Vows), Packages, Services.AirWeld, CSProtocols, UIPresets | `ServerStorage` |
| `RF_ReplicatedFirst.rbxm` | Preloader + Packages.AnimationLoader (with R6 Rig) | `ReplicatedFirst` |
| `SSS_Cmdr.rbxm` | Cmdr admin/debug commands | `ServerScriptService` |
| `SSS_Services_Studio.rbxm` | Services.Studio (Game + Dummy config) | `ServerScriptService.Services` |
| `WS_CombatFolders.rbxm` | Visuals, Map, Live, Moves, Animations, Flashstep | `Workspace` |
| `MAP_1_Shibuya.rbxm` | Shibuya (248k) | `workspace.Map` |
| `MAP_2_Districts.rbxm` | JJK School Map, SecondWard, Brick Estate, Lab, Lab2, Warehouse, Raccoon's Cafe (322k) | `workspace.Map` |
| `MAP_3_Rest.rbxm` | the other ~109 workspace items (68k) | `workspace.Map` |
| `LIVE_jjk_NPCs.rbxm` | jjk NPCs + Alive folders | `workspace.Live` |
| `SG_CoreUI.rbxm` | Player_Display, Custom Inventory, SkillTree, CombatGui, Vignette, InteractUI, Transition | `StarterGui` |

`SG_Hud.rbxm` (the jjk Hud) is in Downloads from an earlier session.

## Architecture of the combat framework (from combat test)

Loader-based. `ServerScriptService.Loader` + `StarterPlayer.StarterPlayerScripts.Loader`
boot the system; `ReplicatedStorage.ModuleLoader` resolves modules.

### Hard structural requirements — these WILL throw if missing

`ReplicatedStorage.Packages.bLua.Paramaters` builds raycast params with
`FilterType = Enum.RaycastFilterType.Include`:

- `workspace.Map` — **the 'map' raycast preset only hits descendants of this folder.**
  Map geometry outside `workspace.Map` is invisible to ground checks, movement and
  hit detection. **This is why all jjk map content was parented under `workspace.Map`.**
- `workspace.Live` — the 'live' preset; also where `Services.Managers.EntityLoader`
  and `Services.AI.AI` parent spawned characters/NPCs. jjk's `NPCs` and `Alive`
  folders were put here rather than under Map, so map raycasts don't hit bodies.
- `workspace.Visuals` — VFX parent, hard-indexed by AnimationLoader, Feint,
  FastSprint, Dodge, SlideVFX, CriticalIndicator, Visuals.Blood.
- `workspace.Ignore` — referenced by `ReplicatedStorage.Functions.LinearVelocity.Linear`
  line 100. **Did not exist in combat test either — latent bug.** An empty Folder
  named `Ignore` was added here to prevent the throw.
- `ServerScriptService.Services.Studio.Game` — required by `Services.Server` and
  `Services.Managers.EntityLoader`. A `Configuration` with ~24k descendants.
- `ServerStorage.Packages` — HitBox, DamageLogic, DamageTypes, Trails, SimplePath,
  AirCombat. Required by every M1/M2/Uppercut/Skill module.
- `ServerStorage.Assets.M2` — per-weapon M2 modules, dispatched by `script.Name`.
- `ServerStorage.Assets.Vows` — required by `Services.Features.Vow`.

### Client scripts belong in StarterPlayerScripts
`Controllers`, `Loader`, `VioletSkillsClient` go in **StarterPlayerScripts**.

Luau **cannot** reparent these into StarterPlayerScripts — it fails with "has
additional values for the Capabilities property: LoadUnownedAsset". Use Studio's
Edit > Cut then Edit > Paste Into instead (the clipboard works within one process).

## Final state of Map + Combat

- `workspace.Map` — 637,886 instances (the whole jjk map), plus combat test's
  Baseplate/LightingSettings.
- `workspace.Live` — combat test dummies (Rig, Parry, ParryTrade, Inf Dummy,
  Attacking, PostureBreak, Blocking) + jjk NPCs + Alive.
- `workspace` also has Visuals, Moves, Animations, Ignore, Flashstep, Terrain.
- `StarterGui` — Hud, Player_Display, Custom Inventory, SkillTree, CombatGui,
  Vignette, InteractUI, Transition.
- All 23 structural dependency checks pass.
- Play test: map loads, character spawns in the jjk map, UI renders,
  `[Animations] Loaded (164/164)`. No Lua errors at boot.

### Spawns
Six SpawnLocations existed after the merge; `workspace.Map.SpawnLocation` (combat
test's baseplate spawn) was **disabled** so players don't randomly land on the empty
baseplate. Still enabled: 4 × `Spawns1` (y≈505, in the jjk map) and `SpawnBox`
(at 31360, 1964, -13850 — jjk's lobby/holding box). Decide which of these you
actually want.

### Deliberately skipped
- `M2Test_TEMP`, `EntityLoaderTest_TEMP` — temp/scratch modules in combat test.
- `ServerStorage.__ReclassTemp` — empty scratch folder.
- combat test's own `StarterGui` (GameUI, Vingette) — UI comes from jjk instead.
- jjk `Terrain` — checked, it has **no voxels** (MaxExtents is the empty signature),
  so there was nothing to transfer.
- jjk's other 26 ScreenGuis and its StarterPlayerScripts/StarterCharacterScripts.

## Error-fix pass (2026-09-15)

### Fixed

1. **86 broken map script paths.** Nesting the map under `workspace.Map` broke every
   teleport script that indexed `game.workspace.Shibuya...`. Rewrote 84 scripts
   (85 replacements): `workspace.X` -> `workspace.Map.X`, and `workspace.NPCs` /
   `workspace.Alive` -> `workspace.Live.X`. Rewrite was identifier-boundary safe and
   skipped names that still exist at the workspace root (`Visuals`, `Ignore`), so jjk's
   duplicates now correctly resolve to the combat framework's folders.

2. **The spawn bug.** All four `Spawns1` SpawnLocations were floating at y≈505 with no
   ground within 300 studs — players spawned and fell. Raycast down to the real Shibuya
   ground and repositioned them (y≈14.7, one at 20.7), anchored. Also disabled the
   `SpawnBox` spawn (a sealed box at 31360, 1964, -13850, ~31k studs from the map), and
   deleted combat test's leftover `Baseplate` (a 512x512x20 opaque slab at the origin
   that was embedded in the Shibuya streets) plus its SpawnLocation.

3. **`GameUI` was missing — this broke combat's whole UI layer.** Five framework
   controllers (`Gui.HealthBar`, `Gui.Hotbar`, `Gui.Inventory`, `Gui.HotbarDrag`, and
   `HotbarAssignment`) do `PlayerGui:WaitForChild("GameUI")`. Because combat test's
   GameUI was deliberately left behind, they all hung forever and the hotbar, inventory
   and health bar never initialized. Imported `GameUI` -> `SG_GameUI.rbxm`.

4. **`ReplicatedStorage.Packages.chrono` was missing.** Required by
   `StarterPlayerScripts.Loader` — the client bootstrap. Restored.

5. **`ReplicatedStorage.Packages.Visuals.Rocks` was missing.** Required by five
   StateController modules (DownSlam, FastSprint, CriticalIndicator, Skills.Template
   OnHit/OnActivate). Restored. Both via `RS_Packages_chrono_Rocks.rbxm`.

   Note: ReplicatedStorage was never part of this merge — it predated it and was
   incomplete. A 119-path audit against combat test now shows **zero** missing paths.

### Verified after the fixes
Play test: player spawns standing on a Shibuya street, GameUI health bar reads 100/100,
hotbar slots 1-9 render, `[Animations] Loaded (164/164)`.

## The one blocking decision: `ReplicatedStorage.PlayerData`

This is a hard name collision and it is the single root cause of every remaining
game-side error.

- The **combat framework** needs `ReplicatedStorage.PlayerData` to be a **ModuleScript**.
  `Functions.UpdateState`, `Services.Player.Data` and `Services.Managers.EntityLoader`
  all `require()` it.
- **jjk's UI** needs `ReplicatedStorage.PlayerData` to be a **Folder** holding one
  subfolder per player (`Faction`, `Money`, ...). Seven scripts want this:
  `Hud.HudClient`, `Hud.ProgressionService`, `ProgressionManagement`,
  `Player_Display.Handler`, `Custom Inventory.inventoryHandler`, `SkillTree.STClient`,
  `InteractUI.Ryo.LocalScript`.

One name cannot be both. Related jjk-only dependencies that are also absent:
`character.AliveData` (a Folder on jjk's StarterCharacter holding Combat/Buffs/CDS/
TempValue/PermValue), `ReplicatedStorage.Remotes.LoadGui`,
`ReplicatedStorage.Remotes.Progression`, `ReplicatedStorage.Assets.Ui`
(Map + Combat has `Assets.UI`, different case, without the Equip templates),
`ReplicatedStorage.Modules.*`, `ReplicatedStorage.BlackHole.*`.

Symptoms today: `AliveData not found for character` (Handler:95) and an infinite-yield
warning at `InteractUI.Ryo.LocalScript:12`. The jjk UIs render but show no live data.

### Options
- **Bridge (recommended).** Keep the framework's `PlayerData` ModuleScript. Add a
  separate Folder (e.g. `ReplicatedStorage.PlayerStats`), repoint the seven jjk scripts
  at it, add `AliveData` to StarterCharacter, create the missing RemoteEvents and Ui
  templates, and add one server script that mirrors the framework's profile into those
  values. jjk's HUD becomes genuinely live.
- **Port jjk's data layer.** Bring over jjk's `Modules`, `BlackHole`, `Remotes` and the
  server scripts that populate them. Highest fidelity to jjk, but you end up with two
  competing damage/posture/data systems.
- **Trim.** Delete or disable the jjk UI scripts that need the absent systems and rely
  on `GameUI` for health/hotbar/inventory. Zero further work, loses those features.

## PlayerStats bridge + WorldInfo HUD (2026-09-15)

The `ReplicatedStorage.PlayerData` collision was resolved with the **bridge** option.

### What was built

`ReplicatedStorage.PlayerStats` (Folder) is now the home for jjk's per-player values.
The framework's `ReplicatedStorage.PlayerData` ModuleScript is untouched, so
`Functions.UpdateState`, `Services.Player.Data` and `Services.Managers.EntityLoader`
still `require()` it exactly as before.

Eight scripts were repointed from `PlayerData` to `PlayerStats`:
`Hud.HudClient`, `Hud.ProgressionService`, `ProgressionManagement`,
`Player_Display.Handler`, `Custom Inventory.inventoryHandler`, `SkillTree.STClient`,
`InteractUI.Ryo.LocalScript`, and `Workspace.Live.Alive.Dummy5.M1`.

**New: `ServerScriptService.PlayerStatsBridge`** — creates `PlayerStats[player.Name]`
on join with Level, LevelExp, Experiance, Rolls, Money, Points, AuraColorR/G/B, Lives,
Faction, Title, FirstName, Clan, Region and an Inventory folder; clones AliveData onto
any character missing it; fires `Remotes.LoadGui`; cleans up on leave. **Money mirrors
the framework profile's `Yen` live** (1s poll - the profile is a plain table with no
change signal). Everything else is a jjk-only concept with no framework equivalent and
starts at a sensible default - that script is where to wire real progression later.

**New: `ServerScriptService.WorldInfoServer`** — publishes `ReplicatedStorage.ServerStartTime`
and resolves each player's region via `LocalizationService:GetCountryRegionForPlayerAsync`
(server-only API) into `PlayerStats.Region`.

Also created: `StarterCharacter.AliveData` (Combat / Buffs / CDS / TempValue / PermValue /
Talents / CarriedPlr, matching jjk's shape), `Remotes.LoadGui`, `DevProductNotif`,
`PurchaseSkill`, `Assets.Ui` (empty - note it is *distinct* from the framework's
`Assets.UI`; Roblox names are case-sensitive), and `Assets.InteractHiglight`
(a Highlight, jjk's spelling, matching jjk's properties).

`Hud.HudClient` had three dead requires removed (`Modules.Kagune.Kaguneinfomation`,
`Modules.Notification`, `BlackHole...Talents.Functions`) - all three were assigned and
never used, so nothing from jjk's `Modules` or `BlackHole` folders needed porting.

### WorldInfo HUD (Version / Character Name / ServerUptime / ServerRegion)

Ported `StarterGui.WorldInfo` from jjk (`SG_WorldInfo.rbxm`) and rewrote
`WorldInfoClient` to drive all four readouts from one script:

| Readout | How it works now |
|---|---|
| Version | `game.PlaceVersion` - shows "V. dev" in Studio, "V. <n>" in a published server. jjk used a `MarketplaceService:GetProductInfo().Updated` web call from a **server Script inside StarterGui, which never ran**. |
| Character Name | `PlayerStats.FirstName` + `Clan` ("Clanless" shows the bare name), Rank from `Title`, Lives from `Lives`. FirstName is seeded from the player's display name. |
| ServerUptime | Derived locally from `ReplicatedStorage.ServerStartTime`. jjk polled a `Timer` RemoteFunction **four times a second from every client**; that RemoteFunction doesn't exist here anyway. |
| ServerRegion | From `PlayerStats.Region`, filled by WorldInfoServer. jjk called the server-only LocalizationService API from a LocalScript. |

Two now-redundant child scripts were deleted: `Main.AgeInfo.ServerAge.LocalScript` and
`Main.GameInfo.GameVersion.Script`.

`WorldInfo.LeekPrevention` came across with the GUI - it is jjk's anti-leak watermark,
36 labels stamped with the viewer's username at 93% transparency. Left as-is.

### Verified in a play test
Player spawns on a Shibuya street. Version "V. dev", ServerUptime ticking ("00:00:58"),
ServerRegion "US", character name + "[Novice]" rank, Yen panel, jjk's health/stamina/
posture bars, and GameUI's 100/100 health bar and hotbar - all rendering together.
Console is clean apart from the asset-permission warnings below.

### Note on HUD overlap
GameUI (combat test) and Player_Display / Custom Inventory (jjk) both draw a health bar
and hotbar. Both are left visible by choice - GameUI is what actually drives combat
input. Decide the final look when you want to.

### Incidents during this pass
- A stray Delete keystroke landed in the Explorer while `GameUI` was selected and
  deleted it; it was re-imported from `SG_GameUI.rbxm` and verified present.
- Studio lost its connection to Roblox during a play test (`Error Code: RCC-288`) and
  closed the place. Nothing was lost - the place had been saved, and reopening confirmed
  every change persisted.

## Floating models / shops / locations fix (2026-09-15)

### Root cause
**`Import Roblox Model` moved each import up in the air.** Studio places an import
as one group and lifts it to avoid overlapping what is already there. Checking the
exported `.rbxm` files against the place showed:

| File | Shift on import | Effect |
|---|---|---|
| `MAP_1_Shibuya.rbxm` | none | fine |
| `MAP_2_Districts.rbxm` | none | fine |
| `MAP_3_Rest.rbxm` | **+485.635 Y** | School, CityHall, ClothesShop, Rosewald Hotel, MapBuilding, Full Construction, Mountain, the cafe props, teleport pads, VFX parts - all hanging over Shibuya |
| `LIVE_jjk_NPCs.rbxm` | **+614.38 Y** | GhoulProgCheckNPC, InfDummy, Dummy0-5 all floating |
| `WS_CombatFolders.rbxm` | +19.7 Y | left alone - it happens to put the combat test dummies on the Shibuya street |

X and Z were never changed, only height. The old "Spawns1 floating at y≈505" bug
was the same shift.

### What was done
- Every `workspace.Map` child that came from `MAP_3_Rest` (all except Shibuya,
  the district folders, Raccoon's Cafe Model and Spawns1) was moved down 485.635 using
  `PivotTo` for models and `CFrame` for loose parts. All bounds now match the export
  to 0.1 studs.
- The eight jjk NPCs in `workspace.Live.NPCs` / `Live.Alive` were put back at
  their exact original root-part heights.
- `Spawns1` was left where the earlier raycast fix put it (on the street).
- Both moves are single undo steps: "Drop MAP_3 items to original height" and
  "Drop jjk NPCs to original height".

Most of the MAP_3 buildings are **interiors that sit under the city** (y≈-100
to -275, and the cafe set near z≈-4200). Players reach them through the teleport
parts, so they are not meant to be seen from the street. All teleport scripts use
the pad's own position (none use fixed coordinates), so they work again.

### Verified
- A camera shot over Shibuya no longer shows buildings in the sky.
- In a play test the player spawned on the street; after a teleport to
  GhoulCafeTeleport1 the player landed on the Ghoul Cafe floor (y≈-272).
- No new console errors.

### Open items found
- **13 MAP_3 items are not in the place at all:** Fridge, Stove Oven, 3 × Cabinet,
  6 × small `Model` (x≈-791, z 1098-1171 - the kitchen inside the `Model` at
  -768,-99,1119), plus 2 loose Parts at (1146,-111,-1593) and (-53,-112,-1593).
  To restore them, re-import `MAP_3_Rest.rbxm` into a temporary folder, move those
  13 into place (then shift down 485.635), and delete the rest.
- `GhoulProgCheckNPC` has no floor within 50 studs at its original spot
  (-115,-158,-276). It was the same in jjk, so it is probably meant to be a hidden
  check NPC.
- Existing errors, not caused by this fix: `Lab2.EnterPart/ExitPart.Script` (`Triggered`
  is not a member of Part), `ReplicatedStorage.Weather` is missing (WeatherManager),
  `Remotes.Progression` and `Remotes.CombatTag` are missing.

## Still outstanding (not code)

- **Asset permissions.** Not fixable from Studio — share these with the Map + Combat
  experience from each asset's page: 125256362480612, 92528293749807, 96689332245371,
  93246076242972, 81380537724452, 135836034783446, 104029923580343, 71086651442213,
  plus SurfaceAppearance ColorMaps 14580622165, 12309761624, 4872978666, 4872978561,
  4872978811, 4872978726. (71086651442213 already errored in combat test pre-merge.)
- **Two pre-existing jjk bugs**, not caused by the merge — verified missing in jjk too:
  `workspace.CochleaOutside` (PoliceStation Teleport4) and `workspace.SmallClouds`
  (WeatherManager).
- `[Fusion] Detected an infinite loop` in the log is your **VFX Studio plugin**
  (`user_VFX.rbxmx`), not the game.

## Original post-merge issue list (superseded by the section above)

1. **`AliveData not found for character`** (Client – Handler:95). jjk's
   `Player_Display.Handler` expects a jjk PlayerData structure that doesn't exist
   here. Same class of problem will hit Custom Inventory / SkillTree / CombatGui —
   they reference jjk RemoteEvents and PlayerData. Either port jjk's PlayerData
   layer or rewire these to the combat framework's `ReplicatedStorage.PlayerData`.
2. **Asset permission errors.** These asset IDs are not shared with the
   Map + Combat experience: 125256362480612, 92528293749807, 96689332245371,
   93246076242972, 81380537724452, 135836034783446, 104029923580343,
   71086651442213, 1230976162… (a SurfaceAppearance ColorMap). Fix from the asset's
   page → share access with this experience. Note 71086651442213 already errored in
   combat test before the merge, so not all of these are new.
3. **jjk map scripts index workspace paths that moved.** `WeatherManager`,
   `GhoulCafeTeleport1/2`, `Teleport Pads`, `TeleportParts` were written against
   `workspace.X` and are now at `workspace.Map.X`.
4. Combat test's `Baseplate` still sits at the origin under the jjk map. Harmless,
   but delete it once you're happy.

## Ghoul script cleanup (2026-09-15)
- Deleted `Map.GhoulCafeTeleport1.Script` and `Map.GhoulCafeTeleport2.Script`. The two pad parts are still there but no longer teleport anyone.
- Reworded a comment in `PlayerStatsBridge` so it no longer says "Ghoul". The code is unchanged.
- Left in place on purpose, since they still mention ghoul but other code depends on them: `Player_Display.Handler` (the jjk HUD bars),
  `Hud.ProgressionService.Modules.ProgressionRankRequirements` and `ProgressionTitleRequirements`.
- `Live.NPCs.GhoulProgCheckNPC` (a model, not a script) is still there.

## World systems: roadmap #4 / #5 / #7 foundations (2026-09-15)

Low-impact, self-contained groundwork. Nothing here touches the combat framework, `PlayerData`, or
`workspace.Map`/`workspace.Live` (so map/live raycasts never hit it). Source copies are in
`world_systems/` in this folder.

### Where things live
| Path | What |
|---|---|
| `workspace.WorldMarkers` | Authoring markers (visible in Edit, hidden in Play). Folders: `POI` (29, copied from Shibuya's PointLocations, tag `WorldPOI`), `DangerZones` (3, tag `DangerZone`), `Secrets` (3 rooftops, tag `WorldSecret`), `EventSpots` (34 auto-scanned open street spots, tag `EventSpot`) |
| `workspace.WorldNPCs` | 3 sample NPCs (tag `DialogueNPC`): Street Informant (Arcade), Shopkeeper (Yum Yum / Convenience Store), Trainer Goro (Fight Club). They're grey R6 rigs with tinted clothes, so they need proper outfits later |
| `workspace.WorldEvents` | Created at runtime; live event objects |
| `ReplicatedStorage.World` | `WorldConfig` (all tuning), `Dialogues` (dialogue trees), `Remotes` (Notify, ZoneChanged, OpenDialogue, DialogueAction, DialogueClosed, Waypoint) |
| `ServerStorage.WorldSignals` | BindableEvents for future systems: `Discovered`, `ZoneChanged`, `NPCAction`, `GrantReward`, `EventStarted`, `EventEnded`, `ForceEvent` |
| `ServerScriptService.World` | `WorldService`, `NPCService`, `EventService` (Scripts); `WorldQuery`, `Notify` (modules) |
| `StarterGui.WorldUI` | `WorldUIClient`: zone banner, toasts, dialogue box, waypoints (Balthazar titles, like jjk's UI) |

### #4 Open World
- Entering a DangerZone sets player attributes `DangerZone` / `DangerLevel` and shows a banner. Zones:
  Port Island (3), Old Town Backstreets (2), Kanon Park (1). These are placeholders, so resize or retag them in Studio.
- Walking near a POI or Secret for the first time shows a toast, adds a BoolValue to
  `PlayerStats[<name>].Discovered`, and fires `WorldSignals.Discovered`. Per session only, because nothing is saved yet.
- To add a marker: add a Part to the right folder with the matching tag. Attributes: `DisplayName`,
  `DangerLevel` (zones), `DiscoverRadius`, `Hint` (secrets).

### #5 NPC System (dialogue shell)
- Any Model tagged `DialogueNPC` with attribute `DialogueId` gets a Talk prompt and a name tag
  (optional `DisplayName`, `Title` attributes).
- Trees are in `ReplicatedStorage.World.Dialogues`. Choice `Action`s run on the server
  (`NPCService` ACTIONS table). The client only sends the node and choice index, so it can't trigger arbitrary actions.
- Actions now: `RevealPOI` (on-screen waypoint), `OpenShop` and `Train` (placeholder toast, and they fire
  `WorldSignals.NPCAction` for the future shop and training systems).
- The dialogue closes when you walk more than 18 studs away or die. Proximity prompts are hidden while it is open.

### #7 Random Events
- Every 4-8 min on live servers (25-40 s in Studio), up to 2 at once, at an unused EventSpot (10-min spot cooldown).
  Announced to everyone with the nearest POI name.
- `CursedSurge`: an orb you absorb by holding the prompt for 1.5 s. `SupplyCache`: a crate you open by holding for 2 s.
  `Whisper` (rare): broadcasts one secret's hint. Unclaimed events vanish after 150 s.
- Rewards fire `WorldSignals.GrantReward(player, {Source, Id, DisplayName, Amount})`. Until loot (#9) listens,
  a placeholder toast says what you'd have gotten (`WorldConfig.Events.PlaceholderRewardNotice`).
- New event: add an entry to `WorldConfig.EventPool` and a function with the same Id to `RUNNERS` in EventService.
- Admin/testing: `game.ServerStorage.WorldSignals.ForceEvent:Fire("SupplyCache")` from a server script or Cmdr.
  (The Studio MCP code runner can't fire it because of the Capabilities sandbox.)

### Verified in play test
Old Town zone banner and attributes, FightClub and SushiRestaurant discovery, informant prompt, dialogue box with 4 choices,
choice clicks moving through the nodes, RevealPOI waypoint ("Fight Club 113 studs"), dialogue closing when you walk away,
two events spawning on schedule, opening a Supply Cache giving the reward toast. No new console errors.

### Tooling note
Creating scripts with `execute_luau` fails with the Capabilities error. Use the `multi_edit` tool with
`className` to create new scripts. Instances created by `execute_luau` also can't be fired from `execute_luau`.

## No-asset shortlist build (2026-09-15, later)
The full status table lives in the claude.ai project doc `claude/no-asset-shortlist.md`. This section is the code map.
Rule from Jay: **tell him what an idea is before building it.**

### Saved data (F-01)
- `Services.Player.Data` now uses ProfileService (store `PlayerData.STORE_NAME` = `PlayerData_v1`, `Reconcile` on load).
- New `PlayerData` fields: `BurialTags`, `Origin`, `TumorGrade`, `SmokeType`, `StatXP`, `AttributePoints`, `Discovered`,
  `HellEntrances`, `HellEntrancesRewarded`.
- Studio saves only with *Game Settings → Security → Enable Studio Access to API Services*; otherwise it's a mock store.

### Smoke (C-02, P-01, S-10) — `ServerScriptService.World`
- `SmokeService` (grade roll, regen, dry window; no regen while a form is up), `Smoke` (API: get/spend/add/rerollGrade),
  `ReplicatedStorage.World.SmokeConfig` (grades), `StarterGui.SmokeGui.SmokeClient` (bar, buffs, tag counter, gun ammo).
- Types (focus list only: Split, Gun, Curse, Regeneration/Strength, Mushroom, Lizard, Dinosaur):
  `ReplicatedStorage.World.SmokeTypes` (tuning) → `SmokeMoveService` (roll, gives move Tools with attribute `SmokeMove`,
  validates `Remotes.SmokeCast`) → `SmokeMoves.<Type>` handlers, helpers `SmokeMoves.Kit` and `SmokeMoves.Forms`.
  Client: `StarterPlayerScripts.SmokeMovesClient`.
- `DamageLogic.Processed` scales damage once per hit by attacker `StrengthMult` × `FormStrength` and target `DamageTakenMult`.

### Economy / NPCs (N-01, N-02, E-12)
- `World.Shops` (catalog; per-shop `Currency` Yen or Tags), `World.Buffs`, `EconomyService` (ShopBuy, GrantReward → Yen/Tags/Smoke,
  Studio-only top-up to ¥1000 + 40 tags), `StarterGui.ShopGui`.
- NPCs: Shopkeeper, Diner (Mama Okame, FastFood POI, safe zone), Gyoza Cook (Sushi POI), Grave Keeper (Hospital, takes tags).
- No NPC attribute training (Jay). `AttributePoints` is for the future character tree UI.
- Safe zones: `WorldMarkers.SafeZones` (tag `SafeZone`) → player attr `SafeZone` → `DamageLogic` refuses hits.

### World / events (W-11, W-03, E-01, W-09)
- `AtmosphereService` (toxic rain effects) + `StarterPlayerScripts.AtmosphereClient` (slum mood, rain visuals, Blue Night sky, Hell tint).
  `Workspace.Map.WeatherManager` is **disabled** (its `ReplicatedStorage.Weather` data is missing).
- `EventService` new events: `ToxicRain`, `BlueNight` (fist-only zombies via NPC config `M1Only`, burial tag drops). `EndEvent` signal.
- `HellService`: 8 tagged `HellEntrance` props (2 toilets, 2 dumpsters, 4 sewer covers) → `workspace.Hell` (part-built mud plain at
  -6000, 400, -6000, covered by the `Hell` safe zone). Saved per-player entrance progress; all 8 pays 5 tags + ¥500.
- AI: `AI.Spawn` now honors `Config.Weapon`; `NPCController` supports `Config.M1Only` and `Config.TargetAttribute`.
- AI crowds (2026-09-23): `Services.AI.AI.NPCSwarm` - separation steering (NPC<->NPC collision stays off), surround
  slots (attack ring 4 studs, waiting ring 10+ studs that slowly orbits), rotating attack tokens (2 per target for full AI,
  3 for `M1Only`; `Config.MaxAttackers` overrides), and 0.35 s swing spacing between different NPCs. Replaces the
  "Agro" TagService tag (it leaked a never-expiring tag per NPC per frame), so `Mobs.clearAgro` / QuestService's
  `clearAgro` are now no-ops. Optional config: `MaxAttackers`, `EngageRadius`, `SwingRange` (M1Only, default 6).
  Pre-change scripts: `ServerStorage.Backup_AI_2026-09-23`.
- AI fairness (2026-09-23, later): `NPCBehaviors.PerceivedAction` - NPCs see the target's action only after a
  0.18-0.3 s reaction time (`Config.ReactionTime`); guard budget of 3 block/parry/dodge/evasive, +1 per 1.6 s
  (`Config.GuardMax`); Evasive out of hitstun only after 3 hits in one combo (`Config.EscapeAfterHits`).
  Wind-ups: `NPCController.StartWindUp` fires client `StateController.Indicators.WindupIndicator` (glow) before
  M1Only punches (0.4 s, `Config.WindUp`), full-AI string openers (0.22 s) and uppercuts (0.38 s); a stun cancels it.
  Pathfinding: `NPCNavigation` paths (PathfindingService) when map geometry blocks line of sight, the height gap
  is > 6 studs, or the NPC is stuck; otherwise swarm steering.
- Combat flow (2026-09-23, players and NPCs; pre-change copies in `ServerStorage.Backup_Combat_2026-09-23`):
  - Input buffer: remote inputs go through `ActionManager:Request`; an input pressed while another action plays is
    kept `BUFFER_TIME` (0.6 s) and fired when that action ends. Releasing Block drops a buffered Block.
  - Cancel windows: Dodge/Block can cancel an Attack-class action once its hitbox went out (`HitBox.LastSwing` vs
    `ActionManager.ActionStartedAt`). A cancelled M1 finisher still pays its 1 s cooldown (`M1Finisher`).
  - Block/Sprint remotes only accept State `Activate`/`Release`/`Cancel` (it used to call any method by name).
  - Posture recovery (`DamageLogic`): 2 s after the last block hit, posture drains 12/s (half while blocking).
  - Perfect dodge: fires the client `Misc.Perfect Dodge` effect (existed, was never fired), Dodge skips its
    cooldown, and the dodger's next hit is a counter (x1.3 damage/posture). Removed the debug `warn(true)`.
  - Soft lock-on: `ReplicatedStorage.Functions.SoftLock.Face`, called by the M1/M2/Uppercut inputs; 9 studs, 75 deg
    cone around the camera, holds 0.3 s. Setting `SoftLock` (SettingsService + CharacterTreeClient row).
  - Combo counter: `DamageLogic.TakeAction` fires `Misc.ComboHit` to the attacking player; client
    `StateController.Misc.ComboHit` shows "N HITS" from the 2nd hit ("COUNTER" on a counter hit), resets after 1.6 s
    or when you get hit.
  - Testing note: requiring `Remotes.UnPackets` from `execute_luau` on the Client makes a second Packet instance
    that throws `Packet:347 ... ResponseReads` on every server packet - harness noise, not a game bug.
- Smoke moves (2026-09-23): move Tools live in `player.SmokeMoves` (not the Backpack; `SmokeCast` only accepts
  tools from there) and `SmokeMovesClient` casts on Z/X/C without equipping - an equipped move used to fire on
  every left click with M1 and showed in the hotbar. A cast pressed mid-swing: the server
  (`ActionManager.HoldInputs`/`ClearBuffer`) holds it until the swing ends and drops queued M1s; the client sets
  player attr `SmokeCastPending` so held M1 pauses. Held M1 repeats only after 0.25 s (a click = one swing).
  `Player_Display.Handler` no longer re-enables the Roblox backpack.
- Hotbar keys 1-9 (2026-09-23): owned by `StarterGui.Custom Inventory` (ContextActionService). The combat-test
  `Controllers.Gui.Hotbar` returns early while `GameUI` is disabled - both toggled equip, so a key press
  equipped then unequipped.
- `Packets.Start` is rate-limited (RemoteGuard key `Start`, 1/s burst 3).
- Swing prediction (2026-09-23): `ReplicatedStorage.Functions.SwingPredict.M1` plays your ground M1 locally on
  click. Roblox is inconsistent about showing the server's copy of an animation the client already plays (no lag:
  never; 150 ms lag: yes), so both are handled: if the server's copy arrives it continues from our position and ours
  stops; otherwise server attr `State_Attacking` on + `Combo` == predicted hit keeps ours, a different hit stops it,
  `State_Attacking` off stops it, and no confirmation in max(0.6s, 0.25s + 4x ping) fades it out. The combo reset
  (1s) is latency-corrected, and within 0.12s + ping of it no prediction is made (a wrong guess = visible restart).
  Needs the character attr `Weapon` (set in `EntityLoader.SetWeapon`). Tested at 0 and 150 ms lag.
- Lighting (2026-09-23): `StarterPlayerScripts.WorldClient.AtmosphereClient` is the only ClockTime writer. The jjk
  free-model `Workspace.Map.Spawns1."Day/Night Cycle"` (full 24h every 12 min, 15 writes/s) is disabled - the two
  fought and flashed the sky to day every 2 s. Jay chose a full day-night cycle: Hole = one 24h day per 35 min,
  computed from `workspace:GetServerTimeNow()` so every player/server shares the time; updated every 1 s; darkness
  follows the clock (midnight darkest). BlueNight/GhostNight/Hell still override. Academy keeps its 9-15 range.
- Ghost toasts (2026-09-23): `UIThemeClient` added its outline/corner images to every panel, including self-sizing
  list frames; the toast's UIListLayout stacked them under the text and AutomaticSize grew it to 280x516 (Cling
  "No target.", burial tag pickups, every Notify). `isPanel` now skips frames with AutomaticSize or a layout.

### Overnight build 2026-09-24 - rep, tumor debuff, Blue Night contracts, devil trial (status)
Spec: `overnight-build-prompt.md` (tasks 1-5). Hard limits respected: no origin split, factions, bounty, masks or
Holey boss work; no new assets. Pre-change copies: `ServerStorage.Backup_Overnight_2026-09-24`.
SelfTest: **35/35** (8 new checks). Save schema changes are additive (defaults in `PlayerData.DEFAULT_PLAYER_DATA`,
filled by `Profile:Reconcile()`; `DataMigrations.sanitize` repairs bad values) - verified on a real save.

| Task | Status | Where |
|---|---|---|
| 1 En's Gang rep | **Done.** `EnGangRep` (+ `...EarnedAt`, `...DecayedAt`). Earned from Ashmask raiders (+2), captains (+5), breaking the sweep (+4) via `GrantReward.EnGangRep`. Decay: none for 24 h after the last gain, then -5/day (offline too), floor 0. | `World.ReputationConfig` (all tunables), `World.Reputation`, `World.ReputationService`, admin `rep` |
| 2 Rep gates | **Done.** Rat (BackAlley) needs rep 10: below it he gives a brush-off tree with no actions and the dialogue box shows "🔒 En's Gang won't vouch for you yet · rep x / 10". His Black Smoke x3 needs rep 30 and still costs ¥330 (server-checked; shop row shows the requirement and a lock). | `Dialogues` (`RepGate`, `BackAlleyLocked`), `Shops` (`RepGate`), `NPCService`, `EconomyService`, `ShopClient`, `WorldUIClient` |
| 3 Tumor debuff | **Done, entry is a placeholder.** `TumorType` Natural/Artificial + `TumorPlaytime`. Artificial = Smoke regen -40%, fading linearly to a permanent -10% over 10 h of playtime (chosen: it never fully clears). Purple "✚ Artificial tumor" tag over the head for everyone. **No transplant mechanic exists** - set with the `tumor` admin command / `WorldSignals.TumorAdmin`; a future transplant should call that. | `World.TumorConfig` (tunables), `World.TumorService`, `SmokeService` (regen × `TumorRegenMult`) |
| 4 Blue Night contract | **Done, effect undesigned.** "The Broker" (cloned rig, blue tint) exists only while `BlueNight` is on, at a random event spot. "Sign the book" pairs you with a waiting signer you've never been bound to (history saved forever, no repicks); the queue clears at dawn. Pact saved on both profiles + attr `ContractPartner`. **Needs design input:** what a pact does, how long it lasts, can it be broken. | `World.Contracts` (rules), `World.ContractService`, dialogue `Contractor` |
| 5 Devil trial | **Scaffold only (as specified).** 4 stages (Summons, Combat Gauntlet, Armor Training, Judgement), all TODO content. Entry: Madame Ise → "I want to become a devil." Fail state: dying mid-trial resets to stage 1 (attempts +1). Stage content should call `WorldSignals.DevilTrial:Invoke(player, "advance" / "fail")`. | `World.DevilTrialConfig` (stages + TODOs), `World.DevilTrial`, `World.DevilTrialService` |

Testing notes: tool-driven playtests can't trigger ProximityPrompts (prompts never show while Studio isn't the
focused window), so `NPCService` has a Studio-only `WorldSignals.DebugTalk:Invoke(player, npc)` that runs the same
Talk path. Pairing with a real second player is only covered by the pure-rule SelfTest (Studio playtests have one
player) - worth one 2-player test on a live server. Jay's Studio save kept test leftovers: En's Gang rep 30 and
5 Black Smoke (`rep set 0` resets the rep).

### 2026-09-24 day pass - Jay's picks "1-3 8-10" (HUD, combat depth, performance, cleanup, spawn warnings, fist reach)
Pre-change copies: `ServerStorage.Backup_HUD_2026-09-24` (+ `Backup_Overnight_2026-09-24`); archived objects in
`ServerStorage.Archive_2026-09-24` (each has `ArchivedFrom` / `ArchivedReason` attrs - move back to restore).
- **Cleanup (8):** archived `Map.WeatherManager`, `Map.Spawns1."Day/Night Cycle"` (the free-model clock that fought
  AtmosphereClient) and `StarterGui.Hud.ProgressionService`. `Visuals.Rocks` Crater no longer errors when
  `Assets.Effects.Slash2.Slam` is missing. `Hud.HudClient` is down to the DevProductNotif toast.
- **Slow-spawn warnings (9):** `Controllers.Gui.Camera` (Humanoid / HumanoidRootPart / Torso), `Gui.Hotbar` and
  `Gui.HealthBar` (`GameUI`) wait with timeouts and bail out instead of printing "Infinite yield possible".
- **Fist reach (10):** `Config.Weapons` Fist hitbox (4,5,4) -> (5,6,5): Fist was the only weapon that whiffed on a
  target the swing visibly touched. 3/3 hits at 6.0 studs in a playtest.
- **Performance (3):** `CastShadow = false` on 19,418 map parts under 1 cubic stud (tag `PerfNoShadow`; revert
  snippet in workspace attr `PerfNoShadowNote`). The 14 free-physics assemblies are all meant to move (claw machine,
  cafe doors), so nothing was anchored. Studio caps at 60 FPS, so the gain only shows on a live server / weak PC.
- **New HUD (1):** `StarterPlayerScripts.WorldClient.VitalsHud` draws HEALTH (current / max, red, fades darker as it
  drops) and GUARD (guard left = 100 - Humanoid attr `Posture`, only while posture > 0, pulses red from 75) directly
  under the SMOKE bar, same width / fonts / track style, above the equipment strip. `Player_Display` stays enabled
  because its Handler still owns the number-key tool hotbar, the combat-tag icon and the CoreGui backpack switch;
  only its `Player` frame (jjk health / stamina / posture bars + faction emblem) is hidden and the "Combat bars" block
  is gone. UIThemeClient skips `VitalsHud`.

### Combat depth (2) - 2026-09-24 (done; pre-change copies in `ServerStorage.Backup_CombatDepth_2026-09-24`)
Everything reuses the existing combat pipeline (DamageLogic.Processed / TakeAction, ActionManager Block, wind-up
tells, NPCSwarm tokens), so blocks, parries, i-frames and perfect dodges work against all of it automatically.
- **Parry payoff (riposte):** a clean parry (the 0.25 s window at the start of Block, not the AutoParry follow-up)
  already stuns the attacker 0.75 s; the parrier's own stun drops 0.4 -> 0.2 s and they get a 1 s `Riposte` tag.
  Their next landed hit in that window does x1.25 damage, shows as a counter on the combo counter and adds stagger.
- **Stagger meter** (Humanoid attr `Stagger`, 0-100, players and NPCs alike): built only by skill plays, never by
  plain damage - being parried +40, eating a riposte or perfect-dodge counter +30, taking an unblocked heavy (M2)
  +20, being hit out of your own swing / wind-up +12. Drains 15/s after 2.5 s without a gain. At 100: the guard-break
  stun + effect plays, and a short `Finisher` window opens - the next hit does x1.5 (heavy) / x1.25 (other).
  Your own stagger shows as a STAGGER row in VitalsHud while > 0.
- **Enemy archetypes** (`Config.NPCArchetypes`, used by `Mobs.spawn` via `spec.Archetype`):
  - *Shield* - holds block facing you (the raise has the normal parry window), drops it only for its own
    telegraphed swing. Break its guard (posture, heavies) or hit it from the side / back.
  - *Rusher* - fast, low health; from 9-16 studs it telegraphs and lunges into a swing.
  - *Thrower* - never takes a melee turn; waits on the outer ring and throws a stone every few seconds after a
    wind-up tell. The stone is a plain scripted ball (no asset) and can be blocked, parried or dodged.
  - *Flanker* - its melee slot is behind you, so it circles to your back; back hits ignore block.
  First user: the Ashmask Sweep's 5 grunts become Shield, Rusher, Thrower, Flanker + 1 plain raider.

Where: `ServerStorage.Packages.DamageLogic` (RIPOSTE_* / STAGGER / FINISHER_* constants at the top,
`Shared.AddStagger`, `Shared.Stagger`; `PostureBreak` now returns its stun length and falls back to 1.2 s when the
animation length reads 0), `ReplicatedStorage.Config.NPCArchetypes` (all archetype tunables),
`Services.AI.AI.NPCController` (`ShieldGuard`, `TryLunge` / `UpdateLunge`, `TryThrow`; NPCs set char attr `WindUp`
during a tell), `Services.AI.AI.NPCRanged` (the stone), `NPCSwarm` (NoMelee, flank slots; a raised guard doesn't
count as busy), `World.Mobs` (`spec.Archetype`), `EventService` (`ASHMASK_ROLES`), client `Misc.ComboHit`
("FINISHER" caption; counters show from the first hit) and `VitalsHud` (GUARD + STAGGER share one line).
Verified: SelfTest **38/38**, clean console. 3 new checks (parry -> riposte x1.25 +30 and attacker stagger 40; 105 stagger -> guard-break stun
+ one x1.5 heavy finisher; heavy +20 / light +0; thrower never gets a token, flanker sits directly behind). Live
Ashmask Sweep: the Slinger threw and hit (3.6 dmg per stone after armor), the Rusher lunged (WalkSpeed 48), the Stalker
spent its close time behind the player (avg facing dot -0.69), the Shieldbearer blocked front M1s (posture rose, chip
only) and took hits + interrupt stagger while its guard was down for its own swing.
Not verified live: a real parry into a riposte, and M2 guard-breaking the Shieldbearer - scripted mouse input can't
hold M2 / time a parry against a moving NPC reliably. Worth a hand test.
Follow-up idea: show the stagger of the enemy you're fighting (overhead bar or a target row in VitalsHud).

### Lamps, animation breakage, Academy sync - 2026-09-24 (details in work-log)
- Lamps: tags `StreetLamp` (lights, shadows) / `StreetLampBulb` / `LampGlow` (parts set Neon, originals in
  attrs `LampOrigMaterial`/`LampOrigColor`); `WorldClient.LampClient` turns street lamps on 17.6-6.4 and
  honours `Setting_Shadows`. `LightingStyle` Realistic; Hole night fill darker in AtmosphereClient.
- Animations: Perfect Dodge only stops Action-layer tracks; AnimationController restarts a stopped
  pose (watchdog); HitWeight skips Core. TrackService/SwingPredict now release finished tracks.
- Academy synced to Map + Combat: 525/525 game scripts identical (fight pit archived there too).
  The shared folders are unpublished Packages - publishing them makes the next sync one "Update All".

### React HUD - 2026-09-24 (both places; details in work-log)
- `ReplicatedStorage.ReactLua`: react-lua package 15621638430 v5, unlinked, with 9 rebuilt luau-polyfill
  modules and 14 link stubs changed to `require()` (the published package is broken without them).
- `ReplicatedStorage.HudUI` (Theme / Assets / HudState / Primitives / Widgets / App) +
  `StarterPlayerScripts.ReactHudClient` (mounts `PlayerGui.ReactHud`, feeds `HudState`, handles button actions).
  Widgets: health + Smoke bars, top-right Character / Settings / Leave icons, quest panel, party panel +
  invite popup, "In Combat" text.
- Data: Humanoid; Smoke attrs; `QuestSync` / `QuestAction`; party attrs `PartyMembersJSON` / `PartyLeader` /
  `PartyInviteFrom` + `Remotes.PartyAction` (PartyService); character attr `State_InCombat` (StateManager
  now mirrors `InCombat`); `CharacterGui.Open` BindableEvent (CharacterTreeClient).
- Replaced (hidden, not deleted): VitalsHud HEALTH row, SmokeGui bar + ammo line, QuestClient tracker
  (`SHOW_TRACKER`). VitalsHud GUARD / STAGGER and SmokeGui buffs sit above the new bars.

### Next up
- **Performance** (measured in Studio 2026-09-23): client 60 FPS steady, worst frame 20 ms, ~131k instances
  streamed in; server heartbeat 4.2 ms avg / 11 ms worst on a 541k-instance workspace. 101 unanchored map parts
  (Brick Estate 50, Shibuya 37, Raccoon's Cafe 13) simulate physics every frame - check whether any are meant to
  move before anchoring. 197 Workspace scripts are all event-driven (no loops). Studio memory numbers include the
  editor, so real per-client memory needs a live-server check.
- Smoke casts pressed mid-swing wait up to 1 s (`SmokeMoveService.SWING_WAIT`, client `QUEUE_TIME`) - 0.6 s was
  shorter than a Fist swing (0.67 s) and let the cast out just before the swing ended.
- `StateManager` states are per-machine memory - clients never saw server states (Attacking, Stun, ...), so every
  client-side `HasState` check on a server state was dead code. The server now mirrors Stun, ParryStun, Ragdoll,
  Paralyzed, Attacking, SkillInUse, Freeze, Dodge as character attrs `State_<Name>`, and client `HasState` reads them.
- `Config.Weapons` M1 lists are sorted by numeric name (`ordered()`); `GetChildren()` came back 3,4,1,2,5 on the
  client for Fist. Server order was already 1..n, so combat is unchanged.
- SelfTest section 3b covers the 2026-09-23 combat/AI work; run with `workspace:SetAttribute("RunSelfTest", true)`.
- Studio Play wedge fix: `scratchpad/studio_f5.ps1` (AttachThreadInput -> verify foreground -> F5 -> restore focus).
- Lawless zones (W-30, cut down): zones at danger level >= `WorldConfig.Law.LawlessLevel` set `Lawless`, pay x1.5 Yen/Smoke,
  and show "Controlled by the Eyeless" on the zone banner. No police, guarded areas or Wanted system (Jay).
- `DoorService` (W-02): 7 `SmokeDoor` doors in `workspace.SmokeDoors`; walk past to unlock (saved `Doors`), hold E for the
  travel list (`StarterGui.DoorGui`); toll ¥30, or ¥60 to Eyeless doors; Sorcerers only.
- `GrudgeService` + `StarterPlayerScripts.GrudgeMonumentClient` (W-31): part-built `workspace.GrudgeMonument` in the Crossing
  plaza. Staring at its eyes as a Smoke user (90 studs, 1.5s) tints the screen red, then Stun 1.2s and -20 Smoke (6s cooldown).
  Config: `WorldConfig.Grudge`; remote `World.Remotes.GrudgeGaze`. (Jay doesn't want it; remove later.)
- Smoke (2026-09-16): no ranks, no Smoke XP. `SmokeBase` is a flat 200 (`SmokeConfig.Base`; Tumor Reroll removed); SmokeMax = base + `SmokePoints` x 5
  (+ `SmokeMaxBonus`); regen 4% of max per second. See `SmokeConfig`, `World.Smoke` (`recompute`).
- POI guide (G): hold = show places with name and distance, tap = beam guide / cycle. `POIIndexService` publishes
  `ReplicatedStorage.World.POIIndex`; `StarterPlayerScripts.POIGuideClient`.

- Character tree (T): `AttributeService` + `AttributeConfig` + `CharacterTreeClient`; ranks saved in `PlayerData.Attributes`;
  Strength/Toughness feed DamageLogic (`AttrStrengthMult`, `AttrDamageTakenMult`), Vitality feeds Buffs (`AttrHealthBonus`),
  Smoke sets `SmokePoints`. Admin: `points <n|reset>`.
- `RumorService` (rumor board in `workspace.WorldProps`), fortune teller NPC (`WorldNPCs.FortuneTeller`, action `Fortune`),
  `World.CombatHooks`: landed M1 refunds 1.5% max Smoke, landed M2 silences Smoke users 2.5s
  (M1/M2 modules pass `M1`/`M2` as the 2nd arg of `DamageLogic.Processed`).
- `JobService` (zombie job board in the Hospital), Dr. Mizoguchi (`HospitalDirector`, action `Heal`),
  `CleanupDay`, `GhostNight` and `RuleOfHour` events (`RuleBannerClient` shows the rule); EventService exposes `WorldSignals.SpawnZombies` / `PickSpot`. Admin: `job <finish|fail|reset>`. `WorldSignals.SpendYen` BindableFunction in EconomyService.

- New WorldSignals events: `SmokeSpent` (from Smoke.spend), `LandedHit` (from CombatHooks), `PlayerKilled` (DamageLogic.TakeAction), `WarMeterAdd`.
- `WarService` + `WarMeterClient`: E-18 Boiling Point meter; a full meter starts a 5-min Riot (Lawless reward bonus everywhere,
  ¥25 per player kill). Studio: workspace attr `DebugWarAdd`.
- Events `PartyMishap` (cake + mushroom smoke at the Diner; player attr `MushroomedUntil` blocks casting) and
  `KillingField` (player attr `KillingField` = reward multiplier; kills there pay via WarService).

- Places: main 87872916277829; **Zogan's Academy Grounds 127609270845586** (the former JJK School Map, published, bare: no game
  systems yet). Main place: `WorldProps.AcademyGate` + `World.PlaceTravelService` (`WorldConfig.Places`); the old map is in
  `ServerStorage.ArchivedMaps`. The main place is not yet published with the gate.

### Plans
- Hell and the Sorcerer world become separate places from the Hole, sharing data and code via package links.
- Remaining map pieces are parked. Next: more Tier 1/2 items from the shortlist.

### Admin commands (Cmdr, F2)
`bluenight [start|status|tp|end]`, `forceevent <Event>`, `endevent <Event>`, `adjustcurrency Tags <n>`, `setsmoke <type>`,
`hell [in|out|list]`, `doors [list|unlock]`. In Studio you can also set workspace attribute `DebugForceEvent` to an event Id.

### Known issues
- The Studio MCP plugin often hangs on "start play hasn't finished" after its version changed; restart Studio.
  Once, a Studio drop lost three unsaved edits — save often.
- Everything above has been play-tested at least once; Smoke kit numbers are still first-pass.
