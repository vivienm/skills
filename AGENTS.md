# AGENTS.md

## Project layout

Put each skill in a root-level directory with a `SKILL.md` file. Match the
frontmatter `name` to the directory name and describe when to use the skill in
the `description` field. Update the list in `README.md` when adding, renaming,
or removing skills.

## Markdown formatting

With npm installed, run this command from the repository root after editing
Markdown:

```sh
npx --yes prettier --write --prose-wrap always --print-width 80 '**/*.md'
```
