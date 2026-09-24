---
name: execute-luau-module-cache-is-isolated
description: Roblox MCP execute_luau has its own require cache; requiring a game module from it creates a second live instance that shadows the real one
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 39577807-33bc-4ea3-a41b-72a41ab16704
  modified: 2026-09-24T15:36:19.237Z
---

`mcp__Roblox_Studio__execute_luau` runs in a script context with a **module cache separate from
the game's real server scripts**. `require(SomeModule)` from injected code returns a *second, fresh*
instance whose top-level state was never initialised — not the instance the running game uses.

Three verified consequences:
- Injected requires share a cache with *each other* (a tag set in one call is visible in the next),
  which makes the isolation easy to mistake for sharing. Confirm against real state instead.
- `_G` and **connections registered from injected code persist across calls** for the whole session.
- Roblox dispatches signal handlers **last-connected-first**, so an injected copy connected later
  will win any race against the real handler. With a one-shot guard (`if flag then flag = false; run() end`)
  the real handler then never runs at all.

**Why:** this silently produced a fake bug. `ServerScriptService.World.SelfTest` reported
`FAIL player data loaded + versioned` for a whole session because the injected copy's `Profiles`
table was empty (`OnStart` never ran in it), while the real game's data layer was working fine —
proven by a real `ShopBuy` debiting Yen and by `WorldSignals.SpendYen` returning true.

- Requiring `ReplicatedStorage.Remotes.UnPackets` from injected **Client** code (to fire real input
  packets) works, but the second Packet instance only knows its own packet ids, so every server packet
  then logs `Packets.Packet:347: attempt to index nil with 'ResponseReads'`. That is harness noise; a
  playtest without the injected require stays clean (verified 2026-09-23).

**SelfTest specifically:** run it ONLY with `workspace:SetAttribute("RunSelfTest", true)` and then poll
`SelfTestResult` / `SelfTestReport` (the runner clears `RunSelfTest` before the result lands). Never
`require(World.SelfTest)` - it was done again on 2026-09-24 and faked the same 4 player-data failures
until a fresh Play session.

**How to apply:** to exercise a real server module, drive it through an entry point the real context
owns — a `ServerStorage.WorldSignals` Bindable, a remote invoked from the `Client` datamodel, or a
Cmdr command — never `require()` it from `execute_luau`. If a probe's result contradicts observed
gameplay, suspect the probe first. See also [[dorohedoro-studio-play-mode-hangs]].
