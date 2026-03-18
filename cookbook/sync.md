# Sync All Installed Items

## Context
Refresh every locally installed skill, agent, and prompt by re-pulling from its source. A fast, lazy "make sure everything is up to date" command.

## Steps

### 1. Sync the Library Repo
Pull the latest catalog before reading:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Read the Catalog
- Read `library.yaml`
- Parse all entries from `library.skills`, `library.agents`, and `library.prompts`

### 3. Find All Installed Items
For each entry in the catalog:
- Determine the type (skill, agent, prompt) and corresponding directories from `default_dirs`
- Check if a directory or file matching the entry name exists in the **default** directory
- Check if a directory or file matching the entry name exists in the **global** directory
- Search recursively for name matches
- Collect every entry that is installed locally (either default or global)
- If nothing is installed, tell the user and exit

### 4. Resolve Dependencies First
Before refreshing an installed entry:
- Check whether it has a `requires` field.
- For each dependency, confirm it exists in the catalog and is included in the sync set.
- If a dependency is not installed locally, pull it first.
- Refresh dependencies before the items that require them.
- When multiple installed items depend on each other, process them in dependency order as much as possible.

### 5. Re-pull Each Installed Item
For each installed entry, fetch the latest from its source in dependency order:

> **Cloud-synced paths:** If the workspace or target directory is inside OneDrive, Dropbox, Google Drive, or a similar synced folder, file operations may be slower. Create directories, copy files, verify the result, and clean up as separate steps.

**If source is a local path** (starts with `/`, `~`, or a drive letter like `C:\`):
- Resolve `~` to the home directory before running shell commands.
- Get the parent directory of the referenced file.
- Create the target path first, then copy files, then verify the expected file exists.
- For skills: copy the entire parent directory contents into the target directory:
  ```bash
  mkdir -p <target_directory>/<name>
  cp -R <parent_directory>/. <target_directory>/<name>/
  ```
  ```powershell
  New-Item -ItemType Directory -Force -Path "<target_directory>\<name>"
  Copy-Item -Recurse -Force "<parent_directory>\*" "<target_directory>\<name>\"
  ```
- For agents: copy just the agent file to the target:
  ```bash
  mkdir -p <target_directory>
  cp <agent_file> <target_directory>/<agent_name>.md
  ```
  ```powershell
  New-Item -ItemType Directory -Force -Path "<target_directory>"
  Copy-Item -Force "<agent_file>" "<target_directory>\<agent_name>.md"
  ```
- For prompts: copy just the prompt file to the target:
  ```bash
  mkdir -p <target_directory>
  cp <prompt_file> <target_directory>/<prompt_name>.md
  ```
  ```powershell
  New-Item -ItemType Directory -Force -Path "<target_directory>"
  Copy-Item -Force "<prompt_file>" "<target_directory>\<prompt_name>.md"
  ```

**If source is a remote URL** (GitHub or Azure DevOps, HTTPS or SSH):
- Parse the URL to extract: `org`, `repo`, `branch`, `file_path` (and `project` for Azure DevOps).
  - See `SKILL.md` Source Parsing Rules for all supported URL patterns.
- Determine the clone URL from the parsed components.
- Determine the parent directory path within the repo (everything before the filename).
- Run the refresh as separate steps:
  1. Create a temporary directory.
     ```bash
     tmp_dir=$(mktemp -d)
     ```
     ```powershell
     $tmp_dir = Join-Path $env:TEMP ("lib_sync_" + [guid]::NewGuid().ToString().Substring(0,8))
     New-Item -ItemType Directory -Force -Path $tmp_dir
     ```
  2. Clone the repo into the temporary directory.
     ```bash
     git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
     ```
     ```powershell
     git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
     ```
  3. Create the target directory.
     ```bash
     mkdir -p <target_directory>/<name>
     ```
     ```powershell
     New-Item -ItemType Directory -Force -Path "<target_directory>\<name>"
     ```
  4. Copy the fetched directory contents into the target.
     ```bash
     cp -R "$tmp_dir/<parent_path>/." <target_directory>/<name>/
     ```
     ```powershell
     Copy-Item -Recurse -Force "$tmp_dir\<parent_path>\*" "<target_directory>\<name>\"
     ```
  5. Verify the expected main file exists in the target directory before cleanup.
  6. Clean up the temporary directory.
     ```bash
     rm -rf "$tmp_dir"
     ```
     ```powershell
     Remove-Item -Recurse -Force "$tmp_dir"
     ```

**If HTTPS clone fails (private repo)**, try the SSH clone URL for the same provider:
  - GitHub: `git@github.com:<org>/<repo>.git`
  - Azure DevOps: `git@ssh.dev.azure.com:v3/<org>/<project>/<repo>`

### 6. Report Results
Display a summary table:

```
## Sync Complete

| Type | Name | Status |
|------|------|--------|
| skill | skill-name | refreshed |
| agent | agent-name | refreshed |
| skill | other-skill | failed: <reason> |

Synced: X items
Failed: Y items
```

If any items failed (e.g., network error, missing source), list them with the reason so the user can fix individually.
