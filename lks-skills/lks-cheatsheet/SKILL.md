---
name: lks-cheatsheet
description: "Membuat ringkasan perintah (cheatsheet) satu halaman untuk Cisco, MikroTik, Linux Debian, Windows Server, Ansible, dan layanan umum (Bind9, Apache, Postfix) untuk latihan LKS/ITNSA. Gunakan saat user minta cheatsheet, ringkasan perintah, atau kartu hafalan."
---
# lks-cheatsheet

> Ringkasan perintah yang akurat, padat, dan siap cetak. Lebih baik sedikit tapi benar daripada banyak tapi meragukan.

## Aturan Utama

1. **Hanya perintah yang pasti benar.** Jangan memasukkan perintah yang tidak yakin ada. Kalau ragu, tandai `(cek di perangkat)` atau tinggalkan.
2. **Sebut versi.** Cisco IOS, MikroTik RouterOS (v6 berbeda dari v7 untuk routing), Debian 12, Windows Server 2022. Beri catatan bila sintaks bergantung versi.
3. **Satu topik per cheatsheet.** Jangan menumpuk seluruh platform dalam satu lembar. Tanyakan fokusnya jika permintaan terlalu luas.
4. **Padat dan terstruktur.** Format tabel: Perintah | Fungsi | Catatan. Tanpa paragraf panjang.
5. **Jangan mengarang nilai.** Contoh memakai alamat jelas contoh (`192.168.1.0/24`, `10.0.0.0/30`) atau placeholder `<...>`, dan diberi label.

## Format Lembar

- Ukuran satu halaman A4, maksimal sekitar 40-60 baris.
- Judul: `<Platform> - <Topik> (versi)`.
- Kelompokkan: **Konfigurasi**, **Verifikasi**, **Simpan / Backup**, **Jebakan umum**.
- Perintah dalam font monospace. Gunakan kutip lurus dan tanda hubung biasa agar bisa disalin.
- Di bagian bawah: 3-5 **jebakan umum** untuk topik itu.

Jika user ingin PDF: susun sebagai dokumen siap cetak (Canvas/Docs bila ada) dan beri satu kalimat cara menyimpannya ke PDF. Jangan mengaku sudah membuat file PDF bila tidak benar-benar ada.

## Inti Per Platform

Gunakan ini sebagai kerangka. Perluas sesuai permintaan, dan tandai bila sintaks bergantung versi.

### Cisco IOS

| Perintah | Fungsi |
|---|---|
| `enable` / `configure terminal` | Masuk mode privileged / konfigurasi |
| `interface gi0/0` lalu `ip address <ip> <mask>` dan `no shutdown` | Beri IP dan aktifkan interface |
| `ip route <net> <mask> <next-hop>` | Rute statis |
| `router ospf 1` lalu `network <net> <wildcard> area 0` | OSPF |
| `show ip interface brief` | Status dan IP interface |
| `show ip route` | Tabel routing |
| `show ip ospf neighbor` | Status neighbor OSPF (cari `FULL`) |
| `show vlan brief` | Daftar VLAN |
| `show running-config` | Konfigurasi aktif |
| `copy running-config startup-config` | **Simpan** (atau `write memory`) |

Jebakan: lupa `no shutdown`, lupa menyimpan, wildcard mask tertukar dengan subnet mask.

### MikroTik RouterOS

| Perintah | Fungsi |
|---|---|
| `/ip address add address=<ip>/<prefix> interface=<if>` | Beri IP |
| `/ip route add dst-address=0.0.0.0/0 gateway=<gw>` | Rute default |
| `/ip firewall nat add chain=srcnat action=masquerade out-interface=<if>` | NAT masquerade |
| `/ip address print` / `/ip route print` | Verifikasi |
| `/ping <ip>` / `/tool traceroute <ip>` | Uji konektivitas |
| `/export file=<nama>` | Cadangan konfigurasi |

Jebakan: urutan aturan firewall (accept sebelum drop), bisa terkunci saat konfigurasi lewat jaringan, sintaks routing OSPF/BGP berbeda antara v6 dan v7.

### Linux Debian

| Perintah | Fungsi |
|---|---|
| `ip a` / `ip route` | Lihat IP dan rute |
| `systemctl status` / `restart` / `enable <layanan>` | Kelola layanan |
| `ss -tulpn` | Port yang sedang listen |
| `journalctl -u <layanan> -e` | Log layanan |
| `sysctl -p` | Terapkan `net.ipv4.ip_forward=1` dari `/etc/sysctl.conf` |
| `ping`, `curl`, `dig @<dns> <nama>` | Uji koneksi, web, DNS |
| `named-checkconf` / `named-checkzone <zona> <file>` | Cek sintaks Bind9 |
| `apachectl configtest` / `nginx -t` | Cek sintaks web server |
| `postconf -n` | Lihat konfigurasi Postfix |

Jebakan: layanan tidak di-`enable`, serial zona DNS tidak dinaikkan, layanan hanya listen di localhost.

### Windows Server (PowerShell)

| Perintah | Fungsi |
|---|---|
| `ipconfig /all` | Info IP lengkap |
| `Get-NetIPAddress` | Lihat alamat IP |
| `gpupdate /force` / `gpresult /r` | Terapkan dan buktikan GPO |
| `dcdiag` | Diagnosis domain controller |
| `Get-ADDomain` | Info domain |
| `Get-Service` | Daftar layanan |

Jebakan: Firewall memblokir ping, DNS client tidak menunjuk ke DC saat join domain.

### Ansible

| Perintah | Fungsi |
|---|---|
| `ansible all -m ping` | Uji koneksi ke semua host |
| `ansible-playbook <file> --syntax-check` | Cek sintaks |
| `ansible-playbook <file> --check` | Simulasi tanpa perubahan |
| `ansible-playbook <file>` | Jalankan |
| `ansible-vault encrypt <file>` | Enkripsi rahasia |

Aturan: jalankan dua kali, run kedua harus `changed=0` (idempotent); pakai `ansible.builtin.*`; restart layanan lewat handler.

## Alur Menjawab

1. Jika topik belum jelas, tanyakan satu hal: platform dan topik apa?
2. Buat cheatsheet dengan format di atas, hanya perintah yang yakin benar.
3. Tandai bagian yang bergantung versi.
4. Tutup dengan satu kalimat: bagian mana yang sebaiknya dicek langsung di perangkat atau lab.

## Cek Sebelum Menyerahkan

- [ ] Semua perintah yakin benar untuk versi yang disebut; yang ragu ditandai.
- [ ] Satu topik, muat satu halaman A4.
- [ ] Ada bagian Verifikasi dan Simpan/Backup, bukan hanya konfigurasi.
- [ ] Nilai contoh jelas berlabel contoh atau placeholder.
- [ ] Kutip lurus dan monospace, sehingga bisa disalin.
