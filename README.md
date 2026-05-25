# ⚽ WC2026 Challenge

> **A private prediction game for 10 friends competing across every match of the FIFA World Cup 2026.**

Built as a single HTML file — no build tools, no install, no server. Just open and play.

---

## Features

| | |
|---|---|
| **Leaderboard** | Live rankings with gold/silver/bronze cards, point totals, and a points-over-time line chart |
| **Group Stage** | Drag-and-drop-style dropdowns to predict finish order for all 12 groups (48 teams) |
| **Knockout Bracket** | Click-to-pick predictions across R32, R16, QF, SF, and the Final |
| **Fixtures** | Full match schedule pulled live from openfootball with scores as they come in |
| **Scoring Rules** | In-app reference so no one argues about points |
| **My Picks view** | Toggle to filter tables down to just your own predictions |
| **Persistence** | All picks are saved to Supabase — predictions survive page reloads and sync across devices |

---

## Scoring System

### Group Stage

Each player ranks the 4 teams in a group from 1st to 4th. Points are awarded per team based on how close the predicted position is to the actual final standing.

| Prediction accuracy | Points |
|---|---|
| Exact position | **15 pts** |
| One place off | **10 pts** |
| Two places off | **5 pts** |
| Three places off | **0 pts** |

Maximum per group: **60 pts** · Maximum across all 12 groups: **720 pts**

### Knockout Stage

Flat points for each correct round winner:

| Round | Points per correct pick |
|---|---|
| Round of 32 (R32) | **20 pts** |
| Round of 16 (R16) | **30 pts** |
| Quarter-finals (QF) | **30 pts** |
| Semi-finals (SF) | **40 pts** |
| Final | **50 pts** |

Maximum knockout points: **(16×20) + (8×30) + (4×30) + (2×40) + (1×50) = 830 pts**

---

## The Groups (WC 2026 — 48-Team Format)

| Group | Teams |
|---|---|
| A | Mexico · South Africa · South Korea · Czech Republic |
| B | Canada · Switzerland · Qatar · Bosnia-Herzegovina |
| C | Brazil · Morocco · Scotland · Haiti |
| D | USA · Paraguay · Australia · Turkey |
| E | Germany · Curacao · Ivory Coast · Ecuador |
| F | Netherlands · Japan · Tunisia · Sweden |
| G | Belgium · Egypt · Iran · New Zealand |
| H | Spain · Cape Verde · Saudi Arabia · Uruguay |
| I | France · Senegal · Norway · Iraq |
| J | Argentina · Algeria · Austria · Jordan |
| K | Portugal · Colombia · Uzbekistan · DR Congo |
| L | England · Croatia · Ghana · Panama |

---

## Players

Adam · Brandon · Nathan · Henry · James · Nikhil · Dev · Keaton · Will · Steve

---

## Getting Started

### 1. Open the app

No installation required. Just open `index.html` in any modern browser:

```
# Option A — double-click the file in Finder/Explorer

# Option B — from the terminal
open index.html        # macOS
start index.html       # Windows
xdg-open index.html    # Linux
```

> Chrome or Firefox recommended. The app uses ES2020 JavaScript and CSS custom properties.

### 2. Select your player

On first load (or when no player is selected), a login overlay appears. Click your name to claim your profile. Your picks are immediately loaded from Supabase.

### 3. Enter your predictions

- **Group Stage tab** — use the dropdowns in each group table to reorder teams (1st → 4th)
- **Knockout tab** — as the bracket populates, click a team name to pick them as the match winner
- Picks auto-save to Supabase on every change

### 4. Track the competition

- **Leaderboard tab** — check overall standings and the points-over-time chart
- **My Picks button** (top-right) — toggle to see only your own row in every table
- **Fixtures tab** — see all match results as they come in

---

## Architecture

Everything lives in a single file: `index.html` (~1,940 lines).

```
index.html
├── <style>          Dark FIFA-themed UI using CSS custom properties + DaisyUI overrides
├── <body>           Five nav pages: leaderboard · groups · knockout · fixtures · rules
└── <script>         Plain ES2020 — no framework, no bundler
    ├── SUPABASE     Client init (anon key, public project)
    ├── CORE DATA    PLAYERS, GROUPS, FLAGS, prediction state, KO bracket state
    ├── RENDERING    Pure DOM string building via .innerHTML
    ├── SCORING      calcGroupPoints(), calcKOPoints(), totalPoints()
    ├── PERSISTENCE  savePredictions(), loadPredictions() via Supabase
    └── FETCH        fetchResults() — hits openfootball JSON feed, derives standings
```

### External dependencies (all CDN, no install)

| Library | Version | Purpose |
|---|---|---|
| [Tailwind CSS](https://tailwindcss.com) | CDN | Utility classes |
| [DaisyUI](https://daisyui.com) | 3.x | Component theme (dark mode) |
| [Chart.js](https://www.chartjs.org) | 4.x | Points-over-time line chart |
| [Supabase JS](https://supabase.com/docs/reference/javascript) | 2.x | Prediction persistence |
| [Google Fonts](https://fonts.google.com) | — | Oswald (headings) + DM Sans (body) |
| [flagcdn.com](https://flagcdn.com) | — | Country flag images |
| [openfootball/worldcup.json](https://github.com/openfootball/worldcup.json) | — | Live match results feed |

---

## Data Flow

```
Page load
  └─► renderAll()          Renders UI with empty/default data
  └─► loadPredictions()    Fetches all player picks from Supabase
  └─► fetchResults()       Hits openfootball JSON feed
        └─► parse scores into groupScores{}
        └─► derive actualGroups{} when all 6 group matches played
        └─► updateKOBracket()  Seeds R32 matchups from group qualifiers
        └─► renderAll() + renderFixtures()
```

Player pick changes auto-call `savePredictions()` which upserts to Supabase immediately.

---

## Customising

### Adding or changing players

Edit the `PLAYERS` array near the top of the `<script>` block:

```js
const PLAYERS = ['Adam','Brandon','Nathan','Henry','James','Nikhil','Dev','Keaton','Will','Steve'];
```

### Changing the Supabase project

Replace the URL and anon key in the `sb` client initialisation (search for `supabase.createClient`).

To find your anon key:
1. Go to your project at [supabase.com/dashboard](https://supabase.com/dashboard)
2. Open **Project Settings** (gear icon in the left sidebar)
3. Click **API** under the Configuration section
4. Scroll down to the **Legacy API keys** section
5. Copy the **`anon` / `public`** key — it's the safe, client-side key intended for browser use

> The **service role** key shown on the same page has full database access and should never go in frontend code.

### Updating team names

The `OFB_MAP` object (search for `OFB_MAP`) maps openfootball feed name variants to the internal `GROUPS` team names. Add entries there if the feed uses different spellings.

---

## Contributing

This is a private group project. If you're one of the 10 players and something looks broken, open an issue or ping Dev directly.
