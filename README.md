# Open Terminal

A free, open-source, Bloomberg-style research terminal that runs locally on your Mac.
Dense, keyboard-first, and built only on **free public data** — no subscriptions, no paid APIs, no AI.
**Research only: no trading, not investment advice.**

Commands: quotes (`Q`), description & valuation (`DES`), watchlist (`W`), news (`N`, `CN`),
financial statements (`FA`), comps (`COMP`), SEC filings (`CF`), macro dashboard (`ECO`),
research briefs (`BRIEF`) and saved briefs (`REP`).

---

## Download (no setup)

Grab the file for your system from the **Releases** page of this repository and double-click it:

| System | File |
|---|---|
| Mac (Apple Silicon: M1–M5) | `OpenTerminal-macOS-arm64.zip` → unzip → `Open Terminal.app` |
| Mac (Intel) | `OpenTerminal-macOS-x64.zip` |
| Windows | `OpenTerminal-Windows.exe` |
| Linux | `OpenTerminal-Linux` (`chmod +x` it first) |

The app is unsigned, so the first launch shows a warning: on macOS **right-click → Open → Open**; on Windows
**More info → Run anyway**. It needs an internet connection and pulls live data directly from each source.

Optional keys: click **SETTINGS** (top right) to add a free FRED key and your SEC name/email. No files to edit;
they're stored only on your computer (`~/Library/Application Support/Open Terminal/` on a Mac).

## Build the app yourself

```bash
make app          # -> dist/Open Terminal.app (macOS) or dist/OpenTerminal(.exe)
```
Releases are built automatically for macOS, Windows and Linux by `.github/workflows/release.yml`
whenever a version tag is pushed: `git tag v1.0.0 && git push origin v1.0.0`.

## Setup (Mac)

1. **Python.** Python 3.11+ is recommended. The code also runs on the Python 3.9 that ships with macOS,
   so you can start right away. To upgrade later:
   ```bash
   # with Homebrew (https://brew.sh)
   brew install python@3.12
   ```
   `make` picks the newest `python3.x` it finds. If you upgrade, delete `.venv/` so it is rebuilt.

2. **Run it**
   ```bash
   cd ~/Desktop/Terminal
   make dev
   ```
   The first run creates `.venv/`, installs dependencies, copies `.env.example` → `.env`,
   starts the server with auto-reload and opens **http://127.0.0.1:8765**.

   Without `make`: `python3 -m venv .venv && .venv/bin/pip install -r requirements.txt && .venv/bin/python run.py --open`

3. **Optional keys** — easiest: click **SETTINGS** in the app (saved instantly, no restart). Or put them in `.env`:
   | Setting | What it unlocks | How to get it |
   |---|---|---|
   | `SEC_USER_AGENT` | `CF` and the SEC check in `FA` | No account — just your name + email, e.g. `SEC_USER_AGENT="Jane Doe jane@uni.edu"`. SEC asks every tool to identify itself |
   | `FRED_API_KEY` | US tiles in `ECO` | Free account at https://fredaccount.stlouisfed.org → API Keys. Euro/ECB data needs no key |

   `.env` is git-ignored. Never commit it.

### Other commands
```
make run          # without auto-reload
make test         # unit tests for the data parsers (no network)
make smoke        # live end-to-end check of every feature against the running server
make clean-cache  # wipe cached market data (keeps your watchlist)
```

---

## Commands

