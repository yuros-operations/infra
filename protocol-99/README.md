# base installation

## disk layout

## physical layout 

### storage 1 (minimum)

| disk | partition | type              | luks  | lvm   | label    | size      | format | mount                      |
| ---- | --------- | ----------------- | ----- | ----- | -------- | --------- | ------ | -------------------------- |
| 0    | 1         | efi                | false | false | boot     | 3.9G      | fat 32 | /boot                      |
| 0    | 2         | linux file system  | true  | false | swap     | 4G        | swap   | swapon                     |
| 0    | 3         | linux file system  | true  | true  | proc     | 95G       | luks   | see logical volume proc    |

#### logical volume proc (minimum)
| partition | list  | group  | name     | size   | mount                | format |
| --------- | ----- | ------ | ----     | ------ | --------------------- | --------|
| 3         | 1     | proc   | ring     | 1G     |                       | xfs     |
| 3         | 2     | proc   | root     | 20G    | /mnt                  | xfs     |
| 3         | 2     | proc   | vars     | 5G     | /mnt/var              | xfs     |
| 3         | 3     | proc   | vlog     | 1G     | /mnt/var/log/         | xfs     |
| 3         | 4     | proc   | vaud     | 512M   | /mnt/var/log/audit    | xfs     |
| 3         | 5     | proc   | vtmp     | 1G     | /mnt/var/tmp/         | xfs     |
| 3         | 6     | proc   | vpac     | 2.5G   | /mnt/var/cache/pacman | xfs     |
| 3         | 7     | proc   | temp     | 2G     | /mnt/tmp/         | xfs     |
| 3         | 8     | proc   | home     | 1G     | /mnt/home         | xfs     |
| 3         | 9     | proc   | srvc     | 1G     | /mnt/srvc/http         | xfs     |
| 3         | 10    | proc   | [custom] | 50G    | /mnt/home/[username]         | luks    |

## Mengembalikan hardisk ke stingan pabrik
```
 wipefs -a /dev/[nama partisi]
```
> `wipefs` untuk memformat semua partisi
> example : wipefs -a /dev/nvme0n1

## Labeling Partisi
```
parted /dev/[nama partisi] mklabel gpt
```
> `parted` untuk memilih partisinya, lalu `mklabel` untuk memberikan label pada partisinya, contoh `gpt` adalah labelnya
> example : parted /dev/nvme0n1 mklabel gpt

## Cryptsetup
```
sudo cryptsetup luksFormat --type luks2 \
--align-payload 4096 \
--sector-size 4096 \
--label "proc" \
/dev/[nama partisi]
```
> example : sudo cryptsetup luksFormat --type luks2 --align-payload 4096 --sector-size 4096 --label "proc" /dev/nvme0n1p3

```
cryptsetup open /dev/[nama partisi] proc \
--perf-no_read_workqueue \
--perf-no_write_workqueue \
--persistent
```
> example : cryptsetup open /dev/nvme0n1p3 proc --perf-no_read_workqueue --perf-no_write_workqueue --persistent


## create physical volume
```
pvcreate --dataalignment 4096 /dev/mapper/proc
```

## create virtual group
```
vgcreate proc /dev/mapper/proc
```

## create logical volume

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
lvcreate -L 2G proc -n temp
```

```
lvcreate -L 1G proc -n srvc
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
mkfs.xfs -q -s size=4096 /dev/proc/temp
```

```
mkfs.xfs -q -s size=4096 /dev/proc/srvc
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
mkdir -p /mnt/{boot,home,var,tmp,srv/http}
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
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vaud /mnt/var/log/audit
```

## cache & pacman
```
mkdir -p /mnt/var/cache /mnt/var/cache/pacman
```

```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vpac /mnt/var/cache/pacman
```

## home
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/home /mnt/home
```

## http

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

## swap

```
swapon /dev/[swap partition]
```

## Core System

### AMD
```
pacstrap /mnt linux-hardened linux-hardened-headers linux-firmware-realtek linux-firmware-amd linux-firmware-atheros linux-firmware-other mkinitcpio amd-ucode base base-devel git base neovim lvm2 openssh iptables-nft firewalld rsync wget tuned which xfsprogs kitty-terminfo reflector iwd apparmor --noconfirm
```

