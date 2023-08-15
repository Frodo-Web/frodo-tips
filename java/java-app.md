# Working with Java Applications (JVM, jboss, wildfly)

#### Print JVM arguments, without using ps, grep
````
$ sudo /opt/jdk/last/bin/jps -v   // !!! sudo
..
4613 Jps -Dapplication.home=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.352.b08-2.el7_9.x86_64 -Xms8m
17670 Bootstrap -Dnop -Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager -Xms4G -Xmx4G -Dlog4j2.formatMsgNoLookups=true -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=4447 -Dcom.sun.management.jmxremote.rmi.port=1100 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Djava.rmi.server.hostname=xxx.xxx.xxx.xxx -Dignore.endorsed.dirs= -Dcatalina.base=/u01/tomcat/apache-tomcat -Dcatalina.home=/u01/tomcat/apache-tomcat -Djava.io.tmpdir=/u01/tomcat/apache-tomcat/temp
````