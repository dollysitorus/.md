# .md — Knowledge Base

Knowledge base bersama untuk semua project **dollysitorus**.

## Struktur

```
.md/
├── _shared/                  # global, cross-project
│   ├── glossary.md           # domain terms (1 istilah = 1 nama)
│   └── standards.md          # coding conventions, patterns
├── <project>/                # 1 folder per project
│   ├── context.md            # stack, arsitektur, konvensi, status
│   ├── decisions/            # ADR keputusan teknis
│   ├── schema.md             # DB tables, models (opsional)
│   └── api.md                # endpoint contracts (opsional)
└── README.md
```

## Konvensi

- **1 folder = 1 project** (nama folder = nama repo)
- `_shared/` — aturan global yang berlaku lintas project
- `context.md` — wajib, ringkasan project
- `decisions/` — opsional, ADR jika ada keputusan teknis
- `schema.md` — opsional, struktur database/models
- `api.md` — opsional, kontrak endpoint
