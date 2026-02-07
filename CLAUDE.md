# CLAUDE.md

## Project Overview

Bitcoin Bottom Indicator Dashboard — a single-page, zero-dependency web application that provides real-time Bitcoin market cycle analysis with buy/sell scoring, trend trading signals, and technical indicators. Runs entirely in the browser as one HTML file.

## Repository Structure

```
BTC/
├── index.html   # Entire application (HTML + CSS + JS, ~1160 lines)
└── README.md    # Project documentation
```

This is a **monolithic single-file architecture**. All CSS is embedded in a `<style>` tag and all JavaScript in a `<script>` tag within `index.html`.

## Tech Stack

- **Vanilla JavaScript (ES5)** — no frameworks, no transpilation
- **Embedded CSS3** — dark theme, responsive grid/flexbox layout
- **SVG** — hand-built charts (no charting library)
- **Zero external dependencies** — no npm packages, no CDN libraries
- **No build step** — open `index.html` directly in a browser

## Architecture

### Data Flow

```
API Sources (6 parallel fetches)
    ↓
Cache Layer (localStorage with TTL)
    ↓
State Object (global `state`)
    ↓
Calculation Engine (scoreBuy, scoreSell, calcMarketCycle)
    ↓
render() → innerHTML replacement
    ↓
Browser DOM
```

### Code Organization (within `index.html <script>`)

| Section | Lines (approx) | Purpose |
|---------|----------------|---------|
| Cache system (`C`) | 158–163 | localStorage-backed cache with TTL and stale-data fallback |
| Utility functions | 165–168 | `esc()` for XSS protection, `fj()` promise-based fetch with timeout |
| API fetch functions | 170–283 | `fetchPrice`, `fetchMA`, `fetchFG`, `fetchFR`, `fetchHR`, `fetchDaily` |
| Technical analysis | 285–534 | SMA, EMA, RSI, MACD, ATR, trend structure, Golden/Death Cross detection |
| Market cycle calc | 536–586 | `calcPriceDeviation`, `calcMarketCycle` — composite 0–100 cycle gauge |
| Buy scoring engine | 588–652 | `scoreBuy()` — 4 uncorrelated indicators, 100 pts max |
| Sell scoring engine | 654–713 | `scoreSell()` — 4 uncorrelated indicators, 100 pts max |
| Action signals | 715–779 | `getAction()` — maps cycle position to HEAVY BUY through HEAVY SELL |
| SVG chart builder | 785–803 | `buildChart()` — 52-week price chart with 200WMA overlay |
| Render engine | 811–1139 | `render()` — single function rebuilds entire DOM |
| Init & refresh | 1141–1159 | `fetchAll()`, auto-refresh intervals (30s price, 60s full) |

### Key Design Decisions

- **4 uncorrelated buy indicators** (Price Deviation 35pts, Hash Ribbons 30pts, Market Sentiment 20pts, Funding Rate 15pts) — deliberately avoids multicollinearity
- **4 uncorrelated sell indicators** (Price Deviation 35pts, Market Mania 30pts, Momentum/RSI 20pts, Leverage 15pts)
- **Market Cycle Gauge** (0–100): 0 = extreme fear/buy, 100 = extreme euphoria/sell
- **Cascading API fallbacks**: each data source has 2–3 fallback providers plus cached data
- **MVRV Z-Score and Realized Price are estimates** based on Price/200WMA ratio (not on-chain data)

## External API Dependencies

All APIs are free with no authentication required.

| API | Endpoint | Data | Cache TTL |
|-----|----------|------|-----------|
| Binance | `/api/v3/ticker/price` | BTC/USDT price | 1 hour |
| Binance | `/api/v3/klines` | Weekly + daily candles | 24 hours / 1 hour |
| Binance Futures | `/fapi/v1/fundingRate` | Funding rates | 5 minutes |
| CoinGecko | `/coins/bitcoin/market_chart` | 1400-day weekly history | 24 hours |
| Alternative.me | `/fng/?limit=1` | Fear & Greed Index | 24 hours |
| Mempool.space | `/api/v1/mining/hashrate/1y` | 1-year hashrate | 1 hour |
| Blockchain.info | `/ticker` | BTC price (fallback) | 1 hour |

## Development Workflow

### Running Locally

Open `index.html` in any modern browser. No server, build step, or installation required.

### Deploying

Push to GitHub and enable GitHub Pages pointing to the branch containing `index.html`.

### Testing

There is no automated test suite. Validate changes by:
1. Opening `index.html` in a browser
2. Checking the browser console for errors
3. Verifying all 6 data sources load (check the footer for source attribution)
4. Confirming the Market Cycle gauge, buy/sell scores, and trend panels render correctly
5. Testing responsive layout at mobile widths (< 900px breakpoint)

### No Build System

There is no `package.json`, linter, formatter, or CI/CD pipeline. The file is edited directly.

## Code Conventions

- **Variable naming**: short/terse names throughout (`p` = price, `ma` = moving average, `fg` = Fear & Greed, `fr` = funding rate, `hr` = hash rate, `C` = cache)
- **ES5 style**: `var` declarations, `function` keyword, `.then()/.catch()` chains (no async/await)
- **Semicolons**: used consistently
- **Comments**: section headers use `// ═══════ SECTION NAME ═══════` dividers
- **XSS protection**: all user-facing data passes through `esc()` before insertion into HTML
- **Error handling**: every fetch function has a `.catch()` that falls back to cached or default data — the dashboard should never crash

## Important Caveats for AI Assistants

1. **Single-file constraint**: all code lives in `index.html`. Do not split into multiple files unless explicitly asked.
2. **No dependencies**: do not introduce npm packages, CDN scripts, or build tooling unless explicitly requested.
3. **ES5 compatibility**: do not use `let`/`const`, arrow functions, template literals, `async/await`, or other ES6+ syntax unless the user agrees to drop ES5 support.
4. **Estimated on-chain metrics**: MVRV Z-Score and Realized Price are mathematical approximations, not real on-chain data. Do not claim they are accurate — they are directionally useful at cycle extremes.
5. **API rate limits**: CoinGecko free tier has aggressive rate limiting. The code already handles this with long cache TTLs (24h) and retry logic. Avoid adding more frequent CoinGecko calls.
6. **DOM rendering**: the `render()` function replaces `innerHTML` of `#app` on every update. There is no virtual DOM or incremental update — the entire UI is rebuilt each cycle.
7. **Cache keys**: localStorage keys are prefixed with `b_` (e.g., `b_price`, `b_ma`). Changing key names will invalidate users' cached data.
8. **Financial disclaimer**: this is an educational tool. All outputs include disclaimers. Do not remove disclaimer text or present signals as financial advice.
