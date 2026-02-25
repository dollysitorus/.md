# Features

## Signal Model Features (18)

### Order Flow (8 features)

| # | Feature | Formula | Apa yang diukur |
|---|---------|---------|-----------------|
| 1 | `order_flow_imbalance` | (buy_vol - sell_vol) / total_vol | Tekanan beli vs jual |
| 2 | `delta` | buy_vol - sell_vol per bar | Net buying per bar |
| 3 | `cumulative_delta` | running sum(delta) | Akumulasi tekanan beli |
| 4 | `buy_ratio` | buy_ticks / total_ticks | % tick naik |
| 5 | `sell_ratio` | sell_ticks / total_ticks | % tick turun |
| 6 | `delta_ma` | MA(delta, 10) | Smoothed order flow |
| 7 | `delta_divergence` | delta - delta_ma | Deviasi order flow |
| 8 | `cd_slope` | slope(cumulative_delta, 5) | Momentum order flow |

### Microstructure (5 features)

| # | Feature | Formula | Apa yang diukur |
|---|---------|---------|-----------------|
| 9 | `spread_compression` | spread / MA(spread) | Spread ketat = likuid |
| 10 | `wide_spread_ratio` | count(spread > 2x mean) / N | Frekuensi spread lebar |
| 11 | `spread_mean` | mean(spread) per bar | Rata-rata spread |
| 12 | `stop_hunt_count` | sudden wick > 2σ | Deteksi stop hunting |
| 13 | `event_count` | significant price moves | Jumlah event penting |

### Volatility & Price (3 features)

| # | Feature | Formula | Apa yang diukur |
|---|---------|---------|-----------------|
| 14 | `rolling_sigma` | std(returns, 20) | Volatilitas terkini |
| 15 | `twap_deviation` | close - TWAP | Deviasi dari fair price |
| 16 | `tick_intensity` | ticks / duration_sec | Kecepatan tick |

### Bar Properties (2 features)

| # | Feature | Formula | Apa yang diukur |
|---|---------|---------|-----------------|
| 17 | `duration_sec` | bar end - bar start (seconds) | Lama bar terbentuk |
| 18 | `tick_count` | total ticks in bar | Volume per bar |

## Normalization

Training set → mean, std per feature → `(x - mean) / std`

Disimpan di EA sebagai array `InpNormMeans` dan `InpNormStds`.

## Planned Regime Features (6)

| Feature | Formula | Signal |
|---------|---------|--------|
| `sigma_z` | (σ - MA(σ,50)) / std(σ,50) | Volatility regime change |
| `atr_ratio` | log(ATR_5 / ATR_20) | Short vs long vol |
| `trend_strength` | abs(close - SMA_20) / σ | Trending vs flat |
| `range_width` | (high_20 - low_20) / σ | Range size |
| `delta_consistency` | sign consistency over 10 bars | Order flow direction |
| `tick_intensity_z` | z-score of tick_intensity | Activity level |

---

## See Also

| Topik | Baca di |
|-------|---------|
| Model apa yang pakai features ini | [models.md](models.md) |
| Script yang generate features | [scripts.md](scripts.md) → `pipeline/engineer-features.py` |
| Bagaimana features dinormalisasi di EA | [ea.md](ea.md) → Normalization |
| Raw data → features pipeline | [architecture.md](architecture.md) → Data Pipeline |
| Data source & statistics | [data.md](data.md) |
