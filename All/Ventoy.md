# Ventoy
```
https://www.ventoy.net/en/doc_start.html
https://www.ventoy.net/en/download.html

https://github.com/ventoy/Ventoy/releases

Качаем утилиту куда нить в хомяк
curl -L -O https://github.com/ventoy/Ventoy/releases/download/v1.0.96/ventoy-1.0.96-linux.tar.gz
Извлекаем
tar -xzvf ventoy-*.tar.gz
Идём в диру с ней
cd ventoy-1.0.96

Вставляем и ищем флешку 32 гига которая, скорее всего она какая нить /dev/sdb
sudo ls /dev/sd*
sudo fdisk -l 
Если 32 гига там то эт она
Заходим на эту флешку
sudo fdisk /dev/sdX
Удаляем все разделы
d -> все цифры перебираем -> в конце w
Затем создадим GPT таблицу
sudo fdisk /dev/sdX
m -> убедимся что создать таблицу это пункт g - Create a new empty GPT partition table
Затем w
Убедимся что GPT таблица создана, а разделов на флешке нет
sudo fdisk -l /dev/sdX
Должны увидеть пустоту по разделам и строку Disklabel type: gpt

Запишем Ventoy на диск, находимся в директории с распакованым Ventoy
sh Ventoy2Disk.sh -i -g /dev/sdX - укажи флешку аккуратно
Смотрим че за разделы создал Ventoy
sudo fdisk -l /dev/sdX  # Здесь смотрим поле Type, оно самое правое
sudo lsblk -f /dev/sdX  # Здесь смотрим поле FSType, сразу за именем девайса
Там должен быть какой то раздел, который отформатирован в exFAT. Возможно он будет первый и жирный.
Нам нужно чтобы на нём было МИНИМУМ 20 гигов, чтобы закинуть образы.
Запомнин цифру раздела и отформатируем лучше в ntfs, например это может быть /dev/sdb1
sudo mkfs -t ntfs /dev/sdXY

Затем давай создадим диру и смонтируем его
Смотришь есть ли на корне или хомяке 20 гигов
df -h
Идём туда где есть место
Создаём диру для образов
mkdir images
Монтируем этот раздел 
sudo mount -t ntfs /dev/sdXY ./images
Заходим в него и качаем образы.
cd ./images
curl -O http://xxx.xxx.xxx.xxx:3000/win-corei.iso
curl -O http:/xxx.xxx.xxx.xxx:3000/win-dima.iso
curl -O http://xxx.xxx.xxx.xxx6:3000/win-eagle.iso
curl -O http://xxx.xxx.xxx.xxx6:3000/win-official.iso

md5sum  win-corei.iso
md5sum  win-dima.iso
md5sum  win-eagle.iso
Как только скачали, выходим с диры и размонтируем.
cd ..
sudo umount /dev/sdXY
Убедимся что дира пустая и всё успешно
ls ./images
```
