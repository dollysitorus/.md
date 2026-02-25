# Architecture

## System Overview

```
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐
│  Raw Ticks   │────►│  Data        │────►│  Training        │
│  (CSV 16GB)  │     │  Pipeline    │     │  Pipeline        │
└─────────────┘     └──────────────┘     └──────┬───────────┘
                                                 │
                     ┌──────────────┐     ┌──────▼───────────┐
                     │  MT5 EA      │◄────│  3 ONNX Models   │
                     │  (live)      │     │  regime + signal  │
                     └──────────────┘     └──────────────────┘
```

## Project Structure

```
mql-ai/
├── mql5/
│   ├── experts/
│   │   └── signal-trader-ea.mq5    ← production EA
│   ├── include/
│   │   └── onnx-helper.mqh
│   └── models/
│       └── signal.onnx             ← deployed ONNX
│
├── python/
│   ├── configs/
│   │   └── default.yaml
│   ├── data/
│   │   └── training/
│   │       └── labeled.parquet
│   ├── models/
│   │   ├── signal-2class.pt        ← production model
│   │   └── onnx/
│   │       └── signal.onnx
│   ├── src/
│   │   ├── pipeline/               ← data processing
│   │   │   ├── clean-ticks.py
│   │   │   ├── build-bars.py
│   │   │   ├── engineer-features.py
│   │   │   └── generate-labels.py
│   │   ├── training/               ← model training
│   │   │   ├── model.py
│   │   │   ├── train-signal-2class.py
│   │   │   ├── export-onnx.py
│   │   │   └── validate-onnx.py
│   │   └── testing/                ← stress tests & analysis
│   │       ├── test-stress.py
│   │       ├── test-confidence.py
│   │       ├── test-filtered-vs-all.py
│   │       ├── simulate-execution.py
│   │       └── tune-params.py
│   └── requirements.txt
│
└── ticks_XAUUSD.csv                ← 16GB raw data (gitignored)
```

## Data Pipeline

```
ticks_XAUUSD.csv (215M ticks, 16GB)
    │
    ▼ pipeline/clean-ticks.py
cleaned.parquet
    │
    ▼ pipeline/build-bars.py
bars.parquet (tick bars: 100 ticks per bar)
    │
    ▼ pipeline/engineer-features.py
features.parquet (18 signal features)
    │
    ├──► pipeline/generate-labels.py → labeled.parquet (BUY/SELL)
    └──► (future) generate-labels-regime.py → regime-labeled.parquet
```

## 3-Model Architecture

```
                    ┌─────────────────────────────────────┐
                    │            EA OnTick()               │
                    │                                      │
   100 ticks ──────►│  Build tick bar + compute features   │
                    │                                      │
                    │  ┌───────────────────────────────┐   │
                    │  │  Model 1: Regime Detector      │   │
                    │  │  Input: 6 regime features      │   │
                    │  │  Output: TRENDING/RANGING/DEAD │   │
                    │  └───────────┬────────────────────┘   │
                    │              │                         │
                    │    ┌─────────┼──────────┐              │
                    │    ▼         ▼          ▼              │
                    │ TRENDING  RANGING     DEAD             │
                    │    │         │          │              │
                    │    ▼         ▼          ▼              │
                    │ ┌───────┐ ┌───────┐  NO TRADE         │
                    │ │Model  │ │Model  │                    │
                    │ │2a:    │ │2b:    │                    │
                    │ │Trend  │ │Range  │                    │
                    │ │Signal │ │Signal │                    │
                    │ └───┬───┘ └───┬───┘                    │
                    │     │         │                         │
                    │     ▼         ▼                         │
                    │  BUY/SELL  BUY/SELL                     │
                    │  SL=$3     SL=$2                        │
                    │  TP=$15    TP=$4                        │
                    │  Trail     No trail                     │
                    │  $8/$1                                  │
                    └─────────────────────────────────────┘
```

## EA Flow (Target 3-Model)

```
OnTick()
  │
  ├─ Collect 100 ticks → build tick bar
  │
  ├─ Compute features (18 signal + 6 regime)
  │
  ├─ Fill lookback buffer
  │
  ├─ ONNX #1: Regime → TRENDING / RANGING / DEAD
  │     │
  │     ├─ DEAD → skip
  │     ├─ TRENDING → ONNX #2a → BUY/SELL (SL=$3, TP=$15, trail $8/$1)
  │     └─ RANGING → ONNX #2b → BUY/SELL (SL=$2, TP=$4, no trail)
  │
  ├─ Filters: spread, confidence, weekend, DD kill
  │
  └─ Execute: OrderSend with regime-specific SL/TP
```

## Execution Parameters

### Trending Mode

| Parameter | Value |
|-----------|-------|
| SL | 3000 pts ($3) |
| TP | 15000 pts ($15) |
| Trailing | $8 start, $1 step |

### Ranging Mode

| Parameter | Value |
|-----------|-------|
| SL | 2000 pts ($2) |
| TP | 4000 pts ($4) |
| Trailing | OFF |

### Shared

| Parameter | Value |
|-----------|-------|
| Confidence | ≥ 0.55 |
| Max Spread | 300 pts |
| Max Hold | 50 bars |
| DD Kill | 15% |

---

## See Also

| Topik | Baca di |
|-------|---------|
| Detail model specs & performance | [models.md](models.md) |
| Features input per model | [features.md](features.md) |
| EA parameters & integration | [ea.md](ea.md) |
| Script per folder | [scripts.md](scripts.md) |
| Data source & processed files | [data.md](data.md) |
| Alasan architecture 3-model | [decisions.md](decisions.md) → D7 |
