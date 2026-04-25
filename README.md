# Achim's Cup — Badminton Mixer Tournament

Single-file HTML app (4592 lines). No build step, no server required. Open `index.html` in a browser.

## What It Does

Runs doubles mixer tournaments (Americano or Mexicano variant). 4–40 players, configurable rounds/courts/scoring. Elo ratings, live standings with sparkline graphs, PDF export, embedded fake tournament simulator for demo purposes.

## Architecture

Single HTML file. All JS inline in a `<script>` block. No external JS dependencies except:
- `html2pdf.js` (CDN) for PDF export
- Google Fonts (CDN)

### Core Classes

| Class | Line | Purpose |
|---|---|---|
| `MatchmakingEngine` | ~1018 | Round generation, partner/opponent selection, bye assignment |
| `SimulationEngine` | ~1183 | Fake tournament runner, score generation, Elo application |

### Key Functions

| Function | Purpose |
|---|---|
| `renderStandingsTable()` | Standings with sparklines, form badges, partner/opponent matrices |
| `renderSparkline()` | Unified SVG renderer (Elo + rank, live + PDF, active + inactive segments) |
| `renderMatchHistory()` | Grouped by round, dropout annotations |
| `renderInformationSection()` | Explanatory text (Elo, matchmaking, scoring, fairness) |
| `showPlayerProfile()` | Slide-up modal with Elo chart, stats, partner/opponent history |
| `exportPDF()` | Full PDF with cover page (player strengths), standings, history, info |
| `calculateAlgorithmHealth()` | Algorithm quality metrics (novelty, balance, fairness) |

### State

`AppState` (line ~984): global tournament state, persisted to `localStorage`.
`SimState`: separate state object for fake tournament simulations (not persisted).

## Algorithms

### Matchmaking (`findBestMatch`)

Three-tier scoring for each candidate 4-player match:
1. **Partner novelty** (weight 1000): new partner = +1000 per team pair, repeat = −300 per prior pairing
2. **Opponent novelty** (weight 25): +25 per new cross-team opponent pair
3. **Team balance** (weight 40): closer average Elo between teams = higher score

### Elo Rating

```
delta = K × ln(|score1 − score2| + 1) × (S − E)
K = 24
S = 1 (winner) / 0 (loser)
E = 1 / (1 + 10^((avgOpp − avgTeam) / 400))
```
Zero-sum: winner gains exactly what loser loses.

### Score Generation (Sim)

Per-point win probability: `pointProb = 0.5 + (matchWinProb − 0.5) × 0.65`
Dampening factor 0.65 controls how decisively Elo gaps translate to scores. Respects deuce/cap rules.

### Simulation Players

17 preset names with deterministic Elos (920–1450). Extra players get random Elo in same range.

## Config

| Option | Default | Notes |
|---|---|---|
| Win target | 21 | Points per game |
| Deuce | on | Win by 2, cap 30 |
| Match win pts | 3 | Tournament points |
| Close loss bonus | +1 | Within threshold (2 pts wizard, 1 pt sim panel) |
| Mixer variant | Americano | Mexicano also available |
| Dropout policy | Ghost | Bye redistribution also available |

## Themes

5 CSS themes via `data-theme` attribute: light, dark, court-green, midnight-blue, high-contrast.

## Testing

87 tests embedded in the `TestSuite` object. Run via browser console: `TestSuite.runAll()`.
Categories: Elo (12), Score Gen (8), Matchmaking (12), Simulation (10), Health (6), Ranking (5), Matrix (5), Edge Cases (10), Stress (4), Global (1), Config (1), Regression (10), Statistical (2), Dropout (1).

## File Structure

```
index.html
```

## Browser Support

Modern browsers (ES2020+). Uses `crypto.randomUUID()`, `structuredClone`-free deep copy via `JSON.parse(JSON.stringify())`.
