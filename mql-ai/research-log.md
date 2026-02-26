# Research Log

Catatan kronologis semua aktivitas, hasil, dan rekomendasi.

---

## 2026-02-23

### Data Pipeline Setup
**Apa:** Build pipeline dari raw ticks → clean → bars → features → labels
**Hasil:**
- 215M ticks → ~100K tick bars (100 ticks/bar)
- 18 features di-engineer
- Labels: BUY (~20%), SELL (~20%), HOLD (~60%)

**Rekomendasi:** Pipeline stabil. Data siap training.

---

### Signal Model — 3-Class (HOLD/BUY/SELL)
**Apa:** Train LSTM 3-class model
**Hasil:** F1 rendah — model kesulitan learn HOLD class
**Rekomendasi:** Ganti ke 2-class (BUY/SELL saja, hapus HOLD)

---

### Signal Model — 2-Class (BUY/SELL) ✅
**Apa:** Train LSTM 2-class model (signal-2class.pt)
**Hasil:**
- Training converge dengan baik
- Stress test PF 2.76+ di semua spread/slippage levels
- Walk-forward stabil per year

**Rekomendasi:** Ini production model. Lock dan jangan ubah.

---

### Stress Test Audit (5 Bug Fixes)
**Apa:** Audit simulation engine, fix 4 bugs (double slippage, SL/TP units, overlap, OHLC fill)
**Hasil:** Bug-free stress test engine
**Rekomendasi:** Semua model baru harus pakai engine ini.

---

### ONNX Export + EA Build
**Apa:** Export model ke ONNX, build EA (signal-trader-ea.mq5) v4.00
**Hasil:**
- ONNX inference berjalan di MT5
- EA trade live di XAUUSD

**Rekomendasi:** EA siap demo.

---

## 2026-02-24

### Dynamic SL/TP Experiment ❌
**Apa:** Train model adaptive SL/TP berdasarkan volatility
**Hasil:**
- SL MAE = 23 pips (terlalu tinggi)
- Hanya 1/24 parameter set profitable
- 2/4 stress test failed

**Rekomendasi:** ABANDONED. Fixed SL/TP ($3/$15) lebih robust.

---

### Confidence Threshold Sweep
**Apa:** Sweep confidence 0.50-0.80
**Hasil:**
- 0.55 = best expectancy
- Higher confidence ≠ better trades (model not well-calibrated)

**Rekomendasi:** Fixed 0.55. Jangan naikkan.

---

### Transition Filter Experiment ❌
**Apa:** Filter trade berdasarkan sigma_z, ATR ratio, delta divergence, spread
**Hasil:**
- Block 34.4% trades
- PnL MEMBURUK: -$335 → -$936
- Hanya losing streak turun (41→33)

**Rekomendasi:** ABANDONED sebagai standalone. Regime model lebih baik.

---

### Trailing Stop Discovery ✅
**Apa:** Test trailing stop configurations
**Hasil:**
| Config | WR | PF | PnL |
|--------|------|------|-------|
| No trail | 18% | 0.98 | -$272 |
| Trail $5/$1 | 37.7% | 1.22 | +$5,444 |
| Trail $8/$1 | 28% | 1.16 | +$3,809 |

**Rekomendasi:** Trailing stop = game changer. Implement $8/$1 di EA.

---

### Session Analysis
**Apa:** Analisis performance per session (Asia/London/NY/Dead)
**Hasil:** Tidak ada perbedaan signifikan antar session
**Rekomendasi:** Session bukan differentiator. Skip.

---

### Root Cause Discovery — Filtered vs All Bars ⚠️
**Apa:** Investigasi kenapa stress test PF 2.76 tapi live sim PF 0.98
**Hasil:**
- Stress test: `label_direction != 0` (filter 41% data) → PF 2.76
- Live: all bars → model dipaksa prediksi noise → PF 0.98
- Model BAGUS di tradeable bars, LEMAH di noise bars

**Rekomendasi:** Butuh Regime Detector model untuk filter non-tradeable bars.

---

## 2026-02-25

