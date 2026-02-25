# Data

## Raw Data

| File | Size | Content |
|------|------|---------|
| `ticks_XAUUSD.csv` | 16 GB | ~215M ticks XAUUSD |

### Fields
`timestamp_ms`, `bid`, `ask`, `flags`

### Time Range
Data mencakup beberapa tahun (2021-2026, estimate dari tick density).

---

## Processed Data

### `python/data/training/labeled.parquet`

| Property | Value |
|----------|-------|
| Size | ~25 MB |
| Bars | ~100K tick bars (100 ticks/bar) |
| Features | 18 |
| Labels | `label_direction` (-1=SELL, 0=HOLD, 1=BUY) |

### Label Distribution
- BUY (~20%): harga naik > $15 sebelum turun > $3
- SELL (~20%): harga turun > $15 sebelum naik > $3
- HOLD (~60%): tidak mencapai TP atau SL

### Train/Val Split
- Split ratio: `train_split` dari `default.yaml` (~70-80%)
- Chronological split (tidak random)
- Normalization dihitung dari training set saja

---

## Tick Bars

Bar terbentuk setiap 100 ticks. Setiap bar memiliki:

| Column | Asal |
|--------|------|
| `open`, `high`, `low`, `close` | OHLC dari 100 ticks |
| `volume` | Jumlah ticks (selalu 100) |
| `timestamp_ms` | Timestamp bar close |
| `buy_vol`, `sell_vol` | Volume per sisi |
| `spread_mean`, `spread_std` | Spread statistik |
| + 18 engineered features | Dari `engineer-features.py` |

---

## GPU Server

Data di-upload ke GPU server untuk training:
```
Server: connect.singapore-a.gpuhub.com
Port: varies per session (rent-based)
Path: /root/mql-ai/
Runtime: Miniconda + PyTorch (CUDA)
```
