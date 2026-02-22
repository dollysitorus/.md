# ADR-001: Infrastructure Stack Selection

## Status
Accepted

## Context
Butuh shared infrastructure yang dipakai semua project (CRM, ERP, dll). Kebutuhan: database, cache, object storage, reverse proxy, dan database GUI.

## Decision

| Concern          | Choice           | Alasan                                          |
|------------------|------------------|-------------------------------------------------|
| Database         | PostgreSQL 16    | Mature, fitur lengkap, ecosystem besar          |
| Cache/Queue      | Redis 7          | Cepat, sudah proven, support pub/sub            |
| Object Storage   | MinIO             | S3-compatible, self-hosted, ringan              |
| Reverse Proxy    | Caddy 2          | Auto HTTPS, config simple, performance baik     |
| Database GUI     | pgAdmin 4        | Official PostgreSQL tool, fitur lengkap         |
| Orchestration    | Docker Compose   | Simple, cukup untuk development & small deploy  |

## Consequences
- Semua project connect ke infra yang sama via Docker network `infra-net`
- Credentials terpusat di `infra/.env`
- Project baru cukup join network, tidak perlu setup DB/cache sendiri
