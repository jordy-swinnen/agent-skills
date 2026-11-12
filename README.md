# Agent Skills

Paste this into any coding agent to install all skills from this repository:

```text
Install Jordy's shared agent skills from https://github.com/jordy-swinnen/agent-skills with APM (Agent Package Manager).

1. Install the CLI if it is missing: brew install apm
2. From the repository root, run:
   apm install jordy-swinnen/agent-skills#v0.3.0
3. Make sure the consuming repository's apm.yml pins `targets: [claude]`. That is the only target that deploys skills
   to .claude/skills/, which both Claude Code and OpenCode read; any other target adds a second copy under
   .agents/skills/. Never run apm compile for this package.
4. Verify: python3 .claude/skills/dual-agent-config/scripts/check_config.py
```

`apm install` adds the dependency to `apm.yml`, writes `apm.lock.yaml` with the pinned commit and content hashes, and
deploys the four skills to `.claude/skills/`. It does not touch the consuming repository's `AGENTS.md`, `CLAUDE.md`
or its own skills.

## What this repository is

`agent-skills` is an [APM](https://microsoft.github.io/apm/) package of skills that are useful in any repository. It is
deliberately project-agnostic: no product paths, build commands, ticket prefixes, domains or model defaults that only
make sense in one codebase. Skills read those facts from the repository they are installed into, usually from
`AGENTS.md`, or fetch live sources at runtime.

The package declares `targets: [claude]` because both Claude Code and OpenCode load `.claude/skills/`. Every other APM
target converges on `.agents/skills/`, which would leave two copies of each skill competing for the same name.

## Included skills

| Skill | What it does |
|---|---|
| [`dual-agent-config`](skills/dual-agent-config/SKILL.md) | Keeps Claude Code and OpenCode 2 (CLI and desktop app) on one rulebook: `AGENTS.md` plus a `CLAUDE.md` import shim, shared skills, deliberate per-tool subagents/commands/MCP, and a file-level checker. Includes `sync_mcp.py` (OAuth-aware) and `check_config.py`. |
| [`agent-handoff`](skills/agent-handoff/SKILL.md) | Hands work between agent harnesses in either direction. The outgoing agent verifies checks, commits safe work, writes a handover document and a paste-ready prompt, and recommends the next model against a budget with BenchLM. The incoming agent takes over from that file. |
| [`autonomous-work`](skills/autonomous-work/SKILL.md) | Keeps the agent working while the user is away: it settles what it can alone, takes reversible defaults and records them, parks blocking questions instead of stopping, never widens what the user approved, and ends with one phone-sized report of results, assumptions and numbered questions. |
| [`opencode-go-plus-build-plan-recommender`](skills/opencode-go-plus-build-plan-recommender/SKILL.md) | Fetches the live OpenCode Go pricing page and current BenchLM rankings, then recommends Build and Plan models and prints cost/allowance tables for Go, Go Plus or Go Teams. No hardcoded prices or scores. |

## Install part of the package

Install only some skills:

```bash
apm install jordy-swinnen/agent-skills#v0.3.0 --skill agent-handoff --skill dual-agent-config
```

Install into the user-level directory instead of one repository:

```bash
apm install -g jordy-swinnen/agent-skills#v0.3.0
```

OpenCode also reads global `~/.claude/skills/`, so the global form serves both tools.

## Update

```bash
apm update        # re-resolve the pinned ref and redeploy
apm outdated      # see what moved upstream, without writing
```

`apm update` rewrites `apm.lock.yaml`; commit it with the redeployed files. A CI run installs the locked state instead:

```bash
apm install --frozen
```

`apm update` is safe here because `apm.yml` declares `targets: [claude]`, which outranks filesystem auto-detection — a
repository that also has `.opencode/` still gets no `.opencode/skills/` and no `.agents/skills/`.

Keep `apm_modules/` out of git (it is the installed copy, like `node_modules/`); the deployed `.claude/skills/` files
are committed so clones and non-APM environments still work.

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

6. Bump `version:` in `apm.yml`, commit, push, and tag (`git tag v0.1.1 && git push origin main --tags`). Consumers
   pull the change with `apm update`.

## Requirements

- For `dual-agent-config`: Python 3 standard library, plus Python for the MCP tests.
- For `agent-handoff`: git and a POSIX shell. Model advice uses the BenchLM MCP server or its REST API.
- For `autonomous-work`: nothing beyond the harness. It uses the repository's own checks and, when the harness has one, a notification tool.
- For `opencode-go-plus-build-plan-recommender`: network access to `https://opencode.ai/docs/console/go/` and the BenchLM MCP server.

## License

MIT. See [LICENSE](LICENSE).
