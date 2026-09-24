# The Sorcerer World — art direction

What the Sorcerer World should feel like, taken from 4 reference images (anime stills) Jay
shared on 2026-09-24. The images are not committed because this repo is public. Keep them locally
in `reference/sorcerer-world/`, which git ignores.

**Status:** planned as its own place, separate from the Hole, sharing data and code with it
through package links (`PROJECT.md`, roadmap). Not started. Nothing exists in-game yet, so this
brief sets the target from scratch rather than fixing an existing map.

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
5. **Whimsy.** Striped towers, onion domes, fairground rides. Sorcerers live in luxury and
   treat the world as a playground.
6. **Masks are normal here.** Sorcerers wear them in the street, so the world should show
   masked NPCs as everyday people. This ties into the mask models in `todo.md`.

Keep designs **inspired by** the series, not copied: no exact buildings, logos or story-specific
elements (such as the flying house in ref 3), since the game is public.

## How to get there in Roblox

**1. Lighting (code only).** A separate lighting profile for this place, applied the same way
`AtmosphereClient` handles the Hole and the Academy:
- Day: low `Atmosphere` density (clear air), teal sky, strong `SunRays`, gentle `Bloom`, warm sun.
- Night: a starry skybox, an aurora made from slow-moving coloured `Beam` curtains high in the
  sky, strong `Bloom` so lights glow.
- Consider a day/night cycle that tilts toward night, so the carnival look is seen often.

**2. String lights (code only).** A script that drapes strings of small Neon parts, or Beams
with a dotted light texture, between attachment points on buildings and rides. It's the same
technique as the Hole's overhead wires, reused. Keep real `PointLight`s rare; Neon plus Bloom
gives the glow cheaply.

**3. Searchlights (code only).** A few sweeping `SpotLight` and `Beam` cones from tall
landmarks at night.

**4. Landmarks (needs assets).** A Ferris wheel and roller coaster (they could turn slowly using
code), the giant spires as the skyline, a domed cathedral, and a large lit tower.

**5. Architecture kit (needs assets).** Ornate facades, arches, domes, bell towers, lampposts,
green copper roofs and statues. This is the heaviest part: detailed Baroque/Gothic meshes from
the Toolbox or a commissioned modeller. Plus overgrowth: trees, vines, roots and moss decals.

**6. Ruins pass.** Rubble piles, broken stairs and collapsed arch sections placed among intact
buildings.

**Performance:** ornate meshes and many lights are costly. Use `StreamingEnabled` like the other
places, Neon plus Bloom instead of real lights, and measure before and after as with the
2026-09-22 optimisation pass.

## Open questions for Jay
- Is this its own place, as `PROJECT.md` plans, and how do players get there (Smoke Doors,
  a portal, quests)?
- Is the night carnival always there at night, or a special event?
- How does Zogan's Academy relate to it: part of this world, or separate?
- What happens here in gameplay: safe hub, PvP zone, quests, shops?
- Where should the assets come from: built, the Toolbox, or a commissioned modeller?
