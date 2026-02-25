# Models

## 3-Model Architecture

```
Bar → Model 1 (Regime)  → TRENDING → Model 2a (Trend Signal) → BUY/SELL
                         → RANGING  → Model 2b (Range Signal) → BUY/SELL
                         → DEAD     → NO TRADE
```

---

## Model 2a: Trend Signal — `signal-2class.pt` ✅ DONE

| Property | Value |
|----------|-------|
| File | `python/models/signal-2class.pt` |
| ONNX | `python/models/signal-2class.onnx` |
| Type | LSTM 2-class classifier |
| Input | `[batch, 10, 18]` (lookback=10, features=18) |
| Output | `[batch, 2]` → P(SELL), P(BUY) |
| Hidden | 128 |
| Layers | 2 |
| Dropout | 0.3 (training), 0 (inference) |
| Training script | `train-signal-2class.py` |

### Performance

| Condition | WR | PF | DD | PnL |
|-----------|------|------|------|------|
| Filtered bars (label!=0), spread $0.25 | ~57% | **2.76+** | low | +$xxx |
| All bars, spread $0.25, slip $0.05-0.10 | 18% | 0.98 | $1,830 | -$335 |
| All bars + trailing $8/$1 | 28% | **1.16** | $1,832 | +$3,809 |
| All bars + trailing $5/$1 | 37.7% | **1.22** | $2,317 | +$5,444 |

### Critical Finding

> ⚠️ PF 2.76+ vs 0.98 gap karena **stress test hanya pada filtered bars** (`label_direction != 0`, 41% data). Live EA berjalan di ALL bars → model dipaksa prediksi noise → PF turun.
>
> **Solusi:** Model 1 (Regime) sebagai filter → hanya kirim tradeable bars ke signal model.

---

## Model 1: Regime Detector — PLANNED

| Property | Value |
|----------|-------|
| File | `python/models/regime.pt` (planned) |
| Type | LSTM or XGBoost 3-class |
| Output | TRENDING (0) / RANGING (1) / DEAD (2) |
| Target | F1-macro > 0.70 |

### Regime Definitions

| Regime | Label | Definisi |
|--------|-------|----------|
| TRENDING | 0 | `abs(close - close.shift(20)) > 2 × σ₂₀` |
| RANGING | 1 | `abs(close - close.shift(20)) < 0.5 × σ₂₀` |
| DEAD | 2 | `tick_intensity < median × 0.3` OR `spread > mean × 3` |

### Regime Features (6 baru, terpisah dari signal features)

| Feature | Formula |
|---------|---------|
| `sigma_z` | (σ - MA(σ,50)) / std(σ,50) |
| `atr_ratio` | log(ATR_5 / ATR_20) |
| `trend_strength` | abs(close - SMA_20) / σ |
| `range_width` | (high_20 - low_20) / σ |
| `delta_consistency` | sign consistency delta over 10 bars |
| `tick_intensity_z` | z-score of tick_intensity |

---

## Model 2b: Range Signal — PLANNED

| Property | Value |
|----------|-------|
| File | `python/models/range-signal.pt` (planned) |
| Type | LSTM 2-class |
| Strategy | Mean reversion |
| SL/TP | $2/$4 (R:R 1:2, tighter than trend) |

---

## Abandoned Models

| Model | Why Abandoned |
|-------|---------------|
| Adaptive SL/TP (sltp.pt) | SL MAE=23 pips too high, overfitting, fragile |
| Filter v1 (train-filter.py) | Superseded by Regime Detector |
| Sizing (train-sizing.py) | Phase 5 only |
| Signal 3-class (train-signal.py) | Replaced by 2-class (better F1) |

---

## See Also

| Topik | Baca di |
|-------|---------|
| Detail 18 signal features | [features.md](features.md) → Signal Model Features |
| Detail 6 regime features | [features.md](features.md) → Planned Regime Features |
| Kenapa model ini dipilih | [decisions.md](decisions.md) → D2, D7 |
| Hasil stress test detail | [research-log.md](research-log.md) → 2026-02-23 |
| Kenapa PF 2.76 vs 0.98 | [research-log.md](research-log.md) → Root Cause Discovery |
| Framework validasi model baru | [stress-test-framework.md](stress-test-framework.md) |
| Script training | [scripts.md](scripts.md) → Training |
| Bagaimana EA pakai model | [ea.md](ea.md) → ONNX Integration |
| Architecture flow lengkap | [architecture.md](architecture.md) → 3-Model Architecture |
