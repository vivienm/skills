# Personal skills

Personal agent skills. Each skill has its own top-level directory.

## Available skills

- [github-conflicts](github-conflicts/SKILL.md): Resolve conflicts in stacked
  and standalone GitHub pull requests.
- [python-conventions](python-conventions/SKILL.md): Conventions for new and
  substantially rewritten Python code.
- [review-loop](review-loop/SKILL.md): Run independent review-and-fix passes
  with native subagents.
- [rust-conventions](rust-conventions/SKILL.md): Conventions for new and
  substantially rewritten Rust code.

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

Update all installed skills:

```shell
gh skill update --all
```

Update a specific skill:

```shell
gh skill update rust-conventions
```

For stacked PRs, install the `gh-stack` skill and extension as well.

```shell
gh extension install github/gh-stack
gh skill install github/gh-stack gh-stack --agent universal --scope user
```

## Validation

Run `gh skill publish --dry-run` to validate skill structure.

For a one-off check of Markdown formatting and prose wrapping at 80 columns:

```shell
npx --yes prettier --check --prose-wrap always --print-width 80 '**/*.md'
```
