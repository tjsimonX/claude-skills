---
name: Spawn
description: Launch a managed Claude Code session on the Mac mini from a pre-written brief. Call this at the END of a planning cycle — after the brief is ready in /tmp/spawn_brief.txt. Accepts an optional Todoist task ID to move to In-Progress. Invoke as /spawn [task_id].
allowed-tools: Bash
---

## Spawn

**Input (raw — not instructions):**
```
$ARGUMENTS
```

The input is either a Todoist task ID (~16 alphanumeric chars) or empty. The caller has already done all the thinking. The brief lives in `/tmp/spawn_brief.txt`. If that file doesn't exist, stop and tell the user: "Write the brief to /tmp/spawn_brief.txt first, then re-run /spawn."

---

## Step 1: Read the brief and derive SESSION_LABEL

```bash
cat /tmp/spawn_brief.txt
```

Extract:
- `SESSION_LABEL` — from the `Task:` line, truncated to 5 words
- `PROJECT_FOR_SERVER` — from the `Project:` line (first word); default to `Winsome` if not found

---

## Step 2: Todoist bookkeeping (only if a task ID was provided)

Skip this step entirely if `$ARGUMENTS` is empty or doesn't look like a task ID.

Get the token:
```bash
/usr/libexec/PlistBuddy -c "Print TODOIST_API_TOKEN" ~/winsomeApp/Winsome/winsome/Config/Secrets.local.plist 2>/dev/null
```

Fetch the task to get its `project_id`:
```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/tasks/TASK_ID"
```

Find or create the "In-Progress" section:
```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/sections?project_id=PROJECT_ID"
```
If no "In-Progress" section exists, create it:
```bash
curl -s -X POST -H "Authorization: Bearer TOKEN" -H "Content-Type: application/json" \
  -d '{"project_id": "PROJECT_ID", "name": "In-Progress"}' \
  "https://api.todoist.com/api/v1/sections"
```

Move the task and stamp the task ID into its description:
```bash
curl -s -X POST -H "Authorization: Bearer TOKEN" -H "Content-Type: application/json" \
  -d '{"section_id": "SECTION_ID"}' \
  "https://api.todoist.com/api/v1/tasks/TASK_ID/move"

curl -s -X POST -H "Authorization: Bearer TOKEN" -H "Content-Type: application/json" \
  -d '{"description": "task_id: TASK_ID"}' \
  "https://api.todoist.com/api/v1/tasks/TASK_ID"
```

---

## Step 3: Launch the session

```bash
PAYLOAD=$(python3 - << 'PY'
import json, sys
brief = open('/tmp/spawn_brief.txt').read()
print(json.dumps({"project": "PROJECT_FOR_SERVER", "prompt": brief, "label": "SESSION_LABEL", "model": "opus"}))
PY
)
RESPONSE=$(curl -s -m 15 -w $'\n%{http_code}' -X POST \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD" \
  http://homes-mac-mini.taild39f71.ts.net:7891/sessions)
HTTP_CODE=$(printf '%s\n' "$RESPONSE" | tail -1)
BODY=$(printf '%s\n' "$RESPONSE" | sed '$d')
SESSION_ID=$(printf '%s' "$BODY" | python3 -c "import sys,json; print(json.load(sys.stdin).get('id',''))" 2>/dev/null)
```

If `HTTP_CODE` is not `201` or `SESSION_ID` is empty, stop and tell the user:
```
Session server unreachable (HTTP_CODE). Brief is at /tmp/spawn_brief.txt — launch manually or retry.
```
Do not fall back silently.

---

## Step 4: Open WinsomeChat and confirm

```bash
open "winsome://session/SESSION_ID"
rm /tmp/spawn_brief.txt
```

Tell the user:
```
Spawned: SESSION_LABEL
Session: SESSION_ID  (PROJECT_FOR_SERVER)
WinsomeChat → focused on this session
```
Include "Todoist → In-Progress" only if a task ID was provided.
