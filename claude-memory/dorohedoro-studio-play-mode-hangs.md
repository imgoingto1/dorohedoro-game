---
name: dorohedoro-studio-play-mode-hangs
description: "Map + Combat wedges Studio's Play toggle; unstick it by sending F5 with AttachThreadInput"
metadata: 
  node_type: memory
  type: project
  originSessionId: 39577807-33bc-4ea3-a41b-72a41ab16704
  modified: 2026-09-24T01:18:44.700Z
---

The Map + Combat place (placeId 87872916277829) is ~714k instances / ~517k in Workspace, and Studio
sits at ~11.4 GB working set with it open. `mcp__Roblox_Studio__start_stop_play` intermittently
wedges: the stop succeeds, the next start times out at 120s, and every later call returns
`Start play hasn't finished yet` while `get_studio_state` still reports Edit and `execute_luau`
keeps working normally against the Edit datamodel. Seen 2026-09-17 and 2026-09-22.

**Why:** it blocks all live playtesting, which is usually the point of the session.

**How to apply — the fix that works:** send **F5 at the OS level** from PowerShell.
`WScript.Shell.SendKeys` does nothing to Studio's Qt UI, and bare `keybd_event` hits the *wrong
window* because `SetForegroundWindow` is refused from a background process. The working sequence is
`AttachThreadInput` (attach our thread to the current foreground window's thread, which grants
foreground rights) → `BringWindowToTop` → `SetForegroundWindow` → `keybd_event(0x74)`. **Always
verify `GetForegroundWindow() == Studio hwnd` before sending any key** — a stray keystroke once
deleted `GameUI` in this project. Two Studio instances are usually open (Map + Combat and Zogan's
Academy Grounds), so target the right PID. Once Play has actually run, `start_stop_play` works again.

**No-focus alternative (2026-09-23), prefer it:** from Edit, `task.spawn(function()
game:GetService("StudioTestService"):ExecutePlayModeAsync({}) end)` enters Play without touching the
keyboard or focus (`RunService:Run()` does nothing). Catch: the MCP bridge does not attach to that
session at first (`Target is not reachable`); it attached later, after the wedged `start_stop_play`
request finished, and then `start_stop_play(false)` stopped it normally. The user games on this PC
(VALORANT) — never take focus while a game is in the foreground, and auto mode refuses unattended
"press keys later" watchers.

`game:SavePlace()` is **not** callable from Edit ("can only be called from a server script"), so the
place cannot be saved programmatically before a restart. Team Create is on, so edits sync anyway —
but back up any script edits to `patches/<date>/` first. See also
[[execute-luau-module-cache-is-isolated]].