### INTEL
```
pacstrap /mnt linux-hardened linux-hardened-headers linux-firmware-realtek linux-firmware-intel linux-firmware-atheros linux-firmware-other mkinitcpio intel-ucode base base-devel git base neovim lvm2 openssh iptables-nft firewalld rsync wget tuned which xfsprogs kitty-terminfo reflector iwd apparmor --noconfirm
```

## copy network

```
cp /etc/systemd/network/* /mnt/etc/systemd/network/
```

```
mkdir -p /mnt/var/lib/iwd
```

```
cp /var/lib/iwd/*.psk /mnt/var/lib/iwd
```

## generate fstab

```
genfstab -U /mnt > /mnt/etc/fstab
```

## chrooting

```
arch-chroot /mnt
```

## hostname

```
echo system > /etc/hostname
```

## time

```
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
```

```
hwclock --systohc
```

## timesync

```
mkdir /etc/systemd/timesyncd.conf.d
```
```
nvim /etc/systemd/timesyncd.conf.d/local.conf
```
```
[Time]
NTP=0.id.pool.ntp.org 1.id.pool.ntp.org 2.id.pool.ntp.org 3.id.pool.ntp.org
FallbackNTP=time.cloudflare.com time.google.com time.aws.com
```
```
timedatectl set-ntp true
```
```
timedatectl set-timezone Asia/Jakarta
```
```
timedatectl status
```
```
timedatectl show-timesync --all
```
> output must like below

```
LinkNTPServers=
SystemNTPServers=0.id.pool.ntp.org 1.id.pool.ntp.org 2.id.pool.ntp.org 3.id.pool.ntp.org
RuntimeNTPServers=
FallbackNTPServers=time.cloudflare.com time.google.com time.aws.com
ServerName=2.id.pool.ntp.org
ServerAddress=103.169.192.230
RootDistanceMaxUSec=5s
PollIntervalMinUSec=32s
PollIntervalMaxUSec=34min 8s
PollIntervalUSec=34min 8s
NTPMessage={ Leap=0, Version=4, Mode=4, Stratum=2, Precision=-23, RootDelay=17.013ms, RootDispersion=504.623ms, Reference=DA498B23, OriginateTimestamp=Tue 2026-05-12 10:49:04 WIB, ReceiveTimestamp=Tue 2026-05-12 10:49:04 WIB, TransmitTimestamp=Tue 2026-05-12 10:49:04 WIB, DestinationTimestamp=Tue 2026-05-12 10:49:04 WIB, Ignored=no, PacketCount=9, Jitter=11.984ms }
Frequency=1082740
```

```
systemctl enable systemd-timesyncd.service
```

> output must like below

```
Created symlink '/etc/systemd/system/dbus-org.freedesktop.timesync1.service' → '/usr/lib/systemd/system/systemd-timesyncd.service'.
Created symlink '/etc/systemd/system/sysinit.target.wants/systemd-timesyncd.service' → '/usr/lib/systemd/system/systemd-timesyncd.service'.
```

## locale

```
nvim /etc/locale.gen
```
> Uncomment the lines below
```
en_US.UTF-8 UTF-8
en_US ISO-8859-1
```

```
locale-gen && locale > /etc/locale.conf
```

```
sed -i 's/^LANG=C.UTF-8/LANG=en_US.UTF-8/' /etc/locale.conf
```

```
sed -i 's/^LC_ALL=/LC_ALL=en_US.UTF-8/' /etc/locale.conf
```

```
cat /etc/locale.conf
```
> outpust must like below
```
LANG=en_US.UTF-8
LC_CTYPE="C.UTF-8"
LC_NUMERIC="C.UTF-8"
LC_TIME="C.UTF-8"
LC_COLLATE="C.UTF-8"
LC_MONETARY="C.UTF-8"
LC_MESSAGES=
LC_PAPER="C.UTF-8"
LC_NAME="C.UTF-8"
LC_ADDRESS="C.UTF-8"
LC_TELEPHONE="C.UTF-8"
LC_MEASUREMENT="C.UTF-8"
LC_IDENTIFICATION="C.UTF-8"
LC_ALL=en_US.UTF-8
```

