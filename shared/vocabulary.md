## Vocabulary

These terms have specific meanings across the Winsome system. Use them consistently.

- **Mining** — a task that needs more work before it's implementation-ready. Lives in the Todoist "Mining" section. Still has open scope, unknown files, or unresolved questions. Processed via `/mine`.
- **Diamond** — an implementation-ready planning note at `Diamonds/[Project]/[Task].md` in the vault. Scope is locked, files identified, decision checklist fully resolved. No open unknowns. Produced by `/diamond` after real investigation. The standard is `Diamonds/SleeperDynastyAgent/Prompt-Cache Consolidation.md`.
- **Observation** — a raw insight or claim captured via Todoist `@observation` label. Routed nightly into `Observations/[Category].md` by `observe.sh`.
- **Protocol** — a structured procedure executed by WinsomeChat. Lives in `Protocols/` in the vault. Executable, not just reference.
- **Anchored phase** — a phase within a multi-phase Protocol whose firing time is computed as an offset (e.g. `1d before start`, `5m before start`, `1h after end`) from the Protocol's anchor event (`anchor_event_contains`). The offsets themselves are not separately named; a Protocol with several anchored phases is just a Protocol with multiple anchored phases, not a "multi-trigger."
