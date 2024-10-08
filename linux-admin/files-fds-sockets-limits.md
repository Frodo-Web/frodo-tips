# Working with files, file descriptors, sockets, processes, limits in Linux
Find logstash worker thread and strace it, find out that elastic responds slowly
````
ps aux | grep jvm
ps -T -p 342690 -o pid,spid,stat,lwp,comm | less
strace -T -tt -o /tmp/logstash-worker.txt -p 343188
````
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
..
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 63047
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 65536
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 63047
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

Number of open files for a user
lsof -u grafana | wc -l

Find ulimit for a process
cat /proc/PID/limits
cat /proc/24193/limits | grep -i 'Open files'
..
Limit                     Soft Limit           Hard Limit           Units     
Max cpu time              unlimited            unlimited            seconds   
Max file size             unlimited            unlimited            bytes     
Max data size             unlimited            unlimited            bytes     
Max stack size            8388608              unlimited            bytes     
Max core file size        0                    unlimited            bytes     
Max resident set          unlimited            unlimited            bytes     
Max processes             4096                 4096                 processes 
Max open files            65535                65535                files     
Max locked memory         unlimited            unlimited            bytes     
Max address space         unlimited            unlimited            bytes     
Max file locks            unlimited            unlimited            locks     
Max pending signals       63047                63047                signals   
Max msgqueue size         819200               819200               bytes     
Max nice priority         0                    0                    
Max realtime priority     0                    0                    
Max realtime timeout      unlimited            unlimited            us

Number of open files for a process
ls -l /proc/24193/fd/ | wc -l
lsof -p 24193 | wc -l

Number of open files by process name
lsof -u grafana | awk '{print $1}' | sort | uniq -c | sort -r | head
lsof -u elasticsearch | awk '{print $1}' | sort | uniq -c | sort -r | head
..
     30 controlle
  22524 java
      1 COMMAND

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

Elastic:
  21698 REG
    767 IPv4
     18 a_inode
      4 FIFO
      4 DIR
      3 CHR
      2 unix
      1 TYPE
````
Networks
````
The net.core.somaxconn parameter controls the maximum number of connections that can be queued for acceptance by a service (e.g., web server).
Check the current backlog size:

sysctl net.core.somaxconn
..
net.core.somaxconn = 4096

Maximum Syn Backlog (for incomplete connection requests):
This value is important because if the SYN backlog (incomplete connections waiting for acknowledgment) is full, new connection attempts will be dropped.

sysctl net.ipv4.tcp_max_syn_backlog
..
net.ipv4.tcp_max_syn_backlog = 512

TCP Retransmission Timeouts: Controls how long to wait for a TCP packet before considering it lost.

sysctl net.ipv4.tcp_retries2
..
net.ipv4.tcp_retries2 = 15

TCP SYN Retries: Determines how many times TCP retransmits a SYN packet (used during connection establishment).

sysctl net.ipv4.tcp_syn_retries
..
net.ipv4.tcp_syn_retries = 6

A high volume of short-lived connections can result in a large number of sockets in the TIME_WAIT state. If the system runs out of resources to track these, new connections may be delayed or droppe
Check the maximum number of time-wait connections:

sysctl net.ipv4.tcp_max_tw_buckets
..
net.ipv4.tcp_max_tw_buckets = 65536

Reduce the TIME_WAIT duration (optional, but be cautious):

sysctl net.ipv4.tcp_fin_timeout
..
net.ipv4.tcp_fin_timeout = 60

Параметр tcp_tw_reuse полезно включить в борьбе за ресурсы, занимаемые TIME_WAIT. TCP-соединение идентифицируется по набору параметров IP1_Port1_IP2_Port2. Когда сокет переходит в состояние TIME_WAIT, при отключенном tcp_tw_reuse установка нового исходящего соединения будет происходить с выбором нового локального IP1_Port1. Старые значения могут быть использованы только тогда, когда TCP-соединение окажется в состоянии CLOSED. Если ваш сервер создает множество исходящих соединений, установите tcp_tw_reuse = 1 и ваша система сможет использовать порты TIME_WAIT в случае исчерпания свободных.

sysctl net.ipv4.tcp_tw_reuse
..
net.ipv4.tcp_tw_reuse = 2

You can use the ss or netstat command to show the state of each connection (e.g., ESTABLISHED, SYN_SENT, CLOSE_WAIT).

ss -s
..
Total: 922
TCP:   776 (estab 772, closed 0, orphaned 0, timewait 0)

Transport Total     IP        IPv6
RAW	  0         0         0        
UDP	  1         1         0        
TCP	  776       776       0        
INET	  777       777       0        
FRAG	  0         0         0

ifconfig -a
ip -s link
..
txqueuelen 1000
RX errors 0  dropped 2591

The send and receive buffer sizes can be important for network performance. To check and adjust these:

