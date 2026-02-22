# ADR-001: Tech Stack — Go + PWA

## Status
Accepted

## Context
Butuh field sales management app yang:
- Mobile-friendly untuk sales di lapangan (GPS, offline-capable)
- Desktop dashboard untuk admin/manager
- Consistent dengan existing tech stack (wa-cs pakai Go)

## Decision
- **Backend**: Go (net/http) — konsisten, performant, single binary
- **Frontend**: Vanilla HTML/CSS/JS sebagai PWA — satu codebase untuk mobile + desktop
- **Database**: PostgreSQL (shared infra) — sudah ada
- **Storage**: MinIO — untuk foto visit, dokumen

## Consequences
- PWA bisa di-install di homescreen HP sales → UX mirip native app
- Responsive design: mobile-first untuk sales, layout berubah desktop untuk admin
- Tidak perlu maintain 2 codebase (mobile + web)
- GPS via browser Geolocation API (cukup untuk check-in/check-out)
