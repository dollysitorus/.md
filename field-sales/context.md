# Field Sales — Context

## Overview
Field service management untuk sales team. Admin dashboard (desktop) + PWA mobile untuk sales di lapangan.

## Tech Stack
| Component | Technology |
|-----------|-----------|
| Backend | Go 1.22 (net/http, pgx, embed) |
| Frontend | Vanilla HTML/CSS/JS (PWA) |
| Database | PostgreSQL 16 (shared infra, DB: `field_sales`) |
| Storage | MinIO (shared infra) |
| Container | `golang:1.22-alpine` |
| Port | 8082 |

## Architecture
- Monolith: single Go binary serves API + embedded static files
- PWA: installable web app, mobile-first untuk sales, responsive desktop untuk admin
- Roles: admin, manager, salesperson

## Conventions
- Sama dengan global standards (lihat `_shared/standards.md`)
- Naming: camelCase (Go public = PascalCase)
- Folder: kebab-case

## Modules
- **Users**: CRUD salesperson/manager/admin, auth (JWT)
- **Customers**: lead/prospect/customer management, geolocation
- **Visits**: GPS check-in/check-out, notes, photos
- **Tasks**: assignment, scheduling, priority, status tracking
- **Dashboard**: reporting, analytics

## Status
- [x] Project scaffold (docker-compose, main.go, web)
- [x] Database schema (users, customers, visits, tasks)
- [x] GitHub repo created
- [ ] Auth (login, JWT, role-based)
- [ ] Users CRUD API
- [ ] Customers CRUD API
- [ ] Visits API (check-in/check-out with GPS)
- [ ] Tasks API
- [ ] Admin dashboard UI
- [ ] Mobile sales UI (PWA)
- [ ] Dashboard & reporting
