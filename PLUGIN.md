# Claude Code plugin (this fork)

This repository is a fork of [tcarac/taskboard](https://github.com/tcarac/taskboard).
Upstream CLI, UI, and MCP tools are unchanged. This fork adds a one-plugin
Claude marketplace at the repo root so coding agents can use Taskboard against
a **per-project** SQLite file.

## Layout

```
.claude-plugin/plugin.json         # plugin manifest
.claude-plugin/marketplace.json    # marketplace catalog (this plugin at ./)
skills/taskboard-workflow/SKILL.md
hooks/hooks.json                   # SessionStart snapshot + Stop commit-sync
scripts/session-sync.sh
scripts/commit-sync.sh
.mcp.json                          # taskboard --db ${CLAUDE_PROJECT_DIR}/.taskboard/taskboard.db mcp
```

## Install

1. Binary on PATH (upstream Homebrew or `make build` here):

   ```bash
   brew tap tcarac/taskboard && brew install taskboard
   ```

2. Claude Code marketplace:

   ```text
   /plugin marketplace add atebites-hub/taskboard
   /plugin install taskboard@taskboard
   ```

The MCP server always uses `${CLAUDE_PROJECT_DIR}/.taskboard/taskboard.db`.
Do not omit `--db` on CLI calls — the default OS config-dir database is the
wrong file for repo-scoped work.

## Hooks (fail-open)

- **SessionStart:** print `taskboard ticket list` (or a skip message). Exit 0 if
  the binary or DB is missing.
- **Stop:** for each `docs/memories/*.md` with `state: completed` and a
  `Taskboard: <uuid>` line, run `ticket move <uuid> --status done`. Exit 0 on
  any error.

C1/C3 git pre-commit compliance is **not** this plugin's job.

## Tickets vs memories

One ticket per sprint item in `docs/agents/implementation_plan.md`. Task
memories remain the C3 ledger. After creating a ticket, write
`Taskboard: <uuid>` on the matching memory (UUID, not `AUTH-1`).
