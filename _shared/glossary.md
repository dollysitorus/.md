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

## WA-CS (WhatsApp Customer Service)

| Istilah | Definisi | ❌ Jangan pakai |
|---------|----------|-----------------|
| agent | User/operator yang membalas chat customer | operator, admin, cs, staff |
| conversation | Thread percakapan antara contact + whatsapp account | chat, thread, room |
| message | Pesan individual dalam conversation | chat message, msg |
| contact | Customer yang chat via WhatsApp | customer, user, client |
| whatsapp account | Nomor WhatsApp terdaftar di sistem | wa number, phone, device |
| whatsmeow | Library Go untuk WhatsApp Web protocol | wa library, wa sdk |

## Field Sales

| Istilah | Definisi | ❌ Jangan pakai |
|---------|----------|-----------------|
| salesperson | User sales yang bertugas di lapangan | sales rep, field agent, sales agent |
| customer | Entitas bisnis/individu yang dilayani | client, account, pelanggan |
| visit | Kunjungan salesperson ke lokasi customer | appointment, meeting, call |
| task | Tugas yang diberikan ke salesperson | assignment, todo, job |
| check-in | GPS timestamp saat tiba di lokasi customer | clock-in, arrive |
| check-out | GPS timestamp saat selesai visit | clock-out, leave |

## MQL-AI (Trading AI)

| Istilah | Definisi | ❌ Jangan pakai |
|---------|----------|-----------------|
| ea | Expert Advisor — automated trading program di MT5 | bot, robot, algo |
| onnx | Open Neural Network Exchange format | model file, ai file |
| indicator | Technical indicator / custom indicator MT5 | signal, metric |
| backtest | Pengujian strategi pada data historis | simulation, test run |

## VDB Dashboard

| Istilah | Definisi | ❌ Jangan pakai |
|---------|----------|-----------------| 
| license key | Key unik per EA instance (`VDB-XXXX-XXXX-XXXX`) | password, token, api key |
| ingest | Proses EA mengirim data ke dashboard API | upload, sync, push |
| bar | VDB bar — event-driven bar dari tick data | candle, candlestick |


