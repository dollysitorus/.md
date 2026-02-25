# mql-ai

## Overview

MQL5 Expert Advisor dengan model AI ONNX untuk automated trading di MetaTrader 5.

## Stack

| Layer | Tech | Keterangan |
|-------|------|------------|
| Trading | MQL5 | Expert Advisor + indicators di MT5 |
| ML Training | Python + PyTorch | Training model AI |
| Model Format | ONNX | Portable format, inference di MT5 |
| Dev Container | python:3.12-alpine | Docker-only development |

## Architecture

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  Historical   │ ───► │   Python     │ ───► │  ONNX Model  │
│  Data (CSV)   │      │  Training    │      │  (.onnx)     │
└──────────────┘      └──────────────┘      └──────┬───────┘
                                                    │
                                                    ▼
                                             ┌──────────────┐
                                             │  MT5 EA      │
                                             │  (onnx-ea)   │
                                             └──────────────┘
```

## Conventions

- **Docker-only**: Semua Python execution via `docker compose run --rm dev`
- **Model output**: 3 kelas — 0=HOLD, 1=BUY, 2=SELL
- **Config**: `python/configs/default.yaml` — satu file config per experiment
- **Naming**: camelCase untuk Python variables, PascalCase untuk MQL5 functions

## Repo

- **GitHub**: https://github.com/dollysitorus/mql-ai (private)
- **Context**: `.md/mql-ai/`

## Documentation

| Doc | Isi |
|-----|-----|
| [context.md](context.md) | Overview, status, 3-model plan, production framework |
| [architecture.md](architecture.md) | System design, data pipeline, EA flow |
| [models.md](models.md) | Model specs, performance, 3-model architecture |
| [features.md](features.md) | 18 signal features + 6 regime features |
| [decisions.md](decisions.md) | 10 key decisions log with evidence |
| [scripts.md](scripts.md) | 37 Python scripts reference |
| [data.md](data.md) | Raw data, processed data, tick bars, label distribution |
| [ea.md](ea.md) | EA parameters, features, log format, ONNX integration |


## Status

### 3-Model Architecture (CURRENT DIRECTION)

```
Bar → Model 1 (Regime)  → TRENDING → Model 2a (Trend Signal) → BUY/SELL [SL=$3 TP=$15]
                         → RANGING  → Model 2b (Range Signal) → BUY/SELL [SL=$2 TP=$4]
                         → DEAD     → NO TRADE
```

#### Model 1: Regime Detector (TODO)
- [ ] Generate regime labels (TRENDING / RANGING / DEAD)
- [ ] Train regime classifier
- [ ] Validate F1-macro > 0.70

#### Model 2a: Trend Signal (DONE — signal-2class.pt)
- [x] LSTM 2-class (BUY/SELL), 18 features, lookback=10
- [x] ONNX exported: `signal-2class.onnx`
- [x] Stress tested: PF 2.76+ on filtered (tradeable) bars
- ⚠️ PF drops to 0.98 on ALL bars — needs regime filter

#### Model 2b: Range Signal (TODO)
- [ ] Mean reversion strategy for ranging market
- [ ] SL/TP lebih ketat ($2/$4)

### Validated Findings
- Fixed SL/TP ($3/$15) = most stable for trending
- Confidence threshold 0.55 = optimal (higher is worse)
- Trailing stop $8/$1 = PF 0.99 → 1.16 (game changer)
- Dynamic SL/TP = abandoned (fragile)
- Transition filters = do not improve as standalone
- Session features = no differentiation
- **Root cause PF drop**: model tested on filtered bars (PF 2.76) but live runs all bars (PF 0.98)

### Production Framework
1. **Risk Guardrail** — risk ≤ 0.5%, DD scaling, losing streak pause
2. **Regime Gate** — Model 1 decides tradeable vs skip
3. **Execution Validation** — live demo 2-4 minggu with logging
4. **Stability Review** — assess DD, confidence drift, trade count
5. **Improve** — ONLY after stability proven

### Prinsip
- Research = cari edge | Production = lindungi edge
- Target v1 = stabil, survive regime shift, DD terkendali
- Profit datang setelah stabilitas

### Todo
- [ ] Build Model 1 (Regime Detector)
- [ ] Re-validate Model 2a on trending-only bars
- [ ] Build Model 2b (Range Signal)
- [ ] Full pipeline simulation
- [ ] Export 3 ONNX → update EA
- [ ] Implement risk guardrails di EA
- [ ] Deploy ke demo 2-4 minggu


