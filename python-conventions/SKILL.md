---
name: python-conventions
description: >-
  Apply personal Python coding conventions to new or substantially rewritten
  Python code.
license: MIT
---

# Python conventions

Follow repository instructions and established local style first. Apply these
preferences only to new or substantially rewritten code.

## Regular expressions

Use `import re` and qualify module members with `re.`.

Prefer `re.compile` where practical, especially for patterns reused across
function calls.

## Function definitions

Keep signatures on one line when they fit the project's line length limit. For
longer signatures, put one parameter per line and add a trailing comma.

```python
def find_record(record_id, source):
    ...


def search_records(
    query,
    source,
    *,
    include_archived_records=False,
    require_verified_source=True,
):
    ...
```
