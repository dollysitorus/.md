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

### Contoh docker-compose project (Node.js)

```yaml
services:
  dev:
    image: node:20-alpine
    working_dir: /app
    volumes:
      - .:/app
      - dep_npm:/root/.npm
    networks:
      - infra-net

volumes:
  dep_npm:
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
| `.gitignore` | Wajib include `.env` dan `.agents/` |
| `.env.example` | Template credentials |
| `.env` | Actual credentials (gitignored) |
| `README.md` | Quick start + overview |
