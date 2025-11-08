# Dokumentasi Lengkap Codebase - DewaKoding Project Management

## 📚 Daftar Isi
1. [Pengenalan](#pengenalan)
2. [Teknologi yang Digunakan](#teknologi-yang-digunakan)
3. [Struktur Project](#struktur-project)
4. [Model Database dan Relasi](#model-database-dan-relasi)
5. [Penjelasan Fitur Utama](#penjelasan-fitur-utama)
6. [Struktur File dan Folder](#struktur-file-dan-folder)
7. [Alur Kerja Aplikasi](#alur-kerja-aplikasi)
8. [Penjelasan Kode Penting](#penjelasan-kode-penting)

---

## 🎯 Pengenalan

**DewaKoding Project Management** adalah aplikasi manajemen proyek yang dibangun dengan Laravel dan Filament. Aplikasi ini memungkinkan tim untuk mengelola proyek, tiket (tugas), anggota tim, dan melacak kemajuan pekerjaan.

### Apa yang Dilakukan Aplikasi Ini?
- **Manajemen Proyek**: Membuat dan mengelola proyek dengan berbagai pengaturan
- **Manajemen Tiket**: Membuat, menugaskan, dan melacak tiket pekerjaan
- **Manajemen Tim**: Menambahkan anggota tim ke proyek dan mengatur peran mereka
- **Pelacakan Progress**: Melihat kemajuan proyek melalui berbagai tampilan (Board, Timeline, dll)
- **Notifikasi**: Mengirim notifikasi ketika ada perubahan pada proyek atau tiket
- **Portal Klien**: Memberikan akses eksternal untuk klien melihat progress proyek

---

## 💻 Teknologi yang Digunakan

### Backend
- **Laravel 12**: Framework PHP untuk pengembangan aplikasi web
- **PHP 8.2+**: Bahasa pemrograman yang digunakan
- **MySQL/MariaDB**: Database untuk menyimpan data

### Frontend & Admin Panel
- **Filament 4**: Admin panel berbasis Laravel untuk mengelola data
- **Livewire**: Untuk membuat komponen interaktif tanpa JavaScript kompleks
- **Tailwind CSS**: Framework CSS untuk styling
- **Vite**: Build tool untuk aset frontend

### Package Tambahan
- **Filament Shield**: Untuk manajemen role dan permission (hak akses)
- **Laravel Socialite**: Untuk login dengan Google
- **Maatwebsite Excel**: Untuk export data ke Excel
- **Spatie Permission**: Untuk sistem permission berbasis role

---

## 📁 Struktur Project

```
Project Management/
├── app/                    # Kode aplikasi utama
│   ├── Events/            # Event yang dipicu ketika sesuatu terjadi
│   ├── Exports/           # Class untuk export data (Excel)
│   ├── Filament/          # File-file untuk admin panel Filament
│   │   ├── Actions/       # Action custom untuk Filament
│   │   ├── Pages/         # Halaman custom di admin panel
│   │   ├── Resources/     # Resource untuk setiap model (CRUD)
│   │   └── Widgets/       # Widget untuk dashboard
│   ├── Http/              # Controller dan middleware
│   ├── Imports/           # Class untuk import data
│   ├── Listeners/         # Listener yang merespons event
│   ├── Livewire/          # Komponen Livewire
│   ├── Mail/              # Template email
│   ├── Models/            # Model database (Eloquent)
│   ├── Policies/          # Policy untuk authorization
│   ├── Providers/         # Service provider
│   └── Services/          # Service class untuk logika bisnis
├── database/
│   ├── migrations/        # File migrasi database
│   ├── seeders/           # Data awal (seeder)
│   └── factories/         # Factory untuk testing
├── resources/
│   ├── views/             # Template Blade
│   ├── css/               # File CSS
│   └── js/                # File JavaScript
├── routes/                # File routing
├── config/                # File konfigurasi
└── public/                # File publik (akses langsung)
```

---

## 🗄️ Model Database dan Relasi

### 1. User (Pengguna)
**Tabel**: `users`

Menyimpan data pengguna yang bisa login ke sistem.

**Field Utama**:
- `id`: ID unik pengguna
- `name`: Nama pengguna
- `email`: Email pengguna
- `password`: Password (ter-enkripsi)
- `google_id`: ID Google (jika login via Google)

**Relasi**:
- `projects()`: Banyak proyek yang diikuti (many-to-many)
- `tickets()`: Tiket yang dibuat
- `assignedTickets()`: Tiket yang ditugaskan kepadanya
- `notifications()`: Notifikasi yang diterima

**Penjelasan**:
Model User mewakili setiap orang yang menggunakan sistem. Satu user bisa menjadi anggota banyak proyek, dan satu proyek bisa memiliki banyak anggota.

---

### 2. Project (Proyek)
**Tabel**: `projects`

Menyimpan data proyek yang sedang dikerjakan.

**Field Utama**:
- `id`: ID unik proyek
- `name`: Nama proyek
- `description`: Deskripsi proyek
- `ticket_prefix`: Prefix untuk ID tiket (contoh: "PROJ", "WEB")
- `color`: Warna untuk proyek (untuk tampilan)
- `start_date`: Tanggal mulai proyek
- `end_date`: Tanggal akhir proyek
- `pinned_date`: Tanggal ketika proyek di-pin (untuk dashboard)

**Relasi**:
- `members()` atau `users()`: Anggota tim proyek (many-to-many)
- `tickets()`: Tiket dalam proyek (one-to-many)
- `ticketStatuses()`: Status tiket untuk proyek (one-to-many)
- `epics()`: Epic dalam proyek (one-to-many)
- `notes()`: Catatan proyek (one-to-many)
- `externalAccess()`: Akses eksternal untuk klien (one-to-one)

**Method Penting**:
- `getProgressPercentageAttribute()`: Menghitung persentase progress proyek
- `getRemainingDaysAttribute()`: Menghitung sisa hari proyek
- `pin()` / `unpin()`: Untuk pin/unpin proyek di dashboard

**Penjelasan**:
Project adalah entitas utama dalam aplikasi. Setiap proyek memiliki prefix unik untuk tiketnya (misalnya "WEB-ABC123"). Proyek memiliki banyak anggota tim, banyak tiket, dan banyak status tiket yang bisa dikustomisasi.

---

### 3. Ticket (Tiket/Tugas)
**Tabel**: `tickets`

Menyimpan data tiket atau tugas dalam proyek.

**Field Utama**:
- `id`: ID unik tiket
- `project_id`: ID proyek yang memiliki tiket ini
- `ticket_status_id`: ID status tiket saat ini
- `priority_id`: ID prioritas tiket
- `name`: Judul tiket
- `description`: Deskripsi detail tiket
- `uuid`: ID unik tiket (contoh: "WEB-ABC123")
- `epic_id`: ID epic (jika tiket bagian dari epic)
- `created_by`: ID user yang membuat tiket
- `start_date`: Tanggal mulai
- `due_date`: Tanggal deadline

**Relasi**:
- `project()`: Proyek yang memiliki tiket ini (belongs-to)
- `status()`: Status tiket saat ini (belongs-to)
- `priority()`: Prioritas tiket (belongs-to)
- `assignees()`: User yang ditugaskan pada tiket (many-to-many)
- `creator()`: User yang membuat tiket (belongs-to)
- `epic()`: Epic yang memiliki tiket ini (belongs-to)
- `histories()`: Riwayat perubahan status (one-to-many)
- `comments()`: Komentar pada tiket (one-to-many)

**Event Otomatis**:
- **Saat Membuat Tiket** (`creating`):
  - Membuat UUID otomatis dengan format: `{prefix}-{random 6 karakter}`
  - Mengisi `created_by` dengan user yang sedang login
  
- **Saat Mengupdate Status** (`updating`):
  - Mencatat perubahan status ke `ticket_histories`

**Method Penting**:
- `assignUser($user)`: Menugaskan user ke tiket
- `unassignUser($user)`: Menghapus penugasan user
- `assignUsers($userIds)`: Menugaskan banyak user sekaligus
- `isAssignedTo($user)`: Mengecek apakah user ditugaskan

**Penjelasan**:
Ticket adalah unit pekerjaan terkecil dalam proyek. Setiap tiket memiliki status, prioritas, dan bisa ditugaskan ke banyak user. Tiket juga memiliki riwayat perubahan status dan komentar untuk kolaborasi tim.

---

### 4. TicketStatus (Status Tiket)
**Tabel**: `ticket_statuses`

Menyimpan status yang bisa digunakan untuk tiket dalam proyek.

**Field Utama**:
- `id`: ID unik status
- `project_id`: ID proyek yang memiliki status ini
- `name`: Nama status (contoh: "To Do", "In Progress", "Done")
- `color`: Warna untuk status (untuk tampilan)
- `sort_order`: Urutan tampilan status
- `is_completed`: Apakah status ini menandakan tiket selesai

**Relasi**:
- `project()`: Proyek yang memiliki status ini (belongs-to)
- `tickets()`: Tiket dengan status ini (one-to-many)

**Penjelasan**:
Setiap proyek bisa memiliki status tiket sendiri. Status digunakan untuk melacak kemajuan tiket. Status dengan `is_completed = true` akan dihitung sebagai tiket selesai dalam perhitungan progress.

---

### 5. TicketPriority (Prioritas Tiket)
**Tabel**: `ticket_priorities`

Menyimpan tingkat prioritas yang bisa digunakan untuk tiket.

**Field Utama**:
- `id`: ID unik prioritas
- `name`: Nama prioritas (contoh: "Low", "Medium", "High", "Critical")
- `color`: Warna untuk prioritas

**Relasi**:
- `tickets()`: Tiket dengan prioritas ini (one-to-many)

**Penjelasan**:
Prioritas bersifat global (tidak spesifik per proyek), sehingga semua proyek menggunakan prioritas yang sama. Ini membantu konsistensi dalam menentukan urgensi pekerjaan.

---

### 6. Epic
**Tabel**: `epics`

Menyimpan epic (kelompok tiket besar) dalam proyek.

**Field Utama**:
- `id`: ID unik epic
- `project_id`: ID proyek yang memiliki epic ini
- `name`: Nama epic
- `description`: Deskripsi epic
- `start_date`: Tanggal mulai epic
- `end_date`: Tanggal akhir epic
- `sort_order`: Urutan tampilan epic

**Relasi**:
- `project()`: Proyek yang memiliki epic ini (belongs-to)
- `tickets()`: Tiket yang termasuk dalam epic ini (one-to-many)

**Penjelasan**:
Epic adalah cara untuk mengelompokkan tiket terkait menjadi satu unit besar. Misalnya, epic "User Authentication" bisa berisi banyak tiket seperti "Login Page", "Registration", "Password Reset", dll.

---

### 7. TicketComment (Komentar Tiket)
**Tabel**: `ticket_comments`

Menyimpan komentar pada tiket.

**Field Utama**:
- `id`: ID unik komentar
- `ticket_id`: ID tiket yang dikomentari
- `user_id`: ID user yang mengomentari
- `comment`: Isi komentar

**Relasi**:
- `ticket()`: Tiket yang dikomentari (belongs-to)
- `user()`: User yang mengomentari (belongs-to)

**Event Otomatis**:
- **Saat Membuat Komentar** (`created`):
  - Mengirim notifikasi ke user terkait (creator, assignees, commenters lain)
  
- **Saat Mengupdate Komentar** (`updated`):
  - Mengirim notifikasi update komentar

**Penjelasan**:
Komentar memungkinkan tim untuk berdiskusi tentang tiket secara langsung dalam aplikasi. Setiap komentar terhubung dengan tiket dan user yang mengomentari.

---

### 8. TicketHistory (Riwayat Tiket)
**Tabel**: `ticket_histories`

Menyimpan riwayat perubahan status tiket.

**Field Utama**:
- `id`: ID unik riwayat
- `ticket_id`: ID tiket yang berubah
- `user_id`: ID user yang mengubah status
- `ticket_status_id`: ID status baru
- `created_at`: Waktu perubahan

**Relasi**:
- `ticket()`: Tiket yang berubah (belongs-to)
- `user()`: User yang mengubah (belongs-to)
- `status()`: Status baru (belongs-to)

**Penjelasan**:
Setiap kali status tiket berubah, riwayatnya dicatat. Ini memungkinkan untuk melihat timeline perubahan status tiket dan siapa yang melakukan perubahan.

---

### 9. Notification (Notifikasi)
**Tabel**: `notifications`

Menyimpan notifikasi untuk user.

**Field Utama**:
- `id`: ID unik notifikasi
- `user_id`: ID user yang menerima notifikasi
- `type`: Jenis notifikasi (contoh: "comment_added", "project_assigned")
- `title`: Judul notifikasi
- `message`: Pesan notifikasi
- `data`: Data tambahan (JSON)
- `read_at`: Waktu ketika notifikasi dibaca (null jika belum dibaca)

**Relasi**:
- `user()`: User yang menerima notifikasi (belongs-to)

**Method Penting**:
- `isRead()`: Mengecek apakah notifikasi sudah dibaca
- `isUnread()`: Mengecek apakah notifikasi belum dibaca
- `markAsRead()`: Menandai notifikasi sebagai sudah dibaca

**Penjelasan**:
Notifikasi digunakan untuk memberi tahu user tentang perubahan penting, seperti komentar baru, penugasan proyek, atau perubahan tiket. Notifikasi bisa dibaca atau belum dibaca.

---

### 10. ExternalAccess (Akses Eksternal)
**Tabel**: `external_access`

Menyimpan token akses untuk portal klien eksternal.

**Field Utama**:
- `id`: ID unik akses
- `project_id`: ID proyek yang bisa diakses
- `access_token`: Token untuk akses (random 32 karakter)
- `password`: Password untuk login (random 8 karakter)
- `is_active`: Apakah akses aktif
- `last_accessed_at`: Waktu terakhir diakses

**Relasi**:
- `project()`: Proyek yang bisa diakses (belongs-to)

**Method Penting**:
- `generateForProject($projectId)`: Membuat akses baru untuk proyek
- `updateLastAccessed()`: Update waktu terakhir akses

**Penjelasan**:
ExternalAccess memungkinkan klien mengakses dashboard proyek tanpa perlu login ke sistem utama. Setiap proyek bisa memiliki satu token akses dengan password.

---

### 11. ProjectNote (Catatan Proyek)
**Tabel**: `project_notes`

Menyimpan catatan untuk proyek.

**Field Utama**:
- `id`: ID unik catatan
- `project_id`: ID proyek
- `created_by`: ID user yang membuat catatan
- `title`: Judul catatan
- `content`: Isi catatan
- `note_date`: Tanggal catatan

**Relasi**:
- `project()`: Proyek yang memiliki catatan (belongs-to)
- `creator()`: User yang membuat catatan (belongs-to)

**Penjelasan**:
ProjectNote memungkinkan tim untuk mencatat informasi penting tentang proyek, seperti meeting notes, keputusan penting, atau catatan lainnya.

---

### 12. Setting (Pengaturan)
**Tabel**: `settings`

Menyimpan pengaturan sistem atau pengaturan user.

**Field Utama**:
- `id`: ID unik pengaturan
- `group`: Grup pengaturan
- `key`: Kunci pengaturan
- `value`: Nilai pengaturan
- `user_id`: ID user (null untuk pengaturan global)

**Method Penting**:
- `getUserValue($key, $default, $userId)`: Mendapat nilai pengaturan user
- `setUserValue($key, $value, $group, $userId)`: Menetapkan nilai pengaturan user
- `setValue($key, $value, $group)`: Menetapkan nilai pengaturan global

**Penjelasan**:
Setting digunakan untuk menyimpan konfigurasi yang bisa berbeda per user atau global untuk semua user.

---

## 🔗 Relasi Antara Model

### Diagram Relasi Sederhana

```
User (Pengguna)
  ├── belongsToMany → Project (melalui project_members)
  ├── hasMany → Ticket (tiket yang dibuat)
  ├── belongsToMany → Ticket (tiket yang ditugaskan, melalui ticket_users)
  └── hasMany → Notification

Project (Proyek)
  ├── belongsToMany → User (anggota tim, melalui project_members)
  ├── hasMany → Ticket
  ├── hasMany → TicketStatus
  ├── hasMany → Epic
  ├── hasMany → ProjectNote
  └── hasOne → ExternalAccess

Ticket (Tiket)
  ├── belongsTo → Project
  ├── belongsTo → TicketStatus
  ├── belongsTo → TicketPriority
  ├── belongsTo → Epic
  ├── belongsTo → User (creator)
  ├── belongsToMany → User (assignees, melalui ticket_users)
  ├── hasMany → TicketComment
  └── hasMany → TicketHistory

TicketStatus (Status)
  ├── belongsTo → Project
  └── hasMany → Ticket

Epic (Epic)
  ├── belongsTo → Project
  └── hasMany → Ticket

TicketComment (Komentar)
  ├── belongsTo → Ticket
  └── belongsTo → User

TicketHistory (Riwayat)
  ├── belongsTo → Ticket
  ├── belongsTo → User
  └── belongsTo → TicketStatus
```

---

## 🎨 Penjelasan Fitur Utama

### 1. Manajemen Proyek

**Lokasi**: `app/Filament/Resources/Projects/ProjectResource.php`

**Fitur**:
- Membuat, mengedit, dan menghapus proyek
- Menentukan prefix tiket (contoh: "WEB", "MOBILE")
- Menentukan warna proyek untuk tampilan
- Menentukan tanggal mulai dan akhir proyek
- Pin/unpin proyek untuk dashboard
- Menambahkan anggota tim ke proyek
- Membuat status tiket untuk proyek
- Membuat epic untuk proyek
- Menambahkan catatan proyek

**Cara Kerja**:
1. User membuat proyek dengan mengisi nama, deskripsi, prefix tiket, dll
2. Sistem membuat proyek dan status default (jika dipilih)
3. User bisa menambahkan anggota tim melalui RelationManager
4. User bisa membuat status tiket sesuai kebutuhan workflow tim
5. Progress proyek dihitung otomatis berdasarkan tiket yang selesai

---

### 2. Manajemen Tiket

**Lokasi**: `app/Filament/Resources/Tickets/TicketResource.php`

**Fitur**:
- Membuat, mengedit, dan menghapus tiket
- Menugaskan tiket ke banyak user (multi-assign)
- Mengubah status tiket
- Menentukan prioritas tiket
- Menentukan epic untuk tiket
- Menentukan tanggal mulai dan deadline
- Menambahkan komentar pada tiket
- Melihat riwayat perubahan status
- Filter tiket berdasarkan proyek, status, prioritas, dll
- Export tiket ke Excel

**Cara Kerja**:
1. User memilih proyek untuk tiket
2. Sistem otomatis membuat UUID dengan format: `{prefix}-{random}`
3. User mengisi detail tiket (nama, deskripsi, status, prioritas, dll)
4. User bisa menugaskan tiket ke banyak user sekaligus
5. Ketika status berubah, sistem mencatat ke TicketHistory
6. User bisa menambahkan komentar yang akan memicu notifikasi

**Event yang Terjadi**:
- **Saat Membuat Tiket**: UUID otomatis dibuat, `created_by` diisi
- **Saat Mengubah Status**: Riwayat dicatat di TicketHistory
- **Saat Menambahkan Komentar**: Notifikasi dikirim ke user terkait

---

### 3. Board View (Kanban Board)

**Lokasi**: `app/Filament/Pages/ProjectBoard.php`

**Fitur**:
- Tampilan kanban board untuk tiket
- Drag and drop tiket antar kolom status
- Filter berdasarkan proyek
- Quick edit tiket dari board

**Cara Kerja**:
1. User memilih proyek untuk dilihat board-nya
2. Sistem menampilkan kolom berdasarkan status tiket proyek
3. Tiket ditampilkan sebagai kartu dalam kolom status
4. User bisa drag and drop tiket untuk mengubah status
5. User bisa klik tiket untuk quick edit

---

### 4. Timeline View

**Lokasi**: `app/Filament/Pages/ProjectTimeline.php` dan `app/Filament/Pages/TicketTimeline.php`

**Fitur**:
- Tampilan timeline proyek dengan Gantt chart
- Tampilan timeline tiket
- Visualisasi start date dan end date
- Export timeline ke Excel

**Cara Kerja**:
1. User memilih proyek atau melihat semua proyek
2. Sistem menampilkan timeline dengan start date dan end date
3. User bisa melihat tiket dalam timeline
4. User bisa export timeline untuk perencanaan

---

### 5. Sistem Notifikasi

**Lokasi**: `app/Services/NotificationService.php`

**Fitur**:
- Notifikasi in-app (dalam aplikasi)
- Notifikasi email (via queue)
- Notifikasi ketika komentar ditambahkan/diupdate
- Notifikasi ketika user ditambahkan/dihapus dari proyek

**Cara Kerja**:
1. Event dipicu (contoh: ProjectMemberAttached, komentar ditambahkan)
2. Listener menangani event
3. NotificationService membuat notifikasi in-app
4. NotificationService mengirim email via queue
5. User melihat notifikasi di dashboard

**Event dan Listener**:
- `ProjectMemberAttached` → `SendProjectAssignmentNotification`
- `ProjectMemberDetached` → `SendProjectRemovalNotification`
- `TicketComment created/updated` → Notifikasi via model boot

---

### 6. Portal Klien (External Dashboard)

**Lokasi**: `app/Livewire/ExternalDashboard.php`

**Fitur**:
- Dashboard khusus untuk klien
- Login dengan token dan password
- Melihat progress proyek
- Melihat tiket proyek
- Melihat statistik proyek
- Timeline view untuk klien

**Cara Kerja**:
1. Admin membuat external access untuk proyek (generate token)
2. Admin memberikan token dan password ke klien
3. Klien mengakses `/external/{token}`
4. Klien login dengan password
5. Klien melihat dashboard proyek (read-only)

**Keamanan**:
- Token unik untuk setiap proyek
- Password untuk autentikasi
- Hanya bisa melihat proyek yang terkait dengan token
- Tidak bisa mengedit data

---

### 7. Sistem Role dan Permission

**Lokasi**: `config/filament-shield.php`, `app/Policies/`

**Fitur**:
- Role-based access control (RBAC)
- Permission untuk setiap resource
- Super admin yang memiliki akses penuh
- Policy untuk authorization

**Cara Kerja**:
1. Sistem menggunakan Spatie Permission untuk RBAC
2. Filament Shield mengintegrasikan dengan Filament
3. Setiap resource memiliki permission (view, create, update, delete)
4. User dengan role tertentu memiliki permission tertentu
5. Policy memeriksa permission sebelum mengizinkan akses

**Role Default**:
- `super_admin`: Akses penuh ke semua fitur
- Role lain bisa dibuat sesuai kebutuhan

---

### 8. Google Login

**Lokasi**: `app/Http/Controllers/Auth/GoogleController.php`

**Fitur**:
- Login dengan akun Google
- Auto-create user jika belum ada
- Link akun Google ke user yang sudah ada

**Cara Kerja**:
1. User klik "Login with Google"
2. Sistem redirect ke Google OAuth
3. User authorize aplikasi
4. Google redirect kembali dengan token
5. Sistem mendapatkan info user dari Google
6. Sistem mencari user berdasarkan google_id atau email
7. Jika user ada, login. Jika tidak, buat user baru

---

### 9. Export Data

**Lokasi**: `app/Exports/TicketsExport.php`

**Fitur**:
- Export tiket ke Excel
- Pilih kolom yang ingin di-export
- Format Excel dengan styling

**Cara Kerja**:
1. User memilih tiket yang ingin di-export
2. User memilih kolom yang ingin disertakan
3. Sistem menggunakan Maatwebsite Excel untuk membuat file
4. File Excel di-download oleh user

---

### 10. Dashboard dan Widget

**Lokasi**: `app/Filament/Widgets/`

**Fitur**:
- Statistik overview
- Chart tiket per bulan
- Chart tiket per proyek
- Chart status proyek
- Timeline proyek
- Aktivitas terkini
- Statistik user

**Widget yang Tersedia**:
- `StatsOverview`: Statistik umum
- `MonthlyTicketTrendChart`: Trend tiket per bulan
- `TicketsPerProjectChart`: Chart tiket per proyek
- `ProjectStatusChart`: Chart status proyek
- `ProjectTimeline`: Timeline proyek
- `RecentActivityTable`: Tabel aktivitas terkini
- `UserStatisticsChart`: Statistik user

---

## 📂 Struktur File dan Folder

### app/Models/

Berisi model Eloquent yang mewakili tabel database.

**File Penting**:
- `User.php`: Model user
- `Project.php`: Model proyek
- `Ticket.php`: Model tiket
- `TicketStatus.php`: Model status tiket
- `TicketPriority.php`: Model prioritas tiket
- `Epic.php`: Model epic
- `TicketComment.php`: Model komentar
- `TicketHistory.php`: Model riwayat tiket
- `Notification.php`: Model notifikasi
- `ExternalAccess.php`: Model akses eksternal
- `ProjectNote.php`: Model catatan proyek
- `Setting.php`: Model pengaturan

**Penjelasan**:
Setiap model mewakili satu tabel database. Model memiliki method untuk relasi, event, dan atribut yang dihitung (computed attributes).

---

### app/Filament/Resources/

Berisi resource Filament untuk CRUD (Create, Read, Update, Delete) setiap model.

**Struktur**:
```
Resources/
├── Projects/
│   ├── ProjectResource.php          # Resource utama
│   ├── Pages/                       # Halaman custom
│   │   ├── CreateProject.php
│   │   ├── EditProject.php
│   │   ├── ListProjects.php
│   │   └── ViewProject.php
│   └── RelationManagers/            # Manager untuk relasi
│       ├── MembersRelationManager.php
│       ├── TicketsRelationManager.php
│       ├── TicketStatusesRelationManager.php
│       ├── EpicsRelationManager.php
│       └── NotesRelationManager.php
├── Tickets/
│   ├── TicketResource.php
│   └── Pages/
├── Users/
│   ├── UserResource.php
│   └── RelationManagers/
└── ...
```

**Penjelasan**:
Resource Filament mengatur bagaimana data ditampilkan dan dimanipulasi di admin panel. Setiap resource memiliki form, table, dan action yang bisa dikustomisasi.

---

### app/Filament/Pages/

Berisi halaman custom di admin panel.

**Halaman yang Tersedia**:
- `ProjectBoard.php`: Kanban board untuk proyek
- `ProjectTimeline.php`: Timeline proyek
- `TicketTimeline.php`: Timeline tiket
- `Leaderboard.php`: Leaderboard anggota tim
- `UserContributions.php`: Kontribusi user
- `EpicsOverview.php`: Overview epic
- `SystemSettings.php`: Pengaturan sistem

**Penjelasan**:
Halaman custom memungkinkan untuk membuat fitur khusus yang tidak bisa dicapai hanya dengan resource standar.

---

### app/Filament/Widgets/

Berisi widget untuk dashboard.

**Widget yang Tersedia**:
- `StatsOverview.php`: Statistik overview
- `MonthlyTicketTrendChart.php`: Chart trend bulanan
- `TicketsPerProjectChart.php`: Chart tiket per proyek
- `ProjectStatusChart.php`: Chart status proyek
- `ProjectTimeline.php`: Timeline proyek
- `RecentActivityTable.php`: Tabel aktivitas
- `UserStatisticsChart.php`: Statistik user

**Penjelasan**:
Widget adalah komponen kecil yang menampilkan informasi di dashboard. Widget bisa berupa statistik, chart, atau tabel.

---

### app/Events/

Berisi event yang dipicu ketika sesuatu terjadi.

**Event yang Tersedia**:
- `ProjectMemberAttached.php`: Dipicu ketika user ditambahkan ke proyek
- `ProjectMemberDetached.php`: Dipicu ketika user dihapus dari proyek

**Penjelasan**:
Event memungkinkan untuk memisahkan logika ketika sesuatu terjadi. Event dipicu dan listener menangani event tersebut.

---

### app/Listeners/

Berisi listener yang merespons event.

**Listener yang Tersedia**:
- `SendProjectAssignmentNotification.php`: Mengirim notifikasi ketika user ditambahkan ke proyek
- `SendProjectRemovalNotification.php`: Mengirim notifikasi ketika user dihapus dari proyek

**Penjelasan**:
Listener menangani event yang dipicu. Listener bisa mengirim email, membuat notifikasi, atau melakukan action lainnya.

---

### app/Services/

Berisi service class untuk logika bisnis.

**Service yang Tersedia**:
- `NotificationService.php`: Service untuk mengelola notifikasi

**Penjelasan**:
Service class memusatkan logika bisnis yang kompleks. Service bisa dipanggil dari controller, listener, atau tempat lain.

---

### app/Livewire/

Berisi komponen Livewire untuk interaktivitas.

**Komponen yang Tersedia**:
- `ExternalDashboard.php`: Dashboard untuk portal klien
- `ExternalLogin.php`: Login untuk portal klien

**Penjelasan**:
Livewire memungkinkan untuk membuat komponen interaktif tanpa JavaScript kompleks. Komponen Livewire memiliki state dan method yang bisa dipanggil dari frontend.

---

### app/Http/Controllers/

Berisi controller untuk routing web.

**Controller yang Tersedia**:
- `Auth/GoogleController.php`: Controller untuk Google login

**Penjelasan**:
Controller menangani request HTTP dan mengembalikan response. Controller biasanya memanggil service atau model untuk memproses data.

---

### app/Mail/

Berisi template email.

**Template yang Tersedia**:
- `ProjectAssignmentNotification.php`: Email notifikasi penugasan proyek

**Penjelasan**:
Template email digunakan untuk mengirim email ke user. Template bisa menggunakan Blade untuk rendering HTML.

---

### app/Exports/

Berisi class untuk export data.

**Export yang Tersedia**:
- `TicketsExport.php`: Export tiket ke Excel
- `TicketsTimelineGantt.php`: Export timeline ke Excel
- `TicketTemplateExport.php`: Export template tiket

**Penjelasan**:
Export class menggunakan Maatwebsite Excel untuk membuat file Excel. Export class menentukan kolom, data, dan styling file Excel.

---

### app/Imports/

Berisi class untuk import data.

**Import yang Tersedia**:
- `TicketsImport.php`: Import tiket dari Excel

**Penjelasan**:
Import class menggunakan Maatwebsite Excel untuk membaca file Excel dan menyimpan data ke database.

---

### app/Policies/

Berisi policy untuk authorization.

**Policy yang Tersedia**:
- `ProjectPolicy.php`: Policy untuk proyek
- `TicketPolicy.php`: Policy untuk tiket
- `UserPolicy.php`: Policy untuk user
- `NotificationPolicy.php`: Policy untuk notifikasi
- `TicketCommentPolicy.php`: Policy untuk komentar
- `TicketPriorityPolicy.php`: Policy untuk prioritas
- `RolePolicy.php`: Policy untuk role

**Penjelasan**:
Policy menentukan siapa yang bisa melakukan apa. Policy memeriksa permission user sebelum mengizinkan akses.

---

### database/migrations/

Berisi migrasi database.

**Penjelasan**:
Migrasi menentukan struktur tabel database. Migrasi bisa dibuat, diubah, atau dihapus untuk mengubah struktur database.

---

### routes/web.php

Berisi routing web.

**Route yang Tersedia**:
- `/`: Halaman welcome
- `/auth/google`: Redirect ke Google OAuth
- `/auth/google/callback`: Callback dari Google OAuth
- `/external/{token}`: Login portal klien
- `/external/{token}/dashboard`: Dashboard portal klien

**Penjelasan**:
Routing menentukan URL dan controller/method yang menangani request.

---

### config/

Berisi file konfigurasi.

**File Penting**:
- `app.php`: Konfigurasi aplikasi
- `database.php`: Konfigurasi database
- `mail.php`: Konfigurasi email
- `filament.php`: Konfigurasi Filament
- `filament-shield.php`: Konfigurasi Filament Shield
- `permission.php`: Konfigurasi Spatie Permission

**Penjelasan**:
File konfigurasi menentukan pengaturan aplikasi, database, email, dll.

---

## 🔄 Alur Kerja Aplikasi

### 1. Alur Membuat Proyek

1. User login ke admin panel (`/admin`)
2. User klik "Projects" di sidebar
3. User klik "Create Project"
4. User mengisi form:
   - Nama proyek
   - Deskripsi
   - Prefix tiket (contoh: "WEB")
   - Warna proyek
   - Tanggal mulai dan akhir
   - Opsi untuk membuat status default
5. User klik "Create"
6. Sistem membuat proyek di database
7. Jika dipilih, sistem membuat status default (Backlog, To Do, In Progress, Review, Done)
8. User di-redirect ke halaman detail proyek
9. User bisa menambahkan anggota tim, membuat status custom, dll

---

### 2. Alur Membuat Tiket

1. User membuka proyek
2. User klik "Tickets" tab atau pergi ke "Tickets" di sidebar
3. User klik "Create Ticket"
4. User mengisi form:
   - Pilih proyek
   - Pilih status (otomatis terisi berdasarkan proyek)
   - Pilih prioritas
   - Nama tiket
   - Deskripsi
   - Pilih epic (opsional)
   - Pilih assignees (bisa banyak)
   - Tanggal mulai dan deadline
5. User klik "Create"
6. Sistem membuat tiket dengan UUID otomatis (contoh: "WEB-ABC123")
7. Sistem mengisi `created_by` dengan user yang login
8. Tiket ditampilkan di list tiket
9. User bisa melihat tiket di board view, timeline, dll

---

### 3. Alur Mengubah Status Tiket

1. User membuka tiket
2. User mengubah status di form atau drag & drop di board
3. Sistem menyimpan perubahan
4. Event `updating` dipicu pada model Ticket
5. Sistem mencatat perubahan ke `ticket_histories`
6. Tiket diperbarui dengan status baru
7. Progress proyek dihitung ulang otomatis

---

### 4. Alur Menambahkan Komentar

1. User membuka tiket
2. User menambahkan komentar
3. Sistem menyimpan komentar
4. Event `created` dipicu pada model TicketComment
5. NotificationService dipanggil
6. Sistem menentukan user yang harus diberi notifikasi:
   - Creator tiket
   - Assignees tiket
   - User yang pernah komentar
7. Sistem membuat notifikasi in-app untuk setiap user
8. Sistem mengirim email via queue (jika dikonfigurasi)
9. User melihat notifikasi di dashboard

---

### 5. Alur Menambahkan Anggota Tim

1. User membuka proyek
2. User klik "Members" tab
3. User klik "Attach"
4. User memilih user yang ingin ditambahkan
5. User klik "Attach"
6. Event `ProjectMemberAttached` dipicu
7. Listener `SendProjectAssignmentNotification` menangani event
8. NotificationService membuat notifikasi in-app
9. NotificationService mengirim email via queue
10. User yang ditambahkan melihat notifikasi

---

### 6. Alur Portal Klien

1. Admin membuka proyek
2. Admin membuat external access (generate token dan password)
3. Admin memberikan token dan password ke klien
4. Klien mengakses `/external/{token}`
5. Klien memasukkan password
6. Sistem memverifikasi token dan password
7. Sistem menyimpan session
8. Klien di-redirect ke dashboard
9. Klien melihat progress proyek, tiket, statistik
10. Klien bisa logout

---

## 💡 Penjelasan Kode Penting

### 1. Model Event (Boot Method)

**Lokasi**: `app/Models/Ticket.php`

```php
protected static function booted()
{
    // Dipanggil saat membuat tiket baru
    static::creating(function ($ticket) {
        // Membuat UUID otomatis
        if (empty($ticket->uuid)) {
            $project = Project::find($ticket->project_id);
            $prefix = $project ? $project->ticket_prefix : 'TKT';
            $randomString = Str::upper(Str::random(6));
            $ticket->uuid = "{$prefix}-{$randomString}";
        }

        // Mengisi created_by otomatis
        if (empty($ticket->created_by) && auth()->id()) {
            $ticket->created_by = auth()->id();
        }
    });

    // Dipanggil saat mengupdate tiket
    static::updating(function ($ticket) {
        // Mencatat perubahan status
        if ($ticket->isDirty('ticket_status_id')) {
            TicketHistory::create([
                'ticket_id' => $ticket->id,
                'user_id' => auth()->id(),
                'ticket_status_id' => $ticket->ticket_status_id,
            ]);
        }
    });
}
```

**Penjelasan**:
- `booted()` adalah method khusus Laravel yang dipanggil saat model di-boot
- `creating` dipanggil sebelum tiket disimpan (create)
- `updating` dipanggil sebelum tiket di-update
- `isDirty('ticket_status_id')` mengecek apakah field `ticket_status_id` berubah
- Event ini memastikan UUID dibuat otomatis dan riwayat status dicatat

---

### 2. Relasi Many-to-Many

**Lokasi**: `app/Models/Project.php`

```php
public function members(): BelongsToMany
{
    return $this->belongsToMany(User::class, 'project_members')
        ->withTimestamps();
}
```

**Penjelasan**:
- `belongsToMany` adalah relasi many-to-many
- `User::class` adalah model yang berelasi
- `'project_members'` adalah nama tabel pivot
- `withTimestamps()` menambahkan `created_at` dan `updated_at` ke tabel pivot
- Relasi ini memungkinkan satu proyek memiliki banyak anggota, dan satu user bisa menjadi anggota banyak proyek

---

### 3. Computed Attribute (Accessor)

**Lokasi**: `app/Models/Project.php`

```php
public function getProgressPercentageAttribute(): float
{
    $totalTickets = $this->tickets()->count();
    
    if ($totalTickets === 0) {
        return 0.0;
    }
    
    $completedTickets = $this->tickets()
        ->whereHas('status', function ($query) {
            $query->where('is_completed', true);
        })
        ->count();
    
    return round(($completedTickets / $totalTickets) * 100, 1);
}
```

**Penjelasan**:
- `getProgressPercentageAttribute()` adalah accessor yang membuat atribut virtual `progress_percentage`
- Accessor dipanggil ketika mengakses `$project->progress_percentage`
- Method ini menghitung persentase tiket yang selesai
- `whereHas('status')` mengecek relasi status
- Hasil dibulatkan ke 1 decimal place

**Cara Menggunakan**:
```php
$project = Project::find(1);
echo $project->progress_percentage; // Contoh: 75.5
```

---

### 4. Event dan Listener

**Lokasi**: `app/Providers/EventServiceProvider.php`

```php
protected $listen = [
    ProjectMemberAttached::class => [
        SendProjectAssignmentNotification::class,
    ],
    ProjectMemberDetached::class => [
        SendProjectRemovalNotification::class,
    ],
];
```

**Penjelasan**:
- `EventServiceProvider` mendaftarkan event dan listener
- Ketika event `ProjectMemberAttached` dipicu, listener `SendProjectAssignmentNotification` dipanggil
- Event dan listener memisahkan logika, membuat kode lebih maintainable

**Cara Memicu Event**:
```php
event(new ProjectMemberAttached($project, $user, $attachedBy));
```

---

### 5. Query Scope

**Lokasi**: `app/Models/Notification.php`

```php
public function scopeUnread(Builder $query): Builder
{
    return $query->whereNull('read_at');
}
```

**Penjelasan**:
- `scopeUnread` adalah query scope yang membuat method `unread()` pada query
- Scope memungkinkan untuk membuat query yang bisa digunakan kembali
- `whereNull('read_at')` menyeleksi notifikasi yang belum dibaca

**Cara Menggunakan**:
```php
$unreadNotifications = Notification::unread()->get();
// atau
$user->notifications()->unread()->get();
```

---

### 6. Policy untuk Authorization

**Lokasi**: `app/Policies/TicketPolicy.php`

```php
public function viewAny(User $user): bool
{
    return $user->hasPermissionTo('view_any_ticket');
}
```

**Penjelasan**:
- Policy menentukan siapa yang bisa melakukan apa
- `viewAny` menentukan siapa yang bisa melihat list tiket
- `hasPermissionTo` mengecek apakah user memiliki permission tertentu
- Policy dipanggil otomatis oleh Filament untuk authorization

---

### 7. Livewire Component

**Lokasi**: `app/Livewire/ExternalDashboard.php`

```php
public function mount($token)
{
    $this->token = $token;
    
    if (!Session::get('external_authenticated_' . $token)) {
        return redirect()->route('external.login', $token);
    }
    
    $externalAccess = ExternalAccess::where('access_token', $token)
        ->where('is_active', true)
        ->first();
    
    if (!$externalAccess) {
        abort(404, 'External access not found');
    }
    
    $this->project = $externalAccess->project;
    // ...
}
```

**Penjelasan**:
- `mount()` dipanggil saat komponen Livewire di-mount
- Method ini menginisialisasi data komponen
- Session dicek untuk autentikasi
- Jika tidak terautentikasi, redirect ke login
- Data proyek di-load berdasarkan token

---

### 8. Queue Job untuk Email

**Lokasi**: `app/Listeners/SendProjectAssignmentNotification.php`

```php
class SendProjectAssignmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    public function handle(ProjectMemberAttached $event): void
    {
        $this->notificationService->notifyProjectAssignment(
            $event->project,
            $event->user,
            $event->attachedBy
        );
    }
}
```

**Penjelasan**:
- `ShouldQueue` membuat listener dijalankan via queue (background)
- `InteractsWithQueue` memberikan method untuk berinteraksi dengan queue
- Email dikirim di background, tidak blocking request
- Queue worker harus berjalan untuk memproses job

---

### 9. Export dengan Maatwebsite Excel

**Lokasi**: `app/Exports/TicketsExport.php`

```php
class TicketsExport implements FromCollection, WithHeadings, WithMapping, WithStyles
{
    public function collection()
    {
        return $this->tickets;
    }

    public function headings(): array
    {
        return ['Ticket ID', 'Title', 'Status', ...];
    }

    public function map($ticket): array
    {
        return [
            $ticket->uuid,
            $ticket->name,
            $ticket->status->name,
            // ...
        ];
    }
}
```

**Penjelasan**:
- `FromCollection` menentukan data yang akan di-export
- `WithHeadings` menentukan header kolom
- `WithMapping` menentukan bagaimana data di-map ke kolom
- `WithStyles` menentukan styling Excel
- Export class digunakan dengan `Excel::download()`

---

### 10. Filter Query di Resource

**Lokasi**: `app/Filament/Resources/Tickets/TicketResource.php`

```php
public static function getEloquentQuery(): Builder
{
    $query = parent::getEloquentQuery();

    if (! auth()->user()->hasRole(['super_admin'])) {
        $query->where(function ($query) {
            $query->whereHas('assignees', function ($query) {
                    $query->where('users.id', auth()->id());
                })
                ->orWhere('created_by', auth()->id())
                ->orWhereHas('project.members', function ($query) {
                    $query->where('users.id', auth()->id());
                });
        });
    }

    return $query;
}
```

**Penjelasan**:
- `getEloquentQuery()` mengubah query dasar untuk resource
- Method ini membatasi tiket yang bisa dilihat user
- User non-admin hanya bisa melihat:
  - Tiket yang ditugaskan kepadanya
  - Tiket yang dibuatnya
  - Tiket dalam proyek yang dia ikuti
- Super admin bisa melihat semua tiket

---

## 🎓 Konsep Penting untuk Pemula

### 1. MVC (Model-View-Controller)

**Model**: Mewakili data dan logika bisnis (contoh: `Ticket.php`)
**View**: Tampilan untuk user (contoh: Blade template, Filament resource)
**Controller**: Menangani request dan response (contoh: `GoogleController.php`)

**Alur**:
1. User membuat request (contoh: klik tombol)
2. Route mengarahkan ke controller
3. Controller memanggil model untuk mengambil data
4. Controller mengembalikan view dengan data
5. View ditampilkan ke user

---

### 2. Eloquent ORM

**ORM (Object-Relational Mapping)**: Cara untuk berinteraksi dengan database menggunakan object, bukan SQL langsung.

**Contoh**:
```php
// Mengambil semua tiket
$tickets = Ticket::all();

// Mengambil tiket dengan ID 1
$ticket = Ticket::find(1);

// Membuat tiket baru
$ticket = Ticket::create([
    'name' => 'Fix bug',
    'project_id' => 1,
]);

// Update tiket
$ticket->update(['name' => 'Fix critical bug']);

// Hapus tiket
$ticket->delete();
```

---

### 3. Relasi Database

**One-to-Many**: Satu proyek memiliki banyak tiket
```php
// Di Project model
public function tickets(): HasMany
{
    return $this->hasMany(Ticket::class);
}

// Penggunaan
$project->tickets; // Mengambil semua tiket proyek
```

**Many-to-Many**: Satu proyek memiliki banyak anggota, satu user bisa di banyak proyek
```php
// Di Project model
public function members(): BelongsToMany
{
    return $this->belongsToMany(User::class, 'project_members');
}

// Penggunaan
$project->members; // Mengambil semua anggota proyek
$project->members()->attach($userId); // Menambahkan anggota
```

**Belongs-To**: Satu tiket milik satu proyek
```php
// Di Ticket model
public function project(): BelongsTo
{
    return $this->belongsTo(Project::class);
}

// Penggunaan
$ticket->project; // Mengambil proyek tiket
```

---

### 4. Event dan Listener

**Event**: Kejadian yang terjadi dalam aplikasi (contoh: user ditambahkan ke proyek)
**Listener**: Kode yang dijalankan ketika event terjadi (contoh: kirim email)

**Alur**:
1. Event dipicu: `event(new ProjectMemberAttached($project, $user, $by));`
2. Laravel mencari listener untuk event tersebut
3. Listener dijalankan: `SendProjectAssignmentNotification::handle()`
4. Listener mengirim email atau melakukan action lain

---

### 5. Middleware

**Middleware**: Kode yang dijalankan sebelum request sampai ke controller

**Contoh**: Autentikasi
1. User mengakses halaman yang memerlukan login
2. Middleware mengecek apakah user sudah login
3. Jika belum login, redirect ke halaman login
4. Jika sudah login, lanjut ke controller

---

### 6. Service Container dan Dependency Injection

**Service Container**: Wadah untuk mengelola dependensi
**Dependency Injection**: Memberikan dependensi ke class melalui constructor

**Contoh**:
```php
class SendProjectAssignmentNotification
{
    public function __construct(
        private NotificationService $notificationService
    ) {}
}
```

Laravel otomatis memberikan instance `NotificationService` ketika membuat instance `SendProjectAssignmentNotification`.

---

### 7. Queue dan Jobs

**Queue**: Antrian pekerjaan yang dijalankan di background
**Job**: Pekerjaan yang dijalankan via queue

**Keuntungan**:
- Request tidak blocking (tidak menunggu email dikirim)
- Bisa retry jika gagal
- Bisa dijalankan oleh worker terpisah

**Contoh**: Mengirim email
1. Email dikirim via queue (tidak blocking)
2. Queue worker memproses email di background
3. User tidak perlu menunggu email dikirim

---

## 🔧 Tips untuk Pemula

### 1. Memahami Struktur File

- **Model**: Di `app/Models/`, mewakili tabel database
- **Controller**: Di `app/Http/Controllers/`, menangani request
- **View**: Di `resources/views/`, tampilan untuk user
- **Route**: Di `routes/web.php`, menentukan URL

### 2. Debugging

- Gunakan `dd()` untuk dump and die (menghentikan eksekusi dan menampilkan data)
- Gunakan `logger()` untuk log data ke file
- Gunakan `var_dump()` atau `print_r()` untuk melihat isi variable

### 3. Database

- Gunakan migration untuk mengubah struktur database
- Gunakan seeder untuk data awal
- Gunakan tinker untuk testing: `php artisan tinker`

### 4. Filament

- Resource adalah CRUD untuk model
- RelationManager untuk mengelola relasi
- Widget untuk dashboard
- Page untuk halaman custom

### 5. Best Practices

- Gunakan service class untuk logika bisnis kompleks
- Gunakan event/listener untuk memisahkan logika
- Gunakan policy untuk authorization
- Gunakan queue untuk pekerjaan yang berat
- Gunakan cache untuk data yang sering diakses

---

## 📝 Kesimpulan

Aplikasi **DewaKoding Project Management** adalah aplikasi manajemen proyek yang lengkap dengan fitur:

1. **Manajemen Proyek**: Membuat dan mengelola proyek dengan berbagai pengaturan
2. **Manajemen Tiket**: Membuat, menugaskan, dan melacak tiket
3. **Manajemen Tim**: Menambahkan anggota tim dan mengatur peran
4. **Pelacakan Progress**: Melihat kemajuan melalui berbagai tampilan
5. **Notifikasi**: Notifikasi in-app dan email
6. **Portal Klien**: Dashboard khusus untuk klien
7. **Export/Import**: Export data ke Excel
8. **Role & Permission**: Sistem hak akses berbasis role

Aplikasi ini dibangun dengan Laravel dan Filament, menggunakan konsep MVC, Eloquent ORM, Event/Listener, Queue, dan berbagai best practices.

---

## 🚀 Langkah Selanjutnya

1. **Pelajari Laravel**: Pahami dasar-dasar Laravel (routing, controller, model, view)
2. **Pelajari Filament**: Pahami cara membuat resource, page, dan widget
3. **Pelajari Database**: Pahami relasi database dan query builder
4. **Praktek**: Coba modifikasi fitur yang ada atau tambah fitur baru
5. **Baca Dokumentasi**: Baca dokumentasi Laravel dan Filament untuk detail lebih lanjut

---

**Dokumentasi ini dibuat untuk membantu pemula memahami codebase dengan mudah. Jika ada pertanyaan, silakan baca dokumentasi Laravel dan Filament, atau tanyakan pada tim developer.**

---

*Terakhir diupdate: 2025*

