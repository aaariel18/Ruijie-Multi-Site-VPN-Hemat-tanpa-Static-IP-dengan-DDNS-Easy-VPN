# Multi-Site VPN Ruijie/Reyee Tanpa Harus Bergantung pada Static IP
### Dua pendekatan: Ruijie DDNS atau Domain Perusahaan + cPanel Dynamic DNS

> **Tujuan utama:** menghubungkan kantor pusat dan banyak cabang secara aman tanpa langsung berasumsi bahwa setiap lokasi harus berlangganan static public IP.

Kalau perusahaan memiliki banyak cabang, recurring cost kecil bisa menjadi besar ketika dikalikan puluhan lokasi dan 12 bulan.

Karena itu saya mencoba melihat kebutuhan VPN antar-site dari sudut yang sedikit berbeda:

> **Apakah kita benar-benar membutuhkan IP yang selalu tetap, atau sebenarnya kita hanya membutuhkan hostname yang selalu mengarah ke public IP terbaru?**

Pada Ruijie/Reyee, ada dua pendekatan yang menurut saya layak dipertimbangkan.

---

# Dua Pilihan DDNS / Two Practical Options

## Opsi A — Ruijie/Reyee DDNS

Pendekatan paling sederhana.

~~~text
Dynamic Public IP
       |
       v
Ruijie/Reyee Gateway
       |
       v
Ruijie DDNS
       |
       v
hostname.ruijieddns.com
       |
       v
Easy VPN / IPsec
~~~

Ruijie Cloud menyediakan fitur DDNS yang dapat memetakan hostname ke public IP gateway.

