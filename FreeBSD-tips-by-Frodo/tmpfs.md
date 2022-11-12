# Use tmpfs as /tmp mount point in FreeBSD
tmpfs is included in Release since 7-Release
````
uname -a
>> FreeBSD Lustre 13.1-RELEASE FreeBSD 13.1-RELEASE releng/13.1-n250148-fc952ac2212 GENERIC amd64
````
Open /etc/fstab and add the line with tmpfs configuration, 2GB of RAM in this example:
````
# Device        Mountpoint      FStype  Options Dump    Pass#
/dev/ada0s1a    /               ufs     rw      1       1
tmpfs           /tmp            tmpfs   rw,mode=1777,size=2048M 0       0
````
Then reboot
````
sudo reboot
````
Check if tmpfs is being used
````
$ df -h

>> Filesystem      Size    Used   Avail Capacity  Mounted on
/dev/ada0s1a    289G    5.8G    260G     2%    /
devfs           1.0K    1.0K      0B   100%    /dev
tmpfs           2.0G    8.0K    2.0G     0%    /tmp
````
