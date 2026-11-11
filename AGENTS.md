# agent-skills repository rules

This repository is an APM (Agent Package Manager) package of shared coding-agent skills. Consumers install `skills/`
into their own `.claude/skills/` directory with `apm install`.

## Portability

- Every skill must work in any repository. Never encode one project's paths, build commands, ticket prefixes, domains, product names or model defaults.
- Read project-specific facts from the consuming repository's `AGENTS.md` at runtime, or fetch live sources. Do not hardcode prices, rankings, versions or model lists.
- Keep each skill inside its own `skills/<name>/` directory and make every path inside it relative to that directory.
- Use only portable Agent Skills frontmatter: `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`. `name` must equal the directory name.
- Put supporting documentation in `references/` and executable helpers in `scripts/`.

## Checks

Run both before committing:

```bash
python3 skills/dual-agent-config/scripts/check_config.py --strict-portable
python3 skills/dual-agent-config/scripts/test_sync_mcp.py
```

The first checks the rulebook, skill names, references, portable frontmatter, the single skill copy and MCP parity. The
second tests MCP translation and OAuth handling. Neither reads secrets.

## Repository layout

```text
agent-skills/
├── apm.yml
├── AGENTS.md
├── CLAUDE.md
├── README.md
├── LICENSE
└── skills/
    ├── agent-handoff/
    ├── dual-agent-config/
    └── opencode-go-plus-build-plan-recommender/
```

`apm.yml` declares `targets: [claude]` on purpose. `claude` is the only APM target that deploys skills to
`.claude/skills/`, which both Claude Code and OpenCode read; every other target converges on `.agents/skills/` and would
leave a second copy competing for the same skill name. `check_config.py` fails when `apm.yml` names another target, when
`.agents/skills/` or `.opencode/skills/` exists, or when a skill sits below the top level of `.claude/skills/`.

Do not add root files that a consumer would receive as primitives. Consumers install the package or a skill subpath,
which keeps this package's `AGENTS.md` and `CLAUDE.md` in this repository and never overwrites a consumer's rulebook.
Never run `apm compile` for this package: it writes a root `CLAUDE.md`/`AGENTS.md` and has no part in shipping skills.

## Changes

- Conventional commit subjects.
- Update `README.md` when the install command, the included-skill table or the requirements change.
- Bump `version` in `apm.yml` for a release consumers should notice, then tag it (`v0.1.1`) so consumers can pin it.
- Do not commit secrets, tokens or personal data. Configuration examples use placeholders.
