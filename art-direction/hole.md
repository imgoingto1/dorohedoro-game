# The Hole — art direction

What the Hole map should feel like, taken from 5 reference images Jay shared on 2026-09-24
(anime stills and one manga panel). The images are not committed because this repo is public.
Keep them locally in `reference/hole/`, which git ignores.

**The problem today:** much of the map came from jjk's Shibuya chunks. It reads as a clean,
modern Tokyo: wide roads, glass, neat facades, open sky. The Hole is the opposite: cramped,
vertical, patched together, industrial and choked with smog.

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
from the show, since the game is public.

## How to get there in Roblox

Ordered from cheapest and code-only to heaviest.

**1. Atmosphere and lighting (code only).** A starting point, to be tuned by eye:
- `Atmosphere`: raise `Density` and `Haze`, set the colour to sepia-green (roughly
  `rgb(150,150,120)`) and `Decay` to a darker brown-grey, so the distance fades into smog.
- `ColorCorrection`: lower saturation and contrast a little, with a slight warm-green tint.
- Street lamps: warm sodium colour (roughly `rgb(255,190,110)`) in `LampClient`, so night gets
  the yellow-pool-in-fog look of reference 1.
- Tie these into `AtmosphereClient`'s existing Hole zone mood, not a separate script.

**2. Overhead wires (code only, and the biggest win per hour).** A script that strings sagging
`Beam`s between attachment points on facing buildings along each street, using the Beam's
`CurveSize0/1` for the sag. Beams are cheap to render and work with streaming.

**3. Skyline (asset-light).** A ring of distant smokestack, furnace and gantry silhouettes
beyond the playable edge, built from simple meshes or parts, with slow dark smoke particles on
top. Seen through the smog layer, low detail is enough.

**4. Dressing kit (needs assets).** Reusable props to layer onto the existing buildings: fire
escapes, external pipes, AC units, corrugated panels, stacked shacks, hanging round signs,
lanterns, banners, posters and charms, barrels, hazard barriers. Layering these over the
Shibuya facades changes the feel without rebuilding every building.

**5. Blockout changes (big).** Narrow the widest roads with shack rows and stalls. Add a
landmark red gate over a main road (ref 4), a tiered central tower with a lit plaza (ref 2), and
a tram line with tram models (ref 1). Remove or cover modern elements: glass towers, clean
signage, modern shopfronts.

**Performance:** the map already streams, and fog hides distant detail, which helps. Measure
before and after, as with the 2026-09-22 optimisation pass. Check new props'
`CastShadow` and `RenderFidelity` settings, since shadows are already the main cost.

## Open questions for Jay
- Which area to prototype first: a single alley, or the main street with trams?
- Keep the Shibuya buildings and dress over them, or rebuild districts from a new kit?
- Where to source assets: build them, use the Toolbox, or commission a modeller?
