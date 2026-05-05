# Laporan Akhir Insiden Keamanan Siber
### Kelompok 06 – Kelas G2 | TEK1314 Keamanan Siber

**Nama Skenario:** Remote Access Security  
**Tanggal Laporan:** Mei 2026  
**Disusun oleh:** Kelompok 06

---

## BAB I – Ringkasan Eksekutif

Pada proyek ini, kelompok kami membangun sebuah lingkungan jaringan simulasi yang terdiri dari tiga mesin virtual, yaitu Kali Linux sebagai node penyerang dengan IP 192.168.6.2, Ubuntu Server 22.04 sebagai server target SSH dengan IP 192.168.6.5, dan Security Onion sebagai sistem monitoring dengan IP 192.168.6.3. Ketiga node tersebut berada dalam satu subnet 192.168.6.0/24 yang berjalan di atas VirtualBox.

Insiden yang kami simulasikan adalah serangan brute force dan dictionary attack terhadap layanan SSH pada port 22 di server target. Serangan ini dilakukan dengan menggunakan tool hydra dari node penyerang, yang secara otomatis mencoba ribuan kombinasi username dan password dalam waktu singkat. Jenis serangan ini dipilih karena cukup umum terjadi di dunia nyata, terutama pada server yang menggunakan password lemah dan tidak memiliki mekanisme pembatasan percobaan login.

Hasil dari simulasi ini menunjukkan bahwa tanpa konfigurasi pengamanan yang memadai, serangan brute force dapat dengan mudah menghasilkan ribuan percobaan login yang tercatat di log server maupun di sistem monitoring Security Onion.

---

## BAB II – Deteksi dan Analisis

### 2.1 Metode Deteksi

Serangan berhasil terdeteksi melalui beberapa cara. Pertama, sistem Security Onion yang berjalan dalam mode promiscuous pada interface eth0 mendeteksi adanya lonjakan koneksi TCP ke port 22 yang tidak wajar dalam rentang waktu yang sangat singkat. Kedua, file log `/var/log/auth.log` pada server target menunjukkan banyak sekali baris yang berisi keterangan gagal login dari IP 192.168.6.2. Ketiga, dashboard Kibana menampilkan lonjakan event SSH yang terlihat jelas pada grafik timeline.

### 2.2 Analisis Log

Berikut adalah contoh pola log yang muncul pada server target saat serangan berlangsung:

```
Failed password for root from 192.168.6.2 port 54231 ssh2
Failed password for admin from 192.168.6.2 port 54232 ssh2
Failed password for ubuntu from 192.168.6.2 port 54233 ssh2
Invalid user test from 192.168.6.2 port 54234
Failed password for invalid user guest from 192.168.6.2 port 54235 ssh2
```

Dari pola log di atas dapat diketahui bahwa penyerang mencoba berbagai nama pengguna umum secara berurutan. Hal ini merupakan ciri khas dari dictionary attack, di mana daftar username dan password yang sering digunakan orang dicoba satu per satu secara otomatis. Volume percobaan yang sangat tinggi dalam waktu singkat juga menjadi indikator utama bahwa serangan ini dilakukan oleh tool otomatis, bukan secara manual.

*[Lampirkan screenshot dashboard Kibana/Sguil di sini]*

### 2.3 Analisis Paket dengan Wireshark

Dari hasil capture paket menggunakan Wireshark, terlihat adanya banyak sekali paket TCP SYN yang dikirimkan dari IP 192.168.6.2 ke IP 192.168.6.5 pada port 22. Setiap koneksi diikuti dengan proses SSH handshake dan percobaan autentikasi, kemudian langsung putus dan digantikan oleh koneksi baru. Pola seperti ini tidak mungkin terjadi pada penggunaan SSH yang normal.

*[Lampirkan screenshot Wireshark di sini]*

### 2.4 Kategori Insiden

