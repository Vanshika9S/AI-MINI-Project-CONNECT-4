# ⚫ Connect Four AI

> A fully interactive Connect Four game powered by the **Minimax algorithm with Alpha-Beta Pruning** — playable in any modern browser with zero dependencies.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Live Demo / Quick Start](#live-demo--quick-start)
- [Features](#features)
- [How to Play](#how-to-play)
- [AI Architecture](#ai-architecture)
  - [Minimax Algorithm](#minimax-algorithm)
  - [Alpha-Beta Pruning](#alpha-beta-pruning)
  - [Heuristic Evaluation Function](#heuristic-evaluation-function)
  - [Move Ordering](#move-ordering)
- [Search Tree Visualizer](#search-tree-visualizer)
- [UI Components Explained](#ui-components-explained)
  - [Left Panel — AI Configuration](#left-panel--ai-configuration)
  - [Center — Game Board](#center--game-board)
  - [Right Panel — Analytics](#right-panel--analytics)
- [Difficulty & Depth Settings](#difficulty--depth-settings)
- [Scoring System](#scoring-system)
- [Technical Details](#technical-details)
  - [Project Structure](#project-structure)
  - [Key Constants](#key-constants)
  - [Board Representation](#board-representation)
  - [Win Detection](#win-detection)
- [Algorithm Deep Dive](#algorithm-deep-dive)
  - [Minimax Pseudocode](#minimax-pseudocode)
  - [Alpha-Beta Cutoff Condition](#alpha-beta-cutoff-condition)
  - [Score Propagation](#score-propagation)
- [Performance Characteristics](#performance-characteristics)
- [Known Limitations](#known-limitations)
- [Browser Compatibility](#browser-compatibility)
- [File Structure](#file-structure)
- [Customization Guide](#customization-guide)
- [Glossary](#glossary)

---

## Overview

This is a single-file HTML application that lets you play **Connect Four** against an AI opponent that "thinks" using classical game-tree search. The goal is to connect four of your discs in a row — horizontally, vertically, or diagonally — before the AI does.

What makes this implementation special:

- **No frameworks, no bundlers, no installs.** Open `c4.html` in a browser and play.
- The AI's full **Minimax + Alpha-Beta search tree** is rendered as an interactive, zoomable, pannable canvas diagram after every AI move.
- Every AI decision comes with a **plain-language explanation** in the move log.
- A **live score graph** tracks board advantage over time.

---

## Live Demo / Quick Start

```bash
# Option 1 — just open the file
open c4.html

# Option 2 — serve it locally (avoids any potential browser restrictions)
npx serve .
# then navigate to http://localhost:3000/c4.html
```

No npm install, no build step, no backend required.

---

## Features

| Feature | Description |
|---|---|
| 🤖 Minimax AI | Full game-tree search with configurable depth (2–5) |
| ✂️ Alpha-Beta Pruning | Eliminates provably suboptimal branches to speed up search |
| 🌳 Tree Visualizer | Full-screen interactive canvas showing the entire search tree after each AI move |
| 📊 Score Graph | Real-time chart showing board evaluation over the course of the game |
| 📝 Move Log | Human-readable explanation of every AI decision |
| 📋 Minimax Trace | Compact text trace of per-column scores from the last AI move |
| 🏆 Win Detection | Highlights the winning four cells with an animation |
| 🔀 Turn Toggle | Option to let AI go first |
| 📈 Statistics Panel | Nodes evaluated, branches pruned, best score, think time |
| 🎨 Responsive UI | Three-column layout; readable on laptop and desktop screens |

---

## How to Play

1. **Open** `c4.html` in your browser.
2. The board is a **6 row × 7 column** grid. You play **🔴 Red**; the AI plays **🟡 Yellow**.
3. Click the **▼ arrow buttons** above the board to drop a disc into that column. Discs fall to the lowest empty row in the selected column.
4. The first player to connect **four discs in a row** (horizontally, vertically, or diagonally) wins.
5. If all 42 cells fill up with no winner, the game is a **draw**.
6. Click **🔄 New Game** to reset at any time.
7. Click **🔀 AI Goes First** to toggle who moves first.
8. After each AI move, click **🌳 View Minimax + Alpha-Beta Tree** to explore the full search tree.

---

## AI Architecture

### Minimax Algorithm

The AI uses **Minimax**, a classical adversarial search algorithm from game theory. The core idea:

- The AI (**MAX player**) tries to **maximize** the board's evaluation score.
- The human (**MIN player**) is assumed to play optimally, trying to **minimize** that score.
- The algorithm recursively explores all possible future game states up to a fixed **depth** (number of plies/half-moves).
- At leaf nodes (terminal states or depth limit), a **heuristic function** estimates how good the position is for the AI.
- Scores **propagate back up** the tree: MAX nodes take the maximum child score; MIN nodes take the minimum.

The root-level column that leads to the highest score is chosen as the AI's move.

### Alpha-Beta Pruning

Alpha-Beta is an optimization layered on top of Minimax that **skips branches** that cannot possibly affect the final decision.

Two values are tracked throughout the search:

| Variable | Meaning |
|---|---|
| **α (alpha)** | The best score the MAX player is guaranteed so far |
| **β (beta)** | The best score the MIN player is guaranteed so far |

**Cutoff rule:** If at any point `alpha >= beta`, the current branch is pruned — the opponent would never allow the game to reach this node, so further exploration is pointless.

In the best case, Alpha-Beta reduces the effective branching factor from **b** to approximately **√b**, allowing roughly **twice the search depth** in the same time compared to plain Minimax.

### Heuristic Evaluation Function

When the search reaches the depth limit (or a non-terminal board state), the board is scored using a custom heuristic:

```
Score = Σ (window scores across all directions) + center column bonus
```

**Window scoring** — every sliding window of 4 consecutive cells (horizontal, vertical, diagonal) is evaluated:

| Pattern in the window | Score |
|---|---|
| 4 AI discs | **+100** |
| 3 AI discs + 1 empty | **+10** |
| 2 AI discs + 2 empty | **+5** |
| 3 Human discs + 1 empty | **−10** |

**Center column bonus** — every AI disc in column 4 (the center) earns **+3** additional points. The center is strategically valuable because it participates in the most possible four-in-a-row windows.

**Terminal state scores** — wins/losses override the heuristic entirely:

| State | Score |
|---|---|
| AI wins | +100,000 + remaining depth (prefer faster wins) |
| Human wins | −100,000 − remaining depth (delay losses as long as possible) |
| Draw | heuristic score (evaluated as a normal board) |

### Move Ordering

Valid columns are evaluated in the order: **3, 2, 4, 1, 5, 0, 6** (center-outward). This is a simple but effective move ordering heuristic — because better moves (closer to center) are evaluated first, Alpha-Beta pruning cuts off more branches earlier, reducing total nodes evaluated.

---

## Search Tree Visualizer

Click **🌳 View Minimax + Alpha-Beta Tree** after any AI move to open the full-screen tree diagram.

### What you see

| Visual element | Meaning |
|---|---|
| **Yellow nodes** | MAX layers — AI is choosing the best move |
| **Red nodes** | MIN layers — simulating human's best response |
| **Green outline / glow** | The path the AI actually chose |
| **Dashed dark nodes** | Pruned branches (alpha ≥ beta cutoff occurred) |
| Score inside node | Evaluated or propagated minimax score |
| `C1`–`C7` above node | Column this move corresponds to |
| `MAX` / `MIN` below node | Which player is deciding at this layer |
| `α` / `β` labels | Alpha and beta bounds at that node |

### Controls

| Action | Effect |
|---|---|
| **Scroll wheel** | Zoom in / out |
| **Click + drag** | Pan the canvas |
| **Hover over a node** | Show detailed tooltip (score, depth, α/β values, terminal reason) |
| **＋ / －** buttons | Zoom in / out via buttons |
| **Fit** | Reset zoom and pan to show the full tree |
| **✕ Close** | Return to the game |

### Tooltip fields

When you hover over a non-pruned node, a tooltip shows:

- **Column** — which column this node represents
- **Player** — MAX (AI) or MIN (Human)
- **Depth left** — how many more plies will be searched below this node
- **Score** — the minimax value at this node
- **α (in → out)** — how alpha changed while processing this node
- **β (in → out)** — how beta changed while processing this node
- **Terminal** — reason the search stopped (AI WIN, HUMAN WIN, DRAW, DEPTH)
- ✓ **AI CHOSE THIS PATH** — shown on nodes along the selected branch

---

## UI Components Explained

### Left Panel — AI Configuration

**Search Depth slider (2–5)**
Controls how many half-moves (plies) ahead the AI searches. Higher = stronger play, longer think time. See [Difficulty & Depth Settings](#difficulty--depth-settings).

**Algorithm Stats** (updated after each AI move):

| Stat | What it measures |
|---|---|
| Nodes Evaluated | Total nodes visited during the search (includes pruned stubs) |
| Branches Pruned | How many Alpha-Beta cutoffs occurred |
| Best Score | The minimax score of the chosen move |
| Think Time | Wall-clock milliseconds for the search |

**Heuristic Table** — a quick reference for the window scoring values used in the evaluation function.

### Center — Game Board

- The **6×7 grid** is rendered as CSS circles inside a styled blue cabinet.
- **▼ buttons** above each column trigger `humanMove(col)`. Buttons are disabled during AI thinking and for full columns.
- The **status bar** shows whose turn it is, thinking animation, and end-game messages.
- The **score bar** at the top tracks wins across multiple games.

### Right Panel — Analytics

**Move Log & Explanation**
Every move (human or AI) appends a card:
- 🔴 Red border = human move
- 🟡 Yellow border = AI move
- 🟢 Green border = game result
- 🔵 Blue border = system message

The AI explanation includes the reason for the chosen move (winning threat found, blocking human, or best heuristic score).

**Board Score Over Time**
A canvas-drawn line chart. Positive scores (above the zero line) indicate AI advantage; negative scores indicate human advantage. Color follows the leader.

**Last Minimax Trace**
A compact text dump of per-column scores from the most recent search, formatted as:
```
Depth:4  Nodes:312  Pruned:47

C1: 12
C2: 8
C3: 25
C4: 31 ←CHOSEN
C5: 18
C6: 10
C7: 5
```

---

## Difficulty & Depth Settings

| Depth | Typical nodes | Typical think time | Strength |
|---|---|---|---|
| 2 | ~50–150 | < 1 ms | Easy — only 2-ply lookahead, misses many threats |
| 3 | ~200–600 | 1–5 ms | Medium — blocks obvious wins, limited foresight |
| 4 | ~800–3,000 | 5–30 ms | Hard — strong defensive and offensive play |
| 5 | ~3,000–15,000 | 30–200 ms | Very Hard — near-optimal play in most positions |

> **Note:** Node counts vary significantly depending on the board state and how much pruning occurs. Boards with many threats prune more aggressively.

---

## Scoring System

Wins are tracked across games in the current session (resets on page refresh):

- **YOU** — incremented when the human connects 4
- **AI** — incremented when the AI connects 4
- Draws do not increment either counter

The score persists across **New Game** resets within the same browser tab session.

---

## Technical Details

### Project Structure

The entire application is a **single HTML file** (`c4.html`) with no external dependencies except two Google Fonts loaded from a CDN:

- `Exo 2` — UI text
- `JetBrains Mono` — monospace values, code-style displays

### Key Constants

```js
const ROWS = 6;      // Board rows
const COLS = 7;      // Board columns
const HUMAN = 1;     // Human piece identifier
const AI = 2;        // AI piece identifier
const EMPTY = 0;     // Empty cell
const NR = 20;       // Node radius (px) in tree visualizer
const LH = 86;       // Level height (px) in tree visualizer
```

### Board Representation

The board is a **2D array** of size `ROWS × COLS`:

```js
board[row][col]  // 0 = empty, 1 = human, 2 = AI
// Row 0 = top, Row 5 = bottom
```

Pieces fall to the **highest valid row index** in a column (lowest visual position):

```js
function getNextRow(board, col) {
  for (let r = ROWS - 1; r >= 0; r--)
    if (board[r][col] === EMPTY) return r;
  return -1; // column full
}
```

### Win Detection

`checkWin(board, piece)` scans all four directions for four consecutive matching pieces:

- **Horizontal** — row-by-row, columns 0–3 as starting points
- **Vertical** — column-by-column, rows 0–2 as starting points
- **Diagonal ↗** — bottom-left to top-right
- **Diagonal ↘** — top-left to bottom-right

`getWinCells(board, piece)` returns the exact four `[row, col]` pairs for highlighting.

---

## Algorithm Deep Dive

### Minimax Pseudocode

```
function minimax(board, depth, alpha, beta, maximizing):
  if terminal(board) or depth == 0:
    return heuristic(board)

  if maximizing:  // AI's turn
    best = -∞
    for each valid column:
      drop AI piece in column
      score = minimax(board, depth-1, alpha, beta, false)
      remove piece
      best = max(best, score)
      alpha = max(alpha, best)
      if alpha >= beta: break  // prune
    return best

  else:           // Human's turn
    best = +∞
    for each valid column:
      drop Human piece in column
      score = minimax(board, depth-1, alpha, beta, true)
      remove piece
      best = min(best, score)
      beta = min(beta, best)
      if alpha >= beta: break  // prune
    return best
```

### Alpha-Beta Cutoff Condition

```
MAX node cutoff: if best_score >= beta → prune remaining siblings
MIN node cutoff: if best_score <= alpha → prune remaining siblings
```

Pruned branches are represented in the tree visualizer as **dashed dark placeholder nodes** — they are never expanded but are shown for completeness.

### Score Propagation

1. Terminal nodes or depth-limit nodes compute a **static score**.
2. Scores bubble upward: MAX nodes return the `max` of all children; MIN nodes return the `min`.
3. The root iterates over all legal first moves and picks the column whose child returned the **highest score**.
4. If the best score exceeds `50,000`, the AI has found a forced win and reports it as `∞WIN`.
5. If the best score is below `−50,000`, the AI is in a losing position and reports `−∞LOSS`.

---

## Performance Characteristics

- The AI runs **synchronously in a `setTimeout` callback** to avoid blocking the UI during the brief computation window. The thinking indicator is shown while this runs.
- At depth 5 on an empty board, the search can evaluate up to ~15,000 nodes in under 200 ms on a modern machine.
- The tree visualizer uses a **Knuth-style layout algorithm** (`assignX`) that assigns each leaf a sequential x-index, then centers parent nodes over their children. This avoids node overlap in most practical cases.
- Canvas rendering uses `bezierCurveTo` for edges and `arc` for nodes, with hit-testing done via a stored `nodeHits` array (world coordinates, checked on every mouse move).

---

## Known Limitations

- **No transposition table** — the same board state may be evaluated multiple times if reached via different move sequences. Adding memoization would dramatically speed up deeper searches.
- **No iterative deepening** — the AI searches to a fixed depth. Iterative deepening with time limits would allow adaptive difficulty.
- **Mobile layout not fully optimized** — the three-column grid is designed for laptop/desktop widths.
- **No undo move** — once a disc is placed, it cannot be retracted.
- **Session-only state** — scores and game history reset on page refresh (no localStorage persistence).
- **Single-threaded** — the AI runs on the main thread. At depth 5+, there may be a brief UI freeze. A Web Worker implementation would eliminate this.

---

## Browser Compatibility

Tested and working in:

| Browser | Minimum Version |
|---|---|
| Chrome / Edge | 80+ |
| Firefox | 75+ |
| Safari | 14+ |
| Opera | 67+ |

Requires: ES6+, Canvas 2D API, CSS Custom Properties, CSS Grid.

---

## File Structure

```
c4.html                  ← The entire application (single file)
├── <head>
│   ├── Google Fonts CDN links
│   └── <style>          ← All CSS (CSS variables, layout, animations)
├── <body>
│   ├── .stars           ← Background decorative element
│   ├── <header>         ← Title and subtitle
│   ├── .layout          ← Three-column grid
│   │   ├── Left panel   ← AI config, heuristic table
│   │   ├── Center       ← Score bar, board, status, buttons
│   │   └── Right panel  ← Move log, score graph, minimax trace
│   ├── #overlay         ← Win/draw modal
│   ├── #treePage        ← Full-screen tree visualizer
│   └── <script>         ← All game logic and rendering (~600 lines JS)
│       ├── Game Engine  ← Board, win detection, piece dropping
│       ├── Heuristic    ← evalBoard, scoreWindow
│       ├── Minimax      ← minimaxRec, getBestMove, markChosen
│       ├── Game Flow    ← humanMove, doAIMove, resetGame
│       ├── Rendering    ← renderBoard, renderColButtons, updateScoreGraph
│       └── Tree Vis     ← layout, renderTreeCanvas, drawNode, pan/zoom
```

---

## Customization Guide

### Change board size

Modify the constants at the top of the `<script>`:

```js
const ROWS = 6;  // change to 7, 8, etc.
const COLS = 7;  // change to 8, 9, etc.
```

Also update `.board-grid` in CSS: `grid-template-columns: repeat(7, 1fr)` → match your new `COLS`.

### Change AI depth range

Edit the `<input>` slider in the HTML:

```html
<input type="range" id="depthSlider" min="2" max="5" value="4" ...>
```

Change `max` to `6` or `7` for deeper (slower) search.

### Adjust heuristic weights

In the `scoreWindow` function:

```js
if (ai === 4) s += 100;          // four in a row — change to 1000 to make AI more aggressive
else if (ai === 3 && em === 1) s += 10;
else if (ai === 2 && em === 2) s += 5;
if (op === 3 && em === 1) s -= 10; // blocking weight
```

### Change color theme

All colors are CSS custom properties in `:root`:

```css
:root {
  --blue: #1a3a6b;
  --gold: #f5a623;
  --red: #e53e3e;
  --yellow: #f6e05e;
  --bg: #0a0f1e;
  /* ... */
}
```

---

## Glossary

| Term | Definition |
|---|---|
| **Minimax** | A recursive decision algorithm for two-player zero-sum games that alternates between maximizing and minimizing players |
| **Alpha (α)** | The best score the maximizing player (AI) is guaranteed in the current search path |
| **Beta (β)** | The best score the minimizing player (Human) is guaranteed in the current search path |
| **Alpha-Beta Pruning** | An optimization that skips branches where α ≥ β, as they cannot affect the final decision |
| **Depth** | The number of half-moves (plies) the AI looks ahead |
| **Ply** | One half-move — either a single AI move or a single human move |
| **Heuristic** | An estimation function that scores a non-terminal board position |
| **Terminal node** | A board state where the game has ended (win, loss, or draw) |
| **Pruned branch** | A subtree that was skipped because it is provably irrelevant |
| **Node** | A single board state in the search tree |
| **Branching factor** | The average number of valid moves at each node (typically 4–7 in Connect Four) |
| **Window** | A sequence of 4 consecutive cells evaluated by the heuristic |
| **MAX layer** | A depth level where the AI chooses the move with the highest score |
| **MIN layer** | A depth level where the human (simulated as adversary) chooses the move with the lowest score |
| **Transposition** | The same board state reached via different sequences of moves |

---

*Built with vanilla HTML, CSS, and JavaScript. No dependencies. No build tools.*