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
| role | VARCHAR(20) | admin / supervisor / agent |
| is_active | BOOLEAN | default true |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

### whatsapp_account
Nomor WhatsApp yang terdaftar.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| phone_number | VARCHAR(20) | nomor WA (diisi setelah login) |
| display_name | VARCHAR(100) | nama tampilan |
| jid | VARCHAR(100) | WhatsApp JID dari whatsmeow |
| status | VARCHAR(20) | connected / disconnected / qr_pending |
| ai_enabled | BOOLEAN | default false, toggle AI per akun |
| openai_api_key | TEXT | API key OpenAI per akun |
| openai_model | VARCHAR(50) | default gpt-4o-mini |
| ai_system_prompt | TEXT | sistem prompt AI per akun |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

> **Note:** Session/encryption keys disimpan di tabel `whatsmeow_*` (managed otomatis oleh whatsmeow sqlstore).

### whatsapp_account_agent
Junction table: WA account ↔ agent assignment.

| Column | Type | Notes |
|--------|------|-------|
| whatsapp_account_id | UUID | FK → whatsapp_account |
| agent_id | UUID | FK → agent |

### contact
Kontak customer yang chat.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| jid | VARCHAR(100) | UNIQUE (phone-based) |
| phone_number | VARCHAR(20) | normalized |
| push_name | VARCHAR(100) | nama dari WA profile |
| custom_name | VARCHAR(100) | nama yang di-set agent |
| company | VARCHAR(100) | perusahaan |
| avatar_url | TEXT | URL avatar di MinIO |
| notes | TEXT | catatan agent |
| hidden | BOOLEAN | soft delete flag |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

### contact_group
Tag/label untuk grouping kontak.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| name | VARCHAR(50) | UNIQUE |
| color | VARCHAR(7) | hex color |
| created_at | TIMESTAMPTZ | |

### contact_group_member
Junction table: contact ↔ contact_group.

| Column | Type | Notes |
|--------|------|-------|
| contact_id | UUID | FK → contact |
| group_id | UUID | FK → contact_group |

### conversation
Thread percakapan.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| whatsapp_account_id | UUID | FK → whatsapp_account |
| contact_id | UUID | FK → contact (nullable for groups) |
| assigned_agent_id | UUID | FK → agent (nullable) |
| chat_type | VARCHAR(10) | private / group |
| group_jid | VARCHAR(100) | WA group JID |
| group_name | VARCHAR(200) | group name |
| status | VARCHAR(20) | open / in_progress / resolved / closed |
| unread_count | INT | default 0 |
| ai_escalated | BOOLEAN | default false |
| last_message_at | TIMESTAMPTZ | |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

### conversation_agent
Junction table: multi-agent assignment (groups).

| Column | Type | Notes |
|--------|------|-------|
| conversation_id | UUID | FK → conversation |
| agent_id | UUID | FK → agent |
| created_at | TIMESTAMPTZ | |

### group_participant
Group members with progressive phone mapping.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| conversation_id | UUID | FK → conversation |
| jid | VARCHAR(100) | |
| phone | VARCHAR(20) | |
| name | VARCHAR(100) | |
| is_admin | BOOLEAN | |

### message
Pesan individual dalam conversation.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| conversation_id | UUID | FK → conversation |
| sender_type | VARCHAR(10) | contact / agent |
| sender_id | UUID | FK → contact atau agent |
| sender_name | VARCHAR(100) | display name |
| wa_message_id | VARCHAR(100) | ID pesan dari WA |
| reply_to_wa_message_id | VARCHAR(100) | quote reply |
| content_type | VARCHAR(20) | text / image / video / audio / document / sticker / poll |
| content | TEXT | isi pesan |
| media_url | TEXT | URL media di MinIO |
| media_mime_type | VARCHAR(50) | |
| media_file_name | VARCHAR(255) | |
| wa_media_key | BYTEA | lazy download key |
| wa_direct_path | TEXT | |
| wa_file_sha256 | BYTEA | |
| wa_file_enc_sha256 | BYTEA | |
| wa_file_length | BIGINT | |
| wa_media_type | VARCHAR(20) | |
| status | VARCHAR(20) | sent / delivered / read / failed / received |
| created_at | TIMESTAMPTZ | |

### ai_knowledge
FAQ entries per WA account for AI auto-reply.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| account_id | UUID | FK → whatsapp_account |
| title | VARCHAR(200) | |
| content | TEXT | |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

### canned_response
Quick reply templates.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| title | VARCHAR(100) | |
| shortcut | VARCHAR(50) | |
| content | TEXT | |
| created_at | TIMESTAMPTZ | |

## Migrations

| # | File | Isi |
|---|------|-----|
| 001 | `001_init.up.sql` | Semua tabel dasar + indexes + seed admin |
| 002 | `002_fix_message_status.up.sql` | Tambah 'received' ke message status |
| 003 | `003_add_supervisor_role.up.sql` | Tambah 'supervisor' ke agent role |
| 004 | `004_high_priority_features.up.sql` | Conversation lifecycle, unread, contact notes, canned responses |
| 005 | `005_reply_support.up.sql` | Reply-to support, media_file_name |
| 006 | `006_group_support.up.sql` | Group chat (chat_type, group_jid, group_name) |
| 007 | `007_conversation_agent.up.sql` | Multi-agent assignment (conversation_agent table) |
| 008 | `008_group_participant.up.sql` | Group participant tracking |
| 009 | `009_contact_company.up.sql` | Contact company field |
| 010 | `010_whatsapp_account_agent.up.sql` | WA account ↔ agent assignment |
| 011 | `011_lazy_media.up.sql` | Lazy media download metadata columns |
| 012 | `012_poll_content_type.up.sql` | Poll content type support |
| 013 | `013_contact_group.up.sql` | Contact grouping (tags/labels) |
| 014 | `014_contact_soft_delete_dedup.up.sql` | Soft delete + phone dedup |
| 015 | `015_ai_config.up.sql` | Global AI config (app_setting + ai_knowledge) |
| 016 | `016_per_account_ai.up.sql` | Move AI config to per-WA account, drop app_setting |
