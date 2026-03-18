# Library Skill Enhancements

Findings from real-world usage of `/library use generate-svg` on Windows/PowerShell with an OneDrive-synced workspace.

## 1. Cross-Platform Shell Commands

**File:** `cookbook/use.md` (lines 69-81)
**Issue:** Step 5 only provides bash commands (`mktemp -d`, `cp -R`, `rm -rf`). These don't exist on Windows/PowerShell. The agent had to improvise, and the first attempt hung for 60+ seconds.

**Recommendation:** Add a cross-platform note or provide both bash and PowerShell equivalents.

```markdown
**Bash (macOS/Linux):**
tmp_dir=$(mktemp -d)
git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
cp -R "$tmp_dir/<parent_path>/" <target_directory>/<name>/
rm -rf "$tmp_dir"

**PowerShell (Windows):**
$tmp_dir = Join-Path $env:TEMP ("lib_fetch_" + [guid]::NewGuid().ToString().Substring(0,8))
git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
Copy-Item -Recurse -Force "$tmp_dir\<parent_path>\*" "<target_directory>\<name>\"
Remove-Item -Recurse -Force "$tmp_dir"
```

## 2. Break Fetch Into Discrete Steps

**File:** `cookbook/use.md` (Step 5)
**Issue:** Compound "clone + copy + cleanup" in a single command caused a hang on an OneDrive path. Breaking into separate steps fixed it.

**Recommendation:** Replace the single compound operation with discrete, verifiable steps:

1. **Clone** the repo into a temp directory
2. **Create** the target directory
3. **Copy** files from temp to target
4. **Verify** the main file exists in the target
5. **Clean up** the temp directory

Each step should be a separate command so the agent can detect failures early.

## 3. Cloud-Synced Path Warning

**File:** `SKILL.md` or `cookbook/use.md`
**Issue:** Workspace was on an OneDrive-synced path (`OneDrive - CSG Systems Inc\Documents\...`). File operations on cloud-synced paths are significantly slower and can hang when done as compound operations.

**Recommendation:** Add a note like:

```markdown
> **Cloud-synced paths:** If the workspace is on OneDrive, Dropbox, or Google Drive,
> file operations may be slower. Break copy operations into separate steps
> (create directory, then copy contents) rather than compound commands.
```

## 4. Windows Local Path Detection

**File:** `SKILL.md` (line 73)
**Issue:** The local path detection rule says paths start with `/` or `~`. This misses Windows drive-letter paths like `C:\Users\...`.

**Current:**
```markdown
**Local paths** start with `/` or `~`:
```

**Recommended:**
```markdown
**Local paths** start with `/`, `~`, or a drive letter (e.g., `C:\`, `D:\`):
```

## Summary

| # | Area | File | Severity |
|---|------|------|----------|
| 1 | Shell commands are Unix-only | `cookbook/use.md` | High - blocks Windows users |
| 2 | Compound operations hang | `cookbook/use.md` | High - caused 60s+ hang |
| 3 | Cloud-synced path slowness | `SKILL.md` / `cookbook/use.md` | Medium - affects OneDrive/Dropbox users |
| 4 | Local path detection | `SKILL.md` | Low - edge case for Windows local sources |
