# My Personal Arch installation guide

This is a personal guide to install Arch linux with minimal Plasma DE. I must warn that this install is super minimal and does not contain many KDE apps like Kmail, Kalendar and KDE Connect!

I am not a professional by any means and I made this quide just for personal use so I highly recommend you to read the official [`Installation guide - Arch wiki`](https://wiki.archlinux.org/index.php/Installation_guide)!

# Begin to install base system

Ensure the system clock is accurate:

```
timedatectl set-ntp true
```

To check the service status, use `timedatectl status`

## Load keys
`(skip this if you use US keymap)`

`localectl list-keymaps` <\-\- list all keymaps

`loadkeys keymap (keymap)` <--load keymap for current session

## Wifi
`(skip this if you use ethernet)`

Connect to WiFi using iwctl. Just enter this command and follow the instructions:

```
iwctl
```

List all your wireless interfaces:

```
device list
```

You need to select the preferred one.

Scan for available network using the command below:

```
station wlan0 scan
```

See the connections available:

```
station wlan0 get-networks
```

Connect to your target Wi-Fi:

```
station wlan0 connect "Name of Network/WiFi"
```

After entering the password:
exit (or quit)

Test internet:

```
ping -c4 google.com
```

## Partitioning

**List your disks**

`fdisk -l` (or lsblk)(will show all of your disks as "/dev/sda or /dev/nvme0n1 etc.")
![Screenshot_20220816_204521](https://user-images.githubusercontent.com/95308907/184945038-16875f8b-dd70-459e-91ee-82784ae5caa3.png)

**If necessary, you can remove the discs**
sudo wipefs -all /dev/(disk name)

**Make root and EFI-partitions**

Choose the disk you want to use

Disk is usually `cfdisk /dev/sda` or `cfdisk /dev/nvme0n1`
![Select-Arch-Linux-Installation-Disk](https://user-images.githubusercontent.com/95308907/184943576-cea39914-feac-4672-8f0e-3467130a27dd.png)

**Create at least 300M EFI system. For 2 kernel 512M or more. You can read about kernels here** 
[`Kernel - Arch wiki`](https://wiki.archlinux.org/title/Kernel).
Press enter key, change partion "Type" from the bottom menu and choose "EFI System partition"

![EFI-System-Type](https://user-images.githubusercontent.com/95308907/184942340-c3c32914-614c-4c08-97bd-5d6dd4e77adb.png)


![Select-EFI-System](https://user-images.githubusercontent.com/95308907/184942636-c29c7650-8b30-4424-bda7-2fdd57efbe11.png)


**Create root partition**

For /(root) partition use the following configuration: "New" -> Size: rest of free space -> change "Type" to Linux filesystem.

After you review Partition Table select "Write", answer with "yes" in order to apply


**Format partitions**

```
mkfs.fat -F32 /dev/(efi partition)
```
```
mkfs.ext4(or other file system) /dev/(root partion)
```

**Mount partitions**

```
mount /dev/(root partition name) /mnt

mkdir -p /mnt/boot/efi

mount /dev/(efi partition name) /mnt/boot/efi
```

## Install base system

```
pacstrap /mnt base base-devel linux linux-firmware linux-headers vim nano
```

Make fstab:

```
genfstab -U /mnt >> /mnt/etc/fstab
```

## Enter your systen using chroot


Chroot your system:

```
arch-chroot /mnt
```


## Set keyboard language and time

```
 localectl list-keymaps <-- list all keymaps
 loadkeys keymap (keymap) <--load keymap for current session
```


Set persistent keymap:

```
nano /etc/vconsole.conf
KEYMAP=(keymap)
```


List available zones:

```
timedatectl list-timezones
```

Set timezone
Use you own Country/City!

```
ln -sf /usr/share/zoneinfo/Europe/Helsinki /etc/localtime
```


Run hwclock to generate /etc/adjtime:

```
hwclock --systohc
```


Set your locale:

```
nano /etc/locale.gen*
```

```
(Uncomment en_US.UTF-8 UTF-8)
```

Run locale-gen:
```
locale-gen
```
Set your locale to
*/etc/locale.conf*
```
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```


Set hostname:
```
echo "yourhostname" > /etc/hostname
```


Set up host file (not necessary anymore if you don't need hostfile)
```
nano /etc/hosts
```
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	(myhostname)
```


## Install reguired packages

```
pacman -S networkmanager network-manager-applet intel-ucode(or amd-ucode) wireless_tools grub efibootmgr bluez bluez-utils power-profiles-daemon
```

Enable Network and bluetooth:

```
systemctl enable NetworkManager
systemctl enable bluetooth.service
```


## Set your user

```
passwd    <-- password for root

useradd -m -G wheel (username)

passwd (username)    <-- passwor for user

EDITOR=nano visudo

%wheel ALL=(ALL:ALL) ALL)    <--(uncomment)
```
In nano after uncoment:
Ctrl + O    <-- save
Ctrl + X    <-- quit

## GRUB install

Install GRUB:

```
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
```

Save GRUB:

```
grub-mkconfig -o /boot/grub/grub.cfg
```


**Reboot and log in**
```
exit

umount -l /mnt

type: reboot

Enter your username and password
```

## After reboot, install Plasma, Display, Sound
`(skip this if you use ethernet)`

Connect to WiFi:

First way
```
nmcli device(or d) wifi connect SDT-wifi(name wifi) password password(your password)
```
The second way using the utility
```
nmtui
```

GPU driver:

```
sudo pacman -S xf86-video-amdgpu (xf86-video-your gpu type, see the wiki)
```

Pipewire audio drivers:

```
sudo pacman -S pipewire pipewire-alsa pipewire-jack pipewire-pulse wireplumber gst-plugin-pipewire sof-firmware alsa-ucm-conf --needed
```

KDE Desktop Enverioment and Software:

```
sudo pacman -S  plasma thunar ark konsole gwenview powerdevil ffmpegthumbs firefox kate spectacle --needed
```

Enable sddm:

```
sudo systemctl enable sddm
```

Enable trim:

```
sudo systemctl enable fstrim.timer
```
## Reboot and enjoy!

**Additionally**

Mirrors installation:
```
sudo pacman -S reflector
```
```
sudo reflector --verbose --country 'Germany'(your country) --protocol https -l 10 --sort rate --save /etc/pacman.d/mirrorlist
```
```
sudo nano /etc/pacman.conf
```
ParallelDownloads=5        <--(uncomment)

**Installation AUR**
```
git clone https://aur.archlinux.org/paru.git

cd paru

makepkg -si

paru
```
**Installation YAY**
```
git clone https://aur.archlinux.org/yay.git

makepkg -sric
```
