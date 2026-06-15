---
name: Diamond
description: >
  Research and scope a Mining task until it's implementation-ready, then write a
  fully-worked Diamond planning note to Diamonds/[Project]/[Task].md.

  A Diamond is implementation-ready: scope locked, files identified, open questions
  answered, decision checklist resolved. It is not a scaffold — it is a plan you
  can hand to a developer (or Claude) and get working code back.

  Use when /mine surfaces a task too open to implement directly. Invoke as
  /diamond <task description or Todoist task ID>.
---

# Diamond Skill

You are turning a rough Mining task into an implementation-ready Diamond note.
The standard is the `Diamonds/SleeperDynastyAgent/Prompt-Cache Consolidation.md`
note in the vault — measured data, worked examples, specific files and functions,
a decision checklist with no open unknowns.

Do not write the note until the research is done. A Diamond written before
the investigation is just a fancy scaffold.

---

## Phase 1 — Resolve the task

**If `$ARGUMENTS` looks like a Todoist task ID** (alphanumeric, ~16 chars):
```bash
TOKEN=$(security find-generic-password -s "com.terrencesimon.winsome" -a "TODOIST_API_TOKEN" -w 2>/dev/null || \
  /usr/libexec/PlistBuddy -c "Print TODOIST_API_TOKEN" ~/winsomeApp/Winsome/winsome/Config/Secrets.local.plist 2>/dev/null)
curl -s -H "Authorization: Bearer $TOKEN" "https://api.todoist.com/api/v1/tasks/TASK_ID"
```
Extract the task title and description. Also fetch the project name to determine where the Diamond lives.

**If `$ARGUMENTS` is a free-form description**, use it directly as the task title and ask the user which project it belongs to.

---

## Phase 2 — Find the project directory

Read `winsomeTerminal/config.json` — `machines.local.routes` maps project names to local paths. Match the project name and resolve the local directory.

If no match, ask the user for the project path directly.

---

## Phase 3 — Investigate

This is the core of the skill. Read source files, measure things, trace the call chain. Don't guess — find out.

For each task, the investigation should answer:

**Scope**
- What exactly changes? Which files, which functions, which data flows?
- Is this a move, a rewrite, a new feature, or a bug fix?
- What is the minimum set of changes to ship it?

**Evidence**
- What is the current behavior? (Read the code, run commands, check logs)
- What data supports the change? (Token counts, timing, error rates, API responses)
- Are there existing tests? What would break?

**Risk**
- What are the call sites? How many places does this touch?
- Are there edge cases that could regress?
- What does rollback look like if something goes wrong?

**Decision points**
- What is the one open question that would change the implementation? Answer it.
- If there are multiple valid approaches, pick one and explain why.

Read files liberally. Run `grep`, `find`, `wc`. Check git log for context. Use every tool available. The investigation is done when there are no open unknowns in the decision checklist.

---

## Phase 4 — Write the Diamond

Vault path: `~/Library/Mobile Documents/com~apple~CloudDocs/winsomeVault`
Diamond path: `Diamonds/[ProjectName]/[TaskName].md`

Create the project subfolder if it doesn't exist. Use the task title as the filename (spaces are fine; strip special characters).

Write the note. Every section should contain real findings from Phase 3 — no placeholders, no "TBD". If something is genuinely unknown after investigation, that's a blocker: say so explicitly and don't ship the Diamond until it's resolved.

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

> [One-line summary: what this implements and why — make it a claim, not a topic]

---

## Background

[Why this matters. What problem it solves. Historical context if relevant.]

---

## Current State

[How the system works today. Specific files, functions, data structures.
Include measured data where relevant — token counts, timing, error rates.]

---

## The Problem

[What specifically is wrong or missing. Be precise — reference line numbers,
function names, data shapes.]

---

## Proposed Implementation

[The specific change. Design decisions already made. Why this approach over alternatives.]

### Files to touch

- `path/to/file.swift` — [what changes and why]
- ...

### Call sites / ripple effects

- [anything downstream that needs updating]

---

## Decision Checklist

- [x] [Question that was answered during investigation — with the answer]
- [x] [Another resolved question]
- [ ] Confirm scope with user before implementing (if above the ask-first bar in CLAUDE.md)

---

## Related

- Todoist: [Project name] → Mining
- [Any related files, PRs, or notes]
```

---

## Phase 5 — Report

Tell the user:
- Where the Diamond was written (`Diamonds/[Project]/[Task].md`)
- A 2-3 sentence summary of what you found and what the implementation involves
- Whether anything came up during investigation that changes the scope or priority

Do not close the Todoist task — the task stays in Mining until the user decides to implement.
