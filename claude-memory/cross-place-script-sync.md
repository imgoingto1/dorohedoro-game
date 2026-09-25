---
name: cross-place-script-sync
description: "How to sync scripts from Map + Combat to Zogan's Academy (two Studio processes, no shared clipboard) and why the packages don't do it"
metadata:
  node_type: memory
  type: project
  originSessionId: 6ff4ea28-a7e5-40c1-a2cb-7c30329cdfd3
  modified: 2026-09-25T01:23:56.484Z
---

Map + Combat (placeId 87872916277829) is the source of truth; Zogan's Academy Grounds (127609270845586)
must mirror its game scripts. The shared folders are Roblox Packages (ServerScriptService.World,
ReplicatedStorage.World, WorldClient, PlayerData, DoorGui/ShopGui/SmokeGui/WorldUI), but as of
2026-09-24 Map + Combat's edits were never *published* to them, so "Update All" does nothing - and
publishing can't be scripted (UI right-click only).

**What worked (2026-09-24, 90 scripts):** execute_luau in each Studio. Hash every LuaSourceContainer
(djb2-style, in both places) to find diffs; for changed files send the target's 4-hex-per-line hashes to
the source, compute LCS hunks there, apply hunks in the target, and verify the full-file hash before
writing `.Source`. New files: dump whole source, create with Instance.new (LocalScripts under
StarterPlayerScripts could be created this way), verify hash. Prefix long-bracket strings with "|" so a
leading newline isn't eaten. End state was checked with one combined hash over all 525 scripts.
Luau drops the newline right after `[==[` by itself: write `m.Source = [==[\n...]==]` with NO `:sub(2)`.
Adding `:sub(2)` ate the first `-` of every new script on 2026-09-24 (line 1 parse error).

**Big packages (react-lua, 8 MB / 1572 scripts) can't go through context:** insert the same asset in
each place with `insert_asset` and re-apply any local fix by script, then compare a combined hash.
The react-lua fix is logged in the work log (2026-09-24). Parity check scope: everything except
`ServerStorage.Backup_*` / `ServerStorage.Archive*` (place-specific) and Workspace (map scripts differ).

**Why:** there is no data channel between the two Studio processes except my own context; the
http_get tool only allows Roblox docs URLs. Hunks cut the transfer to ~1/4 of full copies.

**How to apply:** redo the hash comparison first - never assume Academy is current. Pit code
(PitService/PitClient/PitAction) is intentionally gone from both. Related: [[execute-luau-module-cache-is-isolated]],
[[dorohedoro-studio-play-mode-hangs]].