### 3-Model Architecture Decision
**Apa:** Arsitektur baru: Regime → Trend Signal / Range Signal
**Alasan:** Menyelesaikan root cause PF drop
**Status:** PLANNED — belum diimplementasi

---

### EA Log Fix
**Apa:** Log file overwrite setiap restart → data hilang
**Fix:** Append mode + filename per hari (YYYYMMDD)
**Status:** DONE

---

### Codebase Cleanup
**Apa:** Remove 23 deprecated scripts, 6 deprecated models, 5 .txt files
**Reorganize:** `src/` → `pipeline/`, `training/`, `testing/`
**⚠️ Mistake:** `signal-norm.npz` ikut terhapus (lihat decisions.md D11)
**Status:** DONE

---

### Documentation Overhaul
**Apa:** Build 10 comprehensive docs di `.md/mql-ai/`
**Status:** DONE

---

### Regime Detector — 3-Class LSTM ❌
**Tanggal:** 2026-02-25
**Apa:** Train LSTM 3-class (TRENDING/RANGING/DEAD) dengan 6 regime features
**Hasil:**
| Metric | Value |
|--------|-------|
| MacroF1 | 0.4570 |
| Accuracy | 62.4% |
| TRENDING F1 | 0.49 (precision 0.44, recall 0.56) |
| RANGING F1 | 0.71 (precision 0.76, recall 0.66) |
| DEAD F1 | 0.17 (precision 0.30, recall 0.12) |

**Masalah:** DEAD unlearnable (2.5%), TREND/RANGE overlap, label definition noisy
**Rekomendasi:** Pivot ke 2-class (TREND vs NON-TREND). Tambah better features (ER, flip rate).

---

### Regime Detector — 2-Class LSTM ✅
**Tanggal:** 2026-02-25
**Apa:** Train LSTM 2-class (TREND vs NON-TREND) dengan 8 improved features
**Features:** efficiency_ratio, flip_rate, vol_compression, trend_strength, slope_consistency, range_width_ratio, directional_strength, delta_consistency
**Hasil:**
| Metric | Value |
|--------|-------|
| MacroF1 | **0.8529** |
| Accuracy | 86.9% |
| TREND precision | 77.0% |
| TREND recall | 84.1% |
| NON-TREND precision | 92.2% |
| NON-TREND recall | 88.2% |
| Distribution | TREND 33.7%, NON-TREND 66.3% |

**Key improvement:** ER-based labeling + 8 regime-specific features vs 6 σ-based features
**Model saved:** `regime.pt` + `regime-norm.npz` di GPU server

---

### Regime v1 (ER-based, 2-class) ✅ PRODUCTION
**Tanggal:** 2026-02-25
**Apa:** Train regime detector 2-class (TREND/NON-TREND) dengan 8 features
**Hasil:**

| Metric | Value |
|--------|-------|
| F1 | 0.8531 |
| TREND precision | 0.77 |
| TREND recall | 0.84 |

**T5 System-Level:**

| Metric | ALL bars | TREND only |
|--------|----------|------------|
| PF | 1.09 | **1.44** |
| DD | -$863 | **-$162** |
| Per-trade PnL | $0.21 | **$1.03 (4.9x)** |
| Total PnL | $1,519 | **$3,168** |

**Rekomendasi:** Production-ready sebagai gate sederhana. ✅

---

### Regime v2 (Method A labels + old features) ❌ ABANDONED
**Tanggal:** 2026-02-25
**Apa:** Coba label pakai R≥1.0 → TRADE, dengan 8 regime features
**Hasil:** F1=0.47 — features tidak cocok untuk predict profitability
**Rekomendasi:** Abandoned. Regime sebaiknya cuma gate, bukan prediksi profit.

---

### Regime v3 (Method A + 15 edge features) ❌ ABANDONED
**Tanggal:** 2026-02-25
**Apa:** 15 features baru + LSTM + Method A labels
**Hasil:** Macro F1=0.50, PF improvement marginal (1.03→1.11 at p30)
**Rekomendasi:** Abandoned. LSTM+tabular features = wrong tool. Kembali ke v1.

---

