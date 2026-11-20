---
name: rust-conventions
description: >-
  Apply personal Rust coding conventions to new or substantially rewritten Rust
  code.
license: MIT
---

# Rust conventions

Follow repository instructions first. Apply these preferences only to new or
substantially rewritten code. This excludes code copied or moved unchanged.

## Modules

For modules with submodules, prefer `foo.rs` over `foo/mod.rs`.

Place `use` imports before `mod foo;` declarations, preserving any ordering
required by macros.

## Imports

Prefer direct imports or short, meaningful module qualifiers for types and
traits, even when used only once.

- Import distinctive types directly, such as `Path`, `HashMap`, and
  `PhantomData`.
- Use short qualifiers to clarify origin or distinguish APIs, as in
  `io::Result`, `fs::read_to_string`, and `mpsc::Sender`. Import the module
  alongside distinctive members when useful, as in `fmt::{self, Formatter}`.
- Resolve collisions, including with derive macros, with a meaningful module
  qualifier or descriptive alias, such as `error::Error` or `Error as StdError`.
- Keep fully qualified paths when they improve clarity.
- For items in child modules, prefer `use self::child::Item` over
  `use child::Item`.
- Organize imports into three groups: standard library (`std`, `core`, `alloc`),
  external crates, and local imports (`self`, `super`, `crate`), in that order.
  Separate groups with a blank line.
- Within each group, combine imports from the same crate into a single `use`
  declaration, using nested brace trees to factor out shared path prefixes.
  Apply the same grouping to local imports sharing a `crate`, `self`, or `super`
  root. Keep imports with different visibility or attributes separate.

```rust
use std::{
    fs::{self, File},
    io,
    path::Path,
};

use serde::{Deserialize, Serialize};

use crate::{config::Config, input::Reader};
```

## Path parameters

When practical, functions and methods that accept a filesystem path should use a
generic parameter bounded by `AsRef<Path>` instead of `&Path`, especially in
public APIs. Use `path.as_ref()` inside the body when a `&Path` is needed.

## Derives

Keep `Copy`, `Clone`, `Eq`, `PartialEq`, `Ord`, `PartialOrd`, `Hash`, `Debug`,
`Display` (when supplied by a derive macro), and `Default` in that relative
order. List other external derives, such as `Serialize` and `Deserialize`, next,
followed by local derives such as `CrateSpecificTrait`.
