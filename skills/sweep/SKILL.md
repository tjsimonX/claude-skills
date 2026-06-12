---
name: Sweep
description: Sweeps every Todoist project for tasks that have no section and no due date. Standard projects (including the Dev parent) use a dynamic section-based flow. Dev sub-projects invoke /roadmap-review. Invoke as /sweep.
disable-model-invocation: true
model: sonnet
---

## Sweep

Cycle through every Todoist project and surface tasks that have **no section** and **no due date**. Dev child projects hand off to `/roadmap-review`. All other projects (including the Dev parent) use a dynamic flow based on whatever sections that project already has.

---

## Step 1: Get the Todoist token

```bash
security find-generic-password -s "com.terrencesimon.winsome" -a "TODOIST_API_TOKEN" -w 2>/dev/null
```
If empty, fall back to:
```bash
/usr/libexec/PlistBuddy -c "Print TODOIST_API_TOKEN" ~/winsomeApp/Winsome/winsome/Config/Secrets.local.plist 2>/dev/null
```
If both fail, tell the user the token is missing and stop. Store as `TOKEN`.

---

## Step 2: Fetch all projects and build the list

```bash
curl -s -H "Authorization: Bearer TOKEN" https://api.todoist.com/api/v1/projects
```

From the response:
- Find the project whose `name` is "Dev" (case-insensitive). Store its `id` as `DEV_PARENT_ID`.
- Collect all projects whose `parent_id` equals `DEV_PARENT_ID` — these are **Dev sub-projects** (dev-review flow).
- Find the project whose `name` is "Lists" (case-insensitive). Store its `id` as `LISTS_PARENT_ID`. Exclude this project and any project whose `parent_id` equals `LISTS_PARENT_ID` from the sweep entirely — these are shopping/reference lists.
- Everything else except Inbox — including the Dev parent itself — is a **standard project** (dynamic sections flow).

---

## Step 3: Announce the plan

```
Sweep starting.

Standard projects: [list names, including Dev parent if it has tasks]
Dev sub-projects → /roadmap-review: [list names]

Total: N projects to sweep.
```

---

## Step 4: Standard flow (all non-Dev-child projects)

For each standard project, in order:

### 4a. Fetch tasks and sections in parallel

Tasks:
```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/tasks?project_id=PROJECT_ID"
```
Filter to tasks where **both** conditions hold:
- `section_id` is null or absent
- `due` is null or absent

Skip any task whose `content` starts with `*`.

Sections:
```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/sections?project_id=PROJECT_ID"
```
Store the full list of sections (id + name) for this project.

If no tasks pass the filter, print: "**[Project]** — nothing to triage. Skipping." and move on.

### 4b. Open the project

Print:
```
[Project] — N tasks.
Sections: [Section A · Section B · Section C]   (or "No sections yet" if empty)
```

### 4c. Task loop

For each task in order:

**Present:**
- **Title** + description (if any)
- "**Item N of X**"

**Assess ownership:** Before giving a take, determine who owns this task:
- **Claude** — code changes, file edits, API calls, script runs, config updates, anything executable in a Claude Code session
- **User** — requires physical presence, a human decision, an external account, a phone call, or real-world action

Show this clearly on its own line:
- `[Claude]` — I can do this
- `[You]` — needs you

**Give a take:** 1–2 direct sentences — what is this, and where does it most naturally belong given the project's existing sections?

**Wait for decision:**

| Input | Action |
|-------|--------|
| Any existing section name or unambiguous prefix | Move task to that section |
| "spawn", "spin up", "claude" | Invoke `/spawn <task_id>` — launches a context-aware Claude Code session, moves task to In-Progress, then resumes sweep |
| "drop", "delete", "d" | Delete the task |
| "keep", "skip", "k" | Leave as-is, move on |
| "move to <Project>" or any project name | Move task to that project (no section) |
| "stop", "done", "quit" | End the entire sweep |

If the user types a section name that doesn't exist in this project, ask them to confirm before creating it. If confirmed, create the section first, then move the task.

**Act:**

Move to section:
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"section_id": "MATCHED_SECTION_ID"}' \
  "https://api.todoist.com/api/v1/tasks/TASK_ID"
```
Confirm: "→ [Section name]."

Drop:
```bash
curl -s -X DELETE \
  -H "Authorization: Bearer TOKEN" \
  "https://api.todoist.com/api/v1/tasks/TASK_ID"
```
Confirm: "✗ Deleted."

Keep: Confirm: "↩ Kept." Move on.

Move to another project (look up id from the already-fetched projects list, case-insensitive):
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"project_id": "TARGET_PROJECT_ID"}' \
  "https://api.todoist.com/api/v1/tasks/TASK_ID/move"
```
Confirm: "→ Moved to [Project]."

After acting, immediately present the next task — no pause.

When the project is exhausted, print a one-line summary:
```
[Project] done — Moved: N | Dropped: N | Kept: N
```

---

## Step 5: Dev review flow (Dev sub-projects)

For each Dev sub-project, in order:

Announce: **"Now reviewing [ProjectName] via dev-review."**

Then invoke `/dev-review [ProjectName]`.

Wait for it to complete before moving to the next sub-project.

---

## End of sweep

When all projects are done or the user says stop:

```
Sweep complete.

[One line per project: name — what happened]
```
