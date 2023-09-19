# Shell
Run script as another user
````
sudo -u zabbix bash -c "export external_logging=console;/etc/zabbix/externalscripts/database_query.py database_host.ru db_name table_name"
````
CPU and I/O statistics for devices and partitions
````
sudo iostat -m -Nd -x | grep 'vgRIAK-lv_u01\|Device'
..
Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vgRIAK-lv_u01     0,00     0,00  857,97  131,67    37,36     9,33    96,61     0,16    0,16    0,29    4,36   0,58  57,78
````
Archive all files with maximum compression into zip
````
zip -9 -r /tmp/example.zip ./*
````
Generate random SECRET KEY of specific length, exclude special characters
````
tr -dc A-Za-z0-9 </dev/urandom | head -c 13 ; echo ''
````