## vconsole

```
touch /etc/vconsole.conf
```
```
nvim /etc/vconsole.conf
```
> input value the lines below
```
FONT=lat2-16
FONT_MAP=8859-2
```


## package desktop

```
sudo pacman -S hyprland hyprpolkitagent hypridle hyprlock hyprshot xdg-desktop-portal-hyprland pipewire pipewire-pulse pipewire-jack wireplumber pamixer uwsm libnewt kitty qt5-wayland qt6-wayland ttf-jetbrains-mono-nerd ttf-droid ttf-opensans waybar mako tuned-ppd mpd mpc yt-dlp libsecret gnome-keyring superfile perl-image-exiftool wofi wl-clipboard cliphist firefox-developer-edition btop rsync bash-completion pavucontrol zram-generator 
```

#### change /etc/sudoers

## user

> tambahkan http pada sudoers
```
nvim /etc/sudoers
```
> find ## User privilege specification
```
/## User privilege specification
```
> using yy on line root ALL=(ALL:ALL) ALL to copy and paste with p
```/etc/sudoers
root ALL=(ALL:ALL) ALL
http ALL=(ALL:ALL) ALL
user ALL=(ALL:ALL) ALL
```
```
chown -R http:http /srv/http
```

#### change /etc/passwd

```
nvim /etc/passwd
```
```
root:x:0:0::/root:/usr/bin/bash
bin:x:1:1::/:/usr/bin/nologin
daemon:x:2:2::/:/usr/bin/nologin
mail:x:8:12::/var/spool/mail:/usr/bin/nologin
ftp:x:14:11::/srv/ftp:/usr/bin/nologin
http:x:33:33::/srv/http:/usr/bin/nologin
```
> change line http:x:33:33::/srv/http:/usr/bin/nologin like below
```
http:x:33:33::/srv/http:/usr/bin/bash
```
```
passwd http
```
```
chage -E -1 http 
```
```
su http
```
```
sudo su
```

## os release

```
echo '' > /usr/lib/os-release
```
```
nvim /usr/lib/os-release
```
```
NAME="Blackbird"
PRETTY_NAME="Blackbird"
ID=blackbird
BUILD_ID=rolling
ANSI_COLOR="38;2;23;147;209"
HOME_URL="https://blackbird.lektor.co.id/"
DOCUMENTATION_URL="https://blackbird.lektor.co.id/"
SUPPORT_URL="https://blackbird.lektor.co.id/support/"
BUG_REPORT_URL="https://gitlab.blackbird.org/groups/issues"
PRIVACY_POLICY_URL="https://blackbird.lektor.co.id/privacy-policy/"
LOGO=blackbird-logo
```

## package manager

```
nvim /etc/pacman.conf
```
> add TrustedOnly on line SigLevel = Required DatabaseOptional
```
SigLevel = Required DatabaseOptional TrustedOnly
```
uncomment
```
UseSysLog
Color
VerbosePkgLists
```

## apparmor 

```
systemctl enable apparmor.service
```

## sshd
```
systemctl enable sshd
```

## loging config
```
mkdir -p /etc/systemd/journald.conf.d/
```
```
nvim /etc/systemd/journald.conf.d/01-default.conf
```
```
[Journal]
SystemMaxUse=1G
SystemKeepFree=500M
RuntimeMaxUse=200M
RuntimeKeepFree=50M
MaxFileSec=1month
Storage=persistent
```

## sleep config
```
mkdir -p /etc/systemd/sleep.conf.d/
```
```
nvim /etc/systemd/sleep.conf.d/01-blackbird.conf
```
```
[Sleep]
AllowSuspend=no
AllowHibernation=no
AllowHybridSleep=no
AllowSuspendThenHibernate=no
```

## coredump config
```
nvim /etc/systemd/coredump.conf
```
comment "[coredump]" adding on last line
```
[Coredump]
Storage=none
ProcessSizeMax=0
```

