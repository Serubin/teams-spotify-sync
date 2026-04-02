# teams-spotify-sync

Automatically lowers your Spotify volume when you join a Microsoft Teams meeting, and restores it when you leave.

## Prerequisites

- macOS
- [jq](https://jqlang.github.io/jq/) — `brew install jq`
- [teams-monitor](https://github.com/svrooij/teams-monitor) — `dotnet tool install -g svrooij.teams-monitor`
- Spotify desktop app
- [spotify CLI](https://github.com/mbarkhau/spotify-cli-linux) *(optional)* — `pip install spotify-cli-linux` — enables per-device volume rules

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
- **Per-device rules** — if the `spotify` CLI is installed, configure different volumes per device
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
DEVICE_RULES=""        # Per-device rules (see below)
# TEAMS_MONITOR_CMD=teams-monitor
```

Your actual pre-meeting volume is saved and restored automatically — `DEFAULT_VOLUME` is only used as a fallback if the pre-meeting volume couldn't be captured.

### Device rules

If you have the `spotify` CLI installed, you can set different meeting volumes per device. Rules are comma-separated `pattern=volume` pairs in `DEVICE_RULES`. First match wins.

**Match by name** — case-insensitive substring match against the Spotify device name:

```sh
DEVICE_RULES="FortiBook Pro=40,Living Room=20"
```

**Match by type** — prefix with `type:` to match device type (Computer, Smartphone, Speaker):

```sh
DEVICE_RULES="type:Speaker=20,type:Computer=50"
```

**Mix both** — name rules and type rules can be combined. First match wins:

```sh
DEVICE_RULES="FortiBook Pro=40,type:Speaker=20,type:Computer=65"
```

**Sonos grouping** — Sonos groups appear as "Speaker + 1", "Speaker + 2", etc. The suffix is stripped before matching, so a rule for "Speaker" matches all groups.

**Unmatched devices** are left alone — no volume change occurs.

**Without the `spotify` CLI** or with an empty `DEVICE_RULES`, the global `MEETING_VOLUME` is used for all devices.

## How it works

1. Pipes output from `teams-monitor` directly — no intermediate log files or file watchers
2. Parses JSON meeting state updates and detects state transitions (join/leave)
3. On meeting join: detects the active Spotify device and looks up per-device volume rules
4. Saves current Spotify volume, lowers it to the matched (or global) meeting volume
5. On meeting leave: restores the saved volume
6. Auto-restarts `teams-monitor` with exponential backoff if it crashes
7. Gracefully restores volume on shutdown (SIGTERM/SIGINT)

Spotify volume is controlled via AppleScript. Device detection uses the `spotify` CLI (optional).

## License

MIT
