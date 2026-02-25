# Stress Test Framework

Framework untuk validasi model sebelum live deployment.

---

## Bug Fixes (sudah diterapkan di `test-stress.py`)

| Bug | Masalah | Fix |
|-----|---------|-----|
| BUG 1 | Timeout PnL double-subtract slippage | Exit = close ± halfSpread only |
| BUG 2 | TP/SL scaling ambiguity (pip/tick/price) | Absolute price units only |
| BUG 3 | Overlapping trades | `i = exit_index + 1` |
| BUG 4 | Optimistic OHLC fill | If TP+SL same bar → assume SL (worst case) |

## Execution Realism Checklist

- [x] BUY entry at ask, SELL entry at bid
- [x] BUY exit at bid, SELL exit at ask
- [x] Slippage realistic (≥ 1-3 ticks)
- [x] Absolute spread used
- [x] Non-overlapping trades
- [x] Consistent TP/SL units
- [x] Normalization from training set only

---

## Signal Model Stress Tests (5 tests)

### Test A — Fixed Spread Sweep
Spread: 0.10, 0.15, 0.20, 0.25, 0.30, 0.40, 0.50

**Pass:** PF > 2 at spread=0.30, PF > 1.5 at spread=0.50

### Test B — Walk-Forward Per Year
Train 3 years → test 1 year, repeat across all years.

**Pass:** PF > 2 each window, no collapse

### Test C — Label Shuffle (Leakage Check)
Shuffle labels randomly → retrain → measure F1.

**Pass:** F1 ≈ 0.50, PF ≈ 1 (proves no leakage)

### Test D — Label Shift
Offset labels by N bars → must collapse.

**Pass:** Performance collapses

### Test E — Brutal Slippage Sweep
Slippage: 0.00-0.00, 0.02-0.05, 0.05-0.10, 0.10-0.20

**Pass:** PF > 2 at slippage 0.10-0.20

---

## Filter/Regime Model Tests (7 tests)

Berlaku untuk Model 1 (Regime Detector) dan model filter lainnya.

### Test F1 — Delta Expectancy
Expectancy per trade harus meningkat. Filter harus naikkan risk-adjusted return, bukan cuma kurangi trades.

### Test F2 — Random Gate Benchmark
Buat random filter: `trade if random() < X`. Model harus beat random.

### Test F3 — Shuffle Test (Leakage)
Shuffle labels → retrain → F1 harus collapse ke ~0.50.

### Test F4 — Label Shift
Shift filter labels +50, +100 bars → performance harus collapse.

### Test F5 — Walk-Forward
Train 3yr → test 1yr. Filter harus improve PF di setiap window.

### Test F6 — Threshold Stability
Test range: 0.55, 0.60, 0.65, 0.70. Performance harus stabil.

### Test F7 — Trade Distribution
Monthly trade count stabil, equity smooth, tidak bergantung 1-2 trade besar.

**Pass criteria keseluruhan:**
- PF naik ≥ 10-20%
- Sharpe naik
- DD turun
- Trade count tidak drop > 50%

---

## Dynamic SL/TP Tests (7 tests) — ABANDONED

> ⚠️ Dynamic SL/TP sudah ditinggalkan (lihat `decisions.md` D9). Framework ini disimpan untuk referensi jika revisit nanti.

### Test 1 — Spread & Slippage Sweep
### Test 2 — Walk-Forward Per Year
### Test 3 — Parameter Stability (grid search multipliers)
### Test 4 — Conservative In-Bar Fill
### Test 5 — Tail-Risk / Monte Carlo (1000 reshuffles)
### Test 6 — Event Regime & Wide Spread
### Test 7 — Sanity Controls (random SL/TP, frozen sigma)

---

## Sizing Model Tests (7 tests) — PHASE 5

> ⏸ Position sizing = Phase 5, hanya setelah stability proven.

### Test S1-S7
Risk-adjusted improvement, trade-level contribution, Monte Carlo, volatility regime, walk-forward, size sensitivity, capital stress.

---

## Labeling Strategy (untuk model baru)

### ❌ Jangan lakukan:
1. Label = future return > 0 → redundant dengan signal
2. Label = only winning trades → hindsight bias
3. Label = raw price tanpa execution sim → mismatch

### ✅ Metode yang direkomendasikan:

**Method A — Edge-Based** (Recommended):
1. Run signal model on training data
2. Simulate trades dengan locked SL/TP
3. Compute R = PnL / SL
4. Label: R ≥ 1.0 → TRADE, else → NO_TRADE

**Method B — Rolling Expectancy**:
Rolling expectancy > 0 → TRADE

**Method C — Confidence Calibration**:
Confidence threshold × volatility context

---

## Red Flags

- Sharpe > 10 consistently → suspek
- PF > 4 in all conditions → suspek
- Ultra-smooth equity → suspek
- PnL concentrated in 1-2 trades → suspek

## Validation Before Live

1. Re-run semua stress tests
2. Check yearly drawdowns
3. Run 1-3 month paper forward test
4. Confirm similar behavior with backtest

---

## Deployment Gates

| Gate | Requirement |
|------|-------------|
| Gate 1 (Research) | All tests pass in Python |
| Gate 2 (Demo) | Fixed model, log live vs sim, paper 1-3 months |
| Gate 3 (Micro Live) | Very small risk, confirm no execution drift |
