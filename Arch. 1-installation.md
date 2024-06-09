---
date: 2024-02-04
links:
  - "[[_MOC Linux]]"
  - "[[Arch. 2-customization]]"
tags:
  - linux
  - workstation
aliases:
  - workstation
cssclasses: 
status: true
---
# <span style="color:#ffadad">Installation guide</span>
>[Official guide](https://wiki.archlinux.org/title/Installation_guide_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9))
>This one based on [this](https://github.com/vlad196/arch-installation-guide)

## <span style="color:#ffd6a5">General approach</span>
1. Use connect via SSH. It allows to copy-paste commands
2. Partition and encryption of disks
6. System Settings: clock, user, host, locale
7. Boot section and bootloader
9. Security basics
10. Some more settings and reboot
11. Security settings + packages

>[!warning] Disable Secure boot
>Arch Linux installation images do not support Secure Boot. You will need to [disable Secure Boot](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Disabling_Secure_Boot "Unified Extensible Firmware Interface/Secure Boot") to boot the installation medium. If desired, [Secure Boot](https://wiki.archlinux.org/title/Secure_Boot "Secure Boot") can be set up after completing the installation.
>Check status via `bootctl`

**Hardware System**
Laptop
- Lenovo IdeaPAd L340 Gaming

Core
- Intel(R) Core(TM) i5-9300H CPU @ 2.40GHz

Graphic
- Nvidia GeForce GTX 1650
- Intel(R) UHD Graphics 630

Audio
- Nvidia high definition audio
- Realtek

MotherBoard
- Intel(R) 300 Series Chipset Family Sata AHCI Controller

Ethernet/wi-fi
- Realtek 8821CE Wireless LAN 802.11ac PCI-E NIC
- Realtek PCIe Gbe Family Controller

## <span style="color:#ffd6a5">SSH connection</span>
### <span style="color:#fdffb6">On client machine</span>
Use utility
```bash
iwctl
```

Check devices
```bash
device list
```

Turn on adapter if status 'off'
```bash
adapter <adapter> set-property Powered on
```

Search wi-fi
```bash
station <device> scan
```

Get list of networks
```bash
station <device> get-networks
```

Connect
```bash
station <device> connect _SSID_
```

Now exit
```bash
exit
```

Set password
```bash
passwd
```

Get IP
```bash
ip -br a
```
### <span style="color:#fdffb6">On server</span>
Use `ServerAliveInterval=30` to prevent ssh disconnect

```bash
ssh ssh -o ServerAliveInterval=30 root@<client ip>
```

## <span style="color:#ffd6a5">Disk preparation</span>
Get list and names of disks
```bash
fdisk -l
```

Check if disk supports TRIM (for SSD)
```bash
lsblk --discard
```
And check the values of DISC-GRAN (discard granularity) and DISC-MAX (discard max bytes) columns. Non-zero values indicate TRIM support.

Names differs /dev/nvme0n1 - for SSD. **nvme0n1** - variable part

UEFI and SSD used in this guid
### <span style="color:#fdffb6">Partition</span>
>[!Hint]
>Can use `cfdisk`

Wipe disk
```bash
wipefs -a /dev/nvme0n1
```

Use GPT as partition table
```bash
parted /dev/nvme0n1 mklabel gpt
```

Create sections
>[!warning] Alignment 
>Очень важно, следите за размером секторов на диске! От этого зависит скорость доступа к диску It's important to monitor size of sections. It affects on access speed: https://wiki.archlinux.org/title/Parted#Alignment

```bash
parted /dev/nvme0n1 mkpart '"EFI system partition"' fat32 1MiB 2GiB && \
parted /dev/nvme0n1 mkpart '"swap partition"' linux-swap 2GiB 10GiB && \
parted /dev/nvme0n1 mkpart '"system partition"' ext4 10GiB 100%
```

>[!note] Sizes and file system
>fat32 is only suitable for EFI. If hibernation is going to be used swap partition must have size of RAM

Set flags for sections
```bash
parted /dev/nvme0n1 set 1 esp on && \
parted /dev/nvme0n1 set 2 swap on
```

>[!note] set command
>`set _partition_ boot on`
>`_partition_` - number of section that is flagged (see result of `print`).

Create boot section
```bash
mkfs.fat -F32 /dev/nvme0n1p1
```

### <span style="color:#fdffb6">Encryption</span>
Make sure that modules works
```bash
modprobe dm-crypt && \
modprobe dm-mod
```

Encrypt swap
```bash
cryptsetup --verbose luksFormat --key-size 512 --hash sha512 /dev/nvme0n1p2
```

Encrypt root
```bash
cryptsetup --verbose luksFormat --key-size 512 --hash sha512 /dev/nvme0n1p3
```

Open encrypted sections
>[!warning] SSD Use allow-discard for SSD
> [Discard/TRIM support for solid state drives (SSD)](https://wiki.archlinux.org/title/dm-crypt/Specialties#Discard/TRIM_support_for_solid_state_drives_(SSD))
> [Solid state drive](https://wiki.archlinux.org/title/Solid_state_drive "Solid state drive") users should be aware that, by default, TRIM commands are not enabled by the device-mapper, i.e. block-devices are mounted without the `discard` option unless you override the default.
> If TRIM no supported,  don't use `--allow-discards`

```bash
cryptsetup --allow-discards luksOpen /dev/nvme0n1p2 swap && \
cryptsetup --allow-discards luksOpen /dev/nvme0n1p3 root
```

Check sections 
```bash
ls /dev/mapper/*
```

Export UUID to variables
```bash
export NVME0N1P1=$(lsblk -dno UUID /dev/nvme0n1p1) \
NVME0N1P2=$(lsblk -dno UUID /dev/nvme0n1p2) \
NVME0N1P3=$(lsblk -dno UUID /dev/nvme0n1p3)
```

Export encrypted sections 
```bash
export ROOT=/dev/mapper/root \
SWAP=/dev/mapper/swap
```

### <span style="color:#fdffb6">Create file system</span>
Swap
```bash
mkswap -L swap $SWAP
```

Root
```bash
mkfs.ext4 $ROOT
```

### <span style="color:#fdffb6">Mount disks</span>
Reload info
```bash
systemctl daemon-reload
```

Mount root
```bash
mount $ROOT /mnt
```

Mount swap
```bash
swapon $SWAP
```

Mount efi
```bash
mount --mkdir -o uid=0,gid=0,fmask=0137,dmask=0027  /dev/nvme0n1p1 /mnt/boot
```
### <span style="color:#fdffb6">Install basic packages</span>
```bash
pacstrap -K /mnt base base-devel linux linux-firmware reflector git neovim iwd networkmanager efibootmgr intel-ucode
```
### <span style="color:#fdffb6">FSTAB generetion</span>
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
Check file `/mnt/etc/fstab` and edit if needed

## <span style="color:#ffd6a5">System settings</span>
root in new system
```bash
arch-chroot /mnt
```
### <span style="color:#fdffb6">Clock</span>
List timezone
```bash
timedatectl list-timezones
```

Set timezone
```bash 
ln -sf /usr/share/zoneinfo/Asia/Omsk /etc/localtime
```

>[!note]
>More about clock and timezone [here](https://wiki.archlinux.org/title/System_time_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)#%D0%A7%D0%B0%D1%81%D0%BE%D0%B2%D0%BE%D0%B9_%D0%BF%D0%BE%D1%8F%D1%81)

To prevent timeshift tune [sync](https://wiki.archlinux.org/title/System_time#Time_synchronization) with  [Network Time Protocol](https://en.wikipedia.org/wiki/ru:NTP "wikipedia:ru:NTP") (NTP), e.g. [systemd-timesyncd](https://wiki.archlinux.org/title/Systemd-timesyncd_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9) "Systemd-timesyncd (Русский)").

Sync
```bash
hwclock --systohc
```
### <span style="color:#fdffb6">Locale</span>
English
```bash
sed '/en_IE.UTF-8 UTF-8/s/^#//' -i /etc/locale.gen
```
>[!tip] Ireland locale
>This locale close to international formats, e.g.:
>- week starts on **Monday**
>- date format **dd.mm.yyyy**

Russian
```bash
sed '/ru_RU.UTF-8 UTF-8/s/^#//' -i /etc/locale.gen
```

Generate locale
```bash
locale-gen
```

Default language
```bash
echo LANG=en_US.UTF-8 >> /etc/locale.conf
```

Make date format look **dd.mm.yyyy**
```bash
echo LC_TIME=ru_RU.UTF-8 >> /etc/locale.conf
```


`ruwin_alt_sh-UTF-8` allows to switch keyboard layout in tty by pressing `alt + shift`
```bash
cat <<- _EOF_ > /etc/vconsole.conf
	KEYMAP=ruwin_alt_sh-UTF-8
	FONT=cyr-sun16
_EOF_
```
### <span style="color:#fdffb6">Host</span>
Name
```bash
echo "arch" >> /etc/hostname
```

Local net
```bash
cat << _EOF_ >> /etc/hosts
127.0.0.1		localhost
::1			localhost
127.0.1.1		arch.localdomain	arch
_EOF_
```
### <span style="color:#fdffb6">User</span>
Create password for root
```bash
passwd
```

Add uer
```bash
useradd -mG wheel oshitaka
```

Make backup
```bash
cp /etc/sudoers /etc/sudoers.backup
```

Give superuser rights to new user
```bash
sed '/\%wheel ALL=(ALL:ALL) ALL/s/^# //' -i /etc/sudoers
```

Check if everything correct
```bash
visudo -c /etc/sudoers
```

If ok delete backup
```bash
rm /etc/sudoers.backup
```

Passwaord for new user
```bash
passwd oshitaka
```

## <span style="color:#ffd6a5">Packet manager settings</span>
### <span style="color:#fdffb6">PARU</span>
Use paru to work with AUR library. Download
```bash
sudo -u oshitaka git clone https://aur.archlinux.org/paru.git /home/oshitaka/bin/paru && \
cd /home/oshitaka/bin/paru/
```

Install and go back to root
```bash
sudo -u oshitaka makepkg -si && \
cd /
```

make paru to wait till work is done - no timer
```bash
sed '/SudoLoop/s/^#//' -i /etc/paru.conf
```

Allow parallel downloads
```bash
sed '/ParallelDownloads =/s/^#//' -i /etc/pacman.conf
```

### <span style="color:#fdffb6">Update and install other packages</span>
Update mirrors by speed
```bash
reflector --verbose -l 5 -p https --sort rate --save /etc/pacman.d/mirrorlist
```

Initialisation 
```bash
sudo -u oshitaka paru -Sy archlinux-keyring && sudo -u oshitaka paru -Su
```

Audio and video drivers
```bash
sudo -u oshitaka paru -Sy pipewire pipewire-alsa pipewire-pulse pipewire-audio wireplumber mesa lib32-mesa vulkan-intel lib32-vulkan-intel nvidia-open nvidia-utils lib32-nvidia-utils vulkan-icd-loader lib32-vulkan-icd-loader
```


## <span style="color:#ffd6a5">Boot settings</span>
### <span style="color:#fdffb6">Boot section settings</span>
Set boot modules

>ПMore about `mkinitcpio` - https://wiki.archlinux.org/title/Mkinitcpio

Start services
```bash
systemctl enable NetworkManager.service && \
systemctl enable bluetooth.service
```

Copy mkinitcpio.conf to mkinitcpio.conf.d:
```bash
cp /etc/mkinitcpio.conf /etc/mkinitcpio.conf.d/mkinitcpio.conf 
```

Edit hook mkinitcpio. Replace `udev`, on `systemd`. Modules `keymap` and`consolefont` replace on `sd-vconsole` and after `block` paste hooks `sd-encrypt` and `resume`

```bash
sed -i '/^HOOKS=/ s/kms//g' /etc/mkinitcpio.conf.d/mkinitcpio.conf  && \
sed -i '/^HOOKS=/ s/udev/systemd/' /etc/mkinitcpio.conf.d/mkinitcpio.conf  && \
sed -i '/^HOOKS=/ s/keymap consolefont/sd-vconsole/' /etc/mkinitcpio.conf.d/mkinitcpio.conf && \
sed -i '/^HOOKS=/ s/\(block\)\(.*\)$/\1 sd-encrypt resume\2/' /etc/mkinitcpio.conf.d/mkinitcpio.conf &&\
sed -e 's/\(MODULES=(\)/\1nvidia nvidia_modeset nvidia_uvm nvidia_drm/' -i /etc/mkinitcpio.conf.d/mkinitcpio.conf
```
Hook's queue : https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#Configuring_the_boot_loader


**<span style="color:#fdffb6">Check file</span>**
```bash
nvim /etc/mkinitcpio.conf.d/mkinitcpio.conf 
```

>[!Note]
>Reason replacing modules on systemd - default modules can't open more than 1 encrypted section: https://wiki.archlinux.org/title/Dm-crypt/System_configuration#Using_encrypt_hook
>`kms` delete because we use `nvidia`

Add /etc/crypttab.initramfs
```bash
cat << _EOF_ > /etc/crypttab.initramfs
# Mount /dev/mapper/swap re-encrypting it with a fresh key each reboot
swap UUID=$NVME0N1P2 none timeout=180,tpm2-device=auto,discard
# Mount /dev/mapper/root with the key from TPM
root UUID=$NVME0N1P3 none timeout=180,tpm2-device=auto,discard
_EOF_
```

Add hook for update `initramfs` after updating `nvidia`
```bash
cat << _EOF_ > /etc/pacman.d/hooks/nvidia.hook
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia-open
Target=linux

[Action]
Description=Update NVIDIA module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux*) exit 0; esac; done; /usr/bin/mkinitcpio -P'
_EOF_
```

Tweak `nvidia`

>[!warning]
>When i will find out how to fix hibernation with 2 monitors uncomment 1-st line: [[Arch. 4-tips#<span style="color ffd6a5">Hibernation</span>]]

```bash
cat << _EOF_ > /etc/modprobe.d/nvidia.conf
# options nvidia NVreg_PreserveVideoMemoryAllocations=1
#
# Allow to preserve memory allocations (Required to properly wake up from sleep mode). Not working with PRIME.
options nvidia NVreg_EnableS0ixPowerManagement=1
# An option for saving video memory. Uses s2idle power saving mode.

options nvidia NVreg_TemporaryFilePath=/var/tmp
#
# An alternative option for saving video memory. Turn on if s2idle energy saving mode is not working.

options nvidia NVreg_UsePageAttributeTable=1
#
# NVreg_UsePageAttributeTable=1 (Default 0) - Activating the better
# memory management method (PAT). The PAT method creates a partition type table
# at a specific address mapped inside the register and utilizes the memory architecture
# and instruction set more efficiently and faster.
# If your system can support this feature, it should improve CPU performance.

#options nvidia NVreg_InitializeSystemMemoryAllocations=0
#
# NVreg_InitializeSystemMemoryAllocations=0 (Default 1) - Disables
# clearing system memory allocation before using it for the GPU.
# Potentially improves performance, but at the cost of increased security risks.
# Write "options nvidia NVreg_InitializeSystemMemoryAllocations=1" in /etc/modprobe.d/nvidia.conf,
# if you want to return the default value.
# Note: It is possible to use more video memory (?)

options nvidia NVreg_EnableStreamMemOPs=1
#
# CUDA Stream Memory Operations in user-mode applications.


#options nvidia NVreg_RegistryDwords="EnableBrightnessControl=1 PowerMizerEnable=0x1;PerfLevelSrc=0x3322;PowerMizerDefaultAC=0x1"
#
# Power Setup (PowerMizer). Maximum performance.


options nvidia NVreg_DynamicPowerManagement=0x02
#
# Enable complete power management. From:
# file:///usr/share/doc/nvidia-driver/html/powermanagement.html

#options nvidia NVreg_EnableResizableBar=1
#
#https://www.nvidia.com/en-us/geforce/news/geforce-rtx-30-series-resizable-bar-support/
# Working just for RTX 3xxx series and CPU since amd ryzen 5xxx\ intel 10th gen
# So you can enable it, if you have it
#(also, you need enable resizable bar in the motherboard)

# options nvidia NVreg_EnableGpuFirmware=1
# (May crash suspend. Need to check)
# GSP (GPU System Processor) - this is a special chip which is present on NVIDIA video cards starting
# from Turing and above, which offloads GPU initialization and control tasks, which are usually
#performed on CPU. This should improve performance and reduce the load on the CPU.
#WARNING: I strongly suggest forcing the use of GSP firmware only on the most recent driver, as the first
#releases with its support may contain certain problems. Only starting from 530 you will have support
#for suspend and resume when using GSP firmware. This feature can also work badly on PRIME configurations,
#so please check dmesg logs for errors if you want to use this.

#blacklist nouveau
#alias nouveau off
#
# Nouveau must be blacklisted here as well beside from the initrd to avoid a
# delayed loading (for example on Optimus laptops where the Nvidia card is not
# driving the main display).
# But they allreay blacklisted nvidia-utils package in /usr/lib/modprobe.d/nvidia-utils.conf

options nvidia_drm modeset=1

options nvidia_drm fbdev=1
# This options unlock new nvidia framebuffer
_EOF_
```

Generate scheme for loading
```bash
mkinitcpio -P
```

Turn on `nvidia`
```bash
systemctl enable nvidia-suspend.service nvidia-hibernate.service nvidia-resume.service nvidia-persistenced.service
```

### <span style="color:#fdffb6">Boot loader</span>
```bash
bootctl install --path=/boot
cd /boot/loader
nvim loader.conf

# Insert config into loader.conf:
default  arch.conf
timeout  3
console-mode max
editor   no

# Create boot config
cd /boot/loader/entries
nvim arch.conf

# Insert into  arch.conf:
# UUID can be checked by blkid
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options apparmor=1 security=apparmor resume=/dev/mapper/swap


nvim /boot/loader/entries/arch-fallback.conf
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux
initrd  /initramfs-linux-fallback.img
```
There are no other `options` here, because we changed `/etc/crypttab.initramfs`.More: https://wiki.archlinux.org/title/Dm-crypt/System_configuration#Using_systemd-cryptsetup-generator

Turn on auto update
```bash
systemctl enable systemd-boot-update.service
```

## <span style="color:#ffd6a5">Security settings</span>
### <span style="color:#fdffb6">AppArmor</span>
Access control to apps
```bash
sudo -u oshitaka paru -S apparmor
```

Start service
```bash
systemctl enable apparmor.service
```

Create file
```bash
touch /etc/kernel/cmdline
```

Edit kernel parameters (something wrong with command)
```bash
nvim /etc/kernel/cmdline

sm=landlock,lockdown,yama,integrity,apparmor,bpf audit=1 audit_backlog_limit=8192
```

```bash
sed -i -e 's/$/ lsm=landlock,lockdown,yama,integrity,apparmor,bpf audit=1 audit_backlog_limit=8192/' /etc/kernel/cmdline
```

>[!note]
>audit_backlog_limit=8192 used to avid error: '''bash audit: kauditd hold queue overflow '''

>[!bug]
>If service don't run on start, edit `/boot/loader/enties/arch.conf` adding string `options apparmor=1 security=apparmor`
>https://bbs.archlinux.org/viewtopic.php?id=251807
>https://www.reddit.com/r/archlinux/comments/1cfva3g/apparmor_early_loading/
#### <span style="color:#caffbf">Apparmor notification</span> 
Need additional packages
```bash
sudo -u oshitaka paru -S --needed python-notify2 python-psutil
```

Add audit group to config
```bash
sed '/log_group = root/s/root/wheel/' -i /etc/audit/auditd.conf
```

Crate autostart
```bash
sudo -u oshitaka mkdir -p /home/oshitaka/.config/autostart
```

Add .desktop for notifications
```bash
sudo -u oshitaka bash -c 'cat << _EOF_ >> /home/oshitaka/.config/autostart/apparmor-notify.desktop
[Desktop Entry]
Type=Application
Name=AppArmor Notify
Comment=Receive on screen notifications of AppArmor denials
TryExec=aa-notify
Exec=aa-notify -p -s 1 -w 60 -f /var/log/audit/audit.log
StartupNotify=false
NoDisplay=true
_EOF_'
```

Start audit
```bash
systemctl enable auditd.service
```

Uncomment write-cache
```bash
sed '/write-cache/s/^#//' -i /etc/apparmor/parser.conf
```

### <span style="color:#fdffb6">Hook for microcode</span>
Create directory
```bash
mkdir /etc/pacman.d/hooks
```

Make hook
```bash
cat << _EOF_ > /etc/pacman.d/hooks/ucode.hook
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
# Change to appropriate microcode package
Target=intel-ucode
# Change the linux part above and in the Exec line if a different kernel is used
Target=linux*

[Action]
Description=Update Microcode module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case \$trg in linux*) exit 0; esac; done; /usr/bin/mkinitcpio -P'
_EOF_
```

## <span style="color:#ffd6a5">XDG-USER-DIRS</span>
Create default directories

Install
```bash
sudo -u oshitaka paru -S --needed xdg-user-dirs
```

Create directories
```bash
sudo -u oshitaka mkdir -p /home/oshitaka/.config && \
sudo -u oshitaka cat << _EOF_ > /home/oshitaka/.config/user-dirs.dirs
# This file is written by xdg-user-dirs-update
# If you want to change or add directories, just edit the line you're
# interested in. All local changes will be retained on the next run.
# Format is XDG_xxx_DIR="$HOME/yyy", where yyy is a shell-escaped
# homedir-relative path, or XDG_xxx_DIR="/yyy", where /yyy is an
# absolute path. No other format is supported.
#
XDG_DESKTOP_DIR="/mnt/sdb/Desktop"
XDG_DOWNLOAD_DIR="/mnt/sdb/Downloads"
XDG_DOCUMENTS_DIR="/mnt/sdb/Documents"
XDG_TEMPLATES_DIR="$HOME/Templates"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_MUSIC_DIR="/mnt/sdb/Music"
XDG_PICTURES_DIR="/mnt/sdb/Pictures"
XDG_VIDEOS_DIR="/mnt/sdb/Video"
_EOF_
```

## <span style="color:#ffd6a5">TRIM</span>
If TRIM  supported by SSD (см. [[#<span style="color ffd6a5">Подготовка дисков</span>]]) turn it on
```bash
systemctl enable fstrim.timer
```

## <span style="color:#ffd6a5">Restart</span>
Exit the chroot environment by typing 
```bash
exit
``` 
or pressing `Ctrl+d`. And then
```bash
umount -R /mnt
```

Finally, restart the machine by typing 
```bash
reboot
``` 
Any partitions still mounted will be automatically unmounted by `systemd`. Remember to remove the installation medium and then login into the new system with the root account.

## <span style="color:#ffd6a5">Other settings</span>
### <span style="color:#fdffb6">Security</span>
#### <span style="color:#caffbf">Check Apparmor</span> 
Type `aa-notify`. Should be at least one result
```bash
pgrep -ax aa-notify
```

#### <span style="color:#caffbf">Firewall</span> 
Install
```bash
sudo pacman -S ufw
```

Default settings
```bash
sudo ufw default deny && \
sudo ufw allow from 192.168.0.0/24 && \
sudo ufw limit ssh
```

Run
```bash
sudo ufw enable && \
sudo systemctl enable --now ufw
```

#### <span style="color:#caffbf">Antivirus</span> 
Install
```bash
paru -S clamav
```

Mirror manage
```bash
sudo bash -c "sed '/DatabaseMirror database.clamav.net/s/^/#/' -i /etc/clamav/freshclam.conf"  &&\
sudo bash -c "sed '/DatabaseMirror /a DatabaseMirror https://pivotal-clamav-mirror.s3.amazonaws.com' -i /etc/clamav/freshclam.conf"
```

Run
```bash
sudo systemctl enable --now clamav-daemon.service
```

Change the ownership of directory to be able to update database
```bash
sudo chown -R oshitaka /var/lib/clamav
```

More about ClamAV: https://wiki.archlinux.org/title/ClamAV

#### <span style="color:#caffbf">USB guard</span> 
Install
```bash
paru -S usbguard 
```

Allow all connected devices
```bash
sudo bash -c "usbguard generate-policy > /etc/usbguard/rules.conf"
```

Run
```bash
sudo systemctl enable usbguard.service
```

Add rights to user
```bash
usbguard add-user -d list oshitaka
```

More about USBGuard: https://wiki.archlinux.org/title/USBGuard

#### <span style="color:#caffbf">Rights for important files</span> 
```bash
sudo chmod 600 /etc/cron.deny &&\
sudo chmod 644 /etc/group &&\
sudo chmod 644 /etc/group- &&\
sudo chmod 644 /etc/issue &&\
sudo chmod 644 /etc/passwd &&\
sudo chmod 644 /etc/passwd- &&\
sudo chmod 600 /etc/ssh/sshd_config &&\
sudo chmod 700 /root/.ssh &&\
sudo chmod 700 /etc/cron.d &&\
sudo chmod 700 /etc/cron.daily &&\
sudo chmod 700 /etc/cron.hourly &&\
sudo chmod 700 /etc/cron.weekly &&\
sudo chmod 700 /etc/cron.monthly &&\
sudo chmod 600 /etc/cron.deny &&\
sudo chmod 644 /etc/group &&\
sudo chmod 644 /etc/group- &&\
sudo chmod 644 /etc/issue &&\
sudo chmod 644 /etc/passwd &&\
sudo chmod 644 /etc/passwd- &&\
sudo chmod 600 /etc/ssh/sshd_config &&\
sudo chmod 700 /root/.ssh &&\
sudo chmod 700 /etc/cron.d &&\
sudo chmod 700 /etc/cron.daily &&\
sudo chmod 700 /etc/cron.hourly &&\
sudo chmod 700 /etc/cron.weekly &&\
sudo chmod 700 /etc/cron.monthly
```

#### <span style="color:#caffbf">Password politics</span> 
```bash
sudo bash -c 'cat << _EOF_ > "/etc/pam.d/passwd"
#%PAM-1.0
password required pam_pwquality.so retry=2 minlen=8 difok=6 dcredit=-1 ucredit=-1 lcredit=-1 [badwords=myservice mydomain] enforce_for_root
password required pam_unix.so use_authtok sha512 shadow
_EOF_'
```

>[!Note]
>When password will change next time this rules apply: 
>- prompt 2 times for password in case of an error (retry option)
>- 10 characters minimum length (minlen option)
>- at least 6 characters should be different from old password when entering a new one (difok option)
>- at least 1 digit (dcredit option)
>- at least 1 uppercase (ucredit option)
>- at least 1 lowercase (lcredit option)
>- at least 1 other character (ocredit option)
>- cannot contain the words "myservice" and "mydomain"
>- enforce the policy for root


### <span style="color:#fdffb6">And more packages</span>
This is CLI  packages, without GUI
```bash
sudo pacman -S --needed neofetch btop kitty yazi acpi acpid wireguard-tools tmux cronie dbus-broker rng-tools bluez bluez-utils curl wget nmap wl-clipboard fd fzf ouch
```

Explanations:
- <span style="color:#fdffb6">cronie</span> - time deamon hepls to keep system clean from trash.
- <span style="color:#fdffb6">dbus-broker</span> - Its goal is to provide high performance and reliability while maintaining compatibility with the D-Bus reference implementation. Provides slightly faster communication with the video card via PCIe
- <span style="color:#fdffb6">rng-tools</span> - monitors the entropy of the system, but unlike haveged, it is already through a hardware timer. Necessary to speed up system startup at high performance _systemd-analyze blame_ (more 1 sec)
- <span style="color:#fdffb6">ouch </span>- handy archiver
Info about cronie, dbus-broker, rng-tools took from here: https://ventureo.codeberg.page/v2022.07.01/source/generic-system-acceleration.html

Run services
```bash
sudo systemctl enable --now dbus-broker.service && \
sudo systemctl enable --now cronie.service && \
sudo systemctl enable --now rngd && \
sudo systemctl enable --now bluetooth.service
```

Edit cron 
```bash
sudo EDITOR=nvim crontab -e
```
Insert `15 10 * * sun /sbin/pacman -Scc --noconfirm`. Now system will automatically clean up itself once a week at 15:00 on Sundays

### <span style="color:#fdffb6">Check systemd services</span>
```bash
sudo systemctl --failed
```

---
# <span style="color:#ffadad">Customization</span>
Now time to tweak some features of laptop in [[Arch. 3-tweaks & troubleshooting]]

---
