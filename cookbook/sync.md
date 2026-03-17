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

### 4. Re-pull Each Installed Item
For each installed entry, fetch the latest from its source:

**If source is a local path** (starts with `/` or `~`):
- Resolve `~` to the home directory
- Get the parent directory of the referenced file
- For skills: copy the entire parent directory to the target:
  ```bash
  cp -R <parent_directory>/ <target_directory>/<name>/
  ```
- For agents: copy just the agent file to the target:
  ```bash
  cp <agent_file> <target_directory>/<agent_name>.md
  ```
- For prompts: copy just the prompt file to the target:
  ```bash
  cp <prompt_file> <target_directory>/<prompt_name>.md
  ```

**If source is a remote URL** (GitHub or Azure DevOps, HTTPS or SSH):
- Parse the URL to extract: `org`, `repo`, `branch`, `file_path` (and `project` for Azure DevOps)
  - See SKILL.md Source Parsing Rules for all supported URL patterns
- Determine the clone URL from the parsed components
- Determine the parent directory path within the repo (everything before the filename)
- Clone into a temporary directory:
  ```bash
  tmp_dir=$(mktemp -d)
  git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
  ```
- Copy the parent directory of the file to the target:
  ```bash
  cp -R "$tmp_dir/<parent_path>/" <target_directory>/<name>/
  ```
- Clean up:
  ```bash
  rm -rf "$tmp_dir"
  ```

**If HTTPS clone fails (private repo)**, try the SSH clone URL for the same provider:
  - GitHub: `git@github.com:<org>/<repo>.git`
  - Azure DevOps: `git@ssh.dev.azure.com:v3/<org>/<project>/<repo>`

### 5. Resolve Dependencies
For each installed entry that has a `requires` field:
- Check if each dependency is also installed
- If a dependency is not installed, pull it as well
- Process dependencies before the items that require them

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
