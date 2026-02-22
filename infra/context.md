# Infra — Shared Infrastructure

## Stack

| Service    | Image                | Port(s)    | Purpose              |
|------------|----------------------|------------|----------------------|
| PostgreSQL | `postgres:16-alpine` | 5432       | Primary database     |
| pgAdmin    | `dpage/pgadmin4`     | 5050       | PostgreSQL GUI       |
| Redis      | `redis:7-alpine`     | 6379       | Cache / queue        |
| MinIO      | `minio/minio`        | 9000, 9001 | Object storage (S3)  |
| Mailpit    | `axllent/mailpit`    | 1025, 8025 | Local email testing  |
| Caddy      | `caddy:2-alpine`     | 80, 443    | Reverse proxy        |

## Akses URL (Local)

| Service       | URL                          |
|---------------|------------------------------|
| pgAdmin       | `http://pgadmin.localhost`   |
| MinIO Console | `http://minio.localhost`     |
| MinIO API     | `http://s3.localhost`        |
| Mailpit       | `http://mail.localhost`      |

## Konvensi

- Semua credentials di `.env` (gitignored), template di `.env.example`
- Network: `infra-net` (bridge) — project lain join via `external: true`
- Volumes: named volumes (`pg_data`, `redis_data`, `minio_data`, `mailpit_data`, dll)
- SMTP untuk project: `mailpit:1025` (dari dalam Docker network)

## Status

- [x] Docker Compose (6 services)
- [x] Caddy reverse proxy
- [x] Git + GitHub