View the default socket buffer sizes:
sysctl net.core.rmem_default
sysctl net.core.wmem_default

View the maximum buffer sizes:
sysctl net.core.rmem_max
sysctl net.core.wmem_max

ss -ltn
..
State              Recv-Q             Send-Q                          Local Address:Port                            Peer Address:Port             Process             
LISTEN             0                  4096                                  0.0.0.0:9300                                 0.0.0.0:*                                    
LISTEN             0                  128                                   0.0.0.0:22                                   0.0.0.0:*                                    
LISTEN             0                  4096                                  0.0.0.0:10050                                0.0.0.0:*                                    
LISTEN             0                  4096                                  0.0.0.0:9200                                 0.0.0.0:*

Если смотреть исходники ss, то как я понял, при нормальной работе (получение данных от ядра через netlink-сокет, а не через /proc/net/tcp), в столбцы Recv-Q / Send-Q выводятся значения idiag_rqueue и idiag_wqueue структуры ядра inet_diag_msg. Они формируются в файле /usr/src/linux/net/ipv4/tcp_diag.c:

        if (sk->sk_state == TCP_LISTEN) {
                r->idiag_rqueue = sk->sk_ack_backlog;
                r->idiag_wqueue = sk->sk_max_ack_backlog;
        } else {
                r->idiag_rqueue = max_t(int, tp->rcv_nxt - tp->copied_seq, 0);
                r->idiag_wqueue = tp->write_seq - tp->snd_una;
        }

То есть для слушающего сокета это sk_ack_backlog и sk_max_ack_backlog. Насколько я понимаю, первое это количество tcp-соединений, которые приняты ядром, но ещё не accept()-нуты приложеним, а второе — это максимальное количество таких соединений, по достижению которого ядро не будет ставить их в очередь, а будет просто сбрасывать.

Recv-Q
Established: The count of bytes not copied by the user program connected to this socket.
Listening: Since Kernel 2.6.18 this column contains the current listen backlog.

Send-Q
Established: The count of bytes not acknowledged by the remote host.
Listening: Since Kernel 2.6.18 this column contains the maximum size of the listen backlog.

Recv-Q:
High Recv-Q means the data is put on TCP/IP receive buffer, but the application does not call recv() to copy it from TCP/IP buffer to the application buffer. Customer can check the application listening the port, and see if it is working as expected. For example, if you saw Recv-Q in the following connection:
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4    3223      0  11.10.32.24.8002       11.10.32.12.64672      ESTABLISHED
Customer should check the application listening the port 8002.

Send-Q:
High Send-Q means the data is put on TCP/IP send buffer, but it is not sent or it is sent but not ACKed. So, high value in Send-Q can be related to server network congest, server performance issue or data packet flow control, and so on. 
Please note: The send and receive queue sizes are shown in bytes.



Per-socket buffer usage: To see the current buffer usage per socket (for example, TCP sockets), use ss

ss -tnmi
..
State                Recv-Q                Send-Q                               Local Address:Port                                  Peer Address:Port                 Process                                                                                                                                                               
ESTAB                0                     0                                      10.20.164.5:9300                                  10.20.164.31:47320                
	 skmem:(r0,rb4210224,t0,tb87040,f4096,w0,o0,bl0,d74640) cubic wscale:7,7 rto:211 rtt:10.736/16.635 ato:40 mss:1448 pmtu:1500 rcvmss:1448 advmss:1448 cwnd:9 ssthresh:6 bytes_sent:3621930137 bytes_retrans:78614 bytes_acked:3621851523 bytes_received:37163642708 segs_out:11259958 segs_in:32225296 data_segs_out:5600793 data_segs_in:28333652 send 9710879bps lastsnd:128 lastrcv:216 lastack:127 pacing_rate 11652776bps delivery_rate 221350312bps delivered:5600746 app_limited busy:47608662ms retrans:0/134 dsack_dups:88 rcv_rtt:50.04 rcv_space:528520 rcv_ssthresh:2105112 minrtt:0.119 rcv_ooopack:983 snd_wnd:182272

You can also use netstat to view errors and dropped packets that might indicate congestion or hardware issues:
netstat -s
..
Ip:
    Forwarding: 2
    1179983517 total packets received
    0 forwarded
    0 incoming packets discarded
    1179983517 incoming packets delivered
    856472124 requests sent out
    40 dropped because of missing route
Icmp:
    36428 ICMP messages received
    0 input ICMP message failed
    ICMP input histogram:
        timeout in transit: 26
        echo requests: 36402
    36403 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 1
        echo replies: 36402
IcmpMsg:
        InType8: 36402
        InType11: 26
        OutType0: 36402
        OutType3: 1
