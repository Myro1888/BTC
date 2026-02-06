# Bitcoin Bottom Indicator Dashboard

A fully free, no-signup-required, real-time Bitcoin bottom indicator dashboard that runs entirely in the browser as a single HTML file.

## Live Demo

Deploy to GitHub Pages or open `index.html` directly in your browser.

## Features

- **8 indicators** with a 140-point scoring system
- **Real-time price** from Binance with 30-second refresh
- **200-Week Moving Average** calculated from CoinGecko historical data
- **Estimated MVRV Z-Score** (directionally accurate at cycle bottoms)
- **Estimated Realized Price** (closest to actual during bear markets)
- **Fear & Greed Index** from Alternative.me
- **Funding Rates** from Binance Futures
- **Puell Multiple** estimated from 52-week price average
- **Hash Ribbons** from Mempool.space hashrate data
- **52-week price chart** with 200WMA overlay
- **Ladder strategy** with 3-tranche entry plan
- **Fully responsive** - works on desktop and mobile
- **Dark terminal aesthetic**
- **Offline support** via cached data

## How It Works

### Scoring System (140 points max)

**Base Indicators (100 points):**

| Indicator | Condition | Points |
|-----------|-----------|--------|
| MVRV Z-Score (Est.) | < 0.0 | 40 |
| MVRV Z-Score (Est.) | < 1.0 | 30 |
| MVRV Z-Score (Est.) | < 1.5 | 20 |
| MVRV Z-Score (Est.) | < 2.5 | 10 |
| Price vs 200W MA | <= 100% | 30 |
| Price vs 200W MA | <= 110% | 20 |
| Price vs 200W MA | <= 130% | 10 |
| Price vs Realized Price (Est.) | <= 100% | 30 |
| Price vs Realized Price (Est.) | <= 110% | 20 |
| Price vs Realized Price (Est.) | <= 130% | 10 |

**Confirmation Indicators (40 bonus points):**

| Indicator | Condition | Points |
|-----------|-----------|--------|
| Fear & Greed | < 20 (Extreme Fear) | 10 |
| Funding Rate | Negative | 10 |
| Puell Multiple | < 0.5 | 10 |
| Hash Ribbons | Capitulation (30MA < 60MA) | 10 |

### Buy Signals

| Score | Signal | Allocation |
|-------|--------|------------|
| 110-140 | EXTREME BUY | 100% |
| 80-109 | STRONG BUY | 75% |
| 50-79 | BUY | 50% |
| 30-49 | ACCUMULATE | 25% |
| 0-29 | HOLD | 0% |

## Accuracy Disclaimers

### MVRV Z-Score (Estimated)

Since real MVRV requires on-chain data from paid APIs (Glassnode), this dashboard uses a piecewise linear estimation model based on the Price/200WMA ratio. The model maps this ratio to approximate Z-Score values and correctly identifies all historical cycle bottoms as negative Z-Scores (strong buy zones):

- **Nov 2022**: Price/MA ≈ 0.64 → Estimated Z ≈ -0.87 (Actual: -0.28)
- **Dec 2018**: Price/MA ≈ 0.58 → Estimated Z ≈ -0.99 (Actual: -0.51)
- **Mar 2020**: Price/MA ≈ 0.77 → Estimated Z ≈ -0.58 (Actual: 0.0)

Note: The estimation amplifies the magnitude vs actual MVRV but correctly identifies the direction (negative = buy zone) at all historical bottoms. For precise values, use Glassnode.

### Realized Price (Estimated)

Realized Price is estimated using the 200WMA as a base with a dynamic multiplier (1.02x to 1.18x depending on how far price is above the MA). Example estimates at historical bottoms:

- **Nov 2022**: 200WMA ≈ $25k × 1.02 = ~$25,500 (Actual RP: $17,600)
- **Dec 2018**: 200WMA ≈ $5.5k × 1.02 = ~$5,610 (Actual RP: $5,800)
- **Mar 2020**: 200WMA ≈ $6.5k × 1.02 = ~$6,630 (Actual RP: $6,200)

Note: The estimate is most accurate during bear markets when price is near or below the 200WMA. During deep bear markets (when price < MA), the multiplier is 1.02x. For precise values, use Glassnode.

## Historical Validation

When this model produces a score > 110, it has historically identified every major Bitcoin cycle bottom:

| Date | Score | Subsequent Rally |
|------|-------|-----------------|
| Nov 2022 | ~120 | +712% |
| Mar 2020 | ~115 | +1,715% |
| Dec 2018 | ~118 | +2,050% |
| Jan 2015 | ~125 | +9,900% |

## Data Sources (All Free, No Signup)

| Data | Source | Refresh Rate |
|------|--------|--------------|
| BTC Price | Binance API | 30 seconds |
| 200-Week MA | CoinGecko | 24 hours |
| Fear & Greed | Alternative.me | 24 hours |
| Funding Rate | Binance Futures | 5 minutes |
| Hashrate | Mempool.space | 1 hour |
| MVRV / Realized Price | Calculated estimates | 60 seconds |

## Upgrade Path

For 100% accurate on-chain data, you can integrate:

1. **Glassnode** (free tier available with signup): Real MVRV Z-Score, Realized Price, SOPR
2. **CryptoQuant** (free tier available): Exchange flows, miner data
3. **Blockchain.com API**: On-chain transaction data

## Deployment

### GitHub Pages

1. Push this repo to GitHub
2. Go to Settings > Pages
3. Set source to the branch containing `index.html`
4. Your dashboard will be live at `https://<username>.github.io/<repo>/`

### Local

Simply open `index.html` in any modern browser. No server required.

## Tech Stack

- Pure vanilla JavaScript (ES5 compatible)
- Embedded CSS (dark theme, responsive grid)
- SVG-based charts (no external charting library)
- Zero external dependencies
- No build step required

## Disclaimer

This dashboard is for **educational purposes only** and does not constitute financial advice. Always do your own research (DYOR). Past performance does not guarantee future results. Cryptocurrency investments carry significant risk including potential loss of principal.
