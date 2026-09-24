# Ideas & To-Do

Tasks and ideas grouped by area. Each item is tagged:

- **[gap]** is a known gap already recorded in [`work-log/worklog.md`](work-log/worklog.md).
- **[idea]** is a suggestion that has not been agreed. Per the standing rule in `CLAUDE.md`, ask Jay before building it.
- **needs asset** means it can't be finished from code alone. It needs a model, texture,
  animation or sound uploaded or shared.

Tick items off here and log the work in the work log when done.

## Quick wins (code only, no assets needed)
- [ ] Smoke-cast screen tint: a per-type colour flash on cast, reusing `SmokeTypes.Types[id].Color`. **[gap]**
- [ ] Sound-trigger hook service: named hooks (hit, block, parry, cast, level-up, buy, event
      start/end) so dropping in audio later only means filling in IDs. **[gap]**
- [ ] Check `Lighting.Technology` in the Properties panel. If it is Unified, test ShadowMap. **[gap]**
- [ ] Decide what to do with M1 reach (whiffs at 6 studs, lands at 4.3). **[gap]**

## VFX
- [ ] Replace the flat-colour `Kit.slash` / `Kit.burst` effects on all 16 Smoke moves with real
      particles, beams and trails, one signature look per Smoke type. **[gap]** needs asset
- [ ] Port VFX from Hollow Lineage. This was skipped because some effects have 100+ parts; use
      `.rbxm` export/import, as in the original merge, instead of rebuilding by hand. **[gap]**
- [ ] Hit sparks per weapon, plus block, parry and posture-break effects. **[idea]**
- [ ] Smoke visibly leaking from a sorcerer while casting or when their Smoke is low. **[idea]**
- [ ] Environment art for Hell (currently built from basic parts). **[gap]** needs asset

## Animations
- [ ] `Lizard.ScaleForm`, `Dinosaur.BeastForm` and `Mushroom.SporeTrap` have no animation. **[gap]**
- [ ] Real Lizard and Dinosaur creature rigs and animations (the forms currently reskin the
      player's own rig). **[gap]** needs asset
- [ ] Idle animations for world NPCs. **[gap]** needs asset
- [ ] Cast animations ignore the equipped weapon and don't cut into combos; tune priority and blending. **[gap]**
- [ ] Directional hit reactions and a posture-break stun animation. **[idea]**

## Atmosphere fixes
- [ ] Asset permission errors, e.g. ColorMap `14565342511` on the Underground Lab. Each asset
      has to be shared from its own asset page. **[gap]**
- [ ] Restore the 13 missing MAP_3 props (kitchen items and 2 loose parts). **[gap]**
- [ ] `WeatherManager` / `SmallClouds`: remove the disabled jjk leftovers or restore them properly. **[gap]**
- [ ] Per-danger-zone variation of the day-night cycle and ash layer (currently set per place only). **[gap]**
- [ ] Ambient sound loops for the city, Hell and rain. **[gap]** needs asset
- [ ] Decide the default for the Shadows setting per device (e.g. off on mobile). **[idea]**

## UI
- [ ] Item icons: `Items` has no `Icon` field. `InventoryClient` already has the one-line hook. **[gap]** needs asset
- [ ] Smoke move icons in `SmokeMovesHud`, which currently shows colour swatches. **[gap]** needs asset
- [ ] Decide the final HUD, since GameUI and Player_Display both draw a health bar and hotbar. **[gap]**
- [ ] Party and clan UI, which are chat commands only today. The `PartyLeader` and
      `PartyMembersJSON` attributes are ready for it. **[idea]**
- [ ] Buttons are low-contrast since the switch to the blue-grey accent; decide on a call-to-action colour. **[gap]**
- [ ] Damage numbers, a combo counter, and the target's posture bar. **[idea]**

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

## Combat feel
- [ ] Hit, block, parry and posture-break sound effects; this is the biggest missing feel
      lever. Depends on the sound-trigger hooks above. **[gap]** needs asset
- [ ] Holding LeftControl both fires Slide and arms the M1→Uppercut modifier; decide whether to split them. **[gap]**
- [ ] Spore Burst costs 24 Smoke for 6.8 damage, against 10.2 for a free M1. Test it against a group. **[gap]**
- [ ] Play-test the NPC tuning (more blocking, less dodging) and confirm posture actually breaks. **[gap]**
- [ ] Play-test the balance numbers (a maxed character gets about +70% damage; Jay said they
      "seem very high"). **[gap]**
- [ ] Hitstop scaled by weapon weight, and knockback on heavy hits and finishers. **[idea]**
- [ ] Lock-on camera for 1v1 fights. **[idea]**
