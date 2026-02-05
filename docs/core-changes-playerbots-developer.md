# Core integration notes (Developer-only)

This page is aimed at developers maintaining the **Playerbot core fork** or working on low‑level integration between AzerothCore and mod‑playerbots.

It focuses on **entry points, contracts, and gotchas**.

## Compile‑time contract: `MOD_PLAYERBOTS`

All core changes are intended to be behind a single feature gate:

- `#ifdef MOD_PLAYERBOTS`

The build system defines it automatically when the `mod-playerbots` module is detected during module configuration, and propagates it to the targets the module depends on.

**Rule:** do not introduce additional compile flags for playerbots unless strictly necessary—keep the fork easy to rebase.

## Database contract

### Global accessor

When `MOD_PLAYERBOTS` is enabled, core exposes:

- `PlayerbotsDatabase` (`DatabaseWorkerPool<PlayerbotsDatabaseConnection>`)

The module expects the usual AzerothCore DB patterns to work:
- prepared statements
- transactions
- async query holders
- update flags / revision reporting

### Loader contract

`DatabaseLoader` is extended to allow adding a new DB “kind”:

- `DATABASE_PLAYERBOTS` flag
- `DATABASE_MASK_ALL` includes it when enabled
- `SetUpdateFlags()` allows OR‑ing flags (instead of requiring const at ctor)

**Implementation guideline:** keep all playerbots database behavior module‑driven via scripts (see DatabaseScript hooks), avoid hardcoding playerbots SQL logic in core.

## DB lifecycle hooks: `DatabaseScript`

### Startup gating

`bool DatabaseScript::OnDatabasesLoading()` allows the module to refuse startup (e.g., missing schema, incompatible revision).

Core calls it right after DB loader `Load()` succeeds:
- If it returns `false`, worldserver startup aborts.

### Shutdown / keepalive

- `OnDatabasesClosing()` is called during `StopDB()`.
- `OnDatabasesKeepAlive()` exists as an extension point (module can implement ping/maintenance if needed).

### Sync query warnings bridging

Core forwards `WarnAboutSyncQueries(true/false)` toggles into:
- `OnDatabaseWarnAboutSyncQueries(bool apply)`

This keeps “DB warning mode” consistent if the module does DB work during the world loop.

## PlayerbotScript: module‑side control plane

A new `PlayerbotScript` family exists specifically so playerbots logic stays in the module.

Key dispatch points in `ScriptMgr`:

- `OnPlayerbotUpdate(diff)` and `OnPlayerbotUpdateSessions(player)`
- `OnPlayerbotLogout(player)` and `OnPlayerbotLogoutBots()`
- `OnPlayerbotPacketSent(player, packet)`
- Sanity checks: LFG queue, kill tasks, petitions, “updates to send” gating

**Design intent:** core provides event timing + minimal plumbing; module owns behavior.

## Packet visibility: `ServerScript::OnPacketReceived`

Core adds:

- `virtual void ServerScript::OnPacketReceived(WorldSession*, WorldPacket const&)`

and calls it from `WorldSession::Update()` behind `MOD_PLAYERBOTS`, after handlers run.

**Important behavior:**
- ScriptMgr makes a copy of the packet for dispatch.
- This is a “tap” for observability and module logic; do not mutate session queue inside this callback unless you fully understand ordering.

## WorldSession bot sessions

### Bot flag and address semantics

`WorldSession` ctor adds `isBot`:
- stored in `_isBot`
- if `isBot` and no standard address, `m_Address = "bot"`

Added API:
- `bool IsBot() const`
- `void SetAddress(std::string const&)`
- `LockedQueue<WorldPacket*>& GetPacketQueue()`

**Why it matters:** the module can run sessions that are not backed by a normal socket, while still participating in the update loop.

### Outgoing packet hook + NULL opcode safety

`WorldSession::SendPacket()` now:
- refuses `NULL_OPCODE` (logs and returns)
- calls `ScriptMgr::OnPlayerbotPacketSent(GetPlayer(), packet)` before sending

This is useful for:
- bot client emulation / state sync
- debugging bot‑visible opcodes

## WorldSessionMgr shutdown hook

`WorldSessionMgr::KickAll()` calls:
- `ScriptMgr::OnPlayerbotLogoutBots()` (behind `MOD_PLAYERBOTS`)

This ensures bot cleanup happens on global kicks / shutdown sequences.

## PlayerScript additions

- `OnPlayerAfterUpdate(Player*, uint32)` hook
- chat hook enums:
  - `OnChat`
  - `OnChatWithReceiver`
  - `OnChatWithGroup`
  - `OnChatWithGuild`
  - `OnChatWithChannel`

**Typical use in playerbots:**
- parse/route chat commands
- update bot AI after core player update

## Movement integration: reverse spline orientation

`PointMovementGenerator::DoInitialize()`:
- if `_reverseOrientation` is set, call `MoveSplineInit::SetOrientationInversed()`

This is intentionally surgical: no rewrite of pathing, just an extra orientation mode used by bots.

## Rebase discipline (recommended)

To keep the fork maintainable:

1. Keep all playerbots‑specific core changes either:
   - behind `MOD_PLAYERBOTS`, or
   - in new minimal hook functions that are no‑ops by default.
2. Avoid touching unrelated gameplay logic.
3. Prefer adding **hook points** (ScriptMgr / ScriptDefines) over embedding module logic into core.
4. When adding new entry points, document:
   - call site
   - expected invariants (thread, timing, player existence)
   - failure behavior (abort startup, ignore, etc.)

## Quick reference: core touchpoints

- Build:
  - module detection → `MOD_PLAYERBOTS` define
  - `modules` links against `mysql`
- DB:
  - `PlayerbotsDatabase` pool
  - `DatabaseLoader` flags + `SetUpdateFlags`
- Scripts:
  - `DatabaseScript` lifecycle extensions
  - `PlayerbotScript` family
  - `ServerScript::OnPacketReceived`
  - `PlayerScript` after‑update + chat hooks
- Sessions:
  - `WorldSession` bot flag / packet hooks
  - `WorldSessionMgr::KickAll()` logout‑bots hook
- Movement:
  - reversed orientation in `PointMovementGenerator`