# How to dual boot arch on uefi

## Step 1: Internet
Check your internet connection by running 
```
ip a
```
If you are using wifi then use the command 
```
nmtui
```
connect to your desired network after this is done you synchronise the network time protocol by running the command 
```
timedatectl set-ntp true 
```
## Step 2: Mirrors
Setting up your mirror list to gather the fastests servers to download packages
```
pacman -Syyy
```
Download reflector to sort servers
```
pacman -S recflector
```

## Step 3: Partitioning
First run the commmand `lsblk` to see a list of all the partitions and their mounted directories 
```
lsblk
```
To partition your disk use the command
```
cfdisk /dev/sda
```
Select the unallocated storage space using the arrow keys, select the option `[ New ]` then select the option `[ Write ]` confirm and then quit, this will create a new partition

## Step 4: Formatting 
Format the partition
```
mkfs.ext4 /dev/sda_partition
```
## Step 5: Mounting
Mount your newly created partition to the `/mnt`
directory 
```
mount /dev/sda_partition /mnt
```
Mount your uefi partition to the /efi directory
```
mkdir /mnt/efi
mount /dev/uefi_partition /mnt/efi
```
Mount the windows partition 
```
mkdir /mnt/windows10
mount /dev/windows_partition /mnt/windows10
```
## Step 6: Base install
```
pacstrap /mnt base linux linux-firmware intel-ucode nano
```
## Step 7: FSTAB
Generating your FSTAB 
```
genfstab -U /mnt >> /mnt/etc/fstab
```
Verify everything worked
```
cat /mnt/etc/fstab
```
The mounted partitions should be shown
## Step 8: Chroot
```
arch-chroot /mnt
```
## Step 9: Swapfile 
Create a swap file
```
fallocate -l sizeG /swapfile
```
Change the permissions of swapfile so it can be read 
```
chmod 600 /swapfile
```
Make the swap 
```
mkswap /swapfile
```
Activate the swap
```
swapon /swapfile
```
Add the swapfile to the fstab file
```
nano /etc/fstab
/swapfile none swap defaults 0 0
```
## Step 10: Locales
Set timezone, find country
```
timedatectl list-timezones
```
or 
```
timedatectl list-timezones | grep country_name
```
After finding the country run
```
ln -sf /usr/share/zoneinfo/Country/City /etc/localtime
```
Synchronise the hardware clock to the system clock
```
hwclock --systohc
```
Find prefered locale and uncomment it
```
nano /etc/locale.gen
```
Generate the locales
```
locale-gen
```
Update locale.conf with uncommencted locale
```
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```
## Step 11: Host name
Setup host name write prefered name
```
nano /etc/hostname
```
Setup hosts file
```
nano /etc/hosts

127.0.0.1   localhost
::1         localhost
127.0.1.1   hostname.localdomain    hostname
```
Give the root user a password
```
passwd
```
## Step 12: Grub
```
pacman -S grub efibootmgr os-prober ntfs-3g networkmanager network-manager-applet wireless_tools wpa_supplicant dialog mtools dosfstools base-devel linux-headers git bluez bluez-utils pulseaudio-bluetooth cups
```
Install grub 
```
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
```
Generate grub config
```
grub-mkconfig -o /boot/grub/grub.cfg
```
## Step 13: Activate Services
```
systemctl enable NetworkManager
systemctl enable bluetooth
systemctl enable cups
```
## Step 14: Creating new user 
```
useradd -mG wheel user_name
```
Give the new user a password 
```
passwd user_name
```
Give user sudo permissions
```
EDITOR=nano visudo
```
Find and uncomment `"%wheel = ALL=(ALL) ALL"`
## Step 15: Reboot
```
exit 
umount -a 
reboot
```
## Step 16: Graphics Drivers 
```
sudo pacman -S xf86-video-intel
```
## Step 17: Display Server 
```
sudo pacman -S xorg-server xorg-xinit awesome
```
## Step 18: Install yay
```
sudo git clone https://aur.archlinux.org/yay-git.git

cd /yay/
makepkg -si PKGBUILD
```
## Step 19: Timeshift
```
yay -S timeshift
```
