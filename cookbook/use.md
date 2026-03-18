# Use a Skill from the Library

## Context
Pull a skill, agent, or prompt from the catalog into the local environment. If already installed locally, overwrite with the latest from the source (refresh).

## Input
The user provides a skill name or description.

## Steps

### 1. Sync the Library Repo
Pull the latest catalog before reading:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Find the Entry
- Read `library.yaml`
- Search across `library.skills`, `library.agents`, and `library.prompts`
- Match by name (exact) or description (fuzzy/keyword match)
- If multiple matches, show them and ask the user to pick one
- If no match, tell the user and suggest `/library search`

### 3. Resolve Dependencies
If the entry has a `requires` field:
- For each typed reference (`skill:name`, `agent:name`, `prompt:name`):
  - Look it up in `library.yaml`
  - If found, recursively run the `use` workflow for that dependency first
  - If not found, warn the user: "Dependency <ref> not found in library catalog"
- Process all dependencies before the requested item

### 4. Determine Target Directory
- Read `default_dirs` from `library.yaml`
- If user said "global" or "globally" → use the `global` path
- If user specified a custom path → use that path
- Otherwise → use the `default` path
- Select the correct section based on type (skills/agents/prompts)

### 5. Fetch from Source

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
- If the agent or prompt is nested in a subdirectory under the `agents/` or `commands/` directories, create the matching subdirectory in the target first, then copy the file into it. This keeps related agents or commands grouped together.

**If source is a remote URL** (GitHub or Azure DevOps, HTTPS or SSH):
- Parse the URL to extract: `org`, `repo`, `branch`, `file_path` (and `project` for Azure DevOps).
  - GitHub browser: `https://github.com/<org>/<repo>/blob/<branch>/<path>`
  - GitHub raw: `https://raw.githubusercontent.com/<org>/<repo>/<branch>/<path>`
  - GitHub SSH: `git@github.com:<org>/<repo>.git//<path>#<branch>`
  - Azure DevOps browser: `https://dev.azure.com/<org>/<project>/_git/<repo>?path=/<path>&version=GB<branch>`
  - Azure DevOps API: `https://dev.azure.com/<org>/<project>/_apis/git/repositories/<repo>/items?path=/<path>&versionDescriptor.version=<branch>&...`
  - Azure DevOps SSH: `git@ssh.dev.azure.com:v3/<org>/<project>/<repo>//<path>#<branch>`
- Determine the clone URL from the parsed components (see `SKILL.md` Source Parsing Rules).
- Determine the parent directory path within the repo (everything before the filename).
- Run the fetch as separate steps:
  1. Create a temporary directory.
     ```bash
     tmp_dir=$(mktemp -d)
     ```
     ```powershell
     $tmp_dir = Join-Path $env:TEMP ("lib_fetch_" + [guid]::NewGuid().ToString().Substring(0,8))
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

### 6. Verify Installation
- Confirm the target directory exists
- Confirm the main file (SKILL.md, AGENT.md, or prompt file) exists in it
- Report success with the installed path

### 7. Confirm
Tell the user:
- What was installed and where
- Any dependencies that were also installed
- If this was a refresh (overwrite), mention that
