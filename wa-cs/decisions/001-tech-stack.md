# ADR 001: Tech Stack — Go + whatsmeow

## Status
Accepted (2026-02-22)

## Context
Butuh WhatsApp gateway yang ringan, cepat, dan bisa handle multiple nomor + multiple agent CS secara concurrent.

## Options Considered

| Option | Pros | Cons |
|--------|------|------|
| Go + whatsmeow | Ringan (~10-20MB/session), fast, protocol-level, production-proven (mautrix) | Ecosystem Go lebih kecil dari Node |
| Node.js + Baileys | Large community, familiar | Heavier runtime, callback hell |
| Node.js + Puppeteer | Easy to start | Very heavy (~300-500MB/session), fragile, UI scraping |

## Decision
**Go + whatsmeow** — paling ringan dan cepat. Protocol-level (bukan browser automation). Sudah production-proven di mautrix-whatsapp bridge.

### Supporting Libraries
- `chi` — lightweight HTTP router (lebih ringan dari gin, lebih idiomatic)
- `gorilla/websocket` — standard WebSocket library
- `sqlc` — type-safe SQL, generated code, no runtime reflection
- `minio-go` — official MinIO SDK

## Consequences
- Team harus familiar dengan Go
- No server-rendered UI — frontend harus SPA terpisah
