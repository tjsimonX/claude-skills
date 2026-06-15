---
name: Mine
description: Refine tasks from a project's Todoist Mining section one by one — implement (close) or drop. If a task needs deeper research before it's implementation-ready, flag it for /diamond. Invoke as /mine <ProjectName>.
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
- Description/notes if any (from the `description` field)
- Item number: **"Item N of X"**

### Refine

Probe for:
- **The why** — what problem does this solve?
- **Done criteria** — what does done look like?
- **The where** — which files or components does this touch?
- **Blockers** — what needs to be true first?

Ask one question at a time. The goal is a single clear implementation sentence. If you can't get there — scope is too open, too much unknown, needs investigation — call it: **"This needs a Diamond before it's ready. Use `/diamond [task title]` to research and scope it."** Then move on.

### Wait for a decision

| Input | Meaning |
|---|---|
| "implement", "ship", "close", "do it", "yes" | Implement — close the Todoist task |
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

Reviewed:    N
Implemented: N
Dropped:     N
Remaining:   N
```
