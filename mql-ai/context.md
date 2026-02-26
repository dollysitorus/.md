# mql-ai

## Overview

MQL5 Expert Advisor dengan model AI ONNX untuk automated trading di MetaTrader 5.

## Stack

| Layer | Tech | Keterangan |
|-------|------|------------|
| Trading | MQL5 | Expert Advisor + indicators di MT5 |
| ML Training | Python + PyTorch | Training model AI |
| Model Format | ONNX | Portable format, inference di MT5 |
| Data | VDB (Value-Driven Bars) | Event-driven bars, bukan time-based |
| Dev Container | python:3.12-alpine | Docker-only development |

## Architecture (CURRENT — v7.10)

```
Tick Data → VDB Bar Builder → 15 Features → ONNX Model → Entry/Exit → MT5
           θ_I=6, θ_E=10                    LSTM 2-class
```

Detail: lihat [architecture.md](architecture.md)

## Active EAs

| EA | File | Model | Magic | θ | TP/SL |
|----|------|-------|-------|---|-------|
| Day Trade v7.10 | `vdb-day-ea/vdb-day-ea.mq5` | `vdb_daytrade_6_10.onnx` | 260226 | 6/10 | Dynamic (asymmetric trail) |
| Scalp v7.00 | `vdb-scalp-ea/vdb-scalp-ea.mq5` | `vdb-signal.onnx` | 260227 | 3/5 | Fixed $5/$2 |

## Conventions

