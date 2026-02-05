# Phase 2 — Major execution flows (2–4 days)

**Deliverable:** 3–5 sequence diagrams (or pseudo-sequences).

## 2.1 — AI tick (UpdateAI → UpdateAIInternal → DoNextAction)

**Goal:** describe the per-bot execution loop that drives decisions and actions.

**Entry point**
- `PlayerbotAI::UpdateAI(uint32 elapsed, bool minimal)` is called each tick for a bot.

**Flow (text sequence)**
1. `PlayerbotAI::UpdateAI` applies timing gates and early exits (session/world/teleport/logging out).
2. It applies cheat-driven state fixes (health/power) when enabled.
3. It validates `CanUpdateAI()` and handles current spell checks (interrupts, facing, transport checks).
4. It calls `UpdateAIInternal(elapsed, minimal)`.
5. `UpdateAIInternal` processes chat replies and pending commands.
6. It handles logout logic (session conditions, master checks) and exits if needed.
7. It processes packet queues (bot outgoing, master incoming/outgoing).
8. It ends with `DoNextAction(minimal)` which delegates to the engine to pick/execute actions.

**Key notes**
- `UpdateAIInternal` is the main dispatcher for **command handling** and **action selection**.
- `DoNextAction` is where the current `Engine` drives the next action.

## 2.2 — Next flows to document

1. **Bot login**
2. **Player command → bot**
3. **Random bot lifecycle (spawn, behavior, logout)**