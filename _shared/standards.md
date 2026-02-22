# Standards

Coding conventions dan patterns yang berlaku untuk semua project.

## Naming

| Scope | Convention | Contoh |
|-------|-----------|--------|
| Function/variable | camelCase | `getUserById` |
| Class/type | PascalCase | `UserService` |
| Constant | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Folder | kebab-case | `user-management` |
| File | kebab-case | `user-service.ts` |
| Env variable | SCREAMING_SNAKE_CASE | `POSTGRES_PASSWORD` |
| Docker container | `infra-<service>` | `infra-postgres` |

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
| `.gitignore` | Wajib include `.env` dan `.agents/` |
| `.env.example` | Template credentials |
| `.env` | Actual credentials (gitignored) |
| `README.md` | Quick start + overview |

## Docker

| Aspek | Aturan |
|-------|--------|
| Network | Join `infra-net` sebagai external |
| Naming | Container: `<project>-<service>` |
| Health check | Wajib untuk services kritis |
| Volumes | Named volumes, bukan bind mount (kecuali config) |
