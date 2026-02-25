# Architecture

## System Overview

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  Raw Ticks   │────►│  Data        │────►│  Training    │
│  (CSV 16GB)  │     │  Pipeline    │     │  Pipeline    │
└─────────────┘     └──────────────┘     └──────┬───────┘
                                                 │
                     ┌──────────────┐     ┌──────▼───────┐
                     │  MT5 EA      │◄────│  ONNX Models │
                     │  (live)      │     │  (.onnx)     │
                     └──────────────┘     └──────────────┘
```

## Data Pipeline

```
ticks_XAUUSD.csv (215M ticks, 16GB)
    │
    ▼ clean-ticks.py
cleaned.parquet (remove bad ticks, normalize timestamps)
    │
    ▼ build-bars.py
bars.parquet (tick bars: 100 ticks per bar, OHLCV)
    │
    ▼ engineer-features.py
features.parquet (18 features per bar)
    │
    ▼ generate-labels.py
labeled.parquet (features + direction labels + SL/TP labels)
```

## Model Architecture (Current: 1 model)

```
Input: 18 features × 10 lookback = [batch, 10, 18]
  │
  ▼ LSTM (hidden=128, layers=2)
  │
  ▼ Dropout (0.3)
  │
  ▼ Linear → 2 classes
  │
  ▼ Softmax → P(SELL), P(BUY)
```

## Model Architecture (Target: 3 models)

```
Bar → Model 1 (Regime)  → TRENDING → Model 2a (Trend) → BUY/SELL
                         → RANGING  → Model 2b (Range) → BUY/SELL
                         → DEAD     → NO TRADE
```

## EA Flow

```
OnTick()
  │
  ├─ Collect 100 ticks → build tick bar
  │
  ├─ Compute 18 features (normalize with training stats)
  │
  ├─ Fill lookback buffer (10 bars)
  │
  ├─ ONNX inference → direction + confidence
  │
  ├─ Filters: spread, confidence, weekend, DD kill
  │
  └─ Execute: OrderSend with SL/TP
```

## Execution Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| SL | 3000 pts ($3) | Fixed |
| TP | 15000 pts ($15) | Fixed |
| R:R | 1:5 | |
| Max Spread | 3000 pts | Skip if higher |
| Confidence | 0.55 | Skip if lower |
| Max Hold | 50 bars | Timeout exit |
| Trailing | $8 start, $1 step | PF 0.99→1.16 |
