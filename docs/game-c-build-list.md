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
- [ ] **Attribute ties**: Strength → stagger dealt, Toughness → posture taken, Vitality → PvE
      death Yen resistance, Smoke → regen rate, each layered on the existing flat bonus (round
      25 #109). **[draft]** values — build the hooks, tune later.
- [ ] **Movement**: shared base speed for everyone; mobility only from Smoke type or gear, never
      a stat (round 3 #12).
- [ ] **Smoke moves — framework**: `SmokeMoveService` granting a rolled type's full move set at
      once, Z/X/C casting, a 200-point Smoke bar, 1.5% refund on landed M1s (Game A's existing
      system, kept). Buildable with 2–3 placeholder moves per type before real Smoke VFX exist.
- [ ] **The Devil form — core toggle**: a meter/cooldown-gated transformation that swaps in a
      second moveset on top of Smoke, heals a little on activation (round 15–16). Buildable as a
      toggle + stub moveset before the eligibility/trial content around it exists.
- [ ] **True Devil — core toggle**: same shape, stronger, much longer cooldown (round 24 #94).
      **[draft]** thresholds and cooldown.

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

- [ ] **Blue Night — global clock**: one timer (**[draft]** ~15 min real cooldown, both factions
      need players online, ~5 min duration, round 22 #86) that fires two manifestations at once
      (round 29).
- [ ] **Night of the Living Dead** (the Hole's manifestation): a zombie spawn wave against dummy
      rigs is enough to prototype — kills feed rank, the Devil-path counter, and Tags/rare-gear
      drops (round 29).
- [ ] **Blue Night carnival** (Sorcerer World's manifestation): a lighting/prop on-off toggle is
      the whole mechanic for now — no zombies, no combat (round 29). **needs asset** for the
      actual carnival props (Ferris wheel, roller coaster).
- [ ] **Blue Night contracts**: the pairing mechanic itself (record two players as contracted)
      can be built now; what a contract actually grants is still unspecified (round 29) — stub
      it as a no-op flag until that's answered.
- [ ] **Toxic Rain**: kept exactly as Game A had it (round 22 #87) — port as-is, Hole-only.
- [ ] **The rank ladder**: rank-up logic reading from quests/jobs, faction missions, grips, Night
      of the Living Dead kills, Blue Night (carnival) participation, and raid contribution;
      grants a batch of attribute points and gates weapon-tree tiers (round 6, round 21 #82,
      round 25 #109). **[draft]** the 10 rank names/gates themselves.
- [ ] **Temperament**: one good + one bad roll, bonus (not gated) progress on favored activities
      (round 5 #19, round 6 #23).
- [ ] **Faction system**: join on character creation/first Sorcerer World visit, rep from
      missions and enemy-faction grips, switching (Yen fee + rep/seat loss + ~1 week cooldown),
      weekly seat calculation from top rep earners (round 10, round 21 #83–85). **[draft]** rep
      gain amounts, switch fee/cooldown.
- [ ] **Mission queue**: rank-gated queue for higher-tier repeatables and the raid; low-tier and
      tutorial quests stay direct-from-NPC (round 25 #104). **[draft]** solo vs. party matching.
- [ ] **The raid — skeleton**: voting on entry, `CreditRange`-style contribution tracking, a
      per-death respawn timer inside the raid only (**[draft]** 3 s → 13 s → 23 s, capped 33 s,
      round 25 #105), and a `BossService`-config-driven boss with unique per-phase mechanics
      (round 25 #110, round 26 note). Buildable against one placeholder boss before its real
      encounter design exists.
- [ ] **The Devil path — trial structure**: eligibility counter tracking (4 counters + a top-10
      Elo skip), the gatekeeper NPC's missing-requirement dialogue branching, the inner-world
      duel (even against a placeholder "devil" built from stub Smoke+weapon data), the one-time
      buff/downside roll, the mastery duel that removes the downside (round 15–17, round 20 #79,
      round 24 #96 follow-up).
- [ ] **Death/PvP**: the downed state (spare/carry/execute), 25% carried Yen/Tags on execution,
      a flat PvE death Yen loss (**[draft]** 500), a 40% combat-log penalty, the anti-farming
      guards (enemy-faction-only grip credit, 2-rank floor, ~1 h per-victim Elo cooldown, a
      **[draft]** 1,000 Yen/day transfer cap) (round 1 #2, round 12, round 13, round 14 #54,
      round 15 #58).
- [ ] **Elo**: hidden per-player rating, a public top-10 leaderboard only (round 13 #53, round 22
      #89).
- [ ] **Economy — loot and the market**: mob/raid drop tables (**[draft]** ~3%/~0.5%/~0.1%
      common/rare/Smoke-reroll odds), the merged rotating market (Yen for common/uncommon, Tags
      required for rare+), the weekend rare-drop-odds doubling, the purchase ledger and code
      redemption (round 11, round 20 #80, round 23 #90, round 24 #102, round 25 #107).
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

- [ ] **Core HUD**: health, guard/posture, stagger, and the Smoke bar.
- [ ] **The weapon skill tree screen**: renders the root pick and both branches' 3-tier shape,
      shows locked/available/learned state per node, drives the exclusivity rule visually (round
      26 diagram).
- [ ] **Rank progress panel**: current rank, this rank's gate progress, next rank's reward.
- [ ] **Faction panel**: rep total, seat standings (a private panel, not a public leaderboard,
      round 22 #89), the switch-faction flow.
- [ ] **The rotating market / Tag shop screen**: one merged screen, common/uncommon in Yen, rare+
      greyed out until enough Tags (round 25 #107). Reference Game B's layout for what
      information to show, restyle entirely in Game C's own look (round 25 #106).
- [ ] **Purchase ledger screen** (round 24 #102) — reference B's layout the same way.
- [ ] **Mission/raid queue screen**: rank-gated entry, a queue-position or matchmaking state.
- [ ] **Raid vote panel**: reference B's layout the same way (round 25 #106).
- [ ] **The Devil path panel**: the four eligibility counters' progress, which one is still
      missing (mirrors what the gatekeeper's dialogue says).
- [ ] **Identity-roll picker**: shows the faction-flavored cosmetic roll result, a reroll button.
- [ ] **Top-10 Elo leaderboard** (public) — separate from the faction seat panel (private).
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
