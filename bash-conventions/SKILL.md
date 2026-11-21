---
name: bash-conventions
description: >-
  Apply personal Bash coding conventions to new or substantially rewritten Bash
  code.
license: MIT
---

# Bash conventions

Follow repository instructions and established local style first. Apply these
preferences only to new or substantially rewritten code.

- Use `set -euo pipefail` at the start of Bash scripts that run in their own
  process.
- Follow the [Google Shell Style Guide] for Bash code style.

[Google Shell Style Guide]:
  https://raw.githubusercontent.com/google/styleguide/refs/heads/gh-pages/shellguide.md
