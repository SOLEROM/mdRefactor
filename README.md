# markdown-kb-refactor

A Claude Code skill that analyzes and refactors filesystem-based Markdown knowledge bases.

It scans a folder, detects structural issues, proposes a plan, and — after your approval — applies changes: renames, moves, README creation, link fixes, and a `.logicTree.md` for future incremental audits.

---

## What it does

| Mode | Trigger | Behavior |
|------|---------|----------|
| **Analyze** | No `.logicTree.md` at root | Full scan → propose plan → wait for approval |
| **Refactor** | User approves the plan | Apply approved changes |
| **Incremental** | `.logicTree.md` exists at root | mtime diff → scan only what changed → suggest small updates |

Things it checks and fixes:
- Naming violations (`camelCase`, typos → `snake_case`)
- Missing `README.md` in any folder
- Placeholder files (≤ 3 lines or `TBD`)
- Folders too large (> 10 items) or too deep (> 3 levels)
- Duplicate / overlapping folders
- Broken links in root `README.md`
- Non-KB folders skipped (`< 50%` markdown files)
- Folders listed in `.kmIgnoreFolderList` skipped entirely

Nothing is changed without your explicit approval.

### Incremental efficiency

After the first run, `.logicTree.md` is written at the root of your KB. Every subsequent run uses its modification time as a diff baseline:

```bash
find . -newer .logicTree.md   # only files changed since last run
```

If nothing changed → "KB is up to date", done in seconds. If new files exist → only those are classified and checked. The rest of the KB is trusted from the `## Manifest` stored inside `.logicTree.md`. At the end of each run the manifest is updated and `.logicTree.md`'s mtime advances, becoming the new baseline.

---

## Installation

Skills live under `.claude/skills/<skill-name>/SKILL.md`.

### Personal install (available in all your projects)

```bash
mkdir -p ~/.claude/skills/markdown-kb-refactor
cp SKILL.md ~/.claude/skills/markdown-kb-refactor/SKILL.md
```

### Project install (shared with your team via git)

```bash
mkdir -p .claude/skills/markdown-kb-refactor
cp SKILL.md .claude/skills/markdown-kb-refactor/SKILL.md
git add .claude/skills/markdown-kb-refactor/SKILL.md
git commit -m "add markdown-kb-refactor skill"
```

---

## Usage

Once installed, invoke it with a slash command in any Claude Code session:

```
/markdown-kb-refactor
```

Then tell it which folder to work on:

```
run markdown-kb-refactor on ./docs
```

Because `disable-model-invocation: true` is set, Claude will never auto-trigger this skill — you always invoke it explicitly.

---

## Ignoring folders

Place a `.kmIgnoreFolderList` file at the root of the KB you are refactoring. One folder name per line. Lines starting with `#` are treated as comments.

```
# .kmIgnoreFolderList
vendor
node_modules
archive
```

Any folder matching a name in this list is skipped entirely — no READMEs created, no files moved in or out.

---

## Files in this repo

| Path | Purpose |
|------|---------|
| `SKILL.md` | The skill definition — copy this to install |
| `tests/` | Intentionally messy KB for testing the skill |
| `refTree/` | Real-world reference KB (Claude Code notes) |

---

## Testing

The `tests/` folder is a purpose-built KB with known issues you can run the skill against to verify behavior:

```
/markdown-kb-refactor
→ run on ./tests
```

Issues baked in:

| File / Folder | Issue triggered |
|---|---|
| `ReadMe.md` | Wrong case (naming violation) |
| Broken link in `ReadMe.md` | Pre-existing broken link detection |
| `apiSettings/` | camelCase violation + overlaps with `api/` |
| `api/`, `reference/` | Missing README |
| `docs/overview.md` | Placeholder (TBD, 2 lines) |
| `notes/draft.md` | Single-file folder, ambiguous filename |
| `deep/level2/level3/level4/` | Depth violation (> 3) |
| `assets/` | Non-KB folder (0% markdown) |
| `vendor/` | Listed in `.kmIgnoreFolderList` — must be skipped |