Tcp:
    8949 active connection openings
    46095 passive connection openings
    2 failed connection attempts
    1185 connection resets received
    772 connections established
    1179876081 segments received
    2494392476 segments sent out
    1648169 segments retransmitted
    0 bad segments received
    1541 resets sent
Udp:
    66614 packets received
    1 packets to unknown port received
    0 packet receive errors
    67858 packets sent
    0 receive buffer errors
    0 send buffer errors
UdpLite:
TcpExt:
    873 TCP sockets finished time wait in fast timer
    5 packets rejected in established connections because of timestamp
    19367653 delayed acks sent
    23981 delayed acks further delayed because of locked socket
    Quick ack mode was activated 2973684 times
    540726324 packet headers predicted
    45954953 acknowledgments not containing data payload received
    343062605 predicted acknowledgments
    TCPSackRecovery: 20350
    Detected reordering 634 times using SACK
    Detected reordering 24 times using time stamp
    29 congestion windows fully recovered without slow start
    24 congestion windows partially recovered using Hoe heuristic
    TCPDSACKUndo: 7742
    275 congestion windows recovered without slow start after partial ack
    TCPLostRetransmit: 5153
    TCPSackFailures: 6
    402588 fast retransmits
    6069 retransmits in slow start
    TCPTimeouts: 523
    TCPLossProbes: 1327396
    TCPLossProbeRecovery: 3041
    TCPSackRecoveryFail: 237
    TCPBacklogCoalesce: 36883647
    TCPDSACKOldSent: 2974938
    TCPDSACKOfoSent: 213
    TCPDSACKRecv: 1233687
    TCPDSACKOfoRecv: 26
    141 connections reset due to unexpected data
    1187 connections reset due to early user close
    1 connections aborted due to timeout
    TCPDSACKIgnoredOld: 554
    TCPDSACKIgnoredNoUndo: 1046417
    TCPSpuriousRTOs: 4
    TCPSackShifted: 30667
    TCPSackMerged: 52322
    TCPSackShiftFallback: 148192
    TCPRcvCoalesce: 333630563
    TCPOFOQueue: 849579
    TCPOFOMerge: 187
    TCPSpuriousRtxHostQueues: 2652
    TCPAutoCorking: 12803975
    TCPSynRetrans: 18
    TCPOrigDataSent: 2086784034
    TCPHystartTrainDetect: 572
    TCPHystartTrainCwnd: 32458
    TCPHystartDelayDetect: 2
    TCPHystartDelayCwnd: 136
    TCPACKSkippedSeq: 3
    TCPKeepAlive: 605084
    TCPDelivered: 2088026278
    TCPAckCompressed: 651082
    TcpTimeoutRehash: 1514
IpExt:
    InMcastPkts: 4394
    InOctets: 5676934618356
    OutOctets: 2715149355562
    InMcastOctets: 158184
    InNoECTPkts: 4438887591
MPTcpExt:



To ensure that long-standing idle connections do not consume resources unnecessarily, you can adjust TCP keepalive settings:
Check the keepalive time (idle time before a probe is sent):

sysctl net.ipv4.tcp_keepalive_time
..
net.ipv4.tcp_keepalive_time = 7200

sudo sysctl -w net.ipv4.tcp_keepalive_time=600
sudo sysctl -w net.ipv4.tcp_keepalive_intvl=60
sudo sysctl -w net.ipv4.tcp_keepalive_probes=5
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
Custom ps output example
````
ps faxo stat,euid,ruid,tty,tpgid,sess,pgrp,ppid,pid,pcpu,comm
..
STAT  EUID  RUID TT       TPGID  SESS  PGRP  PPID   PID %CPU COMMAND
S      998   998 ?           -1  1305  1305     1  1306  0.0 chronyd
Ss       0     0 ?           -1  1308  1308     1  1308  0.0 crond
S        0     0 ?           -1  1308  1308  1308 28176  0.0  \_ crond
Zs       0     0 ?           -1 28182 28182 28176 28182  0.0      \_ start_backup.sh <defunct>
Sl     995   995 ?           -1 28251 28251     1 28252  0.1 python3
S      995   995 ?           -1 28251 28251 28252 28468  5.9  \_ python3
R      995   995 ?           -1 28251 28251 28252 28469 11.7  \_ python3
S      995   995 ?           -1 28251 28251 28252 28470  5.7  \_ python3
S      995   995 ?           -1 28251 28251 28252 28471  5.6  \_ python3
Sl     995   995 ?           -1 28325 28325     1 28326  0.1 python3
S      995   995 ?           -1 28325 28325 28326 29129  0.0  \_ python3
R      995   995 ?           -1 28325 28325 28326 29132  9.6  \_ python3
S      995   995 ?           -1 28325 28325 28326 29135  0.0  \_ python3
S      995   995 ?           -1 28325 28325 28326 29137  4.7  \_ python3
S      995   995 ?           -1 28325 28325 28326 29140  9.7  \_ python3
S      995   995 ?           -1 28325 28325 28326 29148  4.8  \_ python3

