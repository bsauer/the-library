# The Library

A meta-skill for private-first distribution of agentics (skills, agents, and prompts) across agents, devices, and teams.

![The Library](images/10_meta_skill.svg)

## Who This Is For

If you're an engineer working on 10+ codebases with agents and you're building specialized private skills, agents, and prompts — this was made for you.

If you work in one or two repos, you don't need this. If you install skills from the public internet without reviewing them, this isn't for you either.

The Library solves a specific problem: you've built powerful agentics scattered across repos, devices, and teams. They're duplicated, out of sync, and hard to coordinate. This gives you a single reference catalog to distribute them privately.

## What It Is

The Library is a single skill whose only job is to manage other skills. It's a catalog of references — local file paths, GitHub repo URLs, and Azure DevOps repo URLs — that point to where your agentics live. Nothing is copied or installed until you ask for it.

Think of it as a `package.json` for agent capabilities — but instead of packages, you're managing skills, agents, and prompts. Instead of a registry, you're pointing at your own private repos and local paths.

**This is a pure agent application.** There are no scripts, no CLIs, no dependencies, no build tools. The entire application is encoded in `SKILL.md` and a set of cookbook instructions that teach the agent exactly what to do. The agent IS the runtime. This matters because:

- Any agent harness that reads skill files can run it (Cursor, Claude Code, Pi, etc.)
- You can modify behavior by editing markdown, not code
- The skill can be extended, forked, and adapted instantly
- An orchestrator agent can chain library commands without any tooling overhead

## Why It Exists

![The Problem: Skill Sprawl](images/26_problem_skill_sprawl.svg)

As you build with AI agents, you accumulate skills, custom agents, and prompts — potentially hundreds of them. You need to:

- **Reuse** them across projects without copy-pasting
- **Distribute** them to your agents running on other devices (Mac mini, remote servers, cloud sandboxes)
- **Share** them with your team without making everything public
- **Keep them private** — these are specialized capabilities built for competitive edge
- **Stay in sync** — one source of truth, not 10 stale copies

![The Problem: Siloed Teams](images/32_problem_team_sharing.svg)

Existing solutions don't fit:
- **Global `~/.cursor/*`** — exposes everything to every agent. Global is the opposite of specialized.
- **Agent plugins** — requires marketplace infrastructure, manifests, and locks you into one platform.
- **Single monorepo** — doesn't reflect reality. You build agentics in specific codebases for specific use cases.

## How It Works

![The Solution: The Library](images/27_solution_library_workflow.svg)

### The Catalog (`library.yaml`)

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

library:
  skills:
    - name: my-skill
      description: What this skill does
      source: /Users/me/projects/tools/skills/my-skill/SKILL.md
      requires: [agent:helper-agent]
    - name: remote-skill
      description: A skill from a private repo
      source: https://github.com/myorg/private-skills/blob/main/skills/remote-skill/SKILL.md
  agents: []
  prompts: []
```

The catalog stores pointers, not copies. Skills live in their source repos. You pull on demand.

### Source Formats

| Format                | Example                                                                          |
| --------------------- | -------------------------------------------------------------------------------- |
| Local filesystem      | `/absolute/path/to/SKILL.md` or `C:\path\to\SKILL.md`                            |
| GitHub browser URL    | `https://github.com/org/repo/blob/main/path/to/SKILL.md`                         |
| GitHub raw URL        | `https://raw.githubusercontent.com/org/repo/main/path/to/SKILL.md`               |
| Azure DevOps browser  | `https://dev.azure.com/org/project/_git/repo?path=/path/to/SKILL.md&version=GBmain` |
| Azure DevOps raw/API  | `https://dev.azure.com/org/project/_apis/git/repositories/repo/items?path=/path/to/SKILL.md&...` |
| GitHub SSH            | `git@github.com:org/repo.git//path/to/SKILL.md#main`                             |
| Azure DevOps SSH      | `git@ssh.dev.azure.com:v3/org/project/repo//path/to/SKILL.md#main`               |

The source points to a specific file. The system pulls the entire parent directory (skills include scripts, references, assets — not just the markdown file).

For private repos, authentication uses SSH keys, `GITHUB_TOKEN` (GitHub), or PAT / Azure AD credentials (Azure DevOps) automatically.

On Windows, drive-letter paths like `C:\Users\me\projects\my-skill\SKILL.md` are valid local sources. When command examples differ by platform, use bash on macOS/Linux and PowerShell on Windows.

### Typed Dependencies

Dependencies use typed references to avoid name collisions:

```yaml
requires: [skill:base-utils, agent:reviewer, prompt:task-router]
```

Dependencies are resolved and pulled first, recursively.

## Prerequisites

