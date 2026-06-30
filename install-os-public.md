# Dokumentasi Instalasi Server Public

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

# 6. Mount Partisi

## Mount Root

```bash
mount /dev/wc/root /mnt
```

Melakukan *mount* partisi root ke direktori `/mnt` sebagai lokasi instalasi sistem.

**Berdasarkan CIS**

Partisi root dipasang sebagai dasar sistem operasi sehingga seluruh direktori dapat dikonfigurasi sesuai struktur partisi yang direkomendasikan CIS.

---

## Mount Filesystem

```bash
mount --mkdir -o uid=0,gid=0,fmask=0077,dmask=0077 /dev/nvme0n1p6 /mnt/boot
```

Melakukan *mount* partisi EFI ke direktori `/boot`.

**Berdasarkan CIS**

Hak akses yang ketat pada partisi EFI membantu melindungi file boot agar tidak mudah diubah oleh pengguna yang tidak memiliki hak akses.

---

```bash
mount --mkdir -o rw,nodev,nosuid,relatime /dev/wc/vars /mnt/var
```

Melakukan *mount* partisi `/var`.

**Berdasarkan CIS**

Opsi `nodev` dan `nosuid` mengurangi risiko penyalahgunaan perangkat khusus (*device file*) maupun file SUID pada direktori `/var`.

---

```bash
mount --mkdir -o rw,nodev,noexec,nosuid,relatime /dev/wc/vlog /mnt/var/log
```

Melakukan *mount* partisi `/var/log`.

**Berdasarkan CIS**

CIS merekomendasikan penggunaan `nodev`, `nosuid`, dan `noexec` pada partisi log untuk mencegah eksekusi program berbahaya.

---

```bash
mount --mkdir -o rw,nodev,noexec,nosuid,relatime /dev/wc/vaud /mnt/var/log/audit
```

Melakukan *mount* partisi `/var/log/audit`.

**Berdasarkan CIS**

Pemisahan log audit membantu menjaga integritas data audit serta mempermudah proses pemantauan keamanan.

---

```bash
mount --mkdir -o rw,nodev,noexec,nosuid,relatime /dev/wc/home /mnt/home
```

Melakukan *mount* partisi `/home`.

**Berdasarkan CIS**

Pemisahan direktori pengguna membantu membatasi dampak apabila terjadi penyalahgunaan akun pengguna.

---

```bash
mount --mkdir -o rw,nodev,noexec,nosuid,relatime /dev/wc/vtmp /mnt/var/tmp
```

Melakukan *mount* partisi `/var/tmp`.

**Berdasarkan CIS**

Direktori sementara sebaiknya menggunakan opsi `noexec`, `nodev`, dan `nosuid` untuk mengurangi risiko eksekusi file berbahaya.

---

```bash
mount --mkdir -o rw,nodev,noexec,nosuid,relatime /dev/wc/podman /mnt/var/lib/containers
```

Melakukan *mount* direktori penyimpanan container.

**Berdasarkan CIS**

Pemisahan penyimpanan container membantu meningkatkan keamanan dan mempermudah pengelolaan data aplikasi.

---

### Verifikasi Mount

```bash
lsblk
```

Memastikan seluruh partisi telah terpasang dengan benar.

**Berdasarkan CIS**

Verifikasi dilakukan untuk memastikan konfigurasi penyimpanan telah sesuai sebelum proses instalasi dilanjutkan.

---

# 7. Install Package

```bash
pacstrap /mnt base intel-ucode linux-hardened linux-hardened-headers linux-firmware mkinitcpio lvm2 openssh firewalld podman pacman sudo wget git neovim iwd grep which pam_mount
```

Menginstal sistem dasar beserta paket yang diperlukan.

**Berdasarkan CIS**

CIS menyarankan hanya menginstal paket yang dibutuhkan (*minimal installation*) untuk mengurangi *attack surface*. Penggunaan `linux-hardened` juga meningkatkan keamanan kernel.

---

# 8. Genfstab

```bash
genfstab -U /mnt > /mnt/etc/fstab
```

Membuat file `fstab` berdasarkan UUID setiap partisi.

**Berdasarkan CIS**

Penggunaan UUID membuat proses *mount* lebih konsisten karena tidak bergantung pada nama perangkat penyimpanan yang dapat berubah.

---

# 9. Mounting RAM

```bash
echo "tmpfs /tmp tmpfs defaults,rw,nodev,nosuid,noexec,relatime,size=512M 0 0" >> /mnt/etc/fstab
```

Menambahkan konfigurasi `tmpfs` pada direktori `/tmp`.

**Berdasarkan CIS**

CIS merekomendasikan penggunaan `tmpfs` serta opsi `nodev`, `nosuid`, dan `noexec` pada `/tmp` untuk mengurangi risiko penyalahgunaan direktori sementara.

---

# 10. Konfigurasi Sistem

Masuk ke lingkungan sistem yang baru diinstal.

```bash
arch-chroot /mnt
```

Mengakses sistem hasil instalasi untuk melakukan konfigurasi lanjutan.

**Berdasarkan CIS**

