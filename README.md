# teams-spotify-sync

Automatically lowers your Spotify volume when you join a Microsoft Teams meeting, and restores it when you leave.

## Prerequisites

- macOS
- [jq](https://jqlang.github.io/jq/) — `brew install jq`
- [teams-monitor](https://github.com/svrooij/teams-monitor) — `dotnet tool install -g svrooij.teams-monitor`
- Spotify desktop app

## Install

### Via Homebrew

```sh
brew tap serubin/tools
brew install teams-spotify-sync
```

### Manual

```sh
curl -o /usr/local/bin/teams-spotify-sync https://raw.githubusercontent.com/serubin/teams-spotify-sync/main/teams-spotify-sync
chmod +x /usr/local/bin/teams-spotify-sync
```

## Getting started

Run the interactive setup to configure your preferences and install the LaunchAgent:

```sh
teams-spotify-sync setup
```

This will prompt you for:
- **Meeting volume** — Spotify volume while in a meeting (default: 65)
- **Default volume** — fallback restore volume (default: 100)
- Whether to install a LaunchAgent so it starts automatically on login

## Commands

```sh
teams-spotify-sync run        # Run in foreground (for testing)
teams-spotify-sync setup      # Interactive configuration
teams-spotify-sync install    # Install LaunchAgent (auto-start on login)
teams-spotify-sync uninstall  # Remove LaunchAgent
teams-spotify-sync status     # Show current status and config
teams-spotify-sync version    # Print version
```

## Configuration

Config file: `~/.config/teams-spotify-sync/config`

```sh
MEETING_VOLUME=65      # Volume during meetings (0-100)
DEFAULT_VOLUME=100     # Fallback restore volume
# TEAMS_MONITOR_CMD=teams-monitor
```

Your actual pre-meeting volume is saved and restored automatically — `DEFAULT_VOLUME` is only used as a fallback if the pre-meeting volume couldn't be captured.

## How it works

1. Pipes output from `teams-monitor` directly — no intermediate log files or file watchers
2. Parses JSON meeting state updates and detects state transitions (join/leave)
3. On meeting join: saves current Spotify volume, lowers it to `MEETING_VOLUME`
4. On meeting leave: restores the saved volume
5. Auto-restarts `teams-monitor` with exponential backoff if it crashes
6. Gracefully restores volume on shutdown (SIGTERM/SIGINT)

Spotify is controlled via AppleScript — no additional CLI tools needed.

## License

MIT
