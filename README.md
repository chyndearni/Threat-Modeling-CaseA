# Threat Modeling Berbasis STRIDE Menggunakan OWASP Threat Dragon – Kasus A

Tugas Keamanan Siber (CAK3CAB3). Studi kasus: **Sistem Presensi Perkuliahan Berbasis QR Code**.

## Deskripsi sistem
Layanan presensi berbasis web. Dosen pengampu membuka sesi, sistem menampilkan QR code, mahasiswa login dengan akun kampus lalu memindai QR, Backend Presensi memvalidasi sesi (berlaku 5 menit) dan status mahasiswa, lalu mencatat kehadiran di Database Akademik. Dosen memantau daftar hadir dan menutup sesi; admin akademik melihat rekap seluruh kelas.

## Ruang lingkup
Buka/tutup sesi, pemindaian QR, validasi kehadiran, pembacaan rekap. Pendaftaran mata kuliah dan perubahan nilai tidak dibahas.

## Aktor dan komponen
Aktor: Mahasiswa, Dosen Pengampu, Admin Akademik. Komponen: Browser/Aplikasi Web, Backend Presensi, Layanan Autentikasi Kampus, Database Akademik.

## Pembagian pekerjaan
[Isi nama anggota kelompok, owner, dan reviewer]

## Ringkasan hasil
8 ancaman (Spoofing 1, Tampering 2, Repudiation 1, Information Disclosure 2, Denial of Service 1, Elevation of Privilege 1): 3 berisiko Tinggi, 5 Sedang. 5 asumsi berlabel. Detail ada di `Dokumentasi/Ringkasan-Analisis.md`.

## Daftar file
- `ThreatDragonModels/Presensi-QR/Presensi-QR.json` – model Threat Dragon (DFD + 8 ancaman)
- `Report/` – tempat menyimpan PDF dari fitur Report Threat Dragon (**belum ada**, lihat catatan)
- `Dokumentasi/Ringkasan-Analisis.md` – aset, asumsi, DFD, tabel ancaman, risiko, mitigasi

## Catatan status
File JSON dibuat lewat skrip dan **belum dibuka di Threat Dragon**. Buka lewat Open Model, rapikan tata letak, lalu ekspor PDF lewat Report dan simpan sebagai `Report/Laporan-Threat-Modeling-Kasus-A.pdf`.
