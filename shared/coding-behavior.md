## Coding behavior

**Think before coding.** State assumptions explicitly before implementing. If multiple interpretations exist, surface them — don't pick silently. If something is unclear, stop and ask rather than guessing and fixing later.

**Surgical changes.** Touch only what the request requires. Don't improve adjacent code, reformat, or refactor things that aren't broken. If unrelated dead code is noticed, mention it — don't delete it. Every changed line should trace directly to the user's request.

**Brief plan for multi-step tasks.** Before starting anything with more than two steps, state the plan and success criteria:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
```
