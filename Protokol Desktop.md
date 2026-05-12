## Mengembalikan hardisk ke stingan pabrik
```
 wipefs -a /dev/[nama partisi]
```
`wipefs` untuk memformat semua partisi

## Labeling Partisi
```
parted /dev/[nama partisi] mklabel gpt
```
`parted` untuk memilih partisinya, lalu `mklabel` untuk memberikan label pada partisinya, contoh `gpt` adalah labelnya

## Cryptsetup
```
sudo cryptsetup luksFormat -- type2 luks2 \ 
--align-payload 4096 \
--sector-size 4096 \
-- label "reng" \
/dev/[nama partisi]
```

```
cryptsetup open /dev/[nama partisi] proc \ 
--pref-no_read_workqueue \
--pref-no_write_workqueue \
--persistent
```

## membuat physical volume
```
pvcreate --dataaligment 4096 /dev/mapper/proc
```
## membuat virtual grup
```
vgcreate proc /dev/mapper/proc
```
## membuat logical volume
```
lvcreate -L 1G proc -n ring
```
```
lvcreate -L 20G proc -n root
```
```
lvcreate -L 5G proc -n vars
```
```
lvcreate -L 1G proc -n vlog
```
```
lvcreate -L 1G proc -n vtmp
```
```
lvcreate -L 2.5G proc -n vpac
```
```
lvcreate -L 2G proc -n home
```
```
lvcreate -L 512M proc -n vaud
```
```
lvcreate -L 2G proc -n tmp
```
```
lvcreate -L 1G proc -n srvc
```
```
lvcreate -L 5G proc -n vars
```
```
lvcreate -L 50G proc -n [nama home pribadi]
```

```
mkfs.xfs -q -s size=4096 /dev/proc/root
```
```
mkfs.xfs -q -s size=4096 /dev/proc/vars
```
```
mkfs.xfs -q -s size=4096 /dev/proc/vlog
```
```
mkfs.xfs -q -s size=4096 /dev/proc/vaud
```
```
mkfs.xfs -q -s size=4096 /dev/proc/vtmp
```
```
mkfs.xfs -q -s size=4096 /dev/proc/vpac
```
```
mkfs.xfs -q -s size=4096 /dev/proc/home
```
```
mkfs.xfs -q -s size=4096 /dev/proc/vaud
```
```
mkfs.xfs -q -s size=4096 /dev/proc/tmp
```
```
mkfs.xfs -q -s size=4096 /dev/proc/srvc
```
```
mkfs.xfs -q -s size=4096 /dev/proc/vars
```
```
mkfs.xfs -q -s size=4096 /dev/proc/[nama home pribadi]
```
```
mkswap /dev/[nama partisi]
```

```
mkfs.fat -F 32 -S 4096 -n "BOOT" /dev/[partisi boot]
```

```
mount /dev/proc/root /mnt
```

## boot & var
```
mkdir -p /mnt/{boot,home,var,tmp,srvc/http}
```

```
mount -o uid=0,gid=0,fmask=0077,dmask=0077 /dev/[nama partisi] /mnt/boot
```

```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vars /mnt/var
```

## var/log

```
mkdir -p /mnt/var/log
```

```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vlog /mnt/var/log
```

## var/log/audit
```
mkdir -p /mnt/var/log/audit
```

```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vaud /mnt/var/audit
```

## cache & pacman
```
mkdir -p /mnt/var/cache /mnt/var/cache/pacman
```

```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vpac /mnt/var/cache/pacman
```

## swapon
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/home /mnt/home
```

```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/srvc /mnt/srv/http
```

untuk mengubah size
```
lvresize -L [berapa G] /dev/proc/home
```

## vtmp
```
mkdir -p /mnt/var/tmp
```

```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vtmp /mnt/var/tmp
```
## temp

```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/temp /mnt/tmp
```

## null (menyesuaikan)
```
mkdir -p /mnt/home/[name]
```

```
mount -o rw,nodev,nosuid,relatime /dev/proc/[name dir] /mnt/home/[name]
```

## Core System

### AMD
```
pacstrap /mnt linux-hardened linux-harderned-headers linux-firmware-realtek linux-firmware-amdgpu linux-firmware-etheros linux-firmware-other mkinitcpio amd-ucode base base-devel git base neovim lvm2 openssh iptables-nft firewalld rsync wget tuned which xfsprogs kitty-terminfo reflector iwd --noconfirm
```

### INTEL
```
pacstrap /mnt linux-hardened linux-harderned-headers linux-firmware-realtek linux-firmware-intelgpu linux-firmware-etheros linux-firmware-other mkinitcpio intel-ucode base base-devel git base neovim lvm2 openssh iptables-nft firewalld rsync wget tuned which xfsprogs kitty-terminfo reflector iwd --noconfirm
```

catatan
untuk home pribadi itu noexec nya di hapus agar bisa mengeksekusi
package kitty-terminfo dan reflector