### EA v5.00 — Router + Trend Engine ✅
**Tanggal:** 2026-02-25
**Apa:** Update EA dari single-model (v4.00) ke 2-model router architecture
**Perubahan:**
- Regime model (router) + Signal model (trend engine)
- Hysteresis: rolling mean of 3 P(TREND), asymmetric (enter 0.60, exit 0.40)
- 30 bar warmup sebelum trading
- HUD menampilkan regime status
- Norm values hardcoded sebagai default

**File:** `signal-trader-ea.mq5` v5.00, `regime.onnx` + `signal.onnx`

---

### Signal Norm Regeneration
**Tanggal:** 2026-02-25
**Apa:** Generate `signal-norm.npz` dari filtered training data (67,830 bars)
**Catatan:** Model `signal-2class.pt` TIDAK diubah — hanya generate norm file untuk deployment

---

### Range Model (Model 2b) — All Approaches FAILED ❌
**Tanggal:** 2026-02-25
**Apa:** 3 attempts to build a mean reversion model for NON-TREND bars

**Attempt 1 — LSTM + Triple Barrier Labels:**
- F1=0.33 (random), model tidak bisa learn triple barrier patterns

**Attempt 2 — LSTM + Confirmed Reversion Labels (Option C):**
- z-score > 1 + reversion within 15 bars + $2 SL
- Labels: BUY 11.5%, SELL 11.4%, HOLD 77.1%
- Best F1=0.49, severe overfitting (train 70% / val 60%)

