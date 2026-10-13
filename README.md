# Personal skills

My personal agent skills. Each skill lives in its own directory at the repository root.

## Available skills

- [rust-conventions](rust-conventions/SKILL.md): Conventions for adding or substantially rewriting Rust code.

## Installation

Install with the [GitHub CLI](https://cli.github.com/manual/gh_skill_install).

Install all skills:

```shell
gh skill install vivienm/skills --all --agent universal --scope user
```

Install a specific skill:

```shell
gh skill install vivienm/skills rust-conventions --agent universal --scope user
```

Update with:

```shell
gh skill update rust-conventions
```
