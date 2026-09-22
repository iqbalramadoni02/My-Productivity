# My Productivity Hub

Dashboard pribadi untuk mengelola **sekolah & kinerja mengajar**, **project & klien**, **tutor class**, **keuangan pribadi & pinjaman**, **jurnal & habit**, serta **arsip file** — dengan database **Google Sheets**, backend **Google Apps Script**, sinkron otomatis ke **Google Calendar**, kirim kwitansi/invoice lewat **Gmail**, dan penyimpanan file di **Google Drive**.

Arsitektur: **Google Sheets = database → Google Apps Script = backend → HTML = tampilan.** HTML disajikan langsung oleh Apps Script (satu URL `/exec`, tanpa hosting lain). Bila ingin, file HTML yang sama juga bisa di-hosting di Netlify / GitHub Pages.

```
my-productivity-hub/
├── gas/                 ← UTAMA: semua file untuk editor Apps Script
│   ├── Code.gs          (backend: database Sheets, Calendar, Gmail, Drive)
│   ├── Index.html       (kerangka halaman)
│   ├── Styles.html      (tampilan / CSS)
│   ├── App.html         (logika aplikasi / JavaScript)
│   └── appsscript.json  (zona waktu Asia/Jakarta, pengaturan web app)
└── web/                 ← OPSIONAL: versi hosting Netlify / GitHub Pages
    ├── index.html, styles.css, app.js
```

---

## 1. Pasang di Google Sheets + Apps Script (±10 menit)

1. Buka **Google Sheets** → buat spreadsheet kosong, beri nama mis. `My Productivity Hub — Database`.
2. Menu **Extensions → Apps Script**.
3. **Code.gs** → hapus isinya, tempel seluruh isi `gas/Code.gs`.
4. Buat 3 file HTML: klik **＋ → HTML**, beri nama persis **`Index`**, **`Styles`**, **`App`** (tanpa `.html`), lalu tempel isi file yang sesuai dari folder `gas/` (hapus dulu isi bawaan).
5. ⚙️ **Project Settings** → centang **Show "appsscript.json" manifest file in editor** → buka `appsscript.json`, ganti isinya dengan `gas/appsscript.json`.
6. Kembali ke **Editor**, pilih fungsi **`setup`** → **Run**.
   - "Authorization required" → **Review permissions** → pilih akun Anda.
   - Jika muncul "Google hasn't verified this app": **Advanced → Go to (nama project) (unsafe) → Allow**. Normal untuk script milik sendiri.
7. Buka **Execution log** → salin **API KEY** (`mph_…`). Semua sheet otomatis dibuat di spreadsheet.
8. **Deploy → New deployment** → ⚙️ **Web app**:
   - Execute as: **Me**
   - Who has access: **Anyone** (agar portal klien bisa dibuka tanpa login; data tetap dikunci API key)
   - **Deploy** → salin **Web app URL** (berakhiran `/exec`).
9. Buka URL tersebut → aplikasi tampil → masukkan **API key** sekali di tiap perangkat. Selesai.

> Simpan URL `/exec` sebagai bookmark atau "Add to Home screen" di HP.  
> Lupa API key? Di Spreadsheet: menu **My Productivity Hub → Tampilkan API key** (muat ulang spreadsheet dulu), atau jalankan `showApiKey` dari editor.

## 2. Pengaturan awal di aplikasi
1. **Pengaturan → Profil & Invoice**: email pribadi, nama usaha, rekening, awal/akhir semester.
2. **Pengaturan → Google Workspace** → **Pasang / perbarui** untuk email ringkasan harian.
3. Data lama di Excel: **Pengaturan → Data → Impor dari Excel** (lihat bagian 5).

## 3. (Opsional) Hosting HTML di Netlify / GitHub Pages
Backend tetap Apps Script yang sama; hanya tampilan yang dipindah ke domain lain.
- **Netlify:** seret folder `web` ke <https://app.netlify.com/drop>.
- **GitHub Pages:** upload isi folder `web` ke repository → **Settings → Pages** → branch `main` / `(root)`.
- Di aplikasi: **Pengaturan → Koneksi** → mode *Google Sheets*, isi **URL /exec** dan **API key** → **Simpan & hubungkan**.

Kode tidak berisi data maupun API key, jadi aman walau repository publik.

## 4. Cara kerja integrasi

