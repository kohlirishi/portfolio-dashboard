# 📊 Portfolio Dashboard

A self-contained, browser-based portfolio tracking and short-term trading system for **US, TSX (Canada), and NSE (India)** markets — no build step, no server required.

Built and maintained by **Rishi & Pragya**.

---

## 🗂 What's Inside

| File | Purpose | Size |
|---|---|---|
| `index.html` | Main portfolio dashboard + short-term trading screener | ~15,700 lines |
| `stock-analyser-final-v34.html` | Individual stock technical analyser (8 SMA/EMA conditions) | ~1,100 lines |

Both files are **fully self-contained** — HTML + CSS + JS in a single file. No npm, no bundler, no backend server needed.

---

## 🚀 Getting Started

### Option A — Open directly (most features)
```bash
open index.html
```

### Option B — Local server (recommended for all features)
```bash
python3 -m http.server 8080
# Then visit: http://localhost:8080
```

### Option C — GitHub Pages
The dashboard is hosted via GitHub Pages. Access it directly at:
```
https://kohlirishi.github.io/portfolio-dashboard/
```

---

## ✨ Features

### 📈 Portfolio Dashboard (`index.html`)
- **Holdings table** — sortable, filterable, with column visibility toggles
- **P&L tracking** — per-person (Rishi / Pragya) and combined views
- **Diversification** — sector allocation donut + bar charts (Chart.js)
- **Sector heatmap** — live Yahoo Finance sector performance tiles
- **Crypto tracker** — Ethereum transaction history with buys/sells
- **Monthly performance** — expandable table from Google Sheets
- **Profit booking advisor** — AI-powered HARVEST / NO ACTION verdicts
- **Watchlist auto-scoring** — 8-condition SMA/EMA scoring with buy signal detection

### ⚡ Short Term Trading Portfolio (`index.html → Short Term Tab`)
- **Paper trading simulation** — $10,000 capital, $100/trade
- **ATH Breakout + Swing High screener** — scans US, TSX & NSE stocks
- **Regime gate** — entries blocked when SPY / XIU / NIFTY below 21-EMA
- **Bull Mode detection** — India VIX < 20 AND NIFTY above 21-EMA → wider targets (+3/+6/+9%), 2% trailing stop
- **Pending entries** — next-session confirmation before trade is placed
- **Live P&L** — real-time price updates with trailing stop monitoring
- **Exit targets** — Stage 1: +3%, Stage 2: +5%, Stage 3: +8% | Stop: −2% trailing
- **Auto screener** — runs daily at **8:00 AM & 1:00 PM EST** (weekdays, skips holidays)
- **VIX Fear Gauge** — live VIX with bull/bear classification
- **SPY Intraday** — 15-min ROC trend signal
- **Run log** — full audit trail of every screener run with regime + signal counts

### 🔔 Telegram Notifications
| Notification | Trigger | Condition |
|---|---|---|
| **Daily Scan Update** | 2:00 PM EST | 0 buy signals + 0 pending + SPY above 21-EMA + not a holiday |
| **EOD Summary** | 4:00 PM EST | Market day only (skips weekends + NYSE holidays) |
| **Stop Loss Alert** | Real-time | Trailing stop breached during live monitoring |
| **New Trade** | On entry | Position confirmed and placed |

### 🇮🇳 India (NSE) Support
- NIFTY 50 breakout monitoring
- India VIX tracking
- NSE-specific regime check (21-EMA)
- IST market hours: **9:15 AM – 3:30 PM IST** (3:45 – 10:00 AM ET)
- NSE holiday calendar built-in (2025–2028)

---

## 🏗 Architecture

### Data Flow
```
Page Load
  └── fetchAll()
        ├── Google Sheets (gviz/tq) → Holdings, P&L, Crypto
        ├── Yahoo Finance → Sector heatmap, VIX, India VIX, SPY intraday
        └── Twelve Data / Alpha Vantage → Stock quotes + technicals

Short Term Trading
  └── Google Apps Script Web App (server-side)
        ├── Screener runs (8 AM + 1 PM EST cron)
        ├── Pending entry confirmation
        ├── Live trade monitoring + trailing stop
        └── Telegram notifications
```

### Key Functions
| Function | Purpose |
|---|---|
| `fetchAll()` | Load all portfolio data from Google Sheets |
| `scoreStock()` | 8-condition SMA/EMA watchlist scoring (shared dashboard + analyser) |
| `checkRegime()` | SPY + XIU 21-EMA regime check |
| `checkRegime_India()` | NIFTY 21-EMA + India VIX regime check |
| `isNSEMarketOpen()` | Live NSE session check (IST hours + holidays) |
| `isUSMarketOpen()` | Live NYSE session check (ET hours + holidays) |
| `updateMarketClock()` | Nav badge: Market Open / 🇮🇳 India Market Open / Market Closed |
| `runScreenerNow()` | Trigger manual screener run via Apps Script |
| `fetchPortfolioState()` | Sync live trade state from Google Sheets backend |

---

## 🔌 Data Sources

