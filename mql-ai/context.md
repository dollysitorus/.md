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
Data (CSV) → Python Pipeline → 3 ONNX Models → MT5 EA (signal-trader-ea)
```

Detail: lihat [architecture.md](architecture.md)

## Conventions

- **Docker-only**: Semua Python execution via `docker compose run --rm dev`
- **Model output**: 2 kelas — P(SELL), P(BUY)
- **Config**: `python/configs/default.yaml`
- **Naming**: camelCase (Python), PascalCase (MQL5)
- **Struktur src/**: `pipeline/`, `training/`, `testing/`

## Repo

- **GitHub**: https://github.com/dollysitorus/mql-ai (private)
- **Context**: `.md/mql-ai/`

## Documentation

| Doc | Isi |
|-----|-----|
| [context.md](context.md) | Overview, status, 3-model plan, production framework |
| [architecture.md](architecture.md) | System design, data pipeline, EA flow |
| [models.md](models.md) | Model specs, performance, 3-model architecture |
| [features.md](features.md) | 18 signal features + 6 regime features |
| [decisions.md](decisions.md) | 11 key decisions log with evidence |
| [scripts.md](scripts.md) | 14 active Python scripts |
| [data.md](data.md) | Raw data, processed data, tick bars, label distribution |
| [ea.md](ea.md) | EA parameters, features, log format, ONNX integration |
| [stress-test-framework.md](stress-test-framework.md) | 5 signal tests, 7 filter tests, labeling strategy, deployment gates |
| [research-log.md](research-log.md) | Kronologis: apa yang dilakukan, hasil, rekomendasi |

### Kapan Update Doc Apa

| Kalau kamu... | Update file ini |
|---------------|----------------|
| Train model baru | `models.md`, `research-log.md`, `scripts.md` (jika ada script baru) |
| Tambah/ubah features | `features.md`, `models.md` (input berubah) |
| Ubah EA parameters | `ea.md` |
| Ubah SL/TP atau strategy | `decisions.md`, `ea.md`, `architecture.md` (jika flow berubah) |
| Run stress test / experiment | `research-log.md` (WAJIB), `decisions.md` (jika ada keputusan) |
| Tambah/hapus script | `scripts.md` |
| Ubah data pipeline | `data.md`, `scripts.md`, `architecture.md` |
| Ubah arsitektur | `architecture.md`, `context.md` |
| Buat keputusan penting | `decisions.md` (WAJIB) |
| Deploy / go live | `context.md` → Production Framework, `research-log.md` |

> ⚠️ **WAJIB**: Setiap kali selesai satu task, update `research-log.md` dengan tanggal, apa yang dilakukan, hasil, dan rekomendasi.

### Development Workflow

Urutan kerja untuk setiap model/experiment baru:

```
1. PLAN    → Tulis tujuan, approach, metrics di research-log.md
2. LABEL   → Generate labels (pipeline/generate-labels*.py)
3. TRAIN   → Train model di GPU server (training/train-*.py)
4. TEST    → Stress test (testing/test-stress.py) — SEMUA 5 test WAJIB pass
5. RECORD  → Catat hasil di research-log.md + decisions.md (jika ada keputusan)
6. EXPORT  → Export ONNX (training/export-onnx.py)
7. DEPLOY  → Update EA, compile, demo
```

### Format Catat Hasil Trial

Setiap experiment di `research-log.md` WAJIB format ini:

```
### [Nama Experiment]
**Tanggal:** YYYY-MM-DD
**Apa:** [deskripsi singkat]
**Hasil:**
| Metric | Value |
|--------|-------|
| PF     | x.xx  |
| WR     | xx%   |
| DD     | $xxx  |
| PnL    | $xxx  |
| Trades | xxx   |
**Rekomendasi:** [keep/abandon/modify + alasan]
```


## Status

### 3-Model Architecture (CURRENT DIRECTION)

```
Bar → Model 1 (Regime)  → TRENDING → Model 2a (Trend Signal) → BUY/SELL [SL=$3 TP=$15]
                         → RANGING  → Model 2b (Range Signal) → BUY/SELL [SL=$2 TP=$4]
                         → DEAD     → NO TRADE
```

#### Model 1: Regime Detector (TODO)
- [ ] Generate regime labels (TRENDING / RANGING / DEAD)
- [ ] Train regime classifier
- [ ] Validate F1-macro > 0.70

#### Model 2a: Trend Signal (DONE — signal-2class.pt)
- [x] LSTM 2-class (BUY/SELL), 18 features, lookback=10
- [x] ONNX exported: `signal-2class.onnx`
- [x] Stress tested: PF 2.76+ on filtered (tradeable) bars
- ⚠️ PF drops to 0.98 on ALL bars — needs regime filter

#### Model 2b: Range Signal (TODO)
- [ ] Mean reversion strategy for ranging market
- [ ] SL/TP lebih ketat ($2/$4)

### Validated Findings
- Fixed SL/TP ($3/$15) = most stable for trending
- Confidence threshold 0.55 = optimal (higher is worse)
- Trailing stop $8/$1 = PF 0.99 → 1.16 (game changer)
- Dynamic SL/TP = abandoned (fragile)
- Transition filters = do not improve as standalone
- Session features = no differentiation
- **Root cause PF drop**: model tested on filtered bars (PF 2.76) but live runs all bars (PF 0.98)

### Production Framework
1. **Risk Guardrail** — risk ≤ 0.5%, DD scaling, losing streak pause
2. **Regime Gate** — Model 1 decides tradeable vs skip
3. **Execution Validation** — live demo 2-4 minggu with logging
4. **Stability Review** — assess DD, confidence drift, trade count
5. **Improve** — ONLY after stability proven

### Prinsip
- Research = cari edge | Production = lindungi edge
- Target v1 = stabil, survive regime shift, DD terkendali
- Profit datang setelah stabilitas

### Todo (Next Steps)

**STEP 1: Build Model 1 — Regime Detector**
- [ ] Buat `pipeline/generate-labels-regime.py` — labeling TRENDING/RANGING/DEAD
- [ ] Buat `training/train-regime.py` — train classifier
- [ ] Run stress tests F1-F7 (lihat [stress-test-framework.md](stress-test-framework.md))
- [ ] Validate F1-macro > 0.70
- [ ] Catat hasil di `research-log.md`

**STEP 2: Re-validate Model 2a pada Trending bars**
- [ ] Filter data ke TRENDING only → run test-stress.py
- [ ] Confirm PF > 2.0
- [ ] Catat di `research-log.md`

**STEP 3: Build Model 2b — Range Signal**
- [ ] Buat `pipeline/generate-labels-range.py` — mean reversion labels
- [ ] Buat `training/train-range-signal.py`
- [ ] Stress test pada RANGING bars
- [ ] Catat di `research-log.md`

**STEP 4: Full Pipeline Simulation**
- [ ] Gabung 3 model: Regime → Trend/Range
- [ ] Simulasi pada ALL bars
- [ ] Target: PF > 1.5 overall

**STEP 5: Deploy**
- [ ] Export 3 ONNX
- [ ] Update EA untuk 3-model pipeline
- [ ] Implement risk guardrails
- [ ] Demo 2-4 minggu

### Known Issues
- ⚠️ `signal-norm.npz` terhapus — regenerate saat training ulang (lihat [decisions.md](decisions.md) D11)
- ⚠️ EA trailing default (3000/500) belum match simulasi optimal (8000/1000) — update saat deploy
- `onnx-ea.mq5` masih ada di repo — deprecated, bisa dihapus