- **Cursor** (or a compatible agent harness that reads `.cursor/skills/` — e.g., Claude Code, Pi)
- **git** — for cloning sources and syncing the catalog
- **gh** (optional) — GitHub CLI for forking, cloning, and private repo access. Install with your platform package manager (for example, `brew install gh`) or see [gh docs](https://cli.github.com)
- **az repos** (optional) — Azure CLI with DevOps extension for Azure DevOps repo access. Install: `az extension add --name azure-devops`
- **GitHub SSH key or `GITHUB_TOKEN`** — for accessing private GitHub repos (not needed if using `gh auth login`)
- **Azure DevOps PAT, SSH key, or Azure AD credentials** — for accessing private Azure DevOps repos (not needed if using Git Credential Manager)
- **just** (optional) — for justfile shortcuts. Install with your platform package manager (for example, `brew install just`) or see [just docs](https://github.com/casey/just)

## Installation

This is a template repo. You fork it, clone it into your global skills directory, and it becomes a `/library` slash command available in every Cursor session.

### 1. Fork This Repo

Fork to your own GitHub account (private repo recommended). This fork is your personal library catalog — you'll push catalog updates to it.

```bash
# Using GitHub CLI
gh repo fork disler/the-library --private --clone=false
```

Or fork manually via the GitHub UI.

### 2. Clone to Global Skills Directory

Clone your fork into `~/.cursor/skills/library`. This path is what makes `/library` available as a global slash command in Cursor.

```bash
# Using git
mkdir -p ~/.cursor/skills/library
git clone <your-fork-url> ~/.cursor/skills/library

# Or using GitHub CLI
gh repo clone <yourname>/the-library ~/.cursor/skills/library
```

```powershell
# Using git in PowerShell
New-Item -ItemType Directory -Force -Path "$HOME/.cursor/skills/library"
git clone <your-fork-url> "$HOME/.cursor/skills/library"
```

If you are on Windows, the cookbook files include PowerShell-friendly command variants for install, use, sync, push, and remove flows.

### 3. Configure

Open `~/.cursor/skills/library/SKILL.md` and update the `## Variables` section with your fork URL. The agent reads these variables at runtime to know where to sync the catalog.

```markdown
# Before (template defaults)
- **LIBRARY_REPO_URL**: `<your forked repo url>`

# After (your values)
- **LIBRARY_REPO_URL**: `https://github.com/yourname/the-library.git`
```

The other two variables (`LIBRARY_YAML_PATH` and `LIBRARY_SKILL_DIR`) are correct by default if you cloned to `~/.cursor/skills/library/`.

### 4. Verify

Start a new Cursor session anywhere. `/library list` should work and show an empty catalog.

## Quick Start

![Full Workflow](images/45_solution_full_workflow.svg)

Here's the typical workflow: **build → catalog → distribute → use**.

### Add a skill to the catalog

You built a deploy skill in one of your repos. Register it:

```
/library add deploy skill from https://github.com/yourorg/infra-tools/blob/main/skills/deploy/SKILL.md
```

This adds a reference to `library.yaml` and pushes the update to your fork.

### Use it in another project

On another device, repo, or agent:

```
/library use deploy
```

This pulls the skill from the source repo into `.cursor/skills/deploy/`.

Want it globally available on this machine?

```
/library use deploy install globally
```

### Push changes back

You improved the skill locally. Push the update to the source repo:

```
/library push deploy
```

Now every device that runs `/library sync` gets the latest version.

### Sync everything

Pull the latest version of all installed items:

```
/library sync
```

## Commands

| Command                     | What It Does                                               |
| --------------------------- | ---------------------------------------------------------- |
| `/library install`          | First-time setup — fork, clone, configure                  |
| `/library add <details>`    | Register a new entry in the catalog                        |
| `/library use <name>`       | Pull from source into local directory (install or refresh) |
| `/library push <name>`      | Push local changes back to the source                      |
| `/library remove <name>`    | Remove from catalog and optionally delete local copy       |
| `/library list`             | Show full catalog with install status                      |
| `/library sync`             | Re-pull all installed items from source                    |
| `/library search <keyword>` | Find entries by name or description                        |

### Justfile Shortcuts

The included `justfile` lets you run library commands from your terminal without opening an interactive Cursor session.

```bash
just list                  # List catalog
just use my-skill          # Pull a skill
just push my-skill         # Push changes back
just add "name: foo, description: bar, source: /path/to/SKILL.md"
just sync                  # Re-pull all installed items
just search "keyword"
```

> **Note:** Justfile recipes use `--dangerously-skip-permissions` because the agent needs filesystem and git access to clone, copy, and push. Review the `justfile` if you want to modify this behavior.

## Architecture

```
~/.cursor/skills/library/     # The Library skill (globally installed)
    SKILL.md                  # Agent instructions — the brain
    library.yaml              # Your catalog of references
    cookbook/                  # Step-by-step guides for each command
        install.md
        add.md
        use.md
        push.md
        remove.md
        list.md
        sync.md
        search.md
    justfile                  # CLI shorthand for all commands
    README.md                 # This file
```

## Design Principles

- **Private-first**: Built for your specialized, competitive-edge agentics. Not a public marketplace.
- **Reference-based**: The catalog stores pointers, not copies. Skills live in their source repos.
- **Pure agent**: No scripts, no build tools. The SKILL.md teaches the agent everything it needs to know.
- **Agent-agnostic**: Default target is `.cursor/skills/` but supports any directory for any agent harness.
- **Catalog, not manifest**: Entries define what's available, not what's installed. Pull on demand.

## The Agentic Stack

![The Agentic Stack](images/03_agentic_stack.svg)

| Layer           | Purpose                                        |
| --------------- | ---------------------------------------------- |
| **Skills**      | Raw capabilities — what an agent can do        |
| **Agents**      | Scale + parallelism + specialization           |
| **Prompts**     | Orchestration — coordinate skills and agents   |
| **Justfile**    | Terminal access without an interactive session |
| **The Library** | Distribution across devices, teams, and agents |
