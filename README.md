# Logbook Proyek PBL Keamanan Siber - Kelompok 06

**Mata Kuliah:** TEK1314 Keamanan Siber  
**Kelas:** G2  
**Skenario:** Remote Access Security – *Brute Force & Dictionary Attack on SSH (Port 22)*  
**Platform:** Kali Linux | Ubuntu Server 22.04 | Security Onion  

---

## Anggota Kelompok

| No | Nama                           | NIM           | Peran                           |
|----|--------------------------------|---------------|---------------------------------|
| 1  | Muhammad Rafi Riza Pratama     | J0404231125   | Lead Analyst                    |
| 2  | Genta Fallah Munggaran Sonagar | J0404231085   | System/Network Engineer & Attacker |
| 3  | Muhammad Rifqi Annaufal        | J0404231060   | Security Analyst                |
| 4  | Muhammad Eka Fauzan            | J0404221157   | Security Analyst                |

---

## Deskripsi Skenario

Kelompok 06 mensimulasikan serangan **Brute Force & Dictionary Attack** terhadap layanan **SSH (Port 22)** pada sebuah server Ubuntu. Penyerang menggunakan tools otomatis (Hydra/Medusa) untuk menebak kombinasi username dan password SSH sebanyak ribuan kali dalam satu menit. Aktivitas ini menghasilkan lonjakan log yang sangat terlihat di sistem SIEM (Security Onion), sehingga memudahkan proses deteksi dan analisis insiden.

**Target Aset:** Management Server – Service: SSH Port 22  
**Jenis Ancaman:** Brute Force Attack / Dictionary Attack  
**Dampak:** Unauthorized Access ke server jika password berhasil ditebak

---

## Infrastruktur

| Hostname      | IP Address   | OS                  | Role          |
|---------------|--------------|---------------------|---------------|
| attacker-kali | 192.168.6.2  | Kali Linux 2026.1   | Attacker Node |
| target-ssh    | 192.168.6.5  | Ubuntu Server 22.04 | Target (SSH)  |
| monitoring-so | 192.168.6.3  | Security Onion      | SIEM / IDS    |
| gateway       | 192.168.6.1  | Virtual Router      | Gateway       |

---

## Log Aktivitas Mingguan

### Minggu 1–3: Fase Setup & Design

**Target:** Instalasi VM, konfigurasi jaringan, dan perancangan topologi.

**Update:**
- Instalasi Kali Linux 2026.1 di VirtualBox → IP: `192.168.6.2`
- Instalasi Ubuntu Server 22.04 sebagai target SSH → IP: `192.168.6.5` (dikonfigurasi via `/etc/netplan/50-cloud-init.yaml`)
- Instalasi Security Onion sebagai SIEM/IDS → IP: `192.168.6.3`
- Semua VM dapat saling berkomunikasi (ping antar node berhasil)
- Perancangan IP Plan dan topologi jaringan selesai
- Repositori GitHub dibuat dan struktur folder dikonfigurasi

**Artefak:**
- [docs/design/ip_plan.md](docs/design/ip_plan.md)
- [docs/design/topology.png](docs/design/topology.png)

---

### Minggu 4–7: Fase Hardening & Baseline

**Target:** Penguatan OS target dan dokumentasi kondisi normal.

**Update:**
- Konfigurasi SSH hardening pada Ubuntu Server:
  - Menonaktifkan root login (`PermitRootLogin no`)
  - Membatasi jumlah percobaan login (`MaxAuthTries 3`)
  - Mengaktifkan logging verbose SSH
- Konfigurasi `ufw` (firewall) pada target: hanya port 22 yang dibuka dari subnet `192.168.6.0/24`
- Security Onion dikonfirmasi merekam traffic jaringan (ping dari attacker terdeteksi di Sguil)
- Baseline traffic normal direkam menggunakan Wireshark
- Security Baseline Document selesai disusun

---

### Minggu 9–11: Fase Serangan (Offensive)

**Target:** Simulasi serangan Brute Force SSH sesuai skenario kelompok 06.

**Update:**
- Tools yang digunakan: `hydra`, `nmap`
- Rencana: Jalankan `hydra -l [user] -P /usr/share/wordlists/rockyou.txt ssh://192.168.6.5`
- Verifikasi bahwa serangan terekam di Security Onion (Sguil alert & Kibana log)

**Artefak:**
- [Evidence-Logs/Phase2-Attack-Proofs/](Evidence-Logs/Phase2-Attack-Proofs/) *(akan diisi)*

---

### Minggu 12–15: Fase Defensive & Incident Response

**Target:** Analisis log, identifikasi serangan, dan penutupan celah keamanan.

**Update:**
- Analisis log di Security Onion (Sguil/Kibana)
- Deep Packet Inspection menggunakan Wireshark
- Penyusunan laporan akhir NIST Standard

**Artefak:**
- [Evidence-Logs/Phase3-Defense-Logs/](Evidence-Logs/Phase3-Defense-Logs/) *(akan diisi)*
- [Reports/Final-Report-NIST.pdf](Reports/Final-Report-NIST.pdf) *(akan diisi)*

---

## Struktur Repository

```
/Kelompok-06
├── /Documentation
│   ├── Network-Topology.png
│   ├── Asset-Inventory.csv
│   └── Rules-of-Engagement.md
├── /Evidence-Logs
│   ├── /Phase2-Attack-Proofs       ← Screenshots & log serangan
│   └── /Phase3-Defense-Logs        ← PCAP & JSON logs analisis
├── /Reports
│   └── Final-Report-NIST.pdf
└── /docs
    └── /design
        ├── topology.png
        └── ip_plan.md
```

