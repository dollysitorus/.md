# Expert Advisor (EA)

## Active EAs

### VDB Day-Trade EA v7.10 — `vdb-day-ea/vdb-day-ea.mq5` (PRODUCTION)

**Model:** `vdb_daytrade_6_10.onnx` | **Magic:** 260226

#### Core Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpConfThreshold` | 0.80 | Min confidence to enter |
| `InpInitialSL` | $4.0 | Initial stop loss (price) |
| `InpMaxTP` | $25.0 | Hard TP cap (safety) |
| `InpLotSize` | 0.01 | Fixed lot size |
| `InpMagic` | 260226 | Magic number |
| `InpMaxPositions` | 3 | Max simultaneous positions |
| `InpMinBarsBetween` | 10 | Min bars between entries (~30s) |
| `InpMaxSpreadPts` | 300 | Max spread (3-digit points) |
| `InpCooldownBars` | 5 | Bars to skip after loss |
| `InpSlippage` | 30 | Max slippage (points) |

#### Dynamic Exit Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpFlipConfMin` | 0.75 | Signal flip exit min confidence |
| `InpBreakevenAt` | $2.0 | Move SL to breakeven at this profit |
| `InpMaxBarsHold` | 200 | Max bars to hold (~10 min) |

#### Asymmetric Trailing Logic

| Profit Level | Lock % | SL Position | Max Giveback |
|-------------|--------|-------------|-------------|
| $0 - $2 | 0% | Initial ($-4) | $4 |
| $2 - $4 | 0% | Breakeven | $2-$4 |
| $4 - $8 | 50% | entry + 50% | $2-$4 |
| $8 - $12 | 65% | entry + 65% | $2.8-$4.2 |
| $12+ | 80% | entry + 80% | $2.4-$4 |

#### Auto Lot

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpAutoLot` | false | Enable auto lot |
| `InpRiskPct` | 1.0% | Risk % per trade |

#### Logging

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpLogToFile` | true | CSV log signals & trades |

---

### VDB Scalp EA v7.00 — `vdb-scalp-ea/vdb-scalp-ea.mq5`

**Model:** `vdb-signal.onnx` | **Magic:** 260227

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpConfThreshold` | 0.80 | Min confidence |
| `InpTP_Price` | $5.0 | Fixed take profit |
| `InpSL_Price` | $2.0 | Fixed stop loss |
| `InpMaxSpreadPts` | 300 | Max spread |
| `InpCooldownBars` | 5 | Loss cooldown |

---

## VDB Bar Mechanic

```
Each tick:
  mid = (bid + ask) / 2
  E += |mid - prev|        ← Energy (activity)
  I += (mid - prev)        ← Imbalance (direction)

Bar closes when: |I| >= θ_I AND E >= θ_E
```

| EA | θ_I | θ_E | ~Duration |
|----|-----|-----|-----------|
| Day-Trade | 6 | 10 | 3.2s |
| Scalp | 3 | 5 | 1.3s |

## ONNX Integration

- Model embedded via `#resource "\\Files\\vdb_daytrade_6_10.onnx"`
- Input: `[1, 30, 15]` (30 bars × 15 features)
- Output: `[1, 2]` → softmax → P(SELL), P(BUY)
- Normalization: Z-score with training mean/std (hardcoded in EA)
- Inference triggered: on VDB bar close (event-driven, not every tick)
- Warmup: 30 bars needed before first prediction

## HUD Display

```
VDB DayTrade v7.10 [READY]
Bars: 35/30 | Total: 127
Spread: 254 (max 300)
E=3.2 I=2.1 |I|/th=35% Teff=8
Last: IR=0.687 E=12.3 Dur=3.1s
3 pos | unrealized: $4.50
WR: 72% (8W/3L) | Net: $15.60
```

### HUD Elements
| Line | Content |
|------|---------|
| Title | Version + status (PREFILL/WARMUP/READY) |
| Bars | Current buffer / required + total built |
| Spread | Current vs max allowed (color: green/red) |
| VDB Accumulation | E, I, % to threshold, effective ticks |
| Last Bar | Imbalance ratio, energy, duration |
| Position | Count + aggregate unrealized PnL |
| Stats | Win rate + net PnL (today, with commission) |

## Statistics Tracking

- **Scope**: Today (from 00:00 server time)
- **Persistent**: Restored from deal history on EA init (survives restart)
- **Filter**: Magic number + symbol (pure EA trades only)
- **PnL**: Net = `DEAL_PROFIT + DEAL_COMMISSION + DEAL_SWAP`

## Log Format (CSV)

```
Time, Duration, E, I, IR, P_BUY, P_SELL, Conf, Signal, Spread
```

---

## Deprecated EAs

| EA | File | Status |
|----|------|--------|
| Router EA v6.00 | `router-ea/router-ea.mq5` | Deprecated — replaced by VDB v7.x |
| Signal Trader v5.00 | `signal-trader-ea.mq5` | Deprecated |
| ONNX EA | `onnx-ea.mq5` | Can be deleted |

---

## See Also

| Topik | Baca di |
|-------|---------|
| Model specs | [models.md](models.md) |
| Architecture flow | [architecture.md](architecture.md) |
| Research experiments | [research-log.md](research-log.md) |
