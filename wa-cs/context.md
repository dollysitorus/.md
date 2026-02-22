# wa-cs — Context

## Overview

WhatsApp Chat Customer Service. Multi-agent, multi-number WhatsApp gateway untuk handle chat customer.

## Stack

| Layer | Tech |
|-------|------|
| Language | Go 1.22 |
| WA Gateway | whatsmeow (multi-device protocol) |
| HTTP Router | chi |
| WebSocket | gorilla/websocket |
| Database | PostgreSQL 16 (shared infra) |
| Object Storage | MinIO (shared infra) |
| Auth | JWT (bcrypt password) |

## Architecture

```
Browser (Dashboard) ←→ REST API + WebSocket (:8080)
                              ↓
                       Business Logic
                     ↙     ↓      ↘
               PostgreSQL  MinIO  whatsmeow (per nomor WA)
```

### Message Pipeline

```
Incoming:  WA → whatsmeow event → handleEvent → callback
           → upsert Contact → FindOrCreate Conversation
           → Create Message → UpdateLastMessage → Hub.Broadcast("new_message")

Outgoing:  Dashboard → POST /conversations/{id}/messages
           → Save Message → Manager.SendMessage (async, context.Background)
           → Hub.Broadcast("new_message")
```

### Key Pattern: context.Background() for Async

Semua operasi async yang berjalan setelah HTTP response (QR channel, SendMessage)
**wajib** pakai `context.Background()`, bukan `r.Context()`. Request context
otomatis cancelled begitu response dikirim.

## Konvensi

- Folder: `internal/` untuk semua business logic (unexported)
- `cmd/server/` untuk entrypoint
- `internal/config/migrations/` untuk SQL migration files
- DB name: `wa_cs`
- Container name: `wa-cs-dev`
- `sender_type`: `contact` (incoming WA) / `agent` (outgoing reply)

## Packages

| Package | Tanggung jawab |
|---------|---------------|
| `internal/config` | Config loader + migration runner (embed.FS) |
| `internal/handler` | HTTP handlers (auth, conversation, whatsapp, ws) |
| `internal/middleware` | JWT auth + role middleware |
| `internal/repository` | DB access (agent, contact, conversation, message, whatsapp_account) |
| `internal/service` | Business logic (auth) |
| `internal/storage` | MinIO client |
| `internal/websocket` | WebSocket hub + client |
| `internal/whatsapp` | WhatsApp manager (multi-device, callbacks) |

## Status

- [x] Project scaffold
- [x] Implementasi whatsmeow connection
- [x] REST API endpoints
- [x] WebSocket hub + realtime broadcast
- [x] Database schema + migrations (001 init, 002 fix status)
- [x] Auth (JWT)
- [x] MinIO integration
- [x] Dashboard frontend
- [x] QR code scan + device linking
- [x] Incoming message pipeline (WA → DB → WebSocket)
- [x] Outgoing message pipeline (Dashboard → WA)
- [x] WA account persistence (whatsapp_account table)
- [x] Auto-reconnect on restart (JIDLookup)
- [x] Group message / broadcast filtering
- [x] WA account delete + disconnect/reconnect
- [x] Media message handling (image/video/audio/document)
