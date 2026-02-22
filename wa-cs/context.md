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
| Query Builder | sqlc |
| Object Storage | MinIO (shared infra) |
| Cache | Redis (shared infra) |
| Auth | JWT |

## Architecture

```
Browser (Dashboard) ←→ REST API (8080) + WebSocket (8081)
                              ↓
                       Business Logic
                     ↙     ↓      ↘
               PostgreSQL  MinIO  whatsmeow (per nomor WA)
```

## Konvensi

- Folder: `internal/` untuk semua business logic (unexported)
- `cmd/server/` untuk entrypoint
- `migration/` untuk SQL migration files
- DB name: `wa_cs`
- Container name: `wa-cs-dev`

## Status

- [x] Project scaffold
- [ ] Implementasi whatsmeow connection
- [ ] REST API endpoints
- [ ] WebSocket hub
- [ ] Database schema + migration
- [ ] Auth (JWT)
- [ ] MinIO integration
- [ ] Dashboard frontend
