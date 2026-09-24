# Dorohedoro game — project brain

## What this repo is for

This repository is the **brain / backbone** for Jay's local station: the machine running Roblox
Studio (Map + Combat and Zogan's Academy Grounds) and the Claude sessions that drive it. The game
itself lives in the Roblox places, not here. This repo holds the knowledge that tells the local
station what has been done, what is open, and how work is expected to be carried out.

**For the local station:** when you need context before acting, such as history, prior decisions,
known gotchas, or what Jay has already said yes or no to, **search this repo first** and treat
it as input. Record new work here so the next session starts with it.

## Where things are

- [`work-log/worklog.md`](work-log/worklog.md): the running log of everything done, newest at
  the bottom. Start with its **"Current state"** section at the top for open items, untested
  work and decisions waiting on Jay. `PROJECT.md` (on the local station) is the more detailed
  source the log was pulled from.
- [`todo.md`](todo.md): the ideas and task list by area (VFX, animations, atmosphere, UI,
  models, masks, combat feel). Items tagged **[idea]** still need Jay's approval.

## Standing rules learned from the log

- **Ask before inventing** story, new items or balance targets. Tell Jay what an idea is before
  building it.
- **Keep both places at parity.** Apply to Map + Combat first, read back, mirror to Zogan's, and
  read back again.
- **Never run Play mode in both places at once.** They share one universe and fight over the
  same ProfileService session lock, which looks exactly like data loss.
- **Never `require` a server module from `execute_luau`** if it connects events at top level. The
  injected copy has empty state and shadows the real handler. Drive real modules through
  WorldSignals, remotes or Cmdr instead.
- **Verify with the Cmdr `selftest`** (`workspace:SetAttribute("RunSelfTest", true)`): the
  expected result is 18 passed / 0 failed. Measure performance changes before applying them.
- **Don't take OS screenshots while Jay is at the machine.** Use MCP `screen_capture` instead.
- Prefer disabling over deleting (reversible), and log every change in the work log with what
  was and wasn't tested.

## Working on this repo

Jay has approved Claude opening and merging pull requests in this repo without asking first
(2026-09-24). Commit changes to the work log, `todo.md` and these docs, open a pull request, and
merge it once it's mergeable.
