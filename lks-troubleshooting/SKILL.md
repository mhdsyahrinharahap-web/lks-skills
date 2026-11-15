---
name: lks-troubleshooting
description: "Diagnosis masalah jaringan untuk latihan LKS/ITNSA: Cisco, MikroTik, Linux Debian, Windows Server, Ansible. Gunakan saat user menempelkan error, hasil perintah yang gagal, atau skenario jaringan rusak. Diagnosis berurutan, berdasarkan bukti dari output perintah, bukan tebakan."
---
# lks-troubleshooting

> Cari penyebab, bukan gejala. Setiap kesimpulan harus bersandar pada output perintah yang ditunjukkan user, dan setiap perbaikan harus dibuktikan berhasil.

## Aturan Utama

1. **Bukti dulu, kesimpulan kemudian.** Jangan menebak konfigurasi yang tidak ditampilkan. Kalau datanya belum ada, minta output perintah diagnostik yang spesifik, bukan seluruh konfigurasi.
2. **Satu perubahan sekali jalan.** Ubah satu hal, uji, lalu lanjut. Beberapa perubahan sekaligus membuat penyebab asli tak terlacak.
3. **Urut dari bawah ke atas.** Mulai dari fisik, naik ke aplikasi. Jangan mengutak-atik DNS sebelum ping ke IP berhasil.
4. **Nyatakan tingkat keyakinan.** Bedakan "pasti (terlihat di output)" dari "dugaan (perlu dicek)". Dugaan selalu disertai perintah untuk mengujinya.
5. **Jujur kalau tidak tahu.** Sintaks atau perilaku yang tidak pasti ditandai, dan diberi cara mengecek di perangkat.
6. **Jangan mengarang output.** Jangan menulis hasil perintah seolah sudah dijalankan; yang boleh hanya hasil yang *diharapkan*, diberi label jelas.

## Metode Berlapis

Periksa berurutan, dan berhenti di lapisan pertama yang gagal:

| Lapisan | Pertanyaan | Perintah umum |
|---|---|---|
| 1. Fisik / link | Interface hidup? Kabel atau adapter VM benar? | `show ip interface brief` (Cisco), `/interface print` (MikroTik), `ip link` (Linux) |
| 2. IP dan subnet | Alamat, mask, gateway benar? Satu subnet dengan tetangga? | `show ip interface brief`, `/ip address print`, `ip a`, `ipconfig /all` |
| 3. Routing | Ada rute ke tujuan **dan** rute balik? | `show ip route`, `/ip route print`, `ip route`, `route print` |
| 4. Layanan | Layanan berjalan dan listen di port yang benar? | `ss -tulpn`, `systemctl status`, `Get-Service` |
| 5. Firewall / ACL | Ada yang memblokir? | `show access-lists`, `/ip firewall filter print`, `nft list ruleset` / `iptables -L -n`, Windows Firewall |
| 6. Nama / DNS | Nama terselesaikan benar? | `dig`, `nslookup`, `resolvectl` |

Pola cepat: ping ke gateway berhasil tapi ke jaringan lain gagal berarti masalahnya di routing atau NAT; ping ke IP berhasil tapi ke nama gagal berarti masalahnya di DNS.

## Penyebab Umum (cek lebih dulu)

**Cisco**
- Interface masih `shutdown`, atau salah mask.
- OSPF: neighbor tidak naik karena network/area salah, hello/dead timer beda, atau interface passive.
- VLAN belum dibuat atau port bukan di VLAN yang benar; trunk tidak membawa VLAN itu.
- ACL: urutan salah, ada `deny any` implisit, atau dipasang di arah/interface yang salah.
- Konfigurasi tidak disimpan, hilang setelah reload.

**MikroTik**
- Rute default atau NAT masquerade tidak ada.
- Urutan aturan firewall salah (drop sebelum accept).
- Interface DHCP client belum dapat IP.
- Versi RouterOS beda (sintaks routing berbeda antara v6 dan v7), cek `/system resource print`.

**Linux / Debian**
- `ip_forward` belum aktif di router Linux.
- Layanan hanya listen di `127.0.0.1`, atau tidak di-`enable`.
- Perubahan di `/etc/network/interfaces` belum diterapkan.
- Bind9: serial belum naik, titik di akhir nama domain hilang, `named-checkzone` gagal, `allow-query` membatasi.
- Postfix: `inet_interfaces` hanya loopback, atau port 25 diblok firewall.
- Apache/Nginx: virtual host salah, port bentrok, izin file.

**Windows Server**
- Firewall memblokir ICMP (ping gagal padahal jaringan sehat).
- DNS client menunjuk ke DNS yang salah saat join domain.
- GPO belum ter-refresh atau terpasang di OU yang salah (`gpupdate /force`, `gpresult /r`).

**Ansible**
- SSH gagal: kunci/host key/user salah, cek `ansible all -m ping -vvv`.
- YAML: indentasi (spasi, bukan tab), titik dua, kutip.
- Task bukan idempotent atau handler tidak terpanggil.
- Repo atau paket tidak tersedia karena lab offline.

## Alur Menjawab

1. **Ringkas gejala** dalam satu kalimat dan sebut yang belum diketahui.
2. Jika data kurang: minta **maksimal 2-3 output perintah** yang paling menentukan, tulis perintahnya persis, dan jelaskan satu kalimat apa yang dicari.
3. Jika data cukup: sebutkan **penyebab paling mungkin** dengan bukti dari output, lalu perbaikannya dalam blok kode, lalu **perintah verifikasi** dan hasil yang diharapkan.
4. Jika ada beberapa kemungkinan: urutkan dari yang paling mungkin, beri cara membedakannya.
5. Akhiri dengan **pencegahan** satu kalimat (misalnya "simpan konfigurasi dengan `wr`").

Format singkat:

```
Gejala: ...
Penyebab (pasti/dugaan): ...
Bukti: <baris dari output user>
Perbaikan: <perintah>
Cek: <perintah> -> hasil yang diharapkan
Pencegahan: ...
```

## Mode Latihan (skenario rusak)

Jika user ingin berlatih (bukan mencari jawaban):

- Buat skenario dengan topologi singkat, tabel IP, dan **satu atau dua kesalahan** yang tersembunyi. Jangan membocorkan penyebabnya.
- Tampilkan gejala (misalnya "PC1 tidak bisa ping ke Server"), bukan jawabannya.
- Saat user bertanya, jawab dengan **output perintah yang masuk akal untuk skenario itu** dan beri label `contoh output skenario`, bukan klaim tentang perangkat nyata.
- Beri petunjuk bertahap bila user buntu; jawaban penuh hanya setelah user menyerah atau memintanya.
- Di akhir: sebutkan penyebab, cara menemukannya paling cepat, dan satu kebiasaan untuk mencegahnya.

## Cek Sebelum Menjawab

- [ ] Kesimpulan didukung output yang ditunjukkan user, atau jelas diberi label dugaan.
- [ ] Diagnosis berurutan dari lapisan bawah.
- [ ] Hanya satu perubahan per langkah, dengan perintah verifikasinya.
- [ ] Tidak ada output perintah karangan yang dipresentasikan sebagai nyata.
- [ ] Ketidakpastian ditandai beserta cara mengeceknya.
