---
name: library
description: Private skill distribution system. Use when the user wants to install, use, add, push, remove, sync, list, or search for skills, agents, or prompts from their private library catalog. Triggers on /library commands or mentions of library, skill distribution, or agentic management.
argument-hint: [command or prompt] [name or details]
---

# The Library

A meta-skill for private-first distribution of agentics (skills, agents, and prompts) across agents, devices, and teams.

## Variables

> Update these after forking and cloning the library repo.

- **LIBRARY_REPO_URL**: `<your forked repo url>`
- **LIBRARY_YAML_PATH**: `~/.cursor/skills/library/library.yaml`
- **LIBRARY_SKILL_DIR**: `~/.cursor/skills/library/`

## How It Works

The Library is a catalog of references to your agentics. The `library.yaml` file points to where skills, agents, and prompts live (local filesystem or GitHub repos). Nothing is fetched until you ask for it.

**The `library.yaml` is a catalog, not a manifest.** Entries define what's *available* — not what gets installed. You pull specific items on demand with `/library use <name>`.

## Commands

| Command                     | Purpose                                  |
| --------------------------- | ---------------------------------------- |
| `/library install`          | First-time setup: fork, clone, configure |
| `/library add <details>`    | Register a new entry in the catalog      |
| `/library use <name>`       | Pull from source (install or refresh)    |
| `/library push <name>`      | Push local changes back to source        |
| `/library remove <name>`    | Remove from catalog and optionally local |
| `/library list`             | Show full catalog with install status    |
| `/library sync`             | Re-pull all installed items from source   |
| `/library search <keyword>` | Find entries by keyword                  |

## Cookbook

Each command has a detailed step-by-step guide. **Read the relevant cookbook file before executing a command.**

| Command | Cookbook                                 | Use When                                                    |
| ------- | --------------------------------------- | ----------------------------------------------------------- |
| install | [cookbook/install.md](cookbook/install.md) | First-time setup on a new device                            |
| add     | [cookbook/add.md](cookbook/add.md)         | User wants to register a new skill/agent/prompt in catalog  |
| use     | [cookbook/use.md](cookbook/use.md)         | User wants to pull or refresh a skill from the catalog      |
| push    | [cookbook/push.md](cookbook/push.md)       | User improved a skill locally and wants to update the source |
| remove  | [cookbook/remove.md](cookbook/remove.md)   | User wants to remove an entry from the catalog               |
| list    | [cookbook/list.md](cookbook/list.md)       | User wants to see what's available and what's installed      |
| sync    | [cookbook/sync.md](cookbook/sync.md)       | User wants to refresh all installed items at once            |
| search  | [cookbook/search.md](cookbook/search.md)   | User is looking for a skill but doesn't know the exact name |

**When a user invokes a `/library` command, read the matching cookbook file first, then execute the steps.**

## Source Format

The `source` field in `library.yaml` supports these formats (auto-detected):

- `/absolute/path/to/SKILL.md` — local filesystem
- `https://github.com/org/repo/blob/main/path/to/SKILL.md` — GitHub browser URL
- `https://raw.githubusercontent.com/org/repo/main/path/to/SKILL.md` — GitHub raw URL
- `https://dev.azure.com/org/project/_git/repo?path=/path/to/SKILL.md&version=GBmain` — Azure DevOps browser URL
- `https://dev.azure.com/org/project/_apis/git/repositories/repo/items?path=/path/to/SKILL.md&versionDescriptor.version=main&versionDescriptor.versionType=branch` — Azure DevOps raw/API URL
- `git@github.com:org/repo.git//path/to/SKILL.md#main` — GitHub SSH URL
- `git@ssh.dev.azure.com:v3/org/project/repo//path/to/SKILL.md#main` — Azure DevOps SSH URL

All URL formats are auto-detected. Parse org, repo, branch, and file path from the URL structure. For private repos, use SSH or `GITHUB_TOKEN` for GitHub auth, and PAT, Azure AD credentials, or SSH keys for Azure DevOps auth.