## login sudoers
```
nvim /etc/sudo.conf
```
adding on last line
```
## Config Log
Defauhardened logfile="/var/log/sudo.log"
```
### network
```
nvim /etc/systemd/network/20-ethernet.network
```
```
[Network]
Address=[IP]/24
Gateway=10.10.1.1
DNS=1.1.1.1 8.8.8.8
MulticastDNS=yes
```
```
systemctl enable systemd-networkd.socket
```
```
systemctl enable systemd-resolved
```


### boot directory
#### intel server
```
rm /boot/initramfs-linux-hardened*
```
```
mkdir -p /boot/efi /boot/efi/linux /boot/efi/systemd /boot/efi/rescue /boot/efi/boot
```
```
mkdir /boot/kernel
```
```
mv /boot/intel-ucode.img /boot/vmlinuz-linux-hardened /boot/kernel
```

#### amd server
```
rm /boot/initramfs-linux-hardened*
```
```
mkdir -p /boot/efi /boot/efi/linux /boot/efi/systemd /boot/efi/rescue /boot/efi/boot
```
```
mkdir /boot/kernel
```
```
mv /boot/amd-ucode.img /boot/vmlinuz-linux-hardened /boot/kernel
```

### kernel parameter

```
mkdir /etc/cmdline.d
```
```
touch /etc/cmdline.d/{01-boot.conf,02-mods.conf,03-secs.conf,04-perf.conf,05-nets.conf,06-misc.conf}
```
### 01-boot
```
echo "rd.luks.name=$(blkid -s UUID -o value /dev/nvme0n1p3)=proc root=/dev/proc/root" > /etc/cmdline.d/01-boot.conf
```

### 04-perf
```
echo "ipv6.disable=1" > /etc/cmdline.d/04-perf.conf
```

### 06-misc

```
echo "rw quiet" > /etc/cmdline.d/06-misc.conf
```

### default.conf for mkinitcpio

### initram directory

```
rm -fr /etc/mkinitcpio.conf.d
```
```
mv /etc/mkinitcpio.conf /etc/mkinitcpio.d/default.conf
```
```
nvim /etc/mkinitcpio.d/default.conf
```
```
# nvim:set ft=sh:
# MODULES
# The following modules are loaded before any boot hooks are
# run.  Advanced users may wish to specify all system modules
# in this array.  For instance:
#     MODULES=(usbhid xhci_hcd)
MODULES=()

# BINARIES
# This setting includes any additional binaries a given user may
# wish into the CPIO image.  This is run last, so it may be used to
# override the actual binaries included by a given hook
# BINARIES are dependency parsed, so you may safely ignore libraries
BINARIES=()

# FILES
# This setting is similar to BINARIES above, however, files are added
# as-is and are not parsed in any way.  This is useful for config files.
FILES=()

# HOOKS
# This is the most important setting in this file.  The HOOKS control the
# modules and scripts added to the image, and what happens at boot time.
# Order is important, and it is recommended that you do not change the
# order in which HOOKS are added.  Run 'mkinitcpio -H <hook name>' for
# help on a given hook.
# 'base' is _required_ unless you know precisely what you are doing.
# 'udev' is _required_ in order to automatically load modules
# 'filesystems' is _required_ unless you specify your fs modules in MODULES
# Examples:
##   This setup specifies all modules in the MODULES setting above.
##   No RAID, lvm2, or encrypted root is needed.
#    HOOKS=(base)
#
##   This setup will autodetect all modules for your system and should
##   work as a sane default
#    HOOKS=(base udev autodetect microcode modconf block filesystems fsck)
#
##   This setup will generate a 'full' image which supports most systems.
##   No autodetection is done.
#    HOOKS=(base udev microcode modconf block filesystems fsck)
#
##   This setup assembles a mdadm array with an encrypted root file system.
##   Note: See 'mkinitcpio -H mdadm_udev' for more information on RAID devices.
#    HOOKS=(base udev microcode modconf keyboard keymap consolefont block mdadm_udev encrypt filesystems fsck)
#
##   This setup loads an lvm2 volume group.
#    HOOKS=(base udev microcode modconf block lvm2 filesystems fsck)
#
##   This will create a systemd based initramfs which loads an encrypted root filesystem.
#    HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block sd-encrypt filesystems fsck)
#
##   NOTE: If you have /usr on a separate partition, you MUST include the
#    usr and fsck hooks.
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)

# COMPRESSION
# Use this to compress the initramfs image. By default, zstd compression
# is used for Linux ≥ 5.9 and gzip compression is used for Linux < 5.9.
# Use 'cat' to create an uncompressed image.
#COMPRESSION="zstd"
#COMPRESSION="gzip"
#COMPRESSION="bzip2"
#COMPRESSION="lzma"
#COMPRESSION="xz"
#COMPRESSION="lzop"
#COMPRESSION="lz4"

# COMPRESSION_OPTIONS
# Additional options for the compressor
#COMPRESSION_OPTIONS=()

# MODULES_DECOMPRESS
# Decompress loadable kernel modules and their firmware during initramfs
# creation. Switch (yes/no).
# Enable to allow further decreasing image size when using high compression
# (e.g. xz -9e or zstd --long --ultra -22) at the expense of increased RAM usage
# at early boot.
# Note that any compressed files will be placed in the uncompressed early CPIO
# to avoid double compression.
#MODULES_DECOMPRESS="no"
```