![Ruijie DDNS Configuration](https://community.ruijie.com/data/attachment/forum/202307/04/161945kl0w72ggt79mhp2l.png)

**Source:** [Ruijie Community - How to configure Ruijie DDNS on Ruijie Cloud](https://community.ruijienetworks.com/forum.php?mod=viewthread&tid=5846)

### Kapan saya memilih opsi ini?

- deployment ingin cepat;
- tidak perlu custom hostname perusahaan;
- semua site memang menggunakan ekosistem Ruijie/Reyee;
- tim IT ingin konfigurasi yang sederhana.

---

## Opsi B — Domain Perusahaan + cPanel Dynamic DNS

Kalau perusahaan sudah mempunyai domain sendiri, endpoint VPN bisa dibuat lebih mudah dikenali.

Contoh:

~~~text
vpn-hq.company.co.id
vpn-dc.company.co.id
vpn-warehouse.company.co.id
~~~

Arsitekturnya:

~~~text
ISP
Dynamic Public IP
       |
       v
Ruijie/Reyee Gateway
       |
       +-------------------------+
                                 |
                     cPanel Dynamic DNS
                                 |
                                 v
                     vpn-hq.company.co.id
                                 |
                                 v
                        Branch Reyee EG
                     Peer Gateway = Domain
~~~

Biznet Gio menjelaskan bahwa fitur **Dynamic DNS pada cPanel** mengaitkan public IP yang berubah dengan hostname tetap. Setelah hostname dibuat, cPanel menyediakan **webcall URL**; ketika URL itu diakses dari jaringan terkait, public IP terdeteksi dan DNS record diperbarui.

**Reference:** [Biznet Gio - Cara Membuat DDNS di cPanel](https://kb.biznetgio.com/id_ID/informasi/cara-membuat-ddns-di-cpanel)

Contoh interface Dynamic DNS pada cPanel:

![cPanel Dynamic DNS Interface](https://blog.cpanel.com/wp-content/uploads/2020/12/01-cpanel-create-dynamic-dns.png)

**Source:** [cPanel - How to Host Dynamic DNS Domains](https://www.cpanel.net/blog/tips-and-tricks/how-to-host-dynamic-dns-domains-with-cpanel/)

### Cara kerjanya

1. Buat hostname di cPanel, misalnya:

~~~text
vpn-hq.company.co.id
~~~

2. cPanel menghasilkan webcall URL.

3. Jalankan webcall tersebut secara periodik dari host di jaringan HQ, misalnya server Linux, automation host, NAS, atau perangkat lain yang memang bisa melakukan HTTP request.

4. Ketika ISP mengganti public IP, webcall berikutnya memperbarui DNS record.

5. Di branch Reyee, gunakan:

~~~text
Peer Gateway = vpn-hq.company.co.id
~~~

Dokumentasi Reyee EG menyebut bahwa pada IPsec client, **Peer Gateway dapat berupa public IP HQ atau domain**.

**Source:** [Reyee EG PoC Guide - IPsec VPN](https://reyee.ruijie.com/en-global/support/documents/slide_76717/)

---

# Jadi Pilih Mana?

| Kebutuhan | Ruijie DDNS | Domain Perusahaan + cPanel DDNS |
|---|---:|---:|
| Cepat dikonfigurasi | ✅ | ✅ |
| Tidak perlu membeli static IP hanya karena alamat berubah | ✅* | ✅* |
| Hostname milik perusahaan | - | ✅ |
| Mudah dikenali dokumentasi internal | Cukup | ✅ |
| Ketergantungan hostname vendor | Ada | Tidak |
| Cocok untuk PoC cepat | ✅ | ✅ |
| Cocok untuk branding internal / enterprise naming | Cukup | ✅ |

`*` Tetap bergantung pada kondisi ISP dan reachability public IP. DDNS tidak membypass CGNAT.

Bagi saya, **Ruijie DDNS cocok untuk deployment cepat**, sedangkan **custom company domain cocok ketika network sudah mulai dikelola sebagai infrastructure jangka panjang**.

---

# Executive Summary untuk Management

Skenario tradisional:

~~~text
HQ
Static Public IP
   |
   +--------- VPN -------- Branch 01
   |                      Static IP
   |
   +--------- VPN -------- Branch 02
   |                      Static IP
   |
   +--------- VPN -------- Branch 03
                          Static IP
~~~

Kalau provider mengenakan biaya static IP per site, recurring cost bertambah bersama jumlah cabang.

Alternatif yang dapat diuji:

~~~text
Internet Existing
+
Dynamic Public IP
+
Ruijie/Reyee Gateway
+
DDNS
+
IPsec / Easy VPN
~~~

DDNS membuat peer menggunakan **hostname**, bukan bergantung pada angka public IP yang selalu sama.

### Business Question

> **Apakah static IP benar-benar menjadi kebutuhan teknis, atau hanya digunakan karena kita membutuhkan alamat endpoint yang konsisten?**

Kalau kebutuhan sebenarnya adalah endpoint yang tetap dapat ditemukan, DDNS dapat menjadi salah satu opsi untuk mengurangi biaya berulang.

---

# Bagaimana Ruijie/Reyee Membantu?

Ruijie/Reyee EG dapat digunakan sebagai IPsec server maupun client.

Pada contoh resmi Reyee:

- HQ berfungsi sebagai IPsec server;
- branch berfungsi sebagai IPsec client;
- Peer Gateway di branch dapat berupa **public IP atau domain**;
- jika IPsec server berada di belakang perangkat NAT, UDP **500 dan 4500** perlu dipetakan.

Artinya skenario ini memungkinkan:

~~~text
Branch
   |
   | VPN initiated
   v
vpn-hq.company.co.id
   |
   v
Current Dynamic Public IP
   |
   v
HQ Ruijie/Reyee Gateway
~~~

**Source:** [Reyee EG PoC Guide](https://reyee.ruijie.com/en-global/support/documents/slide_76717/)

---

# Multi-Branch / Retail Architecture

~~~mermaid
flowchart TB
    CLOUD["Ruijie Cloud<br/>Central Management"]

    HQ["HEAD OFFICE<br/>Ruijie/Reyee EG<br/>Dynamic Public IP"]
    DDNS["DDNS Endpoint<br/>Ruijie DDNS OR Company Domain"]
    APP["Internal Apps / Monitoring / Server"]

    S1["STORE 01<br/>Reyee EG<br/>POS - CCTV - Staff"]
    S2["STORE 02<br/>Reyee EG<br/>POS - CCTV - Staff"]
    S3["STORE 03<br/>Reyee EG<br/>POS - CCTV - Staff"]

    CLOUD -. Management .-> HQ
    CLOUD -. Management .-> S1
    CLOUD -. Management .-> S2
    CLOUD -. Management .-> S3

    DDNS --> HQ
    S1 -->|IPsec VPN| DDNS
    S2 -->|IPsec VPN| DDNS
    S3 -->|IPsec VPN| DDNS

    HQ --> APP
~~~

Ruijie/Reyee juga mendokumentasikan Easy VPN untuk skenario retail chain dan centralized CCTV monitoring.

![Easy VPN - Official Ruijie/Reyee Documentation](https://eo-sgp-cos.ruijie.com/background/other/2024-08-14/90592fcb3496462987b59849fdaa23dd.png)

**Source:** [Ruijie Reyee - Chain Store CCTV Solution](https://reyee.ruijie.com/id-id/blog/cctv-chain-store-solution/)

---

# Kenapa Ini Menarik untuk Retail & Multi-Site?

Satu cabang biasanya bukan hanya mempunyai satu PC.

Di dalam satu site bisa ada:

~~~text
POS
CCTV / NVR
Office PC
Printer
Wi-Fi Staff
Guest Wi-Fi
VoIP
Internal Application
Remote IT Support
~~~

Gateway menjadi lebih bernilai kalau tidak hanya berfungsi sebagai Internet router.

Pada solusi retail Ruijie/Reyee, gateway juga digunakan untuk VPN, centralized/remote maintenance, dual WAN dan kebutuhan jaringan cabang.

![Retail Branch Solution](https://reyee.ruijie.com/id-id/solutions/smb/retailchain/image/page5-img.png)

**Source:** [Ruijie Reyee - Retail & Branch Network Solution](https://reyee.ruijie.com/id-id/solutions/smb/retailchain/)

---

# Cost-Saving Thinking

Saya sengaja tidak memasukkan angka provider tertentu karena harga setiap ISP berbeda.

Management cukup memasukkan biaya aktual.

## Skenario Static IP

~~~text
Annual Static-IP Cost
=
Jumlah Site
x
Monthly Static-IP Fee
x
12
~~~

Contoh cara berpikir:

~~~text
20 Cabang
x
Biaya Static IP / bulan
x
12
=
Recurring Cost / tahun
~~~

## Skenario Dynamic Public IP + DDNS

~~~text
Existing Internet
+
Gateway Investment
+
DDNS
+
Operational Maintenance
~~~

Yang dibandingkan bukan sekadar:

> Harga Router vs Harga Static IP satu bulan.

Tetapi:

> **CAPEX perangkat dibanding recurring OPEX selama beberapa tahun dan beberapa puluh site.**

---

# Dynamic Public IP Bukan CGNAT

Ini bagian terpenting sebelum proposal dibawa ke procurement.

DDNS hanya membantu ketika public IP berubah.

DDNS **tidak membuat private/CGNAT address menjadi public IP**.

Contoh alamat yang perlu dicurigai:

~~~text
10.x.x.x
172.16.x.x - 172.31.x.x
192.168.x.x
100.64.x.x - 100.127.x.x
~~~

### Decision Table

| Kondisi Internet | Hasil |
|---|---|
| Static Public IP | ✅ VPN mudah dibangun |
| Dynamic Public IP | ✅ Sangat cocok untuk DDNS |
| HQ Dynamic Public IP + Branch Dynamic | ✅ Layak diuji |
| HQ di belakang modem NAT + port forwarding | ✅ Dapat disesuaikan |
| ISP CGNAT tanpa inbound mapping | ⚠️ DDNS saja tidak cukup |

Jadi pesan yang tepat bukan:

> **"Dengan Ruijie kita tidak butuh public IP."**

Tetapi:

> **"Kalau ISP sudah memberi dynamic public IP yang reachable, kita belum tentu perlu membayar upgrade static IP hanya untuk menjaga alamat VPN tetap konsisten."**

---

# Dua Contoh Implementasi

## Scenario A — Native Ruijie DDNS

~~~text
HQ ISP
Dynamic Public IP
      |
      v
Ruijie EG
      |
Ruijie Cloud DDNS
      |
      v
hq-example.ruijieddns.com
      |
      +---- Branch 01
      +---- Branch 02
      +---- Branch 03
~~~

### Kelebihan

- sederhana;
- minim komponen tambahan;
- cocok untuk quick deployment;
- mudah dijadikan PoC.

---

## Scenario B — Company Domain

~~~text
HQ ISP
Dynamic Public IP
      |
      v
Ruijie EG
      |
      +----- Local automation/webcall
      |
      v
cPanel Dynamic DNS
      |
      v
vpn-hq.company.co.id
      |
      +---- Branch 01
      +---- Branch 02
      +---- Branch 03
~~~

### Kelebihan

- hostname sesuai identitas perusahaan;
- lebih mudah dibaca pada dokumentasi;
- bisa menggunakan naming convention internal;
- tidak bergantung pada nama domain DDNS vendor.

Contoh naming:

~~~text
vpn-hq.company.co.id
vpn-jkt01.company.co.id
vpn-tgr01.company.co.id
vpn-warehouse.company.co.id
~~~

---

# Network Segmentation Tetap Penting

VPN antar-cabang bukan berarti semua perangkat boleh saling mengakses.

Contoh sederhana:

~~~text
VLAN 10 - POS
VLAN 20 - Staff
VLAN 30 - CCTV
VLAN 40 - Guest Wi-Fi
VLAN 50 - Management
~~~

Kemudian firewall policy menentukan traffic mana yang memang diperlukan.

Contoh:

~~~text
POS -> Application Server        ALLOW
CCTV -> Monitoring Server       ALLOW
Guest -> Internal Network       DENY
Management -> Network Device    ALLOW
~~~

**Connectivity without access control is not a security strategy.**

---

# Proof of Concept yang Saya Sarankan

Sebelum procurement massal:

~~~text
1 HQ
+
2 Branch
~~~

Lakukan test nyata.

## Test 1 — Dynamic IP Change

~~~text
Public IP changes
      |
DDNS record updates
      |
VPN reconnects
      |
Application reachable
~~~

## Test 2 — DNS Recovery Time

Catat berapa lama hostname mulai mengarah ke IP baru.

Ini penting terutama jika menggunakan custom DNS karena TTL dan update path dapat memengaruhi recovery time.

## Test 3 — POS / Internal Application

Pastikan branch dapat mengakses hanya resource yang diperlukan.

## Test 4 — CCTV / Monitoring

Pastikan kebutuhan bandwidth dan latency masih sesuai.

## Test 5 — ISP Failover

Jika gateway menggunakan dual WAN:

~~~text
WAN 1 Down
   |
WAN 2 Active
   |
DDNS / VPN Recovery
   |
Business Service Available
~~~

## Test 6 — Remote Troubleshooting

Pastikan tim IT benar-benar dapat melakukan diagnosis tanpa selalu datang ke site.

---

# Checklist Sebelum Implementasi

- [ ] cek IP WAN yang diterima router;
- [ ] bandingkan dengan public IP Internet;
- [ ] identifikasi apakah ISP memakai CGNAT;
- [ ] tentukan DDNS: Ruijie atau company domain;
- [ ] jika custom domain, siapkan cPanel Dynamic DNS/webcall;
- [ ] tentukan subnet unik per cabang;
- [ ] hindari overlapping subnet;
- [ ] tentukan HQ / VPN hub;
- [ ] buat VLAN POS, Staff, CCTV, Guest, Management;
- [ ] siapkan firewall policy;
- [ ] cek UDP 500/4500 jika endpoint IPsec berada di belakang NAT;
- [ ] test perubahan public IP;
- [ ] test VPN reconnect;
- [ ] test application;
- [ ] dokumentasikan hasil PoC.

---

# Kesimpulan / Conclusion

Menurut saya, keputusan network multi-site sebaiknya tidak dimulai dari pertanyaan:

> **"Provider mana yang harus kita upgrade ke static IP?"**

Pertanyaan yang lebih tepat:

> **"Apa kebutuhan teknis kita, dan apakah dynamic public IP + DDNS sudah cukup untuk memenuhi kebutuhan itu?"**

Ada dua jalur yang sama-sama valid:

~~~text
OPTION A
Ruijie/Reyee DDNS
-> sederhana dan cepat

OPTION B
Company Domain + cPanel DDNS
-> lebih profesional dan fleksibel
~~~

Keduanya dapat digunakan bersama Ruijie/Reyee IPsec selama kondisi ISP dan network design memenuhi persyaratan.

Kalau PoC membuktikan bahwa:

- DDNS update stabil;
- VPN reconnect otomatis;
- application tetap reachable;
- security policy berjalan;
- remote management efektif;

maka perusahaan memiliki dasar teknis yang lebih kuat untuk memutuskan apakah biaya static IP di setiap cabang memang masih diperlukan.

> **Use static IP because the business needs it - not simply because the VPN needs a name that stays the same.**

---

# Official Documentation & References

1. [Reyee EG PoC Guide - IPsec VPN](https://reyee.ruijie.com/en-global/support/documents/slide_76717/)
2. [Ruijie Community - Configure Ruijie DDNS](https://community.ruijienetworks.com/forum.php?mod=viewthread&tid=5846)
3. [Ruijie Reyee - Chain Store CCTV Solution](https://reyee.ruijie.com/id-id/blog/cctv-chain-store-solution/)
4. [Ruijie Reyee - Retail & Branch Network Solution](https://reyee.ruijie.com/id-id/solutions/smb/retailchain/)
5. [Biznet Gio - Cara Membuat DDNS di cPanel](https://kb.biznetgio.com/id_ID/informasi/cara-membuat-ddns-di-cpanel)
6. [cPanel - How to Host Dynamic DNS Domains](https://www.cpanel.net/blog/tips-and-tricks/how-to-host-dynamic-dns-domains-with-cpanel/)

---

## Author

**Muhamad Fahrul**

- GitHub: [github.com/aaariel18](https://github.com/aaariel18)
- LinkedIn: [Muhamad Fahrul](https://www.linkedin.com/in/muhamad-fahrul-428948434/)

> Tulisan ini merupakan technical note dan ide arsitektur. Implementasi production tetap harus melalui assessment ISP, security review, capacity planning, serta Proof of Concept.
