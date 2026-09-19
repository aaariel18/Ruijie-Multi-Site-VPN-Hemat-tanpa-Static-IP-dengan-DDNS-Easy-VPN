# Mengurangi Biaya Static IP Antar Cabang dengan Ruijie/Reyee Easy VPN + DDNS
### A Practical Multi-Site Network Idea for Retail & Branch Offices

> **Ide utamanya sederhana:** untuk banyak cabang, kita tidak selalu harus membeli layanan **static public IP** di setiap lokasi hanya supaya jaringan antar-site bisa terhubung.  
> Jika ISP memberikan **dynamic public IP** yang tetap dapat diakses dari Internet, Ruijie/Reyee dapat memanfaatkan **DDNS + VPN** sehingga koneksi antar-cabang tetap berjalan meskipun alamat IP berubah.

Artikel ini saya tulis sebagai catatan teknis sekaligus ide efisiensi untuk environment **retail, branch office, warehouse, CCTV, POS, dan internal application**.

---

## Executive Summary / Ringkasan untuk Management

Saat perusahaan mulai memiliki banyak cabang, salah satu biaya yang sering ikut membesar adalah konektivitas antar-site.

Pendekatan tradisional biasanya seperti ini:

~~~text
HQ
Static Public IP
   |
   +--------- VPN --------- Branch 01
   |                       Static Public IP
   |
   +--------- VPN --------- Branch 02
   |                       Static Public IP
   |
   +--------- VPN --------- Branch 03
                           Static Public IP
~~~

Secara teknis ini mudah dipahami, tetapi kalau static IP dikenakan biaya bulanan oleh provider, maka biaya akan bertambah seiring bertambahnya jumlah cabang.

Alternatif yang menarik adalah:

~~~text
Internet biasa
+
Dynamic Public IP
+
Ruijie / Reyee Gateway
+
DDNS
+
Easy VPN / IPsec
~~~

Dengan desain ini, VPN tidak lagi bergantung pada angka IP yang harus selalu tetap. Peer dapat menggunakan **domain name** yang mengikuti perubahan dynamic public IP.

**Business idea:** jangan langsung membeli static IP untuk seluruh cabang. Cek dulu apakah koneksi existing mendapatkan **public dynamic IP** dan apakah Ruijie/Reyee gateway di lokasi tersebut dapat menjadi bagian dari desain Easy VPN/DDNS.

---

## Kenapa Static IP Menjadi Biaya yang Perlu Dievaluasi?

Static public IP memang nyaman.

Contohnya:

~~~text
HQ = 203.x.x.10
Branch A = 203.x.x.20
Branch B = 203.x.x.30
~~~

Alamat tidak berubah sehingga peer VPN mudah diarahkan.

Namun pada environment multi-branch, biaya bulanannya dapat menjadi recurring OPEX.

Formula sederhananya:

~~~text
Annual Static-IP OPEX
=
Jumlah Site
x
Biaya Static IP per Bulan
x
12
~~~

Semakin banyak cabang, semakin besar recurring cost.

Karena itu pertanyaan yang menurut saya layak diajukan sebelum upgrade layanan ISP adalah:

> **Apakah kita memang membutuhkan static IP di setiap site, atau sebenarnya hanya membutuhkan endpoint yang tetap bisa ditemukan?**

Di sinilah DDNS menjadi menarik.

---

# 1. Dynamic Public IP Bukan Berarti Tidak Bisa VPN

Dynamic public IP berarti alamat public dari ISP dapat berubah.

Misalnya hari ini:

~~~text
36.x.x.10
~~~

kemudian berubah menjadi:

~~~text
103.x.x.25
~~~

Kalau peer VPN dikonfigurasi menggunakan IP lama, tunnel bisa gagal.

Dengan **Dynamic DNS (DDNS)**, kita menggunakan nama domain:

~~~text
hq-company.ruijieddns.com
~~~

Ketika public IP berubah, mapping domain diperbarui.

Sehingga konsepnya menjadi:

~~~text
hq-company.ruijieddns.com
           |
           v
