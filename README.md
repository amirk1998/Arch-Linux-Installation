# Arch Linux Installation

1) First set boot to UEFI

![arch-linux-install-live-usb.png](/home/amir/Documents/arch-linux-installation/arch-linux-install-live-usb.png)

we can verify the boot using the following command

```bash
ls /sys/firmware/efi/efivars
```

2) Set a good font for high resolution screens

```bash
setfont ter-132b
```

3) Then Set password for root user to able to connect through SSH using Termianl

```bash
passwd
```

4) Connect to the Arch Live using SSH

```bash
ssh root@IP
```

5) Ping to ensure network works fine

```bash
ping -c 3 archlinux.org
```

6) Set Time and date to ensure work correctly

```bash
timedatectl set-ntp true
```

```bash
timedatectl status
```

7) Then see what partitions do we have

```bash
lsblk
```

Also we can use this command 

```bash
fdisk -l
```

Imagine that the partition we want to create on `/dev/sda`

8) Now we start the partitioning 

```bash
cfdisk /dev/sda
```

![02-label-type.jpg](/home/amir/Documents/arch-linux-installation/02-label-type.jpg)

we use GPT for UEFI boot .

Now we want 3 partitions

- `/dev/sda1`   # choose 512Mb of space (UEFI)
- `/dev/sda2`  # choose 2GB to 4GB for Swap (Swap)
- `/dev/sda3`   # choose at least 10 GB of space (root)

![Screenshot from 2023-08-30 20-47-02.png](/home/amir/Documents/arch-linux-installation/Screenshot_from_2023-08-30_20-47-02.png)

The first partition is the UEFI partition. It needs to be formatted with a FAT file system:

```bash
mkfs.fat -F32 /dev/sda1
```

For Swap Partition we use this command to format:

```bash
mkswap /dev/sda2
```

And then use this command to mount the swap partition:

```bash
swapon /dev/sda2
```

For root partition we use this command to format :

```bash
mkfs.ext4 /dev/sda3
```

8-A)  **[OPTIONAL]** ****Configure the Mirrors****

