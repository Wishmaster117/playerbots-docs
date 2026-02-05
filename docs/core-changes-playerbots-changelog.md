# Core changes for mod‑playerbots (Changelog)

This is a compact changelog of AzerothCore core modifications required by the `Playerbot` core branch to support **mod‑playerbots**.

## Build / CMake

- Detect `mod-playerbots` module and define `MOD_PLAYERBOTS` for relevant targets.
- Link `modules` with `mysql` when playerbots is present.

## Databases

- Add a new DB pool (guarded by `MOD_PLAYERBOTS`):
  - `DatabaseWorkerPool<PlayerbotsDatabaseConnection> PlayerbotsDatabase`
- Extend DB forward declarations and alias types for prepared statements/transactions/query holders.
- Extend `DatabaseLoader`:
  - new flag `DATABASE_PLAYERBOTS`
  - `DATABASE_MASK_ALL` includes playerbots (when enabled)
  - add `SetUpdateFlags()` to OR‑in flags
  - add template instantiation for `AddDatabase<PlayerbotsDatabaseConnection>()`

## DB lifecycle hooks (DatabaseScript)

Add new `DatabaseScript` hook points + `ScriptMgr` forwarders:

- `OnDatabasesLoading()` (bool gate)
- `OnDatabasesKeepAlive()`
- `OnDatabasesClosing()`
- `OnDatabaseWarnAboutSyncQueries(bool)`
- `OnDatabaseSelectIndexLogout(Player*, uint32&, uint32&)`
- `OnDatabaseGetDBRevision(std::string&)`

Integrate in worldserver:

- Call `OnDatabasesLoading()` after DB loader load and abort startup on failure.
- Call `OnDatabasesClosing()` on shutdown.
- Forward sync‑query warning toggles into scripts.

## Logging

- Add `Appender.Playerbots` and `Logger.playerbots` (dedicated `Playerbots.log`).

## Scripting: PlayerbotScript family

- Add `PlayerbotScript` type and registration.
- Add `ScriptMgr` dispatch methods:
  - LFG queue check, kill task check, petition account check
  - bot update cadence, per‑session update
  - bot logout (single + all)
  - packet‑sent callback

## Networking / Sessions

- `WorldSession`:
  - add `isBot` constructor parameter + `_isBot` flag
  - set address to `"bot"` for bot sessions without real socket address
  - add `IsBot()`, `SetAddress(...)`, expose packet queue accessor
  - guard against `NULL_OPCODE` in `SendPacket()` (log + return)
  - call `OnPlayerbotPacketSent(...)` on outgoing packets
- `WorldSession::Update()`:
  - call `ScriptMgr::OnPacketReceived(...)` for inbound packets behind `MOD_PLAYERBOTS`
- `WorldSessionMgr::KickAll()`:
  - call `OnPlayerbotLogoutBots()` behind `MOD_PLAYERBOTS`

## Player hooks

- Add `PlayerScript::OnPlayerAfterUpdate(...)`
- Add additional chat hook enums:
  - `OnChat`, `OnChatWithReceiver`, `OnChatWithGroup`, `OnChatWithGuild`, `OnChatWithChannel`

## Movement

- `PointMovementGenerator`:
  - support inverted spline orientation via `SetOrientationInversed()` when `_reverseOrientation` is set