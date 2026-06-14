# Winsome Server — Mac mini
_Last verified: 2026-06-14_

You are running on the Mac mini (`homes-Mac-mini.local`), the always-on server — deployment target, GitHub Actions runner, session host. Full machine topology in `.claude/rules/machines.md` (Winsome project).

## LaunchAgents

| Label | Role | Cadence | Logs |
|-------|------|---------|------|
| `com.terrencesimon.winsome` | Winsome orchestration app — calendar poller, protocol engine, Todoist task creator | Always-on | `/tmp/winsome.stdout.log` |
| `com.terrencesimon.winsome-local-dispatch` | Task dispatcher — polls Todoist for `ship`/`test`/`session` tasks and routes to Claude instances | Every 5 min | `/tmp/winsome-local-dispatch.stdout.log` |
| `com.terrencesimon.winsome-session-server` | Session server — HTTP API on port 7891 for Winsome Field to open Claude sessions via Tailscale | Always-on | `/tmp/winsome-session-server.stdout.log` |
| `com.terrencesimon.winsome-notes-sort` | Notes sorter — moves Todoist `notes` tasks into the Notes section of their project | Daily 09:00 | `/tmp/winsome-notes-sort.stdout.log` |
| `com.terrencesimon.winsome-observe` | Observation filer — categorizes Todoist `observation` tasks via Claude and appends to vault | Daily 22:00 | `/tmp/winsome-observe.stdout.log` |

Dispatcher routes: `winsomeTerminal/config.json` → `machines.server.routes`.

## Skills & Commands — one source of truth

Same symlink pattern as the MacBook — all three point into `~/winsomeApp/claude-skills`:

```
~/.claude/CLAUDE.md ──symlink──▶  ~/winsomeApp/claude-skills/machines/macmini.md
~/.claude/skills    ──symlink──▶  ~/winsomeApp/claude-skills/skills
~/.claude/commands  ──symlink──▶  ~/winsomeApp/claude-skills/commands
```

## Ground rules

- Always `git pull` before starting any work — the MacBook may have pushed recent changes. When launched via Winsome Field, `session_server.py` already does this before Claude starts.
- `git push` when done
- Use `scripts/deploy_winsome.sh` for release builds → installs to `/Applications/Winsome/`
- Use `scripts/deploy_winsome_debug.sh` for debug builds → installs to `/Applications/Winsome_Debug/`
- GitHub Actions auto-deploys on push to `main` when `winsome/`, the Xcode project, or `deploy_winsome.sh` change. Check Actions status before a manual deploy to avoid conflicts.

## Monitoring

After any deploy, confirm Winsome is still running:
```bash
launchctl list | grep com.terrencesimon.winsome
log show --last 10m --predicate 'process == "winsome"' --style compact
```

## Coding behavior

**Think before coding.** State assumptions explicitly before implementing. If multiple interpretations exist, surface them — don't pick silently. If something is unclear, stop and ask rather than guessing and fixing later.

**Surgical changes.** Touch only what the request requires. Don't improve adjacent code, reformat, or refactor things that aren't broken. If unrelated dead code is noticed, mention it — don't delete it. Every changed line should trace directly to the user's request.

**Brief plan for multi-step tasks.** Before starting anything with more than two steps, state the plan and success criteria:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
```

## Claude Code launch context

SSH sessions cannot access the login Keychain — always launch Claude via:
```bash
sudo launchctl asuser $(id -u) sudo -u home /bin/zsh -l -c 'claude --dangerously-skip-permissions'
```
Full explanation in `.claude/rules/machines.md` (Winsome project).
