# Claude Skills

Personal Claude Code skills. Each skill lives in `skills/<name>/SKILL.md` and is invoked as `/<name>` inside a Claude Code session.

## Skills

### `/sweep`
Sweeps every Todoist project for tasks with no section and no due date. Dev sub-projects invoke `/roadmap-review` for each. All other projects (including the Dev parent) use a dynamic flow — reads each project's actual sections and routes tasks into them.

### `/mine <ProjectName>`
Refines raw ideas from a project's Todoist Mining section one by one — discusses each until it's concrete enough to implement, then promotes to Roadmap or drops.

### `/observe`
Farms Todoist tasks labeled `observation`, sorts them into categories in `Observations.md` in the vault, and closes the tasks.

### `/roadmap-review <ProjectName>`
Cycles through tasks in the Roadmap section of a Todoist project. For each task, gives a recommendation and waits for the user to decide: implement (close), drop (delete), or skip to Mining. Uses reasoning to assess complexity and prioritization.

### `/strategic-roadmap`
Turns a raw vision or North Star idea into a complete, multi-year personal strategic roadmap. Extracts the vision, builds a named pillar framework (3–5 positions), researches intellectual lineage and failure modes, grounds the vision physically, maps business/career opportunities by tier, builds a layered people/access strategy, defines a mentor filter, generates daily orientation questions, and produces two polished HTML deliverables — a daily brief and a full navigable plan.

## When to use a skill vs a shell script

**Skill** — task requires reasoning: presenting options, making judgment calls, looping where each step depends on a human decision.

**Shell script** — steps are mechanical and deterministic, no decisions required. A skill with no reasoning is just a slow, expensive shell script.
