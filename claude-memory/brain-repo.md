---
name: brain-repo
description: "The project's notes live in the GitHub repo imgoingto1/dorohedoro-game; the project folder is a checkout - commit and push log updates"
metadata:
  node_type: memory
  type: reference
  originSessionId: 6ff4ea28-a7e5-40c1-a2cb-7c30329cdfd3
  modified: 2026-09-24T20:19:38.331Z
---

Jay's "brain" for this project is https://github.com/imgoingto1/dorohedoro-game (public). Since
2026-09-24 the local project folder `C:\Users\coolj\Dorohedoro Game Roblox` is a git checkout of it
(branch `main`). Tracked: `CLAUDE.md`, `README.md`, `PROJECT.md`, `work-log/worklog.md`,
`claude-memory/` (a mirror of this memory folder). Everything else in the folder is ignored by an
allowlist `.gitignore` (place exports, graphify, captures, patches).

**How to apply:** after any pass, update `work-log/worklog.md` (and its "Current state" block at the
top) plus `PROJECT.md`, copy changed memory files into `claude-memory/`, then commit and push to
`main`. Jay asked for this: the repo, not local files, is the source of truth. Other Claude sessions
(cloud) also edit it, sometimes through PRs - `git pull` before editing. Git Credential Manager holds
the GitHub login; `gh` is not installed. The repo is public, so keep secrets and emails out.
Related: [[cross-place-script-sync]].
