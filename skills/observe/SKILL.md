---
name: Observe
description: Farm Todoist tasks labeled "observation", sort them into categories in Observations.md in the vault, and close the tasks. Invoke as /observe.
disable-model-invocation: true
allowed-tools: Bash
---

## Observe

Pull all Todoist tasks labeled `observation`, sort them into the Observations vault note by category, and close each one.

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
If both fail, tell the user the token is missing and stop. Use the token as `TOKEN`.

---

## Step 2: Fetch observation tasks

```bash
curl -s -G \
  -H "Authorization: Bearer TOKEN" \
  --data-urlencode "filter=@observation" \
  "https://api.todoist.com/api/v1/tasks"
```

If no tasks are returned, tell the user: "No tasks labeled 'observation' found." and stop.

---

## Step 3: Sorting loop

Observations are split into per-category files inside the vault:

| Category | File |
|---|---|
| Power & Systems | `Observations/Power and Systems.md` |
| Culture & Social Dynamics | `Observations/Culture and Social Dynamics.md` |
| Cognition & Learning | `Observations/Cognition and Learning.md` |
| Strategy & Execution | `Observations/Strategy and Execution.md` |
| Geography, Cities, and Density | `Observations/Geography Cities and Density.md` |
| Technology & Infrastructure | `Observations/Technology and Infrastructure.md` |
| Philosophy & Meta Principles | `Observations/Philosophy and Meta Principles.md` |

Open with: **"X observation tasks to sort. Let's go."**

For each task in order:

### Present

Show:
- **Title** (the task content)
- Description/notes if any (from the `description` field)
- Item number: **"Item N of X"**

### Pick a category and file it

Pick the best-fit category from the 7 above — no confirmation needed, just decide and act. Then immediately append to the correct category file.

Use this Python snippet. Replace `CATEGORY_FILE` with the relative path (e.g. `Observations/Power and Systems.md`) and `OBSERVATION_TEXT` with the task content:

```bash
python3 << 'PYEOF'
import os
VAULT_DIR = os.path.expanduser('~/Library/Mobile Documents/com~apple~CloudDocs/winsomeVault')
FILE_PATH = os.path.join(VAULT_DIR, 'CATEGORY_FILE')
TEXT = 'OBSERVATION_TEXT'

with open(FILE_PATH, 'rb') as f:
    content = f.read().decode('utf-8')

bullet = f'- {TEXT}'
if not content.endswith('\n'):
    content += '\n'
content += bullet + '\n'

with open(FILE_PATH, 'wb') as f:
    f.write(content.encode('utf-8'))
print('OK')
PYEOF
```

If the script errors, tell the user what went wrong and don't close the task.

### Close the task

```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  "https://api.todoist.com/api/v1/tasks/TASK_ID/close"
```

Confirm with a single line: "✓ [Category] — [task title]." Then move immediately to the next item.

---

## End of session

When all items are processed or the user says stop:

```
Observe session complete.

Filed:   N
Dropped: N
Remaining: N
```
