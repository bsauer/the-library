---
name: library cross-platform improvements
overview: Review confirms all four suggestions in `library-enhancements.md` are valid, but the impact is broader than the note suggests. The plan is a documentation/instructions consistency pass across the skill and related cookbooks to make Windows/PowerShell behavior explicit, reduce fragile compound operations, and fix a few adjacent workflow gaps.
todos:
  - id: skill-shared-guidance
    content: Update SKILL.md with Windows path detection, shell-compatibility guidance, and cloud-synced path warning
    status: completed
  - id: use-cross-platform
    content: Refactor cookbook/use.md fetch steps into discrete cross-platform instructions
    status: completed
  - id: sync-cross-platform
    content: Refactor cookbook/sync.md fetch steps and dependency ordering for cross-platform reliability
    status: completed
  - id: cookbook-consistency
    content: Update push/install/remove/add cookbooks for PowerShell-safe command guidance and wording consistency
    status: completed
  - id: readme-alignment
    content: Align README.md with the new cross-platform guidance and remaining Cursor wording
    status: completed
isProject: false
---

# Improve Library Cross-Platform Guidance

## Evaluation

I agree with all four suggestions in [library-enhancements.md](library-enhancements.md):

- Cross-platform shell guidance is needed. Bash-only commands appear in [cookbook/use.md](cookbook/use.md), [cookbook/sync.md](cookbook/sync.md), [cookbook/push.md](cookbook/push.md), [cookbook/install.md](cookbook/install.md), and [cookbook/remove.md](cookbook/remove.md).
- Breaking remote fetch into discrete steps is a good improvement for reliability, especially on OneDrive/Dropbox/Google Drive paths.
- A cloud-synced path warning belongs in the shared skill docs and in the fetch-heavy cookbooks.
- Windows local path detection should include drive-letter paths, not just `/` and `~`.

The proposal slightly under-scopes the work: the same shell-compatibility issue shows up beyond `use.md`, and there are a few consistency issues worth fixing at the same time.

## Key Evidence

[cookbook/use.md](cookbook/use.md) currently uses bash-only commands in the main remote fetch flow:

```md
$tmp_dir=$(mktemp -d)
cp -R "$tmp_dir/<parent_path>/" <target_directory>/<name>/
rm -rf "$tmp_dir"
```

[SKILL.md](SKILL.md) still defines local paths too narrowly:

```md
**Local paths** start with `/` or `~`:
```

[README.md](README.md) still includes Unix-only install guidance and one naming inconsistency:

```md
mkdir -p ~/.cursor/skills/library
The included `justfile` lets you run library commands from your terminal without an interactive Claude session.
```

## Planned Improvements

### 1. Add shared cross-platform and path guidance in [SKILL.md](SKILL.md)

Update the core rules so every cookbook can rely on them:

- Expand local path detection to include drive-letter paths such as `C:\` and `D:\`.
- Clarify that `~` resolves to the user home directory on all platforms.
- Add a short shell-compatibility note: prefer bash on macOS/Linux and PowerShell on Windows.
- Add a cloud-synced path warning explaining why clone/copy/cleanup should be separate steps.
- Update the top-level description sentence to say sources may live in local filesystem, GitHub, or Azure DevOps repos.

### 2. Make fetch flows explicit and cross-platform in [cookbook/use.md](cookbook/use.md)

Refactor Step 5 into discrete, verifiable steps for both local and remote sources:

- Detect Windows local paths as valid local sources.
- Split remote fetch into clone, create target, copy, verify, cleanup.
- Add PowerShell equivalents for temp dir creation, copy, delete, and directory creation.
- Keep the existing provider coverage for GitHub/Azure DevOps and HTTPS/SSH.
- Add a short note about cloud-synced paths near the copy steps.

### 3. Mirror the same reliability changes in [cookbook/sync.md](cookbook/sync.md)

Bring sync behavior in line with `use`:

- Update local path detection to include Windows drive-letter paths.
- Split remote re-pull into discrete steps with PowerShell equivalents.
- Add the same cloud-synced path warning.
- Move dependency handling so dependencies are refreshed before dependents, instead of after re-pulling each item.

### 4. Finish the cross-platform pass in the remaining cookbooks

Update command examples where the agent is expected to run shell commands:

- [cookbook/push.md](cookbook/push.md): add PowerShell equivalents for temp directory creation, deletion, and copy operations; add an explicit verify step before staging; consider renaming scope text from “skill” to “item” where the flow also applies to agents/prompts.
- [cookbook/install.md](cookbook/install.md): add a PowerShell equivalent for `mkdir -p` and clone instructions.
- [cookbook/remove.md](cookbook/remove.md): add a PowerShell equivalent for deleting local copies.
- [cookbook/add.md](cookbook/add.md): optionally note that Windows drive-letter paths are valid local paths when validating `source`.

### 5. Align the user-facing docs in [README.md](README.md)

Keep the README consistent with the skill instructions:

- Add a Windows local path example in the Source Formats section.
- Add Windows-friendly installation notes or explicitly point readers to the cookbooks for platform-specific command variants.
- Update prerequisites with non-Homebrew install guidance where helpful.
- Fix the remaining “interactive Claude session” wording to match Cursor-focused docs.

## Scope Note

This is still a documentation-only change set. The repo has no scripts or runtime code to make portable; the improvement is making the agent instructions more explicit, safer on Windows, and more consistent across files.