The installer comes with Reflector, a Python script written for retrieving the latest mirror list the [Arch Linux Mirror Status](https://archlinux.org/mirrors/status/) page. To print out the latest mirror list, simply execute the following command:

```bash
reflector
```

If you have a slow internet connection, you may encounter an error message as follows:

```bash
failed to rate http(s) download (https://arch.jensgutermuth.de/community/os/x86_64/community.db): Download timed out after 5 second(s).
```

This happens when the default timeout (5 seconds) is lower than the actual time it's taking to download the information.

You can remedy to this problem by using the `--download-timeout` option:

```bash
reflector --download-timeout 60
```

Now reflector will wait for a whole minute before starting to scream. A long list of mirrors should show up on your screen:

Going through the entire list to find nearby mirrors would be a pain. That's why reflector can do that for you.

Reflector can generate a list of mirrors based on a plethora of given constraints. For example, I want a list of mirrors that were synchronized within the last 12 hours and that are located either in India or Singapore (these two are closest to my location), and sort the mirrors by download speed.

Turns out, reflector can do that:

```bash
reflector --download-timeout 60 --country Germany --age 12 --protocol https --sort rate
```

Printing out a mirror list like this is not enough. You'll have to persist the list in the `/etc/pacman.d/mirrorlist` location. Pacman, the default package manager for Arch Linux, uses this file to learn about the mirrors.

Before overwriting the default mirror list, make a copy of it:

```bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```

Now execute the reflector command with the `--save` option as follows:

```bash
reflector --download-timeout 60 --country Germany --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

Source : 

[The Arch Linux Handbook – Learn Arch Linux for Beginners](https://www.freecodecamp.org/news/how-to-install-arch-linux/)

9) **Install Arch Linux**

First, sync the Pacman repository so that you can download and install any software:

```bash
pacman -Syy
```

Now that you've created and formatted your partitions, you're ready mount them. You can use the mount command with appropriate mount points to mount any partition:
We must mount the root partition (“`/dev/sda3`“) to the “`/mnt`” directory before we can perform any installation.

```bash
# mount <device> <mount point>

mount /dev/sda3 /mnt
```

With root mounted, it’s time to install all the necessary packages. Use the pacstrap command to install Arch Linux required packages:

```bash
pacstrap -K /mnt base linux linux-firmware sudo nano vim
```

10) **Configure the Installed Arch System**

After the installation completes, generate a “*`/etc/fstab`*” file for your new Arch Linux system by issuing the following command:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Now that we have installed Arch Linux, we need to switch to the physically installed root partition using the `**arch-chroot**` command.

```bash
arch-chroot /mnt
```

****Set Time Zone****

Next, let’s configure the timezone. To find your timezone, you can list (`ls -l`) the contents of the “*`/usr/share/zoneinfo/`*” directory.

Find your preferred timezone (“*`/usr/share/zoneinfo/Zone/SubZone`*“) where “*`Zone/SubZone`*” is your selection, such as “*`America/New_York`*,” “*`Europe/Paris`*,” “*`Asia/Bangkok*,`” and so on. You got the idea.
For example we choose “ `Asia/Tehran` ”

```bash
ln -sf /usr/share/zoneinfo/Asia/Tehran /etc/localtime
```

**Set Locale**

Now we need to set up the locale. The file “*`/etc/locale.gen`*” contains locale settings and system languages and is commented on by default. We must open this file using a text editor and uncomment the line which contains the desired locale.

```bash
vim /etc/locale.gen
```

Uncomnent “*`en_US.UTF-8 UTF-8`*” and “*`en_US ISO-8859-1`*” (by removing the “*#*” sign), and any other needed locales in “*`/etc/locale.gen`*.” Then, press “*Ctrl+O*” followed by “*Enter*” to save, and finally, “*Ctrl+X*” to exit the editor.

![arch-linux-install-locale.png](/home/amir/Documents/arch-linux-installation/arch-linux-install-locale.png)

vim /etc/hosts

Now generate the locale config file using the below commands one by one:

```bash
locale-gen
```

Run the command below to synchronize the hardware clock, automatically creating a “*`/etc/adjtime`*” file containing descriptive information about the hardware mode clock setting and clock drift factor.

```bash
hwclock --systohc
```

Now we will move ahead and [set the hostname](https://linuxiac.com/change-hostname-linux/). A hostname is the computer’s name. So let’s name it, for example, “`archvbox`”. Use the following command:

```bash
echo archvbox > /etc/hostname
```

Add this name to the “*`/etc/hosts`*” file also. Edit the file with Nano editor and add the following lines (replace “`archvbox`” with the hostname you chose earlier).

```bash
vim /etc/hosts
```

```bash
127.0.0.1      localhost
::1            localhost
127.0.1.1      archvbox.localdomain   archvbox
```

Remember to set the password for the root account using the `passwd` command:

```bash
passwd
```

**Create User**

Now we create new user (Change `USER_NAME` ):

```bash
useradd -m USER_NAME
```

And then set password for this user

```bash
passwd USER_NAME
```

And then add user to the wheel group

```bash
usermod -aG wheel,audio,video,optical,storage USER_NAME
```

Finally, you'll have to enable `sudo` privilege for this new user. To do so, open the `/etc/sudoers` file using nano. Once open, locate the following line and uncomment it:

```bash
visudo
```

And then uncomment this line :

```bash
%wheel ALL=(ALL) ALL
```

11 ) **Install GRUB Bootloader on Arch Linux**

Now we install the boot loader for Arch to boot up after restart. The default boot loader for Linux distributions and Arch Linux also is represented by the GRUB package.

Install the GRUB bootloader and EFI boot manager packages:

```bash
pacman -S grub efibootmgr os-prober dosfstools mtools
```

Then create the mount point for “*`/dev/sda1`*” and mount it.

```bash
mkdir /boot/EFI
mount /dev/sda1 /boot/EFI
```

Now let’s install our boot loader.

```bash
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```

```bash
# Output
# Installing for x86_64-efi platform.
# Installation finished. No error reported.
```

Finally, generate the “*`/boot/grub/grub.cfg`*” file.

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

12) ****Configure the Network Manager****

Configuring a network manually in any Linux distribution can be tricky. That's why I advised you to install the `networkmanager` package during the system installation. If you did as I said, you're good to go. Otherwise, use `pacman` to install the package now:

```bash
pacman -S networkmanager vim git
```

```bash
systemctl enable NetworkManager
systemctl status NetworkManager
```

13) **Final Step**

First we should exit from chroot

```bash
exit
```

Then unmount `/mnt`

```bash
umount -R /mnt
```

And reboot

```bash
reboot
```

14) **[OPTIONAL] X-Org Configuration**

****Install X window system and audio****

Our Arch Linux currently contains only the essential software packages needed to manage the system from the command line, with no GUI (Graphical User Interface).

Arch Linux supports a wide range of desktop environments. I will install GNOME as a desktop environment example.

The first step is to install the X environment. Type the below command to install the Xorg as a display server.

```bash
pacman -S xorg-server xorg-apps
```

Or

```bash
pacman -S pulseaudio pulseaudio-alsa xorg xorg-xinit xorg-server
```