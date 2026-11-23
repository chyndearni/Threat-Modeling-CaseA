# Ringkasan Analisis – Threat Modeling STRIDE Kasus A

Sistem Presensi Perkuliahan Berbasis QR Code · Keamanan Siber (CAK3CAB3)

## 1. Aset dan pihak berkepentingan
| Aset | Pihak berkepentingan |
|---|---|
| Identitas pengguna & sesi autentikasi | Mahasiswa, dosen, admin, kampus |
| Keanggotaan kelas dan jadwal | Dosen, admin akademik |
| Sesi presensi dan token/kode QR | Dosen, mahasiswa |
| Catatan kehadiran dan rekap | Mahasiswa, dosen, admin akademik |

## 2. Asumsi
- **Asumsi A1:** Layanan autentikasi kampus (SSO) menerbitkan token ke browser dan Backend memverifikasinya ke layanan tersebut; Database Akademik hanya terjangkau dari zona layanan kampus.
- **Asumsi A2:** Komunikasi antara browser dan layanan kampus memakai HTTPS.
- **Asumsi A3:** QR code berisi ID sesi dan token acak, ditampilkan pada layar kelas sehingga dapat difoto.
- **Asumsi A4:** Peran dosen pengampu dan keanggotaan kelas dibaca dari Database Akademik; belum diketahui apakah otorisasi per kelas diperiksa pada setiap permintaan.
- **Asumsi A5:** Deskripsi tidak menyebut audit log; kelengkapannya belum diketahui, sehingga T05 bersifat kemungkinan, bukan fakta.

## 3. DFD
Elemen utama: 3 aktor (Mahasiswa, Dosen Pengampu, Admin Akademik), Browser/Aplikasi Web (process), Backend Presensi (process), Layanan Autentikasi Kampus (external entity), Database Akademik (data store).

Trust boundary: (1) **Zona Perangkat Pengguna**, (2) **Zona Layanan Kampus**, (3) **Zona Data Akademik**.

| ID | Data flow | Arah |
|---|---|---|
| F1 | Kredensial login + hasil scan QR | Mahasiswa → Browser |
| F2 | Hasil presensi | Browser → Mahasiswa |
| F3 | Perintah buka/tutup sesi, lihat daftar hadir | Dosen → Browser |
| F4 | QR code + daftar hadir | Browser → Dosen |
| F5 | Permintaan rekap | Admin → Browser |
| F6 | Rekap kehadiran | Browser → Admin |
| F7 | Kredensial login kampus | Browser → Layanan Autentikasi |
| F8 | Token autentikasi | Layanan Autentikasi → Browser |
| F9 | Permintaan presensi / perintah sesi / minta rekap (+ token) | Browser → Backend |
| F10 | QR dan status sesi, hasil validasi, daftar hadir, rekap | Backend → Browser |
| F11 | Verifikasi token autentikasi | Backend → Layanan Autentikasi |
| F12 | Identitas dan peran pengguna | Layanan Autentikasi → Backend |
| F13 | Simpan sesi dan catatan kehadiran, query rekap | Backend → Database |
| F14 | Keanggotaan kelas, jadwal, sesi, daftar hadir | Database → Backend |

## 4. Tabel ancaman dan penilaian risiko
Skor = Kemungkinan × Dampak (skala 1–3). 1–2 Rendah, 3–4 Sedang, 6–9 Tinggi.

| ID | Judul | Elemen DFD | STRIDE | K | D | Skor | Tingkat |
|---|---|---|---|---|---|---|---|
| T01 | Titip presensi memakai akun mahasiswa orang lain | Mahasiswa | Spoofing | 3 | 2 | 6 | Tinggi |
| T02 | Dosen membuka sesi untuk kelas yang bukan diampunya | Backend Presensi | Elevation of Privilege | 2 | 3 | 6 | Tinggi |
| T03 | Manipulasi parameter permintaan presensi dari browser | Browser → Backend (Permintaan presensi) | Tampering | 2 | 2 | 4 | Sedang |
| T04 | QR code difoto dan dibagikan ke mahasiswa di luar kelas | Backend → Browser (QR dan status sesi) | Information Disclosure | 3 | 2 | 6 | Tinggi |
| T05 | Penyangkalan buka/tutup sesi dan kehadiran karena jejak audit lemah | Backend Presensi | Repudiation | 2 | 2 | 4 | Sedang |
| T06 | Kebocoran data identitas dan kehadiran dari Database Akademik | Database Akademik | Information Disclosure | 1 | 3 | 3 | Sedang |
| T07 | Banjir permintaan pada Backend saat jendela presensi lima menit | Backend Presensi | Denial of Service | 2 | 2 | 4 | Sedang |
| T08 | Catatan hadir ganda akibat permintaan paralel | Backend → Database Akademik (Simpan catatan kehadiran) | Tampering | 2 | 2 | 4 | Sedang |

