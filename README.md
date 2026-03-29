# EventLogger

A Procon plugin for Battlefield game servers that logs server events to chat, console, logfile, and/or MySQL database.

## Supported Games

- Battlefield 3 (BF3)
- Battlefield 4 (BF4)
- Battlefield Hardline (BFH)
- Bad Company 2 (BC2)

## Features

- Logs player disconnects, kicks, admin kills, admin moves, bans (permanent and timed), and Procon layer account events
- Configurable output destinations per event type: in-game chat, plugin console, logfile, MySQL database
- Regex-based filtering for player names and reasons
- Auto-ban based on disconnect reason patterns (PunkBuster, GGC-Stream, BF4DB, PBBans)
- AdKats ban enforcer integration (cross-server ban sync)
- Plugin settings password protection
- Automatic database cleanup of old entries

## Author

maxdralle (maxdralle@gmx.com)

## License

GPLv3 - see [LICENSE](LICENSE)
