# Models

## VDB Signal Architecture (CURRENT — v7.x)

```
Ticks → VDB Bar (θ_I/θ_E) → 15 Features → LSTM → P(BUY), P(SELL)
```

> Replaces 2-model architecture (regime+signal). VDB bars naturally filter noise.

---

## VDB Day-Trade Signal — `vdb_daytrade_6_10.onnx` ✅ PRODUCTION

| Property | Value |
|----------|-------|
| File (ONNX) | `mql5/Files/vdb_daytrade_6_10.onnx` |
| File (PyTorch) | `vdb-day-signal-best.pt` |
| Norm | `vdb-day-signal-norm.npz` (z-score) |
| Type | LSTM 2-class classifier |
| Input | `[1, 30, 15]` (SEQ_LEN=30, FEAT=15) |
| Output | `[1, 2]` → softmax(logit_SELL, logit_BUY) |
| Hidden | 128 |
| Layers | 2 |
| Dropout | 0.3 (training), 0 (inference) |
| VDB θ | θ_I=6, θ_E=10 |
| Median bar duration | ~3.2 seconds |
| Training script | `vdb-study.py` (pipeline) |
| EA | `vdb-day-ea/vdb-day-ea.mq5` (magic=260226) |

### Performance

| Metric | Value |
|--------|-------|
| Val accuracy | ~70.0% |
| @ conf≥0.80: WR | 82.7% |
| @ conf≥0.80: PF | 11.93 |
| @ conf≥0.80: trades | ~75K |

### VDB Signal Features (15)

| # | Feature | Deskripsi |
|---|---------|-----------|
| 0 | `imb_ratio` | I / E (arah bar) |
| 1 | `persistence_score` | mean(\|IR\|, last 10 bars) |
| 2 | `flip_rate` | sign changes IR last 10 |
| 3 | `E_slope` | E trend (recent vs prev) |
| 4 | `duration_ratio` | log(dur / median_dur) |
| 5 | `E` | total energy bar |
| 6 | `ofi` | (buy-sell) / total ticks |
| 7 | `buy_r` | buy / total ticks |
| 8 | `delta` | buy - sell count |
| 9 | `imb_momentum` | I_now - I_prev bar |
| 10 | `sigma` | volatility (std of mid changes) |
| 11 | `twap_dev` | close - TWAP |
| 12 | `bar_speed` | ticks_eff / duration_sec |
| 13 | `spread_ratio` | spread / median_spread |
| 14 | `range_ratio` | (high-low) / spread |

---

## VDB Scalp Signal — `vdb-signal.onnx` ✅ PRODUCTION

| Property | Value |
|----------|-------|
| File (ONNX) | `mql5/Files/vdb-signal.onnx` |
| Type | LSTM 2-class classifier |
| Input | `[1, 30, 15]` (same architecture) |
| Output | `[1, 2]` → softmax(logit_SELL, logit_BUY) |
| VDB θ | θ_I=3, θ_E=5 |
| Median bar duration | ~1.3 seconds |
| EA | `vdb-scalp-ea/vdb-scalp-ea.mq5` (magic=260227) |

### Performance

| Metric | Value |
|--------|-------|
| Val accuracy | ~76.8% |

---

## VDB Swing Signal — ❌ NOT VIABLE

| Property | Value |
|----------|-------|
| VDB θ | θ_I=15, θ_E=25 |
| Val accuracy | ~57.4% |
| Status | Dropped — too close to random |

---

## Legacy Models (DEPRECATED)

All replaced by VDB architecture v7.x.

| Model | Status |
|-------|--------|
| Regime Detector (`regime.onnx`) | Deprecated — VDB bars replace regime gate |
| Trend Signal (`signal.onnx`, 18 features, 1000-tick bars) | Deprecated — replaced by VDB signal |
| Range Signal (Model 2b) | Abandoned — 3 approaches all failed |
| Adaptive SL/TP (`sltp.pt`) | Abandoned — overfitting |
| Filter v1, Signal 3-class | Superseded |

---

## See Also

| Topik | Baca di |
|-------|---------| 
| EA parameters & dynamic exit | [ea.md](ea.md) |
| Architecture flow | [architecture.md](architecture.md) |
| Decisions history | [decisions.md](decisions.md) |
| Research experiments | [research-log.md](research-log.md) |
