# Module map & glossary (Phase 0)

**Deliverable:** a documentation outline and an initial glossary.

## Module map (folders & roles)

| Folder | Role |
| --- | --- |
| `src/Ai/` | AI logic: strategies, actions, triggers, values, class/dungeon/raid/world specializations. |
| `src/Bot/` | Bot core: `PlayerbotAI`, bot managers, engine factories, command handling. |
| `src/Db/` | Data access and persistence logic for playerbots. |
| `src/Mgr/` | Managers and cross‑cutting systems (services used across bot subsystems). |
| `src/Script/` | Script bindings/integration points with AzerothCore. |
| `src/Util/` | Shared utility helpers used across the module. |
| `src/PlayerbotAIConfig.*` | Central configuration and tunables for bot behavior. |

### `src/Ai/` (subfolders)

- `Base/` — shared base AI logic (actions, triggers, values).
- `Class/` — class-specific AI logic.
- `Dungeon/` — dungeon-specific strategies/triggers/actions.
- `Raid/` — raid-specific strategies/triggers/actions.
- `World/` — world/overworld behavior logic.

### `src/Bot/` (subfolders + key files)

- `Cmd/` — command handling helpers.
- `Debug/` — debugging utilities for bots.
- `Engine/` — decision engine core (strategies/actions orchestration).
- `Factory/` — AI context/engine factories and default strategy wiring.
- `PlayerbotAI.*` — main bot AI entry point.
- `PlayerbotMgr.*` — master-bound bot manager.
- `RandomPlayerbotMgr.*` — random bot manager.

### `src/Db/` (repositories/caches)

- `FlightMasterCache.*`
- `PlayerbotDungeonRepository.*`
- `PlayerbotRepository.*`
- `PlayerbotSpellRepository.*`

### `src/Mgr/` (subfolders)

- `Guild/`
- `Item/`
- `Move/`
- `Security/`
- `Talent/`
- `Text/`
- `Travel/`

### `src/Script/` (scripts)

- `PlayerbotCommandScript.*`
- `Playerbots.*`
- `PlayerbotsSecureLogin.*`
- `playerbots_loader.cpp`
- `WorldThr/`

### `src/Util/` (helpers)

- `BroadcastHelper.*`
- `Helpers.*`
- `LazyCalculatedValue.h`
- `PlaceholderHelper.*`
- `ServerFacade.*`

```mermaid
flowchart TD
  A[src/Bot] --> B[PlayerbotAI / Managers]
  B --> C[src/Ai]
  B --> D[src/PlayerbotAIConfig.*]
  B --> E[src/Db]
  B --> F[src/Mgr]
  B --> G[src/Script]
  B --> H[src/Util]
  C --> C1[Base]
  C --> C2[Class]
  C --> C3[Dungeon]
  C --> C4[Raid]
  C --> C5[World]
  B --> B1[Cmd]
  B --> B2[Debug]
  B --> B3[Engine]
  B --> B4[Factory]
  F --> F1[Guild]
  F --> F2[Item]
  F --> F3[Move]
  F --> F4[Security]
  F --> F5[Talent]
  F --> F6[Text]
  F --> F7[Travel]
```

## Glossary (initial)

- **Bot**: A character controlled by `PlayerbotAI` instead of a human player.
- **Master**: The real player controlling one or more bots via commands (linked to `PlayerbotMgr`).
- **Random bot**: Autonomous bot managed by `RandomPlayerbotMgr` (not tied to a master).
- **Strategy**: A behavior bundle registered into an `Engine` (e.g., combat or non‑combat).
- **Action**: A single executable behavior chosen by the `Engine` (cast, move, loot, etc.).
- **Trigger**: A condition that selects which action should run.
- **Value**: A data provider used by triggers/actions (cached computation).
- **Engine**: The decision dispatcher that runs strategies, evaluates triggers, and executes actions.
- **Context (AiObjectContext)**: Factory + registry for actions, triggers, values, and strategies.

## Common section format (for all later pages)

Use this structure for every component or subsystem page:

1. **Role**  
   What the component does and when it is used.
2. **Key APIs / classes**  
   The entry points and main classes/functions.
3. **Managed data**  
   State, caches, DB tables, or configuration it owns.
4. **Extension points**  
   Where new logic can be added safely.
5. **Risks / pitfalls**  
   Common mistakes, performance costs, or side effects.