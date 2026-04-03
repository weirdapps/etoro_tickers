# eToro Tickers

Maintains the master list of eToro-tradeable instruments (ticker → instrumentId mapping).

## Data
- `instruments.csv` — ~604KB, full instrument list
- Updated via GitHub Actions

## Usage
- Referenced by `etoro-trading` plugin for ticker resolution
- Cached locally at `~/.weirdapps-trading/instruments.json`
