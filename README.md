# eToro Tickers

Reference dataset of all financial instruments available on the [eToro](https://www.etoro.com/) trading platform. The file `instruments.csv` contains ~10,000 instruments including stocks, ETFs, currencies, commodities, crypto, and indices.

## Schema

| Column | Description |
|--------|-------------|
| `instrumentID` | eToro internal instrument identifier |
| `instrumentDisplayName` | Human-readable name (e.g. `EUR/USD`, `Apple`) |
| `symbolFull` | Ticker symbol including exchange suffix (e.g. `AAPL`, `VOD.L`, `BMW.DE`) |
| `instrumentTypeID` | Asset class (`1` = Currencies, `2` = Commodities, `4` = Indices, `5` = Stocks, `6` = ETFs, `10` = Crypto) |
| `exchangeID` | Exchange identifier |
| `priceSource` | Price data provider |
| `hasExpirationDate` | Whether the instrument expires (e.g. futures) |
| `isInternalInstrument` | Whether it is an eToro-internal instrument |
| `image_count` | Number of associated images on the platform |

## Data Source

The data is extracted from eToro's public instrument catalogue via their API.

## Usage

```python
import pandas as pd

instruments = pd.read_csv("instruments.csv")

# Filter stocks only
stocks = instruments[instruments["instrumentTypeID"] == 5]

# Look up a specific ticker
apple = instruments[instruments["symbolFull"] == "AAPL"]
```

## License

[MIT License](LICENSE)
