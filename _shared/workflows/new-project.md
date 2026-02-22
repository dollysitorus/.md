---
description: how to start a new project with git repo, infra integration, and context docs
---

# /new-project Workflow

## Struktur Folder

```
/Volumes/Data/app/
├── .md/            # repo: dollysitorus/.md — shared context
│   ├── _shared/    #   glossary.md, standards.md (cross-project)
│   ├── infra/      #   context untuk project infra
│   └── <nama>/     #   context untuk project <nama>
├── infra/          # repo: dollysitorus/infra
└── <nama>/         # repo: dollysitorus/<nama>
```

### Context Repo (`.md/`) — Struktur Per Project

```
.md/<nama>/
├── context.md              # stack, arsitektur, konvensi, status (wajib)
├── decisions/              # ADR keputusan teknis (opsional)
│   └── 001-xxx.md
├── schema.md               # DB tables, models (opsional, jika pakai DB)
└── api.md                  # endpoint contracts (opsional, jika ada API)
```

### Shared Context

```
.md/_shared/
├── glossary.md             # domain terms — 1 istilah = 1 nama
└── standards.md            # naming, git, error handling, docker conventions
```

## Aturan Penamaan

1. **Nama repo = nama folder** (contoh: folder `crm/` → repo `dollysitorus/crm`)
2. **Format**: lowercase, kebab-case untuk multi-word (contoh: `inventory-api`)
3. **Visibility**: private (default)

## Steps

### 1. Tanya detail project
- Nama project (= nama folder = nama repo)
- Tech stack (Go / Node / Python / etc)
- Port yang dipakai
- Perlu database baru di PostgreSQL? (nama DB)

### 2. Pastikan `.md` repo sudah ada
```bash
# Jika belum di-clone:
gh repo clone dollysitorus/.md /Volumes/Data/app/.md
```

### 3. Baca context sebelum mulai
```bash
# Baca glossary dan standards supaya konsisten
cat /Volumes/Data/app/.md/_shared/glossary.md
cat /Volumes/Data/app/.md/_shared/standards.md
```

### 4. Buat folder project + git init
```bash
mkdir -p /Volumes/Data/app/<nama>
cd /Volumes/Data/app/<nama>
git init
```

### 5. Buat file dasar
- `docker-compose.yml` — dev container + shared dep cache (lihat di bawah)
- `.gitignore` — sesuai tech stack, **wajib include `.env`**
- `.env.example` — template credentials
- `.env` — actual local credentials (gitignored)
- `README.md` — project overview

### 6. Buat `docker-compose.yml` dengan dev container

Wajib punya service `dev` yang mount shared dep cache:

```yaml
# Contoh Node.js
services:
  dev:
    image: node:20-alpine
    container_name: <nama>-dev
    working_dir: /app
    volumes:
      - .:/app
      - dep_npm:/root/.npm
    networks:
      - infra-net
    # command: npm run dev

volumes:
  dep_npm:
    external: true

networks:
  infra-net:
    external: true
```

Pilih image + volume sesuai tech stack:

| Tech | Image | Dep volume | Mount target |
|------|-------|-----------|--------------|
| Node.js | `node:20-alpine` | `dep_npm` | `/root/.npm` |
| Go | `golang:1.22-alpine` | `dep_go` | `/go/pkg/mod` |
| Python | `python:3.12-alpine` | `dep_pip` | `/root/.cache/pip` |
| PHP | `php:8.3-alpine` | `dep_composer` | `/root/.composer/cache` |

### 7. Scaffold struktur sesuai tech stack
Buat folder structure awal sesuai stack yang dipilih.

### 8. Initial commit
```bash
git add -A
git commit -m "chore: initial <nama> setup"
```

### 9. Buat GitHub repo + push
```bash
gh repo create <nama> --private --source . --remote origin --push
```

### 10. Update infra (opsional)
Jika project butuh:
- **Database baru**: tambah init script di infra
- **Reverse proxy**: tambah route di `Caddyfile`
- **Network**: project join `infra-net` sebagai external network

### 10. Buat context di repo `.md`
```bash
mkdir -p /Volumes/Data/app/.md/<nama>/decisions

# Wajib: context.md — stack, arsitektur, konvensi, status
# Opsional: decisions/001-xxx.md — ADR
# Opsional: schema.md — jika pakai DB
# Opsional: api.md — jika ada API

# Update glossary jika ada istilah baru
# Update standards jika ada pattern baru

cd /Volumes/Data/app/.md
git add -A
git commit -m "docs: add <nama> project context"
git push origin main
```

## Catatan Penting

- Tidak boleh ada folder agent (`.agents/`, `.agent/`, dll) di project
- Push WAJIB 2 repo: project + `.md` (Dual Repo Contract)
- Jika salah satu gagal push → stop, jelaskan error
- Baca `_shared/glossary.md` sebelum naming apapun
