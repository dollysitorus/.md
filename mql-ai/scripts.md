# Python Scripts Reference

## Data Pipeline (`src/pipeline/`)

| Script | Input | Output | Fungsi |
|--------|-------|--------|--------|
| `clean-ticks.py` | `ticks_XAUUSD.csv` | `cleaned.parquet` | Remove bad ticks, normalize timestamps |
| `build-bars.py` | `cleaned.parquet` | `bars.parquet` | Build tick bars (100 ticks/bar) |
| `engineer-features.py` | `bars.parquet` | `features.parquet` | 18 trading features |
| `generate-labels.py` | `features.parquet` | `labeled.parquet` | Triple-barrier labels (BUY/SELL) |

## Training (`src/training/`)

| Script | Output | Purpose |
|--------|--------|---------|
| `model.py` | — | LSTM model class definition |
| `train-signal-2class.py` | `signal-2class.pt` | Train BUY/SELL signal model |
| `export-onnx.py` | `signal-2class.onnx` | Convert PT → ONNX |
| `validate-onnx.py` | — | Validate ONNX matches PT output |

## Testing & Analysis (`src/testing/`)

| Script | Purpose | Key Finding |
|--------|---------|-------------|
| `test-stress.py` | 5 audited stress tests (A-E) | PF 2.76+ on filtered bars |
| `test-confidence.py` | Confidence sweep + trailing stop | Trail $8/$1 = PF 1.16 |
| `test-filtered-vs-all.py` | Filtered vs all bars comparison | Root cause PF drop |
| `simulate-execution.py` | Execution simulation engine | — |
| `tune-params.py` | Parameter tuning | — |

## Config

| File | Path |
|------|------|
| `default.yaml` | `python/configs/default.yaml` |

## Models

| File | Path | Status |
|------|------|--------|
| `signal-2class.pt` | `python/models/` | ✅ Production |
| `signal.onnx` | `python/models/onnx/` | ✅ Exported |

---

## See Also

| Topik | Baca di |
|-------|---------|
| Detail features yang dihitung | [features.md](features.md) |
| Data source & processed files | [data.md](data.md) |
| Model yang dihasilkan training | [models.md](models.md) |
| Stress test methodology | [stress-test-framework.md](stress-test-framework.md) |
| Full pipeline flow | [architecture.md](architecture.md) → Data Pipeline |
