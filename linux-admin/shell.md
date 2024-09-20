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
Using sar to see i/o status during time
````
sar -d -p -f /var/log/sa/sa20 | grep -e 'sdb' -e 'vg-data' -e 'DEV'
..
11:50:25 AM       sdb    589.43   6610.13  14663.95     36.09      3.89      6.59      0.95     56.25
11:50:25 AM   vg-data    581.80   6610.13  14663.95     36.57      3.37      5.79      0.96     55.72
12:00:12 PM       sdb    845.24   2283.33  16891.09     22.69      4.75      5.62      0.84     71.41
12:00:12 PM   vg-data    844.73   2283.33  16891.09     22.70      4.29      5.08      0.84     71.15
12:10:25 PM       sdb    729.02   1716.42  15047.21     22.99      4.06      5.57      0.83     60.84
12:10:25 PM   vg-data    727.35   1716.42  15047.28     23.05      3.53      4.85      0.83     60.58
12:20:25 PM       sdb    859.89   3075.32  18342.59     24.91      3.80      4.42      0.72     61.55
12:20:25 PM   vg-data    856.08   3075.32  18342.54     25.02      3.39      3.96      0.72     61.25
Average:          sdb   1140.48  16522.42  32113.95     42.65      5.15      4.51      0.57     65.52
Average:      vg-data   1109.34  16522.42  32113.95     43.84      4.06      3.66      0.58     64.88
````
Archive all files with maximum compression into zip
````
zip -9 -r /tmp/example.zip ./*
````
Compress logs into the same file with extension .gz
````
gzip -9 messages-20230917
..
Creates messages-20230917.gz
````
Generate random SECRET KEY of specific length (BASE64 or HEX string)
````
openssl rand -base64 40
openssl rand -hex 40
````
Create sha512 hash of a password:
`````
mkpasswd -m sha-512 --stdin
mkpasswd -m sha-512 'password'
`````
Debug and find .yaml syntax mistakes
````
yamllint /tmp/public-ngix.yaml
.. 
/tmp/public-ngix.yaml
  1:1       warning  missing document start "---"  (document-start)
  5:1       error    syntax error: found character '\t' that cannot start any token (syntax)
````
# Bash script
```
set -euxo pipefail
```
-e (Exit on error):

    This option causes the script to immediately exit if any command returns a non-zero exit status. This helps prevent the script from continuing execution after an error, which could lead to unintended consequences or harder-to-debug issues.

-u (Treat unset variables as an error):

    This option treats unset variables as an error and causes the script to exit immediately. This helps catch typos and ensures that all variables are properly initialized before they are used.

-x (Print commands and their arguments as they are executed):

    This option prints each command to the terminal before it is executed, which is useful for debugging purposes. It allows you to see the exact commands and their arguments, making it easier to track down where an error might be occurring.

-o pipefail (Return the exit status of the last command in the pipe that failed):

    This option changes the behavior of pipelines so that the pipeline's return status is the value of the last (rightmost) command to exit with a non-zero status, or zero if all commands exit successfully. This is useful for catching errors in pipelines, which can sometimes be masked by the default behavior of only considering the exit status of the last command.
