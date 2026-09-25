# Ideas & To-Do

Tasks and ideas grouped by area. Each item is tagged:

- **[gap]** is a known gap already recorded in [`work-log/worklog.md`](work-log/worklog.md).
- **[idea]** is a suggestion that has not been agreed. Per the standing rule in `CLAUDE.md`, ask Jay before building it.
- **[requested]** was asked for directly by Jay. It is approved to work on, but details marked
  "needs Jay's call" still have to be confirmed first.
- **needs asset** means it can't be finished from code alone. It needs a model, texture,
  animation or sound uploaded or shared.

Tick items off here and log the work in the work log when done.

## Quick wins (code only, no assets needed)
- [ ] Smoke-cast screen tint: a per-type colour flash on cast, reusing `SmokeTypes.Types[id].Color`. **[gap]**
- [ ] Sound-trigger hook service: named hooks (hit, block, parry, cast, level-up, buy, event
      start/end) so dropping in audio later only means filling in IDs. **[gap]**
- [ ] Check `Lighting.Technology` in the Properties panel. If it is Unified, test ShadowMap.
      `LightingStyle` is now Realistic. **[gap]**
- [x] ~~Decide what to do with M1 reach~~: Fist hitbox widened, 3/3 hits at 6 studs (2026-09-24).
- [ ] `Controllers.Gui.Inventory` still logs "Infinite yield possible on GameUI" on slow loads; give
      it the same timeout as Camera, Hotbar and HealthBar. **[gap]**

## Weapons & items
- [ ] **Store Knife**: a knife sold in a shop. **[requested]** Needs Jay's call: whether it's its
      own weapon or a variant of the existing Dagger (which already has a full animation set),
      which shop sells it, its price and stats.

## Smoke types
- [ ] **Time Smoke**: a new Smoke type built around time manipulation, like Doppio. **[requested]**
      Hooks into `SmokeTypes`, `SmokeMoves` and `SmokeMoveService` like the other 7 types. Needs
      Jay's call: its 2-3 moves, costs and cooldowns. Keep the moves original rather than copying
      JoJo abilities.

## Map
- [ ] **Fix the Hole map**: it feels like an ordinary modern city rather than the anime's Hole.
      **[requested]** Likely cause: much of it came from jjk's Shibuya chunks. Jay shared
      reference images on 2026-09-24; the brief is in
      [`art-direction/hole.md`](art-direction/hole.md), with steps ordered from code-only
      (smog lighting, overhead wires) to a full blockout. Big map and asset job; redo it on each
      place's own map.
- [ ] **Build the Sorcerer World** as its own place, sharing data with the Hole via package
      links (planned, not started). **[gap]** Jay shared reference images on 2026-09-24; the
      brief is in [`art-direction/sorcerer-world.md`](art-direction/sorcerer-world.md): ornate
      old-world city in partial ruin by day, saturated carnival by night. Needs Jay's call on
      how players get there and what it's for in gameplay. needs asset

## VFX
- [ ] **New VFX across the game**: Smoke moves, hits, events and world effects. **[requested]**
      needs asset. Covers the Smoke-move item below.
- [ ] Replace the flat-colour `Kit.slash` / `Kit.burst` effects on all 16 Smoke moves with real
      particles, beams and trails, one signature look per Smoke type. **[gap]** needs asset
- [ ] Port VFX from Hollow Lineage. This was skipped because some effects have 100+ parts; use
      `.rbxm` export/import, as in the original merge, instead of rebuilding by hand. **[gap]**
- [ ] Hit sparks per weapon, plus block, parry and posture-break effects. **[idea]**
- [ ] Smoke visibly leaking from a sorcerer while casting or when their Smoke is low. **[idea]**
- [ ] Environment art for Hell (currently built from basic parts). **[gap]** needs asset

## Animations
- [ ] **Rework all animations**: the 7 weapon sets, the 13 Smoke move animations ported from
      Hollow Lineage, and movement. **[requested]** needs asset
- [ ] **New Smoke move animations (IDs supplied 2026-09-24)**: SmokeShot, ChargedBlast, Curse
      (Hex), Cling, SporeBurst, SporeTrap, Unravel, SplitCut, Mend (self and others) and
      Surge. Unravel is reworked to turn the caster invisible for a moment, with all other effects
      removed. **[requested]** Full instructions and the ID table are in
      [`tasks/smoke-move-animations.md`](tasks/smoke-move-animations.md).
- [ ] `Lizard.ScaleForm` and `Dinosaur.BeastForm` have no animation. `Mushroom.SporeTrap` gets one
      from the task above. **[gap]**
