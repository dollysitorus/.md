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

## Status

### Done
- [x] Project scaffolding (Docker, MQL5, Python)
- [x] Skeleton EA (`onnx-ea.mq5`) + helper (`onnx-helper.mqh`)
- [x] Python model/train/export/validate scripts
- [x] GitHub repo created
- [x] Data cleaning pipeline (clean-ticks, build-bars, engineer-features)

### Todo
- [ ] Run full cleaning on 215M ticks (server)
- [ ] Training model di server
- [ ] Export model ke ONNX
- [ ] Complete EA inference logic
- [ ] Backtest di MT5
