# Standards

Coding conventions dan patterns yang berlaku untuk semua project.

## Docker-Only Development (Non-Negotiable)

**Dilarang install runtime/tools di mesin lokal.** Semua jalan di Docker.

| Aturan | Detail |
|--------|--------|
| No local runtime | Node, Go, Python, PHP, dll hanya via Docker image |
| Dev container | Setiap project punya service `dev` di docker-compose |
| Shared dep cache | Mount volume `dep_*` dari infra untuk hemat storage |
| Editor tetap lokal | VS Code / IDE jalan di host, kode di bind mount |

### Shared Dependency Cache Volumes

Dibuat di infra, mount sebagai external di setiap project:

| Volume | Mount target di container | Tech |
|--------|--------------------------|------|
| `dep_npm` | `/root/.npm` | Node.js |
| `dep_go` | `/go/pkg/mod` | Go |
| `dep_pip` | `/root/.cache/pip` | Python |
| `dep_composer` | `/root/.composer/cache` | PHP |

### Per-Project node_modules Volume (Non-Negotiable)

`node_modules` **DILARANG ada di host filesystem**. Gunakan named volume per-project:

| Volume | Mount target | Scope |
|--------|-------------|-------|
| `dep_node_<project>` | `/app/web/node_modules` (atau sesuai project) | Per-project (tidak bisa shared) |

**Kenapa per-project?** `node_modules` berisi resolved dependency tree spesifik per `package.json`. Beda project = beda tree. Kalau di-share akan bentrok. Berbeda dengan `dep_npm` yang hanya cache tarball download (aman shared).

**Naming:** `dep_node_wa_cs`, `dep_node_crm`, dst. Wajib didaftarkan di `infra/docker-compose.yml`.

### Contoh docker-compose project (Node.js)

```yaml
services:
  node:
    image: node:22-alpine
    working_dir: /app/web
    volumes:
      - .:/app
      - dep_npm:/root/.npm
      - dep_node_myproject:/app/web/node_modules
    profiles:
      - build

volumes:
  dep_npm:
    external: true
  dep_node_myproject:
    external: true

networks:
  infra-net:
    external: true
```

## Naming

| Scope | Convention | Contoh |
|-------|-----------|--------|
| Function/variable | camelCase | `getUserById` |
| Class/type | PascalCase | `UserService` |
| Constant | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Folder | kebab-case | `user-management` |
| File | kebab-case | `user-service.ts` |
| Env variable | SCREAMING_SNAKE_CASE | `POSTGRES_PASSWORD` |
| Docker container | `<project>-<service>` | `crm-dev` |

## Git

| Aspek | Aturan |
|-------|--------|
| Branch utama | `main` |
| Commit message | Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`) |
| Dual Repo | Setiap perubahan → commit di project repo + `.md` context repo |

## Error Handling

| Tipe Error | Kapan Pakai |
|------------|-------------|
| ValidationError | Input tidak valid |
| NotFoundError | Resource tidak ditemukan |
| ConflictError | Duplikasi/conflict state |
| AuthError | Authentication/authorization gagal |
| ExternalServiceError | Service luar gagal |

## File Wajib di Setiap Project

| File | Keterangan |
|------|-----------|
| `docker-compose.yml` | Dev container + shared dep cache |
| `.gitignore` | Wajib include `.env` |
| `.env.example` | Template credentials |
| `.env` | Actual credentials (gitignored) |
| `README.md` | Quick start + overview |