| Command | What it does |
|---|---|
| `Q <TICKER> [1D\|1M\|1Y\|5Y]` | Quote + price chart. Typing just a ticker does the same. |
| `1D` / `1M` / `1Y` / `5Y` | Switch the range of the chart on screen |
| `DES <TICKER>` | Description, profile, **EV bridge**, P/E, EV/EBITDA, EV/Revenue, P/B |
| `W add <T1> <T2>…` / `W remove <T>` | Edit the watchlist (also `W rm`, or click × on a row) |
| `N <query>` | News search (Google News). Operators work: `"exact phrase"`, `site:ft.com`, `when:1d`, `OR` |
| `N` | Market headlines (FT, Reuters, WSJ, CNBC, MarketWatch) |
| `CN <TICKER>` | Company news, searched by company name |
| `FA <TICKER>` | Income statement, balance sheet, cash flow — last 4 fiscal years, with margins, growth, net debt, FCF. US and SEC-registered foreign companies get an **SEC XBRL check** |
| `FA <TICKER> Q` | Last 5 quarters + TTM (P&L and cash flow summed over 4 quarters, balance sheet latest); growth y/y vs the same quarter. Half-yearly reporters (many EU companies) get a clear message |
| `DCF <TICKER> [g= margin= tax= wacc= tg= erp=]` | 5-year unlevered FCF DCF with Gordon-growth terminal value, CAPM WACC, equity bridge, WACC × g sensitivity. Defaults come from the company's own history; edit any assumption in place and press Enter |
| `COMP <T1> <T2> … [CCY]` | Comps table: mkt cap, EV, revenue, EBITDA, margin, EV/Rev, EV/EBITDA, P/E, fwd P/E + median/mean/high/low. Optional display currency at the end (`… USD`) |
| `ECO` | Macro dashboard: ECB deposit & refi rate, Fed funds target & effective, €STR, euro 10Y (AAA), US 2Y/10Y, 2s10s, EUR/USD, EUR/GBP, EUR/CHF, euro-area HICP & core, US CPI & core, US unemployment — each with change vs previous and vs 1 year, sparkline and date |
| `ECO <ID>` | 5-year chart of one series, e.g. `ECO UST10`, `ECO EAHICP`, `ECO EURUSD` (click any tile) |
| `BRIEF <TICKER> [PEERS…]` | Research brief built only from the terminal's data: Summary (3 bullets) → Key facts (every number with a numbered source) → What it means (rule-based signals) → Risks / what's uncertain → Sources (links + timestamps). Add peers for a premium/discount vs the peer median, e.g. `BRIEF SU.PA SIE.DE ABBN.SW ETN` |
| `REP` / `REP <#>` | Saved briefs: list, reopen, export to Markdown, delete |
| `CF <TICKER>` | SEC filings (10-K, 10-Q, 20-F, 6-K, 8-K, proxies) with links; filter by ANNUAL / QUARTERLY / EVENTS / PROXY |
| `HELP` | All commands and shortcuts |

Bloomberg order works too: `SU.PA DES`, `AAPL Q 5Y`, `SAN.MC CN`, `SHEL.L FA`.

**News panel (right):** follows whatever ticker is in focus (`Q`, `DES`, `CN`, or clicking the watchlist);
the **MARKETS** tab switches to general market headlines. Refreshes every 5 minutes.

**Command palette:** start typing a company name, ticker or command — suggestions drop down *under the command
line* (it never floats over the screen): a pre-selected top hit with one-step actions (DES / FA / BRIEF / CN), your
watchlist, other listings with last price and day change, commands, and saved briefs. `schneider` + Enter opens
Schneider Electric; `DES erst` completes to `DES EBS.VI`. Click into the empty box (or press ↓) for recent commands.

**Keyboard:** `/` focus command line (typing anywhere also works) · `Enter` run · `Esc` clear ·
`↑`/`↓` history · `Tab` complete mnemonic · `Alt+↑/↓` previous/next watchlist ticker.

**Tickers** use Yahoo symbols with exchange suffixes: `AAPL`, `SU.PA` (Paris), `SAN.MC` (Madrid),
`EBS.VI` (Vienna), `SLYG.DE` (XETRA), `SHEL.L` (London, quoted in **pence**), `^GSPC`, `^STOXX50E`, `EURUSD=X`.

**Futures:** Yahoo uses `NQ=F`; TradingView-style `NQ1!` is translated automatically (`ES1!`, `YM1!`, `RTY1!`,
`CL1!`, `GC1!`, `ZN1!`, `6E1!`…). Yahoo only has the front month, and no Eurex contracts (`FDAX1!`/`FESX1!` →
use `^GDAXI` / `^STOXX50E`).

---

## How the numbers are built

- **Every figure is labelled** with its source and fetch time. `cached` means it was served from SQLite.
  `STALE` means the source is down and you are seeing the last good value.
- **EV = market cap + total debt − cash.** It is computed by the terminal, not taken from Yahoo:
  - Market cap and P/E are rescaled to the latest price, because company data is cached for 6 h.
  - If a company **trades in one currency but reports in another** (e.g. Shell: GBp vs USD), market cap is
    converted at spot FX *before* adding debt, so the whole bridge is in the reporting currency.
    Yahoo's own `enterpriseValue` sometimes skips this step. It is shown greyed out for comparison.
  - London prices in pence (GBp) are normalised to pounds for market cap.
  - For **banks/insurers** the terminal warns that EV multiples are not meaningful.
  - Negative EBITDA or earnings → shown as `n.m.`
