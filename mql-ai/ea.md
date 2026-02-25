# Expert Advisor (EA)

## File
`mql5/experts/signal-trader-ea.mq5` — v4.00

## Input Parameters

### Core Trading

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpStopLoss` | 3000 pts ($3) | Stop loss |
| `InpTakeProfit` | 15000 pts ($15) | Take profit |
| `InpMaxSpread` | 300 pts ($0.30) | Max spread untuk buka posisi |
| `InpEmergencySpread` | 1000 pts ($1) | Emergency close all |
| `InpMinConfidence` | 0.55 | Min prediction confidence |
| `InpMagic` | 20260223 | Magic number |

### Lot Sizing

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpAutoLot` | false | Auto lot sizing |
| `InpFixedLot` | 0.01 | Fixed lot (jika auto=off) |
| `InpRiskPercent` | 1.0% | Risk per trade (jika auto=on) |

### Trailing Stop

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpTrailingStop` | true | Enable trailing |
| `InpTrailStart` | 3000 pts ($3) | Start trailing setelah profit X |
| `InpTrailStep` | 500 pts ($0.50) | Trail step |

> ⚠️ Default trailing di EA: start=3000, step=500. Simulasi optimal: start=8000, step=1000. **Perlu di-update!**

### Weekend Filter

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpWeekendFilter` | true | Tutup posisi sebelum weekend |
| `InpFridayStopGMT` | 20 | Stop new trades (GMT) |
| `InpFridayCloseGMT` | 21 | Force close all (GMT) |

### Drawdown Protection

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpDrawdownKill` | true | Kill switch jika DD terlalu besar |
| `InpMaxDrawdownPct` | 15.0% | Max DD dari peak equity |

### Time Filter

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpTimeFilter` | true | Filter jam trading |
| `InpTimeStart` | 8 | Trading start (server time) |
| `InpTimeEnd` | 20 | Trading end (server time) |

### Daily Limit

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpDailyLimit` | false | Enable daily loss limit |
| `InpMaxDailyLoss` | $50 | Max daily loss |

### Display & Logging

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpShowHUD` | true | On-chart heads up display |
| `InpLogToFile` | true | Log signals & trades to CSV |

### Telegram

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InpTelegram` | false | Telegram alerts |
| `InpTgBotToken` | "" | Bot token (from @BotFather) |
| `InpTgChatId` | "" | Chat ID (from @userinfobot) |

### Normalization

| Parameter | Description |
|-----------|-------------|
| `InpNormMeans` | 18 comma-separated means dari training set |
| `InpNormStds` | 18 comma-separated stds dari training set |

---

## Features

- **HUD**: On-chart display showing signal, confidence, P&L, DD
- **Telegram**: Notifikasi saat posisi dibuka/ditutup
- **Drawdown Kill**: Otomatis pause jika DD > threshold
- **Weekend Filter**: Close all sebelum weekend
- **Emergency Spread**: Close all jika spread spike
- **Time Filter**: Skip non-trading hours

## Log File

```
Location: MQL5/Common/Files/AI_Signal_Log_XAUUSD_YYYYMMDD.csv
Mode: Append (tidak overwrite)
Format: CSV

Columns: DateTime, BarNum, Signal, Confidence,
         P(SELL), P(BUY), Action, Spread, Price,
         SL, TP, Result
```

## ONNX Integration

- Model di-embed langsung di EA (resource `#resource`)
- Input: `[1, LOOKBACK, 18]` float tensor
- Output: `[1, 2]` float tensor → P(SELL), P(BUY)
- Tick bar building: 100 ticks → 1 bar → compute features → normalize → ONNX inference
