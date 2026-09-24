# Task: new Smoke move animations (2026-09-24)

**For the local station.** Jay supplied new animation IDs for the Smoke moves on 2026-09-24. Swap
them in, rework Unravel as described below, verify, and log the result. Read `CLAUDE.md` first:
keep both places at parity, never run Play in both at once, and finish with `selftest`.

## Where the code is
From `PROJECT.md` and the work log (check it's still current before editing):
- Animation instances: `ReplicatedStorage.Assets.Animations.Smoke.<TypeId>.<MoveId>`. 13 exist
  already, with IDs ported from Hollow Lineage on 2026-09-17.
- Move handlers: `ServerScriptService.World.SmokeMoves.<Type>`. Each calls `TrackService.Play(...)`
  next to its `Kit.*` VFX call.
- Move lists and tuning: `ReplicatedStorage.World.SmokeTypes`. The server entry point is
  `SmokeMoveService`; the client is `StarterPlayerScripts.SmokeMovesClient`.

## 1. Swap in the new animation IDs

| Jay's name | Move (Type.MoveId) | New AnimationId | Note |
|---|---|---|---|
| Shooting Skill | `Gun.SmokeShot` | `rbxassetid://84898578407762` | |
| ChargedBlast | `Gun.ChargedBlast` | `rbxassetid://117021544752937` | |
| Curse | `Curse.Hex` | `rbxassetid://111120682208388` | Confirmed by Jay: Hex. Leave Wither's animation unchanged. |
| Cling | `Mushroom.Cling` | `rbxassetid://92421614207420` | |
| Spore Burst | `Mushroom.SporeBurst` | `rbxassetid://130349660804168` | |
| SporeTrap | `Mushroom.SporeTrap` | `rbxassetid://130349660804168` | Same ID as Spore Burst, on purpose (confirmed by Jay). SporeTrap had no animation before, so it needs a new Animation instance and a `TrackService.Play` call in its handler. |
| Unravel | `Split.Unravel` | `rbxassetid://132885200996279` | Also gets new behaviour; see section 2. |
| SplitCut | `Split.SplitCut` | `rbxassetid://138813793759050` | |
| Mend (heal self) | `Regeneration.Mend` | `rbxassetid://98692159434381` | Two animations; see section 3. |
| Mend (heal others) | `Regeneration.Mend` | `rbxassetid://104356504502793` | |
| Surge | `Regeneration.Surge` | `rbxassetid://116967723321316` | |

Not covered by this list, so leave as they are: `Curse.Wither`,
`Lizard.TailSweep`, `Dinosaur.Stomp`, `Dinosaur.Bite`, and the form toggles `ScaleForm` and `BeastForm`.

## 2. Unravel: invisible for a moment

Jay pasted the original skill script. It's a client effects module from another game, driven by
phase strings "1" to "5":

1. Startup: a sound and a particle effect welded to the root part.
2. Nothing.
3. A trail welded to the root part, then `FlashstepClient.Invisible(Character, 0.15)`.
4. A hit effect and sound on the target's root part.
5. `FlashstepClient.Visible(Character, 0.1)`, and the trail is switched off.

Jay's instruction: **remove all assets except the animation.** No sounds, particles or trails,
and none of the other game's modules (`Modules.Shared.Debris`, `SharedFunctions`,
`FlashstepClient`, `Assets.SkillSounds.Kendo`, `Assets.Effects.Kendo`), which don't exist here.
**The only effect kept is the character going invisible for a short moment**, fading out over
about 0.15 s at phase 3 and back in over about 0.1 s at phase 5.

How to build it:
- **Do the invisibility on the server**, in the `Split.Unravel` handler, so every player sees it.
  The original only did it on the client because its effect module was broadcast to all clients.
  Change `LocalTransparencyModifier` or `Transparency` on the character's `BasePart`s, accessory
  handles and face `Decal`s. Store each original value and restore it exactly afterwards; never
  hard-code 0, since some parts are meant to be transparent (e.g. the HumanoidRootPart is 1).
  Tweening `Transparency` works.
- **Timing:** the phases probably come from markers or keyframes in the animation. Check the new
  animation for marker or keyframe names "1" to "5" (`GetMarkerReachedSignal`, `KeyframeReached`)
  and drive the fade from phases 3 and 5 if they exist. If there are none, pick times that match
  the animation and write down what you chose.
- Make sure the character is always restored even if the cast is interrupted (stun, death,
  cancel). Use a `task.delay` safety restore or a cleanup on the Trove/state change.
- Keep Unravel's existing damage, cost and cooldown unless Jay says otherwise. This is a
  presentation change only.

## 3. Mend: two animations
Mend now has a heal-self animation (`98692159434381`) and a heal-others one (`104356504502793`).
Read the `Regeneration.Mend` handler first:
- If it can already target another player, play the heal-others animation on those casts and the
  heal-self animation otherwise.
- If it only heals the caster, **don't invent targeting.** Adding "heal others" would be a new
  mechanic, which needs Jay's call per `CLAUDE.md`. Wire up the heal-self animation, store the
  heal-others Animation instance for later, and ask Jay.

## 4. Verify
- Studio's output should show `|Animations| Loaded (N/N)!` with no failures. If an ID fails to
  load, it probably isn't owned by Jay's account or group, or isn't shared; report which one.
- In Play (one place at a time), cast each changed move on Z/X/C and confirm the new ID is
  playing, by reading `Animator:GetPlayingAnimationTracks()` on the server, as on 2026-09-17.
- For Unravel: confirm the character goes invisible and fully comes back, including after being
  interrupted mid-cast.
- `selftest`: 38/0 in Map + Combat, 37/0/1 in Academy. Clean console.
- Mirror to Zogan's Academy Grounds and confirm parity with the per-script hash check.

## 5. When done
- Add a work-log entry: what changed, what was and wasn't tested, and the answers to the open
  questions above.
- Tick the matching items in `todo.md`.
- Commit, push, open a pull request and merge it (approved in `CLAUDE.md`).
