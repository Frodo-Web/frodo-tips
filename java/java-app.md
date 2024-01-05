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
#### Watch JVM using ps
````
watch -n 0.5 'ps -u wildfly -o pid,uname,pcpu,pmem,comm --sort=-pmem,-pcpu'
watch -n 1 'ps --no-headings -p 6995 -Lo pid,tid,%cpu,%mem,stat,start_time,time,ucmd | sort -nrk 3 | head -30'
````
