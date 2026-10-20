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

### Choosing what to import

Prefer imports that keep module bodies readable. Import a commonly used type
or trait directly, or import its module when that gives a useful short
qualifier. Do not wait for a name to occur multiple times before importing it.

- Import distinctive types directly: `Path`, `PathBuf`, `OsStr`, `OsString`,
  `HashMap`, `HashSet`, `BTreeMap`, `BTreeSet`, `Rc`, `Arc`, `Mutex`, `Duration`,
  `Instant`, etc. Prefer `Path` over `std::path::Path` or `path::Path` in
  signatures and expressions.
- Import utility modules such as `io`, `fs`, `env`, and `process` when useful.
  Keep meaningful qualifiers such as `io::Result`, `io::Error`,
  `io::ErrorKind`, and `fs::read_to_string`, rather than importing every
  member directly. This can coexist with direct imports such as `File`,
  `Read`, and `Write`: use `fs::{self, File}` or `io::{self, Read}`.
- For formatting implementations, prefer `fmt::{self, Formatter}` and write
  `Formatter` and `fmt::Result`. Import `Display` or `Debug` directly
  when useful; retain a qualifier or alias when needed to avoid a collision
  with a derive macro or another name.
- Apply the same readability criterion to external crates and local modules.
  A module qualifier can distinguish related APIs, as in `mpsc::Sender` and
  `oneshot::Sender`, or identify a domain, as in `image::Error`.
- Resolve collisions with a useful module qualifier or a descriptive alias,
  such as `error::Error` or `Error as StdError`. For traits needed only for
  method resolution, prefer `as _` when their name is otherwise unnecessary,
  e.g. `use anyhow::Context as _;` or `use std::fmt::Write as _;`. Keep named
  imports for traits used in bounds or implementations.

These are defaults, not a ban on fully qualified paths. Keep one when it
improves clarity or avoids a collision; occasional utility calls and derive
macro paths need not all become imports. Import only what is used.

For example:

```rust
use std::{
    fmt::{self, Display, Formatter},
    fs,
    io,
    path::Path,
};

fn read_config<P: AsRef<Path>>(path: P) -> io::Result<String> {
    fs::read_to_string(path)
}

struct Label(String);

impl Display for Label {
    fn fmt(&self, f: &mut Formatter) -> fmt::Result {
        f.write_str(&self.0)
    }
}
```

### Grouping and layout

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
