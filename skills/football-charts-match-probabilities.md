---
name: Get Football Charts match probabilities
description: >-
  Pull upcoming fixtures with model probabilities for a league, then drill into one match for the
  full probability block across markets.
api: mcp/football-charts-mcp.yml
surface: mcp+rest
operations: [list_leagues, get_fixtures, get_match, get_results]
rest_operations:
  - 'GET /api/v1/leagues/'
  - 'GET /api/v1/leagues/{league}/fixtures/'
  - 'GET /api/v1/matches/{slug}/'
  - 'GET /api/v1/leagues/{league}/results/'
generated: '2026-09-04'
method: generated
source: >-
  mcp/football-charts-tools-list.json (tool names and inputSchemas fetched anonymously) and
  https://footballcharts-backend.onrender.com/api/v1/ (endpoint list)
---

# Get Football Charts match probabilities

Football Charts runs one Dixon-Coles scoreline model per match and derives every market from that
single joint distribution, so 1X2, over/under and BTTS numbers are mutually consistent by
construction. Free-tier keys get probabilities but **no betting odds**.

## Steps

1. `list_leagues` to resolve the league key (see the standings skill — never guess a key).
2. `get_fixtures` with `{league}` (REST: `GET /api/v1/leagues/{league}/fixtures/`) for upcoming
   matches carrying model probabilities. This tool takes **only** `league` — there is no date range
   or season parameter.
3. For one match, `get_match` with `{slug}` (REST: `GET /api/v1/matches/{slug}/`). The slug comes
   from the fixtures or results payload; there is no search-by-team-name entry point for a match.
4. For what already happened, `get_results` with `{league, season?, team?, last?}` (REST:
   `GET /api/v1/leagues/{league}/results/?season=&team=`) — FT/HT scores and first-goal minute,
   oldest first. `team` is a **name substring filter**, not a key.

## Rules

- `last` on `get_results` is an MCP-only convenience: the REST API has no such parameter, and the
  provider's MCP server fetches the full set and trims client-side. Over REST, expect the whole
  season and trim yourself.
- Free tier covers the current + previous season per league. Asking for an older season will not
  return data your key cannot see — check what `list_leagues` reported.
- Errors: `{"error": {"code", "message"}}`, HTTP 401 for `missing_key` / `unknown_key`. Unrouted
  paths return HTML.
- State the numbers as probabilities with the attribution "Data by football-charts.com". This is a
  statistics service, not a tipster — 18+ and never framed as advice to stake money.
