# Decisions Log

Semua keputusan arsitektur & riset, urut kronologis.

---

### D1: Fixed SL/TP ($3/$15) vs Dynamic

**Keputusan:** Fixed SL=$3 TP=$15 (R:R 1:5)

**Alasan:**
- Dynamic SL/TP (ATR-based) gagal di 2/4 stress test
- Parameter fragility: hanya 1/24 kombinasi profitable
- Slippage $0.05-0.10 menghancurkan edge dynamic SL/TP

**Evidence:** `test-stress-dynamic.py`, `tune-dynamic-sltp.py`

---

### D2: Signal Model 2-Class (BUY/SELL)

**Keputusan:** 2-class model (BUY/SELL) tanpa HOLD

**Alasan:**
- 3-class (HOLD/BUY/SELL) memiliki F1 lebih rendah
- Model 2-class: fokus prediksi arah saja, filter dikerjakan model lain

**Evidence:** `train-signal.py` (3-class) vs `train-signal-2class.py` (2-class)

---

### D3: Confidence Threshold 0.55

**Keputusan:** Tetap 0.55, tidak dinaikkan

**Alasan:**
- Sweep 0.50-0.80: threshold 0.55 = best expectancy (-$0.04)
- Threshold 0.70: WR turun, PF turun, DD naik
- Model confidence tidak well-calibrated

**Evidence:** `test-confidence.py`

---

### D4: Trailing Stop $8/$1

**Keputusan:** Trailing start=$8 (8000pts), step=$1 (1000pts)

**Alasan:**
- Tanpa trailing: PF=0.98, WR=18%, PnL=-$272
- Dengan trailing $8/$1: PF=1.16, WR=28%, PnL=+$3,809
- Simulasi menunjukkan semua trailing configs profitable

**Evidence:** `test-confidence.py` trailing section

---

### D5: Transition Filter Standalone = Tidak Efektif

**Keputusan:** Transition filter TIDAK diimplementasikan sebagai standalone

**Alasan:**
- Filter blocks 34.4% predictions
- PnL memburuk: -$335 → -$936
- Max losing streak membaik 41→33 (satu-satunya benefit)

**Evidence:** `test-transition-filter.py`, `test-gate.py`

---

### D6: Root Cause — Filtered vs All Bars

**Discovery:** Model PF 2.76+ (stress test) vs 0.98 (live simulation)

**Root cause:**
- Stress test (`test-stress.py`): data pre-filtered ke `label_direction != 0` (41% data)
- Live: model prediksi pada ALL bars termasuk noise
- Model kuat di tradeable bars, lemah di noise bars

**Implikasi:** Butuh Regime Detector

---

### D7: 3-Model Architecture

**Keputusan:** Arsitektur 3 model: Regime → Trend Signal / Range Signal

**Alasan:**
- Signal model punya edge (PF 2.76 on filtered data)
- Edge hilang karena model dipaksa prediksi di non-tradeable bars
- Regime model memisahkan TRENDING vs RANGING vs DEAD
- Setiap regime punya model dan SL/TP berbeda

---

### D8: Log File — Append Mode + Daily

**Keputusan:** Log EA format `AI_Signal_Log_XAUUSD_YYYYMMDD.csv`, append mode

**Alasan:** File sebelumnya pakai FILE_WRITE → data hilang setiap EA restart

**Evidence:** Perubahan di `signal-trader-ea.mq5` OnInit()

---

### D9: Adaptive SL/TP — Abandoned

**Keputusan:** Model adaptive SL/TP ditinggalkan

**Alasan:**
- SL MAE = 23 pips (tinggi relatif terhadap range)
- Confidence-adaptive formula (a=2.0, b=3.0, k=2.0): hanya 1 parameter set profitable
- Fixed SL/TP lebih stable across all conditions

**Evidence:** `test-stress-adaptive.py`, `tune-confidence-sltp.py`

---

### D10: Session Features — No Differentiation

**Keputusan:** Session features tidak ditambahkan

**Alasan:**
- Analisis per session (Asia/London/NY/Dead): SL/TP optimal hampir identik
- Session bukan differentiator kuat untuk SL/TP

**Evidence:** `analyze-sessions.py` (removed, finding documented)

---

### D11: ⚠️ Mistake — Norm File Deleted During Cleanup

**Tanggal:** 2026-02-25

