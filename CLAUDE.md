# etoro_tickers

Static reference dataset of the stocks and ETFs available on the eToro platform (not forex, crypto, commodities or indices, which the same API also returns). No application code; this is a pure data repository with CI validation.

## What's Here

```
etoro_tickers/
├── instruments.csv            ← 12,544 rows: symbol, company, exchange (last refreshed 2026-05-27)
├── README.md
├── CLAUDE.md
├── SECURITY.md
├── LICENSE
├── sonar-project.properties
└── .github/
    ├── CODEOWNERS
    ├── dependabot.yml         ← github-actions ecosystem, weekly
    └── workflows/
        ├── ci.yml             ← CSV validation
        ├── sonarcloud.yml
        └── dependabot-auto-merge.yml
```

## The Dataset

`instruments.csv`, three columns (CRLF line endings):
- `symbol`: mostly Yahoo Finance format (eToro raw API normalized: `.US` stripped, HK mostly zero-padded, Scandinavian classes hyphenated, delisted/RTH variants excluded)
- `company`: human-readable name. 78 values have leading/trailing whitespace, so strip before matching.
- `exchange`: exchange label, 32 distinct values. Two carry quirks: `LSE_AIM` uses an underscore and `Stockholm  Stock Exchange` has a double space. Match on the exact string.

Source: `GET https://www.etoro.com/api/public/v1/market-data/instruments`, filtered to `instrumentTypeID` 5 (stock) and 6 (ETF). Single response under `instrumentDisplayDatas`, no pagination. Symbol is `symbolFull`, name is `instrumentDisplayName`, and `exchangeID` is numeric so the `exchange` column needs an ID-to-label map.

**Normalisation is incomplete.** 85 rows (0.68%) are not Yahoo-resolvable and return zero rows through `yfinance`: 32 unpadded HK codes (`3.HK`, `ANT.HK`), 15 Xetra `.DE11`/`.D11`/`.DE22` codes, 11 dot-separated US dual-class (`BRK.B`, not Yahoo's `BRK-B`), 10 `.EUR` eToro shadow listings, 10 Bloomberg-style `.IM`/`.LN`/`.CH`, and 7 warrants/CVRs. Do not describe this file as drop-in for `yfinance` without that caveat.

## eToro API

Canonical domain: `https://www.etoro.com/api/public/v1` (documented endpoint, path prefix `/api/public/v1/`)
Legacy host: `https://public-api.etoro.com/api/v1` (separate host, path prefix `/api/v1/`; throttles silently under batch load, do not use for high-volume calls)
Auth: X-API-KEY + X-USER-KEY (regular, not PERSONAL) + X-REQUEST-ID (UUID) + User-Agent

## Refreshing the Data

No tooling is committed; re-fetch from the eToro API and replace `instruments.csv`. The CI validation will catch format regressions but says nothing about symbol quality.

A working end-to-end implementation of the same fetch, filter, normalise and write sequence (including the exchange ID map) lives in the sibling repo at `etorotrade/scripts/refresh_etoro_universe.py`. Read it before writing a new extractor.

## Testing / Validation

CI validates the CSV inline (pure stdlib, no deps):
- Exact column names: `symbol`, `company`, `exchange`
- At least 1,000 rows
- All symbols unique and non-empty

```bash
# Run the same check locally. Column order matters: CI compares the header as a list.
python -c "
import csv
rows = list(csv.DictReader(open('instruments.csv')))
assert list(rows[0].keys()) == ['symbol','company','exchange']
assert len(rows) >= 1000
symbols = [r['symbol'] for r in rows]
assert len(symbols) == len(set(symbols))
assert all(s.strip() for s in symbols)
print(f'OK: {len(rows)} instruments')
"
```

## CI

- `ci.yml`: inline Python CSV validation (Python 3.12, push/PR to master)
- `sonarcloud.yml`: SonarCloud scan (excludes CSV, LICENSE, markdown). Push to master, any PR, manual dispatch.
- `dependabot-auto-merge.yml`: thin caller for the shared reusable at `weirdapps/shared-workflows/.github/workflows/dependabot-auto-merge.yml@main`. No local merge logic here, so change policy there, not in this repo. Squash-merges patch/minor/grouped once the PR's other checks are green; a standalone major stays open.

## Role in the Ecosystem

Standalone. Nothing in the organisation reads this file on a schedule. In particular `etorotrade` does **not** consume it: it maintains its own universe at `yahoofinance/input/etoro.csv` (same three columns), refreshed weekly from the same eToro instruments endpoint by its own `weekly-universe-refresh.yml` workflow. Treat this CSV as a pinned historical snapshot, not a live feed, and do not claim a downstream dependency that does not exist.
