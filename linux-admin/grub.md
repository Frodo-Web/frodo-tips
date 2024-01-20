# Working with GRUB

## Remove all partitions with different OSes and leave only one OS

````
From LiveCD:
1. Delete those partitions with fdisk, leaving a single OS and efi partition
2. Reinstall GRUB
sudo mkdir /mnt/root && sudo mount /dev/nvme0n1p1 /media/root
sudo mkdir /mnt/efi && sudo mount /dev/nvme0n1p2 /media/efi
sudo grub-install --target=x86_64-efi /dev/nvme0n1 --efi-directory=/mnt/efi --boot-directory=/mnt/root/boot
3. Delete old entries from EFI
3.1. List those entries
sudo efibootmgr -v
3.2. Delete them by their boot number, for example
efibootmgr -b 0002 -B
4. Reboot to main OS
Main OS:
5. Remove old data from /boot/efi
sudo rm -r /boot/efi/EFI/Microsoft
sudo rm -r /boot/efi/EFI/debian
6. Remove old kernel packages
dpkg --list | grep -i -E --color 'linux-image|linux-kernel' | grep '^ii'
sudo apt remove linux-image-5.19*
7. Update GRUB
sudo update-grub2
8. Reboot
````
