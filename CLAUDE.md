# CLAUDE.md — Meine-Finanzen

## Project Overview

**Meine-Finanzen** is a personal finance Progressive Web App (PWA) built as a **single HTML file** (`index.html`). It tracks monthly income/expenses and an investment portfolio, stores all data locally in the browser via `localStorage`, and requires no build step or backend.

- **Language:** German (UI labels, categories, and documentation)
- **Deployment:** Single static file — open `index.html` directly in a browser or serve it from any static host
- **Dependencies:** Only Chart.js 4.4.1 (loaded via CDN)
- **Data storage:** 100% client-side (`localStorage`), no network requests, no analytics

---

## Architecture

The entire application lives in `index.html` (~692 lines), structured in three embedded sections:

```
index.html
├── <style>      — All CSS (lines ~12–120)
├── <body>       — All HTML markup (lines ~122–330)
└── <script>     — All JavaScript logic (lines ~332–690)
```

### Pages (Tab-based navigation)

| Page ID            | Label     | Purpose                                      |
|--------------------|-----------|----------------------------------------------|
| `#page-dashboard`  | Dashboard | KPI overview, monthly trend chart, pie chart |
| `#page-finanzen`   | Finanzen  | Monthly income/expense data entry            |
| `#page-depot`      | Depot     | Investment portfolio management              |
| `#page-settings`   | Settings  | JSON backup/restore, CSV import/export       |

Navigation is handled by `showPage(id)`, which toggles the `.active` class on `.page` elements.

---

## State Management

All application state is held in a single `state` object in memory and persisted to `localStorage` under the key `finanzapp_<year>`.

```js
state = {
  income:    number[12][4],   // monthly × 4 income categories
  expenses:  number[12][12],  // monthly × 12 expense categories
  depot:     DepotItem[],     // investment portfolio positions
}
```

### Persistence functions
- `saveState()` — serializes state to `localStorage`
- `loadState()` — deserializes state from `localStorage` (called on startup and year change)

Data is isolated per year (2024–2027). Switching years calls `loadState()` with the selected year.

---

## Data Categories

### Income (4 categories, index order matters)
0. Gehalt/Lohn
1. Nebeneinkommen
2. Kapitalerträge
3. Sonstige Einnahmen

### Expenses (12 categories, index order matters)
0. Miete/Hypothek
1. Nebenkosten
2. Lebensmittel
3. Transport
4. Versicherungen
5. Telefon & Internet
6. Freizeit
7. Kleidung
8. Gesundheit
9. Essen gehen
10. Abonnements
11. Sonstiges

> **Important:** The numeric indices for categories are baked into the data arrays. Do not reorder or insert categories without migrating existing localStorage data.

### Depot position schema
```js
{
  name:     string,   // Position name / ticker
  type:     string,   // "Aktie" | "ETF" | "Krypto" | "Anleihe" | "Sonstiges"
  sector:   string,   // "Technologie" | "Finanzen" | "Gesundheit" | "Energie"
                      // | "Konsum" | "Industrie" | "Immobilien" | "Sonstiges"
  shares:   number,   // Number of shares/units held
  buyPrice: number,   // Average purchase price per unit (EUR)
  curPrice: number,   // Current price per unit (EUR)
}
```

---

## Key Functions

| Function                | Description                                          |
|-------------------------|------------------------------------------------------|
| `showPage(id)`          | Navigate between the four pages                      |
| `renderDashboard()`     | Calculates KPIs and renders all dashboard charts     |
| `renderMonthPage()`     | Renders income/expense inputs for the selected month |
| `renderDepot()`         | Renders portfolio table and allocation chart         |
| `saveDepotItem()`       | Saves a new or edited depot position from the modal  |
| `openDepotModal(idx)`   | Opens edit modal; pass `null` to add a new position  |
| `exportData()`          | Downloads full state as a JSON backup file           |
| `importData()`          | Restores state from a JSON backup file               |
| `exportCSV()`           | Exports monthly transaction summary as CSV           |
| `importCSV()`           | Imports transactions from a CSV file                 |
| `destroyChart(key)`     | Destroys an existing Chart.js instance before re-rendering |
| `fmt(value)`            | Formats a number as German locale currency string    |
| `fmtShort(value)`       | Compact currency format (e.g. "1,2k")               |

---

## Charts

All Chart.js instances are stored in a global `charts` object and destroyed via `destroyChart(key)` before re-rendering to avoid canvas reuse errors.

| Key               | Type | Location              |
|-------------------|------|-----------------------|
| `dashBar`         | Bar  | Dashboard — monthly trend |
| `dashPie`         | Pie  | Dashboard — expense breakdown |
| `monthBar`        | Bar  | Finanzen — month overview |
| `depotPie`        | Pie  | Depot — allocation    |
| `depotPerf`       | Bar  | Depot — performance   |

---

## Styling Conventions

- CSS custom properties are defined on `:root` (dark theme by default)
- Primary accent color: `#6c63ff` (purple)
- Layout uses CSS Grid and Flexbox
- Mobile-first; bottom navigation bar mimics native app UX
- Safe area insets (`env(safe-area-inset-*)`) handle notched devices

Do not introduce external CSS frameworks or preprocessors — keep everything inline.

---

## Development Workflow

Since there is no build toolchain:

1. **Edit** `index.html` directly
2. **Test** by opening the file in a browser (or using a local static server, e.g. `python3 -m http.server`)
3. **Commit** changes with descriptive messages
4. **Push** to the appropriate feature branch

No `npm install`, no compilation, no bundling required.

### Running locally
```bash
# Option A — open directly
open index.html

# Option B — serve via Python
python3 -m http.server 8080
# Then visit http://localhost:8080
```

---

## PWA Details

The app is installable to a device home screen:
- The `beforeinstallprompt` event is captured and an install banner is shown
- Manifest metadata is embedded in the `<head>` of `index.html`
- There is no service worker implementation yet; offline caching is not implemented

---

## Conventions to Follow

1. **No build tooling** — Keep the project as a single HTML file. Do not introduce npm, webpack, vite, or other build systems unless explicitly requested.
2. **German language** — All UI text, category names, and user-facing strings should remain in German.
3. **Preserve category indices** — Never reorder income or expense categories without a migration strategy for existing `localStorage` data.
4. **Destroy charts before re-render** — Always call `destroyChart(key)` before creating a new Chart.js instance for a given canvas.
5. **localStorage key format** — Use `finanzapp_<year>` (e.g. `finanzapp_2025`) for all state persistence.
6. **No external state or network** — This app intentionally makes no network requests. Keep all logic client-side.
7. **Formatting** — Use `fmt()` for currency display. Number inputs store raw floats; format only for display.
8. **Modal pattern** — Use the existing modal (`#depot-modal`) pattern for any new editing dialogs.

---

## File Map

```
Meine-Finanzen/
├── index.html      # Entire application (HTML + CSS + JS)
├── README.md       # Minimal project description (German)
└── CLAUDE.md       # This file
```
