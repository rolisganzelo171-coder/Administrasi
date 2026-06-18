# Administrasi Sekolah & Dapodik Hub - SMP Negeri 1 Bojongkenyot

Aplikasi web administrasi sekolah *production-ready* yang diintegrasikan secara terpadu dengan standar Dapodikdasmen Kemdikbud RI & EMIS GTK Kemenag.

Repository ini dikembangkan menggunakan **React, TypeScript, Tailwind CSS, Recharts,** dan library **SheetJS (xlsx)** untuk simulasi impor/ekspor data Dapodikdasmen berkecepatan tinggi secara handal. Dokumentasi ini juga menyediakan konfigurasi lengkap bagi integrasi jangka panjang dengan **Next.js 14 App Router, Prisma ORM, PostgreSQL,** dan **NextAuth**.

---

## 🚀 Fitur Utama & Standar Nasional

1. **Sistem Peran Multi-User (RBAC):**
   - **Super Admin & Operator Dapodik:** Berwenang mengelola data profil sekolah, merestorasi data dasar, mengedit/menghapus/menyaring biodata Peserta Didik & GTK, serta memonitor audit log transaksi.
   - **Guru Pendidik:** Berwenang menginput absensi harian dan rekapitulasi nilai UTS/UAS/Harian per rombel binaan dengan auto-konversi nilai huruf A-D. Dilengkapi form Jurnal Mengajar EMIS dengan fitur offline-draft auto-save ke `localStorage`.
   - **Siswa / Murid:** Mengakses rapor digital pribadi lengkap dengan kalender absen bulanan visual dan grafik statistik pencapaian mata pelajaran menggunakan Recharts.

2. **Validasi Checksum Nasional (Real-Time):**
   - Mendeteksi duplikasi NISN (10 digit numerik) & NIK (16 digit numerik dengan validasi tanggal lahir terenkripsi).
   - Menampilkan daftar anomali/eror data kependudukan langsung pada Beranda Operator untuk segera diperbaiki.

3. **Impor & Ekspor Dapodikdasmen Terintegrasi:**
   - **Ekspor Massal (.xlsx):** Menghasilkan file lembar sebar Excel berisi 27 tabel standardisasi Dapodikdasmen (F-SEKOLAH, F-PESERTADIDIK, F-GURUTENDIK, F-ROMBONGANBELAJAR, F-NILAI, F-ABSENSI, dsb).
   - **Impor Massal (.xlsx):** Operator dapat mengunduh berkas template siswa baru, mengedit, lalu menyeret-dan-melepas (drag & drop) berkas teregistrasi ke portal untuk digabungkan secara instan.

4. **Laporan Cetak Rapor Digital (PDF):**
   - Menghasilkan berkas KTSP / Kurikulum Merdeka SMPN 1 formal dengan Kop Kepala Dinas Kabupaten Bekasi lengkap dengan tanda tangan elektronik, tabel kehadiran, nilai rata-rata tertimbang, deskripsi capaian, dan catatan wali kelas.

---

## 🛠️ Langkah Menjalankan Aplikasi (Sandbox Mode)

Aplikasi ini dapat langsung dicompile dan dijalankan di browser sandbox AI Studio dengan performa mulus instan dan database Durabilitas lokal yang awet di `localStorage` (dilengkapi tombol Reset di sudut kanan bawah ke Kondisi Seed 90 Siswa Bawaan).

Untuk menjalankannya di komputer lokal Anda, lakukan langkah berikut:

### 1. Prasyarat Lokal
Pastikan perangkat Anda telah terinstal Node.js v18+, Docker, dan pnpm/npm.

### 2. Instalasi Dependensi
```bash
# Menggunakan pnpm
pnpm install

# Atau menggunakan npm
npm install
```

### 3. Menjalankan Database PostgreSQL via Docker Compose
Buat file `docker-compose.yml` di direktori utama:
```yaml
version: '3.8'
services:
  db:
    image: postgres:15-alpine
    container_name: dapodik-postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: dapodik_db
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```
Lalu jalankan kontainer database:
```bash
docker compose up -d
```

### 4. Sinkronisasi Skema database Prisma (Prisma Migrate)
Konfigurasikan database URL Anda di file `.env` lokal:
```env
DATABASE_URL="postgresql://postgres:mysecretpassword@localhost:5432/dapodik_db?schema=public"
NEXTAUTH_SECRET="rahasia_negari_kemdikbud_321"
```
Jalankan migrasi skema database relasional Dapodik:
```bash
# Membuat migrasi dan menerapkannya ke DB
npx prisma migrate dev --name init_dapodik_schema
```

### 5. Menjalankan Seed Awal Database
Jalankan perintah seed untuk mengisi database dengan profil SMPN 1, 1 Kepala Sekolah, 2 Operator, 5 Guru mapel bersertifikasi, 3 Rombel (VII-A, VII-B, VIII-A), dan 90 Siswa dummy bawaan:
```bash
npx prisma db seed
```

### 6. Menjalankan Server Pengembangan Lokal
```bash
pnpm dev
```
Buka URL `http://localhost:3000` di peramban Anda.

---

## 🗄️ Skema Data Prisma Terintegrasi (Prisma Schema Reference)

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

enum Role {
  SUPER_ADMIN
  OPERATOR_DAPODIK
  GURU
  MURID
}

enum JenisKelamin {
  L
  P
}

enum StatusSiswa {
  AKTIF
  LULUS
  MUTASI
  KELUAR
}

enum StatusKepegawaian {
  PNS
  PPPK
  GTT
  HONORER
}

enum JenisPresensi {
  H
  S
  I
  A
}