**Attempt 3 — XGBoost + Lag Features:**
- F1=0.6963 ✅ (passed initial threshold)
- Top features: z_twap (0.45), bb_pos (0.42)
- **Stress test: FAILED 2/4**
  - Spread sweep: PF drops to 1.01 at $0.30 spread (edge too small)
  - Walk-forward: 3/4 windows PF ≈ 1.01-1.05 (doesn't generalize)
  - Label shuffle: PASSED (no leakage)
  - System-level: PASSED (PF 1.05 vs random 0.84)

**Rekomendasi:** DROP range model. Edge terlalu kecil untuk survive trading costs. EA proceed TREND-only.

---

### Feature Parity Audit ⚠️
**Tanggal:** 2026-02-25
**Apa:** Systematic comparison Python training pipeline vs MQL5 EA live inference (9 sections)
**Hasil:** **7 critical formula mismatches** in signal features:

| Feature | Python (correct) | EA (was wrong) |
|---------|-------------------|----------------|
| OFI | `delta/tick_count` | `delta/(buy+sell)` |
| cumulative_delta | global running sum | per-bar reset |
| twap_deviation | `close - twap` (raw) | `(close-twap)/twap` (relative) |
| tick_intensity | `ticks/duration_sec` | raw count |
| spread_compression | `spread/MA(spread,20)` | `spread/spreadRange` |
| rolling_sigma | `std(log_returns)` across 20 bars | within-bar std |
| spread_mean | absolute | relative to mid |

Plus: delta_ma/divergence/cd_slope simplified (no rolling window), wide_spread_ratio 5x→2x threshold.
Regime features (all 8), normalization, model shapes, softmax, router, execution: all ✅ matched.

**Rekomendasi:** All 7 mismatches fixed in EA v5.10, carried forward to v6.00.

---

### EA v6.00 — Router EA ✅
**Tanggal:** 2026-02-25
**Apa:** New EA (`router-ea.mq5`) addressing all V5.10 analysis issues + feature parity fixes
**File:** `mql5/experts/router-ea/router-ea.mq5`

**8 Fixes Applied:**
| ID | Fix |
|----|-----|
| D1 | Regime exit handling (close at loss, tighten SL at profit) |
| C1 | Windowed cumulative delta (sum last 100 bars, no restart drift) |
| F1 | Loss cooldown (skip 3 bars after loss, configurable) |
| E1 | Trailing BUY fix (currentSL==0 check) |
| G1 | Confidence default 0.60 (was 0.55) |
| H1 | Buffer 120 bars (was 25) |
| I1 | Log flush per bar (was per event) + InpLogLevel |
| P1-02 | cd_slope consistent formula for all bar counts |

**Future improvements (deferred):**
- B1: Hybrid tick bars (tick cap + time cap) — requires retrain
- J1: Simulate hysteresis in training pipeline — requires retrain

---

### EA v6.00 Python Simulation ✅
**Tanggal:** 2026-02-25
**Apa:** Full simulation of EA v6.00 logic in Python (regime hysteresis, cooldown, regime exit, trailing, conf 0.60)
**Script:** `python/src/testing/simulate-ea-v6.py`
**Hasil:**

| Metric | v5.00 (T5) | v6.00 |
|--------|------------|-------|
| PF | 1.44 | **1.58** ✅ |
| DD | -$162 | **-$64** ✅ |
| PnL | $3,168 | **$3,216** |
| Trades | 3,075 | 3,260 |
| WR | — | 44.0% |
| Per Trade | $1.03 | $0.99 |

**Exit types:** SL 84.1%, regime exit 10.8%, TP 5.0%
**Cooldown:** skipped 5,472 bars after losses

**Rekomendasi:** Confirmed v6.00 improvements. Continue with demo account monitoring.

---

## 2026-02-25

### VDB Pipeline — Bar Type Comparison
**Apa:** Built VDB (Value-Driven Bars) pipeline. Compared 3 θ configurations.
**Hasil:**

| Config | θ_I | θ_E | Median Duration | Val Accuracy | Viable? |
|--------|-----|-----|-----------------|-------------|---------|
| Scalp | 3 | 5 | ~1.3s | 76.8% | ✅ |
| Day-Trade | 6 | 10 | ~3.2s | 70.0% | ✅ |
| Swing | 15 | 25 | ~11.3s | 57.4% | ❌ |

**Rekomendasi:** Day-trade dan scalp viable. Swing terlalu dekat random — dropped.

---

### VDB Day-Trade Model — Performance @ Confidence Thresholds
**Apa:** Analyze day-trade model at various confidence levels.
**Hasil:**

| Conf | Trades | WR | PF |
|------|--------|-----|-----|
| ≥0.60 | ~150K | 65% | 3.82 |
| ≥0.70 | ~110K | 72% | 6.14 |
| ≥0.80 | ~75K | 82.7% | 11.93 |
| ≥0.90 | ~30K | 89% | 18+ |

**Rekomendasi:** conf≥0.80 sweet spot — high PF, sufficient trade count.

---

### EA v7.00 — Two Separate EAs
**Apa:** Built two EAs: day-trade (magic 260226) and scalp (magic 260227).
**Hasil:**
- Day-trade: θ=6/10, TP=$10/SL=$4 (fixed), conf≥0.80
- Scalp: θ=3/5, TP=$5/SL=$2 (fixed), conf≥0.80
- Separate magic numbers, HUD prefixes, ONNX models
- Both compile and deploy successfully

**Rekomendasi:** Run both on demo. Day-trade = primary.

---

## 2026-02-26

### EA v7.10 — Dynamic Exit (Optimal Model)
**Apa:** Replaced fixed TP/SL with intelligent exit management.
**Changes:**
- Removed conf decay exit (too aggressive, cuts winners)
- Signal flip: only at conf≥0.75 (was 0.55)
- Time stop: 200 bars (was 50)
- Asymmetric trailing: 0%→50%→65%→80% based on profit
- Breakeven lock at $2 profit
- Multi-position: max 3, min 10 bars spacing
- WR/PnL tracking: today scope, commission+swap included
- Stats survive EA restart (loaded from deal history)

**Hasil:**
- EA compiles and runs
- Live test: model producing signals, entries executing, positions managed
- Dynamic exit logic verified (signal flip hold, trailing engagement)

**Status:** ⚠️ Exit parameters NOT backtested yet. Need Python simulation to validate.

**Rekomendasi:** Run Python simulation before optimizing parameters. Demo 1-2 days minimum.

---

## See Also

| Topik | Baca di |
|-------|---------|
| Keputusan yang diambil dari riset | [decisions.md](decisions.md) |
| Model specs & status | [models.md](models.md) |
| Architecture overview | [architecture.md](architecture.md) |
| Todo & roadmap | [context.md](context.md) → Todo |

