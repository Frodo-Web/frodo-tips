# Working with files, file descriptors, sockets, limits in Linux
Finding Grafana OnCall sockets creating problem
````
Kernel configuration
cat /proc/sys/fs/file-nr // total open files ----- open but not used files ------ max number of open files
cat /proc/sys/fs/file-max // max number of open files

Here is how to change it
sysctl -a | grep file-max
vi /etc/sysctl.conf
..
fs.file-max = 999999

/sbin/sysctl -p

But changes to this file need to re-login - sudo vi /etc/security/limits.conf  (PAM)

Find all soft and hard ulimit for user
sudo -u grafana sh -c "ulimit -Sa"
sudo -u grafana sh -c "ulimit -Ha"

Number of open files for a user
lsof -u grafana | wc -l

Find ulimit for a process
cat /proc/PID/limits
cat /proc/24193/limits | grep -i 'Open files'

Number of open files for a process
ls -l /proc/24193/fd/ | wc -l
lsof -p 24193 | wc -l

Number of open files by process name
lsof -u grafana | awk '{print $1}' | sort | uniq -c | sort -r | head

Number of open files by PID
lsof -u grafana | awk '{print $2}' | sort | less | uniq -c | sort -r -k 1,1

Process name + PID
lsof -u grafana | awk '{print $1, $2}' | sort | uniq -c  | sort -r -n -k 1,1
..
   5495 python3 23155
   5427 grafana 22911
    124 celery 16744
    124 celery 11588
    
By Type
lsof -p 23155 | awk '{print $5}' | sort | uniq -c | sort -r -n -k 1,1
..
   5476 IPv4
     89 REG
      2 unix
      2 DIR
      1 CHR
````
View opened files by pid using /proc
````
ls -l /proc/8039/fd/
````
