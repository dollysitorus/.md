# Architecture

## System Overview

```
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐
│  Raw Ticks   │────►│  VDB Bar     │────►│  LSTM Training   │
│  (CSV 16GB)  │     │  Pipeline    │     │  Pipeline        │
└─────────────┘     └──────────────┘     └──────┬───────────┘
                                                 │
                     ┌──────────────┐     ┌──────▼───────────┐
                     │  MT5 EA      │◄────│  ONNX Model      │
                     │  (live)      │     │  (embedded)       │
                     └──────────────┘     └──────────────────┘
```

## Project Structure

```
mql-ai/
├── mql5/
│   ├── experts/
│   │   ├── vdb-day-ea/              ← PRODUCTION v7.10 (day-trade)
│   │   │   └── vdb-day-ea.mq5
│   │   ├── vdb-scalp-ea/            ← PRODUCTION v7.00 (scalping)
│   │   │   └── vdb-scalp-ea.mq5
│   │   ├── vdb-ea/                  ← base template
│   │   │   └── vdb-ea.mq5
│   │   ├── router-ea/               ← DEPRECATED v6.00
│   │   └── signal-trader-ea.mq5     ← DEPRECATED v5.00
│   └── Files/
│       ├── vdb_daytrade_6_10.onnx   ← day-trade model
│       └── vdb-signal.onnx          ← scalp model
│
├── python/
│   ├── src/
│   │   ├── pipeline/                ← VDB data processing
│   │   ├── training/                ← model training
│   │   └── testing/                 ← stress tests & analysis
│   ├── models/                      ← .pt, .npz, .onnx files
│   └── data/
│
└── ticks_XAUUSD.csv                 ← 16GB raw data (gitignored)
```

## VDB Pipeline

```
ticks_XAUUSD.csv (215M ticks, 16GB)
    │
    ▼ VDB Bar Builder
    │  θ_I=6, θ_E=10 (day-trade)
    │  Bar closes when: |I| >= θ_I AND E >= θ_E
    │
    ▼ Feature Engineering (15 features)
    │  imb_ratio, persistence, flip_rate, E_slope, duration_ratio,
    │  E, ofi, buy_r, delta, imb_momentum, sigma, twap_dev,
    │  bar_speed, spread_ratio, range_ratio
    │
    ▼ Labeling (forward-looking 600s)
    │  BUY: price rises $10 before falling $4
    │  SELL: price falls $10 before rising $4
    │
    ▼ LSTM Training (PyTorch → ONNX export)
       Input: [1, 30, 15]
       Output: [1, 2] → softmax → P(SELL), P(BUY)
```

## VDB Bar Model

VDB = Value-Driven Bars. Event-driven, bukan time-based.

### Bar Accumulation
```
Each tick:
  mid = (bid + ask) / 2
  E += |mid - prev_mid|        ← Energy (total activity)
  I += (mid - prev_mid)        ← Imbalance (net direction)

Bar closes when: |I| >= θ_I AND E >= θ_E
```

### Configurations

| Style | θ_I | θ_E | Median Duration | Status |
|-------|-----|-----|-----------------|--------|
| Scalp | 3 | 5 | ~1.3s | ✅ Production |
| Day-Trade | 6 | 10 | ~3.2s | ✅ Production |
| Swing | 15 | 25 | ~11.3s | ❌ Not viable (acc 57%) |

## EA Flow (v7.10 Day-Trade)

```
OnTick()
  │
  ├─ Accumulate E, I from tick
  │
  ├─ Bar close? (|I| >= 6 AND E >= 10)
  │     │
  │     ├─ Compute 15 features
  │     ├─ Z-score normalization
  │     ├─ ONNX inference → P(BUY), P(SELL), confidence
  │     │
  │     ├─ Has position(s)? → ManagePosition()
  │     │     ├─ Signal flip (conf>=0.75) → CLOSE
  │     │     ├─ Time stop (200 bars) → CLOSE
  │     │     └─ Asymmetric trail → modify SL
  │     │
  │     └─ New entry?
  │           ├─ Positions < 3?
  │           ├─ 10+ bars since last entry?
  │           ├─ Cooldown done?
  │           ├─ Spread ok?
  │           └─ Confidence >= 0.80? → OPEN
  │
  └─ Draw HUD (WR, PnL, positions, spread)
```

## Dynamic Exit Model (Optimal)

### Entry
```
SL = $4 (initial, fixed)
TP = $25 (hard cap, safety)
```

### Asymmetric Trailing SL
```
Profit $0-$2   → SL = initial (-$4)
Profit $2-$4   → SL = breakeven (entry + spread)
Profit $4-$8   → SL = entry + 50% profit
Profit $8-$12  → SL = entry + 65% profit
Profit $12+    → SL = entry + 80% profit
```

### Exit Triggers
| Trigger | Condition | Action |
|---------|-----------|--------|
| Signal Flip | Model says opposite dir, conf ≥ 0.75 | Close position |
| Time Stop | 200 bars (~10 min, matches label horizon) | Close position |
| Trailing SL | Price hits trailed SL | Broker closes |
| Hard TP | Price hits $25 from entry | Broker closes |

### Multi-Position
| Setting | Value |
|---------|-------|
| Max positions | 3 |
| Min bars between entries | 10 (~30s) |
| Per-position management | Each position tracked independently |

## Execution Parameters

### Day-Trade EA v7.10

| Parameter | Value |
|-----------|-------|
| Entry conf | ≥ 0.80 |
| Initial SL | $4 |
| Max TP | $25 (safety cap) |
| Breakeven at | $2 profit |
| Flip exit conf | ≥ 0.75 |
| Time stop | 200 bars |
| Max positions | 3 |
| Entry delay | 10 bars |
| Max spread | 300 pts (3-digit) |
| Cooldown | 5 bars after loss |
| Magic | 260226 |

### Scalp EA v7.00

| Parameter | Value |
|-----------|-------|
| Entry conf | ≥ 0.80 |
| SL | $2 (fixed) |
| TP | $5 (fixed) |
| Max spread | 300 pts |
| Cooldown | 5 bars |
| Magic | 260227 |

---

## See Also

| Topik | Baca di |
|-------|---------| 
| Model specs & performance | [models.md](models.md) |
| EA detail & ONNX integration | [ea.md](ea.md) |
| Decisions history | [decisions.md](decisions.md) |
| Research experiments | [research-log.md](research-log.md) |
