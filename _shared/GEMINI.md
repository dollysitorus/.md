# 🔒 Rule Global — Agentic Cascade AI for Coding (Production)

## 0. Status
Dokumen ini bersifat mengikat untuk semua agent (planner/coder/reviewer). Jika ada konflik: Rule Global menang.
---
## 1. Tujuan Utama (Non-Negotiable)
1) Correctness > speed.  
2) Minimal change, maximal impact.  
3) Single Source of Truth: tidak boleh duplikasi logika/aturan.  
4) Konsistensi (naming, struktur, error handling, style) adalah kewajiban.
---
## 2. Context Lock (Wajib)
1) Setiap task WAJIB menyebut `REPO`.  
2) Selama task berjalan, agent hanya boleh bekerja di repo aktif: `/app/<REPO>/...`  
3) Cross-repo dilarang kecuali `ALLOW_CROSS_REPO=true`. Jika false, agent hanya boleh membuat proposal (tanpa edit lintas repo).  
4) Jika `SCOPE` diberikan, agent hanya boleh menyentuh path yang diizinkan.
---
## 3. Mandatory Execution Flow (Cascade)
Agent wajib menjalankan urutan:
Understand → Plan → Design → Implement (small diff) → Verify → Refactor (opsional)
Larangan:
- Melompati Verify.
- Refactor besar ketika MODE=implement.
---
## 4. Anti-Boros (No Overengineering)
1) Dilarang menambah layer/pattern/dependency tanpa kebutuhan eksplisit.  
2) Dilarang menambah fitur “sekalian”.  
3) Default > konfigurasi berlebihan.  
4) Hindari public API baru kecuali dibutuhkan.
---
## 4b. Docker-Only Development (Non-Negotiable)
1) Dilarang install runtime (Node, Go, Python, PHP, dll) di mesin lokal.  
2) Semua development wajib pakai dev container di Docker.  
3) Dependencies di-cache via shared volumes (`dep_npm`, `dep_go`, `dep_pip`, `dep_composer`).  
4) Setiap project wajib punya `docker-compose.yml` dengan service `dev`.
---
## 5. Anti-Tumpang Tindih (No Overlap)
1) Satu konsep hanya punya satu implementasi.  
2) Tidak boleh ada dua fungsi dengan tanggung jawab identik.  
3) Utility harus domain-agnostic dan kecil; util domain masuk modulnya sendiri.  
4) DRY hanya jika logika identik dan berubah bersama.
---
## 6. Naming & Terminology Law
1) Wajib mengikuti glossary domain (satu istilah = satu nama).  
2) Sinonim untuk konsep sama dilarang (mis. customer vs client).  
3) Nama harus mencerminkan intent, bukan implementasi.  
4) Konvensi penamaan harus konsisten:
   - Function/variable: camelCase (atau standar bahasa proyek)
   - Class/type: PascalCase
   - Constant: SCREAMING_SNAKE_CASE
   - Folder: kebab-case
   - File: pilih 1 gaya konsisten (kebab-case atau snake_case)
5) Nama generik `data`, `helper`, `manager`, `misc` dilarang kecuali didefinisikan jelas.
---
## 7. Struktur & Dependency
1) Domain logic tidak boleh bergantung pada framework/ORM/transport.  
2) Dependency harus satu arah (no cycle).  
3) Boundary (controller/handler) bertugas validasi input + mapping error + calling use-case.
---
## 8. Error Handling & Logging
1) Error wajib terklasifikasi: ValidationError / NotFoundError / ConflictError / AuthError / ExternalServiceError (atau setara).  
2) Pisahkan pesan untuk user vs developer.  
3) Logging dilarang memuat secret/credential/token/PII tanpa masking.  
4) Fail fast di boundary.
---
## 9. Verification & Testing (Wajib)
1) Setiap perubahan wajib bisa diverifikasi (test/command/contoh run).  
2) Minimal: happy path + edge case kritikal.  
3) Jangan test library eksternal; test integrasi dan logika kita.
---
## 10. Change Discipline
1) Dilarang mengubah kode yang tidak relevan dengan GOAL.  
2) Dilarang meninggalkan dead code, unused import, atau komentar usang.  
3) Jika tradeoff diperlukan, pilih: correctness, simplicity, consistency, scope.
---
## 11. Output Standard (Wajib)
Setiap jawaban implementasi wajib memuat:
- Summary perubahan
- File changed/added
- Verification steps
- Catatan tradeoff (jika ada)

# 🔒 Rule Global — Dual Repo Strict (Project + Context)

## 1) Dual-Repo Contract
1. Semua task coding SELALU menghasilkan 2 jejak perubahan:
   - Repo Project: implementasi kode
   - Repo Context: knowledge (glossary/decisions/standards/workflow map)
2. Setiap perubahan penting (naming baru, modul baru, boundary baru, keputusan arsitektur) WAJIB dicatat di repo Context.
3. Saat user meminta "push git", WAJIB push kedua repo:
   - push repo project
   - push repo context
4. Jika salah satu repo gagal push, proses dianggap gagal (stop, jelaskan error, jangan klaim sukses).

## 2) Context Lock
1. Task wajib menyebut REPO_TARGET (crm/erp/infra).
2. Agent hanya boleh mengubah:
   - `/Volumes/Data/app/<REPO_TARGET>/**`
   - `/Volumes/Data/app/.md/**`
3. Cross-project dilarang tanpa izin eksplisit.

## 3) Single Source of Truth
1. Glossary domain ada di `.md/_shared/glossary.md` (bukan tersebar di project).
2. Standar global ada di `.md/_shared/standards.md`.
3. GEMINI.md source of truth ada di `.md/_shared/GEMINI.md` — copy ke `~/.gemini/GEMINI.md` di setiap mesin.
4. Workflow source of truth ada di `.md/_shared/workflows/`.
5. Workflow discovery: agent WAJIB membaca workflows langsung dari `.md/_shared/workflows/`. Tidak boleh ada folder agent (`.agents`, `.agent`, dll) di workspace root. Saat inisiasi conversation baru, agent harus memuat rules dari `.md/_shared/GEMINI.md` dan workflow dari `.md/_shared/workflows/`.

## 4) Minimal Change & No Overlap
- Jangan duplikasi logika
- Jangan overengineering
- Small diff only (kecuali MODE=refactor)

## 5) Verification Wajib
Semua perubahan harus punya langkah verifikasi yang jelas.

## 6) Output Standard
Setiap respons implementasi wajib:
- Summary perubahan (project + context)
- Files changed (project + context)
- Verification steps