> change like below
```
# nvim:set ft=sh:
# MODULES
# The following modules are loaded before any boot hooks are
# run.  Advanced users may wish to specify all system modules
# in this array.  For instance:
#     MODULES=(usbhid xhci_hcd)
MODULES=()

# BINARIES
# This setting includes any additional binaries a given user may
# wish into the CPIO image.  This is run last, so it may be used to
# override the actual binaries included by a given hook
# BINARIES are dependency parsed, so you may safely ignore libraries
BINARIES=()

# FILES
# This setting is similar to BINARIES above, however, files are added
# as-is and are not parsed in any way.  This is useful for config files.
FILES=()

# HOOKS
# This is the most important setting in this file.  The HOOKS control the
# modules and scripts added to the image, and what happens at boot time.
# Order is important, and it is recommended that you do not change the
# order in which HOOKS are added.  Run 'mkinitcpio -H <hook name>' for
# help on a given hook.
# 'base' is _required_ unless you know precisely what you are doing.
# 'udev' is _required_ in order to automatically load modules
# 'filesystems' is _required_ unless you specify your fs modules in MODULES
# Examples:
##   This setup specifies all modules in the MODULES setting above.
##   No RAID, lvm2, or encrypted root is needed.
#    HOOKS=(base)
#
##   This setup will autodetect all modules for your system and should
##   work as a sane default
#    HOOKS=(base udev autodetect microcode modconf block filesystems fsck)
#
##   This setup will generate a 'full' image which supports most systems.
##   No autodetection is done.
#    HOOKS=(base udev microcode modconf block filesystems fsck)
#
##   This setup assembles a mdadm array with an encrypted root file system.
##   Note: See 'mkinitcpio -H mdadm_udev' for more information on RAID devices.
#    HOOKS=(base udev microcode modconf keyboard keymap consolefont block mdadm_udev encrypt filesystems fsck)
#
##   This setup loads an lvm2 volume group.
#    HOOKS=(base udev microcode modconf block lvm2 filesystems fsck)
#
##   This will create a systemd based initramfs which loads an encrypted root filesystem.
#    HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block sd-encrypt filesystems fsck)
#
##   NOTE: If you have /usr on a separate partition, you MUST include the
#    usr and fsck hooks.
#HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole sd-encrypt lvm2 block filesystems fsck)


# COMPRESSION
# Use this to compress the initramfs image. By default, zstd compression
# is used for Linux ≥ 5.9 and gzip compression is used for Linux < 5.9.
# Use 'cat' to create an uncompressed image.
#COMPRESSION="zstd"
#COMPRESSION="gzip"
#COMPRESSION="bzip2"
#COMPRESSION="lzma"
#COMPRESSION="xz"
#COMPRESSION="lzop"
#COMPRESSION="lz4"

# COMPRESSION_OPTIONS
# Additional options for the compressor
#COMPRESSION_OPTIONS=()

# MODULES_DECOMPRESS
# Decompress loadable kernel modules and their firmware during initramfs
# creation. Switch (yes/no).
# Enable to allow further decreasing image size when using high compression
# (e.g. xz -9e or zstd --long --ultra -22) at the expense of increased RAM usage
# at early boot.
# Note that any compressed files will be placed in the uncompressed early CPIO
# to avoid double compression.
#MODULES_DECOMPRESS="no"
```

