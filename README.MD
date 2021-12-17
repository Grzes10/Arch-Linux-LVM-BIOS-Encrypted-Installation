## Pre install setup ##

$ timedatectl set-ntp true

$ pacman -Syyy

## Partition setup using fdisk ##

Open drive you want to install Arch on using:

$ fdisk /dev/nvme0n1

Deleting existing partitions using "d"
Create MBR partition schema using "o"

Create boot partition:

$ n -> p -> 1 -> enter -> +500M -> t -> a

Create root partition:

$ n -> p -> 2 -> enter -> enter -> t -> 2 -> 8E

Write changes to disk:

$ w

## Format boot partition ##
$ mkfs.ext2 /dev/nvme0n1p1

## Create LVM schema, encrypt the root filesystem, set the LVM to active and format it ##

$ cryptsetup luksFormat /dev/nvme0n1p2

$ cryptsetup open --type luks /dev/nvme0n1p2 lvm

$ pvcreate --dataalignment 1m /dev/mapper/lvm

$ vgcreate volgroup0 /dev/mapper/lvm

$ lvcreate -l 100%FREE volgroup0 -n root

$ modprobe dm_mod

$ vgscan

$ vgchange -ay

$ mkfs.ext4 /dev/volgroup0/root


## Mount partitions ##
$ mount /dev/volgroup0/root /mnt

$ mkdir /mnt/boot

$ mount /dev/sda1 /mnt/boot 


## Pacstrap kernel and basic files into the newly mounted filesystem ##

$ pacstrap - i /mnt base base-devel linux linux-firmware


## Creating fstab of new filesystem ##

$ genfstab -U -p /mnt >> /mnt/etc/fstab


## chroot into the new install ##

$ arch-chroot /mnt


## Setting up locale, timezone and other meta stuff ##

Edit "/etc/locale.gen" and uncomment your language

$ locale-gen
$ ln -s /usr/share/zoneinfo/Europe/Warsaw /etc/localtime

Create the locale.conf(5) file, and set the LANG variable accordingly: "/etc/locale.conf" for example: "LANG=en_US.UTF-8"

$ hwclock --systohc --utc

Create the hostname file: "/etc/hostname" put in your hostname for example "ArchPC"


## Install essential software ##

$ pacman -Syu

$ pacman -S vi vim grub-bios os-prober linux-headers mkinitcpio lvm2 networkmanager network-manager-applet nvidia nvidia-settings(if using nvidia card)


## Preparation of the LVM encryption schema ##

Edit "/etc/mkinitcpio.conf" and add "encrypt "and "lvm2" in between block and filesystems

$ mkinitcpio -p linux

## Grub ##

Edit /etc/default/grub: 
add cryptdevice=<PARTUUID>:volgroup0 to the GRUB_CMDLINE_LINUX_DEFAULT line If using standard device naming, the option will look like this: cryptdevice=/dev/nvme0n1p2:volgroup0

Also uncomment the "GRUB_ENABLE_CRYPTODISK=y" line

$ grub-install --target=i386-pc --recheck /dev/nvme0n1 <---the disk, not partition!!!!
                                                               
$ grub-mkconfig -o /boot/grub/grub.cfg

                                                               
## Set up root password ##
                                                               
$ passwd

                                                               
## Unmount filesystems and reboot ##
                                                               
$ exit
$ umount /mnt/boot
$ umount /mnt
$ reboot 
