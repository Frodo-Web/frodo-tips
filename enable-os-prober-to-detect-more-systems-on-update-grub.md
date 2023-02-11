# Multiboot. Enable os-prober to add more systems to the boot menu on update-grub
Here are my partitions, I have 3 systems installed:
````
sudo fdisk -l
...
/dev/nvme0n1p1  EFI System            <- EFI partition
/dev/nvme0n1p2  Linux Filesystem      <- Debian Installation
/dev/nvme0n1p3  Microsoft Reserved
/dev/nvme0n1p4  Linux filesystem      <- Ubuntu Installation
/dev/nvme0n1p5  Microsoft basic data  <- Windows 11 Installation
````
I also have all their grub configurations on EFI partition. But when I start my machine, the first installed, Debian is being loaded. <br>
So I can boot with Windows/Ubuntu only by choosing them in UEFI boot menu, they are NOT listed in grub boot menu when machine starting.
````
su
cd /media/
mkdir efi
mount /dev/nvme0n1p1 ./efi
ls ./efi/EFI
...
Boot debian Microsoft ubuntu
````
os-prober is able to detect them:
````
sudo os-prober
...
/dev/nvme0n1p1@/EFI/Microsoft/Boot/bootmgfw.efi:Windows Boot Manager:Windows:efi
/dev/nvme0n1p4:Ubuntu 22.10 (22.10):Ubuntu:linux
````
But update-grub cant detect them and don't add them to the boot menu:
````
sudo update-grub
....
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
````
So you need to uncomment this line in /etc/default/grub
````
GRUB_DISABLE_OS_PROBER=false
````
Then update-grub will detect and add them:
````
sudo update-grub
...
Found Windows Boot Manager on /dev/nvme0n1p1@/EFI/Microsoft/Boot/bootmgfw.efi
Found Ubuntu 22.10 (22.10) on /dev/nvme0n1p4
````
I got this message on boot with Ubuntu
````
bad shim signature 
you need to load the kernel first
````
I just decided to turn off the Secure Boot in UEFI settings
