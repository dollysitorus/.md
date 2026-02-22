# .md — Knowledge Base

Repo ini adalah knowledge base bersama untuk menyimpan konteks semua project **dollysitorus**.

## Struktur

```
├── projects/      # Konteks per project (stack, arsitektur, konvensi, status)
├── decisions/     # Catatan keputusan teknis (ADR-style)
├── snippets/      # Code snippets & templates reusable
└── references/    # Link, cheatsheet, catatan referensi
```

## Cara Pakai

### Projects
Buat satu file `.md` per project, isi dengan:
- Tech stack & dependencies
- Arsitektur & folder structure
- Konvensi kode & naming
- Status & progress

**Contoh**: `projects/my-app.md`

### Decisions
Catat keputusan teknis penting dengan format:
- **Konteks** — Masalah apa yang dihadapi
- **Keputusan** — Apa yang dipilih
- **Alasan** — Kenapa pilih itu

**Contoh**: `decisions/001-pilih-postgresql.md`

### Snippets
Simpan potongan kode yang sering dipakai ulang.

**Contoh**: `snippets/docker-compose-template.md`

### References
Simpan link, cheatsheet, atau catatan referensi.

**Contoh**: `references/git-cheatsheet.md`
