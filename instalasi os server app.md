# Dokumentasi Instalasi Arch Linux

## 1. Menghubungkan ke Wi-Fi
```
iwctl
```
```
device list
```
Cek driver wifi setiap laptop
```
station wlan0 get-network
```
Melihat jaringan yang tersedia
```
station wlan0 scaan
```
Memindai jaringan yang ada
```
station wlan0 connect "(nama wifi)"
exit
```
Memastikan koneksi internet aktif
```bash
ping 8.8.8.8
```

---

## 2. Mengelola Partisi Disk

```
lsblk
```
Membagi partisi
```
cfdiks /dev/partisi [sda/nvme0n1p1]
```
Minimal partisi
```
boot = 3G [EFI system}
root = 70G [Linux filesystem]
```

## 3. Membuat LVM (Logical Volume Manager)

### Membuat Physical Volume

```bash
pvcreate /dev/nvme0n1p7
```

### Membuat Volume Group

```bash
vgcreate wc /dev/nvme0n1p7
```

### Membuat Logical Volume

```bash
lvcreate -L 5G -n root wc 
lvcreate -L 5G -n vars wc 
lvcreate -L 1G -n vlog wc 
lvcreate -L 1G -n vaud wc 
lvcreate -L 1G -n home wc 
lvcreate -L 1.5G -n vtmp wc 
lvcreate -L 9G -n podman wc 
lvcreate -l l50%FREE -n ngapa wc
```
### Verifikasi Logical Volume
```bash
lsblk
```

## 4. Membuat Filesystem

Memformat logical volume yang telah dibuat:

```bash
mkfs.ext4 /dev/wc/root 
mkfs.ext4 /dev/wc/vars 
mkfs.ext4 /dev/wc/vlog 
mkfs.ext4 /dev/wc/vaud 
mkfs.ext4 /dev/wc/home 
mkfs.ext4 /dev/wc/vtmp 
mkfs.ext4 /dev/wc/podman 
mkfs.ext4 /dev/wc/ngapa
mkfs.ext4 /dev/mapper/ngape     
```

## 4. Konfigurasi LUKS
```bash
cryptsetup luksFormat /dev/wc/ngapa
```
```bash
cryptsetup luksOpen /dev/wc/ngapa ngape   
```
## 5. Mount Partisi

### Mount Root
```bash
mount /dev/wc/root /mnt
```
### Mount Filesystem
```bash
mount /dev/wc/root /mnt
mount --mkdir -o uid=0,gid=0,fmask=0077,dmask=0077 /dev/nvme0n1p6 /mnt/boot   
mount --mkdir -o rw,nodev,nosuid,relatime /dev/wc/vars /mnt/var
mount --mkdir -o rw,nodev,noexec,nosuid,relatime /dev/wc/vlog /mnt/var/log
mount --mkdir -o rw,nodev,noexec,nosuid,relatime /dev/wc/vaud /mnt/var/log/audit
mount --mkdir -o rw,nodev,noexec,nosuid,relatime /dev/wc/home /mnt/home
mount --mkdir -o rw,nodev,noexec,nosuid,relatime /dev/wc/vtmp /mnt/var/tmp
mount --mkdir -o rw,nodev,noexec,nosuid,relatime /dev/wc/podman /mnt/var/lib/containers
```
       
### Verifikasi Mount

```bash
lsblk
```

## 6. Install Package 

```bash
pacstrap /mnt base intel-ucode linux-hardened linux-hardened-headers linux-firmware mkinitcpio lvm2 openssh firewalld podman pacman sudo wget git neovim iwd grep which pam_mount     
```
## 7. Genfstab
```bash
genfstab -U /mnt > /mnt/etc/fstab
```
## 8. Mounting RAM
```bash
echo "tmpfs /tmp tmpfs deafults,rw,nodev,nosuid,noexec,relatime,size=512M 0 0" >> /mnt/etc/fstab
```
## 7. Konfigurasi Sistem

Masuk ke sistem yang baru diinstal:

```bash
arch-chroot /mnt
```

### Mengatur Hostname

```bash
nvim /etc/hostname
```

### Mengatur Timezone

```bash
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc
```

### Mengatur Locale

```bash
nvim /etc/locale.gen
locale-gen
```

---

## 8. Membuat User

Membuat user baru:

```bash
useradd -m auah
passwd auah
```

Memberikan hak akses sudo:

```bash
echo "auah ALL=(ALL:ALL) ALL" > /etc/sudoers.d/auah
```

---

## 9. Mengaktifkan Service

Mengaktifkan layanan yang diperlukan:

```bash
systemctl enable iwd
systemctl enable systemd-networkd
systemctl enable systemd-resolved
systemctl enable sshd
```

---

## 10. Membuat Initramfs

```bash
mkinitcpio -P
```

---

## 11. Konfigurasi Bootloader

Membuat konfigurasi kernel command line:

```bash
mkdir /etc/cmdline.d
nvim /etc/cmdline.d/01-boot.conf
```

---

## 12. Menyelesaikan Instalasi

Keluar dari lingkungan chroot:

```bash
exit
```

Melakukan reboot:

```bash
reboot
```

---

## Kesimpulan

Instalasi Arch Linux berhasil dilakukan menggunakan konfigurasi LVM (*Logical Volume Manager*) dengan kernel `linux-hardened`. Sistem telah dikonfigurasi dengan dukungan jaringan, SSH, firewall, serta pembagian partisi yang terstruktur untuk meningkatkan keamanan dan kemudahan pengelolaan sistem.
