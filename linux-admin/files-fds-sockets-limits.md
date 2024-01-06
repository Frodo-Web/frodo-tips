# Working with files, file descriptors, sockets, processes, limits in Linux
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
## Processes
Find out $PID and list the threads 
````
ps -ef | less
ps -T -p 6995

or by using top -H, you can detect threads with high cpu usage interactively
top -H
````
Find out process info using /proc (A zombie process example)
````
cat /proc/28182/status
..
Name:	start_script.sh
State:	Z (zombie)
Tgid:	28182   // thread group id
Ngid:	0
Pid:	28182
PPid:	28176
TracerPid:	0
Uid:	0	0	0	0
Gid:	0	0	0	0
FDSize:	0
Groups:	0 
Threads:	1
SigQ:	0/127948
SigPnd:	0000000000000000
ShdPnd:	0000000000000000
SigBlk:	0000000000000000
SigIgn:	0000000000000004
SigCgt:	0000000000010000
CapInh:	0000000000000000
CapPrm:	0000001fffffffff
CapEff:	0000001fffffffff
CapBnd:	0000001fffffffff
CapAmb:	0000000000000000
NoNewPrivs:	0
Seccomp:	0
Speculation_Store_Bypass:	thread vulnerable
Cpus_allowed:	ffff
Cpus_allowed_list:	0-15
Mems_allowed:	00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001
Mems_allowed_list:	0
voluntary_ctxt_switches:	52
nonvoluntary_ctxt_switches:	11

ps aux | grep '28176'  // PPID of Zombie process
root     28176  0.0  0.0 182476  2104 ?        S    00:00   0:00 /usr/sbin/CROND -n
````
Ways to find out child processes of parrent
Using /proc
````
cat /proc/<ppid>/task/<tid>/children
..
28182
````
Using utilities
````
ps --ppid 28176
..
  PID TTY          TIME CMD
28182 ?        00:00:00 start_script.sh <defunct>

pgrep -P 28176
..
28182

pstree -p 28176
..
crond(28176)───start_script.sh(28182)
````
Thread and child are different thigs!
````
ps -T -p 28176
..
  PID  SPID TTY          TIME CMD
28176 28176 ?        00:00:00 crond
````
Find the chain to the root parent
````
pstree -p -s 28182
..
systemd(1)───crond(1308)───crond(28176)───start_script.sh(28182)

// When python script calls 'withPool' inside it
pstree -p -s 28252
systemd(1)───python3(28252)─┬─python3(28468)
                            ├─python3(28469)
                            ├─python3(28470)
                            ├─python3(28471)
                            ├─python3(28472)
                            ├─python3(28474)
                            ├─python3(28475)
                            ├─python3(28477)
                            ├─python3(28479)
                            ├─python3(28480)
                            ├─python3(28481)
                            ├─python3(28483)
                            ├─python3(28484)
                            ├─python3(28485)
                            ├─python3(28486)
                            ├─python3(28489)
                            ├─{python3}(28490)
                            ├─{python3}(28498)
                            └─{python3}(28501)
````
## Explanations
#### Do killing the parent process will kill child processes?
No. If the parent is killed, children become children of the init process (that has the process id 1 and is launched as the first user process by the kernel).

The init process checks periodically for new children, and waits for them (thus freeing resources that are allocated by their return value).
#### Threads vs Processes vs Jobs
Process vs Thread - https://www.baeldung.com/linux/process-vs-thread
A process is any running program with its own address space

A jobs is a concept used by the shell - any program you interactively start that doesn't detach is a job. 

You can start a program running in the backgroud by appending an "&" like this: program &. That program would become a background job.
#### UNIX has separate concepts "process", "process group", and "session"
Each shell you get at login becomes the leader of its own new session and process group, and sets the controlling process group of the terminal to itself.

The shell creates a process group within the current session for each "job" it launches, and places each process it starts into the appropriate process group. For example, ls | head is a pipeline of two processes, which the shell considers a single job, and will belong to a single, new process group.

A process is a (collection of) thread of execution and other context, such as address space and file descriptor table. A process may start other processes; these new processes will belong to the same process group as the parent unless other action is taken. Each process may also have a "controlling terminal", which starts off the same as its parent.

The shell has the concept of "foreground" jobs and "background" jobs. Foreground jobs are process groups with control of the terminal, and background jobs are process groups without control of the terminal.

Each terminal has a foreground process group. When bringing a job to the foreground, the shell sets it as the terminal's foreground process group; when putting a job to the background, the shell sets the terminal's foreground process group to another process group or itself.

Processes may read from and write to their controlling terminal if they are in the foreground process group. Otherwise they receive SIGTTIN and SIGTTOU signals on attempts to read from and write to the terminal respectively. By default these signals suspend the process, although most shells mask SIGTTOU so that a background job can write to the terminal uninterrupted.
