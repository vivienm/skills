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

## Modules

For modules with submodules, prefer `foo.rs` over `foo/mod.rs`.

## Imports

- For imports from the current module, prefer `use self::foo` over `use foo`.
- Organize imports into three groups: standard library (`std`, `core`, `alloc`),
  external crates, and local imports (`self`, `super`, `crate`), in that order.
  Separate groups with a blank line.
- Within each group, combine imports from the same crate into a single `use`
  declaration, using nested brace trees to factor out shared path prefixes.
  Apply the same grouping to local imports sharing a `crate`, `self`, or
  `super` root. Keep imports with different visibility or attributes separate.

For example:

```rust
use std::sync::{
    atomic::{AtomicI64, Ordering},
    Arc, Mutex,
};

use crate::{
    config::{Config, Mode},
    input::Reader,
    output::{Format, Writer},
    storage::Store,
};
```

Keep this formatting local to imports you add or substantially rewrite;
do not regroup existing imports or reformat unrelated code solely to enforce
these preferences.

## Derives

When deriving any of the following traits, use this relative order: `Copy`,
`Clone`, `Eq`, `PartialEq`, `Ord`, `PartialOrd`, `Hash`, `Debug`, `Display`,
`Default`. Place any other derived traits after these, with traits from
external crates (e.g. `Serialize`, `Deserialize`) first, followed by traits
from the local crate (e.g. `CrateSpecificTrait`).
