---
name: rename-thread
description: >-
  Rename the current agent conversation or workspace when the user asks to
  change its visible title.
license: MIT
---

# Rename thread

Apply the requested title to the current conversation in the app that displays
it. Use the native rename action for that app.

- **Paseo:** For the sidebar title, call `mcp__paseo__rename_workspace` with
  `title` and omit `workspaceId` to target the current workspace. If the tool is
  unavailable, use `paseo workspace rename -- <workspace-id> <title>` with the
  confirmed workspace ID on the active daemon. Pass the title as one argument.
- **Codex Desktop:** Discover and use `set_thread_title` if exposed.
- **OpenCode:** Call `opencode.session_rename` in Code Mode and omit `sessionID`
  to target the current session. [OpenCode tools]

If those options do not apply, look for an equivalent native action in the
active app. If it requires an ID, confirm that the ID belongs to the current
conversation or workspace. Leave session storage untouched.

Verify the resulting title when possible. If the rename fails or no supported
action can target the current conversation, report the blocker.

[OpenCode tools]: https://opencode.ai/docs/tools
