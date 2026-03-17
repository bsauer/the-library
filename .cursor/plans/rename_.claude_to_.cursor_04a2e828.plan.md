---
name: Rename .claude to .cursor
overview: Replace all `.claude` directory path references with `.cursor` across 6 files in the repository. Approximately 40 occurrences total.
todos:
  - id: library-yaml
    content: Replace 6 occurrences of .claude with .cursor in library.yaml
    status: pending
  - id: skill-md
    content: Replace 14 occurrences of .claude with .cursor in SKILL.md
    status: pending
  - id: readme-md
    content: Replace 19 occurrences of .claude with .cursor in README.md
    status: pending
  - id: cookbook-install
    content: Replace 3 occurrences of .claude with .cursor in cookbook/install.md
    status: pending
  - id: cookbook-add
    content: Replace 1 occurrence of .claude with .cursor in cookbook/add.md
    status: pending
  - id: svg-sprawl
    content: Replace 2 occurrences of .claude with .cursor in images/26_problem_skill_sprawl.svg
    status: pending
isProject: false
---

# Rename All `.claude` References to `.cursor`

6 files contain `.claude` directory path references that need to change to `.cursor`. No other files in the repo are affected. The `justfile` references `claude` (the CLI command), not `.claude` (the directory), so it is excluded.

## File-by-File Changes

### 1. [library.yaml](library.yaml) -- 6 occurrences

All 6 directory paths on lines 3-10:

- `.claude/skills/` -> `.cursor/skills/`
- `~/.claude/skills/` -> `~/.cursor/skills/`
- `.claude/agents/` -> `.cursor/agents/`
- `~/.claude/agents/` -> `~/.cursor/agents/`
- `.claude/commands/` -> `.cursor/commands/`
- `~/.claude/commands/` -> `~/.cursor/commands/`

### 2. [SKILL.md](SKILL.md) -- 14 occurrences

- Line 16: `~/.claude/skills/library/library.yaml` -> `~/.cursor/skills/library/library.yaml`
- Line 17: `~/.claude/skills/library/` -> `~/.cursor/skills/library/`
- Lines 116-123: first `default_dirs` YAML block (6 paths)
- Lines 144-151: second `default_dirs` YAML block (6 paths, note: this block uses `.claude/prompts/` instead of `.claude/commands/`)

All follow the same `.claude` -> `.cursor` replacement.

### 3. [README.md](README.md) -- 19 occurrences

- Line 43: `~/.claude/*` -> `~/.cursor/*`
- Lines 56-63: `default_dirs` YAML block (6 paths)
- Line 104: `.claude/skills/` -> `.cursor/skills/`
- Lines 127, 131, 132, 135: `~/.claude/skills/library` paths in installation instructions
- Line 140: `~/.claude/skills/library/SKILL.md`
- Line 150: `~/.claude/skills/library/`
- Line 180: `.claude/skills/deploy/` -> `.cursor/skills/deploy/`
- Line 237: `~/.claude/skills/library/` in architecture tree
- Line 258: `.claude/skills/` -> `.cursor/skills/`

### 4. [cookbook/install.md](cookbook/install.md) -- 3 occurrences

- Line 10: `~/.claude/skills/`
- Line 42: `~/.claude/skills/library/library.yaml`
- Line 43: `~/.claude/skills/library/`

### 5. [cookbook/add.md](cookbook/add.md) -- 1 occurrence

- Line 55: `.claude/commands/` -> `.cursor/commands/`

### 6. [images/26_problem_skill_sprawl.svg](images/26_problem_skill_sprawl.svg) -- 2 occurrences

- Line 169: comment `<!-- .claude/skills/ -->` -> `<!-- .cursor/skills/ -->`
- Line 173: SVG text element `.claude/skills/` -> `.cursor/skills/`

## Approach

Use `replace_all` with `StrReplace` on each file, replacing `.claude` with `.cursor`. This is safe because every `.claude` occurrence in these files is a directory path reference.