**Apa yang terjadi:**
- Saat cleanup deprecated files, `signal-norm.npz` ikut terhapus
- File ini berisi mean/std per feature (normalization stats dari training set)
- Dibutuhkan oleh `export-onnx.py` untuk re-export ONNX
- File tidak pernah di-commit ke git (binary, gitignored) → tidak bisa recover dari git

**Dampak:**
- Model weights (`signal-2class.pt`) **AMAN — tidak berubah**
- ONNX yang sudah di-export (`signal.onnx`) **AMAN — tidak berubah**
- Tapi kalau perlu re-export ONNX, harus regenerate norm stats dulu

**Recovery:**
- Run training script lagi dengan data + config yang sama → menghasilkan `signal-norm.npz` baru
- Atau extract mean/std dari EA input params `InpNormMeans` / `InpNormStds` jika sudah diset

**Lesson learned:**
> ⚠️ **JANGAN hapus file yang berhubungan dengan model production tanpa verifikasi bahwa file bisa direconstruct.** Selalu cek dependency chain: `.pt` → `.npz` → `.onnx` → EA. Hapus file deprecated HANYA jika yakin tidak ada file production yang bergantung padanya.

---

### D12: Regime Model = Simple Gate (Not Profit Predictor)

**Keputusan:** Regime model hanya sebagai gate sederhana (TREND/NON-TREND), bukan edge predictor.

**Alasan:**
- v2 (profitability labels + regime features): F1=0.47 → gagal total
- v3 (15 edge features + Method A labels): F1=0.50, PF improvement marginal (1.03→1.11)
- v1 (ER-based TREND/NON-TREND): F1=0.85, PF 1.09→1.44, DD turun 5x

**Insight:** Regime model cuma perlu filter "apakah market layak trade" — bukan prediksi profitability. Task profitability diserahkan ke signal model + sizing model.

**Evidence:** `train-regime.py` (v1 ✅), `train-regime-v2.py` (❌), `train-regime-v3.py` (❌)

---

### D13: EA v5.00 — Router Architecture + Hysteresis

**Keputusan:** EA pakai router pattern: regime.onnx → hysteresis → signal.onnx (TREND only)

**Hysteresis:** Rolling mean of 3 P(TREND), asymmetric thresholds enter=0.60 exit=0.40

**Alasan:**
- Bar-by-bar regime switching causes flip-flop → kacau
- Rolling mean 3 bar = structural confirmation (bukan noise)
- Asymmetric: masuk TREND cepat (2 bar), keluar lebih hati-hati (3 bar)

**Evidence:** T5 system test PF=1.44, DD -$162

---

### D14: Range Model (Model 2b) — DROPPED

**Tanggal:** 2026-02-25
**Keputusan:** Range model abandoned. EA proceeds as 2-model only (regime + trend signal).
**Alasan:** 3 approaches tried (LSTM×2 + XGBoost). XGBoost best but failed 2/4 stress tests (spread sweep PF→1.01, walk-forward unstable). Edge too small for real costs.
**Evidence:** `test-range-threshold.py`, stress test results in research-log.md

---

### D15: Feature Parity — 7 Formula Fixes

**Tanggal:** 2026-02-25
**Keputusan:** Fix all 7 formula mismatches between Python training and EA inference.
**Alasan:** OFI, cumDelta, twap_deviation, tick_intensity, spread_compression, rolling_sigma, spread_mean all had different formulas. Model predictions meaningless with wrong features.
**Evidence:** `Feature_Parity_Audit_Checklist.txt`

---

### D16: EA v6.00 — Router EA (replaces v5.00/v5.10)

**Tanggal:** 2026-02-25
**Keputusan:** New EA `router-ea.mq5` v6.00 with regime exit handling, loss cooldown, windowed cumDelta, confidence 0.60, configurable BE offset, log level filtering.
**Alasan:** V5.10 analysis identified 8 correctness/risk issues. All fixed in v6.00.
**Evidence:** `EA_v6_00_Analysis_Checklist.txt`

---

## See Also

| Topik | Baca di |
|-------|---------|
| Timeline lengkap semua riset | [research-log.md](research-log.md) |
| Model specs & performance | [models.md](models.md) |
| Framework stress test | [stress-test-framework.md](stress-test-framework.md) |
| EA parameters aktual | [ea.md](ea.md) |
| Features yang diuji | [features.md](features.md) |
