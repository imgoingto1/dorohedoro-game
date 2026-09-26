# Game C — baseplate build list

Everything below can be prototyped in a **fresh baseplate** — no real map, no finished art,
just a flat floor, a couple of dummy rigs, and placeholder blocks standing in for buildings.
It's pulled from the full design in
[`game-c-merger-audit.md`](game-c-merger-audit.md) (rounds 1–29) and the two
[`art-direction/`](../art-direction/) briefs, reorganized around **what to build first**, not
around the order it was decided in.

Every item cites the round(s) it came from — read that section of the audit before building if
anything is unclear. Numbers marked **[draft]** in the audit are still draft here too; build the
mechanic, use the drafted number as a starting value, don't treat it as final.

**What "buildable now" means:** the mechanic and its trigger conditions are decided. It does
**not** mean the content is finished — most items need placeholder animations, VFX, icons or
dialogue to actually ship, which is called out per item as **needs asset**. Build the system
first, wire placeholders in, swap real content in later without touching the logic.

## Progress (2026-09-25)

Being built in **Doro**, a new place owned by the Guppy's Shop group. Game A's combat core was
ported in by reading each script out of the live Map + Combat place, not from the old `.rbxm`
exports. The Vows, the chrono character system and Game A's unused weapons were left behind.
Test dummies are now built from blank rigs at runtime, and the save uses a fresh store,
`GameC_PlayerData_v1`. Playtested in Doro: M1 combo, M2, dodge, block/posture, the Katana
dummies, and all three movement flags (teleport, flight, noclip). Legit fast movement (sprint,
fast sprint, dodges, slide long jump) raised no false flags.

**2026-09-26:** steps 1–8 are built in Doro: the raid (`World.Raid`) and the Devil path
(`Services.Player.DevilPath`: Madame Ise, the payment, the instanced duels, the one-time roll,
the mastery duel, and the H-key form) are done and playtested. Two things the Devil trial's
first playtest turned up, worth knowing for any later teleport or arena: `MovementGuard` ignores
`workspace.Live` when it looks for ground, so anything with a floor in it reads as flight (build
arenas elsewhere), and with streaming on a teleport has to `RequestStreamAroundAsync` first or
the client falls through the floor before it streams in. Step 9 (economy, loot, market, gear) and
a first pass at step 10 (a code-built HUD with the key panels) followed the same day: see the
audit's Economy "Built in Doro" note and the UI status note below.

## Combat

The shared kit and everything layered on it. All of this works against a couple of dummy rigs
on flat ground — no real weapons or Smoke assets needed, just placeholder animations/VFX.

