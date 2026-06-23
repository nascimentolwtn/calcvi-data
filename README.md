# calcvi-data

Static data repository for [CalcVI 2.0](https://github.com/nascimentolwtn/calcvi2) — Calculadora de Valor Intrínseco.

Serves a monthly JSON snapshot of the most discounted IBOVESPA stocks ranked by intrinsic value (Benjamin Graham BG formula and Felipe Lacerda FL formula). Consumed by the CalcVI app on startup to pre-populate the ranked list without requiring a live API call.

## Endpoint

```
https://nascimentolwtn.github.io/calcvi-data/top-descontos.json
```

## Schema

```json
{
  "updatedAt": "2026-06-23",
  "bg": [
    { "codigo": "BBDC4", "vi": 28.4, "cotacao": 14.2, "desconto": 100.0 }
  ],
  "fl": [
    { "codigo": "WEGE3", "vi": 55.1, "cotacao": 38.0, "desconto": 45.0 }
  ]
}
```

| Field | Description |
|---|---|
| `updatedAt` | ISO date of last snapshot run |
| `bg[]` | Top 20 tickers ranked by BG (Graham) discount, descending |
| `fl[]` | Top 20 tickers ranked by FL (Lacerda) discount, descending |
| `codigo` | B3 ticker (e.g. `BBDC4`) |
| `vi` | Intrinsic value calculated by the respective formula |
| `cotacao` | Market price at snapshot time |
| `desconto` | `100 × (vi − cotacao) / cotacao` — positive means undervalued |

## Update cadence

Updated manually once a month by running `scripts/publishTopDescontos.ts` from the CalcVI project. Intrinsic value is derived from quarterly earnings data (LPA, VPA) and historical P/L and P/VPA series — it changes at most four times a year, so a monthly snapshot ages gracefully.

## Formulas

**BG — Benjamin Graham**
```
VI = √(22.5 × LPA × VPA)
```
Designed for industrial/tangible-asset companies. The constant 22.5 = 15× earnings × 1.5× book value.

**FL — Felipe Lacerda**
```
VI = √(mediana(P/L) × mediana(P/VPA) × LPA × VPA)
```
Designed for intangible/services companies (tech, SaaS, internet). Uses company-specific historical median multiples instead of a universal constant.

## Aviso Legal

Os dados neste repositório são gerados automaticamente a partir de informações financeiras públicas e têm finalidade exclusivamente educacional e de estudo. Não constituem recomendação de investimento. Consulte um Assessor de Investimentos credenciado pela CVM antes de tomar qualquer decisão de investimento.
