# eToro Tickers

Reference dataset of every stock and ETF tradeable on the eToro platform, normalised to Yahoo Finance symbol format.

[![CI](https://github.com/weirdapps/etoro_tickers/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/weirdapps/etoro_tickers/actions/workflows/ci.yml)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=weirdapps_etoro_tickers&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=weirdapps_etoro_tickers)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## What this repository is

A single-file, pure-data repository. `instruments.csv` holds 12,544 instruments (stocks + ETFs) covering 48 exchanges worldwide, refreshed periodically from the eToro Public API. There is no application code; the repository exists so downstream tools can pin to a reviewed, versioned universe file rather than re-scraping the API on every run.

Consumers include [`etorotrade`](https://github.com/weirdapps/etorotrade), which reads this file as the input universe for Yahoo Finance signal processing.

## The dataset

`instruments.csv` is a UTF-8, header-first CSV with three columns:

| Column | Description | Examples |
|---|---|---|
| `symbol` | Yahoo Finance style ticker | `AAPL`, `VOD.L`, `BMW.DE`, `9988.HK`, `VOLV-B.ST` |
| `company` | Human-readable issuer or fund name | `Apple`, `Volkswagen AG`, `Alibaba Group Holding Ltd (Hong Kong)` |
| `exchange` | eToro exchange label | `Nasdaq`, `NYSE`, `LSE`, `FRA`, `Hong Kong Exchanges`, `Stockholm  Stock Exchange` |

Row count and column names are enforced by CI (see below), so downstream code can rely on them.

### Exchange coverage

48 distinct exchanges. The largest cohorts (as of the current snapshot) are Nasdaq (3,660), NYSE (2,609), LSE (1,050), ASX / Sydney (852), Euronext Paris (572), Frankfurt (531), LSE AIM (488), Stockholm (430), Xetra ETFs (318), Oslo (281), Hong Kong (243), and Tokyo (225). Smaller venues include Borsa Italiana, Helsinki, Amsterdam, Copenhagen, Brussels, SIX, Madrid, Vienna, Warsaw, Dublin, Dubai, Abu Dhabi, Budapest, and Prague.

### Symbol normalisation rules

Symbols are converted from eToro's native format to Yahoo Finance format so the file can be used as-is by libraries like `yfinance` and `pandas-datareader`:

- `.US` suffix stripped (`AAPL.US` becomes `AAPL`).
- Hong Kong tickers zero-padded to 4 digits with `.HK` suffix (`9988.HK`, `0939.HK`).
- Scandinavian share classes hyphenated (`VOLV-B.ST`, `CARL-B.CO`, `ERIC-A.ST`).
- `.RTH` and `.DELISTED` variants excluded.

## Data source

```text
GET https://www.etoro.com/api/public/v1/instruments/discover
```

Asset classes fetched: Stocks and ETFs. The eToro Public API requires `X-API-KEY`, `X-USER-KEY`, `X-REQUEST-ID` (UUID), and `User-Agent` headers; details are not required to consume the CSV, only to regenerate it.

## Usage

Read straight from GitHub or clone the repo. Example with `pandas`:

```python
import pandas as pd

instruments = pd.read_csv("instruments.csv")

# Filter by exchange
nasdaq = instruments[instruments["exchange"] == "Nasdaq"]

# Look up a specific ticker
apple = instruments[instruments["symbol"] == "AAPL"]

# Distribution across venues
instruments["exchange"].value_counts()
```

Pure stdlib works too:

```python
import csv
with open("instruments.csv") as f:
    for row in csv.DictReader(f):
        print(row["symbol"], row["company"], row["exchange"])
```

## Refreshing the data

There is intentionally no committed extractor. To refresh:

1. Call `GET https://www.etoro.com/api/public/v1/instruments/discover` with the required headers.
2. Filter to Stocks and ETFs.
3. Apply the normalisation rules above.
4. Overwrite `instruments.csv` and open a PR. CI will reject the change if the schema or row count drifts.

Snapshot date is visible via `git log -1 -- instruments.csv`.

## Continuous integration

Three workflows run on push and pull request against `master`:

- [`ci.yml`](.github/workflows/ci.yml): inline Python (3.12, stdlib only) that asserts the header is exactly `symbol,company,exchange`, that there are at least 1,000 rows, that every symbol is unique, and that no symbol is empty.
- [`sonarcloud.yml`](.github/workflows/sonarcloud.yml): SonarCloud scan on the non-CSV, non-doc surface (see `sonar-project.properties`). Skipped automatically if `SONAR_TOKEN` is not set.
- [`dependabot-auto-merge.yml`](.github/workflows/dependabot-auto-merge.yml): auto-squashes Dependabot patch, minor, and grouped updates; majors require manual review.

Dependabot itself watches `github-actions` weekly (see [`.github/dependabot.yml`](.github/dependabot.yml)).

## Local validation

Run the exact CI check against your working tree:

```bash
python - <<'PY'
import csv
rows = list(csv.DictReader(open("instruments.csv")))
assert list(rows[0].keys()) == ["symbol", "company", "exchange"], "header mismatch"
assert len(rows) >= 1000, f"only {len(rows)} rows"
symbols = [r["symbol"] for r in rows]
assert len(set(symbols)) == len(symbols), "duplicate symbols"
assert all(r["symbol"].strip() for r in rows), "empty symbol"
print(f"OK: {len(rows)} instruments")
PY
```

## Security

See [SECURITY.md](SECURITY.md). Vulnerabilities to `plessas@nbg.gr`, not public issues.

## License

[MIT](LICENSE). Copyright (c) 2026 Dimitris Plessas.
