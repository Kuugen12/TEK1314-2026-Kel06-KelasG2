# Rules of Engagement (RoE)
## Kelompok 06 – Remote Access Security | TEK1314 Keamanan Siber

---

### 1. Identitas Kelompok

| Info           | Detail                             |
|----------------|------------------------------------|
| Kelompok       | 06 – Kelas G2                      |
| Skenario       | Remote Access Security (SSH)       |
| Mata Kuliah    | TEK1314 Keamanan Siber             |

---

### 2. Scope (Ruang Lingkup Serangan)

Serangan **HANYA** boleh dilakukan terhadap IP berikut:

| Target          | IP Address   | Keterangan              |
|-----------------|--------------|-------------------------|
| target-ssh      | 192.168.6.5  | VM milik kelompok 06    |

---

### 3. Yang DILARANG

- ❌ Melakukan scanning/attack ke jaringan kampus
- ❌ Melakukan scanning/attack ke Wi-Fi publik
- ❌ Menyerang IP di luar subnet `192.168.6.0/24`
- ❌ Menyerang VM kelompok lain
- ❌ Menyerang perangkat nyata (laptop dosen, server kampus, dll.)

---

### 4. Tools yang Diizinkan

- `hydra` / `medusa` – Brute Force SSH
- `nmap` – Vulnerability Scanning
- `wireshark` – Packet Capture & Analisis
- `Security Onion (Sguil/Kibana)` – Log Monitoring
- `metasploit` – Auxiliary scanner (opsional)

---

### 5. Anggota Kelompok

| Nama                              | NIM           | Peran                  | 
|-----------------------------------|---------------|------------------------|
| Muhammad Rafi Riza Pratama        | J0404231125   | Lead Analyst           | 
| Genta Fallah Munggaran Sonagar    | J0404231085   | System/Network Engineer & Attacker | 
| Muhammad Rifqi Annaufal           | J0404231060   | Security Analyst       | 
| Muhammad Eka Fauzan               | J0404221157   | Security Analyst       | 

