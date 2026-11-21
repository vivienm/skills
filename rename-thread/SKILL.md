---
name: rename-thread
description: >-
  Rename the current conversation or workspace when a user or authorized
  workflow requests a title change.
license: MIT
---

# Rename thread

Use a native rename action in the app that displays the current title. Check
which tools and schemas are exposed in the current session.

- **Paseo:** For the sidebar title, prefer `mcp__paseo__rename_workspace`. Pass
  the requested title using the exposed schema and omit `workspaceId` when the
  tool defaults to the current workspace. When either action needs an ID, use
  `PASEO_WORKSPACE_ID` if nonempty or confirm the current workspace ID. If the
  tool is unavailable, assign the resolved ID to `WORKSPACE_ID` and the literal
  requested title to `TITLE`. On the active daemon, run
  `paseo workspace rename -- "$WORKSPACE_ID" "$TITLE"`.
- **Codex Desktop:** Discover and use `set_thread_title` if exposed.
- **OpenCode:** If exposed, call `opencode.session_rename` in Code Mode and omit
  `sessionID` to target the current session. [OpenCode tools]

If those options do not apply, look for an equivalent native action in the
active app. If it requires an ID, confirm that the ID belongs to the current
conversation or workspace. Leave session storage untouched.

Verify the resulting title when possible. If the rename fails or no supported
action can target the current conversation, report the blocker.

[OpenCode tools]: https://opencode.ai/docs/tools
