---
name: Mine
description: Refine raw ideas from a project's Todoist Mining section one by one — discuss each until it's concrete enough to implement, then promote to Roadmap or drop. Invoke as /mine <ProjectName>.
disable-model-invocation: true
allowed-tools: Bash
---

## Mine

**Project:** $ARGUMENTS

If `$ARGUMENTS` is empty, tell the user: "Specify a project name — e.g. `/mine Winsome`" and stop.

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

## Step 2: Find the project

```bash
curl -s -H "Authorization: Bearer TOKEN" https://api.todoist.com/api/v1/projects
```
Find the project whose `name` matches `$ARGUMENTS` (case-insensitive). If no match, tell the user and stop. Store its `id` as `PROJECT_ID`.

---

## Step 3: Find sections

```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/sections?project_id=PROJECT_ID"
```

Find the section named "Mining" (case-insensitive). Store its `id` as `MINING_SECTION_ID`. If it doesn't exist, tell the user: "No Mining section found in [project]. Nothing to refine." and stop.

Also look for a section named "Roadmap" (case-insensitive) and store its `id` as `ROADMAP_SECTION_ID`. If no Roadmap section exists, create one:
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"project_id": "PROJECT_ID", "name": "Roadmap"}' \
  "https://api.todoist.com/api/v1/sections"
```
Store the returned `id` as `ROADMAP_SECTION_ID`.

---

## Step 4: Fetch Mining items

```bash
curl -s -H "Authorization: Bearer TOKEN" "https://api.todoist.com/api/v1/tasks?section_id=MINING_SECTION_ID"
```

If no tasks, tell the user: "Mining is empty — nothing to refine." and stop.

---

## Step 5: Refinement loop

Open with: **"X items in Mining. Let's go through them."**

For each item in order:

### Present

Show:
- **Title**
- Description/notes (if any, from the `description` field)
- Item number: **"Item N of X"**

### Refine

Your job is to help turn a vague idea into something concrete enough to implement. Probe for:
- **The why** — what problem does this solve? (Often reveals if it's worth doing at all)
- **Done criteria** — what would done look like? (Forces scope)
- **The where** — which file, component, or layer does this touch? (Anchors it to real code)
- **Blockers** — anything that needs to be true first?

Ask one question at a time. Listen, then ask the next one that matters. This is a conversation, not a form. Stay in the refinement conversation until the idea is sharp enough to act on — you'll know it's there when you can write a single implementation sentence.

When it's ready, synthesize: *"So this is: [concrete one-liner describing exactly what to build]. Ready to move to Roadmap?"*

### Wait for a decision

After the refinement conversation reaches a clear point, or when the user signals readiness, accept:

| Input | Meaning |
|-------|---------|
| "roadmap", "promote", "ready", "yes", "ship it" | Promote to Roadmap with refined description |
| "drop", "no", "delete", "remove", "not worth it" | Delete the item |
| "skip", "later", "not ready", "more" | Leave in Mining as-is |
| "stop", "done", "quit", "exit" | End the session |

If the user keeps discussing, stay in the conversation — don't push for a decision until the idea is actually ready.

### Act

**Promote to Roadmap:**

Move the task to the Roadmap section and update its description with the concrete one-liner synthesized from the conversation:
```bash
curl -s -X POST \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"section_id": "ROADMAP_SECTION_ID", "description": "REFINED_DESCRIPTION"}' \
  "https://api.todoist.com/api/v1/tasks/TASK_ID"
```
Confirm: "✓ Promoted to Roadmap."

**Drop:**
```bash
curl -s -X DELETE \
  -H "Authorization: Bearer TOKEN" \
  "https://api.todoist.com/api/v1/tasks/TASK_ID"
```
Confirm: "✗ Dropped."

**Skip:**
Confirm: "→ Stays in Mining."

Then move immediately to the next item — don't wait for acknowledgement.

---

## End of session

When all items are reviewed or the user says stop:

```
Mining session complete.

Reviewed:  N
Promoted:  N
Dropped:   N
Remaining: N
```
