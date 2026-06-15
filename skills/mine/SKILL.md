---
name: Mine
description: Refine tasks from a project's Todoist Mining section one by one — implement (close), create a Diamond planning note in the vault, or drop. Invoke as /mine <ProjectName>.
disable-model-invocation: true
allowed-tools: Bash
---

## Mine

**Project:** $ARGUMENTS

If `$ARGUMENTS` is empty, tell the user: "Specify a project name — e.g. `/mine Winsome`" and stop.

---

## Step 1: Get the Todoist token

```bash
security find-generic-password -s "com.terrencesimon.winsome" -a "TODOIST_API_TOKEN" -w 2>/dev/null
```
If empty, fall back to:
```bash
/usr/libexec/PlistBuddy -c "Print TODOIST_API_TOKEN" ~/winsomeApp/Winsome/winsome/Config/Secrets.local.plist 2>/dev/null
```
If both fail, tell the user the token is missing and stop. Use the token as `TOKEN`.

---

## Step 2: Find the project

```bash
curl -s -H "Authorization: Bearer TOKEN" https://api.todoist.com/api/v1/projects
```
Find the project whose `name` matches `$ARGUMENTS` (case-insensitive). If no match, tell the user and stop. Store its `id` as `PROJECT_ID`.

---

## Step 3: Find the Mining section

```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/sections?project_id=PROJECT_ID"
```

Find the section named "Mining" (case-insensitive). Store its `id` as `MINING_SECTION_ID`. If it doesn't exist, tell the user: "No Mining section found in [project]. Nothing to refine." and stop.

---

## Step 4: Fetch Mining tasks

```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/tasks?section_id=MINING_SECTION_ID"
```

If no tasks, tell the user: "Mining is empty — nothing to refine." and stop.

---

## Step 5: Refinement loop

Open with: **"X items in Mining. Let's go through them."**

For each task in order:

### Present

Show:
- **Title**
- Description/notes (if any, from the `description` field)
- Item number: **"Item N of X"**

### Refine

Your job is to help determine whether this task is ready to implement directly, or whether it needs a Diamond — a vault planning note to lock in scope, research, and design before touching code.

Probe for:
- **The why** — what problem does this solve?
- **Done criteria** — what does done look like?
- **The where** — which files or components does this touch?
- **Blockers** — what needs to be true first?

Ask one question at a time. Stay in the conversation until the path is clear: either the scope is tight enough to implement directly, or it's too open to start without a plan.

### Wait for a decision

| Input | Meaning |
|---|---|
| "implement", "ship", "close", "do it", "yes" | Implement — close the Todoist task |
| "diamond", "plan", "vault", "scope it" | Create a Diamond planning note in the vault |
| "drop", "no", "delete", "not worth it" | Delete the task |
| "skip", "later", "not ready" | Leave in Mining as-is |
| "stop", "done", "quit" | End the session |

### Act

**Implement (close):**
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  "https://api.todoist.com/api/v1/tasks/TASK_ID/close"
```
Confirm: "✓ Closed — go build it."

---

**Diamond (create vault planning note):**

The vault lives at `~/Library/Mobile Documents/com~apple~CloudDocs/winsomeVault`.
Diamonds live at `Diamonds/[ProjectName]/[TaskName].md`.

Use the project name from `$ARGUMENTS` as the subfolder. Slugify the task title for the filename (spaces → spaces is fine; strip special characters). Create the subfolder if it doesn't exist.

Write the note using this template — fill in what you know from the refinement conversation, leave placeholders where you don't:

```markdown
---
title: [Task title]
project: [Project name]
status: proposed
type: implementation-note
created: [Today's date YYYY-MM-DD]
---

↑ [[Projects/[ProjectName]|[ProjectName]]]

# [Task title]

> [One-line summary: what this implements and why]

---

## Background

[Why this matters; what problem it solves]

---

## The Problem

[Current state; what's broken or missing]

---

## Proposed Implementation

[What to build; key design decisions]

### Files to touch

-

---

## Decision Checklist

- [ ] [Key open question from the refinement conversation]
- [ ] Confirm scope before implementing

---

## Related

- Todoist: [Project name] → Mining
```

Leave the Todoist task in Mining — the Diamond is the work surface. The task stays until the Diamond's checklist is resolved and you're ready to implement.

Confirm: "◆ Diamond created at Diamonds/[ProjectName]/[TaskName].md — open it to plan."

---

**Drop:**
```bash
curl -s -X DELETE \
  -H "Authorization: Bearer TOKEN" \
  "https://api.todoist.com/api/v1/tasks/TASK_ID"
```
Confirm: "✗ Dropped."

**Skip:**
Confirm: "→ Stays in Mining."

Then move immediately to the next item.

---

## End of session

```
Mining session complete.

Reviewed:  N
Implemented: N
Diamonds:  N
Dropped:   N
Remaining: N
```
