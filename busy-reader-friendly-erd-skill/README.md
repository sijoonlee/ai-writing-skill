# busy-reader-friendly-erd

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) for writing or restructuring an Engineering Requirements Document (ERD) so a busy, interrupted reader can follow it on one pass: diagrams and tables first, a TL;DR on the first screen, rationale behind folds.

## What is in this repo

| Path | Purpose |
| --- | --- |
| `SKILL.md` | The skill. Claude Code reads this when the skill triggers. |
| `erd-template.md` | The company ERD template the skill follows. `SKILL.md` links to it by relative path, so the two files must stay in the same folder. |
| `example/erd.md` | An ERD written before applying the skill. |
| `example/erd-revised.md` | The same ERD after applying the skill. |
| `some-references/` | Writing notes the skill was distilled from. Not loaded by Claude. |

## Install (user level)

User-level skills live in `~/.claude/skills/<skill-name>/` and are available in every project. The folder name becomes the slash command.

### Option A: clone

```sh
git clone https://github.com/sijoonlee/busy-reader-friendly-erd-skill.git \
  ~/.claude/skills/busy-reader-friendly-erd
```

Update later with `git -C ~/.claude/skills/busy-reader-friendly-erd pull`.

### Option B: copy only the files Claude needs

```sh
mkdir -p ~/.claude/skills/busy-reader-friendly-erd
cp SKILL.md erd-template.md ~/.claude/skills/busy-reader-friendly-erd/
```

Either way the result must contain both files side by side:

```
~/.claude/skills/busy-reader-friendly-erd/
├── SKILL.md
└── erd-template.md
```

Do not symlink the folder or the files. Claude Code does not reliably follow symlinks when discovering skills.

## Install (project level)

To share the skill with one repo instead, put the same two files in `<repo>/.claude/skills/busy-reader-friendly-erd/` and commit them.

## Verify

Start a new Claude Code session and type `/`. `busy-reader-friendly-erd` should appear in the skill list. Run `/busy-reader-friendly-erd` or ask Claude to "make this ERD readable" to trigger it.

## Use

Ask Claude to write, draft, rewrite, or restructure an ERD, design doc, or SDD. The skill keeps the template's section list and order and changes only what goes inside each section. See `example/` for a before and after.

## Update the template

Edit `erd-template.md` when the Confluence template changes. The header of that file records the source page and the version it was copied from.
