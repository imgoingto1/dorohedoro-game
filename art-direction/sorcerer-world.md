# The Sorcerer World — art direction

What the Sorcerer World should feel like, taken from 4 reference images (anime stills) Jay
shared on 2026-09-24. The images are not committed because this repo is public. Keep them
locally in `reference/sorcerer-world/`, which git ignores.

**Context for Game C:** this brief predates the Game C merger audit and restart
([`docs/game-c-merger-audit.md`](../docs/game-c-merger-audit.md)). It's no longer a "someday"
region — Sorcerer World is one of Game C's two launch hubs (round 2 #5), and the Academy tutorial
lives inside it (round 7 #25). Revised round 28 for Game C's tone: this hub is fully PvP outside
faction HQs and the Academy (round 19 #75, round 20 #81), so the beauty this brief describes now
sits directly next to real danger — that contrast is part of the point, not softened for it.

## What the references show

| # | Scene | What to take from it |
|---|---|---|
| 1 | An overgrown city under giant spires | Huge needle-like stone spires, half rock and half carved building, towering over everything. A blue-domed, cathedral-like building. Ornate facades in mixed colours (purple-tiled walls, a deep red tower). Trees, vines and roots growing over the stonework, with statues in niches. Light rays falling through leaves. A teal-green cloudy sky. Lush and ancient. |
| 2 | A street by day, with masked sorcerers | Cream and white European-style palaces carved in fine detail. A crumbling triumphal arch, rubble and broken stairs. Black-and-gold double-headed ornate lampposts. A bell tower and spires topped with crosses, green copper roofs, cypress trees. A bright, clear teal sky. People in everyday clothes, some wearing masks. |
| 3 | A festival at night | A Ferris wheel covered in patterned string lights. A roller-coaster track dotted with pink glowing lights. Striped, rocket-shaped towers and a warm golden lit tower. A starry sky with aurora-like colour bands and jagged purple mountains behind. Strings of lights draped between everything. Extremely saturated. |
| 4 | A giant red structure in the night sky | A huge red rounded structure with rows of windows and searchlight beams sweeping from it. Candy-striped onion domes, tree-like spires, string lights and a dense starry blue sky. Whimsical and grand. |

## The rules of the look

1. **The opposite of the Hole.** The Hole is cramped, grey, industrial and Asian-urban. The
   Sorcerer World is grand, colourful and European-ornate, with clean air and big skies.
2. **Old-world architecture at huge scale.** Baroque and Gothic palaces, domes, arches, bell
   towers and spires, dwarfed by colossal rock-and-stone needles.
3. **Beauty in decay.** By day the city is partly ruined: crumbled arches, rubble, broken stairs,
   and nature reclaiming it with trees, vines and roots.
4. **Two moods.** Day is bright, clear and teal-skied, with soft light rays. Night is a
   carnival: saturated colour, string lights on everything, searchlights, stars and aurora.
5. **Whimsy that's not safety.** Striped towers, onion domes, fairground rides — sorcerers live
   in luxury and treat the world as a playground, but this same gorgeous ground is fully PvP the
   moment you step outside a faction HQ or the Academy (round 28). The prettier it looks, the
   less it should be trusted.
6. **Masks are normal here.** Sorcerers wear them in the street, so the world should show masked
   NPCs as everyday people — ties into the identity-roll cosmetic system (round 25 #103).

Keep designs **inspired by** the series, not copied: no exact buildings, logos or story-specific
elements (such as the flying house in ref 3). Closed-community-first (round 21) allows canon
names as content, not art assets lifted from the show.

## No separate "danger zone" look (round 28)

Same rule as the Hole: there's no extra-hazardous visual layer to add on top of the base look.
Almost all of Sorcerer World is open PvP by default (round 19 #75); the references' own
grandeur-plus-ruin already carries the tension. The only places that need to look **different**
are the **safe zones** — faction HQs and the Academy.

## Faction HQs get real identity here — the Academy doesn't (round 28)

Unlike the Hole, **each faction's Sorcerer World HQ gets its own colour and banner identity** —
Jay's call was "like in Dorohedoro": distinct home turf per faction, built from this brief's own
palette (En's Family and Cross-Eyes each get an accent pulled from the existing ornate look —
gold/warm vs. a cooler contrast, or similar — not a clash with the world's own colours). This is
the one place in the whole game where a faction visually owns ground.

**The Academy stays neutral.** It's shared tutorial ground for both factions' new players
(round 7 #25) and the game's other safe zone here — no faction colour, same rule as the Hole's
neutral HQs, just for a different reason (shared, not gang-owned).

## How to get there in Roblox

**1. Lighting (code only).** A separate lighting profile for this place, the fresh-place
equivalent of what `AtmosphereClient` did for the Hole and Academy before:
- Day: low `Atmosphere` density (clear air), teal sky, strong `SunRays`, gentle `Bloom`, warm sun.
- Night: a starry skybox, an aurora made from slow-moving coloured `Beam` curtains high in the
  sky, strong `Bloom` so lights glow.
- **Shadows stay on by default** (round 28), same rich-look-everywhere call as the Hole.

**2. String lights (code only).** A script that drapes strings of small Neon parts, or Beams
with a dotted light texture, between attachment points on buildings and rides — the same
technique as the Hole's overhead wires, reused. Keep real `PointLight`s rare; Neon plus Bloom
gives the glow cheaply.

**3. Searchlights (code only).** A few sweeping `SpotLight` and `Beam` cones from tall
landmarks at night.

**4. Landmarks (needs assets).** A Ferris wheel and roller coaster (turning slowly via code),
the giant spires as the skyline, a domed cathedral, a large lit tower, and each faction's HQ
(see above) as a real, distinct building.

**5. Architecture kit (needs assets).** Ornate facades, arches, domes, bell towers, lampposts,
green copper roofs and statues — the heaviest part, detailed Baroque/Gothic meshes from the
Toolbox or a commissioned modeller. Plus overgrowth: trees, vines, roots, moss decals.

**6. Ruins pass.** Rubble piles, broken stairs and collapsed arch sections among intact
buildings.

**Performance:** ornate meshes and many lights are costly. Use `StreamingEnabled`, Neon plus
Bloom instead of real lights, and measure before and after any pass — the Ferris wheel/aurora
Beams especially, since they're always-on effects across the whole hub at night.

## Open questions for Jay

- Is the night carnival always on, or does it come and go (e.g. tied to Blue Night, round 24
  #86, rather than running independently)?
- Where do the two faction HQs sit relative to each other and to the Academy — adjacent
  districts, opposite ends, something else?
- Does the raid (round 24 #96) live in Sorcerer World, the Hole, or its own instanced space? Not
  addressed by this brief or the design doc yet.
- Where should assets come from: build them, use the Toolbox, or commission a modeller?
