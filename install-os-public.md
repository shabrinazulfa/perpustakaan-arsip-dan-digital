# Dokumentasi Instalasi OS Server App

## 1. Menghubungkan ke Wi-Fi

```bash
iwctl
```

Digunakan untuk membuka utilitas **iNet Wireless Daemon (iwd)** sebagai media konfigurasi jaringan nirkabel.

**Berdasarkan CIS**

Penggunaan jaringan yang aman dan terpercaya selama instalasi membantu menjaga integritas sistem serta mengurangi risiko akses dari pihak yang tidak berwenang.

---

```bash
device list
```

Menampilkan perangkat Wi-Fi yang terdeteksi oleh sistem.

**Berdasarkan CIS**

Verifikasi perangkat dilakukan untuk memastikan perangkat jaringan telah dikenali sebelum proses konfigurasi dilanjutkan.

---

```bash
station wlan0 get-networks
```

Menampilkan daftar jaringan Wi-Fi yang tersedia.

**Berdasarkan CIS**

Pemilihan jaringan yang terpercaya merupakan salah satu langkah untuk menjaga keamanan selama proses instalasi.

---

```bash
station wlan0 scan
```

Memperbarui daftar jaringan Wi-Fi yang dapat dijangkau.

**Berdasarkan CIS**

Pemindaian dilakukan agar sistem memperoleh informasi jaringan terbaru sebelum melakukan koneksi.

---

```bash
station wlan0 connect "(nama wifi)"
exit
```

Menghubungkan perangkat ke jaringan Wi-Fi kemudian keluar dari utilitas **iwd**.

**Berdasarkan CIS**

Koneksi yang aman diperlukan agar proses instalasi paket berlangsung dengan baik dan meminimalkan risiko gangguan keamanan.

---

```bash
ping 8.8.8.8
```

Menguji koneksi internet.

**Berdasarkan CIS**

Verifikasi koneksi memastikan sistem siap mengakses repositori resmi selama proses instalasi.

---

# 2. Mengelola Partisi Disk

```bash
lsblk
```

Menampilkan informasi perangkat penyimpanan beserta partisinya.

**Berdasarkan CIS**

Identifikasi perangkat penyimpanan sebelum melakukan perubahan membantu mencegah kesalahan konfigurasi yang dapat menyebabkan kehilangan data.

---

```bash
cfdisk /dev/partisi
```

Contoh:

```bash
cfdisk /dev/sda
```

atau

```bash
cfdisk /dev/nvme0n1
```

Membuat atau mengubah partisi pada media penyimpanan.

Minimal partisi:

```text
boot = 3 GB (EFI System)
root = 70 GB (Linux filesystem)
```

**Berdasarkan CIS**

Pemisahan partisi penting seperti **/boot**, **/home**, **/var**, **/var/log**, **/var/log/audit**, dan **/tmp** membantu meningkatkan keamanan serta membatasi dampak apabila salah satu partisi mengalami gangguan.

---

# 3. Membuat LVM (Logical Volume Manager)

## Membuat Physical Volume

```bash
pvcreate /dev/nvme0n1p7
```

Menginisialisasi partisi menjadi **Physical Volume**.

**Berdasarkan CIS**

LVM mempermudah pengelolaan ruang penyimpanan dan mendukung penerapan partisi yang terpisah sesuai rekomendasi CIS.

---

## Membuat Volume Group

```bash
vgcreate wc /dev/nvme0n1p7
```

Membuat **Volume Group** dari Physical Volume.

**Berdasarkan CIS**

Volume Group memberikan fleksibilitas dalam pengaturan kapasitas penyimpanan sehingga memudahkan penerapan hardening sistem.

---

## Membuat Logical Volume

```bash
lvcreate -L 5G -n root wc
lvcreate -L 5G -n vars wc
lvcreate -L 1G -n vlog wc
lvcreate -L 1G -n vaud wc
lvcreate -L 1G -n home wc
lvcreate -L 1.5G -n vtmp wc
lvcreate -L 9G -n podman wc
lvcreate -l50%FREE -n ngapa wc
```

Membuat Logical Volume untuk setiap kebutuhan sistem.

**Berdasarkan CIS**

Pemisahan direktori penting memudahkan penerapan kebijakan keamanan seperti **nodev**, **nosuid**, dan **noexec**, serta mengurangi risiko apabila salah satu direktori mengalami masalah.

---

```bash
lsblk
```

Memastikan seluruh Logical Volume telah berhasil dibuat.

**Berdasarkan CIS**

Verifikasi konfigurasi diperlukan untuk memastikan struktur penyimpanan telah sesuai sebelum proses instalasi dilanjutkan.

---

# 4. Membuat Filesystem

```bash
mkfs.ext4 /dev/wc/root
mkfs.ext4 /dev/wc/vars
mkfs.ext4 /dev/wc/vlog
mkfs.ext4 /dev/wc/vaud
mkfs.ext4 /dev/wc/home
mkfs.ext4 /dev/wc/vtmp
mkfs.ext4 /dev/wc/podman
```

Membuat filesystem **EXT4** pada setiap Logical Volume.

**Berdasarkan CIS**

Filesystem yang terpisah memudahkan penerapan pengaturan keamanan pada masing-masing partisi sesuai fungsinya.

---

# 5. Konfigurasi LUKS

```bash
cryptsetup luksFormat /dev/wc/ngapa
```

Membuat enkripsi LUKS pada Logical Volume.

**Berdasarkan CIS**

Enkripsi media penyimpanan membantu melindungi kerahasiaan data apabila perangkat hilang atau diakses oleh pihak yang tidak berwenang.

---

```bash
cryptsetup luksOpen /dev/wc/ngapa ngape
```

Membuka volume terenkripsi agar dapat digunakan oleh sistem.

**Berdasarkan CIS**

Volume terenkripsi hanya dapat diakses setelah proses autentikasi sehingga keamanan data lebih terjaga.

---

```bash
mkfs.ext4 /dev/mapper/ngape
```

Membuat filesystem pada volume hasil dekripsi.

**Berdasarkan CIS**

Filesystem pada volume terenkripsi memastikan data tetap tersimpan dalam media yang telah dilindungi oleh mekanisme enkripsi.
