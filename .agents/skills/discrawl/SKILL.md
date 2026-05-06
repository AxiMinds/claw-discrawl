---
name: discrawl
description: Use for local Discord archive search, sync freshness, DMs, channel summaries, desktop/API/git-share sources, TUI browsing, and Discrawl repo/release work.
---

# Discrawl

Use local Discord archive data first for Discord questions. Hit Discord APIs
only when the archive is stale, missing the requested scope, or the user asks
for current external context.

## Sources

- DB: `~/.discrawl/discrawl.db`
- Config: `~/.discrawl/config.toml`
- Cache: `~/.discrawl/cache`
- Logs: `~/.discrawl/logs`
- Git share repo: `~/.discrawl/share`
- Repo: `openclaw/discrawl`; use `~/GIT/_Perso/discrawl` only after verifying
  its remote targets `openclaw/discrawl`, otherwise use a fresh checkout
- Preferred CLI: `discrawl`; fallback to `go run ./cmd/discrawl` from the repo if the installed binary is stale

## Freshness

For recent/current questions, check freshness before analysis:

```bash
discrawl status --json
```

For precise freshness from the default database:

```bash
sqlite3 ~/.discrawl/discrawl.db \
  "select coalesce(max(updated_at),'') from sync_state where scope like 'channel:%';"
```

Routine diagnostics:

```bash
discrawl doctor
```

Desktop-local refresh:

```bash
discrawl sync --source wiretap
```

Bot API latest refresh, when credentials are available:

```bash
discrawl sync
```

Use `--full` only for deliberate historical backfills:

```bash
discrawl sync --full
```

If SQLite reports busy/locked, check for stray `discrawl` processes before retrying.

## Query Workflow

1. Resolve scope: guild, channel, DM, author, keyword, date range.
2. Check freshness for recent/current requests.
3. Prefer CLI search/messages for slices; use read-only SQL for exact counts.
4. Report absolute date spans, counts, channel/DM names, and known gaps.

Common commands:

```bash
discrawl search "query"
discrawl messages --channel '#maintainers' --days 7 --all
discrawl dms --last 20
discrawl tui --dm
discrawl sql "select count(*) from messages;"
```

When the installed CLI lacks a new feature, build or run from a verified
`openclaw/discrawl` checkout before concluding the feature is missing.

## Discord Boundaries

Bot API sync requires configured Discord bot credentials; do not invent token
availability. Desktop wiretap mode reads local Discord Desktop artifacts and
must not extract credentials, use user tokens, call Discord as the user, or
write to Discord application storage. Wiretap/Desktop cache DMs are local-only
and must not be described as part of the published Git snapshot. Git-share
snapshots must not include secrets or `@me` DM rows.

## Verification

For repo edits, prefer existing Go gates:

```bash
GOWORK=off go test ./...
```

Then run targeted CLI smoke for the touched surface, for example:

```bash
discrawl doctor
discrawl status --json
discrawl search "test" --limit 5
```
