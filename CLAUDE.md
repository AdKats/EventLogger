# EventLogger - Procon v2 Plugin

## Project Overview

EventLogger is a C# event logging plugin for Procon v2 (Battlefield game server administration). It captures server events and logs them to chat, console, logfile, and/or MySQL database. The legacy Procon v1 version lives on the `legacy` branch.

- **Language:** C#
- **License:** GPLv3
- **Author:** maxdralle (maxdralle@gmx.com)
- **Supported games:** BF3, BF4, BFH, BC2
- **Dependencies:** MySqlConnector, Dapper, Procon v2 (runtime only)

## Architecture

Single-file plugin at `src/EventLogger.cs`. Class `EventLogger` extends `PRoConPluginAPI` and implements `IPRoConPluginInterface`.

## Event Registrations

The plugin registers for the following Procon events via `RegisterEvents()`:

- `OnPunkbusterMessage` - PunkBuster violation messages (currently returns immediately / unused)
- `OnPlayerJoin` - Tracks player join times
- `OnListPlayers` - Player list updates (used for AdKats ban enforcer GUID tracking)
- `OnPlayerDisconnected` - Player disconnect with reason, auto-ban trigger
- `OnRoundOver` - Round end timestamp tracking
- `OnPlayerKicked` - Player kicked events
- `OnPlayerKickedByAdmin` - Admin kick events
- `OnPlayerKilledByAdmin` - Admin kill events
- `OnPlayerMovedByAdmin` - Admin move/force-move events
- `OnBanAdded` - Permanent and timed ban events
- `OnAccountCreated` - Procon layer account creation
- `OnAccountDeleted` - Procon layer account deletion
- `OnAccountLogin` - Procon layer login events
- `OnAccountLogout` - Procon layer logout events

## Threading Model

The plugin uses Procon's task scheduler (`procon.protected.tasks.add`) for periodic work:

- **WriteLogfile** - Flushes buffered log entries to `Configs/EVENT-LOGGER.txt` (every 500s cycle)
- **AutoDBCleaner** - Deletes old database entries beyond configured retention (every 9999s cycle)
- **CheckRestart** - Detects layer restarts within first 200 seconds
- **AdkatsBanEnforcerThread** - Polls AdKats database for banned players (every 29s cycle)
- **TmpListCleaner** - Clears temporary lookup caches (every 80000s cycle)

Ad-hoc threads (`new Thread`) are used for:
- `WriteLogfile` - File I/O on background thread (`ThreadWorker588`)
- `TableBuilder` - MySQL table creation on background thread (`MySQLWorker4`)
- `AdkatsBanEnforcerThread` - Ban check queries on background thread (`ThreadWorkerEventLogger1`)

## Database

### Connection

MySQL via `MySqlConnector`. Connection strings built from plugin settings (host, port, database, username, password). Two separate connections supported: one for EventLogger's own database, one for AdKats database (ban enforcer).

### Tables

**`event_logger`** - Main logging table:
- `ID` INT NOT NULL AUTO_INCREMENT (PRIMARY KEY)
- `gameserver` VARCHAR(30) NOT NULL
- `event` VARCHAR(30) NOT NULL
- `timestamp` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
- `playername` VARCHAR(35) NULL
- `msg` VARCHAR(400) NULL

The plugin auto-creates this table if it does not exist (`TableBuilder` method). Auto DB cleaner deletes rows older than configurable days via `TIMESTAMPDIFF`.

### AdKats Integration

Reads from AdKats tables (`tbl_playerdata`, `tbl_server_player`, `adkats_bans`, `adkats_records_main`) for cross-server ban enforcement. Uses raw SQL queries with `MySqlCommand` and `MySqlDataAdapter`.

### SQL Safety

Uses a custom `strSqlProtection()` method that strips dangerous characters. Does NOT use parameterized queries -- the existing raw SQL concatenation is preserved from legacy code.

## Code Style

Style is enforced by `.editorconfig` and checked via `dotnet format` in CI.

**Critical conventions:**
- **Use `String`, `Int32`, `Boolean`, `Double`** -- NOT `string`, `int`, `bool`, `double`. The codebase uses explicit System type names everywhere.
- **Allman brace style** -- opening brace on its own line
- **4 spaces** for indentation, LF line endings
- **Block-scoped namespaces** (not file-scoped)
- **`using` directives outside namespace**, System usings first

## Build & CI

- `EventLogger.csproj` at root is a **CI-only artifact** for `dotnet format`. It is NOT a real build file -- Procon v2 assemblies are unavailable for compilation.
- **CI workflow** (`.github/workflows/ci.yml`): runs on push to `master` and PRs. Checks `dotnet format whitespace` and `dotnet format style --exclude-diagnostics IDE1007`.
- **Release workflow** (`.github/workflows/release.yml`): triggered by `v*` tags. Packages `.cs` files from `src/` into a zip and creates a GitHub Release.

### Running style checks locally

```bash
dotnet restore
dotnet format whitespace --verify-no-changes
dotnet format style --verify-no-changes --severity warn --exclude-diagnostics IDE1007
```

## Branch Structure

- `master` -- current development, Procon v2 only
- `legacy` -- archived Procon v1 version, no longer maintained
