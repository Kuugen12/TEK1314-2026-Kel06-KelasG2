# Final Incident Response Report – Standar NIST
## Kelompok 06 – Remote Access Security | TEK1314 Keamanan Siber

**Tanggal Laporan:** *5 Mei 20226*  
**Dibuat oleh:** Muhammad Rafi Riza Pratama (Lead Analyst)  
**Skenario:** Remote Access Security – Brute Force & Dictionary Attack on SSH  

---

## BAB I: RINGKASAN EKSEKUTIF

Kelompok 06 membangun infrastruktur simulasi keamanan siber yang terdiri dari tiga node utama: **Kali Linux** sebagai attacker (`192.168.6.2`), **Ubuntu Server 22.04** sebagai target SSH (`192.168.6.5`), dan **Security Onion** sebagai sistem monitoring SIEM/IDS (`192.168.6.3`), semua terhubung dalam subnet `192.168.6.0/24`.

Insiden yang disimulasikan adalah serangan **Brute Force & Dictionary Attack** terhadap layanan SSH (Port 22). Penyerang menggunakan tool `hydra` untuk mencoba ribuan kombinasi password secara otomatis dalam waktu singkat. Serangan ini berhasil menghasilkan ribuan baris log pada server target dan memicu alert di sistem Security Onion, yang kemudian dianalisis menggunakan Sguil, Kibana, dan Wireshark.

**Jenis Insiden:** Brute Force Attack / Unauthorized Access Attempt  
**Aset Terdampak:** Management Server – SSH Port 22 (192.168.6.5)  
**Tingkat Keparahan:** HIGH  

---

## BAB II: FASE DETEKSI & ANALISIS

### 2.1 Metode Deteksi

Serangan terdeteksi melalui:
1. **Alert Sguil di Security Onion** – IDS mendeteksi lonjakan koneksi TCP ke port 22 yang tidak normal
2. **Log `/var/log/auth.log`** di target server – menunjukkan ribuan baris `Failed password for [user] from 192.168.6.2`
3. **Dashboard Kibana** – visualisasi lonjakan event SSH dalam rentang waktu singkat

### 2.2 Analisis Log

**Log dari `/var/log/auth.log` pada target (192.168.6.5):**

```
May XX 04:XX:XX target-ssh sshd[XXXX]: Failed password for root from 192.168.6.2 port XXXXX ssh2
May XX 04:XX:XX target-ssh sshd[XXXX]: Failed password for admin from 192.168.6.2 port XXXXX ssh2
May XX 04:XX:XX target-ssh sshd[XXXX]: Failed password for ubuntu from 192.168.6.2 port XXXXX ssh2
...
```

> *[Lampirkan screenshot dashboard SIEM / Kibana di sini]*

**Indikator Serangan:**
- **Source IP:** 192.168.6.2 (attacker-kali)
- **Destination IP:** 192.168.6.5 (target-ssh)
- **Port:** 22 (SSH)
- **Frekuensi:** Ratusan hingga ribuan percobaan per menit
- **Pattern:** Sequential password attempts dengan username yang berulang

### 2.3 Analisis Paket (Deep Packet Inspection – Wireshark)

> *[Lampirkan screenshot Wireshark yang menampilkan payload TCP handshake berulang ke port 22]*

**Temuan dari analisis paket:**
- Banyak koneksi TCP `SYN` dari `192.168.6.2` ke `192.168.6.5:22` dalam waktu singkat
- Setiap koneksi diikuti SSH handshake dan authentication attempt
- Pattern ini mengindikasikan automated brute force tool

### 2.4 Kategori Insiden

**Kategori:** Unauthorized Access Attempt – Brute Force Attack  
**Teknik (MITRE ATT&CK):** T1110.001 – Password Guessing

---

## BAB III: FASE PENAHANAN & PEMULIHAN

### 3.1 Penahanan (Containment)

Langkah cepat yang diambil untuk menghentikan serangan:

1. **Blokir IP penyerang** menggunakan `ufw`:
   ```bash
   sudo ufw deny from 192.168.6.2 to any port 22
   ```
2. **Restart layanan SSH** untuk memutus semua koneksi aktif:
   ```bash
   sudo systemctl restart ssh
   ```
3. **Pantau log secara real-time** untuk memastikan serangan berhenti:
   ```bash
   sudo tail -f /var/log/auth.log
   ```

### 3.2 Pembersihan (Eradication)

Langkah untuk menghapus celah keamanan yang dieksploitasi:

1. **Install dan konfigurasi Fail2ban:**
   ```bash
   sudo apt install fail2ban -y
   sudo systemctl enable fail2ban
   ```
   Konfigurasi `/etc/fail2ban/jail.local`:
   ```
   [sshd]
   enabled = true
   maxretry = 3
   bantime = 3600
   findtime = 600
   ```
2. **Perkuat kebijakan password** – gunakan password kompleks minimal 12 karakter
3. **Nonaktifkan password authentication, gunakan SSH key-based auth:**
   ```bash
   # Di /etc/ssh/sshd_config:
   PasswordAuthentication no
   PubkeyAuthentication yes
   ```

### 3.3 Pemulihan (Recovery)

1. Unblock IP setelah simulasi selesai dan sistem diverifikasi aman
2. Verifikasi SSH masih berjalan normal dengan koneksi menggunakan SSH key
3. Konfirmasi Security Onion masih merekam traffic dengan benar
4. **Status:** Sistem kembali berjalan normal ✅

---

## BAB IV: AKTIVITAS PASCA INSIDEN

### 4.1 Lesson Learned

1. **Password lemah adalah vektor serangan utama** – Dictionary attack berhasil menemukan password dalam waktu singkat apabila password tidak kompleks
2. **Fail2ban wajib diaktifkan** – tanpa rate limiting, brute force dapat berjalan tanpa hambatan
3. **SSH key authentication jauh lebih aman** daripada password authentication
4. **Monitoring real-time sangat penting** – Security Onion membantu mendeteksi serangan sebelum berhasil masuk

### 4.2 Rekomendasi Hardening

| Rekomendasi                        | Prioritas | Implementasi                                    |
|------------------------------------|-----------|-------------------------------------------------|
| Aktifkan Fail2ban                  | 🔴 HIGH   | `apt install fail2ban` + konfigurasi jail SSH  |
| Gunakan SSH Key Authentication     | 🔴 HIGH   | `PasswordAuthentication no` di sshd_config     |
| Ubah Port SSH dari default (22)    | 🟡 MEDIUM | Ganti ke port non-standar (misal: 2222)         |
| Implementasi MFA untuk SSH         | 🟡 MEDIUM | Install `libpam-google-authenticator`           |
| Batasi IP yang boleh SSH           | 🟡 MEDIUM | `ufw allow from [IP trusted] to any port 22`   |
| Pasang IDS/IPS (Security Onion)    | 🟢 LOW    | Sudah terpasang ✅                              |

---

*Laporan ini disusun sesuai dengan kerangka NIST SP 800-61 Rev. 2 – Computer Security Incident Handling Guide.*

*Kelompok 06 – TEK1314 Keamanan Siber – Kelas G2*
