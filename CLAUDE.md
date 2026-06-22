# etoro_tickers

Static reference dataset of all financial instruments available on the eToro platform. No application code — this is a pure data repository with CI validation.

## What's Here

```
etoro_tickers/
├── instruments.csv     ← 12,544 rows: symbol, company, exchange (last refreshed 2026-05-25)
└── .github/workflows/
    ├── ci.yml          ← CSV validation
    ├── sonarcloud.yml
    └── dependabot-auto-merge.yml
```

## The Dataset

`instruments.csv` — three columns:
- `symbol`: Yahoo Finance format (eToro raw API normalized — `.US` stripped, HK zero-padded, Scandinavian classes hyphenated, delisted/RTH variants excluded)
- `company`: human-readable name
- `exchange`: exchange code

Source: `GET https://www.etoro.com/api/public/v1/instruments/discover` (Stocks + ETFs).

## Refreshing the Data

No tooling is committed — re-fetch from the eToro API and replace `instruments.csv`. The CI validation will catch format regressions.

## Testing / Validation

CI validates the CSV inline (pure stdlib, no deps):
- Exact column names: `symbol`, `company`, `exchange`
- At least 1,000 rows
- All symbols unique and non-empty

```bash
# Run the same check locally
python -c "
import csv
rows = list(csv.DictReader(open('instruments.csv')))
assert set(rows[0].keys()) == {'symbol','company','exchange'}
assert len(rows) >= 1000
symbols = [r['symbol'] for r in rows]
assert len(symbols) == len(set(symbols))
print(f'OK: {len(rows)} instruments')
"
```

## CI

- `ci.yml`: inline Python CSV validation (Python 3.12, push/PR to master)
- `sonarcloud.yml`: SonarCloud scan (excludes CSV, LICENSE, markdown)
- `dependabot-auto-merge.yml`: auto-squash patch/minor; major = manual review

## Role in the Ecosystem

`etorotrade` ingests these 12,544 tickers as its universe for Yahoo Finance signal processing, producing the `etoro.csv` of ~4,051 processed instruments used in daily briefings and portfolio analysis.
