# Incident Response Playbook & Forensic Report - Operation Chimera
> simulasi investigasi insiden keamanan siber yang bertujuan untuk mengidentifikasi mekanisme serangan, menganalisis bukti digital, serta menyusun rekomendasi penanganan terhadap sistem yang mengalami kompromi keamanan. 
<br></br>
# Ringkasan
- [Tools](#tools)
- [Executive Summary](Executive-Summary)
- [Hasil Temuan](#hasil-temuan)
- [Rekomendasi Mitigasi](rekomendasi-mitigasi)
## Tools
- forensic_triage.py
- Operation_Chimera_Student_Pack
<br></br>
## Executive Summary
Insiden keamanan terjadi pada 30 Oktober 2026 yang menargetkan HR Portal internal PT Vinix Seven Aurum. Insiden dikategorikan sebagai Critical karena ditemukan indikasi pengambilalihan server melalui kerentanan Insecure Deserialization pada mekanisme autentikasi sesi.
Berdasarkan hasil analisis, penyerang diduga memanfaatkan penggunaan pickle.loads() terhadap cookie yang tidak terpercaya untuk memperoleh akses awal sebagai www-data (UID 33). Selanjutnya, penyerang memanfaatkan kerentanan PATH Hijacking pada program SUID backup_manager sehingga proses /tmp/tar dijalankan dengan hak akses root (UID 0).
Hasil pemeriksaan forensik juga menemukan dua anomali timestamp pada file chimera_memo.wav dan hr_portal.log. Selain itu, ditemukan data tersembunyi pada bagian akhir file chimera_memo.wav yang menghasilkan flag CHIMERA_GHOST_2026.
Dampak insiden berpotensi menyebabkan akses tidak sah terhadap data karyawan, informasi payroll, manipulasi sistem, serta gangguan terhadap integritas log. Oleh karena itu, diperlukan isolasi sistem, penghapusan komponen berbahaya, perbaikan kode aplikasi, dan peningkatan pemantauan keamanan.

## Hasil Temuan
Hasil pemindaian otomatis menggunakan forensic_triage.py pada artefak menunjukkan Chain of Custody dari tiap artefak di direktori Evidence, seperti nama file target, ukuran file, nomor inode, waktu akuisisi hingga panjang hash SHA-256. Dari hasil analisis artefak evidence/hr_portal.log diperoleh IP Address Attacker yakni 10.10.14.18 dengan User-Agent, Mozilla/5.0 (X11; Linux x84_64). Juga terdapat Compromised Service / Account: HR Portal/admin, terdapat percobaan autentikasi gagal pada pukul 09:03:51 UTC oleh penyerang. Terdapat juga Suspicious Files / Paths yakni [/bin/sh],[/usr/local/bin/backup_manager],[/tmp/tar],[/var/backups/chimera_memo.wav],[/var/log/hr_portal.log].

Terdapat kerentanan pada aplikasi web (auth_cookie_audit.py) seperti manipulasi cookie, kerentanan Insecure Deserialization, hingga penggunaan XOR sebagai mekanisme keamanan yang lemah. Juga terdapat kerentanan Privilege Escalation yang berdampak bagi hak akses, dimana Ketika penyerang yang telah memperoleh akses sebagai www-data dapat memanfaatkan binary SUID, Program berpotensi dapat menjalankan executable tidak tepercaya dari direktori /tmp, hingga berpotensi membuat penyerang dapat mengakses file sensitif, memodifikasi sistem, dan mengubah log untuk menyamarkan aktivitasnya.

Dari hasil pemindaian direktori evidence/ menggunakan perintah python3 forensic_triage.py --timestomp evidence untuk mengidentifikasi file yang stempel waktunya telah dimanipulasi (timestomping). Dari hasil pemindaian diperoleh 2 anomali dengan atribut mtime (waktu modifikasi), atime (waktu akses), dan ctime (waktu perubahan).

## Rekomendasi Mitigasi
1. Short-Term Containment
   - Isolasi Server
   - Pengamanan Akun
   - Pengamanan BUkti
2. Eradication Steps
   - Identifikasi Komponen Berbahaya
   - Penghapusan Akses Tidak Sah
   - Pemulihan SIstem
3. Code Remediation & Hardening
   - Menghentikan penggunaan pickle.loads() terhadap data cookie yang berasal dari pengguna
   - Menghapus hak akses SUID pada program backup_manager apabila tidak diperlukan.
   - Menerapkan pemantauan integritas file untuk mendeteksi perubahan pada file sistem, konfigurasi aplikasi, log, dan direktori backup. 
