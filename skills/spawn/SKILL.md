---
name: Spawn
description: Spin up a context-aware Claude Code session in iTerm2 for a task. Accepts a Todoist task ID or a free-form description. Loads the right project directory, reads CLAUDE.md for context, crafts a focused brief, moves the task to In-Progress, and launches the session. Invoke as /spawn <task_id_or_description>.
allowed-tools: Bash
---

## Spawn

**Input:** $ARGUMENTS

If `$ARGUMENTS` is empty, tell the user: "Provide a Todoist task ID or a description — e.g. `/spawn 6gqgWvwgQgv9vq6j` or `/spawn Build a Booksy automation`" and stop.

---

## Step 1: Get the Todoist token

```bash
security find-generic-password -s "com.terrencesimon.winsome" -a "TODOIST_API_TOKEN" -w 2>/dev/null
```
If empty, fall back to:
```bash
/usr/libexec/PlistBuddy -c "Print TODOIST_API_TOKEN" ~/winsomeApp/Winsome/winsome/Config/Secrets.local.plist 2>/dev/null
```
If both fail, set TOKEN to empty and skip Todoist steps. Use the token as `TOKEN`.

---

## Step 2: Resolve the task

If `$ARGUMENTS` looks like a Todoist task ID (alphanumeric, ~16 chars, no spaces), fetch the task:

```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/tasks/TASK_ID"
```

Extract:
- `TASK_CONTENT` — the task title
- `TASK_DESCRIPTION` — the task description (may be empty)
- `PROJECT_ID` — the project this task belongs to

Then fetch the project name:
```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/projects/PROJECT_ID"
```
Store as `PROJECT_NAME`.

If `$ARGUMENTS` is free-form text (has spaces or doesn't look like an ID), use it directly as `TASK_CONTENT`. Set `PROJECT_ID` and `PROJECT_NAME` to empty.

---

## Step 3: Map project to a local directory

Read the project routes from:
```bash
cat ~/winsomeApp/Winsome/winsomeTerminal/config.json
```

Match `PROJECT_NAME` against the keys in `machines.server.routes` (case-insensitive). Store the matched path as `PROJECT_PATH`, expanding `~` to the full home directory.

If no match, use `~/winsomeApp/Winsome` as the default and note it in the brief.

---

## Step 4: Read project context

If `PROJECT_PATH` exists, check for a CLAUDE.md:
```bash
head -40 PROJECT_PATH/CLAUDE.md 2>/dev/null
```

Extract a one-liner summary of what the project is and does. Store as `PROJECT_CONTEXT`. If no CLAUDE.md, leave it empty.

---

## Step 5: Craft the brief

Write a focused brief to `/tmp/spawn_brief.txt`. The brief must give the spawned session everything it needs to start immediately — no clarifying questions needed for the basics.

Structure:
```
Task: TASK_CONTENT
Todoist Task ID: TASK_ID
Project: PROJECT_NAME (PROJECT_PATH)
Context: PROJECT_CONTEXT

Description: TASK_DESCRIPTION   ← omit this line if description is empty

Mission: [1-2 sentences stating exactly what to build or investigate, derived from the task content and description. Be specific — name the tool, API, file, or behavior involved.]

Start by [one concrete first step — e.g. "researching what the Copilot Money app API exposes" or "reading the existing dispatch.py to understand the routing logic"].

---
When this task is complete:
1. If this involved a running app or server-side feature, invoke /verify to confirm the change works before closing.
2. Close it in Todoist:
  TOKEN=$(/usr/libexec/PlistBuddy -c "Print TODOIST_API_TOKEN" ~/winsomeApp/Winsome/winsome/Config/Secrets.local.plist 2>/dev/null)
  curl -s -X POST -H "Authorization: Bearer $TOKEN" "https://api.todoist.com/api/v1/tasks/TASK_ID/close"
```

Write this to `/tmp/spawn_brief.txt`:
```bash
cat > /tmp/spawn_brief.txt << 'BRIEF'
[brief content here]
BRIEF
```

---

## Step 6: Move task to In-Progress

Only if a `PROJECT_ID` was resolved:

Fetch sections for the project:
```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/sections?project_id=PROJECT_ID"
```

Find a section named "In-Progress" (case-insensitive). If none, create it:
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"project_id": "PROJECT_ID", "name": "In-Progress"}' \
  "https://api.todoist.com/api/v1/sections"
```

Move the task and stamp the task ID into its description so it's self-documenting in Todoist:
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"section_id": "IN_PROGRESS_SECTION_ID"}' \
  "https://api.todoist.com/api/v1/tasks/TASK_ID/move"
```

Then update the task description to include the task ID (preserve any existing description):
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"description": "task_id: TASK_ID\nEXISTING_DESCRIPTION"}' \
  "https://api.todoist.com/api/v1/tasks/TASK_ID"
```
If there is no existing description, use just `"task_id: TASK_ID"`.

---

## Step 7: Launch the session

Open a new iTerm2 window, `cd` to the project directory, and start Claude Code with the brief as the opening prompt:

```bash
osascript << EOF
tell application "iTerm2"
  create window with default profile
  tell current window
    tell current session
      write text "cd PROJECT_PATH && claude --model opus --dangerously-skip-permissions \"$(cat /tmp/spawn_brief.txt)\""
    end tell
  end tell
end tell
EOF
```

---

## Step 8: Confirm

Tell the user:
```
Spawned: TASK_CONTENT
Project: PROJECT_NAME → PROJECT_PATH
Todoist: → In-Progress
```

If called from within /sweep, resume the sweep at the next task immediately.