- Debt, cash, EBITDA and revenue in `DES` come from Yahoo's summary data (TTM / most recent quarter); `FA`
  shows the full annual statements.
- Yahoo prices can be delayed 15–20 min depending on the exchange.

### FA, COMP and the SEC check
- **FA** shows fiscal years oldest → newest, in millions of the *reporting* currency, negatives in (parentheses).
  Italic rows are computed: growth, margins, effective tax rate, net debt (same definition as the EV bridge),
  net debt / EBITDA, FCF = operating cash flow + capex, FCF margin, FCF / net income.
  Rows that don't apply (gross profit for a bank) are dropped, and banks/insurers get a warning.
- **SEC XBRL check:** for companies that file with the SEC — US 10-K filers *and* foreign issuers filing 20-F
  (Shell, SAP, Banco Santander…) — revenue, net income, diluted EPS, operating cash flow and total assets are
  compared with the figures tagged in the actual filings. ✓ = within 0.5%. A gap is usually a definition difference
  (e.g. Santander: Yahoo's net income to *common* shareholders excludes AT1 coupons; the filing's is to all owners).
  European tickers are matched to EDGAR by company name, never by ticker prefix (`SU.PA` ≠ Suncor).
- **COMP** uses exactly the same EV as `DES`. Sizes are converted into one display currency (default: first
  company's reporting currency) at spot FX; multiples are never converted. Negative EBITDA/earnings are n.m. and
  excluded from median/mean.

### BRIEF (research brief, no AI)
- Built by fixed, readable rules on the same data every other screen shows — nothing is generated or estimated,
  so every number can be traced to its source (`[n]` links to the numbered source list; click it to jump there).
- Rules and thresholds live at the top of `backend/brief.py` (e.g. net debt / EBITDA > 3x = high leverage,
  revenue growth ≥ 10% = strong, ±15% vs peer median = premium/discount). Change them to match your own views;
  each signal in the brief states the rule that produced it.
- The peer median **excludes** the company itself. Banks/insurers skip EBITDA/FCF rules and use ROE instead.
- Indices, futures and FX get a price/performance brief (no company fundamentals).
- Every brief is saved in SQLite (`REP`) and can be exported as Markdown with a full source list.

### Look & feel
Designed to feel like a real terminal app (Bloomberg / Claude Code / btop): one monospace font at one size on a
fixed line grid (the Mac's own SF Mono/Menlo, no web fonts), rounded box panels with titles in the border,
emphasis only through bold, colour and inverse video, `[1D]`-style text buttons, block-character charts and
sparklines, and a braille spinner as the only animation. Colours are semantic: cream = key values, tan =
focus/accent, green/red = moves, ochre = warnings, grey = secondary.

### DCF
- UFCF = EBIT × (1 − t) + D&A − capex − ΔNWC; revenue growth starts at the historical CAGR (capped −5…+15%) and fades
  linearly to terminal growth by year 5; EBIT margin held at the latest year; tax, D&A, capex and NWC at historical averages.
- WACC = CAPM cost of equity (rf = US 10Y via FRED for USD reporters, euro AAA 10Y via ECB for EUR, ERP 5.5%) and the
  company's own cost of debt (interest / total debt), weighted at market value. Implausible Yahoo betas are bounded to 0.5–2.0.
- Equity = EV − net debt − minorities, ÷ diluted shares, converted into the trading currency (handles GBp and reporting ≠
  trading currency). Warnings flag terminal value > 85% of EV and implied values more than ±50% from the market.
- Banks and insurers are refused (FCF DCF doesn't apply). A screening model, not a price target.

### ECO (macro)
- **ECB Data Portal** (no key): policy rates, €STR, AAA euro yield curve 10Y, ECB reference FX rates (set ~14:15 CET),
  euro-area HICP. Note: euro-area inflation moved to the new `HICP` dataflow in 2026 — the old `ICP` series are frozen at
  Dec 2025. The terminal uses the new series.
- **FRED** (free key): Fed funds target range & effective rate, Treasuries (2Y, 10Y, 2s10s), CPI and core CPI (y/y computed
  from the index, matched by calendar month), unemployment.
- Every tile shows the **date of the latest observation**. A tile is flagged **STALE** if its series stops updating
  (>10 days for daily data, >80 days for monthly) — so a discontinued series can never pass as current.
- Rate changes in basis points, inflation changes in percentage points, FX changes in %. Tile/series codes link back to
  the ECB/FRED source page in `ECO <ID>`.

### News sources
- **Company news** searches Google News by *company name*, not ticker (`SU.PA` finds nothing,
  `"Schneider Electric"` finds everything). Legal suffixes are stripped (`Banco Santander, S.A.` → `"Banco Santander"`);
  one-word names like Shell/Apple get investor terms added so you don't get "Old Shell Road".
- **Market headlines** merge FT Markets, CNBC and MarketWatch RSS with Reuters and WSJ via Google News
  (`site:` search). Reuters retired public RSS in 2020 and WSJ's public feeds stopped updating in Jan 2025.
  Any feed whose newest item is over 7 days old is flagged as stale rather than shown.
- Duplicate stories across outlets are merged; auto-generated "stock price" pages are dropped.
- Feed XML is parsed with `defusedxml` (safe against malicious feeds).

### Cache TTLs
| Data | TTL |
|---|---|
| Quotes | 60 s |
| Intraday chart (1D) | 60 s |
| 1M / 1Y / 5Y charts | 15 min / 1 h / 6 h |
| Company profile & fundamentals | 6 h |
| FX rates (for EV conversion) | 1 h |
| News (search, company, market) | 5 min |
| Annual statements (FA) | 24 h |
| SEC ticker map / XBRL facts | 24 h |
| SEC filings list | 1 h |
| Macro series (ECB, FRED) | 1 h |

At most 4 parallel requests go to Yahoo, and each has a 15 s timeout. Identical concurrent requests are
merged into one.

---

## Project layout

```
run.py                  entry point (uvicorn)
Makefile                make dev / test / clean-cache
backend/
  config.py             settings from .env
  db.py                 SQLite: cache, watchlist
  cache.py              TTL cache, single-flight, stale fallback
  errors.py             error types → HTTP status codes
  main.py               FastAPI routes (/api/...) + serves the frontend
  sources/
    status.py           per-source health for the status bar
    yahoo.py            yfinance: quotes, history, profile, FX, EV maths
    news.py             Google News search + market RSS feeds
    edgar.py            SEC EDGAR: CIK lookup, filings, XBRL company facts
    fred.py             FRED API (US macro)
    ecb.py              ECB Data Portal (euro rates, FX, HICP)
  macro.py              ECO series list, y/y, changes, staleness, dashboard
  brief.py              BRIEF: rule-based research brief + Markdown export
  analytics.py          shared valuation (DES/COMP), comps stats, SEC reconciliation
frontend/
  index.html            layout
  styles.css            terminal theme
  app.js                command bar, views, SVG chart, watchlist
tests/
  test_yahoo_parsers.py parser + valuation tests (no network)
  test_news_parsers.py  RSS parsing, dates, dedupe, query building
  test_phase3.py        statements, EDGAR parsers, comps, reconciliation
  test_brief.py         BRIEF rules, formatting, Markdown export
  test_macro.py         FRED/ECB parsers, y/y, change units, staleness
scripts/
  smoke.py              live end-to-end check of every endpoint (make smoke)
data/                   SQLite DB (git-ignored, created on first run)
```

### API (for poking around)
`GET /api/quote/{sym}` · `GET /api/history/{sym}?range=1Y` · `GET /api/describe/{sym}` ·
`GET|POST /api/watchlist` · `DELETE /api/watchlist/{sym}` · `GET /api/news/search?q=` ·
`GET /api/news/ticker/{sym}` · `GET /api/news/market` · `GET /api/financials/{sym}` ·
`GET /api/search?q=` · `GET /api/quotes?symbols=A,B` · `GET /api/comps?symbols=A,B,C&ccy=EUR` · `GET /api/filings/{sym}` · `GET /api/macro` · `GET /api/macro/{id}` ·
`GET /api/status`. Interactive docs: `/docs`.

---

*Data from free public sources, provided as-is and possibly delayed or wrong. For learning and research.
Not investment advice.*

## License

**PolyForm Noncommercial 1.0.0**: free for personal and other noncommercial use (learning, research,
hobby projects). Commercial use is not permitted. See [LICENSE](LICENSE). Copyright (c) 2026 L.K.R.U.
