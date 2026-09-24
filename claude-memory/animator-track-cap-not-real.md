---
name: animator-track-cap-not-real
description: "Roblox's \"256 AnimationTracks per Animator\" cap did not reproduce in this game; don't blame animation breakage on it"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 6ff4ea28-a7e5-40c1-a2cb-7c30329cdfd3
  modified: 2026-09-24T20:13:17.027Z
---

On 2026-09-24 I blamed "animations break randomly" on the 256-loaded-tracks-per-Animator limit and
said so to Jay before testing. A control test then loaded 300 tracks on one Animator, held them alive,
and played each - no failure and no warning, on both server and client. I had to retract it.

The real causes found that day were the Perfect Dodge client effect stopping every playing track
(idle/walk included) and AnimationController never restarting a stopped pose; also
`Enum.AnimationPriority.Core` has value 1000, so `priority.Value >= Action.Value` wrongly includes Core
walk tracks.

**Why:** a confidently stated wrong cause cost trust and an extra round of testing.

**How to apply:** for animation bugs, reproduce before naming a cause, and treat engine-limit theories
as hypotheses until a control test hits them. Related: [[execute-luau-module-cache-is-isolated]].