- [ ] Real Lizard and Dinosaur creature rigs and animations (the forms currently reskin the
      player's own rig). **[gap]** needs asset
- [ ] Idle animations for world NPCs. **[gap]** needs asset
- [ ] Cast animations ignore the equipped weapon and don't cut into combos; tune priority and blending. **[gap]**
- [x] ~~Animations breaking at random~~: fixed 2026-09-24 (perfect dodge and hit-stop no longer
      stop movement tracks, and a watchdog restarts a stopped looping pose).
- [ ] Directional hit reactions and a posture-break stun animation. **[idea]**

## Atmosphere fixes
- [ ] **Fix the rain**, which doesn't look good. Swap in a weather module's rain for the rain
      events (ToxicRain etc.). **[requested]** Today's rain is `AtmosphereClient`'s
      camera-follow `ParticleEmitter` (Rate 900 streaks plus `smoke_main.dds` mist). Keep it
      compatible with the `AmbientFX` setting (it toggles emitters under `CurrentCamera`) and
      with the Hell/BlueNight/GhostNight overrides, and check the module's licence.
- [ ] Asset permission errors, e.g. ColorMap `14565342511` on the Underground Lab. Each asset
      has to be shared from its own asset page. **[gap]**
- [ ] Restore the 13 missing MAP_3 props (kitchen items and 2 loose parts). **[gap]**
- [x] ~~`WeatherManager` / `SmallClouds`~~: archived 2026-09-24 along with the jjk day/night cycle.
- [ ] Check the 2026-09-24 street-lamp pass by eye: bulb brightness, bloom and how dark night is.
      The logic was verified in Play, but the viewport rendered blank. **[gap]**
- [ ] Per-danger-zone variation of the day-night cycle and ash layer (currently set per place only). **[gap]**
- [ ] Ambient sound loops for the city, Hell and rain. **[gap]** needs asset
- [ ] Decide the default for the Shadows setting per device (e.g. off on mobile). **[idea]**

## UI
- [ ] **New UI**: a full redesign. **[requested]** The 2026-09-22/23 pass centralised colours and
      fonts in `ReplicatedStorage.World.UITheme`, so a new look can start with the tokens there,
      then layouts. Needs Jay's call on the style direction.
- [ ] Item icons: `Items` has no `Icon` field. `InventoryClient` already has the one-line hook. **[gap]** needs asset
- [ ] Smoke move icons in `SmokeMovesHud`, which currently shows colour swatches. **[gap]** needs asset
- [x] ~~Decide the final HUD~~: `VitalsHud` replaced jjk's Player_Display bars (2026-09-24).
- [x] ~~Party UI~~: the React HUD's party panel + invite popup (2026-09-24, `Remotes.PartyAction`).
      Clan UI is still chat commands only. **[idea]**
- [ ] React HUD readability: the quest and party text is thin over bright ground. It needs a backing or
      a heavier stroke. **[idea]**
- [ ] Buttons are low-contrast since the switch to the blue-grey accent; decide on a call-to-action colour. **[gap]**
- [ ] Damage numbers and the target's guard/stagger bar. The combo counter already exists
      (`Misc.ComboHit`). **[idea]**

## Models
- [ ] Outfits for the world NPCs, which are grey R6 rigs today: Street Informant, Shopkeeper,
      Trainer Goro, Diner Cook, Gyoza Cook, Grave Keeper, Fortune Teller and Hospital Director. **[gap]** needs asset
- [ ] Equipment meshes for Cleaner's Coat, Scrap Vest, Smoke Charm and Grave Bell, each
      currently a single part. **[gap]** needs asset
- [ ] Remove the Grudge Monument (Jay doesn't want it). **[gap]**
- [ ] Staff shops on Zogan's Academy NPCs; there are none today. Needs a decision. **[gap]**

## Mask models
- [ ] Real meshes for Gas Mask, Bone Mask and Devil Mask. Each is a single resized part today. **[gap]** needs asset
- [ ] A mask per Smoke type, since masks are central to a sorcerer's identity in Dorohedoro.
      Keep the designs original, inspired by the series rather than copied from it. **[idea]** needs asset
- [ ] Show the equipped mask in the inventory and character panel preview. **[idea]**

## Sounds
- [ ] **New sounds**: a full sound library for combat, Smoke casts, UI, world ambience and
      events. **[requested]** needs asset. Build the sound-trigger hook service in Quick wins
      first so sounds only need asset IDs filled in.

## Combat feel
- [ ] Hit, block, parry and posture-break sound effects; this is the biggest missing feel
      lever. Depends on the sound-trigger hooks above. **[gap]** needs asset
- [ ] Holding LeftControl both fires Slide and arms the M1→Uppercut modifier; decide whether to split them. **[gap]**
- [ ] Spore Burst costs 24 Smoke for 6.8 damage, against 10.2 for a free M1. Test it against a group. **[gap]**
- [ ] Play-test with real input: a clean parry into a riposte, and M2 guard-breaking the Ashmask
      Shieldbearer (scripted input can't time these). **[gap]**
- [ ] Play-test the balance numbers (a maxed character gets about +70% damage; Jay said they
      "seem very high"). **[gap]**
- [ ] Hitstop scaled by weapon weight, and knockback on heavy hits and finishers. **[idea]**
- [x] ~~Lock-on camera~~: soft lock-on built (`Functions.SoftLock`, setting `SoftLock`), along with
      the stagger meter, finishers and enemy archetypes (2026-09-24).