## 5. Detail ancaman, alasan nilai, dan mitigasi
### T01 – Titip presensi memakai akun mahasiswa orang lain
- **Elemen DFD:** Mahasiswa
- **Kategori STRIDE:** Spoofing
- **Skenario:** Jika mahasiswa dapat meminjam/memakai kredensial atau sesi login mahasiswa lain pada Mahasiswa (aktor), maka bisa terjadi catatan kehadiran palsu atas nama orang yang tidak hadir.
- **Dampak:** lihat skenario; penilaian dampak 2/3.
- **Alasan nilai:** Kemungkinan tinggi karena berbagi akun mudah dilakukan; dampak sedang karena merusak keabsahan data presensi tetapi tidak membocorkan data sensitif.
- **Skor:** 3 × 2 = 6 (Tinggi)
- **Mitigasi (rekomendasi, belum tentu diterapkan):** Autentikasi kuat lewat layanan autentikasi kampus (misal MFA); batasi satu sesi aktif per akun; catat IP/perangkat saat presensi untuk bahan audit dosen.

### T02 – Dosen membuka sesi untuk kelas yang bukan diampunya
- **Elemen DFD:** Backend Presensi
- **Kategori STRIDE:** Elevation of Privilege
- **Skenario:** Jika dosen (atau akun yang disalahgunakan) dapat mengubah ID kelas pada permintaan buka sesi ke Backend Presensi, maka bisa terjadi sesi dibuat untuk kelas lain dan melanggar aturan bisnis hanya dosen pengampu yang boleh membuka sesi.
- **Dampak:** lihat skenario; penilaian dampak 3/3.
- **Alasan nilai:** Kemungkinan sedang karena bergantung pada ada tidaknya pemeriksaan otorisasi per kelas (asumsi A4); dampak tinggi karena presensi kelas lain menjadi tidak sah.
- **Skor:** 2 × 3 = 6 (Tinggi)
- **Mitigasi (rekomendasi, belum tentu diterapkan):** Backend memeriksa hubungan dosen–kelas di Database Akademik pada setiap permintaan buka/tutup sesi; ID kelas tidak dipercaya dari klien; tolak dan catat percobaan yang gagal.

### T03 – Manipulasi parameter permintaan presensi dari browser
- **Elemen DFD:** Browser → Backend (Permintaan presensi)
- **Kategori STRIDE:** Tampering
- **Skenario:** Jika pengguna dapat mengubah isi permintaan (ID mahasiswa, ID sesi, atau waktu) pada aliran Browser ke Backend, maka bisa terjadi catatan hadir atas nama orang lain atau di sesi yang salah.
- **Dampak:** lihat skenario; penilaian dampak 2/3.
- **Alasan nilai:** Kemungkinan sedang karena alat developer browser tersedia bagi semua pengguna; dampak sedang karena terbatas pada data kehadiran.
- **Skor:** 2 × 2 = 4 (Sedang)
- **Mitigasi (rekomendasi, belum tentu diterapkan):** Identitas mahasiswa diambil dari token autentikasi yang diverifikasi server, bukan dari isi permintaan; validasi ID sesi dan waktu di server; gunakan HTTPS (asumsi A2).

### T04 – QR code difoto dan dibagikan ke mahasiswa di luar kelas
- **Elemen DFD:** Backend → Browser (QR dan status sesi)
- **Kategori STRIDE:** Information Disclosure
- **Skenario:** Jika pihak di dalam kelas dapat memotret atau menyalin QR code yang ditampilkan pada layar, lalu mengirimkannya ke mahasiswa yang tidak hadir, maka bisa terjadi presensi dari luar ruang kelas selama sesi masih berlaku.
- **Dampak:** lihat skenario; penilaian dampak 2/3.
- **Alasan nilai:** Kemungkinan tinggi karena QR tampil terbuka di layar kelas (asumsi A3); dampak sedang karena terbatas oleh masa berlaku lima menit.
- **Skor:** 3 × 2 = 6 (Tinggi)
- **Mitigasi (rekomendasi, belum tentu diterapkan):** Token QR acak dan diganti berkala selama sesi (rotasi singkat); masa berlaku sesi lima menit ditegakkan di server; dosen dapat menutup sesi lebih awal; opsional validasi lokasi/jaringan kampus.

