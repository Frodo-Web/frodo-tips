## Работа с LVM в Linux
````
Создание физического тома, группы томов, логического тома. Создание файловой системы в логическом томе.

1. Создал 3 раздела на отформатированной флешке
Device     Boot    Start      End  Sectors  Size Id Type
/dev/sda1           2048  8390655  8388608    4G 83 Linux
/dev/sda2        8390656 16779263  8388608    4G 83 Linux
/dev/sda3       16779264 30031871 13252608  6,3G 83 Linux

2. Создаём физический том
sudo pvcreate /dev/sda1
..
Physical volume "/dev/sda1" successfully created.

3. Можно посмотреть созданные физические тома
sudo pvdisplay
..
  "/dev/sda1" is a new physical volume of "4,00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sda1
  VG Name               
  PV Size               4,00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               yr8RBf-HQpc-0BRU-L0Le-irsn-98QT-9zVbUj
  
  4. Создаём группу томов example и добавляем в неё физический том /dev/sda1 
  sudo vgcreate example /dev/sda1
  ..
  Volume group "example" successfully created
  
  5. Посмотреть группы томов, можно посмотреть её общий размер, сколько доступно и сколько было уже выделено памяти под логические тома. А также права доступа, UUID и прочую информацию.
  sudo vgdisplay
  ..
  --- Volume group ---
  VG Name               example
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <4,00 GiB
  PE Size               4,00 MiB
  Total PE              1023
  Alloc PE / Size       512 / 2,00 GiB
  Free  PE / Size       511 / <2,00 GiB
  VG UUID               E6ERiU-rDTJ-S2nf-0g6d-Y1uQ-bPob-QL7Lw6
  
  
  6. Создаём логический том на 2 ГБ от объёма группы томов
  sudo lvcreate -n testvolume -L 2G example
  ..
  Logical volume "testvolume" created.
  
  7. Посмотреть созданные логические тома и информацию по ним
  sudo lvdisplay
  ..
  --- Logical volume ---
  LV Path                /dev/example/testvolume
  LV Name                testvolume
  VG Name                example
  LV UUID                11kPgW-lMDU-SeHI-id3i-6GgH-08k6-dxRLIz
  LV Write Access        read/write
  LV Creation host, time Lustre, 2023-07-10 17:25:45 +0300
  LV Status              available
  # open                 0
  LV Size                2,00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
  
  8. Можно убедиться что раздел уже доступен:
  ls -l /dev/example/testvolume 
  ..
  lrwxrwxrwx 1 root root 7 июл 10 17:25 /dev/example/testvolume -> ../dm-0
  
  9. Создаём файловую систему на этом томе
  sudo mkfs.ext4 /dev/example/testvolume
  
  10. По умолчанию ext4 резервирует часть места для системных данных. Если будут использоваться пользовательские файлы, можно отменить резервирование:
  sudo tune2fs -r 0 /dev/example/testvolume
  
  11. Примонтировать том и создать какой-нибудь файл
  sudo mount /dev/example/testvolume /mnt/lvm-testing/
  sudo touch /mnt/lvm-testing/file.txt
  df -h
  ..
  /dev/mapper/example-testvolume  2,0G  536K  1,9G   1% /mnt/lvm-testing
  
  12. Для авто-монтирования на этапе загрузки системы, можно отредактировать файл /etc/fstab, например:
  /dev/mapper/example-testvolume	/mnt/lvm-testing	ext4	noatime,nodiratime	0	2
  
  
  
  Создание дополнительного физического тома, расширение за счет него группы томов, перемещение данных с одного физического тома на другой и удаление физического тома из группы томов
  
  1. Создаём дополнительный физический том
  
  sudo pvcreate /dev/sda2
  
  2. Расширяем группу томов
  
  sudo vgextend example /dev/sda2
  
  3. Перемещаем данные с одного физического тома на другой (может быть использовано для сохранения важной инфы с умирающего диска)
  
  sudo pvmove /dev/sda1 /dev/sda2
  ..
  /dev/sda1: Moved: 0,59%
  /dev/sda1: Moved: 7,03%
  ...
  /dev/sda1: Moved: 94,53%
  /dev/sda1: Moved: 99,22%
  
  4. Удаляем физический том из группы томов
  
  vgreduce example /dev/sda1 
  
  5. Для убедительности можно удалить и сам физический том
  
  sudo pvremove /dev/sda1
  ..
  Labels on physical volume "/dev/sda1" successfully wiped.
  
  6. Можно примонтировать логический том и убедиться что файл на месте
  ls /mnt/lvm-testing/
  ...
  file.txt  lost+found
  
  7. Группа томов example теперь использует физический том /dev/sda2
  
  sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               example
  PV Size               4,00 GiB / not usable 4,00 MiB
  Allocatable           yes 
  PE Size               4,00 MiB
  Total PE              1023
  Free PE               511
  Allocated PE          512
  PV UUID               3f3J4V-Vddv-ZD5S-MVrI-IMOE-DV76-V4PZuq


Увеличение и уменьшение размера логического тома

1. Создаём физ. том и расширяем группу
sudo pvcreate /dev/sda3
sudo vgextend example /dev/sda3

2. Увеличиваем размер логического тома
sudo lvresize -L +5G /dev/example/testvolume
..
  Size of logical volume example/testvolume changed from 2,00 GiB (512 extents) to 7,00 GiB (1792 extents).
  Logical volume example/testvolume successfully resized.

3. Затем ФС логического тома нужно проверить на ошибки, а затем растянуть по всему размеру тома:
sudo e2fsck -f /dev/example/testvolume
sudo resize2fs /dev/example/testvolume

4. Монтируем и проверяем новый размер:
df -h
..
/dev/mapper/example-testvolume  6,9G  536K  6,9G   1% /mnt/lvm-testing

Уменьшить размер логического тома можно командой
sudo lvresize -L -5G /dev/example/testvolume
````
## Список команд для работы с LVM
````
dmsetup - низкоуровневая работа с драйвером device-mapper
lvm - инструмент для настройки LVM
lvmdiskscan - сканирует доступные диски, показывает их размер и принадлежность к PV LVM
Physical Volume
pvcreate - инициализация устройства как PV
pvchange - изменение атрибутов PV
pvremove - удаление неиспользуемого PV
pvmove - перемещение PV между различными устройствами
pvresize - изменение размера PV занятой VG
pvscan - сканирование дисков на предмет PV
pvs - вывод информации о PV
pvdisplay - вывод атрибутов PV
Volume Group
vgcfgbackup - резервное копирование области описания VG в файл
vgcfgrestore - восстановление области описания VG из файла
vgconvert - конвертация метаданных из LVM1 в LVM2
vgcreate - создание VG
vgremove - удаление VG
vgchange - изменение параметров VG
vgrename - переименовывание VG
vgmerge - склеивание двух VG
vgsplit - разделение одной VG на две
vgscan - сканирование дисков на наличие VG
vgdisplay - вывод атрибутов VG
vgs - вывод информации о VG
vgexport - отключение VG
vgimport - подключение VG
vgextend - добавление PV в VG
vgreduce - удаление неиспользуемых PV из VG
vgck - проверка целостности метаданных VG.
vgmknodes - пересоздание файлов для VG в каталоге /dev
Logical Volume
lvcreate - создание LV
lvremove - удаление LV
lvrename - переименовывание LV
lvchange - изменение параметров LV
lvresize - изменение размера LV
lvextend - увеличение размера LV
lvreduce - уменьшение размера LV
lvscan - поиск LV в VG
lvdisplay - вывод атрибутов LV
lvs - вывод информации о LV
````
