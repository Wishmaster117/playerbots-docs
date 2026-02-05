# Phase 1 — High-level overview (1–2 days)

**Deliverable:** a macro diagram + introduction page.

## 1.A — Macro component map

- `PlayerbotAI`: decision core.
- `PlayerbotMgr`: bots linked to a master player.
- `RandomPlayerbotMgr`: autonomous bots.
- `PlayerbotAIConfig`: configuration.

## 1.B — Roles and interactions

> This section is the first concrete deliverable: a written macro map that can later be converted into a diagram.

### Core components (entry points)

- **PlayerbotAI** — main decision engine for a bot instance.  
  Location: `src/Bot/PlayerbotAI.h` / `src/Bot/PlayerbotAI.cpp`.
- **PlayerbotMgr** — manages bots linked to a master player (commands, packets, login/logout).  
  Location: `src/Bot/PlayerbotMgr.h` / `src/Bot/PlayerbotMgr.cpp`.
- **RandomPlayerbotMgr** — manages autonomous/random bots (lifecycle, scheduling, stats).  
  Location: `src/Bot/RandomPlayerbotMgr.h` / `src/Bot/RandomPlayerbotMgr.cpp`.
- **PlayerbotAIConfig** — centralized configuration and tunables for bot behavior.  
  Location: `src/PlayerbotAIConfig.h` / `src/PlayerbotAIConfig.cpp`.

### High-level responsibilities

- **AI decision flow:** `PlayerbotAI` orchestrates decision-making and behavior for a single bot.
- **Master-bound bots:** `PlayerbotMgr` handles player-controlled bots and their command interface.
- **Random bots:** `RandomPlayerbotMgr` handles autonomous bots and their lifecycle policies.
- **Configuration:** `PlayerbotAIConfig` provides shared settings and timing values.

### Directory overview (macro)

- `src/Bot/` — core bot AI, managers, and command handling.
- `src/Ai/` — AI logic (strategies, actions, triggers, class-specific behavior).
- `src/Mgr/` — manager helpers and cross‑cutting systems.
- `src/Db/` — data access and persistence logic.
- `src/Util/` — shared utilities.

### Interaction sketch (textual)

1. **Bot login** enters via `PlayerbotMgr` or `RandomPlayerbotMgr`.
2. A **PlayerbotAI** instance is associated with the bot player.
3. **Update cycles** call into AI decisions.
4. **Configuration** values from `PlayerbotAIConfig` guide timing and behaviors.

## 1.C — Notes for diagram (confirmed from code)

- **AI engine construction & strategy injection**
  - `PlayerbotAI::PlayerbotAI(Player* bot)` creates the **AI context** via
    `AiFactory::createAiObjectContext`, then builds the **combat**, **non-combat**, and **dead**
    engines with `AiFactory::createCombatEngine`, `createNonCombatEngine`, and `createDeadEngine`.
  - Each engine is built by `AiFactory` where default strategies are attached
    (`AddDefaultCombatStrategies`, `AddDefaultNonCombatStrategies`, `AddDefaultDeadStrategies`),
    then `Engine::Init()` finalizes the setup.
- **Update tick entry point**
  - `PlayerbotAI::UpdateAI` is the per-bot tick function; it gates updates (state checks, cheats),
    then calls `UpdateAIInternal`.
  - `PlayerbotAI::UpdateAIInternal` handles queued packets, commands, and ends with
    `DoNextAction(minimal)` which drives the action selection/dispatch.
  - Manager ticks: `PlayerbotMgr::UpdateAIInternal` runs error notification timers for master-bound bots,
    while `RandomPlayerbotMgr::UpdateAIInternal` manages random bot lifecycle, counts, and scheduling.
- **Random bot data stores (state/queues/caches)**
  - **Battle/queue tracking:** `BattlegroundData`, `VisualBots`, `Supporters`, `LfgDungeons`.
  - **Caching:** `BattleMastersCache`, `eventCache`, `rpgLocsCacheLevel`, `zone2LevelBracket`,
    `locsPerLevelCache`, `allianceStarterPerLevelCache`, `hordeStarterPerLevelCache`,
    `bankerLocsPerLevelCache`, `addclassCache`.
  - **Runtime lists:** `players`, `currentBots`, plus inherited `playerBots` from `PlayerbotHolder`.