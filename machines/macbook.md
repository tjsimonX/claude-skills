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

## Shared modules

One source of truth in `claude-skills/shared/`, imported by both machine configs. Edit those files, not copies.

@~/winsomeApp/claude-skills/shared/git-behavior.md
@~/winsomeApp/claude-skills/shared/vocabulary.md
@~/winsomeApp/claude-skills/shared/vault-behavior.md
@~/winsomeApp/claude-skills/shared/coding-behavior.md
