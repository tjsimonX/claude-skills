---
name: Roadmap Review
description: Interactively review a project's Todoist Roadmap section item by item — discuss each task and decide to implement (complete), drop (delete), or skip (move to Mining for further clarity). Invoke as /roadmap-review <ProjectName>.
disable-model-invocation: true
allowed-tools: Bash
---

## Roadmap Review

**Project:** $ARGUMENTS

If `$ARGUMENTS` is empty, tell the user: "Specify a project name — e.g. `/roadmap-review Winsome`" and stop.

---

## Step 1: Get the Todoist token

Run:
```bash
security find-generic-password -s "com.terrencesimon.winsome" -a "TODOIST_API_TOKEN" -w 2>/dev/null
```
If empty, fall back to:
```bash
/usr/libexec/PlistBuddy -c "Print TODOIST_API_TOKEN" ~/winsomeApp/Winsome/winsome/Config/Secrets.local.plist 2>/dev/null
```
If both fail, tell the user the token is missing and stop. Use the token as `TOKEN` for all subsequent calls.

---

## Step 2: Find the project

```bash
curl -s -H "Authorization: Bearer TOKEN" https://api.todoist.com/api/v1/projects
```
Find the project whose `name` matches `$ARGUMENTS` (case-insensitive). If no match, tell the user and stop. Store its `id` as `PROJECT_ID`.

---

## Step 3: Find the Roadmap section

```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/sections?project_id=PROJECT_ID"
```
Find the section named "Roadmap" (case-insensitive). Store its `id` as `SECTION_ID`.

If no Roadmap section exists, tell the user: "No Roadmap section found in [project]. Showing all tasks in the project instead." Then fetch tasks without a section filter.

Also look for a section named "Mining" (case-insensitive) and store its `id` as `MINING_SECTION_ID`. If no Mining section exists, create one now:
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"project_id": "PROJECT_ID", "name": "Mining"}' \
  "https://api.todoist.com/api/v1/sections"
```
Store the returned `id` as `MINING_SECTION_ID`.

---

## Step 4: Fetch roadmap tasks

```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/tasks?section_id=SECTION_ID"
```

If no tasks, tell the user the roadmap is clear and stop.

---

## Step 5: Review loop

Open with: **"X items in the Roadmap. Let's go through them one by one."**

For each task in order:

### Present
Show clearly:
- **Title**
- Description/notes (if any — read the `description` field from the already-fetched task)
- Item number: "**Item N of X**"

### Discuss
Give a direct 1–2 sentence take: is this worth implementing now? Consider complexity, fit with what's been shipped recently, and any signals in the description. Don't hedge — give a real recommendation.

### Wait for the user's decision

Accept any of:
| Input | Meaning |
|-------|---------|
| "implement", "yes", "do it", "keep" | Implement — close the task |
| "drop", "no", "delete", "remove" | Drop — delete the task |
| "skip", "later", "hold", "next" | Move to Mining — needs more clarity before it's ready |
| "stop", "done", "quit", "exit" | End the session now |

### Act

**Implement:**
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  "https://api.todoist.com/api/v1/tasks/TASK_ID/close"
```
Confirm: "✓ Marked complete — we're building this."

**Drop:**
```bash
curl -s -X DELETE \
  -H "Authorization: Bearer TOKEN" \
  "https://api.todoist.com/api/v1/tasks/TASK_ID"
```
Confirm: "✗ Deleted — off the list."

**Skip (move to Mining):**
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"section_id": "MINING_SECTION_ID"}' \
  "https://api.todoist.com/api/v1/tasks/TASK_ID"
```
Confirm: "→ Moved to Mining — needs more clarity."

Then immediately move to the next item — don't wait for acknowledgement.

---

## End of session

When all items are reviewed or the user says stop:

Show a clean summary:
```
Roadmap review complete.

Reviewed:    N
Implemented: N
Dropped:     N
Mined:       N
```

If items remain unreviewed (user said stop early), add: "X items left in the Roadmap."
