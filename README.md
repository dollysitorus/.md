# .md — Knowledge Base

Repo ini adalah knowledge base bersama untuk menyimpan konteks semua project **dollysitorus**.

## Struktur

```
.md/
├── <project>/              # 1 folder per project
│   ├── context.md          # stack, arsitektur, konvensi, status
│   └── decisions/          # ADR-style keputusan teknis
│       └── 001-xxx.md
└── README.md
```

## Konvensi

- **1 folder = 1 project** (nama folder = nama project/repo)
- `context.md` — wajib ada, berisi ringkasan project
- `decisions/` — opsional, berisi ADR jika ada keputusan teknis

## Cara Pakai

### Context (`<project>/context.md`)
- Tech stack & dependencies
- Arsitektur & folder structure
- Konvensi kode & naming
- Status & progress

### Decisions (`<project>/decisions/xxx.md`)
Format ADR:
- **Status** — Accepted / Superseded / Deprecated
- **Context** — Masalah apa yang dihadapi
- **Decision** — Apa yang dipilih (tabel perbandingan)
- **Consequences** — Dampak dari keputusan
