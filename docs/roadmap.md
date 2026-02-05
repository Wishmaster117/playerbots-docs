# Architecture Plan Roadmap

This page tracks the remaining phases and expected outputs.

## Phase 3 — AI architecture (1–2 weeks)

**Deliverable:** a detailed map of AI modules and extension points.

1. **Context / Engine**
2. **Strategies**
3. **Actions**
4. **Triggers**
5. **Values**

Goal: explain how a behavior is assembled, with one concrete example.

## Phase 4 — Gameplay-specific systems (2–4 weeks)

**Deliverable:** dedicated sections for complex systems.

- Pathfinding / Movement
- Combat (melee/ranged/heal)
- Group / Raid logic
- LFG / BG / Arena
- Random bot RPG logic

## Phase 5 — Data & configuration architecture (1 week)

**Deliverable:** a complete mapping of configs & data.

- `playerbots.conf` and key variables
- DB tables used by the module
- Runtime data (caches, flags, transient state)

## Phase 6 — Quick contribution guide (1 week)

**Deliverable:** a "first patch" guide.

1. Where to make a safe small change
2. Where **not** to modify first
3. How to test locally
4. "PR-ready" checklist