| Source | Used For | Limit |
|---|---|---|
| **Google Sheets** (gviz/tq) | Holdings, P&L, Crypto, Screener state | Unlimited (published) |
| **Yahoo Finance** | Sector heatmap, VIX, SPY intraday, NIFTY | Unofficial — no key needed |
| **Twelve Data** | Time series, quotes | 800 req/day × 4 keys (auto-rotate) |
| **Alpha Vantage** | Stock overview, weekly/daily OHLCV | 25 req/day × 7 keys (auto-rotate) |
| **Google Apps Script** | Server-side screener, Telegram, trade state | Quota per account |

### API Key Management
Keys are embedded client-side with automatic round-robin rotation stored in `localStorage`.

```js
// Add custom Twelve Data keys:
localStorage.setItem('userTDKeys', JSON.stringify(['YOUR_KEY_1', 'YOUR_KEY_2']))

// Add custom Alpha Vantage keys:
localStorage.setItem('userAVKeys', JSON.stringify(['YOUR_KEY_1']))
```

---

## 🛠 Common Tasks

### Refresh portfolio data
```
Click ↻ Refresh  —or—  fetchAll()  in browser console
```

### Edit holdings
Modify the Google Sheet → data reloads automatically on next refresh.  
Sheet ID: `1LM0lfjSdnkZu7lYNPjCoGsORXphkxU742EqWnUcxM6g`

### Run screener manually
Dashboard → **Short Term Trading Portfolio** → **▶ Run Now** dropdown → choose market:
- `▶ US / TSX Screener`
- `🇮🇳 India Screener (NSE)`
- `▶ Both`

### Analyse a stock
Open `stock-analyser-final-v34.html` → type ticker → Enter  
Supports: NYSE, NASDAQ, TSX (append `.TO`), NSE (append `.NS`)

### Clear all trade data
Dashboard → Short Term Portfolio → **⚠ Data Management** → **🗑 Clear All Trade Data**

---

## 📋 Screener Rules

### Entry Criteria (all must pass)
| Filter | Rule |
|---|---|
| **Regime** | SPY / XIU / NIFTY above 21-EMA |
| **Signal type** | ATH Breakout (within 1% of ATH) or Swing High (EMA reclaim) |
| **Probability** | ≥ 50% composite score |
| **Signal freshness** | ATH: ≤ 5 trading days old · Swing: ≤ 7 days old |
| **Earnings** | No earnings within 7 days |
| **Volume** | Above-average on signal day |
| **Market conditions** | VIX, SPY ROC, same-day spike check |

### Exit Rules
| Stage | Target | Condition |
|---|---|---|
| Stage 1 | Sell 50% | +3% gain |
| Stage 2 | Sell 25% | +5% gain |
| Stage 3 | Sell 25% | +8% gain |
| Stop Loss | Sell 100% | −2% trailing stop |

### Auto-Screener Schedule
```
8:00 AM EST  — Morning scan (US + TSX pre-market setup)
1:00 PM EST  — Afternoon scan (mid-session confirmation)

Weekdays only · Skips NYSE holidays · No scan when regime is bearish
```

---

## 🌐 Market Status Badge

The nav badge updates every second:

| State | Badge | Condition |
|---|---|---|
| US/TSX open | `● Market Open` 🟢 | 9:30 AM–4:00 PM ET, weekday, no NYSE holiday |
| India open | `🇮🇳 India Market Open` 🟢 | 9:15 AM–3:30 PM IST (3:45–10:00 AM ET), weekday, no NSE holiday |
| Closed | `○ Market Closed` 🔴 | All other times |

---

## 📁 File Structure

```
portfolio-dashboard/
├── index.html                    # Main dashboard + short-term trading portfolio
├── stock-analyser-final-v34.html # Standalone stock technical analyser
├── README.md                     # This file
├── CLAUDE.md                     # AI coding assistant guidance
├── portfolio_pl.csv              # P&L export (not loaded at runtime)
├── crypto.csv                    # Ethereum transaction history export
└── mutual_funds.csv              # Mutual fund tracking export
```

---

## ⚙️ Google Apps Script Backend

The short-term trading portfolio uses a **Google Apps Script Web App** as its backend for:
- Running the stock screener (server-side, avoids CORS)
- Storing trade state in Google Sheets
- Sending Telegram notifications
- Executing cron-based daily scans

**GAS files (stored in Google Drive, deployed as Web App):**
- `App_Script.txt` — Main screener, trade management, Telegram bot, regime checks

To update the GAS backend: edit locally → paste into Apps Script editor → **Deploy → Manage deployments → New version**.

---

## 📝 Notes

- **No build step** — everything runs in the browser
- **Private data** — Google Sheet is published read-only via gviz endpoint; API keys are personal
- **Paper trading only** — all trades are simulations; no real money is moved
- **CAD/USD conversion** — TSX prices auto-converted at live rate via Frankfurter API
- **EMA periods** — Regime gate uses **21-EMA** for all markets (US, CA, IN) to match short-term screener intent
- **Holiday calendars** — NYSE (2024–2028) and NSE (2025–2028) built into the client; update annually
- **Telegram bot** — notifications suppressed on weekends, NYSE holidays, and when market regime is bearish with 0 signals

---

## 🔒 Security Note

API keys are embedded in client-side HTML. This repo should remain **private** or keys should be rotated periodically. Do not share the live URL publicly if you want to protect your API quotas.

---

*Last updated: May 2026*
