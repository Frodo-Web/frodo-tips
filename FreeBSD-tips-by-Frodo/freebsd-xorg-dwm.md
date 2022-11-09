# Install xorg with dwm on FreeBSD
$ - as user, # - as root
## 1. Install xorg
````
# pkg install xorg
````
## 2. Add user to video & wheel groups
Replace "user" with your username
````
# pw groupmod video -m "user"
# pw groupmod wheel -m "user"
````
## 3. Install drm-kmod
### Pay attention to the output after the installation, it will help you to choose your correct driver
````
# pkg install drm-kmod
````
## 4. Edit /etc/rc.conf
Open it:
````
# ee /etc/rc.conf
````
And add the following line, **replace the driver (i915kms)** with yours:
````
kld_list="i915kms"
````
## 5. Edit config.h and build DWM
Download and extract ports collection:
````
# portsnap fetch extract
````
Or update if it's already downloaded:
````
# portsnap update
````
Find out dwm location:
````
# whereis dwm
>>> dwm: /usr/local/bin/dwm
````
Move to that directory and extract source files:
````
# cd /usr/local/bin/dwm
# make extract
````
Switch to user and save the config in your directory for future edits:
````
$ mkdir ~/dwm
$ cp ./work/dwm-6.3/config.def.h ~/dwm/config.h
````
Customize dwm:
````
$ ee ~/dwm/config.h
````
Now specify the config path and build dwm:
````
# make DWM_CONF=/home/{Your_username}/dwm/config.h install clean.
````
## 6. Create .xinitrc
Switch back to user!
````
$ ee ~/.xinitrc
````
Add the following lines:
````
#!/bin/sh
exec dwm
````
## 7. Reboot and try it out!
Reboot:
````
# reboot
````
Log-in as user, and type this:
````
$ startx
````
xorg with dwm should run without issues