### T05 – Penyangkalan buka/tutup sesi dan kehadiran karena jejak audit lemah
- **Elemen DFD:** Backend Presensi
- **Kategori STRIDE:** Repudiation
- **Skenario:** Jika dosen atau mahasiswa dapat menyangkal telah membuka/menutup sesi atau melakukan presensi karena log aktivitas tidak memadai pada Backend Presensi, maka bisa terjadi sengketa kehadiran yang tidak dapat dibuktikan.
- **Dampak:** lihat skenario; penilaian dampak 2/3.
- **Alasan nilai:** Kemungkinan sedang karena deskripsi tidak menyebut audit log (asumsi A5, belum tentu tidak ada); dampak sedang karena menyulitkan penyelesaian sengketa.
- **Skor:** 2 × 2 = 4 (Sedang)
- **Mitigasi (rekomendasi, belum tentu diterapkan):** Catat log audit (siapa, kapan, aksi, ID sesi/kelas) untuk buka/tutup sesi dan presensi; log hanya dapat ditambah, tidak diubah; sinkronkan waktu server.

### T06 – Kebocoran data identitas dan kehadiran dari Database Akademik
- **Elemen DFD:** Database Akademik
- **Kategori STRIDE:** Information Disclosure
- **Skenario:** Jika pihak tidak berwenang dapat membaca Database Akademik (akses langsung atau kredensial database bocor), maka bisa terjadi kebocoran identitas pengguna, keanggotaan kelas, dan catatan kehadiran.
- **Dampak:** lihat skenario; penilaian dampak 3/3.
- **Alasan nilai:** Kemungkinan rendah karena database berada di zona internal (asumsi A1); dampak tinggi karena memuat data pribadi banyak mahasiswa.
- **Skor:** 1 × 3 = 3 (Sedang)
- **Mitigasi (rekomendasi, belum tentu diterapkan):** Database hanya dapat diakses dari Backend Presensi; akun database dengan hak minimal; kredensial disimpan aman di luar kode; enkripsi data tersimpan jika tersedia.

### T07 – Banjir permintaan pada Backend saat jendela presensi lima menit
- **Elemen DFD:** Backend Presensi
- **Kategori STRIDE:** Denial of Service
- **Skenario:** Jika banyak permintaan (mahasiswa serentak atau skrip otomatis) dikirim ke Backend Presensi dalam jendela lima menit, maka bisa terjadi layanan lambat atau tidak tersedia dan sebagian mahasiswa gagal presensi tepat waktu.
- **Dampak:** lihat skenario; penilaian dampak 2/3.
- **Alasan nilai:** Kemungkinan sedang karena lonjakan di awal sesi lazim terjadi; dampak sedang karena mengganggu sebagian pengguna dan bersifat sementara.
- **Skor:** 2 × 2 = 4 (Sedang)
- **Mitigasi (rekomendasi, belum tentu diterapkan):** Rate limiting per akun dan per IP pada endpoint presensi; antrean atau penskalaan backend; indeks pada tabel sesi dan kehadiran; pemantauan beban.

### T08 – Catatan hadir ganda akibat permintaan paralel
- **Elemen DFD:** Backend → Database Akademik (Simpan catatan kehadiran)
- **Kategori STRIDE:** Tampering
- **Skenario:** Jika mahasiswa dapat mengirim beberapa permintaan presensi bersamaan untuk sesi yang sama, maka bisa terjadi lebih dari satu catatan hadir pada penyimpanan data dan melanggar aturan satu catatan per sesi.
- **Dampak:** lihat skenario; penilaian dampak 2/3.
- **Alasan nilai:** Kemungkinan sedang karena pengiriman ganda bisa terjadi bahkan tanpa niat jahat (klik berulang); dampak sedang karena merusak integritas rekap.
- **Skor:** 2 × 2 = 4 (Sedang)
- **Mitigasi (rekomendasi, belum tentu diterapkan):** Constraint UNIQUE (ID mahasiswa, ID sesi) di Database Akademik; operasi simpan bersifat atomik/idempoten; backend menolak duplikat dengan respons jelas.

Catatan: mitigasi di atas adalah rekomendasi kontrol, bukan kontrol yang sudah terbukti diterapkan.
