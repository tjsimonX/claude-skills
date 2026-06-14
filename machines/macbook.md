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

## Coding behavior

**Think before coding.** State assumptions explicitly before implementing. If multiple interpretations exist, surface them — don't pick silently. If something is unclear, stop and ask rather than guessing and fixing later.

**Surgical changes.** Touch only what the request requires. Don't improve adjacent code, reformat, or refactor things that aren't broken. If unrelated dead code is noticed, mention it — don't delete it. Every changed line should trace directly to the user's request.

**Brief plan for multi-step tasks.** Before starting anything with more than two steps, state the plan and success criteria:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
```
