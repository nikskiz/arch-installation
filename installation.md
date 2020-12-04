# arch-kde

Instructions provided install

1. EFI boot
1. lvm encrypted partition
1. arch installation

pacman -Syyy

fdisk /dev/nvme0n1

g (to create an empty GPT partition table)
n
enter
enter
+500M
t
1 (For EFI)

n
enter
enter
+500M

n
enter
enter
enter
t
enter
30

w

mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2

cryptsetup luksFormat /dev/nvme0n1p3
cryptsetup open --type luks /dev/nvme0n1p3 lvm

pvcreate --dataalignment 1m /dev/mapper/lvm
vgcreate volgroup0 /dev/mapper/lvm
lvcreate -l 100%FREE volgroup0 -n lv
modprobe dm_mod
vgscan
vgchange -ay

mkfs.ext4 /dev/volgroup0/lv

mount /dev/volgroup0/lv /mnt

mkdir /mnt/boot

mount /dev/nvme0n1p2 /mnt/boot

mkdir /mnt/etc

genfstab -U -p /mnt >> /mnt/etc/fstab

cat /mnt/etc/fstab

pacstrap -i /mnt base linux-lts linux-lts-headers linux-firmware vim

arch-chroot /mnt

pacman -S networkmanager wpa_supplicant wireless_tools netctl dialog lvm2 sudo grub efibootmgr dosfstools os-prober mtools base-devel

systemctl enable NetworkManager

### Edit /etc/mkinitcpio.conf

#### Add 'ext4' to MODULES

#### Add 'encrypt' and 'lvm2' to HOOKS before filesystems

vim /etc/mkinitcpio.conf

mkinitcpio -p linux-lts

### Setting up Locale

locale-gen
echo LANG=en_AU.UTF-8 > /etc/locale.conf
export LANG=en_AU.UTF-8

timedatectl set-timezone Australia/Sydney

echo arch > /etc/hostname

touch /etc/hosts

127.0.0.1 localhost
::1 localhost
127.0.0.1 arch

### Set users

passwd

useradd -m -g users -G wheel <username>

passwd <username>

visudo

uncomment %wheel ALL=(ALL) ALL

/etc/default/grub
uncomment GRUB_ENABLE_CRYPTODISK=y

Add cryptdevice=<PARTUUID>:volgroup0 to the GRUB_CMDLINE_LINUX_DEFAULT line If using standard device naming, the option will look like this:
cryptdevice=/dev/nvme0n1p3:volgroup0:allow-discards

mkdir /boot/efi

mount /dev/nvme0n1p1 /boot/efi

grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck

grub-mkconfig -o /boot/grub/grub.cfg

fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile

cp /etc/fstab /etc/fstab.bak

echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab

### Install CPU Microde files (Intel CPU)

pacman -S intel-ucode

### Install KDE Plasma

pacman -S xorg plasma plasma-wayland-session kde-applications

systemctl enable sddm

exit

shutdown now

# Installing Packages

sudo pacman -S git vscode go

cd /tmp && wget https://aur.archlinux.org/packages/google-chrome/ && cd google-chrome && makepkg -i -s
