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

The Observations vault note lives at:
```
/Users/home/Library/Mobile Documents/com~apple~CloudDocs/winsomeVault/Observations 👀.md
```

The defined categories (section headers in the file) are:
1. Power & Systems
2. Culture & Social Dynamics
3. Cognition & Learning
4. Strategy & Execution
5. Geography, Cities, and Density
6. Technology & Infrastructure
7. Philosophy & Meta Principles

Open with: **"X observation tasks to sort. Let's go."**

For each task in order:

### Present

Show:
- **Title** (the task content)
- Description/notes if any (from the `description` field)
- Item number: **"Item N of X"**

### Pick a category and file it

Pick the best-fit category from the 7 above — no confirmation needed, just decide and act. Then immediately append to the vault file.

Use this Python snippet to append the observation as a bullet under the correct section header. Replace `CATEGORY` with the exact section title (e.g. `Power & Systems`) and `OBSERVATION_TEXT` with the task content:

```bash
python3 << 'PYEOF'
VAULT = '/Users/home/Library/Mobile Documents/com~apple~CloudDocs/winsomeVault/Observations \U0001f440.md'
CATEGORY = 'CATEGORY'
TEXT = 'OBSERVATION_TEXT'

with open(VAULT, 'rb') as f:
    content = f.read().decode('utf-8')

lines = content.split('\n')

# Find the section header
header_idx = None
for i, line in enumerate(lines):
    stripped = line.lstrip('#').strip()
    if stripped.lower() == CATEGORY.lower():
        header_idx = i
        break

if header_idx is None:
    print(f'ERROR: section "{CATEGORY}" not found')
else:
    # Find insertion point: last non-empty line before the next section header
    insert_at = header_idx + 1
    for i in range(header_idx + 1, len(lines)):
        if lines[i].startswith('#'):
            break
        if lines[i].strip():
            insert_at = i + 1

    lines.insert(insert_at, f'* {TEXT}')
    with open(VAULT, 'wb') as f:
        f.write('\n'.join(lines).encode('utf-8'))
    print('OK')
PYEOF
```

If the script prints `ERROR`, tell the user what went wrong and don't close the task.

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
