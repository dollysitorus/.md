# Python Scripts Reference

## Data Pipeline

| Script | Input | Output | Fungsi |
|--------|-------|--------|--------|
| `clean-ticks.py` | `ticks_XAUUSD.csv` | `cleaned.parquet` | Remove bad ticks, normalize timestamps |
| `build-bars.py` | `cleaned.parquet` | `bars.parquet` | Build tick bars (100 ticks/bar) |
| `engineer-features.py` | `bars.parquet` | `features.parquet` | 18 trading features |
| `engineer-liquidity-features.py` | `features.parquet` | `features-liq.parquet` | Liquidity features (experimental) |
| `generate-labels.py` | `features.parquet` | `labeled.parquet` | Triple-barrier labels (BUY/SELL) |
| `generate-labels-adaptive.py` | `labeled.parquet` | `labeled-adaptive.parquet` | Adaptive SL/TP labels (abandoned) |
| `generate-labels-trigger.py` | `labeled.parquet` | `labeled-trigger.parquet` | Trigger timing labels (experimental) |

## Training

| Script | Model | Status |
|--------|-------|--------|
| `train-signal-2class.py` | Signal 2-class (BUY/SELL) | ✅ Production |
| `train-signal.py` | Signal 3-class (HOLD/BUY/SELL) | ❌ Replaced by 2-class |
| `train.py` | Original trainer | ❌ Deprecated |
| `train-filter.py` | Filter model v1 | ❌ Superseded by Regime |
| `train-filter-v2.py` | Filter model v2 | ❌ Superseded by Regime |
| `train-sizing.py` | Position sizing | ⏸ Phase 5 |
| `train-sltp.py` | Adaptive SL/TP | ❌ Abandoned |
| `model.py` | Model class definitions | ✅ Active |
| `export-onnx.py` | PT → ONNX conversion | ✅ Active |
| `validate-onnx.py` | ONNX validation | ✅ Active |

## Stress Tests & Analysis

| Script | Purpose | Key Result |
|--------|---------|------------|
| `test-stress.py` | 5 audited tests (spread, walk-forward, shuffle, shift, slippage) | PF 2.76+ (filtered bars) |
| `test-stress-dynamic.py` | Dynamic SL/TP stress test | Failed — fragile |
| `test-stress-adaptive.py` | Adaptive SL/TP stress test | Failed — 1/24 profitable |
| `test-filter-stress.py` | Filter model stress test | Superseded |
| `test-confidence.py` | Confidence threshold sweep + trailing stop | 0.55 best; trail $8/$1 = PF 1.16 |
| `test-transition-filter.py` | Regime transition filter | Not effective standalone |
| `test-gate.py` | Phase 2 transition gate | DD turun, PnL turun |
| `test-filtered-vs-all.py` | Filtered bars vs all bars comparison | Root cause PF drop |
| `test-trail-sweep.py` | Trailing stop sweep | All configs profitable |
| `test-account-sim.py` | Account simulation ($200, lot 0.02) | — |
| `test-pipeline.py` | Combined model pipeline | — |
| `test-realism.py` | Realism checks | — |
| `test-session.py` | Session-based analysis | No differentiation |
| `test-sltp-sweep.py` | SL/TP sweep | $3/$15 optimal |
| `analyze-sessions.py` | Per-session performance | Consistent across sessions |
| `simulate-execution.py` | Execution simulation | — |

## Tuning

| Script | Purpose |
|--------|---------|
| `tune-params.py` | General parameter tuning |
| `tune-dynamic-sltp.py` | Walk-forward grid search SL/TP multipliers |
| `tune-confidence-sltp.py` | Confidence-adaptive SL/TP tuning |

## Configs

| File | Description |
|------|-------------|
| `configs/default.yaml` | Main config (model params, training, execution) |
| `configs/rr14.yaml` | R:R 1:4 experiment |
