---
name: Sharingan
description: >
  Perceive the current problem or idea, hunt the internet for cases that exactly
  match or closely rhyme with it, then bring what others learned back and apply it
  to the problem at hand.

  Use when you're stuck, designing something new, or want to know how the world has
  already solved (or failed to solve) the thing in front of you — before reinventing
  it. Invoke as /sharingan [problem or idea; omit to use the current conversation].
---

# Sharingan Skill

The Sharingan perceives a technique, copies its true form, and adapts it. This skill
does the same with problems: see the problem clearly, find where it has already been
solved, copy what worked, adapt it to here.

The failure mode this skill exists to prevent is **solving in a vacuum** — burning
effort on a problem the world has already chewed through. Do not skip the hunt and
answer from memory. The whole point is to go outside and come back with evidence.

**Input:** `$ARGUMENTS` is an optional problem/idea statement. If empty, derive the
problem from the current conversation context (the code being worked on, the last
question asked, the bug being chased).

---

## Phase 1 — Perceive (capture the real problem)

Before searching, get the problem into sharp focus. A blurry problem produces blurry
search queries and useless matches.

1. **State the problem in one or two sentences.** What is actually being attempted or
   blocked? If working from conversation context, read the relevant code/files first so
   the statement is grounded in specifics, not vibes.
2. **Pin the specifics that constrain the answer:** language/framework, platform,
   versions, scale, the exact error or symptom, the constraint that makes the obvious
   answer wrong. These are what separate a matching solution from a misleading one.
3. **Name the underlying shape.** Strip away the surface domain: is this really a
   cache-invalidation problem? A consensus problem? A rate-limit/backoff problem? A
   schema-migration problem? The shape is what lets you find analogous solutions in
   other domains.

Show the user a compact read of the problem before going to the net:

```
Problem:   <one-line statement>
Specifics: <stack / versions / scale / constraints>
Shape:     <the abstract class of problem>
```

If the problem is genuinely ambiguous and the ambiguity changes what you'd search for,
ask one clarifying question. Otherwise proceed.

---

## Phase 2 — Abstract (cast the net at multiple altitudes)

Generate several search queries deliberately spanning two altitudes. Matches hide at
both:

- **Literal / exact-match queries** — the specific error string, the exact API + symptom,
  the precise feature ("`<framework>` <exact symptom>", paste the literal error text).
  These find people in the identical situation.
- **Structural / analogous queries** — the underlying shape from Phase 1, phrased in the
  vocabulary of people who study it ("idempotent retry", "optimistic concurrency",
  "fan-out write amplification"). These find better solutions from adjacent domains that
  the literal search never surfaces.

Write out 4–8 candidate queries before searching. Prefer the canonical name a problem
goes by over your own description of it — half of finding the answer is knowing what the
thing is called.

---

## Phase 3 — Hunt (search and read)

Use `WebSearch` for each altitude of query, then `WebFetch` the most promising sources to
read them properly — search snippets are not enough to copy a technique from.

Prioritize sources in roughly this order:
1. Primary docs, RFCs, specs, official issue trackers
2. Source code / commits / PRs where the fix actually lives
3. Substantive engineering writeups, conference talks, papers
4. Stack Overflow / forum threads with a *verified or well-argued* answer
5. Recent, dated discussion (note publication dates — a 2014 answer may be obsolete)

Hunt rules:
- **Corroborate before trusting.** A single blog post is a lead, not a conclusion. Look
  for a second independent source, especially for anything you'll recommend acting on.
- **Chase the trail.** If a thread points at a root cause or a named pattern, search that
  next. The first result is rarely the deepest.
- **Capture the source.** Keep the URL for every finding you'll cite — Phase 5 reports them.
- **Know when to stop.** Stop when sources start repeating and a clear picture has formed,
  or when it's evident the problem is genuinely novel / unsolved. Don't grind past
  diminishing returns — say so if the net came back light.

---

## Phase 4 — Copy (extract the technique)

For each strong match, extract what actually transfers — not a summary of the page, but
the reusable substance:

- **What they did** — the concrete approach, in enough detail to reimplement.
- **Why it worked** — the mechanism, so you can judge whether it holds under *our*
  constraints.
- **What it cost / where it broke** — tradeoffs, failure modes, "we later regretted X".
  The dead ends others hit are often more valuable than the wins.
- **Fit delta** — how their situation differs from ours (older version, bigger scale,
  different language) and whether that difference invalidates the transfer.

Discard matches that look similar on the surface but fail on a constraint from Phase 1.
Say why you discarded them — a ruled-out approach is a real result.

---

## Phase 5 — Adapt (apply it back here)

This is the payload. Bring the findings home and map them onto *our* specific problem.
A generic "here's what the internet says" is a failure of this skill — the output must be
about the problem in front of us.

Produce:

```
## Sharingan: <problem in one line>

### What the world already knows
- <finding> — <source URL>
- <finding> — <source URL>

### What applies here
<The recommended approach, adapted to our stack/constraints/scale. Concrete: name the
files, functions, params, or steps in *our* context. Note what carries over cleanly and
what needs adjustment.>

### Watch out for
<Failure modes / tradeoffs others hit that we'd be walking into.>

### Confidence
<High / Medium / Low> — <corroboration level and what would raise it. If the hunt came
back thin or the problem looks novel, say so plainly rather than overstating.>
```

Then, if there's an obvious next action in the codebase, offer to take it — but make the
recommendation first; don't auto-implement off web findings without surfacing them.

---

## Principles

- **Evidence over memory.** If you didn't search, you didn't run the skill. Cite sources.
- **Honesty about fit.** The internet's answer was written for someone else's constraints.
  The value you add is judging the delta, not relaying the link.
- **Both altitudes, always.** The exact-match search and the structural-analogy search
  find different things. Skipping the analogy is how you miss the better answer.
- **A null result is a result.** "Nobody seems to have solved this; here's the closest
  prior art and why it doesn't quite reach" is a legitimate, useful outcome.
