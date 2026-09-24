---
name: playtest-prompts-dont-fire
description: "MCP-driven Studio playtests can't trigger ProximityPrompts; use the Studio-only WorldSignals.DebugTalk hook"
metadata:
  node_type: memory
  type: reference
  originSessionId: 019b5288-0fad-40fe-90cd-e255df773d9c
  modified: 2026-09-24T06:15:51.330Z
---

In playtests driven through the Roblox Studio MCP tools, ProximityPrompts never show (PromptShown
never fires), so neither `user_keyboard_input` E nor `prompt:InputHoldBegin/End` can open an NPC
dialogue - most likely because the Studio window isn't focused/rendering. Verified 2026-09-24.

**How to apply:** open NPC dialogues with `ServerStorage.WorldSignals.DebugTalk:Invoke(player, npcModel)`
(Studio-only, added to `NPCService`; runs the exact Talk path and returns openedTreeId, lock). Then send
choices from the Client datamodel with `ReplicatedStorage.World.Remotes.DialogueAction:FireServer(npc,
nodeKey, choiceIndex)` - the player must be within `DialogueBreakDistance` of the NPC. Number keys
1-9 also can't be sent (reserved by the CoreGui backpack). See also [[execute-luau-module-cache-is-isolated]].
