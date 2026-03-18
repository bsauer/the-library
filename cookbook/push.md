# Push an Installed Item to the Source

## Context
The user has improved a locally installed skill, agent, or prompt and wants to push changes back to the source.

## Input
The user provides an item name or description.

## Steps

### 1. Find the Entry
- Read `library.yaml`
- Search across all sections for the matching entry
- If no match, tell the user the item wasn't found in the catalog

### 2. Locate the Local Copy
- Check the default directory for the type (from `default_dirs`)
- Check the global directory
- If found in multiple places, ask which one to push
- If not found locally, tell the user there's nothing to push

### 3. Check for Conflicts

**If source is a local path:**
- Compare the local installed copy with the source
- If the source has been modified since last pull, warn the user:
  "The source has changes that aren't in your local copy. Pushing will overwrite them. Continue?"

**If source is a remote URL** (GitHub or Azure DevOps, HTTPS or SSH):
- Parse the URL to determine the clone URL (see SKILL.md Source Parsing Rules)
- Clone the repo to a temp directory (shallow):
  ```bash
  tmp_dir=$(mktemp -d)
  git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
  ```
  ```powershell
  $tmp_dir = Join-Path $env:TEMP ("lib_push_" + [guid]::NewGuid().ToString().Substring(0,8))
  New-Item -ItemType Directory -Force -Path $tmp_dir
  git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
  ```
- Compare the cloned item path with the local copy
- If they differ AND the remote has changes not in the local copy, warn about conflict
- Ask the user to resolve before continuing

### 4. Push to Source

**If source is a local path:**
- For skills: copy the entire local directory to the source location, overwriting:
  ```bash
  cp -R <local_item_path>/. <source_parent_directory>/
  ```
  ```powershell
  Copy-Item -Recurse -Force "<local_item_path>\*" "<source_parent_directory>\"
  ```
- For agents or prompts: copy the local file to the source location, overwriting:
  ```bash
  cp <local_item_path> <source_file_path>
  ```
  ```powershell
  Copy-Item -Force "<local_item_path>" "<source_file_path>"
  ```
- Confirm the overwrite and verify the updated file or directory exists at the source location.

**If source is a remote URL** (GitHub or Azure DevOps, HTTPS or SSH):
- Parse the URL to determine the clone URL (see SKILL.md Source Parsing Rules)
- If we don't already have a tmp clone from step 3, clone now:
  ```bash
  tmp_dir=$(mktemp -d)
  git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
  ```
  ```powershell
  $tmp_dir = Join-Path $env:TEMP ("lib_push_" + [guid]::NewGuid().ToString().Substring(0,8))
  New-Item -ItemType Directory -Force -Path $tmp_dir
  git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
  ```
- For skills:
  - Remove the old directory in the clone.
    ```bash
    rm -rf "$tmp_dir/<item_path_in_repo>"
    ```
    ```powershell
    Remove-Item -Recurse -Force "$tmp_dir\<item_path_in_repo>"
    ```
  - Recreate the directory and copy the local contents into it.
    ```bash
    mkdir -p "$tmp_dir/<item_path_in_repo>"
    cp -R <local_item_path>/. "$tmp_dir/<item_path_in_repo>/"
    ```
    ```powershell
    New-Item -ItemType Directory -Force -Path "$tmp_dir\<item_path_in_repo>"
    Copy-Item -Recurse -Force "<local_item_path>\*" "$tmp_dir\<item_path_in_repo>\"
    ```
- For agents or prompts:
  - Remove the old file in the clone.
    ```bash
    rm -f "$tmp_dir/<item_path_in_repo>"
    ```
    ```powershell
    Remove-Item -Force "$tmp_dir\<item_path_in_repo>"
    ```
  - Create the parent directory if needed, then copy the local file into place.
    ```bash
    mkdir -p "$tmp_dir/<item_parent_path>"
    cp <local_item_path> "$tmp_dir/<item_path_in_repo>"
    ```
    ```powershell
    New-Item -ItemType Directory -Force -Path "$tmp_dir\<item_parent_path>"
    Copy-Item -Force "<local_item_path>" "$tmp_dir\<item_path_in_repo>"
    ```
- Verify the expected file exists in the cloned repo before staging.
- Stage ONLY the relevant changes:
  ```bash
  cd "$tmp_dir"
  git add <item_path_in_repo>
  ```
- Commit with the standard format:
  ```bash
  git commit -m "library: updated <name> <brief description of what changed>"
  ```
- Push:
  ```bash
  git push
  ```
- Clean up:
  ```bash
  rm -rf "$tmp_dir"
  ```
  ```powershell
  Remove-Item -Recurse -Force "$tmp_dir"
  ```

### 5. Confirm
Tell the user:
- What was pushed and where
- The commit message used
- If it was a local path push, confirm the overwrite