- [x] **The shared kit**: M1 combo, M2, dodge (i-frames + perfect dodge → counter), block/parry
      (0.25 s window → 1 s Riposte), feint, air juggle, 0.6 s input buffer. This is Game A's
      existing combat core, kept as-is (round 3 #9) — port it first, everything else hooks in.
      *Ported to Doro.*
- [x] **Posture and stagger**: posture 0–100, 15% chip on block, drains after 2 s, guard-break
      at 100; stagger from being parried/countered/hit-out-of-a-swing, opens a Finisher window
      (round 24 combat-depth carryover, unchanged from Game A). *Ported with `DamageLogic`.*
- [ ] **Hyperarmor** on bosses and heavy/slow weapon swings only (round 3 #10) — no clash system.
      *The hook is in: `DamageLogic` skips flinch and knockback while the target has the
      `HyperArmor` state. Nothing grants that state yet.* **Next:** neither launch weapon is
      heavy (Katana and Gauntlets, round 4 #13 — Axe was the "heavy/slow" example and isn't
      shipping), so there's no player swing to grant it to yet. Grant `HyperArmor` from
      `BossService` instead — flag it per swing in each boss's phase config, defaulting to "on"
      for the heavier/slower attacks in whatever boss is being tested (Proctor Dunmore or The
      Skinner, round 25 #110) — so the hook has something real to prove against. Revisit
      weapon-side hyperarmor once a heavy weapon ships post-launch.
- [ ] **The 2.5 s Smoke silence** on a landed M2 (round 3 #11).
- [x] **Bare-handed baseline**: the shared kit with no weapon equipped, always available, no
      signature techniques (round 26 #111). *Fist is the default weapon. A sheathed Katana
      falls back to it.*
- [x] **The weapon skill tree — mechanical skeleton**: root node (pick Katana or Gauntlets, free,
      granted at rank 1), Foundation (2 nodes, both learnable), Specialization (3 nodes, pick 2
      of 3, exclusive), Capstone (2 nodes, pick 1 of 2, exclusive), cooldown-only activation, no
      new resource bar (round 26–27). Buildable now with the 14 placeholder names already
      drafted — wire each node to a stub ability (a damage number + a generic VFX) so the
      tree's gating/exclusivity logic can be tested end-to-end. **needs asset** for real
      technique effects/animations later. *Built in Doro:*
      - *`Config.SkillTree` holds the tree, gates and stub numbers. Each tier's stub gives
        Damage/Posture/Cooldown: Foundation 8/20/8 s, Specialization 11/28/12 s, Capstone
        16/40/25 s (**[draft]**). Specialization needs rank 4 and Capstone rank 8
        (**[draft]**).*
      - *`Services.Player.SkillTree` is the server authority over the `SkillTreeRequest`
        remote (PickRoot, Learn, RespecNode, RespecRoot).*
      - *Every node runs one stub `Technique` action: the weapon's heavy swing, a hitbox and
        the generic critical flash. Keys 1–5 fire learned techniques in tree order, and the
        default backpack is off so it doesn't take those keys.*
      - *Gauntlets now exists as a real weapon. It borrows the Fist animations and heavy
        until it has its own (**needs asset**).*
      - *Goro's quests aren't required yet (`REQUIRE_QUESTS = false`). They mark nodes
        through `SkillTree.CompleteQuest`.*
      - *`Rank` is a save field until step 4 builds the ladder. Studio-only `Debug_SetRank`
        and `Debug_AddYen` actions exist for testing.*
      - *Playtested: every gate, exclusivity rule and respec path answered correctly, and a
        technique hit a dummy once and then held its cooldown. Number keys couldn't be tested
        with Studio's input tool (it can't send them), so press 1 to confirm by hand.*
      - **Next:** confirm the 1–5 keys fire the right learned technique by hand (the one thing
        the automated playtest above couldn't reach). Once Trainer Goro exists (see Quests and
        missions, below), flip `REQUIRE_QUESTS` back on and wire `SkillTree.CompleteQuest` to
        his real quest completions instead of the debug call.
- [x] **Weapon respec**: per-node respec (Yen + cooldown, mirrors the attribute respec) and the
      pricier full root respec (round 27) — build both cost/cooldown hooks even with **[draft]**
      numbers.
      - *Node respec (Specialization/Capstone only): the first is free, then ¥250 with a
        10 min cooldown. Dropping a Specialization pick also drops the Capstone that needed
        it.*
      - *Root respec: ¥1,000 with a 60 min cooldown. It wipes the branch and equips the
        other weapon.*
      - *All four figures are **[draft]**.*
- [x] **Attribute ties**: Strength → stagger dealt, Toughness → posture taken, Vitality → PvE
      death Yen resistance, Smoke → regen rate, each layered on the existing flat bonus (round
      25 #109). **[draft]** values — build the hooks, tune later. *Built in Doro
      (`Config.Attributes`, `Services.Player.Progression`), rebuilt from Game A's
      AttributeService at Game C's lower ceiling:*
      - *Flat bonus 1% per point for every attribute, up to 30 points in one.*
      - *Ties per point: Strength +1.5% stagger dealt (`DamageLogic` now scales stagger by the
        dealer's `AttrStaggerMult`), Toughness −1% posture taken, Vitality 1% PvE-death Yen kept
        (`AttrPvEDeathResist`, read by step 5), Smoke +1% pool and +1% regen (player attributes
        waiting for the Smoke system).*
      - *Respec as in Game A: first free, then ¥250 with a 10 min cooldown.*
      - *All values **[draft]**. Playtested: spending, the cap, every effect and respec applied
        correctly.*
- [ ] **Movement**: shared base speed for everyone; mobility only from Smoke type or gear, never
      a stat (round 3 #12).
- [ ] **Smoke moves — framework**: `SmokeMoveService` granting a rolled type's full move set at
      once, Z/X/C casting, a 200-point Smoke bar, 1.5% refund on landed M1s (Game A's existing
      system, kept). Buildable with 2–3 placeholder moves per type before real Smoke VFX exist.
- [x] **The Devil form — core toggle**: a meter/cooldown-gated transformation that swaps in a
      second moveset on top of Smoke, heals a little on activation (round 15–16). Buildable as a
      toggle + stub moveset before the eligibility/trial content around it exists. *Built in Doro
      (round 36 #151): the H key toggles it, a 10% heal, then the roll's buffs while a 90 s meter
      drains, then a 5 min cooldown. **The form's own moveset is not built**; a red-horned look
      stands in until there is art.*
- [x] **True Devil — core toggle**: same shape, stronger, much longer cooldown (round 24 #94).
      **[draft]** thresholds and cooldown. *Built in Doro (2026-09-26): after the mastery, the same four counters at double (plus 5 more hours at rank 10) unlock a third duel against a 3x-HP devil; the True form raises every rolled buff to the 1.5 power, lasts a 120 s meter, has 40% longer horns and rests 2 h. Playtested end to end; the panel shows the higher counters.*

### Anti-cheat (build alongside combat, not after)

- [x] Reach check (45 studs) and hit-speed check in the damage pipeline (kept from Game A). The
      per-player remote rate limits (`RemoteGuard`) came along too.
- [x] **New**: movement validation (flight/noclip), warn-then-kick rather than auto-ban (round 25
      #100, round 26 note). **[draft]** warn count before a kick. *`World.MovementGuard` checks
      teleport (over 60 studs in 0.25 s), sustained speed (over 110 studs/s across 1 s), flight
      (more than 12 studs off the ground for 3 s without coming down) and noclip (the path
      between samples passes through a solid part). A flag moves the player back and warns them
      in chat. The 4th flag inside 60 s kicks (**[draft]** 3 warnings). Studio never kicks. Air
      combat, paralysis, ragdoll and the first 3 s after spawning are exempt. Before a server
      teleport, set the player attribute `MoveGuardGraceUntil`.*

## Events & systems

Backend logic and state machines — none of this needs real art, just trigger volumes, dummy
NPCs, and mock data to drive against.

- [x] **Blue Night — the clock**: one timer (**[draft]** ~15 min real cooldown, both factions
      need players online, ~5 min duration, round 22 #86) that fires both manifestations (round
      29). *Built in Doro (`World.BlueNight`, `Config.BlueNight`):*
      - *Each server keeps its own clock (round 33 #140). It checks every 60 s, starts once 15 min
        have passed since the last one (or since the server started), and only if both factions
        are in the server. Studio skips the faction check, since one tester can't be in both.*
      - *While it runs, workspace attributes `BlueNight` and `BlueNightEndsAt` are set. The
        workspace attribute `Hub` (`Hole` / `SorcererWorld`) picks which manifestation a server
        hosts; unset (the baseplate) runs both.*
      - *`WorldSignals.ForceBlueNight` / `EndBlueNight` start or end it on demand (Studio and admin
        testing); `BlueNightStarted` / `BlueNightEnded` fire for other systems.*
- [x] **Night of the Living Dead** (the Hole's manifestation): kills feed rank, the Devil-path
      counter, and Tags/rare-gear drops (round 29). *Built in Doro:*
      - *Game A's NPC AI stack is ported as `Services.AI` (round 33 #135), checked line for line
        against Map + Combat. Only the unused `chrono` hooks were dropped, and rigs are now built
        at runtime instead of cloned from a stored model.*
      - *Zombies use Game A's look and numbers (Fist, M1 only, 60 HP). They spawn at
        `WorldMarkers.ZombieSpots` and top back up to 12 every 45 s (**[draft]**, #141).*
      - *Every player who damaged a zombie gets `NotLDKills` +1 (#136); `AI.GetHitters` tracks
        them. Each zombie drops one Burial Tag anyone can take (#137).*
      - *Zombies down and grip players as round 30 #124 says. Two fixes came out of the first real
        mob fight: NPCs no longer damage each other (their swings clipped each other in a crowd,
        which cancelled grips), and a mob whose grip is cancelled tries again.*
      - *Round 34 rebuilt the NPC brain to play like Game B's mobs. They grip knocked-out players
        and never hit a body, one NPC fights a player at a time while the rest wait 25 studs back,
        and they tick at a fixed 20 Hz and chase with `MoveTo`. Playtested: 1 zombie fights while 11
        wait, a knocked player is gripped 0.5 s later and never hit, and the full-AI Katana fighter
        (a new Studio fixture, `AI Fighter`) slides in, strings M1s and uppercuts without tripping
        the movement guard.*
      - *Rare gear and Smoke-reroll drops wait for step 9 (loot).*
      - *Playtested: 12 zombies swarm and hit, kills credit the counter, tags pick up, a downed
        player is gripped and executed, and dawn clears everything.*
- [x] **Blue Night carnival** (Sorcerer World's manifestation): a lighting/prop on-off toggle is
      the whole mechanic for now — no zombies, no combat (round 29). **needs asset** for the
      actual carnival props (Ferris wheel, roller coaster). *Built in Doro:*
      - *Anything tagged `BlueNight` switches on: Lights, Beams, ParticleEmitters and Trails are
        enabled, and parts take their `NightMaterial` / `NightTransparency` attributes.*
      - *On the client (`Controllers.World.BlueNight`), `BlueNightSpin` parts turn and
        `BlueNightSweep` parts swing. The lighting eases into Game A's blue night and back at
        dawn.*
      - *The baseplate has a blockout `Carnival`: a Ferris wheel, a coaster stand-in, string
        lights and two searchlights. There's no aurora yet (needs asset).*
- [x] **Blue Night contracts**: the pairing mechanic itself (record two players as contracted);
      what a contract actually grants is still unspecified (round 29), so it's a no-op flag.
      *Built in Doro (`Services.Player.Contract`):*
      - *The Broker appears at `WorldMarkers.Broker` only during Blue Night. The first player signs
        for the partner standing beside them (#138), and the partner signs back within 20 s. That
        prompt stands in for choosing a partner until the UI exists;
        `Remotes.ContractRequest("Sign", userId)` is ready for that panel.*
      - *A pact is saved on both players (`PlayerData.Contract`, the same id on both). It is
        permanent until either player breaks it (`"Break"`), one at a time (#139), and is mirrored
        to the `ContractPartner` attribute.*
      - *Breaking a pact while the partner is offline is recorded in `GameC_ContractBreaks_v1`, and
        their side clears on their next join.*
      - *Tested solo (the remote's refusals are correct). The two-player signing still needs a
        2-player Studio test.*
- [ ] **Toxic Rain**: kept exactly as Game A had it (round 22 #87) — port as-is, Hole-only.
- [x] **The rank ladder**: rank-up logic reading from the Academy tutorial, quests/jobs, the
      first Smoke cast, faction missions, enemy-faction grips, Night of the Living Dead kills,
      and a story-boss first-clear (the audit's actual rank table, corrected below); grants a
      batch of attribute points and gates weapon-tree tiers (round 6, round 21 #82, round 25
      #109). **[draft]** the 10 rank names/gates themselves. *Built in Doro:*
      - *`Config.Ranks` holds the audit's drafted table: names, gates, points (1,1,2,2,3,3,4,4,5,5
        = 30) and unlocks. Rank 7 reads "Cross-Eyed Blade" for the Cross-Eyes faction.*
      - *`PlayerData.RankProgress` keeps cumulative counters: Tutorial, SmokeCast, Quests,
        FactionMissions, Grips, NotLDKills, BossClears. Systems report them through
        `Progression.AddProgress` or `WorldSignals.RankProgress`. Every report checks the next
        gates and ranks up as far as they allow, grants points and unlocks, and announces it in
        chat.*
      - *The skill tree's tiers now read the real rank. `Progression.ProgressBonus` is the hook
        temperament fills in.*
      - *Playtested from Unranked to rank 10 with Studio-only debug counters.*
      - **Resolved:** this bullet's own description used to list Blue Night carnival
        participation and raid contribution as rank sources — that was wrong, a slip when this
        build list was written. The audit's rank table has neither: round 24's follow-up
        explicitly moved raid contribution to the Devil-path counters instead ("grips remain the
        only thing that moves the rank ladder"), and carnival participation was never a rank
        source at all. Built correctly per the audit's real table; the description above is now
        fixed to match. If Blue Night carnival or raid contribution should count toward rank
        after all, that's a new design decision for the audit to record first, not something to
        build from a stray line in this list.
        count.*
- [ ] **Temperament**: one good + one bad roll, bonus (not gated) progress on favored activities
      (round 5 #19, round 6 #23).
- [x] **Faction system**: join on character creation/first Sorcerer World visit, rep from
      missions and enemy-faction grips, switching (Yen fee + rep/seat loss + ~1 week cooldown),
      weekly seat calculation from top rep earners (round 10, round 21 #83–85). **[draft]** rep
      gain amounts, switch fee/cooldown. *Built in Doro (`Config.Factions`,
      `Services.Player.Faction`, `World.FactionSeats`; round 31 answers):*
      - *Join / Leave / Seats / Info go over the `FactionRequest` remote. Leaving costs
        ¥2,000, wipes rep and any seat, and blocks rejoining for 7 days. That's how a switch
        works.*
      - *An enemy-faction grip gives +5 rep. `Faction.AddRep` is ready for missions (+8) and
        objectives (+3).*
      - *Decay: after 7 idle days, 10 rep/day, worked out from when rep was last earned. That
        lets the seat job rank offline players.*
      - *Each Monday the first server to notice ranks both factions' rep boards and saves the
        top 5 as that week's seats. `Seats` only answers for the asker's own faction.*
      - *Downed's grip rule now uses real factions. Rank 7 shows Family Blade / Cross-Eyed Blade.*
      - *Playtested in Studio: joining, rep, decay, the rank title, leaving (fee, cooldown,
        rejoin), and the seat ranking.*
      - ***Not tested:** the DataStore side (rep boards, weekly award). Doro has Studio API
        access off, so it needs Game Settings → Security → "Enable Studio Access to API
        Services", or a live server.*
      - ***Not built:** the join picker (character creation / first Sorcerer World visit). That's
        UI (step 10). Parties aren't in this step either.*
- [ ] **Mission queue**: rank-gated queue for higher-tier repeatables and the raid; low-tier and
      tutorial quests stay direct-from-NPC (round 25 #104). **[draft]** solo vs. party matching.
- [x] **The raid — skeleton**: built Game B-style rather than as a boss instance (round 35): a
      per-server timer pulls every faction member into `workspace.RaidArena`, they vote a mode
      (King of the Hill, Team Grips, Capture the Flag), and rewards go by contribution. *Built in
      Doro as `World.Raid` (`Config.Raid`) and playtested; raiders respawn on their side; the
      winning faction's top contributor gets a Devil's Seal; contribution also sums into
      `DevilProgress.RaidContribution`. The boss with per-phase mechanics from rounds 25–26 is
      replaced by this design, and the 3 s → 33 s respawn curve is not built.*
- [x] **The Devil path — trial structure**: eligibility counter tracking (4 counters + a top-10
      Elo skip), the gatekeeper NPC's missing-requirement dialogue branching, the inner-world
      duel (even against a placeholder "devil" built from stub Smoke+weapon data), the one-time
      buff/downside roll, the mastery duel that removes the downside (round 15–17, round 20 #79,
      round 24 #96 follow-up). *Built in Doro as `Services.Player.DevilPath` and playtested end to
      end (pay, duel, roll, mastery, form, cooldown): see the audit's "Built in Doro" note under
      The Devil path. Studio-only `Debug_*` actions on `Remotes.DevilRequest` skip the grind.*
- [x] **Death/PvP**: the downed state (spare/carry/execute), 25% carried Yen/Tags on execution,
      a flat PvE death Yen loss (**[draft]** 500), a 40% combat-log penalty, the anti-farming
      guards (enemy-faction-only grip credit, 2-rank floor, ~1 h per-victim Elo cooldown, a
      **[draft]** 1,000 Yen/day transfer cap) (round 1 #2, round 12, round 13, round 14 #54,
      round 15 #58). *Built in Doro as Game B's knock → carry → grip, rebuilt from how B plays
      (`Services.Player.Downed`, the `Grip` action, `ServerStorage.Packages.Ragdoll`; rounds 30
      and 32):*
      - *`DamageLogic` knocks players out at 0 HP instead of killing them: a limp ragdoll (the
        owning client puts the Humanoid into Physics), immune to damage, for 12 s. Then they
        come to with +10% max HP and 1 s of i-frames.*
      - *V carries or drops within 6 studs (0.5 s toggle cooldown). A dropped body lands 3.5
        studs ahead with its knockout restarted.*
      - *G grips within 6 studs, only a body on the ground and not carried. Both players snap
        face to face 2.75 studs apart and are held in place. The kill lands on the grip
        animation's `EventFrame` keyframe (3 s fallback). A hit cancels it and restarts the
        knockout.*
      - *Grip animations are looked up per weapon at `Animations.Weapons.<weapon>.Grip` /
        `.Gripped`, falling back to `Animations.Default.Grip` / `.Gripped`.*
      - *A player executioner gets 25% of the victim's Yen and Tags. The grip goes to rank only
        against an enemy faction within 2 ranks; until factions exist (step 6) it counts in
        Studio only.*
      - *Mobs walk over and grip the players they downed; that costs the victim the PvE loss,
        reduced by Vitality.*
      - *Combat log: leaving within 60 s of a PvP hit, or while downed by a player, destroys 40%.
        It's applied through a new `Data.LeaveHooks` before the save is released, and the
        player is told on their next join.*
      - *Spawn grace was ported (3 s, ends when you land a hit). Messages go to chat for now
        (`Misc.Notice`).*
      - ***Playtested:** a mob knocked out, ragdolled and gripped a player (¥1,000 → ¥500),
        with both freed after. A hit on the mob cancelled its grip; the body went back to the
        ragdoll, and the player came to 12 s later at 11 HP (1 + 10%). **Not yet playtested:**
        a player gripping a player (money, grip credit, Elo), carry, and combat log. All three
        need two players (Studio: Test → Clients and Servers → 2 players).*
      - ***Not built:** the Yen transfer cap. There's no give/drop feature yet, so it belongs
        with the economy step.*
      - ***needs asset:** Game C's own grip / gripped pair (one per weapon, or one shared
        Default pair) and carrying / carried animations. The grip pair needs an `EventFrame`
        keyframe where the kill lands. Game B's animations can't be used. The current
        placeholder is a slowed uppercut on the executioner, with the victim held still.*
- [x] **Elo**: hidden per-player rating, a public top-10 leaderboard only (round 13 #53, round 22
      #89). *Standard Elo, K=24 (Game A's), updated on player executions; the 1 h per-victim
      cooldown is saved per executioner (`EloCooldowns`). `World.EloBoard` writes ratings to
      the OrderedDataStore `GameC_Elo_v1` and publishes the top 10 as names and rank titles
      only, never the number (workspace attribute `EloTop10`, for the UI step).*
- [x] **Economy — loot and the market**: mob/raid drop tables (**[draft]** ~3%/~0.5%/~0.1%
      common/rare/Smoke-reroll odds), the merged rotating market (Yen for common/uncommon, Tags
      required for rare+), the weekend rare-drop-odds doubling, the purchase ledger and code
      redemption (round 11, round 20 #80, round 23 #90, round 24 #102, round 25 #107).
      *Built in Doro (drafts, see the audit's Economy "Built in Doro" note): `Config.Items` (10 slots, 50 placeholder pieces), `Config.Economy`, `Services.Player.Equipment` / `.Loot` / `.Market`; the raid pays into the loot pool and Night of the Living Dead kills roll drops. The Yen transfer cap is done. Not built: the Smoke reroll's use, Black Smoke, cosmetics, Robux products; giving Yen is untested (needs two clients).*
- [ ] **Identity rolls**: the cosmetic roll mechanic itself (markings/color/callout tied to
      faction), even before the real roll table exists (round 25 #103). **needs asset** for the
      actual table.

## Atmosphere & lighting

This is exactly what a baseplate is good for — build and tune these before the real map exists,
against placeholder blockout geometry.

- [ ] **The Hole — lighting profile**: `Atmosphere` Density/Haze up, sepia-green colour
      (`rgb(150,150,120)`), darker brown-grey `Decay`; `ColorCorrection` slightly desaturated
      with a warm-green tint; sodium-yellow street lamps (`rgb(255,190,110)`)
      (`art-direction/hole.md`).
- [ ] **Sorcerer World — lighting profile**: day (low `Atmosphere` density, teal sky, strong
      `SunRays`, gentle `Bloom`, warm sun) and an ordinary night (dark, starry, **no** carnival
      effects — those are Blue-Night-only now, round 29) (`art-direction/sorcerer-world.md`).
- [ ] **Overhead wires** (the Hole): sagging `Beam`s strung between attachment points on facing
      blockout buildings, using `CurveSize0/1` for the sag. The single biggest visual win per
      hour, and needs nothing but two placeholder walls to test against.
- [ ] **String lights** (Sorcerer World carnival): the same Beam technique, reused, draped
      between placeholder ride/building shapes; only active during Blue Night (round 29).
- [ ] **Searchlights** (Sorcerer World carnival): a few sweeping `SpotLight`/`Beam` cones from a
      tall placeholder landmark, Blue-Night-only.
- [ ] **Day-night cycle** driver for Sorcerer World's ordinary (non-carnival) day/night swing.
- [ ] **Shadows on by default**, both hubs, with the existing quality-setting toggle
      (Shadows/PostFX/AmbientFX/FpsCounter shape) for players who need the FPS (round 28 #120).
- [ ] **Night of the Living Dead — visual shift**: not designed yet beyond "the Hole, but with
      zombies" — a fog-thickening or colour-correction pass is a good baseplate experiment once
      the base Hole lighting above exists (`art-direction/hole.md`).

## UI

React-only (round 18 #72). Every panel below can be built and tested against mock data before
the systems behind it are finished — the UI doesn't need real backend numbers to prove its
layout.

> **UI status (2026-09-26):** the first HUD is built in Doro as `Controllers.Gui.Hud`, from code
> with a small kit, **not React**. Round 18 #72 says React-only, but Doro has no React package
> (Game A's ReactLua is ~8 MB and lives in Map + Combat), so this is a stopgap: each panel is a
> refresh function over data the server already sends, so a React port rewrites the views only.
> Built: the resource/rank strip, world banners (raid score, Blue Night, inner-world timer), the
> Devil form meter, health and posture bars, and the panels Rank & attributes (K), Techniques (J), Gear (B), Market with ledger and codes
> (M), Faction (Y), Devil path (L), Top 10 (T), plus the raid vote panel. Playtested with screenshots.
> Not built: the Smoke bar (no Smoke system in Doro yet), a stagger bar, the
> mission/raid queue, the identity-roll picker, settings.

- [ ] **Core HUD**: health, guard/posture, stagger, and the Smoke bar. *Health and posture bars are built; the stagger and Smoke bars aren't.*
- [x] **The weapon skill tree screen**: renders the root pick and both branches' 3-tier shape,
      shows locked/available/learned state per node, drives the exclusivity rule visually (round
      26 diagram).
- [x] **Rank progress panel**: current rank, this rank's gate progress, next rank's reward.
- [x] **Faction panel**: rep total, seat standings (a private panel, not a public leaderboard,
      round 22 #89), the switch-faction flow.
- [x] **The rotating market / Tag shop screen**: one merged screen, common/uncommon in Yen, rare+
      greyed out until enough Tags (round 25 #107). Reference Game B's layout for what
      information to show, restyle entirely in Game C's own look (round 25 #106).
- [x] **Purchase ledger screen** (round 24 #102) — reference B's layout the same way.
- [ ] **Mission/raid queue screen**: rank-gated entry, a queue-position or matchmaking state.
- [x] **Raid vote panel**: reference B's layout the same way (round 25 #106).
- [x] **The Devil path panel**: the four eligibility counters' progress, which one is still
      missing (mirrors what the gatekeeper's dialogue says).
- [ ] **Identity-roll picker**: shows the faction-flavored cosmetic roll result, a reroll button.
- [x] **Top-10 Elo leaderboard** (public) — separate from the faction seat panel (private).
- [ ] **Settings panel**: Shadows/PostFX/AmbientFX/FpsCounter toggles, same shape as before the
      restart.

## Suggested build order

Not a hard sequence, but a reasonable one for a single baseplate session:

1. Shared combat kit + posture/stagger (nothing else works without this).
2. Anti-cheat alongside it — cheap to add now, expensive to retrofit later.
3. Weapon skill tree skeleton, stubbed techniques.
4. Rank ladder + attribute ties (most other systems read from rank).
5. Death/PvP + Elo (needs the rank ladder's grip-counting to exist first).
6. Faction system (rep needs somewhere to come from — grips, above).
7. Blue Night's two manifestations (Night of the Living Dead needs zombie mobs; the carnival
   needs nothing but a light/prop toggle).
8. Raid skeleton + the Devil path trial (both depend on rank, grips and Blue Night existing).
9. Economy/market/loot (ties everything above into Yen/Tags).
10. Atmosphere and UI can run in parallel with all of the above — none of it blocks on backend
    systems being finished, only on knowing what data each panel will eventually show.
