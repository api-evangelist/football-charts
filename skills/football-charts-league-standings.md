---
name: Read a Football Charts league table
description: >-
  Resolve a league key and season, then read the standings for it — classic, luck-adjusted or
  goals-based — from the Football Charts API or its MCP server.
api: mcp/football-charts-mcp.yml
surface: mcp+rest
operations: [list_leagues, get_league_table, get_rankings]
rest_operations: ['GET /api/v1/leagues/', 'GET /api/v1/leagues/{league}/table/']
generated: '2026-09-04'
method: generated
source: >-
  mcp/football-charts-tools-list.json (tool names and inputSchemas fetched anonymously) and
  https://footballcharts-backend.onrender.com/api/v1/ (endpoint list)
---

# Read a Football Charts league table

Football Charts covers 90+ leagues. League keys are **bespoke strings** (`premier`, `spain1`,
`brazil1`, `wgermany1`) and season strings differ by calendar (`2026` for summer leagues,
`2026-2027` for winter ones). Neither is derivable — you must resolve both first.

## Before you start

- Get a free key: `POST https://footballcharts-backend.onrender.com/api/v1/keys/register/` with
  `{"email": "..."}`. It is shown once.
- Send it as `Authorization: Bearer fc_<key>` (or `X-API-Key: fc_<key>`). On MCP, either put it in
  the connector URL (`https://mcp.football-charts.com/<key>/mcp`) or send the Bearer header.
- Attribute the data as **"Data by football-charts.com"** wherever you surface it.

## Steps

1. **Always call `list_leagues` first** (REST: `GET /api/v1/leagues/`). It returns every league with
   its country, its league key, and the seasons your key may query, newest first. Do not guess a
   league key from a competition name — the provider's own tool description tells callers to resolve
   keys here.
2. Pick the league key and, if the user named a season, the season string **exactly as returned**.
   Omit `season` to get the latest.
3. Call `get_league_table` with `{league, season?}` (REST:
   `GET /api/v1/leagues/{league}/table/?season=`). You get position, points, W/D/L, goals, goal
   difference and last-5 form.
4. If the user is asking who is over- or under-performing, call `get_rankings` with
   `{league, view, season?}` where `view` is `luck` or `goals`. `view` is **required** on this tool.
   It hits the same REST path with a `view` query param, so do not call both tools for the same
   view — one request covers it.

## Rules

- Read-only. Every tool here is annotated `readOnlyHint: true` by the provider; nothing you call can
  change state.
- No pagination. A league table is a bounded collection; there is no page or cursor parameter.
- Errors come back as `{"error": {"code", "message"}}` with `application/json`. `missing_key` and
  `unknown_key` are both HTTP 401 — read `error.code`, not just the status.
- A path that does not exist returns an **HTML** 404, not the JSON envelope. Guard your JSON parse.
- Budget: 5,000 requests/day and 60/minute. No `X-RateLimit-*` headers are returned, so you cannot
  see remaining quota — cache `list_leagues`, which changes rarely.
- These are probabilities from match data. Present them as statistics, never as betting advice.