**Important:** The source points to a specific file (SKILL.md, AGENT.md, or prompt file). We always pull the entire parent directory, not just the file.

## Source Parsing Rules

**Local paths** start with `/` or `~`:
- Use the path directly. Copy the parent directory of the referenced file.

**GitHub browser URLs** match `https://github.com/<org>/<repo>/blob/<branch>/<path>`:
- Parse: `org`, `repo`, `branch`, `file_path`
- Clone URL: `https://github.com/<org>/<repo>.git`
- File location within repo: `<path>`

**GitHub raw URLs** match `https://raw.githubusercontent.com/<org>/<repo>/<branch>/<path>`:
- Parse: `org`, `repo`, `branch`, `file_path`
- Clone URL: `https://github.com/<org>/<repo>.git`
- File location within repo: `<path>`

**Azure DevOps browser URLs** match `https://dev.azure.com/<org>/<project>/_git/<repo>?path=/<path>&version=GB<branch>`:
- Parse: `org`, `project`, `repo`, `branch` (strip `GB` prefix from `version` param), `file_path` (from `path` param)
- Clone URL: `https://dev.azure.com/<org>/<project>/_git/<repo>`
- File location within repo: `<path>` (strip leading `/` from the `path` query param)

**Azure DevOps raw/API URLs** match `https://dev.azure.com/<org>/<project>/_apis/git/repositories/<repo>/items?path=/<path>&versionDescriptor.version=<branch>&versionDescriptor.versionType=branch`:
- Parse: `org`, `project`, `repo`, `branch` (from `versionDescriptor.version` param), `file_path` (from `path` param)
- Clone URL: `https://dev.azure.com/<org>/<project>/_git/<repo>`
- File location within repo: `<path>` (strip leading `/` from the `path` query param)

**GitHub SSH URLs** match `git@github.com:<org>/<repo>.git//<path>#<branch>`:
- Parse: `org`, `repo`, `file_path` (after `//`), `branch` (after `#`, defaults to `main` if absent)
- Clone URL: `git@github.com:<org>/<repo>.git`
- File location within repo: `<path>`

**Azure DevOps SSH URLs** match `git@ssh.dev.azure.com:v3/<org>/<project>/<repo>//<path>#<branch>`:
- Parse: `org`, `project`, `repo`, `file_path` (after `//`), `branch` (after `#`, defaults to `main` if absent)
- Clone URL: `git@ssh.dev.azure.com:v3/<org>/<project>/<repo>`
- File location within repo: `<path>`

## GitHub Workflow

When working with GitHub sources, prefer `gh api` for accessing single files (e.g., reading a SKILL.md to check metadata). For pulling entire skill directories, clone into a temp dir per the steps below.

**Fetching (use):**
1. Clone the repo with `git clone --depth 1 <clone_url>` into a temporary directory
2. Navigate to the parent directory of the referenced file
3. Copy that entire directory to the target local directory
4. The temporary directory is cleaned up automatically

**Pushing (push):**
1. Clone the repo with `git clone --depth 1 <clone_url>` into a temporary directory
2. Overwrite the skill directory in the clone with the local version
3. Stage only the relevant changes: `git add <skill_directory_path>`
4. Commit with message: `library: updated <skill name> <what changed>`
5. Push to remote
6. The temporary directory is cleaned up automatically

## Azure DevOps Workflow

When working with Azure DevOps sources, use `az repos` CLI or the REST API for accessing single files. For pulling entire skill directories, clone into a temp dir using the same fetch/push pattern as GitHub.

**Authentication:** Azure DevOps clone URLs require a PAT (Personal Access Token) or Azure AD credentials. The clone URL embeds the PAT like `https://<PAT>@dev.azure.com/<org>/<project>/_git/<repo>`, or you can use Git Credential Manager which handles auth automatically.

**Fetching (use):**
1. Clone the repo with `git clone --depth 1 <clone_url>` into a temporary directory
2. Navigate to the parent directory of the referenced file
3. Copy that entire directory to the target local directory
4. The temporary directory is cleaned up automatically