Konfigurasi sistem dilakukan sebelum sistem digunakan agar seluruh pengaturan keamanan dapat diterapkan sejak awal.

---

## Mengatur Hostname

```bash
nvim /etc/hostname
```

Menentukan nama host sistem.

**Berdasarkan CIS**

Hostname yang jelas memudahkan identifikasi perangkat pada lingkungan jaringan.

---

## Mengatur Timezone

```bash
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc
```

Mengatur zona waktu dan menyinkronkan waktu perangkat keras.

**Berdasarkan CIS**

Waktu sistem yang akurat sangat penting untuk pencatatan log dan audit keamanan.

---

## Mengatur Locale

```bash
nvim /etc/locale.gen
```

Mengaktifkan locale yang diperlukan.

```bash
locale-gen
```

Membuat konfigurasi locale.

```bash
locale > /etc/locale.conf
```

Menyimpan konfigurasi locale.

```bash
nvim /etc/locale.conf
```

Mengubah konfigurasi locale menjadi:

```text
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8
```

**Berdasarkan CIS**

Konfigurasi locale yang konsisten membantu memastikan aplikasi dan layanan sistem berjalan dengan benar.

---

# 11. Membuat User

## Membuat Direktori Home

```bash
mkdir /home/user
```

Membuat direktori home pengguna.

---

## Menambahkan User

```bash
useradd -d /home/user auah
```

Menambahkan akun pengguna baru.

**Berdasarkan CIS**

CIS merekomendasikan penggunaan akun pengguna terpisah dan menghindari penggunaan akun `root` untuk aktivitas sehari-hari.

---

## Mengatur Kepemilikan Direktori

```bash
chown -R auah:auah /home/user
```

Memberikan kepemilikan direktori kepada pengguna.

**Berdasarkan CIS**

Hak kepemilikan yang benar membantu mencegah akses tidak sah terhadap data pengguna.

---

## Mengatur Password

```bash
passwd
```

Mengatur kata sandi akun `root`.

```bash
passwd auah
```

Mengatur kata sandi pengguna.

**Berdasarkan CIS**

CIS merekomendasikan penggunaan kata sandi yang kuat untuk meningkatkan keamanan autentikasi.

---

## Memberikan Hak Akses Sudo

```bash
echo "auah ALL=(ALL:ALL) ALL" > /etc/sudoers.d/auah
```

Memberikan hak akses `sudo` kepada pengguna.

**Berdasarkan CIS**

Hak akses administrator sebaiknya hanya diberikan kepada pengguna yang membutuhkan sesuai prinsip **least privilege**.

# 12. Konfigurasi PAM Mount

Mengedit file konfigurasi PAM Mount.

```bash
nvim /etc/security/pam_mount.conf.xml
```

Digunakan untuk mengatur konfigurasi **PAM Mount** agar partisi terenkripsi dapat dipasang secara otomatis ketika pengguna berhasil login.

**Berdasarkan CIS**

CIS merekomendasikan perlindungan data sensitif melalui mekanisme autentikasi dan enkripsi sehingga akses terhadap data hanya diberikan kepada pengguna yang sah.

---

Menambahkan konfigurasi volume terenkripsi.

```xml
<volume
    user="auah"
    fstype="crypt"
    path="/dev/wc/ngapa"
    mountpoint="/home/user"
/>
```

Menentukan volume terenkripsi yang akan dibuka dan dipasang secara otomatis saat pengguna melakukan login.

**Berdasarkan CIS**

Integrasi autentikasi dengan media penyimpanan terenkripsi membantu menjaga kerahasiaan data pengguna.

---

Mengedit konfigurasi PAM Login.

```bash
nvim /etc/pam.d/system-login
```

Menambahkan modul PAM Mount.

```text
auth       required   pam_mount.so
session    optional   pam_mount.so
```

Mengaktifkan modul **pam_mount** pada proses autentikasi dan sesi login.

**Berdasarkan CIS**

Penggunaan modul autentikasi memastikan hanya pengguna yang berhasil melakukan autentikasi yang dapat mengakses data terenkripsi.

---

Mengedit konfigurasi `mkinitcpio`.

```bash
nvim /etc/mkinitcpio.conf
```

Menambahkan **sd-encrypt** dan **lvm2** pada bagian **HOOKS**.

**Berdasarkan CIS**

Penambahan modul yang diperlukan memastikan sistem mampu membuka volume terenkripsi dan LVM saat proses boot tanpa mengurangi keamanan sistem.

---

# 13. Menyalin Konfigurasi Jaringan

Keluar dari lingkungan `chroot`.

```bash
exit
```

Keluar dari sistem hasil instalasi sementara.

---

Menyalin konfigurasi jaringan.

```bash
cp /etc/systemd/network/* /mnt/etc/systemd/network
```

Menyalin konfigurasi jaringan ke sistem yang baru diinstal.

**Berdasarkan CIS**

Konfigurasi jaringan yang konsisten membantu mengurangi kesalahan konfigurasi setelah sistem selesai diinstal.

---

Masuk kembali ke lingkungan `chroot`.

```bash
arch-chroot /mnt
```

