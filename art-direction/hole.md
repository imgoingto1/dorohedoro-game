# The Hole — art direction

What the Hole should feel like, taken from 5 reference images Jay shared on 2026-09-24 (anime
stills and one manga panel). The images are not committed because this repo is public. Keep
them locally in `reference/hole/`, which git ignores.

**Context for Game C:** this brief predates the Game C merger audit and restart
([`docs/game-c-merger-audit.md`](../docs/game-c-merger-audit.md)) but targets the same hub —
the Hole is still one of Game C's two launch places (round 2 #5). Revised round 28 for Game C's
tone: near-universal PvP (round 19 #74) and a hard faction line (round 21 #83), on top of the
original references. Everything below is the visual target for a **fresh place** (round 18
#70), not a fix to the old Map + Combat map.

## What the references show

| # | Scene | What to take from it |
|---|---|---|
| 1 | A street with two old blue trams | Tall 6–10 storey concrete blocks with dense balconies, fire escapes and pipes on the outside. Smokestacks rise *between* the housing. A tangle of overhead wires and tram lines. Warm sodium-yellow lamps on tall poles glowing in fog. Low red-roofed kiosks. A hazard-striped barrier. Hazy, low-contrast, sepia-green air. |
| 2 | Manga panel, a big central building at night | A stepped, tiered tower lit warm orange from inside, with a large round emblem and huge painted characters on the front. A crowded plaza with steps and puffs of smoke. Deep blue night sky. A landmark you can navigate by. |
| 3 | The industrial skyline | Blast furnaces, steel lattice towers, conveyor gantries, chimneys pouring black smoke. Ruined high-rises with collapsed facades. Thick fog at ground level, grey overcast sky. Almost monochrome. |
| 4 | A road under giant red gate pillars | Huge red torii-style pillars with a round medallion across the road. Tiled-roof shops beside patched corrugated shacks. A cracked road with debris. Posters and paper charms stuck on the pillars. Rusty barrels. |
| 5 | A narrow alley (a gyoza banner in view) | Makeshift shacks of corrugated metal and wood stacked up and leaning out over the alley. Mismatched coloured panels (teal, purple, orange, rust). Barred windows, hanging round signs and lanterns. Sagging wires crossing overhead. Fog, and an industrial tower looming at the end. |

## The rules of the look

1. **Narrow and vertical.** Streets should feel squeezed, with buildings rising above the
   player on both sides and little sky visible.
2. **Nothing is new.** Every surface is weathered, patched, rusted or stained. No glass towers,
   clean signs or modern shopfronts.
3. **Industry is always in view.** Smokestacks, furnaces and gantries fill the skyline, and
   smoke is always rising.
4. **Wires everywhere.** Dense, sagging overhead cables across nearly every street. This is the
   cheapest, most recognisable signature.
5. **Smog.** Thick, low-contrast, greenish-sepia haze by day. At night, dark blue with warm
   sodium-yellow lamp pools.
6. **Patchwork colour.** A grey-brown concrete base with scattered, mismatched accent panels
   (teal, purple, rust orange, faded red), never a clean palette.
7. **Hand-painted signage.** Big painted characters, round emblems, banners and paper posters
   instead of modern backlit signs.

Keep designs **inspired by** the series, not copied: no exact logos, emblems or text lifted
from the show. Closed-community-first (round 21) means canon names and locations (En's Kitchen,
etc.) are fair game as content, but original art — same as everywhere else in this design.

## No separate "danger zone" look (round 28)

There is no discrete "PvP zone" visual layer to design. In Game C almost the entire Hole is
open PvP by default (round 19 #74) — the grit and menace *this brief already describes* is what
danger looks like here; it doesn't need a second, extra-hazardous treatment layered on top. The
only place that needs to look **different** from the rest of the Hole is a **safe zone**.

## Night of the Living Dead (round 29)

The Hole's manifestation of the global Blue Night event — renamed so it doesn't share a name
with Sorcerer World's carnival version, see `docs/game-c-merger-audit.md`, Events. Zombies hit
the streets for the event's ~5-minute window; no visual identity for it is designed yet beyond
"the same Hole, now with zombies in it" — worth its own pass later (fog thickening, maybe a
colour-correction shift) once the base look above is built and there's something to react
against.

## Faction HQs stay visually neutral (round 28)

Unlike Sorcerer World, **the Hole's faction HQs get no faction colour or banner treatment** —
Jay's call was "the Hole would be completely neutral, like in Dorohedoro": En's Kitchen isn't a
glowing faction-branded compound, it's just a real, recognisable building using the Hole's own
patchwork-industrial language. Each faction's Hole HQ (its one safe zone here, round 20 #81)
should read as a **landmark** — distinctive architecture, maybe a stronger torii-style gate or a
tiered tower like reference 2 or 4 — without a faction accent colour anywhere on it. "Safe" is
communicated by it being a recognisable, defensible-looking building, not by paint.

## How to get there in Roblox

Ordered from cheapest and code-only to heaviest. This is a from-scratch place (round 18 #70), so
these are written as what to build, not what to patch — though the same technique/config
approach (`AtmosphereClient`-style zone mood, `LampClient`-style street lamps) is worth reusing
even though the code itself is rebuilt fresh.

**1. Atmosphere and lighting (code only).** A starting point, to be tuned by eye:
- `Atmosphere`: raise `Density` and `Haze`, set the colour to sepia-green (roughly
  `rgb(150,150,120)`) and `Decay` to a darker brown-grey, so the distance fades into smog.
- `ColorCorrection`: lower saturation and contrast a little, with a slight warm-green tint.
- Street lamps: warm sodium colour (roughly `rgb(255,190,110)`).
- **Shadows stay on by default** (round 28 — Game C defaults to the rich look everywhere, with
  a player-facing quality toggle for anyone who needs the FPS, not a per-hub split).

**2. Overhead wires (code only, and the biggest win per hour).** A script that strings sagging
`Beam`s between attachment points on facing buildings along each street, using the Beam's
`CurveSize0/1` for the sag. Beams are cheap to render and work with streaming.

**3. Skyline (asset-light).** A ring of distant smokestack, furnace and gantry silhouettes
beyond the playable edge, built from simple meshes or parts, with slow dark smoke particles on
top. Seen through the smog layer, low detail is enough.

**4. Dressing kit (needs assets).** Reusable props: fire escapes, external pipes, AC units,
corrugated panels, stacked shacks, hanging round signs, lanterns, banners, posters and charms,
barrels, hazard barriers.

**5. Blockout (big).** Narrow streets with shack rows and stalls. A landmark red gate over a
main road (ref 4), a tiered central tower with a lit plaza (ref 2), a tram line (ref 1), and
each faction's HQ landmark (see above) somewhere defensible-feeling.

**Performance:** streaming plus fog hides distant detail. Measure before and after any pass, and
check new props' `CastShadow`/`RenderFidelity`, since shadows are the main cost once the quality
setting is on.

## Open questions for Jay

- Which area to prototype first: a single alley, or the main street with trams?
- Build a new district kit from scratch, or start from Toolbox pieces and iterate?
- Where should each faction's Hole HQ actually sit on the map, and what makes it read as
  "defensible" architecturally (walls, a single choke-point entrance, elevation)?
- Where to source assets: build them, use the Toolbox, or commission a modeller?
