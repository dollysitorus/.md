# vdb-dashboard

## Overview

Web dashboard untuk monitoring dan management VDB EA (Expert Advisor) di MetaTrader 5.

## Stack

| Layer | Tech |
|-------|------|
| Frontend | Next.js 15 (App Router) |
| Styling | Tailwind CSS v4 |
| ORM | Drizzle |
| Auth | NextAuth.js (credentials) |
| Database | PostgreSQL (`vdb_dashboard`) |
| Charts | Recharts / Lightweight Charts |
| Dev Container | node:22-alpine |

## Architecture

```
EA (MT5) --[POST + License Key]--> API (/api/ingest/*) --> PostgreSQL
                                                              ↓
                                     User/Admin <-- Dashboard (Next.js)
```

## Key Concepts

| Konsep | Deskripsi |
|--------|-----------|
| License Key | `VDB-XXXX-XXXX-XXXX` — 1 key per EA instance |
| Admin | CRUD users, generate/revoke keys, lihat semua data |
| User | Lihat dashboard untuk key miliknya |
| Ingest API | EA POST data (bars, trades, positions, events) ke API |

## Port

- Dev: `3100` (mapped ke internal 3000)

## Konvensi

- Folder: kebab-case
- File: kebab-case
- Docker-only development
- Dual Repo Contract (project + `.md`)

## Status

| Item | Status |
|------|--------|
| Project setup | ✅ Done |
| DB schema | 🔄 Pending |
| Auth | 🔄 Pending |
| Key management | 🔄 Pending |
| Ingest API | 🔄 Pending |
| Dashboard UI | 🔄 Pending |
