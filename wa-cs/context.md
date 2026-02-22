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
| `internal/repository` | DB access (agent, contact, conversation, message, whatsapp_account, canned_response, group_participant) |
| `internal/service` | Business logic (auth) |
| `internal/storage` | MinIO client |
| `internal/websocket` | WebSocket hub + client |
| `internal/whatsapp` | WhatsApp manager (multi-device, callbacks, contact sync) |

## Status

- [x] Project scaffold
- [x] Implementasi whatsmeow connection
- [x] REST API endpoints
- [x] WebSocket hub + realtime broadcast
- [x] Database schema + migrations (001 init, 002 fix status, 003 supervisor role, 004 high priority features, 008 group_participant)
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
- [x] Supervisor role (migration 003, role hierarchy admin > supervisor > agent)
- [x] Assign agent UI (supervisor + admin can assign agent to conversation)
- [x] Chat window scroll fix (flex min-height chain + nav logo contrast)
- [x] Conversation status lifecycle (open → in_progress → resolved → closed, migration 004)
- [x] Unread count + badge (per conversation, reset on open)
- [x] Contact profile (editable custom_name + notes)
- [x] Search conversations (by contact name/phone, debounce 300ms)
- [x] Canned responses / quick reply (shared global, CRUD + picker)
- [x] SVG icon migration (all emoji → inline SVG: search, attach, send, info, close, empty states)
- [x] Detail panel polish (stacked form layout, styled inputs/selects, action button)
- [x] Contacts page (list all contacts, card grid, GET /api/contacts)
- [x] Quick replies management page (CRUD UI, add/edit modal, delete confirm)
- [x] Group chat support (separate /groups tab, group conversations)
- [x] Auto-sync joined groups on WA connect
- [x] Multi-agent assignment for group conversations (add/remove agents)
- [x] Group participant tracking (group_participant table, progressive phone mapping from messages)
- [x] Sender name in group chat bubbles (agent=accent, contact=orange, from contact table)
- [x] Contact sync on connect (272+ contacts from WA store, phone-based JIDs only)
- [x] LID-to-phone resolution (resolve LID contacts to real phone via push_name match)
- [x] Group detail panel (no Contact section, no Status/Assigned Agent — only Group Agents + Group Members)
- [x] Delivery & read status tracking (sent ✓, delivered ✓✓, read ✓✓ blue — events.Receipt → DB → WebSocket → UI)

## LID JID Handling

WhatsApp now uses LID (Linked Identity) JIDs in group messages instead of phone-based JIDs.

**Strategy:**
1. On connect → sync all contacts from WA store (phone-based JIDs)
2. On group message → contact created with LID JID (push_name available)
3. Group Members query → LEFT JOIN by push_name to resolve LID → real phone
4. If no phone match → display name only (no phone number shown)
5. Auto-resolve: when contact later chats privately or saved on phone → next sync fills phone