#### configure linux preset

```
nvim /etc/mkinitcpio.d/linux-hardened.preset
```

```
# mkinitcpio preset file for the 'linux-hardened' package

ALL_config="/etc/mkinitcpio.conf"
ALL_kver="/boot/vmlinuz-linux-hardened"
#ALL_kerneldest="/boot/vmlinuz-linux-hardened"

PRESETS=('default')
#PRESETS=('default' 'fallback')

#default_config="/etc/mkinitcpio.conf"
default_image="/boot/initramfs-linux-hardened.img"
#default_uki="/efi/Linux/arch-linux-hardened.efi"
#default_options="--splash /usr/share/systemd/bootctl/splash-arch.bmp"

#fallback_config="/etc/mkinitcpio.conf"
#fallback_image="/boot/initramfs-linux-hardened-fallback.img"
#fallback_uki="/efi/EFI/Linux/arch-linux-hardened-fallback.efi"
#fallback_options="-S autodetect"
```

> change like below
```
# mkinitcpio preset file for the 'linux-hardened' package

ALL_config="/etc/mkinitcpio.d/default.conf"
ALL_kver="/boot/kernel/vmlinuz-linux-hardened"
ALL_kerneldest="/boot/kernel/vmlinuz-linux-hardened"

PRESETS=('default')
#PRESETS=('default' 'fallback')

#default_config="/etc/mkinitcpio.conf"
#efault_image="/boot/initramfs-linux-hardened.img"
default_uki="/boot/efi/linux/arch-linux-hardened.efi"
#default_options="--splash /usr/share/systemd/bootctl/splash-arch.bmp"

#fallback_config="/etc/mkinitcpio.conf"
#fallback_image="/boot/initramfs-linux-hardened-fallback.img"
#fallback_uki="/efi/EFI/Linux/arch-linux-hardened-fallback.efi"
#fallback_options="-S autodetect"
```
```
bootctl --path=/boot install
```
```
mkinitcpio -P
```

## umount

```
umount -R /mnt
```

```
reboot
```

## after installation

```
rm .bashrc .bash_profile
```

```
git clone https://github.com/almuhdilkarim/galium
```

```
cp -r galium/conf/.* /srv/http
```

## after login user using http
### preparation
```
sudo cryptsetup luksFormat --type luks2 \
--align-payload 4096 \
--sector-size 4096 \
--label "dm" \
/dev/proc/[user name]
```
```
sudo cryptsetup open /dev/proc/[user name] _dev_dm_11 \
--perf-no_read_workqueue \
--perf-no_write_workqueue \
--persistent
```
```
sudo mkfs.xfs -q -s size=4096 /dev/mapper/_dev_dm_11
```
```
sudo mkdir -p /home/[name]
```

```
sudo mount -o rw,nodev,nosuid,relatime /dev/mapper/_dev_dm_11 /home/[name]
```
```
sudo pacman -S pam_mount
```
### Configure the Volume

```
nvim /etc/security/pam_mount.conf.xml
```
> adjust like the lines below
```
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE pam_mount SYSTEM "pam_mount.conf.xml.dtd">
<!--
	See pam_mount.conf(5) for a description.
-->

<pam_mount>

		<!-- debug should come before everything else,
		since this file is still processed in a single pass
		from top-to-bottom -->

<debug enable="0" />

		<!-- Volume definitions -->


		<!-- pam_mount parameters: General tunables -->

<!--
<luserconf name=".pam_mount.conf.xml" />
-->

<!-- Note that commenting out mntoptions will give you the defaults.
     You will need to explicitly initialize it with the empty string
     to reset the defaults to nothing. -->
<mntoptions allow="nosuid,nodev,loop,encryption,fsck,nonempty,allow_root,allow_other" />
<!--
<mntoptions deny="suid,dev" />
<mntoptions allow="*" />
<mntoptions deny="*" />
-->
<mntoptions require="nosuid,nodev" />

<!-- requires ofl from hxtools to be present -->
<logout wait="0" hup="no" term="no" kill="no" />

<!-- Example entry for a LUKS partition -->
<volume 
    user="[user name]" 
    fstype="crypt" 
    path="/dev/proc/[user name]" 
    mountpoint="/home/[user name]" 
/>
		<!-- pam_mount parameters: Volume-related -->

<mkmountpoint enable="1" remove="true" />


</pam_mount>
```

