# ESPN Fantasy Football MCP Server

## Overview

This MCP (Model Context Protocol) server allows LLMs like Claude to interact with the ESPN Fantasy Football API. It provides tools for accessing league data, team rosters, player statistics, and more through a standardized interface. It can work with both public and private ESPN Leagues.

## Features (MCP Tools)

- **`authenticate`**: Store ESPN credentials for the current session (for private leagues)
- **`get_league_info`**: Basic information about a league
- **`get_league_settings`**: Scoring rules, roster slots and league configuration
- **`get_league_standings`**: Current rankings, records and points for/against
- **`get_team_roster`**: A team's current roster with slots, injury status and projections
- **`get_team_info`**: Team statistics and transaction counts
- **`get_player_stats`**: Stats for a specific player by name
- **`get_matchup_info`**: Weekly matchup details
- **`get_free_agents`**: The waiver wire and free-agent pool, optionally filtered by position
- **`logout`**: Clear stored credentials

## Installation

### Prerequisites

- Python 3.10 or higher
- `uv` package manager
- [Claude Desktop](https://claude.ai/download) for the best experience

### Usage with Claude Desktop

1. Update the Claude Desktop config:

- MacOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Include reference to the MCP server

  ```json
  {
    "mcpServers": {
      "espn-fantasy-football": {
        "command": "uv",
        "args": [
          "--directory",
          "/absolute/path/to/directory",
          "run",
          "espn_fantasy_server.py"
        ]
      }
    }
  }
  ```

2. Restart Claude Desktop

### Private leagues

Private leagues need your `ESPN_S2` and `SWID` cookies, which you can read out of
your browser after logging in to ESPN. There are two ways to supply them.

**Per session**, by calling the `authenticate` tool with the two values. These are
held in memory only and are cleared by `logout`.

**Per install**, by adding them to the server's `env` block in the Claude Desktop
config, which avoids re-authenticating on every restart:

```json
{
  "mcpServers": {
    "espn-fantasy-football": {
      "command": "uv",
      "args": ["--directory", "/absolute/path/to/directory", "run", "espn_fantasy_server.py"],
      "env": {
        "ESPN_S2": "your_espn_s2_cookie",
        "SWID": "{your-swid-cookie}"
      }
    }
  }
}
```

Credentials passed to `authenticate` take priority over the environment, so you can
override the configured account for a session without editing the config.

## Notes

- ESPN team IDs are **not** guaranteed to be contiguous `1..N`. A league that has
  dropped a team can expose IDs above the team count, which is what you see in the
  URL bar. Use the `team_id` reported by `get_league_standings` rather than assuming
  it matches a team's rank.
- The season year defaults to the current calendar year, or the previous year if the
  current month is before July, since the NFL season spans two calendar years.
- League data is cached for 60 seconds, so roster changes made in the ESPN app show
  up on the next call rather than needing a server restart.

## Acknowledgements

Forked from [KBThree13/mcp_espn_ff](https://github.com/KBThree13/mcp_espn_ff) by Kyle
Brogan, which this builds on under the MIT license.

[cwendt94/espn-api](https://github.com/cwendt94/espn-api) for the nifty python wrapper around the ESPN Fantasy API