Berdasarkan hasil analisis di atas, insiden ini dikategorikan sebagai **Unauthorized Access Attempt** dengan teknik **Brute Force Attack** (MITRE ATT&CK T1110.001 – Password Guessing).

---

## BAB III – Penahanan, Pembersihan, dan Pemulihan

### 3.1 Penahanan

Langkah pertama yang kami lakukan setelah serangan terdeteksi adalah memblokir IP penyerang menggunakan firewall ufw agar serangan tidak berlanjut:

```bash
sudo ufw deny from 192.168.6.2 to any port 22
```

Setelah itu kami me-restart layanan SSH untuk memutus seluruh sesi yang sedang aktif:

```bash
sudo systemctl restart ssh
```

Kami juga terus memantau log secara langsung untuk memastikan aktivitas mencurigakan sudah berhenti setelah langkah-langkah di atas dilakukan.

### 3.2 Pembersihan

Setelah serangan berhasil dihentikan, kami melakukan beberapa langkah untuk menutup celah yang dieksploitasi. Langkah utama adalah menginstal dan mengonfigurasi Fail2ban, yaitu tool yang secara otomatis memblokir IP yang terlalu sering gagal login:

```bash
sudo apt install fail2ban -y
```

Selain itu, kami juga menonaktifkan autentikasi berbasis password pada SSH dan menggantinya dengan autentikasi menggunakan SSH key, yang jauh lebih aman dan tidak bisa diserang dengan brute force:

```bash
# Perubahan di /etc/ssh/sshd_config
PasswordAuthentication no
PubkeyAuthentication yes
```

### 3.3 Pemulihan

Setelah semua langkah pembersihan selesai, kami melakukan verifikasi bahwa layanan SSH masih berjalan dengan baik menggunakan metode autentikasi yang baru. Sistem monitoring Security Onion juga dikonfirmasi masih aktif merekam traffic jaringan. Pada tahap ini sistem dinyatakan kembali beroperasi secara normal.

---

## BAB IV – Evaluasi Pasca Insiden

### 4.1 Pelajaran yang Didapat

Dari simulasi ini ada beberapa hal yang menjadi pelajaran penting bagi kelompok kami. Yang pertama adalah bahwa penggunaan password yang lemah merupakan celah yang sangat berbahaya, karena dictionary attack dapat menemukan password dalam waktu yang sangat singkat apabila password yang digunakan adalah kata-kata umum. Yang kedua, tanpa adanya mekanisme pembatasan percobaan login seperti Fail2ban, penyerang dapat bebas mencoba ribuan kombinasi tanpa hambatan apapun. Yang ketiga, keberadaan sistem monitoring seperti Security Onion terbukti sangat membantu dalam mendeteksi serangan secara dini sebelum penyerang berhasil masuk ke sistem.

### 4.2 Rekomendasi

Berdasarkan pengalaman dari simulasi ini, berikut beberapa rekomendasi teknis yang kami usulkan agar serangan serupa tidak berhasil di masa mendatang:

Pertama, aktifkan Fail2ban dengan konfigurasi yang ketat. Batas tiga kali percobaan gagal login sudah cukup untuk menghentikan sebagian besar serangan brute force otomatis.

Kedua, nonaktifkan autentikasi password pada SSH dan wajibkan penggunaan SSH key. Dengan cara ini, brute force terhadap password menjadi tidak relevan sama sekali.

Ketiga, pertimbangkan untuk memindahkan port SSH dari port default 22 ke port lain yang tidak umum. Langkah ini tidak akan menghentikan penyerang yang sudah menargetkan server secara spesifik, namun dapat mengurangi jumlah serangan acak secara signifikan.

Keempat, batasi IP yang diizinkan untuk melakukan koneksi SSH hanya dari jaringan yang dipercaya menggunakan aturan firewall yang spesifik.

---

*Laporan ini disusun sebagai bagian dari Proyek PBL mata kuliah TEK1314 Keamanan Siber.*  
*Kelompok 06 – Kelas G2 – Semester Genap 2025/2026*