| Kejadian di aplikasi | Otomatis terjadi |
|---|---|
| DP / pelunasan project dicatat | Pemasukan masuk Buku Kas (sumber *Project*), kwitansi PDF bisa langsung dikirim ke Gmail klien |
| Honor sesi bimbel dibayar | Pemasukan (sumber *Tutor*), sesi ditandai lunas |
| Gaji / honor / biaya bahan ajar di modul Sekolah | Masuk Buku Kas (sumber *Sekolah*) |
| Pinjaman baru dengan "dana sudah cair" | Pencairan tercatat sebagai pemasukan kas; jadwal cicilan dibuat |
| Klik **Bayar** cicilan | Pengeluaran tercatat, sisa pokok berkurang, status cicilan *Lunas* |
| Hapus transaksi otomatis | Status terkait dikembalikan (cicilan → Belum, sesi → belum dibayar) |
| Jadwal mengajar | Event mingguan berulang di Google Calendar sampai akhir semester |
| Agenda, deadline tugas, deadline project, sesi bimbel | Event Google Calendar + pengingat H-3, H-1, 1 jam |
| Jatuh tempo cicilan | Event + pengingat H-5, H-3, H-1 |
| Sesi bimbel (bila "undang klien" aktif) | Klien menerima undangan Google Calendar |
| Setiap pagi (trigger harian) | Email ringkasan: jadwal, sesi, agenda, deadline ≤3 hari, cicilan ≤5 hari, saldo |

Event dibuat di kalender terpisah bernama **My Productivity Hub** agar mudah disembunyikan/ditampilkan. File yang di-upload tersimpan di Drive, folder **My Productivity Hub — File Storage**, dipisah per modul.

**Portal klien:** di detail project → *Salin tautan*. Klien bisa melihat status, progres tahapan, dan sisa pembayaran tanpa login (hanya-baca, memakai token acak per project; bisa dibuat ulang kapan saja).

## 5. Impor dari Excel lama
Impor mengenali otomatis:
- **MY_TEACHING_MANAGEMENT**: sheet `Schedule` (grid Senin–Sabtu → jadwal mingguan, slot berurutan digabung), `Task Management` → Tugas & Ujian, `Appointment` dan `Deadline` → Agenda, `TP EW X / TP EL XI / …` → Tujuan Pembelajaran.
- **My_Personal_Finance**: `PROJECT 2026` dan `REKAP PROJECT` → Project + pembayaran DP/termin otomatis masuk Buku Kas.
- Tabel transaksi apa pun dengan kolom *Tanggal, Jumlah, Keterangan (Kategori, Jenis)*.
- File cadangan dari tombol **Ekspor semua**.

Pratinjau selalu ditampilkan sebelum impor; baris yang sudah ada tidak diimpor dua kali. Data lampau (tanggal sudah lewat) tidak dibuatkan event kalender agar kalender tetap bersih.

## 6. Tips & batasan
- **Update kode (Code.gs atau file HTML):** setelah mengubah, buka **Deploy → Manage deployments → ✏️ Edit → Version: New version → Deploy**. URL tetap sama.
- **Kuota Google (akun gratis):** ±100 email/hari lewat MailApp, ±5.000 event kalender/hari, eksekusi maks. 6 menit per permintaan. Sisa kuota email terlihat di Pengaturan.
- **Mode lokal** menyimpan data di browser (localStorage + IndexedDB) — cocok untuk mencoba. Setelah terhubung, gunakan **Pengaturan → Data → Kirim data lokal ke Google Sheets**.
- Mengedit langsung di Google Sheets boleh (mis. menambah baris); baris tanpa `id` otomatis diberi id saat aplikasi memuat data. Jangan ubah nama sheet atau judul kolom baris pertama.
- Pintasan: **Ctrl + K** untuk mencari klien, transaksi, pinjaman, file, atau menjalankan perintah.
- Ganti API key bila bocor: menu Spreadsheet **My Productivity Hub → Buat API key baru**, lalu perbarui di Pengaturan tiap perangkat.

## 7. Pemecahan masalah
| Pesan | Solusi |
|---|---|
| Halaman /exec kosong atau error "Index" | Pastikan nama file HTML persis `Index`, `Styles`, `App`, lalu deploy versi baru. |
| "Respons server bukan JSON" (versi Netlify) | Deployment belum "Anyone" atau memakai URL `/dev`. Buat deployment baru dengan akses *Anyone*, pakai URL `/exec`. |
| "API key salah atau kosong" | Salin ulang dari *Tampilkan API key*. |
| "Database belum disiapkan" | Jalankan `setup()` sekali dari editor. |
| Event kalender tidak muncul | Pastikan *Sinkron otomatis* aktif; klik **Sinkronkan ulang semua** di Pengaturan → Google Workspace. |
| Grafik / PDF / Excel tidak jalan | Butuh internet untuk memuat library dari cdnjs; muat ulang halaman. |
