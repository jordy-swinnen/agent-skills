# agent-skills repository rules

This repository is an OpenPackage package of shared coding-agent skills. Consumers install `skills/` into their own `.claude/skills/` directory with OpenPackage.

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

The first checks the rulebook, skill names, references, portable frontmatter and MCP parity. The second tests MCP translation and OAuth handling. Neither reads secrets.

## Repository layout

```text
agent-skills/
├── openpackage.yml
├── AGENTS.md
├── CLAUDE.md
├── README.md
├── LICENSE
└── skills/
    ├── agent-handoff/
    ├── dual-agent-config/
    └── opencode-go-plus-build-plan-recommender/
```

Do not add a root `root/` directory or other files that OpenPackage would install into a consumer. Consumers install with `--skills` so this package's `AGENTS.md` stays in this repository and never overwrites a consumer's rulebook.

## Changes

- Conventional commit subjects.
- Update `README.md` when the install command, the included-skill table or the requirements change.
- Bump `version` in `openpackage.yml` for a release consumers should notice.
- Do not commit secrets, tokens or personal data. Configuration examples use placeholders.
