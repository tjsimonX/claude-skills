---
name: pkm-split
description: >
  Restructure an Obsidian vault to honor Zettelkasten/LYT principles — atomic notes,
  proper linking discipline, MOC-based navigation, and clean file separation.

  Use this skill when the user asks to "restructure the vault", "split obsidian files",
  "apply Zettelkasten", "organize my PKM", "separate my notes", "link my notes", or
  any time they want their Obsidian vault reorganized for navigability and atomicity.

  Invoke as /pkm-split [vault-path]
  Defaults to ~/Library/Mobile Documents/com~apple~CloudDocs/winsomeVault if no path given.
---

# PKM Split Skill

You are restructuring an Obsidian vault to honor these three principles:

1. **Zettelkasten** — one idea per note, titles as claims, every note linked
2. **LYT** — MOCs as navigation layer, Home note as vault root, folders only for file types
3. **Compass of Zettelkasten** — North/South/East/West links surface conceptual relationships

The PKM linking guide lives at `~/winsomeApp/pkm-linking-guide.md`. Read it in full before starting.

---

## Phase 1 — Resolve the vault path

If `$ARGUMENTS` is provided, use it as the vault path. Otherwise default to:
`~/Library/Mobile Documents/com~apple~CloudDocs/winsomeVault`

Expand `~` to the full home directory.

---

## Phase 2 — Audit the vault

Run these in sequence:

```bash
find "<vault-path>" -type f -name "*.md" | sort
```

Then read every file that has substantive content (more than frontmatter + a few lines).
Skip files that are clearly Winsome system files (Protocols/, Projects/) — read everything else.

As you read, classify each file into one of these categories:
- **Project stub** — thin Todoist-linked wrapper with frontmatter `todoist_project_id`
- **Monolith** — one file doing multiple jobs (multiple distinct ideas bundled together)
- **Atomic** — already one idea, one file
- **Dump** — bullet-point collection across many topics
- **Reference** — lookup tables, glossaries, scripts
- **Creative** — fiction, worldbuilding, production bibles

Track:
- Files with duplicate content (same ideas in multiple places)
- Files with no incoming links (orphan candidates)
- Large files that mix distinct ideas

---

## Phase 3 — Ask four clarifying questions before doing anything

Present your audit findings briefly, then ask:

**Q1 — System file scope**
Are there folders that should be left completely alone? (e.g. Protocols/, Projects/ for Winsome users)
Options: Leave them alone / Reorganize them too

**Q2 — Duplicate content**
List any notes with overlapping content you found. For each pair/group:
Should I merge them into one canonical note, keep them separate, or pick one and drop the other?

**Q3 — Dump file atomization level**
For each dump/bullet-point file you found, ask how granularly to split it:
- Full Zettelkasten — one file per claim (~50-100 new files for a big dump)
- Category files — one file per section/theme (manageable, ~5-10 files per dump)
- Clean up in place — better headers, no new files

**Q4 — Monolith strategy**
For each monolith, confirm: split into atomic notes, or keep unified with better linking?
Note: long but cohesive living documents (like a North Star plan) usually stay unified.

Do not proceed until all four questions are answered.

---

## Phase 4 — Plan and state it

Before writing a single file, state the full plan:

```
New files to create:
- Home.md
- Maps/[Name] MOC.md (list each)
- [Canonical merged notes] (list each, note source files)
- [Split atomic notes] (list each, note source file)
- [Category files from dumps] (list each)

Files to modify:
- [file] — [what changes: content removed, links added, etc.]

Files left alone:
- [list folders/files not touched]
```

Wait for the user to confirm the plan before executing.

---

## Phase 5 — Execute

Work in this order:

### 5a — Create folder structure
- `Maps/` — for MOC files only
- Domain subfolders only if there are 5+ notes that belong together (e.g. `Hermetic/`, `Observations/`)
- Never create folders for knowledge organization — use MOCs instead

### 5b — Create Home.md
The vault root. Links only to MOCs (not individual notes). Stable and minimal.

```markdown
---
type: home
---

# [Vault Name]

> [One-line vault purpose or North Star]

---

## Maps

- [[Maps/[Name] MOC|[Name]]]
- ...

---

## Key Notes

- [[Note 1]]
- ...
```

### 5c — Create MOC files
One MOC per major knowledge domain. Each MOC:
- Opens with `↑ [[Home]]`
- Has a one-line purpose statement
- Groups related notes with section headers
- Uses `[[wikilink]]` syntax throughout
- Is a living document — explicitly note it should be updated as notes are added

```markdown
---
type: MOC
pillar: [domain]
---

↑ [[Home]]

# [Domain] MOC

> [One sentence: what this map covers]

---

## [Section]

- [[Note]] — one-line description
```

### 5d — Create merged canonical notes
For each duplicate pair/group: write one canonical file that reconciles both versions.
Use the more complete/refined version as the base. Add unique items from the other.
In the source files, replace the duplicate content with a link to the canonical note.

### 5e — Create atomic notes from monoliths
For each section extracted from a monolith:
- Title as a claim or concept, not a topic ("Mentalism — the universe is mind" not "Hermetic Principle 1")
- Add `↑ [[Parent MOC]]` at top
- Add Compass links at bottom where clear (→ related East, ⟳ tension West)
- Keep each file focused on one idea

### 5f — Create category files from dumps
For each dump file:
- Split into category files named `[Domain] — [Category].md`
- Convert the original dump file into an index linking to all category files
- Add `↑ [[Parent MOC]]` to each category file

### 5g — Add navigation to existing notes
For any file not already linking up to a MOC, add `↑ [[Maps/[Name] MOC|[Name] MOC]]` after the frontmatter.

For North Star / vision documents: add the MOC nav link but leave content intact.

---

## Phase 6 — Link discipline pass

After all files are created/modified, do one pass specifically for links:

For each new atomic note, run the Compass:
- **North** — what concept or category is this part of? → link up to MOC or parent concept
- **South** — what are concrete examples or instances? → link down to examples
- **East** — what is similar or analogous? → link to related notes
- **West** — what contradicts or tensions with this? → use `⟳ [[Note]]`

Only add links where there is a genuine conceptual relationship. Don't force it.

---

## Phase 7 — Confirm before closing

Present a summary:
- New files created (count + list)
- Files modified (count + what changed)
- Files untouched

Ask: "Does this look right?" Do not close any Todoist task or commit any git changes until the user confirms.

---

## Rules throughout

- **Never delete source content without replacement.** Extract to a new file, then replace with a link. Don't silently remove.
- **Preserve Todoist frontmatter.** Never remove `id` or `todoist_project_id` fields from project stubs.
- **Folders are for file types, not knowledge.** MOCs handle knowledge organization.
- **Titles are claims, not topics.** "AI shortens the integration phase between learning and application" beats "AI and Learning".
- **Orphan notes are a signal.** If a note has no MOC link after the restructure, flag it.
- **One MOC per domain, not one MOC per file.** MOCs should be sparse at first and grow as the vault grows.
- **Don't over-atomize.** A cohesive living document (North Star plan, series bible, social OS) can stay unified. Atomic doesn't mean microscopic — it means one idea, fully developed.
