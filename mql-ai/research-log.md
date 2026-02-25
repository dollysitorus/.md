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
**Apa:** Build 9 comprehensive docs di `.md/mql-ai/`
**Status:** DONE

---

## See Also

| Topik | Baca di |
|-------|---------|
| Keputusan yang diambil dari riset | [decisions.md](decisions.md) |
| Model specs & status | [models.md](models.md) |
| Framework validasi | [stress-test-framework.md](stress-test-framework.md) |
| Architecture overview | [architecture.md](architecture.md) |
| Todo & roadmap | [context.md](context.md) → Todo |
