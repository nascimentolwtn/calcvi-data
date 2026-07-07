# calcvi-data

Static data repository for CalcVI 2.0 — Calculadora de Valor Intrínseco App (v2.4.x).

Serves monthly JSON snapshots of:
- **IBOVESPA stocks** — top-discounted BR tickers ranked by intrinsic value (BG & FL formulas)
- **US stocks** — top-discounted US tickers ranked by the same formulas
- **All stocks** — complete catalog (BR + US) with fundamentals and historical multiples

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
  "atualizadoEm": "2026-07-02 21:14",
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

### All Stocks (Complete Catalog)

**File:** `stocks-all.json` or `us-stocks-all.json`

```json
{
  "atualizadoEm": "2026-07-02 21:14",
  "stocks": [
    {
      "codigo": "ABEV3",
      "nome": "Ambev S.A.",
      "lpa": 0.99,
      "vpa": 5.779,
      "weekLow": 8.5,
      "weekHigh": 17.04,
      "pls": [16.899, 18.2, 15.5],
      "pvpas": [2.895, 3.1, 2.7],
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
| `lpa` | Earnings per share from latest quarterly report |
| `vpa` | Book value per share from latest quarterly report |
| `weekLow` | 52-week low price |
| `weekHigh` | 52-week high price |
| `pls[]` | Historical median P/L (Price-to-Earnings) multiples used in FL formula |
| `pvpas[]` | Historical median P/VPA (Price-to-Book) multiples used in FL formula |
| `lastQuarter` | Date of most recent quarterly earnings (ISO 8601) |

## Update Cadence

**Brazil (IBOVESPA):** Updated monthly by running `npm run publish-top` from the CalcVI project.
**USA:** Updated monthly by running `npm run publish-us` from the CalcVI project.

Intrinsic value is derived from quarterly earnings data (LPA, VPA) and historical median P/L and P/VPA multiples. Fundamentals change at most four times per year (Q1–Q4), so monthly snapshots age gracefully for several weeks.

**In the App:**
- HomeScreen displays `atualizadoEm` alongside ranked lists so users see data freshness
- DetailScreen shows fundamentals from the pre-fetched catalog before falling back to live Yahoo Finance prices
- All platforms (Android, iOS, web) read the same JSON endpoints — no platform-specific formatting

## Formulas

Both formulas calculate intrinsic value (VI) per share using LPA (earnings) and VPA (book value).

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

## Aviso Legal

Os dados neste repositório são gerados automaticamente a partir de informações financeiras públicas e têm finalidade exclusivamente educacional e de estudo. Não constituem recomendação de investimento. Consulte um Assessor de Investimentos credenciado pela CVM antes de tomar qualquer decisão de investimento.
