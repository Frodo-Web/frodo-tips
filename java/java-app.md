# Working with Java Applications (JVM, jboss, wildfly)

#### Java Process Status (jps) Tool. Print JVM arguments, without using ps, grep
````
$ sudo /opt/jdk/last/bin/jps -v   // !!! sudo
..
4613 Jps -Dapplication.home=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.352.b08-2.el7_9.x86_64 -Xms8m
17670 Bootstrap -Dnop -Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager -Xms4G -Xmx4G -Dlog4j2.formatMsgNoLookups=true -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=4447 -Dcom.sun.management.jmxremote.rmi.port=1100 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Djava.rmi.server.hostname=xxx.xxx.xxx.xxx -Dignore.endorsed.dirs= -Dcatalina.base=/u01/tomcat/apache-tomcat -Dcatalina.home=/u01/tomcat/apache-tomcat -Djava.io.tmpdir=/u01/tomcat/apache-tomcat/temp
````
#### JVM GC Stats
Frequent GC of (O)ld space (Full GC - FGC) may cause high CPU usage
````
sudo -u wildfly /opt/jdk/java-19/bin/jstat -gccause $PID 2000 0
````
#### Class instances and usage stats
````
sudo -u wildfly /opt/jdk/java-19/bin/jmap -histo $PID | head -n 60
..
num     #instances         #bytes  class name
...
   4:      33534521      568248158  java.util.HashMap$Node
   5:       2521143      351352434  [Ljava.lang.Object;
   6:      13452534      464758655  java.lang.String
...
````
#### Java Heap Dump
Make heap dump then run <b> jhat </b> utility, which opens a web-interface on port 7000
````
su - wildfly -c "/opt/jdk/last/bin/jmap -dump:format=b,file=/mnt/wildfly/heapdump.hprof $PID"
````
It's better to use OQL (Object Query Language):
````
select s from java.lang.String s where s.count >= 50
````
You can use internal constructor referrers() to find all objects which refer to objects of specific type.
For example, this query will find all objects refer to files:
````
select referrers(f) from java.io.File f
````
#### JVM Heap capacity and utilization statistics
Find out options being used for running jvm
```
-Xms<size>        set initial Java heap size
-Xmx<size>        set maximum Java heap size
-Xss<size>        set java thread stack size
```
Find out process ID
```
jps -l
..
36288 org.elasticsearch.bootstrap.Elasticsearch
99508 org.logstash.Logstash
101688 jdk.jcmd/sun.tools.jps.Jps
```
Show heap stats
```
jstat -gc 99508
..
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT    CGC    CGCT     GCT   
195904.0 195904.0 134615.6  0.0   1567680.0 1013792.5 2236352.0  1343015.3  144364.0 122590.0 22508.0 16346.5    546    6.934   0      0.000  16      1.078    8.011
```
Here is brief explanation on those fields
```
S0C (Survivor space 0 capacity): The current capacity of the first survivor space in the young generation (in KB).
S1C (Survivor space 1 capacity): The current capacity of the second survivor space in the young generation (in KB).
S0U (Survivor space 0 utilization): The amount of memory currently used in the first survivor space (in KB).
S1U (Survivor space 1 utilization): The amount of memory currently used in the second survivor space (in KB).
EC (Eden space capacity): The current capacity of the Eden space in the young generation (in KB).
EU (Eden space utilization): The amount of memory currently used in the Eden space (in KB).
OC (Old generation capacity): The current capacity of the old generation (in KB).
OU (Old generation utilization): The amount of memory currently used in the old generation (in KB).
MC (Metaspace capacity): The current capacity of the metaspace (in KB).
MU (Metaspace utilization): The amount of memory currently used in the metaspace (in KB).
CCSC (Compressed class space capacity): The current capacity of the compressed class space (in KB).
CCSU (Compressed class space utilization): The amount of memory currently used in the compressed class space (in KB).
YGC (Number of young generation GC events): The number of garbage collection events that have occurred in the young generation.
YGCT (Young generation garbage collection time): The total time (in seconds) spent on garbage collection in the young generation.
FGC (Number of full GC events): The number of full garbage collection events that have occurred.
FGCT (Full garbage collection time): The total time (in seconds) spent on full garbage collection.
CGC (Number of concurrent GC events): The number of concurrent garbage collection events that have occurred.
CGCT (Concurrent garbage collection time): The total time (in seconds) spent on concurrent garbage collection.
GCT (Total garbage collection time): The total time (in seconds) spent on all types of garbage collection.
```
Convert it
```
S0C: 195904.0 KB -> 191.29 MB
S1C: 195904.0 KB -> 191.29 MB
S0U: 134615.6 KB -> 131.41 MB
S1U: 0.0 KB -> 0.00 MB
EC: 1567680.0 KB -> 1531.64 MB
EU: 1013792.5 KB -> 990.80 MB
OC: 2236352.0 KB -> 2183.94 MB
OU: 1343015.3 KB -> 1311.94 MB
```
Calculate it
```
Total Heap Capacity (EC + S0C + S1C + OC):
1531.64 MB (EC) + 191.29 MB (S0C) + 191.29 MB (S1C) + 2183.94 MB (OC) = 4098.16 MB

Total Heap Utilization (EU + S0U + S1U + OU):
990.80 MB (EU) + 131.41 MB (S0U) + 0.00 MB (S1U) + 1311.94 MB (OU) = 2434.15 MB
```
#### Working with threads and jstack
Find out JVM $PID and list the threads after
````
ps -ef | less
ps -T -p 6995
````
or by using top -H, you can detect threads with high cpu usage interactively
````
top -H
````
You can make stack trace:
````
/opt/jdk/java-19/bin/jstack -l $PID > /tmp/jstack.threads.$PID
````
Convert TID to HEX
````
 $ printf 0x%x 11340
 0x2c4c
````
Then find in jstack.threads.$PID that thread and its method which causes the problem

More in the article - https://habr.com/ru/companies/otus/articles/427513/
#### Watch JVM using ps
````
watch -n 0.5 'ps -u wildfly -o pid,uname,pcpu,pmem,comm --sort=-pmem,-pcpu'
watch -n 1 'ps --no-headings -p 6995 -Lo pid,tid,%cpu,%mem,stat,start_time,time,ucmd | sort -nrk 3 | head -30'
````

