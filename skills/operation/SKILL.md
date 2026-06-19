---
name: Operation
description: Operationalize a goal — run the Goal Framework interview, then materialize it into Todoist (project + sub-projects + tasks) and the Obsidian vault (Goal + Objective notes) with full parity, placed on the Life OS ladder by horizon (now/next/later). Use for a single goal rung; use /strategic-roadmap for long-term, multi-year visions. Invoke as /operation <name or goal>.
model: sonnet
allowed-tools: Bash, Read
---

# Operation

Run the [[Goal Framework]] interview for a single **goal**, then build it out
in both Todoist and the vault with bidirectional parity, placed on the [[Life OS]]
ladder by **horizon** (now / next / later, climbing toward the North Star).

"Operation" here is the **verb** — you *operationalize* a goal. The artifact produced is a
**Goal** note (a rung on the ladder), not a separate "operation" object.

**Scope boundary:** This is for one goal with a concrete finish line (weeks to ~a year).
For long-term, multi-year visions, use `/strategic-roadmap` instead. If the goal is clearly
multi-year and open-ended, say so and point to that skill.

## The parity model (don't deviate)

The reference structure is the existing **Eagle Has Landed** Todoist project:

| Element | Todoist | Vault |
|---|---|---|
| **Goal** | top-level project | `Goals/<Name>/Goal — <Name>.md` |
| **Objective** | sub-project | `Goals/<Name>/Objective — <Obj>.md` |
| **Step** | task under the sub-project | numbered line in the Objective note |

Parity is bidirectional and is created by the script, not by you:
- vault → todoist: `todoist_project_id` frontmatter
- todoist → vault: a pinned `* obsidian://advanced-uri?vault=winsomeVault&uid=<id>` task
  whose uid equals the note's `id` frontmatter

## Procedure

### 1. Interview — fill the framework, in order

Walk the four layers of [[Goal Framework]] **in order**. The whole value of this
skill is refusing to skip Layer 1. Do not let the user jump straight to steps.

- **Layer 1 — The Why (Fi anchor).** Ask for: core value statement, authenticity check
  ("would you want this if no one ever saw the result?"), violation test. **Do not
  proceed to Layer 2 until these are answered.** If the user resists, that resistance is
  the signal — surface it, don't paper over it.
- **Layer 2 — The Terrain (Ne).** Briefly explore possible paths, then pin: chosen path,
  kill criteria. Keep this light — it's exploration, not commitment.
- **Layer 3 — The Operation (Te).** Goal name; **horizon** (`now` / `next` / `later` — where
  it sits on the Life OS ladder); the **North Star** it climbs toward; what it **promotes to**
  when complete (the goal that becomes Now); the goal; cadence (default
  "Weekly review by default."); kill switch. Then gather **objectives** — each becomes a
  sub-project. For each objective: a one-line `why`, and its `steps` (each step is a task;
  capture a due date where one exists, or a recurring interval).
  Once horizon is set, read the **Horizon allocation** block in [[Life OS]] and tell the user
  the focus share for that rung (e.g. "this is your Now rung → ~70% of focus") — so they're
  committing a realistic slice of time, not treating it as overflow.
- **Layer 4 — The Persona.** Name, identity (present tense), 2–3 routines, internal motto,
  shadow risk + Fi guardrail. This is optional-but-encouraged; skip fields the user has no
  answer for rather than inventing them.

Never invent Layer 1 or Layer 4 answers — those are the user's. Steps and structure you
may draft for confirmation.

### 2. Assemble the spec

Build a JSON spec (schema below) from the interview. Write it to a temp file.

### 3. Preview, then confirm

Run the script with `--dry-run` and show the user the tree + note preview. This is an
outward mutation (creates real Todoist projects) — **get explicit confirmation before the
live run.**

```bash
bash "$HOME/winsomeApp/Winsome/scripts/operation_create.sh" --spec /tmp/operation_spec.json --dry-run
```

### 4. Build it

On confirmation, drop `--dry-run`:

```bash
bash "$HOME/winsomeApp/Winsome/scripts/operation_create.sh" --spec /tmp/operation_spec.json
```

The script refuses to run if a top-level project of the same name or the vault folder
already exists — so it won't duplicate a goal. Report the created project id, note
paths, and counts back to the user.

### 5. Close out

Remind the user the vault notes are under `Goals/<Name>/` and linked to Todoist. The
`notes.sh` pipeline already routes `@notes` tasks by `todoist_project_id`, so the new
sub-projects participate automatically — no pipeline change needed.

## Spec schema

```json
{
  "goal": {
    "name": "Eagle Has Landed",
    "pillar": "Identity",
    "horizon": "now",
    "north_star": "Network State Superpower",
    "promotes_to": "Tech Job",
    "goal": "Single overarching outcome.",
    "why":     { "core_value": "", "authenticity": "", "violation": "" },
    "terrain": { "chosen_path": "", "kill_criteria": "" },
    "cadence": "Weekly review by default.",
    "kill_switch": "Pre-decided pause/restructure condition.",
    "persona": {
      "name": "", "identity": "",
      "routines": ["", ""],
      "motto": "", "shadow_risk": ""
    }
  },
  "objectives": [
    {
      "name": "Education",
      "why": "One line tying it to the goal.",
      "steps": [
        { "content": "Submit graduation application", "due": "2026-07-01" },
        { "content": "Purchase textbooks" },
        { "content": "Weekly ops review", "due_string": "monday" }
      ],
      "notes": "Free field — blockers, criteria, context."
    }
  ]
}
```

- A step uses **either** `due` (a `YYYY-MM-DD` date) **or** `due_string` (a recurring
  interval like `"monday"` / `"every 2 weeks"`), or neither.
- `horizon` is `now`, `next`, or `later` — the rung on the Life OS ladder. `north_star` is
  the fixed star it climbs toward. `promotes_to` is the goal name that becomes Now when this
  one completes (omit if unknown).
- `pillar` mirrors the existing vault convention (e.g. `Identity`). Ask once; applies to the
  whole goal.
- Omit persona/terrain fields the user didn't answer — the script renders only what's present.
