# calcvi-data

Static data repository for CalcVI 2.0 — Calculadora de Valor Intrínseco App (v2.6.0+).

Serves monthly JSON snapshots of:
- **IBOVESPA stocks** — top-discounted BR tickers ranked by intrinsic value (BG & FL formulas)
- **US stocks** — top-discounted US tickers ranked by the same formulas (REITs excluded from rankings, see [REIT Valuation](#reit-valuation))
- **All stocks** — complete catalog (BR + US) with fundamentals and quarterly historical multiples

Consumed by the CalcVI app on startup and throughout sessions to pre-populate ranked lists and stock data without requiring live API calls to fundamentals servers.

## Endpoints

**Brazil (IBOVESPA):**
```
https://nascimentolwtn.github.io/calcvi-data/top-descontos.json
https://nascimentolwtn.github.io/calcvi-data/stocks-all.json
```

**USA (OTC + NASDAQ):**
```
https://nascimentolwtn.github.io/calcvi-data/us-top-descontos.json
https://nascimentolwtn.github.io/calcvi-data/us-stocks-all.json
```

## Schemas

### Top-Descontos (Ranked Top 10)

**File:** `top-descontos.json` or `us-top-descontos.json`

```json
{
  "atualizadoEm": "2026-07-10 01:22",
  "topBG": [
    { "codigo": "BBDC4", "nome": "Banco Bradesco S.A.", "lpa": 1.2, "vpa": 8.5 }
  ],
  "topFL": [
    { "codigo": "WEGE3", "nome": "Weg S.A.", "lpa": 2.45, "vpa": 12.3 }
  ]
}
```

| Field | Description |
|---|---|
| `atualizadoEm` | Timestamp of last snapshot run (ISO 8601 with time) |
| `topBG[]` | Top 10 tickers ranked by BG (Graham) formula, descending by discount |
| `topFL[]` | Top 10 tickers ranked by FL (Lacerda) formula, descending by discount |
| `codigo` | Ticker symbol (B3 for Brazil, NASDAQ/OTC for US) |
| `nome` | Company name |
| `lpa` | Earnings per share (LPA in PT) from latest quarterly report |
| `vpa` | Book value per share (VPA in PT) from latest quarterly report |

**Note:** REITs are excluded from `us-top-descontos.json` rankings. Benjamin Graham and Lacerda formulas are designed for earnings-based valuation; real estate investment trusts use FFO-based (Funds From Operations) and dividend-based valuation models instead (see [REIT Valuation](#reit-valuation)).

### All Stocks (Complete Catalog)

**File:** `stocks-all.json` or `us-stocks-all.json`

#### General Schema

```json
{
  "atualizadoEm": "2026-07-10 01:22",
  "stocks": [
    {
      "codigo": "ABEV3",
      "nome": "Ambev S.A.",
      "lpa": 0.99,
      "vpa": 5.779,
      "weekLow": 8.5,
      "weekHigh": 17.04,
      "periods": ["Q1 25", "Q2 25", "Q3 25", "Q4 25", "Q1 26"],
      "pls": [16.899, 18.2, 15.5, 17.1, 16.2],
      "pvpas": [2.895, 3.1, 2.7, 2.9, 2.85],
      "lastQuarter": "2026-03-31"
    }
  ]
}
```

| Field | Description |
|---|---|
| `atualizadoEm` | Timestamp of last snapshot run |
| `stocks[]` | Complete list of all fetched stocks |
| `codigo` | Ticker symbol |
| `nome` | Company name |
| `lpa` | Earnings per share (GAAP trailing EPS — always, including for REITs; the FFO figure lives in the separate `ffoPs` field) |
| `vpa` | Book value per share from latest quarterly report |
| `weekLow` | 52-week low price |
| `weekHigh` | 52-week high price |
| `periods[]` | **[v2.6.0+]** Array of quarter labels (e.g., "Q1 25", "Q2 25", oldest → newest). Present when ≥2 clean quarters available from yfinance quarterly statements; omitted otherwise (app falls back to 52-week-range approximation). |
| `pls[]` | **[v2.6.0+]** Quarter-end P/L (Price-to-Earnings) multiples, 1:1 aligned with `periods` array. For REITs, these are P/FFO (Price-to-Funds-From-Operations) instead. Real historical ratios, not snapshots. Indices correspond 1:1 with `periods` and `pvpas`. |
| `pvpas[]` | **[v2.6.0+]** Quarter-end Price-to-Book (P/VPA) multiples, 1:1 aligned with `periods` array. Real historical ratios, not snapshots. |
| `lastQuarter` | Date of most recent quarterly earnings (ISO 8601) |

#### Quarterly Data Accumulation

Starting in v2.6.0, US stocks (`us-stocks-all.json`) include quarter-by-quarter historical multiples for improved FL formula accuracy. Each monthly pipeline run:
1. Fetches new quarters from yfinance quarterly financial statements
2. Merges new quarters by period label into previously published `periods`, `pls`, and `pvpas` arrays (previously published values win on overlap to avoid re-processing)
3. Array depth grows over time beyond the initial 5 quarters

**Example timeline:**
- **Month 1 (July 2026):** Q1–Q5 2025 available → `periods: ["Q1 25", ..., "Q1 26"]`, `pls` and `pvpas` length = 5
- **Month 2 (August 2026):** Q2 2026 closes → merge as "Q2 26" → `periods` length = 6, arrays grow
- **Over time:** Arrays accumulate historical depth, allowing the app to compute rolling multiples and trend analysis

**Fallback:** Stocks with <2 clean quarters omit `periods`, `pls`, `pvpas` entirely and retain the previous shape. DetailScreen falls back to the 52-week-range approximation (weekLow/weekHigh) for FL formula in this case.

### REIT Valuation

REITs (Real Estate Investment Trusts) are marked separately in `us-stocks-all.json` and use alternative valuation schemas.

#### Equity REITs (Operating Properties)

Equity REITs own and operate real estate properties. They are included in `us-stocks-all.json` with FFO-based valuation:

```json
{
  "codigo": "AMH",
  "nome": "Ares Manufactured Housing...",
  "lpa": 4.32,
  "vpa": 42.857,
  "weekLow": 68.5,
  "weekHigh": 82.08,
  "periods": ["Q1 25", "Q2 25", "Q3 25", "Q4 25", "Q1 26"],
  "pls": [18.0967, 16.6057, 16.0125, 16.3485, 16.0624],
  "pvpas": [1.4273, 1.3707, 1.3375, 1.35, 1.4359],
  "isReit": true,
  "dividendRate": 3.108,
  "ffoPs": 4.3683
}
```

| Field | Description |
|---|---|
| `isReit` | `true` — marks this entry as a REIT |
| `pls[]` | For REITs, these are **P/FFO** (Price-to-Funds-From-Operations), not P/L. FFO is the standard multiple for equity REITs because it adjusts for non-cash depreciation. |
| `pvpas[]` | Price-to-Book multiples (same as non-REITs) |
| `dividendRate` | Trailing annual dividend per share (TTM basis), used in Bazin dividend fair price model: `VI = dividendRate ÷ 0.06` |
| `ffoPs` | Funds From Operations per share (TTM basis). Calculated as `(net income + depreciation & amortization) ÷ shares outstanding`. Used in P/FFO multiple calculation and as alternative earnings proxy for REITs. |

**Valuation:** for equity REITs the app suppresses BG entirely (GAAP earnings are distorted by real-estate depreciation) and computes FL with FFO in place of earnings — `VI = √(median(P/FFO) × median(P/VPA) × ffoPs × VPA)` — alongside the Bazin dividend fair price.

#### Mortgage REITs (Loan Portfolios)

Mortgage REITs finance real estate mortgages rather than owning properties. They are included in `us-stocks-all.json` but excluded from `us-top-descontos.json` rankings. They omit `periods`, `pls`, `pvpas`, and `ffoPs`:

```json
{
  "codigo": "ABR",
  "nome": "Arbor Realty Trust",
  "lpa": 0.4,
  "vpa": 11.625,
  "weekLow": 4.86,
  "weekHigh": 12.58,
  "isReit": true,
  "isMortgageReit": true,
  "dividendRate": 1.07
}
```

| Field | Description |
|---|---|
| `isReit` | `true` — marks this entry as a REIT |
| `isMortgageReit` | `true` — sub-classifies as a mortgage REIT (lender, not operator) |
| `dividendRate` | Trailing annual dividend per share; mortgage REITs emphasize dividend yield for valuation |
| *(no quarterly fields)* | FFO is not a standard measure for mortgage REITs (they are lenders, not property operators), so the pipeline skips their quarterly statements entirely — no `periods`/`pls`/`pvpas`/`ffoPs`. |

**Valuation:** the app uses only the Bazin dividend model (`VI = dividendRate ÷ 0.06`) for mortgage REITs; BG and FL are fully suppressed (shown as `—`).

**Exclusion from rankings:** Both equity and mortgage REITs are absent from `us-top-descontos.json` because discount rankings assume earnings-based intrinsic value; REITs' FFO and dividend models are incommensurable with traditional stock rankings.

## Update Cadence

**Brazil (IBOVESPA):** Updated monthly by running `npm run publish-top` from the CalcVI project.
**USA:** Updated monthly by running `npm run publish-us` from the CalcVI project.

Intrinsic value is derived from quarterly earnings data (LPA, VPA) and historical quarter-end P/L and P/VPA multiples. Fundamentals change at most four times per year (Q1–Q4), so monthly snapshots age gracefully for several weeks.

**In the App:**
- HomeScreen displays `atualizadoEm` alongside ranked lists so users see data freshness
- DetailScreen shows fundamentals from the pre-fetched catalog before falling back to live Yahoo Finance prices
- All platforms (Android, iOS, web) read the same JSON endpoints — no platform-specific formatting
- REITs are filtered from ranked lists and use alternative valuation pipelines (FFO + dividend model)

## Formulas

Both BG and FL formulas calculate intrinsic value (VI) per share using LPA (earnings) and VPA (book value). For equity REITs, BG is suppressed and FL substitutes FFO per share (`ffoPs`) for LPA; mortgage REITs use the Bazin dividend model only.

**BG — Benjamin Graham (Graham Formula)**
```
VI = √(22.5 × LPA × VPA)
```
Designed for industrial/tangible-asset companies. Uses a universal constant 22.5 (representing 15× historical earnings multiple and 1.5× book value multiple). Conservative and less sensitive to company-specific multiples.

**FL — Felipe Lacerda (Brazilian Value Investing Adaptation)**
```
VI = √(median(P/L) × median(P/VPA) × LPA × VPA)
```
Designed for intangible-asset companies (tech, SaaS, services). Uses company-specific historical median P/L (price-to-earnings) and P/VPA (price-to-book) multiples from past quarters instead of a universal constant. More adaptable to sector and company characteristics. Better for growth-oriented sectors where BG may undervalue quality assets.

**Bazin — Dividend Fair Price (REITs & High-Yield Stocks)**
```
VI = dividendRate ÷ 0.06
```
Used for REITs and dividend-focused strategies. Assumes a 6% annual dividend yield as a fair return benchmark.

## Aviso Legal

Os dados neste repositório são gerados automaticamente a partir de informações financeiras públicas e têm finalidade exclusivamente educacional e de estudo. Não constituem recomendação de investimento. Consulte um Assessor de Investimentos credenciado pela CVM antes de tomar qualquer decisão de investimento.
