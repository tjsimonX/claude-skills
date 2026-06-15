---
name: Dev Review
description: Process unsectioned tasks in a Dev project one by one — implement (close), mine (move to Mining), or drop. Invoke as /dev-review <ProjectName>.
allowed-tools: Bash
---

## Dev Review

**Project:** $ARGUMENTS

If `$ARGUMENTS` is empty, tell the user: "Specify a project name — e.g. `/dev-review Winsome`" and stop.

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

## Step 3: Find or create the Mining section

```bash
SCRIPT=~/winsomeApp/Winsome/scripts/sweep_fetch.sh
SECTION_ROWS=$(bash "$SCRIPT" sections PROJECT_ID TOKEN)
```

`SECTION_ROWS`: one line per section — `SECTION_ID|NAME`. Parse by splitting on `|`.

Find the row whose name matches "Mining" (case-insensitive). Store its id as `MINING_SECTION_ID`.

If no Mining section exists, create it:
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"project_id": "PROJECT_ID", "name": "Mining"}' \
  "https://api.todoist.com/api/v1/sections"
```
Store the returned `id` as `MINING_SECTION_ID`.

---

## Step 4: Fetch unsectioned tasks

```bash
TASK_ROWS=$(bash "$SCRIPT" tasks PROJECT_ID TOKEN)
```

`TASK_ROWS`: one line per qualifying task — `TASK_ID|TITLE|DESCRIPTION` (description ≤100 chars). Pre-filtered: no section, no due date, not starting with `*`.

Parse each line by splitting on `|` to get `task_id`, `title`, `desc`.

If `TASK_ROWS` is empty, tell the user: "No unsectioned tasks in [project] — all clear." and stop.

---

## Step 5: Review loop

Open with: **"X items to review in [project]. Let's go."**

For each task in order:

### Present
- **Title** + description (if any)
- "**Item N of X**"

### Give a take
1–2 direct sentences: is this ready to build now, or does it need more thinking first? Don't hedge.

### Wait for decision

| Input | Action |
|-------|--------|
| "implement", "yes", "do it", "ship", "i" | Close the task |
| "mine", "m", "later", "skip" | Move to Mining |
| "drop", "delete", "no", "d" | Delete the task |
| "stop", "done", "quit" | End the session |

### Act

**Implement:**
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  "https://api.todoist.com/api/v1/tasks/TASK_ID/close"
```
Confirm: "✓ Implementing."

**Mine:**
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"section_id": "MINING_SECTION_ID"}' \
  "https://api.todoist.com/api/v1/tasks/TASK_ID/move"
```
Confirm: "→ Mining."

**Drop:**
```bash
curl -s -X DELETE \
  -H "Authorization: Bearer TOKEN" \
  "https://api.todoist.com/api/v1/tasks/TASK_ID"
```
Confirm: "✗ Dropped."

After acting, immediately present the next task — no pause.

---

## End of session

```
Dev review complete.

Reviewed:    N
Implementing: N
Mined:       N
Dropped:     N
```

If items remain (user said stop early): "X tasks still unsectioned."
