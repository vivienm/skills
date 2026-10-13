---
name: rust-conventions
description: Apply personal Rust coding conventions when adding or substantially rewriting Rust code. Do not use to enforce style on unchanged code.
license: MIT
---

# Rust conventions

Follow these preferences unless repository instructions specify otherwise.

Apply them only to code you add or substantially rewrite. Do not flag or
change existing code solely to enforce them, including code copied or moved
unchanged. Do not perform unrelated style cleanups.

## Coding style

- For modules with submodules, prefer `foo.rs` over `foo/mod.rs`.
- For imports from the current module, prefer `use self::foo` over `use foo`.
- Organize new or substantially rewritten imports into three groups: standard
  library (`std`, `core`, `alloc`), external crates, and local imports (`self`,
  `super`, `crate`), in that order. Separate groups with a blank line. Do not
  regroup existing imports solely to enforce this preference.
- When deriving any of the following traits, use this relative order: `Copy`,
  `Clone`, `Eq`, `PartialEq`, `Ord`, `PartialOrd`, `Hash`, `Debug`, `Display`,
  `Default`. Place any other derived traits after these, with traits from
  external crates (e.g. `Serialize`, `Deserialize`) first, followed by traits
  from the local crate (e.g. `CrateSpecificTrait`).
