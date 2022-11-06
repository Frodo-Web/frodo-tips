# Solve smashed/squished screen issue after FreeBSD fresh install (on first boot)
# Some info
Here are the problems I found on net: <br>
https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=202309 <br>
https://wiki.freebsd.org/Laptops/HP_ProBook_430_G1 <br>

This also happened on my HP ProBook 430 G1. <br> In short, the boot system fails to detect/miscalculating screen resolution/frame buffer size.
# Let's solve it
### 1. After fresh install with USB stick, you need to boot from that USB stick again in LiveCD mode. Pick Multi-user mode on prompt.
### 2. Type "root" as a login. There is no password.
### 3. Mount your root partition to /tmp, because other dirs are read-only
````
cd /tmp
mkdir myDir
mount /dev/ada0s1 ./myDir
````
### 4. Open loader.conf
````
cd ./myDir
ee ./boot/loader.conf  // Make sure you are opening the file from installation, not your USB stick one!
````
### 5. Switch to text mode console. Add the following line and save it:
Since FreeBSD is using pixel mode console as default, we want to try to switch to more old, more stable and more basic one, text mode console:
````
screen.textmode="0"
````
### 6. Reboot. Now it should work
````
reboot
````
