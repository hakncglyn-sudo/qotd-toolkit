# qotd-toolkit

A small, opinionated [Claude Code](https://claude.com/claude-code) plugin packaged from the
QOTD training scenario. Drop it into any TypeScript/SQLite/HTTP-API project to get the
same multi-agent team, skills, and safety hooks the scenario produced — without re-authoring
them per repo.

## What it ships

**5 specialist subagents:**

| Agent | Role | Model | Key trait |
| --- | --- | --- | --- |
| `architect` | Read-only planner — outputs plans, not code | Opus | `tools: Read, Grep, Glob` |
| `backend-dev` | Server + tests builder | Sonnet | `memory: project` (cross-session notes) |
| `frontend-dev` | Static UI builder, no Bash | Sonnet | preloads `project-map` skill |
| `tester` | Worktree-isolated test author | Sonnet | `isolation: worktree` |
| `security-auditor` | 5-category PR diff review | Opus | read-only Bash(git diff/log) |

**2 portable skills:**

- `project-map` — reference-style architectural map; auto-loads when Claude is asked
  about structure or where to put new code.
- `db-audit` — `context: fork` skill; runs in a forked Explore subagent so heavy schema
  + data-quality investigation doesn't bloat the main conversation context.

**2 hooks:**

- `PreToolUse` (Edit|Write) → `hooks/protect-files.sh` blocks writes to `.env`, `.env.local`,
  `data/qotd.db`, `.git/`, `.github/workflows/`. Adapt the protected list to your repo.
- `PostToolUse` (Edit|Write) → `npx prettier --write --ignore-unknown` on every edited file.

## Install

### Local install (testing)

```bash
claude --plugin-dir /path/to/qotd-toolkit
```

### Marketplace install

```text
/plugin marketplace add hakncglyn-sudo/qotd-toolkit
/plugin install qotd-toolkit@qotd-toolkit
/reload-plugins
```

## Requirements

- `jq` on PATH (the protect-files hook parses tool input with it).
- Prettier is invoked via `npx`, so no global install needed — but `npm install` in the
  consuming project should provide it (or remove the PostToolUse hook from `hooks.json`).
- For the `db-audit` skill, your project should have a SQLite file path you can pass
  to `sqlite3` and a corresponding `db.ts` style migration file.

## Security note

Plugin subagents cannot use `permissionMode`, `mcpServers`, or per-agent `hooks`
(security restriction in Claude Code). The `security-auditor` agent shipped here therefore
runs in whatever permission mode the parent session is in. If you want it to be plan-only,
add a `permissionMode: plan` line to a project-level copy in `.claude/agents/`.

## Origin

Built end-to-end as Phase 12 of the Softtech "Claude Code End-to-End Scenario" training,
extracted from `../qotd-service`.
