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

### Update mirror list

pacman -S pacman-contrib
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist

### Install CPU Microde files (Intel CPU)

pacman -S intel-ucode

### Install KDE Plasma

pacman -S xorg plasma plasma-wayland-session kde-applications

systemctl enable sddm

exit

shutdown now

# Installing Packages

sudo pacman -S git vscode go docker docker-compose python3 jq python-pip

systemctl enable docker

cd /tmp && git clone https://aur.archlinux.org/google-chrome.git && cd google-chrome && makepkg -i -s
cd /tmp && git clone https://aur.archlinux.org/slack-desktop.git && cd slack-desktop && makepkg -i -s

curl -L -o /tmp/zoom_x86_64.pkg.tar.xz https://zoom.us/client/latest/zoom_x86_64.pkg.tar.xz && sudo pacman -U /tmp/zoom_x86_64.pkg.tar.xz

### bluetooth

#### For setup instructions go to https://wiki.archlinux.org/index.php/bluetooth_headse

sudo pacman -S pulseaudio-bluetooth bluez-utils
systemctl enable bluetooth && systemctl start bluetooth

### SSH agent

vim ~/.config/systemd/user/ssh-agent.service

```
[Unit]
Description=SSH key agent

[Service]
Type=simple
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
# DISPLAY required for ssh-askpass to work
Environment=DISPLAY=:0
ExecStart=/usr/bin/ssh-agent -D -a $SSH_AUTH_SOCK

[Install]
WantedBy=default.target
```

vim ~/.pam_environment

```
SSH_AUTH_SOCK DEFAULT="${XDG_RUNTIME_DIR}/ssh-agent.socket"
```

systemctl --user enable ssh-agent && systemctl --user start ssh-agent

### git config

```
[user]
        email = <email>
        name = <name>
[url "git@github.com:"]
        insteadOf = https://github.com/
[core]
        editor = vim
[pull]
	rebase = false
```

**_ logout and log back in_**

## Kubernete Tools

sudo pacman -S k9s kubectl kubectx helm

sudo curl -L -o /usr/bin/stern https://github.com/wercker/stern/releases/download/1.11.0/stern_linux_amd64 && sudo chmod +x /usr/bin/stern

## Auth tools

CURRENT*VERSION=2.27.1
curl -L -o /tmp/saml2aws*${CURRENT_VERSION}_linux_amd64.tar.gz https://github.com/Versent/saml2aws/releases/download/v${CURRENT*VERSION}/saml2aws*${CURRENT_VERSION}_linux_amd64.tar.gz
sudo tar -xzvf /tmp/saml2aws_${CURRENT_VERSION}\_linux_amd64.tar.gz -C /usr/bin
sudo chmod u+x /usr/bin/saml2aws

## Aws

sudo pacman -S aws-cli
cd /tmp && git clone https://aur.archlinux.org/aws-session-manager-plugin.git && cd aws-session-manager-plugin && makepkg -i -s
pip3 install --user boto3

**_ setup ~/.aws/config and ~/.aws/credentials _**

## terraform

sudo pacman -S terraform terragrunt
