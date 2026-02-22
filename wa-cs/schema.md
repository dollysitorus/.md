# wa-cs — Database Schema

Database: `wa_cs` (PostgreSQL)

## Tables

### agent
User/operator yang membalas chat.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| email | VARCHAR(255) | UNIQUE, untuk login |
| password_hash | VARCHAR(255) | bcrypt |
| name | VARCHAR(100) | display name |
| role | VARCHAR(20) | admin / agent |
| is_active | BOOLEAN | default true |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

### whatsapp_account
Nomor WhatsApp yang terdaftar.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| phone_number | VARCHAR(20) | nomor WA (diisi setelah login) |
| display_name | VARCHAR(100) | nama tampilan (input user saat add account) |
| jid | VARCHAR(100) | WhatsApp JID dari whatsmeow (diisi setelah login) |
| status | VARCHAR(20) | connected / disconnected / qr_pending |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

> **Note:** Session/encryption keys disimpan di tabel `whatsmeow_*` (16 tabel, managed otomatis oleh whatsmeow sqlstore).

### contact
Kontak customer yang chat.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| jid | VARCHAR(100) | UNIQUE, WhatsApp JID |
| phone_number | VARCHAR(20) | |
| push_name | VARCHAR(100) | nama dari WA profile |
| custom_name | VARCHAR(100) | nama yang di-set agent |
| avatar_url | TEXT | URL avatar di MinIO |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

### conversation
Satu thread percakapan antara contact + whatsapp_account.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| whatsapp_account_id | UUID | FK → whatsapp_account |
| contact_id | UUID | FK → contact |
| assigned_agent_id | UUID | FK → agent (nullable) |
| status | VARCHAR(20) | open / resolved / pending |
| last_message_at | TIMESTAMPTZ | untuk sorting |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

### message
Pesan individual dalam conversation.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| conversation_id | UUID | FK → conversation |
| sender_type | VARCHAR(10) | contact / agent |
| sender_id | UUID | FK → contact atau agent |
| wa_message_id | VARCHAR(100) | ID pesan dari WA |
| content_type | VARCHAR(20) | text / image / video / audio / document |
| content | TEXT | isi pesan (text) |
| media_url | TEXT | URL media di MinIO (nullable) |
| media_mime_type | VARCHAR(50) | MIME type (nullable) |
| status | VARCHAR(20) | sent / delivered / read / failed / received |
| created_at | TIMESTAMPTZ | |

## Indexes

- `conversation(whatsapp_account_id, contact_id)` — UNIQUE
- `conversation(assigned_agent_id)` — untuk filter by agent
- `conversation(status, last_message_at DESC)` — untuk inbox sorting
- `message(conversation_id, created_at)` — untuk chat history
- `message(wa_message_id)` — untuk dedup
- `contact(jid)` — UNIQUE

## Migrations

| # | File | Isi |
|---|------|-----|
| 001 | `001_init.up.sql` | Semua tabel + indexes + seed admin |
| 002 | `002_fix_message_status.up.sql` | Tambah 'received' ke message status constraint |
