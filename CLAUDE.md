# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build system — open `index.html` directly in a browser. There are no dependencies to install, no server required, and no build step.

## Architecture

Everything lives in a single file: `index.html`. It is split into three logical sections:

1. **CSS** (`<style>` block) — dark football-pitch theme using CSS custom properties (`--pitch`, `--card`, `--accent`, etc.). Prediction score cells use four color classes: `pred-15`, `pred-10`, `pred-5`, `pred-0`, `pred-p`.

2. **HTML** — seven `<div class="page">` sections (leaderboard, groups, knockout, live, standings, fixtures, rules) shown/hidden by `showPage()`. Navigation buttons toggle the `active` class.

3. **JavaScript** (`<script>` block at bottom) — no framework, plain ES2020.

### Core Data Structures

| Variable | Purpose |
|---|---|
| `PLAYERS` | Array of 10 player names — the competition participants |
| `GROUPS` | 12 groups A–L, each with 4 teams (WC 2026 expanded 48-team format) |
| `predictions[player][group]` | Array of 4 team names in predicted finish order |
| `koPredictions[player][round][idx]` | Predicted winner per knockout match |
| `actualGroups[group]` | Actual sorted standings once a group is complete (populated by fetch) |
| `actualKO[round][idx]` | Actual knockout winners (populated by fetch or `setKOResult()`) |
| `groupScores[group]` | Raw match scores for each of the 6 intra-group matches |
| `koMatches[round]` | Bracket structure — teams and result per match |
| `allFixtures` | Raw match list from openfootball fetch |

### Data Flow

1. On load, `renderAll()` runs with default/empty data (all predictions default to alphabetical group order, no actual results).
2. `fetchResults()` hits the openfootball JSON feed (`OFB_URL`), parses scores into `groupScores`, derives `actualGroups` standings when all 6 group matches are played, calls `updateKOBracket()` to populate R32 matchups from group qualifiers, then calls `renderAll()` + `renderFixtures()`.
3. All rendering is pure DOM string building (`.innerHTML = html`). There is no virtual DOM or reactive framework.
4. **Predictions are not persisted** — they reset on page reload. Each player must re-enter their picks each session.

### Scoring Engine

- `calcGroupPoints(player, group)` — awards 15/10/5/0 per team based on position correctness and advancement.
- `calcKOPoints(player, round, idx)` — flat points per correct knockout winner (R32: 20, R16/QF: 30, SF: 40, Final: 50).
- `totalPoints(player)` — sums all group and knockout points.

### External Dependencies

- **Google Fonts** — Bebas Neue (headings) + DM Sans (body), loaded from fonts.googleapis.com.
- **openfootball** — results fetched from `https://raw.githubusercontent.com/openfootball/worldcup.json/master/2026/worldcup.json`. The `OFB_MAP` object normalises team name variants from this feed to match the internal `GROUPS` names.

### KO Bracket Seeding Logic

`updateKOBracket()` maps the 12 group qualifiers (1st, 2nd, 3rd place) into the 16 R32 matchups following the WC 2026 bracket formula. The `pairs` array in that function encodes which group positions face each other.

### Adding/Changing Predictions

Predictions are set via `<select>` dropdowns in the rendered tables. Group picks call `setPrediction(player, group, newPos, teamName)` which swaps teams in `predictions[player][group]` and re-renders. Knockout picks call `setKOPick(player, round, idx, team)`. Actual KO results (for tournament admin) are set via `setKOResult(round, idx, team)`.
