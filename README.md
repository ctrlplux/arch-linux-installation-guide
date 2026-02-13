<h1 align="center">Arch Linux Installation Guide<br><br></h1>  

### Getting Started

Welcome to the Arch Linux with KDE Installation Guide!

This guide provides you with a step-by-step walkthrough of installing Arch Linux along with KDE. It has been carefully created based on my own experience installing Arch Linux. This guide aims to make your installation process as smooth as possible.

To begin your Arch Linux installation journey, please follow the step-by-step instructions provided below.

---

### What is Arch Linux?

Arch Linux is arguably one of the best Linux distributions out there. It’s free and open source, offering x86_64 CPU architecture. Arch is also a rolling release, which means you never have to update your system with major releases. Every update through its package manager (Pacman) is the latest and greatest bleeding edge.

The biggest lament is that Arch Linux is agnostic when it comes to window management or installers, so newcomers to the Linux world often steer clear of installing Arch on their machines. But today I will show you how easy it really is.

---

### ISO Writing

First off, we need to grab an ISO copy of Arch Linux and burn it to a USB drive for installation.

Visit https://www.archlinux.org/download/ and download the torrent.

Next, we want to write the ISO file to a USB drive. It is usually ideal to have at least a 4–8 GB drive to do this.

Head over to https://rufus.ie/ and download Rufus. Open Rufus, insert your USB drive, and hit “SELECT” to choose the Arch Linux ISO (archlinux-20XX.XX.XX-x86_64.iso) you just downloaded.

<img width="471" height="577" alt="image" src="https://github.com/user-attachments/assets/16a30da4-d9eb-4d19-bf02-3f99eff85eef" />  

It should look like this. Then hit “START” to write. Once you have your media ready, go to the machine you are installing Arch to and boot up from your USB in the BIOS.

*Note:* **ISO Image Writer** for Linux and **EtchDroid** for mobile.

---

### Let’s dive into the installation

Mount the ISO media, then boot into UEFI. Restart your computer and immediately press the dedicated hotkey (commonly F2, F12, Del, or Esc) before Windows loads. In boot options, select the USB disk (or) make the USB disk the primary boot device, then save and exit.

Select **Arch Linux install medium**.

Once Arch Linux has booted to a prompt on the machine you are installing it to, we need to connect to a wired connection or Wi-Fi. Let’s connect to Wi-Fi. Fire off the following commands:

```
iwctl

device list

station wlan0 connect "WIFI_NAME"

exit
```

---

Before creating partitions, we need to identify the correct disk where Arch Linux will be installed.

Run the following command:

```
lsblk
```

This will list all available storage devices and their partitions.

Example output:

```
NAME        SIZE TYPE MOUNTPOINT
nvme0n1     512G disk
├─nvme0n1p1 100M part
├─nvme0n1p2 200G part
└─nvme0n1p3 312G part
```

In most cases:

* `nvme0n1` → NVMe SSD
* `sda` → SATA HDD/SSD
* `vda` → Virtual machine disk

Make sure you select the correct disk before proceeding, as all data on the selected disk will be erased.

---

⚠️ Backup important data before proceeding.  
We now need to set up EFI, swap, and main partitions. We use cfdisk for this.

```
cfdisk /dev/nvme0n1
```

These are the partitions we need to prepare:

```
Partition    Size    Type    Purpose

EFI          512M    EFI     System        Bootloader
Swap         4–8GB   Linux   swap          RAM extension
Root         Rest    Linux   filesystem    OS + files
```

Write → Quit

---

Let’s build the file systems for our partitions with mkfs.

```
mkfs.fat -F32 /dev/nvme0n1p1  
mkfs.ext4 /dev/nvme0n1p3 
```

Swap using mkfs and mkswap:

```
mkswap /dev/nvme0n1p2
swapon /dev/nvme0n1p2 
```

---

Now it’s time to mount our volumes, EFI, and boot drives.

```
mount /dev/nvme0n1p3 /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot 
```

---

Install the base Arch Linux installation to our new mount.

```
pacstrap /mnt base linux linux-firmware grub efibootmgr nano sudo
```

Generate an fstab file.

```
genfstab -U /mnt >> /mnt/etc/fstab  
```

Time to chroot into the mount point.

```
arch-chroot /mnt
```

---

Set localtime and system clock.

```
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime

hwclock --systohc
```

---

Configure your language.

```
nano /etc/locale.gen  // uncomment your locales, i.e. 'en_US.UTF-8'

locale-gen

echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

---

Change your hostname.

```
echo "yourhostname" > /etc/hostname

nano /etc/hosts
```

Add:

```
127.0.0.1 localhost
::1       localhost
127.0.1.1 yourhostname
```

---

Change the root user password and add a user.

```
passwd

useradd -m -G wheel yourusername

passwd yourusername
```

Enable sudo:

```
EDITOR=nano visudo  // uncomment this line %wheel ALL=(ALL:ALL) ALL
```

---

Now it is time to install the GRUB bootloader.

```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

grub-mkconfig -o /boot/grub/grub.cfg
```

---

KDE is a more customizable and user-interactive desktop environment, especially for beginners and users newly switched from Windows.

```
pacman -S xorg plasma kde-applications-meta sddm
```

Enable login manager:

```
systemctl enable sddm
```

You should also install XFCE, GNOME, and other desktop environments if needed.

---

Recommended to install extras:

```
pacman -S konsole dolphin firefox vlc git base-devel
```

---

Exit, unmount, and reboot the system.

```
exit
umount -R /mnt
reboot
```
 
Once the system has rebooted, you can now log in with your user and password. Done deal!  

<br>
<br>
<br>
<h1 align="center">Post Installation<br><br></h1>  

After logging into KDE for the first time, it is recommended to perform some basic system configuration.


### Enable automatic time synchronization

Activate network time synchronization using NTP:

```
timedatectl set-ntp true
```

You can verify status using:

```
timedatectl status
```

---

#### Update the system

Always update packages after fresh installation:

```
sudo pacman -Syu
```

---

### Install essential utilities

These packages are commonly used for daily operations:

```
sudo pacman -S git wget curl nano vim htop neofetch unzip zip
```

---

### Install filesystem support

Useful for mounting external drives:

```
sudo pacman -S ntfs-3g exfat-utils dosfstools mtools
```

---

### Install AUR Helper (yay)

Arch User Repository (AUR) allows you to install community-maintained packages that are not available in official repositories.

To install, first ensure you have the necessary base tools, then clone and build from the AUR:

```
sudo pacman -S --needed base-devel git

git clone https://aur.archlinux.org/yay.git

cd yay

makepkg -si
```

Once installed, you can install AUR packages using:

```
yay -S package_name  # example: yay -S google-chrome
```

---

### Enable Bluetooth support (optional)

```
sudo pacman -S bluez bluez-utils
sudo systemctl enable bluetooth
```

---

### Install sound utilities (optional)

KDE installs basic audio support, but additional tools are useful:

```
sudo pacman -S pavucontrol pipewire pipewire-alsa pipewire-pulse wireplumber
```

---

### Clean package cache (optional)

```
sudo pacman -Sc
```

---

### Reboot system

After completing post-install configuration:

```
reboot
```

Your Arch Linux KDE environment is now fully ready for daily use.

