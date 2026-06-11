# eToro Tickers

[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=weirdapps_etoro_tickers&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=weirdapps_etoro_tickers)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Reference dataset of all financial instruments available on the [eToro](https://www.etoro.com/) trading platform. The file `instruments.csv` contains 12,544 instruments including stocks and ETFs across global exchanges.

## Schema

| Column | Description |
|--------|-------------|
| `symbol` | Ticker symbol in Yahoo Finance format (e.g. `AAPL`, `VOD.L`, `BMW.DE`) |
| `company` | Human-readable company name (e.g. `Apple`, `Volkswagen AG`) |
| `exchange` | Exchange name (e.g. `Nasdaq`, `NYSE`, `London`) |

## Data Source

The data is extracted from eToro's public Asset Explorer API:

```text
GET https://www.etoro.com/api/public/v1/instruments/discover
```

Asset classes fetched: **Stocks** and **ETFs**. Symbols are normalized to Yahoo Finance format (`.US` suffix stripped, HK 5-digit padded to 4-digit, Scandinavian share classes hyphenated, `.RTH` / `.DELISTED` variants excluded).

**Last refresh:** 2026-05-25 11:59 UTC

## Usage

```python
import pandas as pd

instruments = pd.read_csv("instruments.csv")

# Filter by exchange
nasdaq = instruments[instruments["exchange"] == "Nasdaq"]

# Look up a specific ticker
apple = instruments[instruments["symbol"] == "AAPL"]

# Count instruments per exchange
instruments["exchange"].value_counts()
```

## License

[MIT License](LICENSE)
