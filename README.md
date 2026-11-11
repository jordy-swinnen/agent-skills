# Agent Skills

Paste this into any coding agent to install all skills from this repository:

```text
Install Jordy's shared agent skills from https://github.com/jordy-swinnen/agent-skills with OpenPackage.

1. Install the CLI if it is missing: npm install -g opkg
2. From the repository root, run these three commands:
   opkg install gh@jordy-swinnen/agent-skills/skills/dual-agent-config --platforms claude
   opkg install gh@jordy-swinnen/agent-skills/skills/agent-handoff --platforms claude
   opkg install gh@jordy-swinnen/agent-skills/skills/opencode-go-plus-build-plan-recommender --platforms claude
3. Always pass --platforms claude: both Claude Code and OpenCode discover .claude/skills/ natively, and this repository does not create .opencode/skills/.
4. Verify: python3 .claude/skills/dual-agent-config/scripts/check_config.py
```

Those commands add three portable skills to `.claude/skills/` and record each dependency in `.openpackage/` so updates can be pulled later. Installing a skill subpath avoids this package's root `AGENTS.md`, so a consumer's rulebook is never overwritten.

## What this repository is

`agent-skills` is an [OpenPackage](https://openpackage.dev) package of skills that are useful in any repository. It is deliberately project-agnostic: no product paths, build commands, ticket prefixes, domains or model defaults that only make sense in one codebase. Skills read those facts from the repository they are installed into, usually from `AGENTS.md`, or fetch live sources at runtime.

The package installs through [OpenPackage](https://github.com/enulus/OpenPackage), a universal package manager for coding-agent rules, commands, agents and skills. OpenPackage converts one package for many tools. This package intentionally targets the `claude` platform because both Claude Code and OpenCode load `.claude/skills/`; that keeps a single copy and avoids duplicate `.opencode/skills/`.

## Included skills

| Skill | What it does |
|---|---|
| [`dual-agent-config`](skills/dual-agent-config/SKILL.md) | Keeps Claude Code and OpenCode on one rulebook: `AGENTS.md` plus a `CLAUDE.md` import shim, shared skills, deliberate per-tool subagents/commands/MCP, and a file-level checker. Includes `sync_mcp.py` (OAuth-aware) and `check_config.py`. |
| [`agent-handoff`](skills/agent-handoff/SKILL.md) | Hands work between agent harnesses in either direction. The outgoing agent verifies checks, commits safe work, writes a handover document and a paste-ready prompt, and recommends the next model against a budget with BenchLM. The incoming agent takes over from that file. |
| [`opencode-go-plus-build-plan-recommender`](skills/opencode-go-plus-build-plan-recommender/SKILL.md) | Fetches the live OpenCode Go pricing page and current BenchLM rankings, then recommends Build and Plan models and prints cost/allowance tables for Go, Go Plus or Go Teams. No hardcoded prices or scores. |

## Install part of the package

Install one skill:

```bash
opkg install gh@jordy-swinnen/agent-skills/skills/agent-handoff --platforms claude
```

Install into the user-level directory instead of one repository:

```bash
opkg install gh@jordy-swinnen/agent-skills/skills/dual-agent-config --platforms claude --global
```

OpenCode also reads global `~/.claude/skills/`, so the global form serves both tools.

## Update

From the consuming repository, rerun the three subpath installs with `--platforms claude --force`:

```bash
opkg install gh@jordy-swinnen/agent-skills/skills/dual-agent-config --platforms claude --force
opkg install gh@jordy-swinnen/agent-skills/skills/agent-handoff --platforms claude --force
opkg install gh@jordy-swinnen/agent-skills/skills/opencode-go-plus-build-plan-recommender --platforms claude --force
```

OpenPackage records what it installed in `.openpackage/openpackage.yml` and `.openpackage/openpackage.index.yml`. Do not run bare `opkg install` in a repo that also has `.opencode/`: without `--platforms claude` it redetects platforms and writes duplicate `.opencode/skills/` copies, which both Claude Code and OpenCode already get from `.claude/skills/`.

## Add a skill

1. Create `skills/<lowercase-kebab-name>/SKILL.md`.
2. Keep the frontmatter to portable Agent Skills fields: `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`. `name` must match the directory name.
3. Keep every path inside the skill relative to its own directory. Put supporting docs in `references/` and scripts in `scripts/`.
4. Keep the skill generic. Read project commands and conventions from the consumer repository's `AGENTS.md`; never encode one project's paths, ticket format, build tools or domain.
5. Run the package checks:

   ```bash
   python3 skills/dual-agent-config/scripts/check_config.py --strict-portable
   python3 skills/dual-agent-config/scripts/test_sync_mcp.py
   ```

6. Commit and push. Consumers pull the change with `opkg install`.

## Requirements

- For `dual-agent-config`: Python 3 standard library, plus Python for the MCP tests.
- For `agent-handoff`: git and a POSIX shell. Model advice uses the BenchLM MCP server or its REST API.
- For `opencode-go-plus-build-plan-recommender`: network access to `https://opencode.ai/docs/console/go/` and the BenchLM MCP server.

## License

MIT. See [LICENSE](LICENSE).
