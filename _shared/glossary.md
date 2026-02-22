# Glossary

Satu istilah = satu nama. Sinonim dilarang.

## Infrastructure

| Istilah | Definisi | ❌ Jangan pakai |
|---------|----------|-----------------|
| infra | Shared infrastructure (Docker services) | infrastructure, server |
| service | Satu container dalam docker-compose | container, instance |
| infra-net | Docker network bersama | network, bridge |
| dev container | Container untuk development (bukan lokal) | local env, dev machine |
| dep cache | Shared volume untuk dependency packages | node_modules, vendor |

## Services

| Istilah | Definisi | ❌ Jangan pakai |
|---------|----------|-----------------|
| postgres | PostgreSQL database server | postgresql, pg, db-server |
| pgadmin | PostgreSQL GUI (pgAdmin 4) | db-gui, admin-panel |
| redis | Redis cache/queue server | cache-server |
| minio | MinIO object storage (S3-compatible) | s3, object-store, storage |
| mailpit | Local email testing (SMTP + Web UI) | mailtrap, mail-server, smtp |
| caddy | Reverse proxy (Caddy 2) | nginx, proxy |

## General

| Istilah | Definisi | ❌ Jangan pakai |
|---------|----------|-----------------|
| project | Satu unit kerja dengan repo sendiri | app, application, module |
| context | Knowledge/documentation tentang project | docs, documentation |
| ADR | Architecture Decision Record | decision log, tech decision |
