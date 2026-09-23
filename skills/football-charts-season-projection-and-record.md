---
name: Project a Football Charts season and check the model's record
description: >-
  Read the Monte Carlo season projection for a league — title, top-4 and relegation probabilities —
  and check it against the provider's own settled prediction ledger before you quote it.
api: mcp/football-charts-mcp.yml
surface: mcp+rest
operations: [list_leagues, get_season_projection, get_track_record, get_team, get_goal_timing]
rest_operations:
  - 'GET /api/v1/leagues/'
  - 'GET /api/v1/leagues/{league}/projection/'
  - 'GET /api/v1/track-record/'
  - 'GET /api/v1/leagues/{league}/teams/{team}/'
  - 'GET /api/v1/leagues/{league}/goal-timing/'
generated: '2026-09-04'
method: generated
source: >-
  mcp/football-charts-tools-list.json (tool names and inputSchemas fetched anonymously) and
  https://footballcharts-backend.onrender.com/api/v1/ (endpoint list)
---

# Project a Football Charts season and check the model's record

The projection is a 10,000-run Monte Carlo simulation, re-run daily. The track record is the
provider's own settled ledger of published signals — it exists so a caller can quote the model with
its hit rate attached rather than on trust.

## Steps

1. `list_leagues` to resolve the league key and confirm the season is one your key may query.
2. `get_season_projection` with `{league}` (REST: `GET /api/v1/leagues/{league}/projection/`) for
   title / top-4 / relegation percentages and points ranges. Takes **only** `league`; it always
   returns the current season.
3. `get_track_record` with `{days?}` (REST: `GET /api/v1/track-record/`) to pull the settled ledger.
   Quote the projection **with** the record — that is the point of the service.
4. For a single club's underlying numbers, `get_team` with `{league, team, season?}` (REST:
   `GET /api/v1/leagues/{league}/teams/{team}/`) — match log, goal timing and stats. Here `team` is
   a key in the path, not a substring filter.
5. For a whole league's scoring pattern, `get_goal_timing` with `{league, season?}` (REST:
   `GET /api/v1/leagues/{league}/goal-timing/?season=&team=`) — goals per 15-minute bin, per team.

## Rules

- The projection changes at most once a day. Re-requesting it inside a session wastes quota against
  a 5,000/day, 60/minute ceiling you cannot observe (no rate-limit headers are returned).
- There is no MCP tool for the league's team roster — if you need the list of teams in table order,
  call the REST operation `GET /api/v1/leagues/{league}/teams/?season=` directly, or take the names
  out of `get_league_table`.
- Read-only throughout; no idempotency key is needed or offered because nothing here mutates.
- Attribute as "Data by football-charts.com". Present projections as probabilities. Statistics, not
  betting advice; 18+.
