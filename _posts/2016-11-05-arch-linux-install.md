---
layout: post
date: 2016-11-05
category: Tech
tags: tech arch linux install guide
description: A guide to installing Arch Linux with LVM configuration and root partition disk encryption.
---
The first distro I ever installed was OpenSUSE, many years ago. At the time I didn't really understand what the different distros meant and so I just installed from a free disk I received with a magazine.

I quickly started playing around with a number of other distros including Fedora, Ubuntu, and Debian. Quickly I realised there was no "right" distro and it was all about choice. Fedora was the distro I ended up using for a number of years... until the introduction of Gnome3 and the GnomeShell. Straight away this wasn't for me, so I sought out alternative Desktop Environments. It was also at this opportunity that I decided to give Arch a try.

Even in the few short years I had used Linux I knew for certain this was for me and therefore I wanted to know more about it. Arch was a good choice because it enables you to get much more hands on.

Arch is not recommended for absolute beginners, but if you want to improve your understanding of how the OS works it is a good distro to play around with. Things may (will) break, but that is all part of the fun!

This guide details the steps I take to quickly run up an Arch install. Security is always something I consider and therefore I encrypt my root filesystem. A lot of the steps detailed below are taken straight out of the [official install guide](https://wiki.archlinux.org/index.php/Installation_guide){:target="_blank" rel="noopener noreferrer"}. These steps are not intended to replace the information on the Arch wiki, this is merly my adoption of the process.

Pre-reqs:
<ul class="sidebar">
<li>- You should be comfortable with using the Linux command line as there is no GUI for this installation</li>
<li>- I use vi for editing the files in this guide. Feel free to use nano if you prefer</li>
<li>- Grab the latest [Arch ISO](https://www.archlinux.org/downloads){:target="_blank" rel="noopener noreferrer"}</li>
<li>- Internet connection</li>
</ul>

Boot from the ISO and select "Boot Arch Linux (x86_64)"

After the ISO has loaded you will be presented with a prompt
```
root@archiso ~ #
```
The first thing I tend to do is load the keymap for my UK keyboard. This is an optional step, but will help if you use certain characters when setting passwords (#, @, /, \, etc).

First show a list of all available QWERTY keymaps, then load the desired map
```
ls -l /usr/share/kbd/keymaps/i386/qwerty/
loadkeys uk
```
The next step is to ensure you have a working internet connection. While this step is also optional I will be using an internet connection later on
```
systemctl start dhcpd
ip a
ping -c 3 archlinux.org
```
_If you are unsure how to get wifi working check out my guide [here]()_

It is advised to enable ntp (Network Time Protocol) to ensure the system clock is accurate
```
timedatectl set-ntp true
timedatectl status
```
I am only using one disk in this guide. Be careful when there are multiple disks attached to you system, make sure you specify the correct device, e.g. sda
```
lsblk
```
As mentioned security is pretty important. Before we continue we will write lots of random data to the disk so that it is completely wiped clean. This may take a while, so grab a brew!
```
dd if=/dev/urandom of=/dev/sda bs=1M
```
Once that is completed we need to create the partition table. We are only going to create two partitons, one for the boot partition and one for the rest of the disk. Later on we will use LVM ([Logical Volume Manager](https://en.wikipedia.org/wiki/Logical_Volume_Manager_%28Linux%29){:target="_blank" rel="noopener noreferrer"}) to break the disk down further
```
fdisk /dev/sda
```
You will now be in the fdisk utility, and you will see the prompt has changed
```
Command (? for help):
p (should be empty)
n
p
1
[return]
+512M
t
L
83
n
p
2
[return]
[return]
t
2
8e
p
w
```
Make sure the device now shows the two partitions, sda1 and sda2
```
lsblk
```
Format /boot partition
```
mkfs.ext3 /dev/sda1
```
Before we set up LVM on the second partition we need to encrypt it. We will be using LUKS (Linux Unified Key Setup)

First make sure the module is loaded
```
modprobe dm-crypt
```
The encryption setup is fairly standed. We are using "aes-xts-plain64" cipher for LUKS. We'll include the `-y` option to verify the passphrase (by asking twice) and we set the key size to 512 bits (this argument must be a multiple of 8)
```
cryptsetup -c aes-xts-plain64 -y -s 512 luksFormat /dev/sda2
YES
(enter passphrase twice)
```
Now we open the encrypted partition under `/dev/mapper/lvm`. Then add it as a physical volume group on the whole partition
```
cryptsetup luksOpen /dev/sda2 lvm
(enter passphrase)
pvcreate /dev/mapper/lvm
pvs
vgcreate vg_arch /dev/mapper/lvm
vgs
```
For my logical volumes I have sized the partitions based on various best practice rules that I have picked up over the years. This is by no meeans a strict rule, but it is advised to split up `/boot`, `/var`, and `/home`. The `/boot` directory has been placed on a seperate partition due to the encryption we're going to use. We will also create some [swap space](http://linuxjournal.com/article/10678){:target="_blank" rel="noopener noreferrer"}. In this exampe I am using a 64GB disk
- /boot = 512MB
- /var = 10GB
- swap = 8GB
- / = 20GB
- /home = 25.49GB

Create the logical volumes, create filesystems on each volume and ensure the swap space is active
```
lvcreate -L 20GB -n lv_root vg_arch
lvcreate -L 10GB -n lv_var vg_arch
lvcreate -L 8GB -n lv_swap vg_arch
lcreate -l +100%FREE -n lv_home vg_arch
lvs
mkfs.ext4 /dev/mapper/vg_arch-lv_root
mkfs.ext4 /dev/mapper/vg_arch-lv_var
mkfs.ext4 /dev/mapper/vg_arch-lv_home
mkswap /dev/mapper/vg_arch-lv_swap
swapon /dev/mapper/vg_arch-lv_swap
```
We are going to mount the root filesystem under `/mnt` then create a few directories for the other volumes
```
mount /dev/mapper/vg_arch-lv_root /mnt
mkdir /mnt/{boot,var,home}
mount /dev/sda1 /mnt/boot
mount /dev/mapper/vg_arch-lv_var /mnt/var
mount /dev/mapper/vg_arch-lv_home /mnt/home
df -ah
```
Before we install Arch we need to configure the mirrorlist. As I am currently in the UK I will generate a relevant mirrorlist. This is where an internet connection comes in useful. We will use the Arch [mirrorlist generator](https://www.archlinux.org/mirrorlist/){:target="_blank" rel="noopener noreferrer"} and `wget` to pull it onto our system. By default the lines are all commented out so we'll use `sed` to uncomment the correct lines. Then we will switch the current mirrorlist with our new one, it is good practice to always make backup copies of configuration files before replacing or modifying them
```
wget -O mirrorlist "https://www.archlinux.org/mirrorlist/?country=GB&protocol=http&protocol=https&ip_version=4&use_mirror_status=on"
cat mirrorlist
sed -i 's/^#S/S/g' mirrorlist
mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
mv mirrorlist /etc/pacman.d/mirrorlist
```
Now we can install the base Arch packages
```
pacstrap /mnt base
```
Generate fstab, setting the `-U` option to use UUIDs. The `-p` excludes pseudofs mounts
```
genfstab -p -U /mnt > /mnt/etc/fstab
cat /mnt/etc/fstab
```
To configure the rest of the system we're going to use [chroot](https://en.wikipedia.org/wiki/Chroot){:target="_blank" rel="noopener noreferrer"} to change the root directory. This makes it easier to configure
```
arch-chroot /mnt
```
You will notice that the prompt has now changed
```
[root@archiso /]#
```
Set a symbolic link to the timezone file for your city, in this case London
```
ln -s /usr/share/zoneinfo/Europe/London /etc/localtime
```
Set up the locale settings, in this case we are using en_GB
```
vi /etc/locale.gen
```
Remove the # at the start of the "en_GB.UTF-8 UTF-8" line
Save and quit
```
locale-gen
echo LANG=en_GB.UTF-8 > /etc/locale.conf
export LANG=en_GB.UTF-8
```
We need to set the keymap so that it sets on boot
```
echo 'KEYMAP="gb"' > /etc/vconsole.conf
```
Pick a hostname that is relevant, or clever, or funny
```
echo gibson > /etc/hostname
vi /etc/hosts
```
Navigate to line beginning with "127.0.0.1" and append your hostname to the end
Save and quit
Ensure dhcpcd is enabled on boot
```
systemctl enable dhcpcd
```
Next we need to install and configure the [GRUB](https://en.wikipedia.org/wiki/GNU_GRUB){:target="_blank" rel="noopener noreferrer"} bootloader
```
pacman -S grub
y
grub-install --target=i386-pc /dev/sda
vi /etc/default/grub
```
Navigate to line GRUB_CMDLINE_LINUX=""
Change to the following (replacing `vg_arch` with your volume group name)
```
GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda2:vg_arch"
```
Save and quit
```
grub-mkconfig -o /boot/grub/grub.cfg
vi /etc/mkinitcpio.conf
```
Navigate to the following line
```
HOOKS="base udev autodetect modconf block filesystems keyboard fsck"
```
Add in the hooks "encrypt" and "lvm2" after "block"
```
HOOKS="base udev autodetect modconf block encrypt lvm2 filesystems keyboard fsck"
```
Now we generate the initial ramdisk
```
mkinitcpio -p linux
```
We must set the root password - make sure it's secure!
```
passwd
(no visual output - don't worry!)
vi /etc/sudoers
```
Navigate to the following line and remove the # at the start
```
# %wheel ALL=(ALL) PASSWD: ALL
```
Save and quit - you may need to force write with `:wq!` (in vi) as the file is read-only.

Finally we need to add a standard user. This command will create a "users" group for the user and also add them into the "wheel" group so that they can run the `sudo` command. Don't forget to set a strong password!
```
useradd -m -g users -G wheel -s /bin/bash pyratebeard
passwd pyratebeard
exit
```
That's it, all done. We have exited back to the ISO and you should see the prompt change. All that is left is to unmount the system and reboot
```
umount /mnt/{boot,var,home}
umount /mnt
reboot
```
Remove the ISO media and the system should boot off your new Arch Linux install.

Enter the encryption passphrase we set when prompted.

Login as your user account.

Congratulations, you've successfully installed Arch Linux with an encrypted root filesystem.

I will be doing another log soon which details how I set up my system, including the Window Manager, various applications, and the all important dotfiles!

