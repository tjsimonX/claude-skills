# Winsome Local — MacBook Pro
_Last verified: 2026-06-14_

You are running on the MacBook Pro (`Terrences-MacBook-Pro.local`), the local dev machine. The always-on server is the Mac mini (`homes-Mac-mini.local`) — `ssh home@homes-Mac-mini.local`. Full machine topology in `.claude/rules/machines.md` (Winsome project).

## Ground rules

- Always `git pull` before starting work on any project
- `git push` when done so the Mac mini picks up changes
- Do not run `deploy_winsome.sh` from this machine — deploys target the Mac mini's `/Applications/`. Use GitHub Actions (push to `main`) or SSH to the Mac mini.

## Skills & Commands — one source of truth

Personal Claude **skills and commands are version-controlled**, never loose files. There is exactly one source of truth: the `claude-skills` repo (`github.com/tjsimonX/claude-skills`), cloned at `~/winsomeApp/claude-skills`. The live directories under `~/.claude/` are symlinks into it:

```
~/.claude/CLAUDE.md ──symlink──▶  ~/winsomeApp/claude-skills/machines/macbook.md
~/.claude/skills    ──symlink──▶  ~/winsomeApp/claude-skills/skills
~/.claude/commands  ──symlink──▶  ~/winsomeApp/claude-skills/commands
```

The identical symlink arrangement exists on the Mac mini — `~/.claude/CLAUDE.md` points to `machines/macmini.md`.

**Never create loose files directly under `~/.claude/`.** That bypasses the repo and is exactly how the machines drift apart. Everything goes through the repo.

To add or change a skill/command: edit inside `~/winsomeApp/claude-skills`, commit, push, then `git pull` on the Mac mini. A skill can pin its model via a `model:` frontmatter key (e.g. `model: sonnet`).

## Git behavior

**Confirm before large commits.** Before running `git commit`, check `git diff --staged --stat`. If the total lines changed (insertions + deletions) is ≥ 50, or ≥ 5 files are changed, show the stat summary and proposed commit message and wait for approval. Below that threshold, commit directly.

## Vocabulary

These terms have specific meanings across the Winsome system. Use them consistently.

- **Mining** — a task that needs more work before it's implementation-ready. Lives in the Todoist "Mining" section. Still has open scope, unknown files, or unresolved questions. Processed via `/mine`.
- **Diamond** — an implementation-ready planning note at `Diamonds/[Project]/[Task].md` in the vault. Scope is locked, files identified, decision checklist fully resolved. No open unknowns. Produced by `/diamond` after real investigation. The standard is `Diamonds/SleeperDynastyAgent/Prompt-Cache Consolidation.md`.
- **Observation** — a raw insight or claim captured via Todoist `@observation` label. Routed nightly into `Observations/[Category].md` by `observe.sh`.
- **Protocol** — a structured procedure executed by WinsomeChat. Lives in `Protocols/` in the vault. Executable, not just reference.

## Vault behavior (Obsidian / PKM)

When working with vault notes at `~/Library/Mobile Documents/com~apple~CloudDocs/winsomeVault`, stay alert for these signals and surface them — don't act without asking:

- **Monolith signal** — a note is doing more than one job (e.g. virtues + hermetic principles + goals all in one file). Flag it and offer `/pkm-split <file>`.
- **Dump signal** — a note is a growing bullet-point list with no argument behind the claims. Flag it; observations belong in `Observations/[category].md` via the `@observation` Todoist label.
- **Duplicate signal** — the same idea appears in two notes. Flag it; one should become canonical and the other should link to it.
- **Orphan signal** — a new note has no link to a MOC or any other note. Add `↑ [[Maps/[Name] MOC]]` or flag it.
- **Title signal** — a note is titled as a topic ("Habits") not a claim ("Consistency beats intensity for habit formation"). Suggest a claim-style rename.

When *creating* new vault notes, default to: claim-style title, `↑ [[parent MOC]]` at top, at least one compass link (→ related, ⟳ tension) at the bottom.

## Coding behavior

**Think before coding.** State assumptions explicitly before implementing. If multiple interpretations exist, surface them — don't pick silently. If something is unclear, stop and ask rather than guessing and fixing later.

**Surgical changes.** Touch only what the request requires. Don't improve adjacent code, reformat, or refactor things that aren't broken. If unrelated dead code is noticed, mention it — don't delete it. Every changed line should trace directly to the user's request.

**Brief plan for multi-step tasks.** Before starting anything with more than two steps, state the plan and success criteria:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
```
