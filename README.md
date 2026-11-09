# Personal skills

Personal agent skills. Each skill has its own top-level directory.

## Available skills

- [github-conflicts](github-conflicts/SKILL.md): Resolve conflicts in stacked
  and standalone GitHub pull requests.
- [python-conventions](python-conventions/SKILL.md): Conventions for adding or
  substantially rewriting Python code.
- [review-loop](review-loop/SKILL.md): Iterate on a change set with independent
  reviews by native subagents.
- [rust-conventions](rust-conventions/SKILL.md): Conventions for adding or
  substantially rewriting Rust code.

## Installation

Install with the [GitHub CLI](https://cli.github.com/manual/gh_skill_install).

For stacked PRs, `github-conflicts` also requires the
[`gh-stack` skill](https://github.com/github/gh-stack/tree/main/skills/gh-stack)
and the [`gh stack` extension](https://github.com/github/gh-stack#installation).

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

## Validation

Run `gh skill publish --dry-run` to validate skill structure.

For a one-off check of Markdown formatting and prose wrapping at 80 columns:

```shell
npx --yes prettier --check --prose-wrap always --print-width 80 '**/*.md'
```
