# 4 — Gameplay‑specific systems

This page maps the **gameplay systems** that sit on top of the core AI architecture described in Phase 3.

## 4.1 — Pathfinding / Movement

**Role**
- Handles movement commands, path choices, follow behavior, and special movement logic (BG paths, raid mechanics).

**Key components**
- **Movement actions** under `src/Ai/Base/Actions/MovementActions.*` and specialized movement actions in raid files.  
- **Travel system**: `TravelMgr` (`src/Mgr/Travel/`) and `TravelTarget` values (`"travel target"` value in AI context).  
- **BG movement**: `BattleGroundTactics` (`src/Ai/Base/Actions/BattleGroundTactics.cpp`) with prebuilt paths.

**How it works (high level)**
1. Strategies enqueue movement actions (e.g., follow, move to, flee, BG pathing).
2. Movement actions call helpers like `MoveTo` / `MoveNear`.
3. `TravelMgr` provides destinations for RPG/quest travel and updates `TravelTarget`.

**Extension points**
- Add new movement behaviors as `MovementAction` subclasses.
- Register new travel destinations or logic in `TravelMgr`.
- Extend BG pathing by adding paths/waypoints in `BattleGroundTactics`.

**Risks / pitfalls**
- Movement is sensitive to world state; avoid forcing movement when a bot is already in flight, teleporting, or logging out.
- Prefer existing movement helpers and flags rather than manual movement flag hacks.

## 4.2 — Combat (melee / ranged / heal)

**Role**
- Defines the combat decision‑making for different classes, roles, and situations.

**Key components**
- **Class strategies** under `src/Ai/Class/*/Strategy/` (combat logic per class/spec).
- **Default engine composition** in `AiFactory` (`src/Bot/Factory/AiFactory.cpp`).
- **Core combat actions** under `src/Ai/Base/Actions/` (generic attacks, interrupts, buffs).

**How it works (high level)**
1. `AiFactory` builds the combat engine and attaches default strategies.
2. Class strategies define triggers and default actions (e.g., warrior DPS rotation).
3. The engine selects the highest‑priority action and executes it.

**Extension points**
- Add or adjust class strategies for new rotations or smarter responses.
- Add new combat actions and expose them via the AI context.

**Risks / pitfalls**
- Avoid duplicating existing combat checks (use shared helpers and values).
- Keep action relevance balanced to prevent “spammy” or unsafe actions.

## 4.3 — Group / Raid logic

**Role**
- Coordinates bots in groups and raids, including raid‑specific boss logic.

**Key components**
- **Raid strategies** and **raid action contexts** under `src/Ai/Raid/`.
- **Raid trigger/action pairs** for encounter logic (e.g., ICC, Magtheridon).
- **Group behaviors** in base strategies (assist, follow, heal, tank).

**How it works (high level)**
1. Raid strategy contexts register encounter strategies.
2. Encounter strategies add triggers tied to boss mechanics.
3. Actions implement positioning, target selection, and special mechanics.

**Extension points**
- Add new raid strategies and register them in `RaidStrategyContext`.
- Add encounter actions/triggers in the relevant raid folder.

**Risks / pitfalls**
- Raid logic often relies on movement timing; keep movement changes minimal and tested.
- Avoid hard‑coding actions that conflict with generic combat logic.

## 4.4 — LFG / BG / Arena

**Role**
- Manages bots in group finder, battlegrounds, and arena contexts.

**Key components**
- **RandomPlayerbotMgr** queue checks: BG/LFG periodic checks and join logic.  
- **BattleGroundTactics** for BG pathing and objectives.
- **Arena strategies** via combat/non‑combat engine strategy switches.

**How it works (high level)**
1. `RandomPlayerbotMgr::UpdateAIInternal` schedules BG/LFG checks.
2. BG logic uses path tables and objective selection in `BattleGroundTactics`.
3. Arena adds/adjusts strategies via `AiFactory` (arena strategy flags).

**Extension points**
- Add/adjust BG tactics (paths, flags, objectives).
- Extend queue logic and scheduling in `RandomPlayerbotMgr`.

**Risks / pitfalls**
- BG paths must exist for the target BG; missing paths can cause movement failures.
- Arena strategy tuning should stay consistent with core combat expectations.

## 4.5 — Random bot RPG logic

**Role**
- Drives autonomous “life‑like” behavior: travel, grinding, questing, and roaming.

**Key components**
- **RPG strategies** in `src/Ai/World/Rpg/`.
- **TravelMgr** and travel targets for destination planning.
- **RandomPlayerbotMgr** randomization and teleport helpers.

**How it works (high level)**
1. Random bots select an RPG strategy (classic RPG or new RPG).
2. Travel targets are chosen and movement actions are issued to reach them.
3. Random bot lifecycle updates (login/refresh/teleport) keep population active.

**Extension points**
- Add new RPG actions or triggers (e.g., smarter roam, safer grinding).
- Add or improve travel destinations and selection criteria.

**Risks / pitfalls**
- Avoid heavy computations every tick; prefer cached values with intervals.
- Respect existing random bot scheduling to avoid server load spikes.