ps faxo stat,lstart,sess,pgrp,ppid,pid,pcpu,comm | grep python3
..
Sl   Sat Jan  6 00:00:01 2024 28251 28251     1 28252  0.1 python3
S    Sat Jan  6 00:00:02 2024 28251 28251 28252 28468  4.9  \_ python3
R    Sat Jan  6 00:00:02 2024 28251 28251 28252 28469 13.4  \_ python3
S    Sat Jan  6 00:00:02 2024 28251 28251 28252 28470  4.7  \_ python3
S    Sat Jan  6 00:00:02 2024 28251 28251 28252 28471  4.6  \_ python3
S    Sat Jan  6 00:00:02 2024 28251 28251 28252 28472  5.2  \_ python3

ps faxo stat,lstart,sess,pgrp,ppid,pid,pcpu,comm | grep start_backup
..
Zs   Sat Jan  6 00:00:00 2024 28182 28182 28176 28182  0.0      \_ start_backup.sh <defunct>

ps faxo stat,lstart,sess,pgrp,ppid,pid,pcpu,comm | grep '28176'
..
S    Sat Jan  6 00:00:00 2024  1308  1308  1308 28176  0.0  \_ crond
Zs   Sat Jan  6 00:00:00 2024 28182 28182 28176 28182  0.0      \_ start_backup.sh <defunct>
````
## setsid
setsid command is used to run a program in a new session. The command will call the fork(2) if already a process group leader. Else, it will execute a program in the current process.
A session is a collection of processes that share a common controlling terminal. By running a program in a new session, setsid detaches it from the controlling terminal of the current session, making it immune to hangups, signals, or other events that would otherwise be sent to the terminal.

In Unix-like systems, every process is associated with a process group, which is a collection of processes that share the same process ID. A process group leader is a process that is responsible for controlling the other processes in the group. If the calling process is not a process group leader, setsid creates a new session and sets the calling process as the session leader.
The setsid command can be used to start a new process in the background, without any association with the current terminal. For example, the command setsid my_program & will start my_program in a new session and detach it from the current terminal, allowing it to continue running even after the terminal session has ended.

In addition to detaching a process from a controlling terminal, setsid can also be used to create a daemon process. A daemon process is a background process that runs continuously, without any interactive input or output. By using setsid to create a new session for the daemon process, it can be run independently of any user sessions or terminal sessions, ensuring that it continues to run even if the user logs out.
````
# setsid program

Run a program in a new session discarding the resulting output and error:
# setsid program > /dev/null 2>&1

Run a program creating a new process:
#setsid --fork program

Return the exit code of a program as the exit code of setsid when the program exits:
# setsid --wait program

Run a program in a new session setting the current terminal as the controlling terminal:
# setsid --ctty program
````
## Jobs
Must have: <br>
https://biriukov.dev/docs/fd-pipe-session-terminal/3-process-groups-jobs-and-sessions/  <br>
https://www.win.tue.nl/~aeb/linux/lk/lk-10.html <br>

Run background jobs without using disown
````
nohup /usr/bin/curl http://someaddress.com &
..
[1] 873

Then exit of this terminal, a task proceed to be continued

You can even create pid file for it
nohup -p pid_file /usr/bin/curl http://someaddress.com &
````

### CROND problem
https://stackoverflow.com/questions/1506902/why-do-processes-spawned-by-cron-end-up-defunct?rq=4
````
STAT  EUID  RUID TT       TPGID  SESS  PGRP  PPID   PID %CPU COMMAND
Ss       0     0 ?           -1  3197  3197     1  3197  0.0 cron
S        0     0 ?           -1  3197  3197  3197 18825  0.0  \_ cron
Zs    1000  1000 ?           -1 18832 18832 18825 18832  0.0      \_ sh <defunct>
S     1000  1000 ?           -1 18832 18832     1 18836  0.0 sleep
````
Notice that the sh and the sleep are in the same SESS.

````
#!/bin/bash
setsid sleep 27
````
Notice you don't need &, setsid puts it in the background.

But that doesn't my case. I have different sess and pgrp

This is the solution:
````
to my opinion it's caused by process CROND (spawned by crond for every task) waiting for input on stdin which is piped to the stdout/stderr of the command in the crontab. This is done because cron is able to send resulting output via mail to the user.

So CROND is waiting for EOF till the user command and all it's spawned child processes have closed the pipe. If this is done CROND continues with the wait-statement and then the defunct user command disappears.

So I think you have to explicitly disconnect every spawned subprocess in your script form the pipe (e.g. by redirecting it to a file or /dev/null.

so the following line should work in crontab :

* * * * * ( /tmp/launcher.sh /tmp/tester.sh &>/dev/null & ) 
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