- **Docker-only**: Semua Python execution via `docker compose run --rm dev`
- **Model output**: 2 kelas — P(SELL), P(BUY)
- **Config**: `python/configs/default.yaml`
- **Naming**: camelCase (Python), PascalCase (MQL5)
- **Struktur src/**: `pipeline/`, `training/`, `testing/`

## Agent Rules (WAJIB DIPATUHI)

### Saat Menerima Ide / Request Baru
1. **Baca `context.md` DULU** — pahami status, architecture, known issues
2. **Cek apakah ide sesuai dengan architecture** — VDB pipeline atau menyimpang?
3. **Cek `decisions.md`** — apakah ide ini sudah pernah dicoba dan gagal?
4. **Cek `research-log.md`** — apakah ada experiment terkait sebelumnya?
5. **Diskusikan dengan user** — jelaskan impact terhadap architecture, jangan langsung implement
6. **Buat plan** — tulis di research-log apa yang akan dilakukan, metrics target
7. **Baru implement** — ikuti Development Workflow

### Saat Implementasi
1. **Ikuti Development Workflow** — jangan skip step
2. **JANGAN hapus file tanpa cek dependency chain** — `.pt` → `.npz` → `.onnx` → EA
3. **JANGAN ubah model production** tanpa approval user
4. **Catat SEMUA hasil** di `research-log.md` — termasuk yang gagal
5. **Update docs sesuai tabel "Kapan Update Doc Apa"**
6. **Push kedua repo** (mql-ai + .md) setiap selesai task

### Anti-Pattern (DILARANG)
- ❌ Implement tanpa baca context dulu
- ❌ Hapus file tanpa verifikasi dependency
- ❌ Skip stress test
- ❌ Lupa update `research-log.md`
- ❌ Push satu repo, lupa repo satunya

## Repo

- **GitHub**: https://github.com/dollysitorus/mql-ai (private)
- **Context**: `.md/mql-ai/`

## Documentation

| Doc | Isi |
|-----|-----|
| [context.md](context.md) | Overview, status, architecture, production framework |
| [architecture.md](architecture.md) | System design, VDB pipeline, EA flow |
| [models.md](models.md) | Model specs, performance, VDB signal models |
| [features.md](features.md) | 15 VDB signal features |
| [decisions.md](decisions.md) | Key decisions log with evidence |
| [scripts.md](scripts.md) | Active Python scripts |
| [data.md](data.md) | Raw data, processed data, VDB bars |
| [ea.md](ea.md) | EA parameters, features, log format, ONNX integration |
| [research-log.md](research-log.md) | Kronologis: apa yang dilakukan, hasil, rekomendasi |

### Kapan Update Doc Apa

| Kalau kamu... | Update file ini |
|---------------|----------------|
| Train model baru | `models.md`, `research-log.md` |
| Tambah/ubah features | `features.md`, `models.md` |
| Ubah EA parameters | `ea.md` |
| Ubah SL/TP atau strategy | `decisions.md`, `ea.md`, `architecture.md` |
| Run stress test / experiment | `research-log.md` (WAJIB) |
| Ubah arsitektur | `architecture.md`, `context.md` |
| Buat keputusan penting | `decisions.md` (WAJIB) |
| Deploy / go live | `context.md`, `research-log.md` |

> ⚠️ **WAJIB**: Setiap kali selesai satu task, update `research-log.md`.

## Status

### VDB Day-Trade Architecture (CURRENT — v7.10)

```
Tick → VDB Bar Builder (θ_I=6, θ_E=10)
     → 15 features (normalized z-score)
     → ONNX LSTM (SEQ_LEN=30, FEAT=15)
     → P(BUY), P(SELL) + confidence
     → Entry: conf >= 0.80, SL=$4 initial, MaxTP=$25
     → Exit: asymmetric trail + signal flip + time stop
```

#### VDB Day-Trade Signal Model ✅ DONE
- LSTM 2-class, val accuracy ~70.0%
- @ 80% conf threshold: WR=82.7%, PF=11.93
- File: `vdb_daytrade_6_10.onnx`
- θ_I=6, θ_E=10 (~3.2s median bar duration)

#### VDB Scalp Signal Model ✅ DONE
- LSTM 2-class, val accuracy ~76.8%
- File: `vdb-signal.onnx`
- θ_I=3, θ_E=5 (~1.3s median bar duration)

#### EA v7.10 Day-Trade ✅ DONE
- [x] VDB bar builder with event-driven closure
- [x] 15-feature engineering
- [x] ONNX inference on bar close
- [x] Dynamic exit: asymmetric trailing (50%→65%→80%)
- [x] Signal flip exit (conf >= 0.75)
- [x] Time stop (200 bars ~10 min)
- [x] Multi-position (max 3, min 10 bars between entries)
- [x] WR/PnL tracking (today, with commission+swap)
- [x] HUD: live PnL, WR, positions, spread
- [x] PreFill: historical tick warmup

#### EA v7.00 Scalp ✅ DONE
- [x] Separate EA for scalping model
- [x] Fixed TP=$5/SL=$2
- [x] Magic 260227

### Legacy (DEPRECATED)

> v6.00 router-ea (2-model architecture with regime detector) is **deprecated**.
> v5.00 signal-trader-ea is **deprecated**.
> All replaced by VDB-based architecture v7.x.

### Todo (Next Steps)

**STEP 1: Validate Dynamic Exit** ⭐ CURRENT
- [ ] Python simulation with tick data to compare fixed vs dynamic exit
- [ ] Demo trading 1-2 days, observe logs
- [ ] Optimize asymmetric trail breakpoints from data

**STEP 2: Production**
- [ ] Demo 2-4 weeks with logging
- [ ] Monitor WR, PnL, DD daily
- [ ] Tune parameters if needed

**STEP 3: Improvements (AFTER stability proven)**
- [ ] VDB swing model (θ_I=15, θ_E=25) — val acc 57.4%, not viable yet
- [ ] Confidence-weighted lot sizing
- [ ] Multi-timeframe correlation

### Known Issues
- ⚠️ Asymmetric trail breakpoints ($2/$4/$8/$12) belum dioptimasi dari data
- ⚠️ Signal flip conf≥0.75 belum validated — mungkin terlalu ketat/longgar
- ⚠️ **GPU server training files SUDAH DIHAPUS** (2026-02-25). Jika perlu retrain:
  - Upload ulang `ticks_XAUUSD.csv` ke server
  - Re-run full VDB pipeline
  - ONNX yang sudah di-deploy **AMAN — sudah embedded di EA**
