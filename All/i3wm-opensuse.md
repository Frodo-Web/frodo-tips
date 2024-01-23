# Using i3wm on OpenSUSE TumbleWeed
## Installation
1. Install minimal version of openSUSE, without xorg and any additional packages
2. Install packages:
```bash
zypper install xorg-x11-server xinit i3wm kitty firefox
```
3. Add command to .xinitrc
```bash
echo 'exec i3' >> ~/.xinitrc
```
4. Run i3wm
```bash
startx
```
## Shortcuts
````
General

    mod + Enter - запустить терминал
    mod + d - запустить dmenu для запуска любой программы
    mod + Shift + q - закрыть активное окно программы
    mod + 1, mod + 2, mod + …n - переключение рабочих столов от 1 до 9
    mod + Shift + 1, mod + Shift + … n - перемещение активного окна на другой рабочий стол
    mod + Shift + … ←, →, ↑, ↓ - изменение положения окон в рамках рабочего стола
    mod + r - ресайз активного окна
    mod + Shift + c - перечитать конфиг
    mod + Shift + r - перезапуск i3
    mod + Shift + e - выход из i3 с подтверждением
    mod + Shift + space - перевод окна в плавающий режим

Windows

 $mod+w --> tabbed layout
 $mod+e --> vertical and horizontal layout (switches to and between them)
 $mod+s --> stacked layout
 $mod+f --> fullscreen
You can toggle floating mode for a window by pressing $mod+Shift+Space. 

Movement (or ARROWS)

$mod+j  left
$mod+k  down
$mod+l  up
$mod+;  right

Splitting
To split a window vertically, press $mod+v before you create the new window. To split it horizontally, press $mod+h.
````
## Config 
Add background images to displays:
````
exec --no-startup-id nitrogen --set-auto /path/pic1.jpg --head=0 && nitrogen --set-auto /path/pic2.png --head=1
````
## Layouts
Create a file /etc/X11/xorg.conf.d/00-keyboard.conf with content like this, restart xorg:
````
Section "InputClass"
        Identifier "US RU Layout"
        MatchIsKeyboard "on"
        Option "XkbLayout" "us,ru"
        Option "XkbOptions" "grp:alt_shift_toggle,grp_led:caps"
EndSection
````

## Install snap
````
sudo zypper addrepo --refresh https://download.opensuse.org/repositories/system:/snappy/openSUSE_Tumbleweed snappy
sudo zypper --gpg-auto-import-keys refresh
sudo zypper dup --from snappy
sudo zypper install snapd
sudo systemctl enable --now snapd
sudo systemctl enable --now snapd.apparmor
````
### Install apps with snap
````
sudo snap install telegram-desktop
````
