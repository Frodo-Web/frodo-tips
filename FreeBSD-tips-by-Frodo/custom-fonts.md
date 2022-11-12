# Setup custom xorg fonts in FreeBSD
## 1. Download Font Awesome 6
````
https://fontawesome.com/download
>>> fontawesome-free-6.2.0-desktop.zip
````
## 2. Rename them to whatever you want
````
Font Awesome 6 Brands-9.otf
Font Awesome 6 Free-Regular-400.otf
Font Awesome 6 Free-Solid-9.otf
````
## 3. Create new directory in /usr/local/share/fonts and move fonts to there
````
# mkdir /usr/local/share/fonts/customFonts
# mv myFonts/* /usr/local/share/fonts/customFonts
````
## 4. Create custom .conf file
````
# nvim /usr/local/etc/X11/xorg.conf.d/customFonts.conf
````
### Then add font paths
````
Section "Files"
    ModulePath    "/usr/local/lib/xorg/modules"
    FontPath    "/usr/local/share/fonts/customFonts/"
    FontPath    "/usr/local/share/fonts/otherFonts/"
EndSection
````
## 5. Run X
````
$ startx
````