enum JenisNilai {
  UTS
  UAS
  HARIAN
}

model Sekolah {
  id                   String   @id @default(uuid())
  npsn                 String   @unique
  nama                 String
  alamat               String
  id_kecamatan_dapodik String
  tahun_ajaran_aktif   String
}

model PesertaDidik {
  id           String        @id @default(uuid())
  nisn         String        @unique
  nik          String        @unique
  nama         String
  jk           JenisKelamin
  tgl_lahir    DateTime
  agama        String
  no_kk        String
  nama_ayah    String
  nama_ibu     String
  alamat       String
  rombel_id    String
  rombel       Rombel        @relation(fields: [rombel_id], references: [id])
  status       StatusSiswa   @default(AKTIF)
  absensi      Absensi[]
  nilai        Nilai[]
}

model GTK {
  id                 String           @id @default(uuid())
  nuptk              String?          @unique
  nik                String           @unique
  nip                String?          @unique
  nama               String
  mapel_sertifikasi  String
  status_kepegawaian StatusKepegawaian
  golongan           String
  tmt                DateTime
  user_id            String           @unique
  user               User             @relation(fields: [user_id], references: [id])
  rombel_wali        Rombel[]
  pembelajaran       Pembelajaran[]
  jurnal             JurnalMengajar[]
}

model Rombel {
  id           String         @id @default(uuid())
  nama         String
  tingkat      Int
  kurikulum    String
  wali_kelas_id String
  wali_kelas   GTK            @relation(fields: [wali_kelas_id], references: [id])
  tahun_ajaran String
  siswa        PesertaDidik[]
  pembelajaran Pembelajaran[]
}

model Pembelajaran {
  id           String   @id @default(uuid())
  guru_id      String
  guru         GTK      @relation(fields: [guru_id], references: [id])
  rombel_id    String
  rombel       Rombel   @relation(fields: [rombel_id], references: [id])
  mapel        String
  jam_mengajar Int
  nilai        Nilai[]
}

model Nilai {
  id              String       @id @default(uuid())
  pd_id           String
  peserta_didik   PesertaDidik @relation(fields: [pd_id], references: [id])
  pembelajaran_id String
  pembelajaran    Pembelajaran @relation(fields: [pembelajaran_id], references: [id])
  jenis           JenisNilai
  nilai_angka     Int
  nilai_huruf     String
  semester        Int
}

model Absensi {
  id            String       @id @default(uuid())
  pd_id         String
  peserta_didik PesertaDidik @relation(fields: [pd_id], references: [id])
  tanggal       DateTime
  status        JenisPresensi
  keterangan    String?
}

model JurnalMengajar {
  id        String   @id @default(uuid())
  guru_id   String
  guru      GTK      @relation(fields: [guru_id], references: [id])
  tanggal   DateTime
  materi    String
  metode    String
  jam_ke    Int
  media     String
  penilaian String
}

model User {
  id            String   @id @default(uuid())
  username      String   @unique
  password_hash String
  role          Role
  is_active     Boolean  @default(true)
  gtk           GTK?
}

model AuditLog {
  id         String   @id @default(uuid())
  user_id    String
  aksi       String
  tabel      String
  record_id  String
  old_data   String?   @db.Text
  new_data   String?   @db.Text
  created_at DateTime @default(now())
}
```

---

## 🔒 Konfigurasi Keamanan Router & NextAuth (RBAC Middleware)

NextAuth digunakan untuk mengamankan kredensial (`CredentialsProvider`) menggunakan skema JWT:

```typescript
// auth.config.ts / middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { getToken } from 'next-auth/jwt';

export async function middleware(req: NextRequest) {
  const token = await getToken({ req, secret: process.env.NEXTAUTH_SECRET });
  const { pathname } = req.nextUrl;

  // Izinkan akses API Auth
  if (pathname.startsWith('/api/auth') || pathname === '/login') {
    if (token && pathname === '/login') {
      // Redirect ke dashboard yang sesuai jika sudah login
      return NextResponse.redirect(new URL(getDashboardRoute(token.role), req.url));
    }
    return NextResponse.next();
  }

  // Jika tidak terotentikasi, proteksi semua rute internal
  if (!token) {
    return NextResponse.redirect(new URL('/login', req.url));
  }

  // Proteksi Berbasis Peran (RBAC)
  if (pathname.startsWith('/admin') && token.role !== 'SUPER_ADMIN' && token.role !== 'OPERATOR_DAPODIK') {
    return NextResponse.rewrite(new URL('/403-unauthorized', req.url));
  }
  if (pathname.startsWith('/guru') && token.role !== 'GURU') {
    return NextResponse.rewrite(new URL('/403-unauthorized', req.url));
  }
  if (pathname.startsWith('/murid') && token.role !== 'MURID') {
    return NextResponse.rewrite(new URL('/403-unauthorized', req.url));
  }

  return NextResponse.next();
}

function getDashboardRoute(role: string) {
  if (role === 'SUPER_ADMIN' || role === 'OPERATOR_DAPODIK') return '/admin';
  if (role === 'GURU') return '/guru';
  return '/murid';
}
```

---

## 🏛️ Desain Estetis Professional ala Kemdikbud
- **Skema Warna:** Dominansi Biru Deep Navy (Kepercayaan, Kewibawaan, Standar Pemerintah) dipadu dengan putih bersih transparan dan abu-abu arsitektur.
- **Micro-Animations:** Transisi halus bertenaga menggunakan `motion` pada modal, tombol hover feedback, tab switching, dan alert banner.

*SMP Negeri 1 Bojongkenyot • Dinas Pendidikan Terpadu Indonesia • Maju Bersama Mencerdaskan Bangsa*