Current Public IP
           |
           v
Ruijie/Reyee Gateway
           |
           v
VPN Tunnel
~~~

Ruijie mendokumentasikan DDNS pada Ruijie Cloud dengan opsi **map to Public IP**.

![Ruijie DDNS Configuration](https://community.ruijie.com/data/attachment/forum/202307/04/161945kl0w72ggt79mhp2l.png)

**Source:** [Ruijie Community - How to configure Ruijie DDNS on Ruijie Cloud](https://community.ruijie.com/forum.php?mod=viewthread&tid=5846)

---

# 2. Ruijie/Reyee Mendukung Peer VPN Menggunakan IP atau Domain

Pada dokumentasi pengujian IPsec Reyee EG, branch dikonfigurasi sebagai client dan **Peer Gateway** dapat menggunakan:

~~~text
HQ Public IP
atau
HQ Domain Name
~~~

Artinya, ketika HQ menggunakan public IP dinamis, domain/DDNS dapat menjadi layer yang menjaga endpoint tetap dapat ditemukan.

Desain sederhana:

~~~mermaid
flowchart LR
    HQ["HQ - Ruijie/Reyee EG<br/>Dynamic Public IP + DDNS"]
    B1["Branch 01<br/>Reyee EG"]
    B2["Branch 02<br/>Reyee EG"]
    B3["Branch 03<br/>Reyee EG"]

    B1 -->|IPsec / Easy VPN| HQ
    B2 -->|IPsec / Easy VPN| HQ
    B3 -->|IPsec / Easy VPN| HQ
~~~

Pada banyak skenario, branch cukup menjadi pihak yang **menginisiasi tunnel** ke endpoint HQ.

---

# 3. Easy VPN untuk Multi-Branch

Ruijie/Reyee memang memposisikan Easy VPN untuk skenario seperti retail dan CCTV multi-cabang.

Dalam dokumentasi resmi mereka, koneksi VPN digunakan untuk menghubungkan cabang ke kantor pusat, sehingga monitoring dan resource dapat dikelola secara terpusat.

![Easy VPN - Official Ruijie/Reyee Documentation](https://eo-sgp-cos.ruijie.com/background/other/2024-08-14/90592fcb3496462987b59849fdaa23dd.png)

**Source:** [Ruijie Reyee - Network Solution in Chain Store CCTV Scenarios](https://reyee.ruijie.com/id-id/blog/cctv-chain-store-solution/)

Bagi saya, konsep ini tidak hanya relevan untuk CCTV.

VPN antar-site juga dapat digunakan untuk menghubungkan resource seperti:

- POS / Point of Sale;
- application server;
- internal web application;
- database service tertentu;
- NVR / CCTV monitoring;
- printer atau device management;
- VoIP;
- monitoring server;
- file/service internal;
- remote IT management.

Tentu akses tetap harus dibatasi dengan VLAN, firewall policy, dan prinsip **least privilege**.

---

# 4. Satu Gateway, Beberapa Fungsi

Hal yang membuat pendekatan ini menarik secara bisnis adalah gateway Reyee tidak hanya berfungsi sebagai VPN router.

Pada lini router mereka, Ruijie/Reyee juga menawarkan kemampuan seperti:

- routing;
- load balancing;
- multi-WAN;
- QoS;
- Easy VPN;
- NAT traversal;
- DDNS;
- cloud management;
- remote maintenance;
- VLAN;
- SD-WAN / multi-branch features pada model/fitur yang sesuai.

Jadi investasi perangkat tidak hanya menyelesaikan satu kebutuhan VPN saja.

Konsepnya:

~~~text
                 INTERNET
                    |
            +-------+-------+
            |  Reyee / EG   |
            +-------+-------+
                    |
       +------------+------------+
       |            |            |
      POS          CCTV         Staff
      VLAN         VLAN         VLAN
       |            |            |
       +------ Secure VPN -------+
                    |
                    v
                   HQ
~~~

Untuk retail, manfaatnya menjadi lebih luas:

**Connectivity + Security + VPN + Cloud Management + Remote Troubleshooting**

---

# 5. Gambaran Retail / Branch Solution

Ruijie/Reyee juga menyediakan solusi khusus untuk retail dan branch.

![Retail Branch Topology - Ruijie Reyee](https://reyee.ruijie.com/id-id/solutions/smb/retailchain/image/page5-img.png)

**Source:** [Ruijie Reyee - Retail & Branch Network Solution](https://reyee.ruijie.com/id-id/solutions/smb/retailchain/)

Di solusi retail mereka, gateway digunakan untuk melayani berbagai kebutuhan site seperti komputer, Wi-Fi, VoIP, printer, dan POS, sekaligus menyediakan remote/cloud management dan fitur VPN.

---

# 6. Apa Keuntungan untuk Perusahaan?

## A. Mengurangi Ketergantungan pada Static IP per Cabang

Ini adalah benefit yang paling menarik.

Kalau selama ini desainnya:

~~~text
Setiap Cabang
=
Internet
+
Static IP add-on
+
Router
~~~

kita bisa mengevaluasi:

~~~text
Setiap Cabang
=
Internet dengan Public Dynamic IP
+
Ruijie/Reyee Gateway
+
VPN/DDNS
~~~

Recurring cost static IP berpotensi dikurangi pada site-site yang secara teknis memenuhi syarat.

---

## B. Centralized Management

Admin tidak harus selalu datang ke cabang hanya untuk mengetahui kondisi router.

Ruijie Cloud memberikan visibility dan remote management untuk perangkat yang terhubung.

![Ruijie Cloud Gateway List](https://community.ruijie.com/data/attachment/forum/202307/04/161924oxkkc4r9faxs9wqm.png)

**Source:** [Ruijie Community - Ruijie DDNS Configuration](https://community.ruijie.com/forum.php?mod=viewthread&tid=5846)

Untuk tim IT dengan banyak cabang, ini penting karena biaya bukan hanya ISP.

Ada juga:

~~~text
Operational Cost
=
Travel
+
Troubleshooting Time
+
Downtime
+
Manpower
~~~

Remote visibility dapat membantu mengurangi sebagian operational overhead tersebut.

---

## C. Lebih Mudah Scale-Up

Ketika cabang baru dibuka:

~~~text
New Branch
   |
Internet
   |
Ruijie/Reyee Gateway
   |
VPN to HQ
   |
Central Resources
~~~

Secara desain, kita tidak perlu langsung mengubah strategi menjadi leased line atau static IP di semua lokasi.

Tambahkan site, definisikan policy, buat tunnel, lalu masukkan ke monitoring yang sama.

---

## D. Cocok untuk Retail

Pada toko, yang kita butuhkan bukan sekadar Internet.

Biasanya ada:

~~~text
POS
CCTV
Office PC
Printer
Wi-Fi Staff
Guest Wi-Fi
VoIP
Remote Support
~~~

Semua traffic tersebut tidak seharusnya bercampur tanpa kontrol.

Dengan gateway yang mendukung VLAN, VPN, QoS, multi-WAN dan cloud management, satu perangkat bisa menjadi pusat kontrol jaringan cabang.

---

# 7. Contoh Desain yang Saya Pilih

Untuk skenario multi-store:

~~~mermaid
flowchart TB
    CLOUD["Ruijie Cloud<br/>Central Management"]

    HQ["HEAD OFFICE<br/>Reyee EG Gateway<br/>Public Dynamic IP + DDNS"]
    APP["Application / Monitoring / Internal Resources"]

    S1["STORE 01<br/>Reyee EG<br/>POS - CCTV - Staff"]
    S2["STORE 02<br/>Reyee EG<br/>POS - CCTV - Staff"]
    S3["STORE 03<br/>Reyee EG<br/>POS - CCTV - Staff"]

    CLOUD -. Management .-> HQ
    CLOUD -. Management .-> S1
    CLOUD -. Management .-> S2
    CLOUD -. Management .-> S3

    S1 -->|Secure VPN| HQ
    S2 -->|Secure VPN| HQ
    S3 -->|Secure VPN| HQ
    HQ --> APP
~~~

Saya lebih suka model **hub-and-spoke** untuk banyak cabang karena lebih mudah dikontrol:

~~~text
Branch A ----\
Branch B -----\
Branch C ------> HQ
Branch D -----/
Branch E ----/
~~~

Tidak semua cabang perlu membuat tunnel langsung ke semua cabang lainnya.

---

# 8. Cost-Saving Model

Saya tidak memasukkan angka harga provider karena tarif berbeda-beda.

Management cukup memasukkan harga aktual ke formula berikut.

### Opsi A - Static IP di Semua Site

~~~text
Yearly Cost
=
(Number of Branches x Monthly Static-IP Fee x 12)
+
Internet Subscription
+
Router
~~~

### Opsi B - Dynamic Public IP + Ruijie/Reyee

~~~text
Yearly Cost
=
Internet Subscription
+
Gateway Investment
+
Operational Maintenance
~~~

Setelah CAPEX router dikeluarkan, biaya static IP bulanan pada cabang yang tidak membutuhkannya dapat dihindari.

### Break-Even Thinking

~~~text
Device Investment
vs
Static-IP Recurring Cost
~~~

Semakin banyak cabang dan semakin panjang periode operasional, semakin penting menghitung total recurring cost daripada hanya melihat harga perangkat di awal.

---

# 9. Tetapi Ada Satu Hal Penting: Dynamic Public IP != CGNAT

Ini bagian yang wajib dicek sebelum membeli perangkat.

DDNS bekerja baik ketika WAN benar-benar memiliki **public IP**, meskipun IP tersebut berubah.

Kalau ISP menempatkan pelanggan di balik **CGNAT**, WAN router bisa mendapat IP seperti:

~~~text
10.x.x.x
172.16.x.x - 172.31.x.x
192.168.x.x
100.64.x.x - 100.127.x.x
~~~

Pada kondisi tersebut, DDNS saja tidak otomatis membuat router dapat menerima koneksi dari Internet.

### Decision Table

| Kondisi WAN | VPN/DDNS |
|---|---|
| Static Public IP | ✅ Sangat mudah |
| Dynamic Public IP | ✅ Cocok dengan DDNS |
| HQ Public IP + Branch Dynamic | ✅ Sangat masuk akal |
| Router di balik modem NAT tetapi port forwarding tersedia | ✅ Bisa disesuaikan |
| CGNAT ISP dan tidak ada inbound mapping | ⚠️ Perlu solusi lain / koordinasi ISP |

Jadi rekomendasi saya bukan:

> "Tidak perlu public IP sama sekali."

Tetapi:

> **"Tidak perlu membayar static IP di setiap site jika dynamic public IP yang tersedia sudah memenuhi kebutuhan VPN dan DDNS."**

Perbedaannya penting.

---

# 10. Checklist Sebelum Implementasi

Sebelum procurement massal:

- [ ] cek WAN IP dari ISP;
- [ ] bandingkan WAN IP router dengan public IP Internet;
- [ ] pastikan tidak berada di CGNAT untuk endpoint yang perlu inbound;
- [ ] cek dukungan DDNS pada model gateway;
- [ ] cek protocol VPN yang akan digunakan;
- [ ] tentukan HQ sebagai hub;
- [ ] tentukan subnet unik di setiap cabang;
- [ ] hindari subnet overlap;
- [ ] pisahkan POS, CCTV, staff, guest dengan VLAN;
- [ ] buat firewall policy;
- [ ] uji failover Internet jika menggunakan dual WAN;
- [ ] uji reconnect VPN ketika public IP berubah;
- [ ] dokumentasikan recovery dan troubleshooting.

---

# 11. Proof of Concept yang Saya Sarankan

Jangan langsung beli untuk seluruh cabang.

Mulai dari:

~~~text
1 HQ
+
2 Branch
~~~

Test selama beberapa minggu.

Yang perlu diuji:

### Test 1 - Dynamic IP Change

Reconnect ISP atau tunggu lease berubah.

Pastikan:

~~~text
IP changes
   |
DDNS updates
   |
VPN reconnects
   |
Service reachable again
~~~

### Test 2 - POS / Internal Application

Pastikan aplikasi cabang dapat berkomunikasi dengan service di HQ sesuai policy.

### Test 3 - CCTV

Uji akses NVR/IPC antar-site bila diperlukan.

### Test 4 - Failover

Jika menggunakan dual WAN:

~~~text
WAN 1 Down
   |
WAN 2 Active
   |
VPN / Business Traffic Recover
~~~

### Test 5 - Remote Troubleshooting

Pastikan tim IT bisa melihat status gateway dan melakukan diagnosis melalui cloud management.

---

# 12. Perangkat yang Bisa Dievaluasi

Dalam materi chain-store resmi mereka, Ruijie/Reyee antara lain menyebut:

- **RG-EG105GW(T)**
- **RG-EG310GH-P-E**

sebagai perangkat yang digunakan pada contoh solusi retail/CCTV mereka.

Bukan berarti dua model itu otomatis cocok untuk semua perusahaan.

Sebelum memilih model, cek:

- jumlah user;
- Internet throughput;
- VPN throughput;
- jumlah tunnel;
- kebutuhan PoE;
- jumlah WAN;
- jumlah VLAN;
- jumlah branch;
- kebutuhan Wi-Fi;
- model firmware dan fitur yang tersedia.

**Choose the device based on workload, not only price.**

---

# 13. Kesimpulan / Conclusion

Kalau perusahaan mempunyai banyak site, saya melihat Ruijie/Reyee bukan hanya sebagai router Internet.

Nilai besarnya justru ketika beberapa fungsi digabungkan:

~~~text
Routing
+
VPN
+
DDNS
+
VLAN
+
Multi-WAN
+
Remote Management
+
Cloud Visibility
~~~

Dari sisi bisnis, ini membuka opsi untuk **mengurangi ketergantungan pada static public IP berbayar di setiap cabang**, selama ISP di lokasi memberikan public dynamic IP yang sesuai.

Artinya perusahaan dapat mengevaluasi:

> Apakah lebih efektif terus membayar recurring static-IP fee di puluhan cabang, atau berinvestasi pada gateway yang sekaligus memberikan routing, VPN, centralized management, dan network control?

Bagi saya, jawabannya perlu dibuktikan melalui **Proof of Concept**, bukan asumsi.

Kalau POC menunjukkan DDNS update berjalan, VPN reconnect stabil, service cabang tetap tersedia, dan remote management efektif, maka solusi ini layak dipertimbangkan sebagai strategi multi-site yang lebih hemat dan scalable.

> **Invest once where it makes sense. Reduce recurring costs where they are not necessary.**

---

## Official Documentation & Sources

1. [Ruijie Reyee - Network Solution in Chain Store CCTV Scenarios](https://reyee.ruijie.com/id-id/blog/cctv-chain-store-solution/)
2. [Ruijie Reyee - Retail & Branch Network Solution](https://reyee.ruijie.com/id-id/solutions/smb/retailchain/)
3. [Ruijie Reyee Routers](https://reyee.ruijie.com/id-id/products/reyee-router/)
4. [Ruijie Community - How to configure Ruijie DDNS on Ruijie Cloud](https://community.ruijie.com/forum.php?mod=viewthread&tid=5846)
5. [Ruijie Community - DDNS Use Cases](https://community.ruijie.com/forum.php?mod=viewthread&tid=2225)
6. [Reyee EG PoC Guide - IPsec VPN](https://reyee.ruijie.com/en-global/support/documents/slide_76717/)

---

## Author

**Muhamad Fahrul**

- GitHub: https://github.com/aaariel18
- LinkedIn: https://www.linkedin.com/in/muhamad-fahrul-428948434/

> Tulisan ini merupakan technical note dan ide arsitektur. Implementasi production tetap harus melalui assessment ISP, security review, capacity planning, dan Proof of Concept.
