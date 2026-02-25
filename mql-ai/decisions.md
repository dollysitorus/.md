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

**Evidence:** `analyze-sessions.py`
