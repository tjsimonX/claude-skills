## Vault behavior (Obsidian / PKM)

When working with vault notes at `~/Library/Mobile Documents/com~apple~CloudDocs/winsomeVault`, stay alert for these signals and surface them — don't act without asking:

- **Monolith signal** — a note is doing more than one job (e.g. virtues + hermetic principles + goals all in one file). Flag it and offer `/pkm-split <file>`.
- **Dump signal** — a note is a growing bullet-point list with no argument behind the claims. Flag it; observations belong in `Observations/[category].md` via the `@observation` Todoist label.
- **Duplicate signal** — the same idea appears in two notes. Flag it; one should become canonical and the other should link to it.
- **Orphan signal** — a new note has no link to a MOC or any other note. Add `↑ [[Maps/[Name] MOC]]` or flag it.
- **Title signal** — a note is titled as a topic ("Habits") not a claim ("Consistency beats intensity for habit formation"). Suggest a claim-style rename.

When *creating* new vault notes, default to: claim-style title, `↑ [[parent MOC]]` at top, at least one compass link (→ related, ⟳ tension) at the bottom.
