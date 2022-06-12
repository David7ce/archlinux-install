# Arch Linux Guide

## Contents:
- [Arch linux installation](#Arch-linux-installation)
	- [Pre-installation](#Pre-installation)
	- [Installation](#Installation)
	- [Configure the system](#Configure-the-system)
	- [Post-installation](#Post-installation)
	- [Reboot](#Reboot)
- [Code block](#Code-block)
- [App installation](#App-installation)
- [References](#References)

# Arch linux installation
## Pre-installation
- Verify signature
```bash
$ gpg --keyserver-options auto-key-retrieve --verify archlinux-_version_-x86_64.iso.sig
```

Alternatively, from an existing Arch Linux installation run:
```bash
$ pacman-key -v archlinux-_version_-x86_64.iso.sig
```

- Boot the live environment
1.  Point the current boot device to the one which has the Arch Linux installation medium. Typically it is achieved by pressing a key during the [POST](https://en.wikipedia.org/wiki/Power-on_self_test "wikipedia:Power-on self test") phase, as indicated on the splash screen. Refer to your motherboard's manual for details.
2.  When the installation medium's boot loader menu appears, select _Arch Linux install medium_ and press `Enter` to enter the installation environment.
        **Tip:** The installation image uses [systemd-boot](https://wiki.archlinux.org/title/Systemd-boot "Systemd-boot") for booting in UEFI mode and [syslinux](https://wiki.archlinux.org/title/Syslinux "Syslinux") for booting in BIOS mode. See [README.bootparams](https://gitlab.archlinux.org/mkinitcpio/mkinitcpio-archiso/blob/master/docs/README.bootparams) for a list of [boot parameters](https://wiki.archlinux.org/title/Kernel_parameters#Configuration "Kernel parameters").
    3.  You will be logged in on the first [virtual console](https://en.wikipedia.org/wiki/Virtual_console "wikipedia:Virtual console") as the root user, and presented with a [Zsh](https://wiki.archlinux.org/title/Zsh "Zsh") shell prompt.
To switch to a different console—for example, to view this guide with [Lynx](https://lynx.invisible-island.net/lynx_help/Lynx_users_guide.html) alongside the installation—use the `Alt+_arrow_` [shortcut](https://wiki.archlinux.org/title/Keyboard_shortcuts "Keyboard shortcuts"). To [edit](https://wiki.archlinux.org/title/Textedit "Textedit") configuration files, [mcedit(1)](https://man.archlinux.org/man/mcedit.1), [nano](https://wiki.archlinux.org/title/Nano#Usage "Nano") and [vim](https://wiki.archlinux.org/title/Vim#Usage "Vim") are available. See [packages.x86_64](https://gitlab.archlinux.org/archlinux/archiso/-/blob/master/configs/releng/packages.x86_64) for a list of the packages included in the installation medium.

- Set the console keyboard layout
```bash
ls /usr/share/kbd/keymaps/**/*.map.gz
loadkeys es
loadkeys de-latin1
```

- Verify the boot mode
To verify the boot mode, list the [efivars](https://wiki.archlinux.org/title/Efivars "Efivars") directory:
```bash
 # ls /sys/firmware/efi/efivars
```

If the command shows the directory without error, then the system is booted in UEFI mode. If the directory does not exist, the system may be booted in [BIOS](https://en.wikipedia.org/wiki/BIOS "wikipedia:BIOS") (or [CSM](https://en.wikipedia.org/wiki/Compatibility_Support_Module "wikipedia:Compatibility Support Module")) mode. If the system did not boot in the mode you desired, refer to your motherboard's manual.

- Connect internet
To set up a network connection in the live environment, go through the following steps:

-   Ensure your [network interface](https://wiki.archlinux.org/title/Network_interface "Network interface") is listed and enabled, for example with [ip-link(8)](https://man.archlinux.org/man/ip-link.8):
 ```bash
# ip link
```
-   For wireless and WWAN, make sure the card is not blocked with [rfkill](https://wiki.archlinux.org/title/Rfkill "Rfkill").
-   Connect to the network:
    -   Ethernet—plug in the cable.
    -   Wi-Fi—authenticate to the wireless network using [iwctl](https://wiki.archlinux.org/title/Iwctl "Iwctl").
    -   Mobile broadband modem—connect to the mobile network with the [mmcli](https://wiki.archlinux.org/title/Mmcli "Mmcli") utility.
-   Configure your network connection:
    -   [DHCP](https://wiki.archlinux.org/title/DHCP "DHCP"): dynamic IP address and DNS server assignment (provided by [systemd-networkd](https://wiki.archlinux.org/title/Systemd-networkd "Systemd-networkd") and [systemd-resolved](https://wiki.archlinux.org/title/Systemd-resolved "Systemd-resolved")) should work out of the box for Ethernet, WLAN, and WWAN network interfaces.
    -   Static IP address: follow [Network configuration#Static IP address](https://wiki.archlinux.org/title/Network_configuration#Static_IP_address "Network configuration").
-   The connection may be verified with [ping](https://en.wikipedia.org/wiki/ping_(networking_utility) "wikipedia:ping (networking utility)"):
 ```bash
# ping archlinux.com
```

- Update the system clock
```bash
# timedatectl set-ntp true
```
To check the service status, use `timedatectl status`.

- Partition the disks (EFI, SWAP, system)
When recognized by the live system, disks are assigned to a [block device](https://wiki.archlinux.org/title/Block_device "Block device") such as `/dev/sda`, `/dev/nvme0n1` or `/dev/mmcblk0`. To identify these devices, use [lsblk](https://wiki.archlinux.org/title/Lsblk "Lsblk") or _fdisk_. Also you can use _cfdsik_ to create partitions.
```bash
# fdisk -l
```
Results ending in `rom`, `loop` or `airoot` may be ignored. The following [partitions](https://wiki.archlinux.org/title/Partition "Partition") are **required** for a chosen device:
- One partition for the [root directory](https://en.wikipedia.org/wiki/Root_directory "wikipedia:Root directory") `/`.
- For booting in [UEFI](https://wiki.archlinux.org/title/UEFI "UEFI") mode: an [EFI system partition](https://wiki.archlinux.org/title/EFI_system_partition "EFI system partition").
If you want to create any stacked block devices for [LVM](https://wiki.archlinux.org/title/LVM "LVM"), [system encryption](https://wiki.archlinux.org/title/Dm-crypt "Dm-crypt") or [RAID](https://wiki.archlinux.org/title/RAID "RAID"), do it now. Use [fdisk](https://wiki.archlinux.org/title/Fdisk "Fdisk") or [parted](https://wiki.archlinux.org/title/Parted "Parted") to modify partition tables. For example:

```bash
# fdisk dev/sda    # /dev/the_disk_to_be_partitioned
g    # gtp
n    # new partition
+550M    # EFI partition

n # new partition
+2G    # SWAP partition, not neccesary

t    # rest

1    # Partition 1 (EFI)
19   # Partition 2 (swap)
```

- Example layouts
| Mount point | Partition                   | Partition type        | Size                    |
| ----------- | --------------------------- | --------------------- | ----------------------- |
| `/mnt/boot` | `/dev/efi_system_partition` | EFI system partition  | ~300 MB                 |
| `[SWAP]`    | `/dev/swap_partition`       | Linux swap            | more than 512 MB        |
| `/mnt`      | `/dev/root_partition`       | Linux x86-64 root (/) | Remainder of the device |

- Format partitions
- Format Ext4 file system:
```bash
# mkfs.fat -F 32 /dev/sda1
```
- Format swap partition:
```bash
# mkswap /dev/sda2
# swapon /dev/sda2
```
- EFI system partition, format it to FAT32:
```bash
# mkfs.ext4 /dev/sda3
```

- Mount the file systems
- Mount the root volume to `/mnt` (mount /dev/root_partition /mnt)
```bash
# mount /dev/sda3 /mnt
```

- For UEFI systems, mount the EFI system partition:
```console
# mount --mkdir /dev/_efi_system_partition_ /mnt/boot
```

- If you created a [swap](https://wiki.archlinux.org/title/Swap "Swap") volume, enable it with [swapon(8)](https://man.archlinux.org/man/swapon.8):
```console
# swapon /dev/swap_partition
```

## Installation
- List of servers:
```bash
# vim /etc/pacman.d/mirrorlist
```
- Install essential packages:
```bash
# pacstrap /mnt base linux linux-firmware
```

## Configure the system
- Fstab
```bash
# genfstab -U /mnt >> /mnt/etc/fstab
```

- Chroot
```bash
# arch-chroot /mnt
```

- Timezone
```bash
# ls /usr/share/zoneinfo/
# ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
```

- Run [hwclock(8)](https://man.archlinux.org/man/hwclock.8) to generate `/etc/adjtime`:
```bash
# hwclock --systohc
# pacman -S nano
# nano /etc/locale.gem  # (uncomment en-US or es-ES)
```

- Localization
```bash
# locale-gen
```

- Network configuration
```bash
# nano /etc/hostname
# passwd
```

- Add user
```bash
# useradd -m d7
# passwd d7
```

- Boot loader
Choose and install a Linux-capable [boot loader](https://wiki.archlinux.org/title/Boot_loader "Boot loader"). If you have an Intel or AMD CPU, enable [microcode](https://wiki.archlinux.org/title/Microcode "Microcode") updates in addition.
```bash
# usermod -aG wheel,audio,video,optical,storage d7
# pacman -S sudo

EDITOR=nano visudo
(uncomment edit wheel=(ALL) ALL)


# pacaman -Sy grub
# pacman -Sy efibootmgr dosstools os-prober mtools

# mkdir /boot/EFI
# mount /dev/sda1 /boot/EFI/
# grub-install --target=x86_64-efi --bootloader-id=grub-uefi --recheck

# grub-mkconfig -o /boot/grub/grub.cfg
```

## Post-installation
See [General recommendations](https://wiki.archlinux.org/title/General_recommendations "General recommendations") for system management directions and post-installation tutorials (like creating unprivileged user accounts, setting up a graphical user interface, sound or a touchpad).

For a list of applications that may be of interest, see [List of applications](https://wiki.archlinux.org/title/List_of_applications "List of applications").
- Install with pacman: netoworkmanager, git, neofetch)
```bash
# pacman -S networkmanager vim
# pacman -S git
# pacman -S neofetch
# sudo pacman -S xf86-video-fbdev xorg xorg-xinit nitrogen picon vim git neofetch alacrity firefox 
```

- Terminal emulator and window manager, enabling AUR installing "yay"
```bash
$ sudo pacman -S base-devel
$ git clone https://aur.archlinux.org/yay-git.git
$ ls -la
$ makepkg -si
$ yay -S dwm-distrotube-git st-distrotube-git dmenu-distrotube-git nerd-fonts-mononoki
```

```bash
$ cd
$ cp /etc//X11/xinit/xinitrc /home/dt/.xinitrc
$ vim .xinitrc 

remove 4 lines and write:
nitrogen --restore &
picom &
exec dwm
```

- Run from git repository (GitLab, GitHub) or from a bash script

- Systemctl enable
```bash
# systemctl enable NetworkManager
```

## Reboot
Exit the chroot environment by typing `exit` or pressing `Ctrl+d`. Optionally manually unmount all the partitions with `umount -R /mnt`: this allows noticing any "busy" partitions, and finding the cause with [fuser(1)](https://man.archlinux.org/man/fuser.1).

Finally, restart the machine by typing `reboot`: any partitions still mounted will be automatically unmounted by _systemd_. Remember to remove the installation medium and then login into the new system with the root account.

```bash
# exit
# umount -R /mnt
# reboot

# startx
```


---

# Code block

- Download bittorrent ArchLinux
- Open VM (Virtual Box or VMWare), create Virtual Machine, create VDI, change boot order (hard disk first), add .iso file

```bash
root@archiso ~ # loadkeys es

root@archiso ~ # ping example.com

root@archiso ~ # timedatectl set-ntp true

root@archiso ~ # lsblk

root@archiso ~ # cfdisk /dev/sda
	level type: gpt, dos*, sgi, sun
		/dev/sda1 128M (primary bootable) -> Boot partition
		/dev/sda2 rest memory (primary) -> File system partition
		Write changes and Quit

root@archiso ~ # mkfs.ext4 /dev/sda1
root@archiso ~ # mkfs.ext4 /dev/sda2

root@archiso ~ # mount /dev/sda2 /mnt

root@archiso ~ # mkdir /mnt/boot
root@archiso ~ # mount /dev/sda1 /mnt/boot
root@archiso ~ # lsblk

root@archiso ~ # pacstrap /mnt base base-devel linux linux-firmware vim

root@archiso ~ # genfstab /mnt
root@archiso ~ # genfstab -U /mnt >> /mnt/etc/fstab
root@archiso ~ # genfstab /mnt

root@archiso ~ # arch-chroot /mnt /bin/bash 

[root@archiso /] # ls

[root@archiso /] # pacman -S networkmanager grub
[root@archiso /] # systemctl enable NetworkManager
[root@archiso /] # grub-install /dev/sda
[root@archiso /] # grub-mkcofig -o /boot/grub/grub.cfg

[root@archiso /] # passwd

[root@archiso /] # vim /etc/locale.gen
	en_US.UTF-8 UTF-8
	en_US ISO-8859-1
[root@archiso /] # locale-gen
[root@archiso /] # vim /etc/locale.conf
	LANG=en-US.UTF-8
	:wq

[root@archiso /] # vim /etc/hostname
	- MachineName

[root@archiso /] # ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime

[root@archiso /] # exit

root@archiso ~ # umount -R /mnt
root@archiso ~ # reboot

Arch Linux 5.6.2-arch1-2 (tty1)
MachineName login: root
Password: ...
[root@MachineName ] # pacman -S neofetch
```

---

## App installation

- List of Arch Linux apps: https://wiki.archlinux.org/title/List_of_applications

- Install with pacman multiple apps
```bash
pacman -Sy alacrity konsole dolphin lf ranger freefilesync rsync audacity bleachbit blender davinci-resolve deepin-camera digikam foobar gimp kdenlive lmms emacs-org-mode obsidian spek vlc openrgb handbrake-cli vscodium-bin peazip-gtk2-bin base-devel git godot librewolf ungoogled-chromium qbitorrent discord telegram-desktop virtualbox-host-modules-arch
```

- Install OBS with flatpak:
```bash
sudo pacman su

sudo pacman -Sy v4l2loopback-dkms

sudo rm -r /var/lib/pacman/db.lck

sudo pacman -Sy flatpak

flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

flatpak install flathub com.obsproject.Studio
flatpak run com.obsproject.Studio

flatpak install flathub com.usebottles.bottles
flatpak run com.usebottles.bottles

```

- Install Google Earth pro with trizen:
```bash
sudo pacman -S git base-devel

git clone https://aur.archlinux.org/trizen.git

cd trizen
makepkg -sri

trizen -S google-earth-pro
```

---

# References
- [Arch Linux installation](https://wiki.archlinux.org/title/Installation_guide)
- [Dotfiles - DT](https://gitlab.com/dwt1/dotfiles)
- [Dotfiles - Antonio Sarosi](https://github.com/antoniosarosi/dotfiles)
- [Arch Linux installation Guide](https://www.youtube.com/watch?v=rUEnS1zj1DM)
- [Arch Linux installation 2020](https://www.youtube.com/watch?v=PQgyW10xD8s)
- [Pasando de Noob a Pro Arch Linux 20'](https://www.youtube.com/watch?v=VoJOOx2WLy0)
