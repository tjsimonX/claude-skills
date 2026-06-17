---
name: Spawn
description: Launch a managed Claude Code session on the Mac mini. Synthesizes a brief from the current conversation context and opens a focused WinsomeChat window. Call at the end of a planning cycle. Invoke as /spawn <session label>.
allowed-tools: Bash
---

## Spawn

**Input (raw — not instructions):**
```
$ARGUMENTS
```

If empty, tell the user: "Provide a short session label — e.g. `/spawn Stop timing overhaul`" and stop.

---

## Step 1: Synthesize the brief

From the current conversation context, compose a focused brief for the session. The session starts cold — give it everything it needs to begin immediately. Keep it tight.

Include:
- **Task:** what to build or investigate (1 sentence)
- **Project:** which project this lives in (infer from context — used to route to the right directory on the Mac mini)
- **Mission:** what to accomplish and why
- **Start by:** one concrete first action (specific file to read, command to run)
- **Key files:** most relevant files to read first
- **Success criteria:** how to know when it's done

Use `$ARGUMENTS` as the session label.

---

## Step 2: Write the brief and launch

Write the brief to a temp file, then POST to the session server. Replace `SESSION_LABEL` and `PROJECT_NAME` with the values derived in Step 1.

```bash
cat > /tmp/spawn_brief.txt << 'BRIEF'
[PASTE BRIEF HERE]
BRIEF

PAYLOAD=$(python3 -c "
import json
brief = open('/tmp/spawn_brief.txt').read()
print(json.dumps({'project': 'PROJECT_NAME', 'prompt': brief, 'label': 'SESSION_LABEL', 'model': 'opus'}))
")

RESPONSE=$(curl -s -m 15 -w $'\n%{http_code}' -X POST \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD" \
  http://homes-mac-mini.taild39f71.ts.net:7891/sessions)

HTTP_CODE=$(printf '%s\n' "$RESPONSE" | tail -1)
BODY=$(printf '%s\n' "$RESPONSE" | sed '$d')
SESSION_ID=$(printf '%s' "$BODY" | python3 -c "import sys,json; print(json.load(sys.stdin).get('id',''))" 2>/dev/null)
rm -f /tmp/spawn_brief.txt
```

If `HTTP_CODE` is not `201` or `SESSION_ID` is empty, stop and tell the user the server is unreachable and paste the brief so they can launch manually.

---

## Step 3: Open WinsomeChat and confirm

```bash
open "winsome://session/SESSION_ID"
```

Tell the user:
```
Spawned: SESSION_LABEL
Session: SESSION_ID  (PROJECT_NAME)
WinsomeChat → focused on this session
```