**Pushing (push):**
1. Clone the repo with `git clone --depth 1 <clone_url>` into a temporary directory
2. Overwrite the skill directory in the clone with the local version
3. Stage only the relevant changes: `git add <skill_directory_path>`
4. Commit with message: `library: updated <skill name> <what changed>`
5. Push to remote
6. The temporary directory is cleaned up automatically

## Typed Dependencies

The `requires` field uses typed references to avoid ambiguity:
- `skill:name` — references a skill in the library catalog
- `agent:name` — references an agent in the library catalog
- `prompt:name` — references a prompt in the library catalog

When resolving dependencies: look up each reference in `library.yaml`, fetch all dependencies first (recursively), then fetch the requested item.

## Target Directories

By default, items are installed to the **default** directory from `library.yaml`:

```yaml
default_dirs:
    skills:
        - default: .cursor/skills/
        - global: ~/.cursor/skills/
    agents:
        - default: .cursor/agents/
        - global: ~/.cursor/agents/
    prompts:
        - default: .cursor/commands/
        - global: ~/.cursor/commands/
```

- If the user says "global" or "globally", use the `global` directory.
- If the user specifies a custom path, use that path.
- Otherwise, use the `default` directory.

## Library Repo Sync

The library skill itself lives in `<LIBRARY_SKILL_DIR>` as a cloned git repo. When running `add` (which modifies `library.yaml`), always:
1. `git pull` in the library directory first to get latest
2. Make the changes
3. `git add library.yaml && git commit && git push`

This keeps the catalog in sync across devices.

## Example Filled Library File

```yaml
default_dirs:
  skills:
    - default: .cursor/skills/
    - global: ~/.cursor/skills/
  agents:
    - default: .cursor/agents/
    - global: ~/.cursor/agents/
  prompts:
    - default: .cursor/prompts/
    - global: ~/.cursor/prompts/

library:
  skills:
    - name: firecrawl
      description: Scrape, crawl, and search websites using Firecrawl CLI
      source: /Users/me/projects/tools/skills/firecrawl/SKILL.md

    - name: meta-skill
      description: Creates new Agent Skills following best practices
      source: /Users/me/projects/tools/skills/meta-skill/SKILL.md

    - name: diagram-kroki
      description: Generate diagrams via Kroki HTTP API supporting 28+ languages
      source: https://github.com/myorg/private-skills/blob/main/skills/diagram-kroki/SKILL.md
      requires: [skill:firecrawl]

    - name: green-screen-captions
      description: Generate and burn AI-powered captions onto green screen videos
      source: https://raw.githubusercontent.com/myorg/video-tools/main/skills/green-screen-captions/SKILL.md
      requires: [agent:video-processor, prompt:caption-style]

    - name: deploy-infra
      description: Deploy infrastructure using Terraform and Azure pipelines
      source: https://dev.azure.com/myorg/platform/_git/infra-skills?path=/skills/deploy-infra/SKILL.md&version=GBmain

    - name: secret-scanner
      description: Scan repos for leaked secrets and credentials
      source: git@github.com:myorg/security-tools.git//skills/secret-scanner/SKILL.md#main

    - name: pipeline-lint
      description: Validate and lint Azure DevOps pipeline YAML files
      source: git@ssh.dev.azure.com:v3/myorg/platform/pipeline-tools//skills/pipeline-lint/SKILL.md#main

  agents:
    - name: video-processor
      description: Processes video files with ffmpeg and whisper transcription
      source: /Users/me/projects/tools/agents/video-processor/AGENT.md

    - name: code-reviewer
      description: Reviews code for quality, security, and performance
      source: https://github.com/myorg/agent-configs/blob/main/agents/code-reviewer/AGENT.md

  prompts:
    - name: caption-style
      description: Style guide for generating video captions
      source: /Users/me/projects/content/prompts/caption-style.md

    - name: commit-message
      description: Standardized commit message format for all projects
      source: https://github.com/myorg/team-prompts/blob/main/prompts/commit-message.md
```
