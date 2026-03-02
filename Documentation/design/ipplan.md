# IP Plan - Kelompok 06 | Remote Access Security

## Subnet: 192.168.6.0/24 | Subnet Mask: 255.255.255.0

| Hostname       | IP Address    | OS                  | Role          | Port Penting |
|----------------|---------------|---------------------|---------------|--------------|
| attacker-kali  | 192.168.6.100 | Kali Linux          | Attacker Node | -            |
| target-ssh     | 192.168.6.5   | Ubuntu Server 22.04 | Target (SSH)  | 22 (SSH)     |
| monitoring-so  | 192.168.6.10  | Security Onion      | SIEM / IDS    | 443 (UI)     |
| gateway        | 192.168.6.1   | Virtual Router      | Gateway       | -            |

## Port yang Dibutuhkan

| Port | Protokol | Layanan | Keterangan                              |
|------|----------|---------|-----------------------------------------|
| 22   | TCP      | SSH     | Port utama serangan Brute Force         |
| 514  | UDP      | Syslog  | Kirim log dari Target ke Security Onion |
| 443  | TCP      | HTTPS   | Dashboard Security Onion (Kibana)       |

## OS Target yang Dipilih
Ubuntu Server 22.04 LTS (CLI)