Edit the `pam_mount` configuration file at `/etc/security/pam_mount.conf.xml`. You need to add a `<volume>` entry for your encrypted device.

```xml
<!-- Example entry for a LUKS partition -->
<volume 
    user="[user name]" 
    fstype="crypt" 
    path="/dev/proc/[user name]" 
    mountpoint="/home/[user name]" 
/>
```
*   **path:** The identifier for your encrypted partition (e.g., `/dev/sdb1` or a UUID).
*   **mountpoint:** Where the partition should be accessible after unlocking.
*   **options:** `allow-discard` is useful for SSD performance.

### Update PAM Configuration

```
nvim /etc/pam.d/system-login
```
> adjust like the lines below
```
#%PAM-1.0

auth       required   pam_shells.so
auth       requisite  pam_nologin.so
auth       include    system-auth
auth       required   pam_mount.so

account    required   pam_access.so
account    required   pam_nologin.so
account    include    system-auth

password   include    system-auth

session    optional   pam_loginuid.so
session    optional   pam_keyinit.so       force revoke
session    include    system-auth
session    optional   pam_lastlog2.so      silent
session    optional   pam_motd.so
session    optional   pam_mail.so          dir=/var/spool/mail standard quiet
session    optional   pam_umask.so
session    optional  pam_mount.so
-session   optional   pam_systemd.so
session    required   pam_env.so
```

You must tell the system to use `pam_mount` during the login process. Edit `/etc/pam.d/system-login` to include the following lines in the correct sections:

```pam
# Add to the 'auth' section
auth        required    pam_mount.so

# Add to the 'session' section
session     optional    pam_mount.so
```
*Note: If you use a Display Manager (like GDM or SDDM), ensure its specific PAM file also includes these or inherits from `system-login`.*

### Verification
After saving the files, log out and log back in. The partition should automatically prompt for the password (via the login screen) and mount it to the specified location. You can verify this by running `lsblk` or `mount`.

```
lsblk
```

```
NAME               MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
nvme0n1            259:0    0 476.9G  0 disk  
├─nvme0n1p1        259:1    0     4G  0 part  /boot
├─nvme0n1p2        259:2    0     4G  0 part  [SWAP]
├─nvme0n1p3        259:3    0   100G  0 part  
│ └─proc           253:0    0   100G  0 crypt 
│   ├─proc-ring    253:1    0     1G  0 lvm   
│   ├─proc-root    253:2    0    20G  0 lvm   /
│   ├─proc-vars    253:3    0     5G  0 lvm   /var
│   ├─proc-vlog    253:4    0     1G  0 lvm   /var/log
│   ├─proc-vtmp    253:5    0     1G  0 lvm   /var/tmp
│   ├─proc-vpac    253:6    0   2.5G  0 lvm   /var/cache/pacman
│   ├─proc-home    253:7    0     2G  0 lvm   /home
│   ├─proc-vaud    253:8    0   512M  0 lvm   /var/log/audit
│   ├─proc-temp    253:9    0     2G  0 lvm   /tmp
│   ├─proc-srvc    253:10   0     1G  0 lvm   /srv/http
│   └─proc-[user name]  253:11   0    50G  0 lvm   
│     └─_dev_dm_11 253:12   0    50G  0 crypt /home/[user name]
└─nvme0n1p4        259:4    0   200G  0 part  
```

## user real

```
useradd -d /home/[user name] user
```

```
chown -R user:user /home/[user name]
```

```
passwd user
```
> password must same like luks for this partition