Melanjutkan konfigurasi pada sistem hasil instalasi.

---

Mengedit preset kernel.

```bash
nvim /etc/mkinitcpio.d/linux-hardened.preset
```

Mengubah konfigurasi menjadi:

```text
PRESETS=('default')
default_uki="/boot/EFI/Linux/arch-linux-hardened.efi"
```

**Berdasarkan CIS**

Penggunaan **linux-hardened** memberikan perlindungan tambahan terhadap berbagai jenis eksploitasi dan meningkatkan keamanan kernel.

---

# 14. Instalasi Bootloader (systemd-boot)

Menginstal **systemd-boot**.

```bash
bootctl --path=/boot install
```

Menginstal bootloader pada partisi EFI.

**Berdasarkan CIS**

Bootloader yang dikonfigurasi dengan benar membantu memastikan proses boot berjalan secara aman dan konsisten.

---

Keluar dari lingkungan `chroot`.

```bash
exit
```

---

Menginstal kembali bootloader pada sistem hasil instalasi.

```bash
bootctl --path=/mnt/boot install
```

Memastikan bootloader telah terpasang pada sistem yang baru diinstal.

**Berdasarkan CIS**

Verifikasi instalasi bootloader membantu memastikan sistem dapat melakukan proses boot dengan benar.

---

# 15. Konfigurasi Kernel Command Line

Masuk kembali ke sistem.

```bash
arch-chroot /mnt
```

Melanjutkan konfigurasi sistem.

---

Mengedit konfigurasi kernel.

```bash
nvim /etc/cmdline.conf
```

Isi file:

```text
root=/dev/wc/root rw
```

Menentukan lokasi partisi root saat proses boot.

---

Membangun kembali initramfs.

```bash
mkinitcpio -P
```

Menerapkan perubahan konfigurasi kernel.

**Berdasarkan CIS**

Pembaruan initramfs memastikan seluruh konfigurasi keamanan diterapkan saat sistem melakukan boot.

---

Menghapus konfigurasi lama.

```bash
rm -fr /etc/cmdline.conf
```

Menghapus file konfigurasi yang tidak lagi digunakan.

---

Membuat direktori konfigurasi.

```bash
mkdir /etc/cmdline.d
```

Menyiapkan direktori penyimpanan konfigurasi kernel.

---

Membuat file konfigurasi baru.

```bash
nvim /etc/cmdline.d/01-boot.conf
```

Isi file:

```text
root=/dev/wc/root rw
```

Menyimpan konfigurasi kernel pada direktori yang baru.

---

Membangun kembali initramfs.

```bash
mkinitcpio -P
```

Memastikan konfigurasi terbaru diterapkan.

**Berdasarkan CIS**

Verifikasi konfigurasi boot membantu memastikan sistem menggunakan parameter kernel yang telah ditentukan.

---

# 16. Mengaktifkan Service

Mengaktifkan layanan Wi-Fi.

```bash
systemctl enable iwd
```

Mengaktifkan layanan **iwd** saat sistem dijalankan.

---

Mengaktifkan layanan jaringan.

```bash
systemctl enable systemd-networkd
```

Mengaktifkan pengelolaan jaringan.

---

Mengaktifkan DNS Resolver.

```bash
systemctl enable systemd-resolved
```

Mengaktifkan layanan resolusi nama domain.

---

Mengaktifkan Firewall.

```bash
systemctl enable firewalld
```

Mengaktifkan **Firewalld** saat proses boot.

**Berdasarkan CIS**

CIS merekomendasikan penggunaan firewall untuk membatasi lalu lintas jaringan sehingga hanya layanan yang diperlukan yang dapat diakses.

---

Mengaktifkan SSH.

```bash
systemctl enable sshd
```

Mengaktifkan layanan **SSH**.

**Berdasarkan CIS**

SSH sebaiknya hanya diaktifkan apabila diperlukan serta dikonfigurasi dengan mekanisme autentikasi yang aman.

---

Mengaktifkan Podman.

```bash
systemctl enable --global podman
```

Mengaktifkan layanan Podman.

**Berdasarkan CIS**

Layanan yang digunakan secara rutin dapat diaktifkan, sedangkan layanan yang tidak diperlukan sebaiknya dinonaktifkan untuk mengurangi *attack surface*.

---

# 17. Menyelesaikan Instalasi

Keluar dari lingkungan `chroot`.

```bash
exit
```

Keluar dari sistem hasil instalasi.

---

Melepas seluruh partisi.

```bash
umount -R /mnt
```

Melepas seluruh partisi yang telah dipasang.

**Berdasarkan CIS**

Unmount dilakukan untuk memastikan seluruh data telah tersimpan dengan baik sebelum sistem dimatikan atau dihidupkan ulang.

---

Melakukan reboot.

```bash
reboot
```

Memulai ulang sistem agar seluruh konfigurasi yang telah dilakukan dapat diterapkan.

**Berdasarkan CIS**

Restart merupakan tahap akhir untuk memastikan seluruh konfigurasi keamanan telah aktif dan sistem siap digunakan dengan konfigurasi yang telah dihardening.
