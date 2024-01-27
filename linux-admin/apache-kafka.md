# Managing Apache Kafka

## Configuration
### zookeeper.properties (zookeeper-server-start.sh)
````
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
# 
#    http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# the directory where the snapshot is stored.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
# Disable the adminserver by default to avoid port conflicts.
# Set the port to something non-conflicting if choosing to enable this
admin.enableServer=false
# admin.serverPort=8080
````

### server.properties (kafka-server-start.sh)
````
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

#
# This configuration file is intended for use in ZK-based mode, where Apache ZooKeeper is required.
# See kafka.server.KafkaConfig for additional details and defaults
#

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. If not configured, the host name will be equal to the value of
# java.net.InetAddress.getCanonicalHostName(), with PLAINTEXT listener name, and port 9092.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
#listeners=PLAINTEXT://:9092

# Listener name, hostname and port the broker will advertise to clients.
# If not set, it uses the value for "listeners".
#advertised.listeners=PLAINTEXT://your.host.name:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/tmp/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
#log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=localhost:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0
````

## Installation
1. Download and untar steps like in official doc
````
cd /opt/
curl -L -O https://dlcdn.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz
tar -xzf kafka_2.13-3.6.1.tgz
rm kafka_2.13-3.6.1.tgz
````
2. Creating kafka user and setting permissions
````
cd /opt
adduser kafka
ln -s kafka_2.13-3.6.1/ kafka
chown -R kafka:kafka kafka
````
3. Creating unit files

3.1. kafka-zookeeper.service
````
vim /etc/systemd/system/kafka-zookeeper.service
..
[Unit]
Description=Apache Zookeeper server (Kafka)
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
User=kafka
Environment=JAVA_HOME=/usr/lib/jvm/jdk-17-oracle-x64
ExecStart=/opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties
ExecStop=/opt/kafka/bin/zookeeper-server-stop.sh

[Install]
WantedBy=multi-user.target
````
3.2. kafka.service
````
vim /etc/systemd/system/kafka.service
..
[Unit]
Description=Apache Kafka server (broker)
Documentation=http://kafka.apache.org/documentation.html
Requires=network.target remote-fs.target
After=network.target remote-fs.target kafka-zookeeper.service

[Service]
Type=simple
User=kafka
Environment=JAVA_HOME=/usr/lib/jvm/jdk-17-oracle-x64
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
````
4. Delete previous data folders, because they keep previous cluster ids, owners, etc... which will cause errors at startup our services
````
// Zookeeper
rm -rf /tmp/zookeeper/
// Kafka
rm -rf /tmp/kafka-logs/
````
5. Enable and start services
````
systemctl daemon-reload
systemctl enable kafka-zookeeper.service
systemctl enable kafka.service
systemctl start kafka-zookeeper.service
systemctl start kafka.service
systemctl status kafka-zookeeper.service
systemctl status kafka.service

journalctl -u kafka-zookeeper.service -e
journalctl -u kafka.service -e
````
## Kafka CLI tools overview 
````
kafka-server-start.sh
Use the kafka-server-start tool to start a Kafka server. You must pass the path to the properties file you want to use. If you are using ZooKeeper for metadata management, you must start ZooKeeper first. For KRaft mode, first generate a cluster ID and store it in the properties file.

USAGE: ./kafka-server-start.sh [-daemon] server.properties [--override property=value]*
Option               Description
------               -----------
--override <String>  Optional property that should override values set in
                     server.properties file
--version            Print version information and exit.

kafka-server-stop.sh
Use the kafka-server-stop tool to stop the running Kafka server. When you run this tool, you do not need to pass any arguments.

zookeeper-server-start.sh
Use the zookeeper-server-start tool to start the ZooKeeper server. ZooKeeper is the default method for metadata management for Kafka versions prior to 3.4. To run this tool, you must pass the path to the ZooKeeper properties file.

zookeeper-server-stop.sh
Use the zookeeper-server-stop tool to stop the ZooKeeper server. Running this tool does not require arguments

kafka-storage.sh
Use the kafka-storage tool to generate a Cluster UUID and format storage with the generated UUID when running Kafka in KRaft mode. You must explicitly create a cluster ID for a KRaft cluster, and format the storage specifying that ID.
For example, the following command generates a cluster ID and stores it in a variable named KAFKA_CLUSTER_ID. The next command formats storage with that ID.

KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
bin/kafka-storage.sh format -t KAFKA_CLUSTER_ID -c config/kraft/server.properties

USAGE: kafka-storage.sh [-h] {info,format,random-uuid}
The Kafka storage tool.
positional arguments:
 {info,format,random-uuid}
   info                 Get information about the Kafka log directories on this node.
   format               Format the Kafka log directories on this node.
   random-uuid          Print a random UUID.
optional arguments:
 -h, --help             show this help message and exit

kafka-cluster.sh
Use the kafka-cluster tool to get the ID of a cluster or unregister a cluster. The following example shows how to retrieve the cluster ID, which requires a bootstrap-server argument.

bin/kafka-cluster.sh cluster-id --bootstrap-server localhost:9092
..
Cluster ID: WZEKwK-b123oT3ZOSU0dgw

USAGE: kafka-cluster.sh [-h] {cluster-id,unregister}
The Kafka cluster tool.
positional arguments:
{cluster-id,unregister}
  cluster-id           Get information about the ID of a cluster.
  unregister           Unregister a broker.
optional arguments:
  -h, --help             show this help message and exit

zookeeper-shell.sh
Use the zookeeper-shell tool to connect to the interactive ZooKeeper shell.

Following is an example of how to connect to the ZooKeeper shell:
bin/zookeeper-shell.sh localhost:2181

Your results might look like the following:
Welcome to ZooKeeper!
JLine support is disabled

USAGE: ./zookeeper-shell.sh zookeeper_host:port[/path] [-zk-tls-config-file file] [args...]
Once the shell has started, you can complete the following:
ZooKeeper -server host:port [-zk-tls-config-file <file>] cmd args
  addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE
  addauth scheme auth
  close
  config [-c] [-w] [-s]
  connect host:port
  create [-s] [-e] [-c] [-t ttl] path [data] [acl]
  delete [-v version] path
  deleteall path [-b batch size]
  delquota [-n|-b] path
  get [-s] [-w] path
  getAcl [-s] path
  getAllChildrenNumber path
  getEphemerals path
  history
  listquota path
  ls [-s] [-w] [-R] path
  printwatches on|off
  quit
  reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
  redo cmdno
  removewatches path [-c|-d|-a] [-l]
  set [-s] [-v version] path data
  setAcl [-s] [-v version] [-R] path acl
  setquota -n|-b val path
  stat [-w] path
  sync path
  version

kafka-features.sh
Use the kafka-features tool to manage feature flags to disable or enable functionality at runtime in Kafka. Pass the describe argument to describe the current active feature flags, upgrade to upgrade one or more feature flags, downgrade to downgrade one or more, and disable to disable one or more feature flags, which is the same as downgrading the version to zero.

USAGE: kafka-features.sh [-h] --bootstrap-server BOOTSTRAP_SERVER [--command-config COMMAND_CONFIG]
                {describe,upgrade,downgrade,disable} ...
This tool manages feature flags in Kafka.
positional arguments:
  {describe,upgrade,downgrade,disable}
    describe             Describe one or more feature flags.
    upgrade              Upgrade one or more feature flags.
    downgrade            Upgrade one or more feature flags.
    disable              Disable one or more feature flags. This is the same as downgrading the version to zero.
optional arguments:
  -h, --help             show this help message and exit
  --bootstrap-server BOOTSTRAP_SERVER
      A comma-separated list of host:port pairs  to  use  for establishing the connection to the
                        Kafka cluster.
  --command-config COMMAND_CONFIG
      Property file containing configs to be passed to Admin Client.

kafka-broker-api-versions.sh
The kafka-broker-api-versions tool retrieves and displays broker information. For example, the following command outputs the version of Kafka that is running on the broker:
bin/kafka-broker-api-versions.sh --bootstrap-server host1:9092 --version

This tool helps to retrieve broker version information.
Option                                 Description
------                                 -----------
--bootstrap-server <String: server(s)  REQUIRED: The server to connect to.
  to use for bootstrapping>
--command-config <String: command      A property file containing configs to
  config property file>                  be passed to Admin Client.
--help                                 Print usage information.
--version                              Display Kafka version.

kafka-metadata-quorum.sh
Use the kafka-metadata-quorum tool to query the metadata quorum status. This tool is useful when you are debugging a cluster in KRaft mode. Pass the describe command to describe the current state of the metadata quorum.
The following code example displays a summary of the metadata quorum:
The output for this command might look like the following.
ClusterId:              fMCL8kv1SWm87L_Md-I2hg
LeaderId:               3002
LeaderEpoch:            2
HighWatermark:          10
MaxFollowerLag:         0
MaxFollowerLagTimeMs:   -1
CurrentVoters:          [3000,3001,3002]
CurrentObservers:       [0,1,2]

kafka-metadata-shell.sh
The kafka-metadata-shell tool enables you to interactively examine the metadata stored in a KRaft cluster.
The following example shows how to open the shell:
kafka-metadata-shell.sh --directory tmp/kraft-combined-logs/_cluster-metadata-0/
..
Loading...
 [ Kafka Metadata Shell ]
 >> ls
 brokers  configs  features  linkIds  links  shell  topicIds  topics
 >> ls /topics
 test
 >> cat /topics/test/0/data
 {
   "partitionId" : 0,
   "topicId" : "5zoAlv-xEh9xRANKXt1Lbg",
   "replicas" : [ 1 ],
   "isr" : [ 1 ],
   "removingReplicas" : null,
   "addingReplicas" : null,
   "leader" : 1,
   "leaderEpoch" : 0,
   "partitionEpoch" : 0
   }
 >> exit

kafka-configs.sh
Use the kafka-configs tool to change and describe topic, client, user, broker, or IP configuration settings. To change a property, specify the entity-type to the desired entity (topic, broker, user, etc), and use the alter option. The following example shows how you might add the delete.retention configuration property for a topic with kafka-configs.

/bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-default --alter --add-config delete.retention.ms=172800000

This tool helps to manipulate and describe entity configuration for a topic, client, user, broker or IP address.
Option                                 Description
------                                 -----------
--add-config <String>                  Key Value pairs of configs to add.
                                       Square brackets can be used to group
                                       values which contain commas: 'k1=v1,
                                       k2=[v1,v2,v2],k3=v3'. The following
                                       is a list of valid configurations:
                                       For entity-type 'topics':
                                       cleanup.policy
                                       compression.type
                                       delete.retention.ms
                                       file.delete.delay.ms
                                       flush.messages
                                       flush.ms
                                       follower.replication.throttled.
                                       replicas
                                       index.interval.bytes
                                       leader.replication.throttled.replicas
                                       local.retention.bytes
                                       local.retention.ms
                                       max.compaction.lag.ms
                                       max.message.bytes
                                       message.downconversion.enable
                                       message.format.version
                                       message.timestamp.difference.max.ms
                                       message.timestamp.type
                                       min.cleanable.dirty.ratio
                                       min.compaction.lag.ms
                                       min.insync.replicas
                                       preallocate
                                       remote.storage.enable
                                       retention.bytes
                                       retention.ms
                                       segment.bytes
                                       segment.index.bytes
                                       segment.jitter.ms
                                       segment.ms
                                       unclean.leader.election.enable
                                       For entity-type 'brokers':
                                       advertised.listeners
                                       background.threads
                                       compression.type
                                       follower.replication.throttled.rate
                                       leader.replication.throttled.rate
                                       listener.security.protocol.map
                                       listeners
                                       log.cleaner.backoff.ms
                                       log.cleaner.dedupe.buffer.size
                                       log.cleaner.delete.retention.ms
                                       log.cleaner.io.buffer.load.factor
                                       log.cleaner.io.buffer.size
                                       log.cleaner.io.max.bytes.per.second
                                       log.cleaner.max.compaction.lag.ms
                                       log.cleaner.min.cleanable.ratio
                                       log.cleaner.min.compaction.lag.ms
                                       log.cleaner.threads
                                       log.cleanup.policy
                                       log.flush.interval.messages
                                       log.flush.interval.ms
                                       log.index.interval.bytes
                                       log.index.size.max.bytes
                                       log.message.downconversion.enable
                                       log.message.timestamp.difference.max.
                                       ms
                                       log.message.timestamp.type
                                       log.preallocate
                                       log.retention.bytes
                                       log.retention.ms
                                       log.roll.jitter.ms
                                       log.roll.ms
                                       log.segment.bytes
                                       log.segment.delete.delay.ms
                                       max.connection.creation.rate
                                       max.connections
                                       max.connections.per.ip
                                       max.connections.per.ip.overrides
                                       message.max.bytes
                                       metric.reporters
                                       min.insync.replicas
                                       num.io.threads
                                       num.network.threads
                                       num.recovery.threads.per.data.dir
                                       num.replica.fetchers
                                       principal.builder.class
                                       producer.id.expiration.ms
                                       replica.alter.log.dirs.io.max.bytes.
                                       per.second
                                       sasl.enabled.mechanisms
                                       sasl.jaas.config
                                       sasl.kerberos.kinit.cmd
                                       sasl.kerberos.min.time.before.relogin
                                       sasl.kerberos.principal.to.local.rules
                                       sasl.kerberos.service.name
                                       sasl.kerberos.ticket.renew.jitter
                                       sasl.kerberos.ticket.renew.window.
                                       factor
                                       sasl.login.refresh.buffer.seconds
                                       sasl.login.refresh.min.period.seconds
                                       sasl.login.refresh.window.factor
                                       sasl.login.refresh.window.jitter
                                       sasl.mechanism.inter.broker.protocol
                                       ssl.cipher.suites
                                       ssl.client.auth
                                       ssl.enabled.protocols
                                       ssl.endpoint.identification.algorithm
                                       ssl.engine.factory.class
                                       ssl.key.password
                                       ssl.keymanager.algorithm
                                       ssl.keystore.certificate.chain
                                       ssl.keystore.key
                                       ssl.keystore.location
                                       ssl.keystore.password
                                       ssl.keystore.type
                                       ssl.protocol
                                       ssl.provider
                                       ssl.secure.random.implementation
                                       ssl.trustmanager.algorithm
                                       ssl.truststore.certificates
                                       ssl.truststore.location
                                       ssl.truststore.password
                                       ssl.truststore.type
                                       unclean.leader.election.enable
                                       For entity-type 'users':
                                       SCRAM-SHA-256
                                       SCRAM-SHA-512
                                       consumer_byte_rate
                                       controller_mutation_rate
                                       producer_byte_rate
                                       request_percentage
                                       For entity-type 'clients':
                                       consumer_byte_rate
                                       controller_mutation_rate
                                       producer_byte_rate
                                       request_percentage
                                       For entity-type 'ips':
                                       connection_creation_rate
                                       Entity types 'users' and 'clients' may
                                       be specified together to update
                                       config for clients of a specific
                                       user.
--add-config-file <String>             Path to a properties file with configs
                                       to add. See add-config for a list of
                                       valid configurations.
--all                                  List all configs for the given topic,
                                       broker, or broker-logger entity
                                       (includes static configuration when
                                       the entity type is brokers)
--alter                                Alter the configuration for the entity.
--bootstrap-server <String: server to  The Kafka server to connect to. This
connect to>                            is required for describing and
                                       altering broker configs.
--broker <String>                      The broker's ID.
--broker-defaults                      The config defaults for all brokers.
--broker-logger <String>               The broker's ID for its logger config.
--client <String>                      The client's ID.
--client-defaults                      The config defaults for all clients.
--command-config <String: command      Property file containing configs to be
config property file>                  passed to Admin Client. This is used
                                       only with --bootstrap-server option
                                       for describing and altering broker
                                       configs.
--delete-config <String>               config keys to remove 'k1,k2'
--describe                             List configs for the given entity.
--entity-default                       Default entity name for
                                       clients/users/brokers/ips (applies
                                       to corresponding entity type in
                                       command line)
--entity-name <String>                 Name of entity (topic name/client
                                       id/user principal name/broker id/ip)
--entity-type <String>                 Type of entity
                                       (topics/clients/users/brokers/broker-
                                       loggers/ips)
--force                                Suppress console prompts
--help                                 Print usage information.
--ip <String>                          The IP address.
--ip-defaults                          The config defaults for all IPs.
--topic <String>                       The topic's name.
--user <String>                        The user's principal name.
--user-defaults                        The config defaults for all users.
--version                              Display Kafka version.
--zk-tls-config-file <String:          Identifies the file where ZooKeeper
ZooKeeper TLS configuration>           client TLS connectivity properties
                                       are defined.  Any properties other
                                       than zookeeper.clientCnxnSocket,
                                       zookeeper.ssl.cipher.suites,
                                       zookeeper.ssl.client.enable,
                                       zookeeper.ssl.crl.enable, zookeeper.
                                       ssl.enabled.protocols, zookeeper.ssl.
                                       endpoint.identification.algorithm,
                                       zookeeper.ssl.keystore.location,
                                       zookeeper.ssl.keystore.password,
                                       zookeeper.ssl.keystore.type,
                                       zookeeper.ssl.ocsp.enable, zookeeper.
                                       ssl.protocol, zookeeper.ssl.
                                       truststore.location, zookeeper.ssl.
                                       truststore.password, zookeeper.ssl.
                                       truststore.type are ignored.
--zookeeper <String: urls>             DEPRECATED. The connection string for
                                       the zookeeper connection in the form
                                       host:port. Multiple URLS can be
                                       given to allow fail-over. Required
                                       when configuring SCRAM credentials
                                       for users or dynamic broker configs
                                       when the relevant broker(s) are
                                       down. Not allowed otherwise.

zookeeper-security-migration.sh
Use the zookeeper-security-migration tool to restrict or provide access to ZooKeeper metadata. The tool updates the ACLs of znodes.

kafka-topics.sh
Use the kafka-topics tool to create or delete a topic. You can also use the tool to retrieve a list of topics associated with a Kafka cluster. For more information, see Topic Operations.

bin/kafka-topics.sh --bootstrap-server host1:9092 --topic test-topic --partitions 3

This tool helps to create, delete, describe, or change a topic.
Option                                   Description
------                                   -----------
--alter                                  Alter the number of partitions and
                                          replica assignment. Update the
                                          configuration of an existing topic
                                          via --alter is no longer supported
                                          here (the kafka-configs CLI supports
                                          altering topic configs with a --
                                          bootstrap-server option).
--at-min-isr-partitions                  if set when describing topics, only
                                          show partitions whose isr count is
                                          equal to the configured minimum.
--bootstrap-server <String: server to    REQUIRED: The Kafka server to connect
  connect to>                              to.
--command-config <String: command        Property file containing configs to be
  config property file>                    passed to Admin Client. This is used
                                          only with --bootstrap-server option
                                          for describing and altering broker
                                          configs.
--config <String: name=value>            A topic configuration override for the
                                          topic being created or altered. The
                                          following is a list of valid
                                          configurations:
                                                cleanup.policy
                                                compression.type
                                                delete.retention.ms
                                                file.delete.delay.ms
                                                flush.messages
                                                flush.ms
                                                follower.replication.throttled.
                                          replicas
                                                index.interval.bytes
                                                leader.replication.throttled.replicas
                                                local.retention.bytes
                                                local.retention.ms
                                                max.compaction.lag.ms
                                                max.message.bytes
                                                message.downconversion.enable
                                                message.format.version
                                                message.timestamp.difference.max.ms
                                                message.timestamp.type
                                                min.cleanable.dirty.ratio
                                                min.compaction.lag.ms
                                                min.insync.replicas
                                                preallocate
                                                remote.storage.enable
                                                retention.bytes
                                                retention.ms
                                                segment.bytes
                                                segment.index.bytes
                                                segment.jitter.ms
                                                segment.ms
                                                unclean.leader.election.enable
                                        See the Kafka documentation for full
                                          details on the topic configs. It is
                                          supported only in combination with --
                                          create if --bootstrap-server option
                                          is used (the kafka-configs CLI
                                          supports altering topic configs with
                                          a --bootstrap-server option).
--create                                 Create a new topic.
--delete                                 Delete a topic
--delete-config <String: name>           A topic configuration override to be
                                          removed for an existing topic (see
                                          the list of configurations under the
                                          --config option). Not supported with
                                          the --bootstrap-server option.
--describe                               List details for the given topics.
--exclude-internal                       exclude internal topics when running
                                          list or describe command. The
                                          internal topics will be listed by
                                          default
--help                                   Print usage information.
--if-exists                              if set when altering or deleting or
                                          describing topics, the action will
                                          only execute if the topic exists.
--if-not-exists                          if set when creating topics, the
                                          action will only execute if the
                                          topic does not already exist.
--list                                   List all available topics.
--partitions <Integer: # of partitions>  The number of partitions for the topic
                                          being created or altered (WARNING:
                                          If partitions are increased for a
                                          topic that has a key, the partition
                                          logic or ordering of the messages
                                          will be affected). If not supplied
                                          for create, defaults to the cluster
                                          default.
--replica-assignment <String:            A list of manual partition-to-broker
  broker_id_for_part1_replica1 :           assignments for the topic being
  broker_id_for_part1_replica2 ,           created or altered.
  broker_id_for_part2_replica1 :
  broker_id_for_part2_replica2 , ...>
--replication-factor <Integer:           The replication factor for each
  replication factor>                      partition in the topic being
                                          created. If not supplied, defaults
                                          to the cluster default.
--topic <String: topic>                  The topic to create, alter, describe
                                          or delete. It also accepts a regular
                                          expression, except for --create
                                          option. Put topic name in double
                                          quotes and use the '\' prefix to
                                          escape regular expression symbols; e.
                                          g. "test\.topic".
--topic-id <String: topic-id>            The topic-id to describe.This is used
                                          only with --bootstrap-server option
                                          for describing topics.
--topics-with-overrides                  if set when describing topics, only
                                          show topics that have overridden
                                          configs
--unavailable-partitions                 if set when describing topics, only
                                          show partitions whose leader is not
                                          available
--under-min-isr-partitions               if set when describing topics, only
                                          show partitions whose isr count is
                                          less than the configured minimum.
--under-replicated-partitions            if set when describing topics, only
                                          show under replicated partitions
--version                                Display Kafka version.


kafka-get-offsets.sh
Use the kafka-get-offsets tool to retrieve topic-partition offsets.
An interactive shell for getting topic-partition offsets.

Option                                   Description
------                                   -----------
--bootstrap-server <String: HOST1:       REQUIRED. The server(s) to connect to
  PORT1,...,HOST3:PORT3>                 in the form HOST1:PORT1,HOST2:PORT2.
--broker-list <String: HOST1:PORT1,...,  DEPRECATED, use --bootstrap-server
  HOST3:PORT3>                             instead; ignored if --bootstrap-
                                          server is specified. The server(s)
                                          to connect to in the form HOST1:
                                          PORT1,HOST2:PORT2.
--command-config <String: config file>   Property file containing configs to be
                                          passed to Admin Client.
--exclude-internal-topics                By default, internal topics are
                                          included. If specified, internal
                                          topics are excluded.
--partitions <String: partition ids>     Comma separated list of partition ids
                                          to get the offsets for. If not
                                          present, all partitions of the
                                          authorized topics are queried.
                                          Cannot be used if --topic-partitions
                                          is present.
--time <String: <timestamp> / -1 or      timestamp of the offsets before that.
  latest / -2 or earliest / -3 or max-     [Note: No offset is returned, if the
  timestamp>                               timestamp greater than recently
                                          committed record timestamp is
                                          given.] (default: latest)
--topic <String: topic>                  The topic to get the offsets for. It
                                          also accepts a regular expression.
                                          If not present, all authorized
                                          topics are queried. Cannot be used
                                          if --topic-partitions is present.
--topic-partitions <String: topic1:1,    Comma separated list of topic-
  topic2:0-3,topic3,topic4:5-,topic5:-3    partition patterns to get the
  >                                        offsets for, with the format of:
                                          ([^:,]*)(?::(?:([0-9]*)|(?:([0-9]*)
                                          -([0-9]*))))?. The first group is
                                          an optional regex for the topic
                                          name, if omitted, it matches any
                                          topic name. The section after ':'
                                          describes a partition pattern,
                                          which can be: a number, a range in
                                          the format of NUMBER-NUMBER (lower
                                          inclusive, upper exclusive), an
                                          inclusive lower bound in the format
                                          of NUMBER-, an exclusive upper
                                          bound in the format of -NUMBER or
                                          may be omitted to accept all
                                          partitions.


kafka-transactions.sh
Use the kafka-transactions tool to list and describe transactions. Use to detect and abort hanging transactions. For more information, see Detect and Abort Hanging Transactions
usage: kafka-transactions.sh [-h] [-v] [--command-config FILE] --bootstrap-server host:port COMMAND ...

This tool is used to analyze the transactional state  of  producers  in  the  cluster. It can be used to detect and
recover from hanging transactions.
optional arguments:
  -h, --help             show this help message and exit
  -v, --version          show the version of this Kafka distribution and exit
  --command-config FILE  property file containing configs to be passed to admin client
  --bootstrap-server host:port
                        hostname and port for the broker to connect  to, in the form `host:port`
                       (multiple comma-separated entries can be given)
commands:
    list                 list transactions
    describe             describe the state of an active transactional-id
    describe-producers   describe the states of active producers for a topic partition
    abort                abort a hanging transaction (requires administrative privileges)
    find-hanging         find hanging transactions

kafka-reassign-partitions.sh
Use the kafka-reassign-partitions to move topic partitions between replicas You pass a JSON-formatted file to specify the new replicas. To learn more, see Changing the replication factor in the Confluent documentation.

This tool helps to move topic partitions between replicas.
Option                                  Description
------                                  -----------
--additional                            Execute this reassignment in addition
                                          to any other ongoing ones. This
                                          option can also be used to change
                                          the throttle of an ongoing
                                          reassignment.
--bootstrap-server <String: Server(s)   REQUIRED: the server(s) to use for
  to use for bootstrapping>               bootstrapping.
--broker-list <String: brokerlist>      The list of brokers to which the
                                          partitions need to be reassigned in
                                          the form "0,1,2". This is required
                                          if --topics-to-move-json-file is
                                          used to generate reassignment
                                          configuration
--cancel                                Cancel an active reassignment.
--command-config <String: Admin client  Property file containing configs to be
  property file>                          passed to Admin Client.
--disable-rack-aware                    Disable rack aware replica assignment
--execute                               Kick off the reassignment as specified
                                          by the --reassignment-json-file
                                          option.
--generate                              Generate a candidate partition
                                          reassignment configuration. Note
                                          that this only generates a candidate
                                          assignment, it does not execute it.
--help                                  Print usage information.
--list                                  List all active partition
                                          reassignments.
--preserve-throttles                    Do not modify broker or topic
                                          throttles.
--reassignment-json-file <String:       The JSON file with the partition
  manual assignment json file path>       reassignment configurationThe format
                                          to use is -
                                        {"partitions":
                                                [{"topic": "foo",
                                                  "partition": 1,
                                                  "replicas": [1,2,3,4],
                                                  "observers":[3,4],
                                                  "log_dirs": ["dir1","dir2","dir3","
                                          dir4"] }],
                                        "version":1
                                        }
                                        Note that "log_dirs" is optional. When
                                          it is specified, its length must
                                          equal the length of the replicas
                                          list. The value in this list can be
                                          either "any" or the absolution path
                                          of the log directory on the broker.
                                          If absolute log directory path is
                                          specified, the replica will be moved
                                          to the specified log directory on
                                          the broker.
                                        Note that  "observers" is optional.
                                          When it is specified it must be a
                                          suffix of the replicas list.
--replica-alter-log-dirs-throttle       The movement of replicas between log
  <Long: replicaAlterLogDirsThrottle>     directories on the same broker will
                                          be throttled to this value
                                          (bytes/sec). This option can be
                                          included with --execute when a
                                          reassignment is started, and it can
                                          be altered by resubmitting the
                                          current reassignment along with the
                                          --additional flag. The throttle rate
                                          should be at least 1 KB/s. (default:
                                          -1)
--throttle <Long: throttle>             The movement of partitions between
                                          brokers will be throttled to this
                                          value (bytes/sec). This option can
                                          be included with --execute when a
                                          reassignment is started, and it can
                                          be altered by resubmitting the
                                          current reassignment along with the
                                          --additional flag. The throttle rate
                                          should be at least 1 KB/s. (default:
                                          -1)
--timeout <Long: timeout>               The maximum time in ms to wait for log
                                          directory replica assignment to
                                          begin. (default: 10000)
--topics-to-move-json-file <String:     Generate a reassignment configuration
  topics to reassign json file path>      to move the partitions of the
                                          specified topics to the list of
                                          brokers specified by the --broker-
                                          list option. The format to use is -
                                        {"topics":
                                                [{"topic": "foo"},{"topic": "foo1"}],
                                        "version":1
                                        }
--verify                                Verify if the reassignment completed
                                          as specified by the --reassignment-
                                          json-file option. If there is a
                                          throttle engaged for the replicas
                                          specified, and the rebalance has
                                          completed, the throttle will be
                                          removed
--version                               Display Kafka version.

kafka-delete-records.sh
Use the kafka-delete-records tool to delete partition records. Use this if a topic receives bad data. Pass a JSON-formatted file that specifies the topic, partition, and offset for data deletion. Data will be deleted up to the offset specified. Example:
bin/kafka-delete-records.sh --bootstrap-server host1:9092 --offset-json-file deleteme.json

kafka-log-dirs.sh
Use the kafka-log-dirs tool to get a list of replicas per log directory on a broker.
This tool helps to query log directory usage on the specified brokers.

------                                  -----------
--bootstrap-server <String: The server  REQUIRED: the server(s) to use for
  (s) to use for bootstrapping>           bootstrapping
--broker-list <String: Broker list>     The list of brokers to be queried in
                                          the form "0,1,2". All brokers in the
                                          cluster will be queried if no broker
                                          list is specified
--command-config <String: Admin client  Property file containing configs to be
  property file>                          passed to Admin Client.
--describe                              Describe the specified log directories
                                          on the specified brokers.
--help                                  Print usage information.
--topic-list <String: Topic list>       The list of topics to be queried in
                                          the form "topic1,topic2,topic3". All
                                          topics will be queried if no topic
                                          list is specified (default: )
--version                               Display Kafka version.

kafka-replica-verification.sh
Use the kafka-replica-verification tool to verify that all replicas of a topic contain the same data. Requires a broker-list parameter that contains a comma-separated list of <hostname:port> entries specifying the server/port to connect to.

Validate that all replicas for a set of topics have the same data.
Option                                  Description
------                                  -----------
--broker-list <String: hostname:        REQUIRED: The list of hostname and
  port,...,hostname:port>                 port of the server to connect to.
--fetch-size <Integer: bytes>           The fetch size of each request.
                                          (default: 1048576)
--help                                  Print usage information.
--max-wait-ms <Integer: ms>             The max amount of time each fetch
                                          request waits. (default: 1000)
--report-interval-ms <Long: ms>         The reporting interval. (default:
                                          30000)
--time <Long: timestamp/-1(latest)/-2   Timestamp for getting the initial
  (earliest)>                             offsets. (default: -1)
--topic-white-list <String: Java regex  DEPRECATED use --topics-include
  (String)>                               instead; ignored if --topics-include
                                          specified. List of topics to verify
                                          replica consistency. Defaults to '.
                                          *' (all topics) (default: .*)
--topics-include <String: Java regex    List of topics to verify replica
  (String)>                               consistency. Defaults to '.*' (all
                                          topics) (default: .*)
--version                               Print version information and exit.

connect-mirror-maker.sh
Use the connect-mirror-maker tool to replicate topics from one cluster to another using the Connect framework. You must pass an an mm2.properties MM2 configuration file. For more information, see KIP-382: MirrorMaker 2.0 or Getting up to speed with MirrorMaker 2.

kafka-verifiable-consumer.sh
The kafka-verifiable-consumer tool consumes messages from a topic and emits consumer events as JSON objects to STDOUT. For example, group rebalances, received messages, and offsets committed. Intended for internal testing.

kafka-verifiable-producer.sh
The kafka-verifiable-producer tool produces increasing integers to the specified topic and prints JSON metadata to STDOUT on each send request. This tool shows which messages have been acked and which have not. This tool is intended for internal testing.

kafka-producer-perf-test.sh
The kafka-producer-perf-test tool enables you to produce a large quantity of data to test producer performance for the Kafka cluster.

usage: producer-perf-test [-h] --topic TOPIC --num-records NUM-RECORDS
                            [--payload-delimiter PAYLOAD-DELIMITER] --throughput THROUGHPUT
                            [--producer-props PROP-NAME=PROP-VALUE [PROP-NAME=PROP-VALUE ...]]
                            [--producer.config CONFIG-FILE] [--print-metrics]
                            [--transactional-id TRANSACTIONAL-ID]
                            [--transaction-duration-ms TRANSACTION-DURATION]
                            (--record-size RECORD-SIZE | --payload-file PAYLOAD-FILE)

This tool is used to verify the producer performance.
optional arguments:
  -h, --help             show this help message and exit
  --topic TOPIC          produce messages to this topic
  --num-records NUM-RECORDS
                        number of messages to produce
  --payload-delimiter PAYLOAD-DELIMITER
                        provides  delimiter  to  be  used  when  --payload-file  is  provided.
                        Defaults to new line. Note that  this  parameter will be ignored if --
                        payload-file is not provided. (default: \n)
  --throughput THROUGHPUT
                        throttle maximum  message  throughput  to  *approximately*  THROUGHPUT
                        messages/sec. Set this to -1 to disable throttling.
  --producer-props PROP-NAME=PROP-VALUE [PROP-NAME=PROP-VALUE ...]
                        kafka  producer  related  configuration   properties  like  bootstrap.
                        servers, client.id  etc.  These  configs  take  precedence  over  those
                        passed via --producer.config.
  --producer.config CONFIG-FILE
                        producer config properties file.
  --print-metrics        print out metrics at the end of the test. (default: false)
  --transactional-id TRANSACTIONAL-ID
                        The transactionalId to use if  transaction-duration-ms  is > 0. Useful
                        when testing the  performance  of  concurrent  transactions. (default:
                        performance-producer-default-transactional-id)
  --transaction-duration-ms TRANSACTION-DURATION
                        The max age of each transaction.  The commitTransaction will be called
                        after this time has  elapsed.  Transactions  are  only enabled if this
                        value is positive. (default: 0)

  either --record-size or --payload-file must be specified but not both.

  --record-size RECORD-SIZE
                        message size in bytes. Note that  you  must  provide exactly one of --
                        record-size or --payload-file.
  --payload-file PAYLOAD-FILE
                        file to read the  message  payloads  from.  This  works only for UTF-8
                        encoded text files.  Payloads  will  be  read  from  this  file  and a
                        payload will be randomly  selected  when  sending  messages. Note that
                        you must provide exactly one of --record-size or --payload-file.

kafka-consumer-groups.sh
Use the kafka-consumer-groups tool to get a list of the active groups in the cluster.
bin/kafka-consumer-groups.sh \
         --bootstrap-server localhost:9092 \
         --describe --group user-group
TOPIC          PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG CONSUMER-ID       HOST         CLIENT-ID
user           0          2               4               2   consumer-1-...    /127.0.0.1   consumer-1
user           1          2               3               1   consumer-1-...    /127.0.0.1   consumer-1
user           2          2               3               1   consumer-2-...    /127.0.0.1   consumer-2

Option                                  Description
------                                  -----------
--all-groups                            Apply to all consumer groups.
--all-topics                            Consider all topics assigned to a
                                          group in the `reset-offsets` process.
--bootstrap-server <String: server to   REQUIRED: The server(s) to connect to.
  connect to>
--by-duration <String: duration>        Reset offsets to offset by duration
                                          from current timestamp. Format:
                                          'PnDTnHnMnS'
--command-config <String: command       Property file containing configs to be
  config property file>                   passed to Admin Client and Consumer.
--delete                                Pass in groups to delete topic
                                          partition offsets and ownership
                                          information over the entire consumer
                                          group. For instance --group g1 --
                                          group g2
--delete-offsets                        Delete offsets of consumer group.
                                          Supports one consumer group at the
                                          time, and multiple topics.
--describe                              Describe consumer group and list
                                          offset lag (number of messages not
                                          yet processed) related to given
                                          group.
--dry-run                               Only show results without executing
                                          changes on Consumer Groups.
                                          Supported operations: reset-offsets.
--execute                               Execute operation. Supported
                                          operations: reset-offsets.
--export                                Export operation execution to a CSV
                                          file. Supported operations: reset-
                                          offsets.
--from-file <String: path to CSV file>  Reset offsets to values defined in CSV
                                          file.
--group <String: consumer group>        The consumer group we wish to act on.
--help                                  Print usage information.
--list                                  List all consumer groups.
--members                               Describe members of the group. This
                                          option may be used with '--describe'
                                          and '--bootstrap-server' options
                                          only.
                                        Example: --bootstrap-server localhost:
                                          9092 --describe --group group1 --
                                          members
--offsets                               Describe the group and list all topic
                                          partitions in the group along with
                                          their offset lag. This is the
                                          default sub-action of and may be
                                          used with '--describe' and '--
                                          bootstrap-server' options only.
                                        Example: --bootstrap-server localhost:
                                          9092 --describe --group group1 --
                                          offsets
--reset-offsets                         Reset offsets of consumer group.
                                          Supports one consumer group at the
                                          time, and instances should be
                                          inactive
                                        Has 2 execution options: --dry-run
                                          (the default) to plan which offsets
                                          to reset, and --execute to update
                                          the offsets. Additionally, the --
                                          export option is used to export the
                                          results to a CSV format.
                                        You must choose one of the following
                                          reset specifications: --to-datetime,
                                          --by-period, --to-earliest, --to-
                                          latest, --shift-by, --from-file, --
                                          to-current.
                                        To define the scope use --all-topics
                                          or --topic. One scope must be
                                          specified unless you use '--from-
                                          file'.
--shift-by <Long: number-of-offsets>    Reset offsets shifting current offset
                                          by 'n', where 'n' can be positive or
                                          negative.
--state [String]                        When specified with '--describe',
                                          includes the state of the group.
                                        Example: --bootstrap-server localhost:
                                          9092 --describe --group group1 --
                                          state
                                        When specified with '--list', it
                                          displays the state of all groups. It
                                          can also be used to list groups with
                                          specific states.
                                        Example: --bootstrap-server localhost:
                                          9092 --list --state stable,empty
                                        This option may be used with '--
                                          describe', '--list' and '--bootstrap-
                                          server' options only.
--timeout <Long: timeout (ms)>          The timeout that can be set for some
                                          use cases. For example, it can be
                                          used when describing the group to
                                          specify the maximum amount of time
                                          in milliseconds to wait before the
                                          group stabilizes (when the group is
                                          just created, or is going through
                                          some changes). (default: 5000)
--to-current                            Reset offsets to current offset.
--to-datetime <String: datetime>        Reset offsets to offset from datetime.
                                          Format: 'YYYY-MM-DDTHH:mm:SS.sss'
--to-earliest                           Reset offsets to earliest offset.
--to-latest                             Reset offsets to latest offset.
--to-offset <Long: offset>              Reset offsets to a specific offset.
--topic <String: topic>                 The topic whose consumer group
                                          information should be deleted or
                                          topic whose should be included in
                                          the reset offset process. In `reset-
                                          offsets` case, partitions can be
                                          specified using this format: `topic1:
                                          0,1,2`, where 0,1,2 are the
                                          partition to be included in the
                                          process. Reset-offsets also supports
                                          multiple topic inputs.
--verbose                               Provide additional information, if
                                          any, when describing the group. This
                                          option may be used with '--
                                          offsets'/'--members'/'--state' and
                                          '--bootstrap-server' options only.
                                        Example: --bootstrap-server localhost:
                                          9092 --describe --group group1 --
                                          members --verbose
--version                               Display Kafka version.


kafka-consumer-perf-test.sh
This tool tests the consumer performance for the Kafka cluster.
Option                                   Description
------                                   -----------
--bootstrap-server <String: server to    REQUIRED unless --broker-list
  connect to>                              (deprecated) is specified. The server
                                          (s) to connect to.
--broker-list <String: broker-list>      DEPRECATED, use --bootstrap-server
                                          instead; ignored if --bootstrap-
                                          server is specified.  The broker
                                          list string in the form HOST1:PORT1,
                                          HOST2:PORT2.
--consumer.config <String: config file>  Consumer config properties file.
--date-format <String: date format>      The date format to use for formatting
                                          the time field. See java.text.
                                          SimpleDateFormat for options.
                                          (default: yyyy-MM-dd HH:mm:ss:SSS)
--fetch-size <Integer: size>             The amount of data to fetch in a
                                          single request. (default: 1048576)
--from-latest                            If the consumer does not already have
                                          an established offset to consume
                                          from, start with the latest message
                                          present in the log rather than the
                                          earliest message.
--group <String: gid>                    The group id to consume on. (default:
                                          perf-consumer-50334)
--help                                   Print usage information.
--hide-header                            If set, skips printing the header for
                                          the stats
--messages <Long: count>                 REQUIRED: The number of messages to
                                          send or consume
--num-fetch-threads <Integer: count>     DEPRECATED AND IGNORED: Number of
                                          fetcher threads. (default: 1)
--print-metrics                          Print out the metrics.
--reporting-interval <Integer:           Interval in milliseconds at which to
  interval_ms>                             print progress info. (default: 5000)
--show-detailed-stats                    If set, stats are reported for each
                                          reporting interval as configured by
                                          reporting-interval
--socket-buffer-size <Integer: size>     The size of the tcp RECV size.
                                          (default: 2097152)
--threads <Integer: count>               DEPRECATED AND IGNORED: Number of
                                          processing threads. (default: 10)
--timeout [Long: milliseconds]           The maximum allowed time in
                                          milliseconds between returned
                                          records. (default: 10000)
--topic <String: topic>                  REQUIRED: The topic to consume from.
--version                                Display Kafka version.

kafka-acls
Use the kafka-acls tool to add, remove and list ACLs. For example, if you wanted to add two principal users, Jose and Jane to have read and write permissions on the user topic from specific IP addresses, you could use a command like the following:
bin/kafka-acls.sh --bootstrap-server localhost:9092 --add --allow-principal User:Jose --allow-principal User:Jane --allow-host 198.51.100.0 --allow-host 198.51.100.1 --operation Read --operation Write --topic user

kafka-delegation-tokens.sh
Use the kafka-delegation-tokens tool to create, renew, expire and describe delegation tokens. Delegation tokens are shared secrets between Kafka brokers and clients, and are a lightweight authentication mechanism meant to complement existing SASL/SSL methods.

kafka-e2e-latency.sh
The kafka-e2e-latency tool is a performance testing tool used to measure end-to-end latency in Kafka. It works by sending messages to a Kafka topic and then consuming those messages from a Kafka consumer. The tool calculates the time difference between when a message was produced and when it was consumed, giving you an idea of the end-to-end latency for your Kafka cluster. This tool is useful for testing the performance of your Kafka cluster and identifying any bottlenecks or issues that may be affecting latency.

Following are the required arguments
        broker_list: The location of the bootstrap broker for both the producer and the consumer
        topic: The topic name used by both the producer and the consumer to send/receive messages
        num_messages: The number of messages to send
        producer_acks: The producer setting for acks.
        message_size_bytes: size of each message in bytes

For example:
kafka-e2e-latency.sh localhost:9092 test 10000 1 20

acks
The number of acknowledgments the producer requires the leader to have received before considering a request complete. This controls the durability of records that are sent. The following settings are allowed:

    acks=0 If set to zero then the producer will not wait for any acknowledgment from the server at all. The record will be immediately added to the socket buffer and considered sent. No guarantee can be made that the server has received the record in this case, and the retries configuration will not take effect (as the client won’t generally know of any failures). The offset given back for each record will always be set to -1.
    acks=1 This will mean the leader will write the record to its local log but will respond without awaiting full acknowledgement from all followers. In this case should the leader fail immediately after acknowledging the record but before the followers have replicated it then the record will be lost.
    acks=all This means the leader will wait for the full set of in-sync replicas to acknowledge the record. This guarantees that the record will not be lost as long as at least one in-sync replica remains alive. This is the strongest available guarantee. This is equivalent to the acks=-1 setting.

USAGE: java org.apache.kafka.tools.EndToEndLatency broker_list topic num_messages producer_acks message_size_bytes [optional] properties_file

kafka-dump-log.sh
The kafka-dump-log tool can be used in KRaft mode to parse a metadata log file and output its contents to the console. Requires a comma-separated list of log files. The tool will scan the provided files and decode the metadata records.
The following example shows using the cluster-metadata-decoder argument to decode the metadata records in a log segment.
bin/kafka-dump-log.sh --cluster-metadata-decoder --files tmp/kraft-combined-logs/_cluster_metadata-0/00000000000000023946.log

kafka-jmx.sh
The kafka-jmx tool enables you to read JMX metrics from a given endpoint. This tool only works reliably if the JmxServer is fully initialized prior to invoking the tool.

trogdor.sh
Trogdor is a test framework for Kafka. Trogdor can run benchmarks and other workloads. Trogdor can also inject faults in order to stress test the system. 

The Trogdor fault injector.
Usage:
  ./trogdor.sh [action] [options]
Actions:
  agent: Run the trogdor agent.
  coordinator: Run the trogdor coordinator.
  client: Run the client which communicates with the trogdor coordinator.
  agent-client: Run the client which communicates with the trogdor agent.
  help: This help message.
````
## Kafka CLI with examples
```bash
// ./kafka-topics.sh --list --zookeeper localhost:2181
// Option zookeeper is deprecated, use --bootstrap-server instead.

./kafka-topics.sh --create --topic registrations --replication-factor 1 --partitions 1 --bootstrap-server localhost:9092
..
Created topic registrations.

./kafka-topics.sh --list --bootstrap-server localhost:9092
..
registrations

./kafka-topics.sh --bootstrap-server=localhost:9092 --describe --topic registrations
..    
Topic: registrations	TopicId: 9cBL5BFJQMePGzm9dfaUtg	PartitionCount: 1	ReplicationFactor: 1	Configs: 
Topic: registrations	Partition: 0	Leader: 0	Replicas: 0	Isr: 0

./kafka-topics.sh --delete --bootstrap-server localhost:9092 --topic registrations

./kafka-console-producer.sh --topic users.registrations --bootstrap-server localhost:9092
>Hello Kafka
>Umm

./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092
Теперь мы, по идее, должны увидеть записанное сообщение. Но ничего не происходит.
С этой проблемой сталкиваются многие, кто начинает использовать Кафку. Это не значит, что сообщения потерялись, или что-то не работает. Все проще: консьюмер Кафки по умолчанию начинает читать данные с конца топика в тот момент, когда он запустился (см. настройку auto.offset.reset). Поэтому, чтобы прочитать данные, записанные ДО старта консьюмера, нужно переопределить эту конфигу.

Значение earliest показывает, что чтение записей будет начинаться с самого раннего доступного сообщения. Вот так:
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic users.registrations --consumer-property auto.offset.reset=earliest
..
Hello Kafka
Umm
Processed a total of 2 messages

Shortcut --from-beginning:
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic users.registrations --from-beginning --max-messages 2
..
Hello Kafka
Umm
Processed a total of 2 messages

// Pass the group name
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic users.registrations --from-beginning --max-messages 2 --group slurp

./kafka-cluster.sh cluster-id --bootstrap-server localhost:9092
..
Cluster ID: cy984x31RQuDE_ytCIYXqA

./kafka-broker-api-versions.sh --bootstrap-server localhost:9092 --version
..
3.6.1

./kafka-transactions.sh --bootstrap-server localhost:9092 list
..
TransactionalId	Coordinator	ProducerId	TransactionState

// Delete partition records, if topic receives bad data
cat /tmp/offset_example.json
..
{"partitions":
  [{"topic": "users.registrations", "partition": 0,
  "offset": 1}],
  "version":1
}

./kafka-console-consumer.sh  --bootstrap-server localhost:9092 --topic users.registrations --from-beginning
Hello Kafka
Umm
Hmmm
???
^CProcessed a total of 4 messages

./kafka-delete-records.sh --bootstrap-server localhost:9092 --offset-json-file /tmp/offset_example.json
..
Executing records delete operation
Records delete operation completed:
partition: users.registrations-0	low_watermark: 1

./kafka-console-consumer.sh  --bootstrap-server localhost:9092 --topic users.registrations --from-beginning
..
Umm
Hmmm
???
^CProcessed a total of 3 messages

Но оно работает так что оффсеты не сбрасываются. Отсчёт идёт от начала, если мы удалили 1 и 2 оффсеты, чтобы потом отдельной командой после этого удалить 3 и 4 нужно будет указывать 4, а не 2. Но возможно это как то можно сбросить, отфрагментировать хз.

Если указать оффсет за пределами текущих, то будет ошибка и сообщения в топике не удалятся
Executing records delete operation
Records delete operation completed:
[2024-01-26 04:44:06,559] ERROR [AdminClient clientId=adminclient-1] DeleteRecords request for topic partition users.registrations-0 failed due to an unexpected error OFFSET_OUT_OF_RANGE (org.apache.kafka.clients.admin.internals.DeleteRecordsHandler)
partition: users.registrations-0	error: org.apache.kafka.common.errors.OffsetOutOfRangeException: The requested offset is not within the range of offsets maintained by the server.

Задавая оффсет -1 можно удалить все сообщения в топике:

cat /tmp/offset_example.json
..
{"partitions":
  [{"topic": "users.registrations", "partition": 0,
  "offset": -1}],
  "version":1
}

// Perfomance tests
./kafka-producer-perf-test.sh --topic users.registrations --num-records 20000 --record-size 1000 --throughput 10000000 --producer-props bootstrap.servers=localhost:9092
..
20000 records sent, 13745.704467 records/sec (13.11 MB/sec), 507.94 ms avg latency, 725.00 ms max latency, 592 ms 50th, 716 ms 95th, 723 ms 99th, 724 ms 99.9th

Then we can clear them out like in previous example with offset -1:
./kafka-delete-records.sh --bootstrap-server localhost:9092 --offset-json-file /tmp/offset_example.json


./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --all-groups
GROUP           TOPIC               PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                           HOST             CLIENT-ID
slurp           users.registrations 0          20012           20012           0               console-consumer-30cc3146-66c0-42e5-a4b1-712937c3a8bd /192.168.122.251 console-consumer

./kafka-e2e-latency.sh localhost:9092 users.registrations 10000 1 2000
..
0	64.69036
1000	0.538343
2000	0.41516000000000003
3000	0.459293
4000	0.31889999999999996
5000	0.377733
6000	0.395324
7000	0.439079
8000	0.276823
9000	0.25577099999999997
Avg latency: 0.7439 ms
Percentiles: 50th = 0, 99th = 11, 99.9th = 13
```
### About offset
````
./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest --group slurp 

После запуска консьюмера мы увидим те же записанные ранее сообщения. При этом, в логе брокера отобразится, что консьюмер подключился с группой slurm, которую мы передали. Закрываем консьюмер, перезапускаем его снова той же командой. Сообщения опять пропали!
Почему? Консьюмер группы в Кафке может коммитить свои оффсеты (свою позицию) для какой-то партиции, которую он уже прочитал. Чтобы при перезапуске продолжить обработку с этой позиции. Именно это поведение мы здесь и наблюдаем. Хотя лучше лишний раз проверить.

./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurp --describe
..
Consumer group 'slurp' has no active members.
GROUP           TOPIC               PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
slurp           users.registrations 0          30012           30012           0               -               -               -

Сейчас ни один инстанс группы не живет, но мы все равно можем проверить ее сохраненный стейт. Мы видим, что никакого активного члена группы у нас нет (Consumer group ‘slurm’ has no active members), и это правильно. Еще мы видим, что эта группа в топике registrations в партиции 0 сохранила свою позицию на offset-е 2 (CURRENT-OFFSET). Именно этот offset является концом топика. LAG у нас 0, значит консьюмер полностью прочитал все сообщения и не лагает. Получается, что наш консольный консьюмер автоматически закоммитил свою позицию.

А если законнектиться консьюмером, то его видно и всю дату по нему
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurp --describe
..
GROUP           TOPIC               PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                           HOST             CLIENT-ID
slurp           users.registrations 0          30012           30012           0               console-consumer-1f4cd273-d78d-4b5f-ad2b-c27a99d97324 /192.168.122.251 console-consumer

Что можно сделать? Можем сбросить позицию консьюмера на начало. Разумеется, если вы используете клиенты, у вас будет целый набор инструментов для удобного контроля позиции своего консьюмера в любой момент.
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurp --reset-offsets --to-earliest --topic users.registrations --execute
..
GROUP                          TOPIC                          PARTITION  NEW-OFFSET     
slurp                          users.registrations            0          30012

Надо сказать, если в топике были удалены прошлые сообщения, а после оффсетов удалённых сообщений идут "живые" сообщения, то он при reset поставит оффсет на первое "живое" с которого сможет прочитать все дальнейшие сообщеения.

А если например сделать ресет, затем удалить живые сообщения, затем запустить консьюмер он поставит последний оффсет без вывода сообщений.
А в логах кафки вот что будет, вобщем кафка сервер сам поправляет оффсет:
INFO [UnifiedLog partition=users.registrations-0, dir=/home/kafka/kafka-logs] Incremented log start offset to 30015 due to client delete records request (kafka.log.UnifiedLog)
Спустя 30 секунд  - Dynamic member with unknown member id joins group slurp in Empty state. Created a new member id console-consumer-95035ee3-90dc-41cd-8c84-06f4ec36d305 and request the member to rejoin with this id. (kafka.coordinator.group.GroupCoordinator)
````
### Topic Retention
Этот механизм служит основным способом удаления данных из Кафки. Мы можем включить его по времени или по размеру партиции. Рассмотрим retention по времени. На данном этапе в нашем топике этот механизм не настроен, поэтому данные будут храниться вечно (до тех пор, пока диск на брокере не заполнится до предела).

Для начала изменим одну из настроек брокера, чтобы облегчить себе жизнь: нам будет видно, что происходит с данными после включения retention. Останавливаем брокер, если он уже запущен. Копируем конфигурационный файл, с которым мы изначально запустили этот брокер
````
cp server.properties slurp-server.properties
````
Открываем этот конфиг. Настройка, которую будем менять, называется log.retention.check.interval.ms. Она диктует частоту, с которой удаляющий данные с диска тред (LogCleaner) проверяет retention. Значение по умолчанию — 5 минут. Для production-систем это замечательно. Однако мы будем менять конфиги, поэтому нам хочется видеть отклик быстрее. Поменяем значение на 1 секунду: log.retention.check.interval.ms=1000

Таким образом LogCleaner будет проверять данные для возможного удаления раз в секунду. Сохраняем файл и запускаем сервер с новой конфигурацией. Сделано.

Теперь включим retention у топика, а также заальтерим одну из конфигурационных опций — retention.ms. Выставим значение 60000 (одна минута). Для этого воспользуемся скриптом 
````
./kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name users.registrations --alter --add-config retention.ms=60000
````
Пошли логи:
````
Jan 26 10:07:10 kafka-01 kafka-server-start.sh[15550]: [2024-01-26 10:07:10,015] INFO [Admin Manager on Broker 0]: Updating topic users.registrations with new configuration : retention.ms -> 60000 (kafka.server.ZkAdminManager)
Jan 26 10:07:10 kafka-01 kafka-server-start.sh[15550]: [2024-01-26 10:07:10,087] INFO Processing notification(s) to /config/changes (kafka.common.ZkNodeChangeNotificationListener)
Jan 26 10:07:10 kafka-01 kafka-server-start.sh[15550]: [2024-01-26 10:07:10,097] INFO Processing override for entityPath: topics/users.registrations with config: HashMap(retention.ms -> 60000) (kafka.server.ZkConfigManager)
Jan 26 10:07:14 kafka-01 kafka-server-start.sh[15550]: [2024-01-26 10:07:14,239] INFO [LocalLog partition=users.registrations-0, dir=/home/kafka/kafka-logs] Rolled new log segment at offset 30015 in 4 ms. (kafka.log.LocalLog)
Jan 26 10:07:14 kafka-01 kafka-server-start.sh[15550]: [2024-01-26 10:07:14,239] INFO [UnifiedLog partition=users.registrations-0, dir=/home/kafka/kafka-logs] Deleting segment LogSegment(baseOffset=0, size=40957101, lastModifiedTime=1706278933455, largestRecordTimestamp=Some(1706278932449)) due to log retention time 60000ms breach based on the largest record timestamp in the segment (kafka.log.UnifiedLog)
...
Jan 26 10:08:14 kafka-01 kafka-server-start.sh[15550]: [2024-01-26 10:08:14,243] INFO [LocalLog partition=users.registrations-0, dir=/home/kafka/kafka-logs] Deleting segment files LogSegment(baseOffset=0, size=40957101, lastModifiedTime=1706278933455, largestRecordTimestamp=Some(1706278932449)) (kafka.log.LocalLog$)
Jan 26 10:08:14 kafka-01 kafka-server-start.sh[15550]: [2024-01-26 10:08:14,246] INFO Deleted log /home/kafka/kafka-logs/users.registrations-0/00000000000000000000.log.deleted. (kafka.log.LogSegment)
Jan 26 10:08:14 kafka-01 kafka-server-start.sh[15550]: [2024-01-26 10:08:14,247] INFO Deleted offset index /home/kafka/kafka-logs/users.registrations-0/00000000000000000000.index.deleted. (kafka.log.LogSegment)
Jan 26 10:08:14 kafka-01 kafka-server-start.sh[15550]: [2024-01-26 10:08:14,247] INFO Deleted time index /home/kafka/kafka-logs/users.registrations-0/00000000000000000000.timeindex.deleted. (kafka.log.LogSegment)
````
Место в директории kafka-logs высвободилось.

Проведем эксперимент. Скажем Кафке удалять данные из топика после 10 секунд:
````
./kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name users.registrations --alter --add-config retention.ms=10000
````
Теперь запустим консольного продюсера, чтобы он считывал все новые лайны из файла и перекидывал их в Кафку. Скрипт такой: 
````
touch /tmp/data && tail -f -n0 /tmp/data | ./kafka-console-producer.sh --topic users.registrations --bootstrap-server=localhost:9092 --sync
````
Он создает файл /tmp/data, тейлит этот файл и передает весь output консольному продюсеру, чтобы тот писал эти сообщения в наш топик registrations. Теперь откроем другое окно и запустим еще один скрипт: 
````
for i in $(seq 1 3600); do echo "test${i}" >> /tmp/data; sleep 1; done
````
Он будет каждую секунду аппендить новые лайны в этот файл: test1, test2, test3 и так далее до 3600. Все лайны будут автоматически передаваться нашему продюсеру. Открываем третье окно и запускаем консольный консьюмер, чтобы посмотреть, какие сообщения хранятся сейчас в топике:
````
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic users.registrations --from-beginning
````
Мы задали настройку, чтобы наши сообщения удалялись после 10 секунд. Также мы отправляем test1, test2, test3 и далее в наш топик registrations раз в секунду.

Видим следующее: в топике до сих пор хранятся все сообщения, несмотря на то, что прошло уже больше 10 секунд. Более того, мы явно указали Кафке, что чекер LogCleaner-а должен проверять данные на удаление раз в секунду. Давайте запустим консьюмер еще раз. Мы снова видим все сообщения в топике. Что не так?

Давайте разбираться. Нам нужно заглянуть во внутреннюю структуру данных партиции и понять: как именно Кафка сохраняет данные на диск.

Кстати когда отключился и подождал вот так получилось:
````
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurp --describe
..
Consumer group 'slurp' has no active members.

GROUP           TOPIC               PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
slurp           users.registrations 0          30151           30159           8               -               -               -

./kafka-console-consumer.sh  --bootstrap-server localhost:9092 --topic users.registrations --from-beginning --group slurp
..
^CProcessed a total of 0 messages

./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurp --describe
..
Consumer group 'slurp' has no active members.

GROUP           TOPIC               PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
slurp           users.registrations 0          30159           30159           0               -               -               -
````
Сообщения в топике пропали, а также кафка сервер хранил LAG, пока консьюмер не подключился и оффсет не поправился

Похоже вот он сегмент и данные по нему, именуется последним оффсетом
````
ls -lh ~/kafka-logs/users.registrations-0/
..
-rw-r--r--. 1 kafka kafka   0 Jan 26 10:34 00000000000000030159.log
-rw-r--r--. 1 kafka kafka  56 Jan 26 10:34 00000000000000030159.snapshot
-rw-r--r--. 1 kafka kafka 10M Jan 26 10:34 00000000000000030159.timeindex
-rw-r--r--. 1 kafka kafka  12 Jan 26 10:34 leader-epoch-checkpoint
-rw-r--r--. 1 kafka kafka  43 Jan 26 03:14 partition.metadata
````
А вот и настройки для контроля размера сегмента

- log.segment.bytes: the max size of a single segment in bytes (default 1 GB)
- log.segment.ms: the time Kafka will wait before committing the segment if not full (default 1 week)

И вот что важно

- A Kafka broker keeps an open file handle to every segment in every partition - even inactive segments. This leads to a usually high number of open file handles, and the OS must be tuned accordingly.
#### Структура партиции
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/kafka%20segments.png?raw=true)
Разбираемся, как Кафка хранит данные на диске. Партиции состоят из набора файлов, которые называются сегментами. Данные, которые продюсеры присылают брокеру, сохраняются в открытый или головной сегмент партиции. Через некоторое время, согласно некоторому набору правил, он роллапится (закрывается). После этого открывается новый сегмент. Закрытые сегменты хранятся на диске, но при этом в них никогда уже не происходит запись (они становятся полностью иммутабельными). Важно понимать, что LogCleaner Кафки удаляет данные исключительно посегментно. То есть, он удаляет файлы целиком. Для того чтобы LogCleaner понял, можно удалять файл или нет (если мы говорим о retention по времени), он производит следующий простой набор операций:

- находит максимальный таймстамп сообщения внутри одного сегмента;
- находит разницу между максимальным таймстампом и текущим временем;
- определяет, больше ли эта разница во времени, чем заданный конфигурационной опцией retention.ms;
- если разница больше, то сегмент уже старый, его можно удалить.

До этого мы писали сообщения continuously в открытый сегмент партиции топика registrations, поэтому постоянно увеличивали максимальный таймстамп (раз в секунду), тем самым не давая LogCleaner-у удалить этот сегмент. 

Напомним, что retention.ms у нас был выставлен в 10 секунд. Разница во времени никогда не превысит это заданное значение, потому что мы постоянно дописываем сообщения. Если бы мы остановили продюсера и подождали 10 секунд, то увидели бы, что данные удалены. Плюс, если бы текущий сегмент закрылся, то оказавшиеся в нем данные очень быстро удалились. 

Но эта операция роллапа не происходила, потому что изначально мы не изменили дефолтные настройки. Их две:

1. segment.ms — период роллапа сегмента после его открытия, 1 неделя по умолчанию
2. segment.bytes — максимальный размер сегмента, 1 ГБ по умолчанию.

Понятно, что мы не написали данных на 1 ГБ и точно не прождали неделю, чтобы дождаться retention-а. В этом случае мы можем выйти из ситуации двумя способами: выставить segment.bytes на очень маленькое значение (пару КБ) или сказать segment.ms роллапить сегмент чаще, чем раз в неделю (через 10 секунд, например).

Важно сказать, что обе эти настройки работают одновременно по правилу ИЛИ, поэтому контролировать их можно (и нужно) одновременно.

Мы еще не затрагивали retention по байтам, но он очень простой — это максимальный размер партиции на диске в байтах. Этой настройкой приходится пользоваться не так уж и часто, потому что сложно сказать, как долго хранятся данные. Это сильно зависит от того, с какой скоростью записываются данные на диск. Может быть, сегодня продюсер отправляет по 10 КБ в секунду, а в дальнейшем объем данных вырастет, и они начнут удаляться быстрее при условии сохранения старых настроек. Но есть и плюс: retention по байтам защищает ваших брокеров от переполнения данными.

К слову, retention.ms и retention.bytes также работают по правилу ИЛИ, поэтому их можно задать одновременно. Допустим: мы сохраняем данные минимум на неделю, а еще ограничиваем максимальный размер партиции в 1 ТБ.

Еще один момент: большая часть настроек Кафки может быть реализована на двух уровнях.

Уровень брокера или сервера содержит дефолты всех настроек и часто имеет префикс log. Например, log.retention.ms — это глобальный дефолт retention-а для всех топиков, который задается в конфигурационном файле сервера server.properties. Topic-level конфиги — это оверрайды для отдельных топиков, которые мы задавали через команду kafka-configs.sh. Их значения хранятся в ZooKeeper. 

Пользоваться можно любыми из настроек. Работают они, по большому счету, одинаково. Практический совет: можно выставить разумные дефолтные настройки на уровне брокера, а уже для конкретных топиков задавать индивидуальные настройки. Полный перечень настроек ищите на сайте самой Кафки: https://kafka.apache.org/documentation/#configuration.

Теперь переходим к борьбе с проблемой неудаляющихся данных, с которой столкнулись ранее.

Снова открываем консольный консьюмер (--topic registrations), останавливая при этом продюсер. Через 15 секунд все сообщения из топика будут удалены. Мы знаем, как хранятся файлы, поэтому давайте заглянем в папку и узнаем, что там лежит. По умолчанию хранение происходит в папке /tmp/kafka-logs/. Здесь куча разных папок, но нас интересует registrations 0 (топик registrations, партиция 0). 
````
ls -la /tmp/kafka-logs/registrations-0
````
Здесь есть только 1 файл, но он абсолютно пустой, потому что все данные из него были удалены.

Приступим к настройке. В первую очередь, поменяем segment.ms у нашего топика: зададим override и скажем, что хотим роллапить сегменты для этого топика раз в 10 секунд. Для этого воспользуемся командой:
````
./kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name registrations --alter --add-config segment.ms=10000
````
После этого запускаем консольного продюсера, который тейлит файл tmp/data, а затем — форлуп, который генерит сообщения раз в секунду. Запись началась!

Прежде чем запускать консольного консьюмера, заглянем в папку /tmp/kafka-logs/ и увидим, что динамика есть. Файлы роллапятся. Файл с самым большим оффсетом — наш головной сегмент. Процесс идет таким образом: старые сегменты закрываются, новые открываются. Старые сегменты при этом помечаются как deleted, затем еще один бэкграунд тред полностью удаляет их с диска.
````
[kafka@kafka-01 users.registrations-0]$ ls -lh
total 84K
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:33 00000000000000030159.index
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:33 00000000000000030159.log
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:33 00000000000000030159.timeindex
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:33 00000000000000030169.index
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:33 00000000000000030169.log
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:33 00000000000000030169.snapshot
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:33 00000000000000030169.timeindex
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:34 00000000000000030179.index
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:34 00000000000000030179.log
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:33 00000000000000030179.snapshot
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:34 00000000000000030179.timeindex
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:34 00000000000000030189.index
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:34 00000000000000030189.log
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:34 00000000000000030189.snapshot
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:34 00000000000000030189.timeindex
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:34 00000000000000030199.index
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:34 00000000000000030199.log
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:34 00000000000000030199.snapshot
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:34 00000000000000030199.timeindex
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:34 00000000000000030209.index
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:34 00000000000000030209.log
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:34 00000000000000030209.snapshot
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:34 00000000000000030209.timeindex
-rw-r--r--. 1 kafka kafka 10M Jan 27 01:34 00000000000000030219.index
-rw-r--r--. 1 kafka kafka 355 Jan 27 01:34 00000000000000030219.log
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:34 00000000000000030219.snapshot
-rw-r--r--. 1 kafka kafka 10M Jan 27 01:34 00000000000000030219.timeindex
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:16 leader-epoch-checkpoint
-rw-r--r--. 1 kafka kafka  43 Jan 26 03:14 partition.metadata
````
Затем я останавливаю продюсер...

У меня был выставлен большой retention.ms, я поправил на 10 секунд (retention.ms=10000) и увидел эту пометку 
````
[kafka@kafka-01 users.registrations-0]$ ls -lh
total 116K
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:33 00000000000000030159.index.deleted
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:33 00000000000000030159.log.deleted
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:33 00000000000000030159.timeindex.deleted
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:33 00000000000000030169.index.deleted
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:33 00000000000000030169.log.deleted
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:33 00000000000000030169.snapshot.deleted
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:33 00000000000000030169.timeindex.deleted
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:34 00000000000000030179.index.deleted
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:34 00000000000000030179.log.deleted
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:33 00000000000000030179.snapshot.deleted
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:34 00000000000000030179.timeindex.deleted
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:34 00000000000000030189.index.deleted
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:34 00000000000000030189.log.deleted
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:34 00000000000000030189.snapshot.deleted
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:34 00000000000000030189.timeindex.deleted
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:34 00000000000000030199.index.deleted
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:34 00000000000000030199.log.deleted
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:34 00000000000000030199.snapshot.deleted
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:34 00000000000000030199.timeindex.deleted
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:34 00000000000000030209.index.deleted
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:34 00000000000000030209.log.deleted
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:34 00000000000000030209.snapshot.deleted
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:34 00000000000000030209.timeindex.deleted
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:34 00000000000000030219.index.deleted
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:34 00000000000000030219.log.deleted
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:34 00000000000000030219.snapshot.deleted
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:34 00000000000000030219.timeindex.deleted
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:34 00000000000000030229.index.deleted
-rw-r--r--. 1 kafka kafka 710 Jan 27 01:34 00000000000000030229.log.deleted
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:34 00000000000000030229.snapshot.deleted
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:34 00000000000000030229.timeindex.deleted
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:40 00000000000000030239.index.deleted
-rw-r--r--. 1 kafka kafka 497 Jan 27 01:35 00000000000000030239.log.deleted
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:34 00000000000000030239.snapshot.deleted
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:40 00000000000000030239.timeindex.deleted
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:40 00000000000000030246.log
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:40 00000000000000030246.snapshot
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:40 leader-epoch-checkpoint
-rw-r--r--. 1 kafka kafka  43 Jan 26 03:14 partition.metadata
````
А затем видимо отрабатывает log.retention.check.interval.ms оно чекает пометки и удаляет сегменты окончательно:
````
[kafka@kafka-01 users.registrations-0]$ ls -lh
total 12K
-rw-r--r--. 1 kafka kafka   0 Jan 27 01:40 00000000000000030246.log
-rw-r--r--. 1 kafka kafka  56 Jan 27 01:40 00000000000000030246.snapshot
-rw-r--r--. 1 kafka kafka 10M Jan 27 01:40 00000000000000030246.timeindex
-rw-r--r--. 1 kafka kafka  12 Jan 27 01:40 leader-epoch-checkpoint
-rw-r--r--. 1 kafka kafka  43 Jan 26 03:14 partition.metadata
````
Открываем консольный консьюмер, чтобы проверить, что данные удаляются согласно заданным настройкам. Мы перезапустили форлуп, поэтому при неполадках видели бы сообщения test1, test2 и т. д. Если все происходит правильно, видим, что сообщения уже идут какое-то время (в нашем случае — test101, test102 и далее). Более ранних сообщений в этом топике нет, поскольку все роллапится согласно заданным правилам. Перезапускаем консьюмер еще раз, чтобы убедиться наверняка. Видим сообщения test121, test122 и т. д.

Я же останавливал продюсер, поэтому старые сегменты с сообщениями уже были удалены и роллапнулось на пустой, консьюмером нет сообщений
````
./kafka-console-consumer.sh  --bootstrap-server localhost:9092 --topic users.registrations --from-beginning --group slurp
..
Processed a total of 0 messages
````
А если всё запустить и подсоединиться консьюмером спустя время то действительно начинает не с нуля, тк сегменты ролапнулись и удалились
````
./kafka-console-consumer.sh  --bootstrap-server localhost:9092 --topic users.registrations --from-beginning --group slurp
..
31
32
33
34
35
36
````
Странно, но как будто log.retention.check.interval.ms через заданное время удаляет только один самый старый сегмент, потому что из директории меченные .deleted исчезают по одному в 10 сек примерно

## Log Compaction
Помимо функционала удаления данных по retention.ms и retention.bytes, которые мы рассмотрели выше, Кафка предоставляет еще один механизм удаления данных — log compaction или сжатие данных в партиции. Этот механизм использует ключи сообщений, чтобы решить: удалять данные или нет.
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/log_compaction_1.png)
В этом примере мы видим, что в партицию были последовательно записаны три сообщения. Первое было записано с ключом slurm, два последующих — с ключом foo. После завершения compaction в партиции остались два сообщения: с ключом slurm и ключом foo и его последним значением.

Помимо этого compaction позволяет выборочно удалять данные из партиции.
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/log_compaction_2.png)
На этой картинке мы видим, что третьим отправили сообщение с Value: NULL и ключом foo. После завершения compaction в данном случае в партиции осталось только сообщение с ключом slurm. Оставшиеся два сообщения были полностью удалены из нашей партиции. Это произошло потому, что сообщение с Value: NULL (т. н. delete marker) тоже распознается Кафкой, как необходимое к удалению.

Из документации не всегда бывает очевидно, что и ретеншен по времени/размеру, и compaction могут быть включены для топика одновременно. Для этого их нужно указать через запятую в настройке cleanup.policy: delete для ретеншена (включен по умолчанию) и compact для compaction.
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/log_compaction_3.png)
Голова (log head) в compacted топике абсолютно идентична обычной партиции. В ней хранятся все сообщения, даже с одинаковым ключом. Log compaction вносит изменения в то, как работает хвост лога партиции (log tail). Сообщения в хвосте не меняют свои оффсеты, вместо этого в хвосте появляются «дыры». Например, оффсеты 36, 37 и 38 будут идентичны. Соответственно, чтение с 36 и 37 будет идентично чтению с 38, поскольку он единственный, который остался.

Delete markers, сообщения с нулевым пэйлоадом, будут удалены Кафкой спустя некоторое время, чтобы освободить место на диске. На картинке это отмечено записью Delete Retention Point: после этого времени все delete markers будут удалены.

Сам compaction выполняется Кафкой в бэкграунд треде, который сжимает и перезаписывает закрытые сегменты. Активный сегмент никогда не подвергается сжатию, пока не станет закрытым. При этом log compaction как процесс не блокирует чтение данных.
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/log_compaction_4.png)
Это еще один пример compaction. В первоначальной версии лога есть несколько версий сообщений с ключом K1, а также несколько версий с ключом K2. После compaction останется сжатый вариант этой партиции с последними записанными значениями по соответствующим ключам.
#### Характеристики Log Compaction

- Это очень трудоемкий процесс для брокера, при котором возрастает нагрузка на диск (перезапись сегментов), память (загрузка данных из сегмента в java-процесс), процессор (проведение обработки).
- Он не атомарен. В определенные моменты времени могут одновременно присутствовать несколько сообщений, записанных с одним и тем же ключом. Вам придется делать обработку такой ситуации в консьюмере.
- Оффсеты не меняются, порядок записей остается прежним.
- Позволяет «удалять» записи по ключу, что хорошо подходит для снэпшоттинга и восстановления последнего состояния системы после падения/перезагрузки.
- Механизм крайне мощный и полезный, но его понимание и работа с ним в продакшене не самые простые.

Приведем пример compaction из рабочей практики. Механизм применяется для соблюдения закона GDPR в Европе для того, чтобы удалять данные о пользователях из Кафки. Кафка не является БД, нельзя просто так взять и удалить оффсет. Можно включить retention, но при этом будут удаляться целые куски данных. Log compaction же позволяет выборочно удалять сообщения.

## Коротко о ZooKepeer
Кафка использует эту систему как хранилище метаданных (например, конфигурации топиков), механизм leader election и для других операций, где требуется высокая консистентность данных.

Чтобы открыть ZooKeeper, воспользуемся скриптом ./bin/zookeeper-shell.sh и передадим ему адрес ZooKeeper-а, к которому хотим подключиться. В нашем случае это localhost:2181.
````
./bin/zookeeper-shell.sh localhost:2181

ls /zookeeper
..
[config, quota]

ls /
..
[admin, brokers, cluster, config, consumers, controller, controller_epoch, feature, isr_change_notification, latest_producer_id_block, log_dir_event_notification, zookeeper]
````
Данные хранятся как ключи значения, организованные в структуру папок и файлов. Здесь есть путь до ключа, допустим /a, /b, /c. Эта нода и является вашим ключом. Чтобы получить value, нужно сделать операцию get.

Разумеется, ничто не мешает вам хранить все ключи на самом высоком уровне под рутом; но описанным выше образом легче организовывать данные в иерархическую структуру. Это то, что делает Кафка.

Итак, смотрим. В рутовом ключе есть целый набор подключей. Мы можем заглянуть чуть глубже: сделаем ls /brokers, увидим там еще подключи. Чтобы получить значение, которое хранится в ZooKeeper-е по ноде для контроллера, можно воспользоваться командой 
````
get /controller
````
В нашем случае контроллером выступает “brokerid”:0 — тот единственный брокер, который сейчас запущен. 

Мы можем посмотреть состояние нашей партиции: 
````
get /brokers/topics/users.registrations/partitions/0/state
..
{"controller_epoch":1,"leader":0,"version":1,"leader_epoch":0,"isr":[0]}
````
Получаем еще один json, в котором хранится текущее состояние партиции. Лидером у нашей партиции, например, является брокер 0, потому что он у нас один.

Посмотреть метаданные о ноде можно через такую команду:
````
stat /brokers/ids/0
..
cZxid = 0xdc
ctime = Sat Jan 27 01:16:29 EST 2024
mZxid = 0xdc
mtime = Sat Jan 27 01:16:29 EST 2024
pZxid = 0xdc
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x10000005a320000
dataLength = 200
numChildren = 0
````
Мы смотрим именно этот ключ не случайно: нода эфемерна. Так называются ноды ZooKeeper-а, которые хранятся в нем до тех пор, пока между клиентом и сервером есть устойчивое соединение и обмен heartbeat-ами. Что и делает Кафка: она подключается к ZooKeeper и начинает посылать heartbeat-ы, чтобы убедиться в устойчивости соединения. До тех пор, пока такое подключение будет работать, эфемерная нода будет доступна для чтения другими приложениями.

По большому счету, именно таким образом контроллер Кафки узнает, какие брокеры в данный момент запущены в кластере. Если мы остановим брокер, то эфемерная нода исчезнет.

Посмотреть, какие брокеры сейчас подключены, можно через команду
````
ls /brokers/ids
````
В основном, данными из ZooKeeper пользуется контроллер ноды в кластере Кафки. Именно она манипулирует здесь данными, смотрит на список активных брокеров, выбирает новых лидеров партиции. Затем через API самой Кафки и request-response между брокерами она распределяет полученную информацию и отсылает ее своим «подчиненным» в кластере.

## Understanding Kafka end-to-end latency
End-to-end latency is the time between when the application logic produces a record via KafkaProducer.send() to when the record can be consumed by the application logic via KafkaConsumer.poll(). 
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/kafka_end_to_end_latency.png)

    Produce time: processing the record and batching with other records in the internal Kafka producer
    Publish time: sending the record from Kafka producer to broker and appending the record to the leader replica log
    Commit time: replicating the record to follower replicas for fault tolerance
    Catch-up time: catching up to the record’s offset in the log
    Fetch time: fetching the record from the broker
    
#### Produce time: Record processing and batching delays in Kafka producer
Produce time refers to the time between when the application logic produces a record via KafkaProducer.send() to when the produce request containing the record is sent to the leader broker of the topic partition. A Kafka producer batches records for the same topic partition to optimize network and IO requests issued to Kafka. By default, the producer is configured to send the batch immediately, in which case the batch usually contains one or a few records produced by the application at about the same time. To improve batching, the producer can be configured with a small amount of artificial delay (linger.ms) that the record will spend waiting for more records to arrive and get added to the same batch. Once the linger.ms delay has passed or the batch size reaches the maximum configured size (batch.size), the batch is considered complete.

If compression is enabled (compression.type), the Kafka producer compresses the completed batch. Before batch completion, the size of the batch is estimated based on a combination of the compression type and the previously observed compression ratio for the topic.

The batch may need to wait longer in the producer if the number of unacknowledged produce requests sent to the leader broker has already reached the maximum (max.inflight.requests.per.connection), which is five by default. Thus, the faster the broker acknowledges produce requests, the smaller the chance of an additional wait time in the producer.

#### Publish time: Sending and appending the record to the leader replica
Publish time is the time between when the internal Kafka producer sends a produce request to the broker, to when the corresponding message gets appended to the leader replica log. When the request arrives to the broker, the network thread responsible for the connection picks up the request and places it in the request queue. One of the request handler threads picks up requests from the queue and processes them. Thus, publish time includes network time of the produce request, queueing time at the broker, and the time it takes to append the message to the log (most often, this is page cache access time). When the brokers are lightly loaded, network and log append time dominate publish time. As brokers become more loaded, queueing delays increasingly dominate publish time.

#### Commit time: Replicating the record from leader to followers
Commit time is the time it takes to replicate the message to all in-sync replicas. Kafka only exposes messages to the consumer after they have been committed, that is, replicated to all in-sync replicas for fault tolerance. The followers replicate messages from the leader in parallel, and we normally do not expect replicas to be out of sync in a healthy cluster. This means that the time it takes to commit the record equals the time it takes for the slowest in-sync follower broker to fetch the record from the leader broker and append it to the follower replica log.

To replicate data, follower brokers send the leader fetch requests, the exact same type of requests that consumers use to fetch messages. The broker default configuration is optimized for latency of replica fetch requests: the leader sends the response as soon as a single byte is available for fetching (controlled by replica.fetch.min.bytes) or when replica.fetch.wait.max.ms delay is up. Commit times are mainly impacted by the replication factor configuration and cluster load.

#### Catch-up time: Catching up to the record’s offset in the log
Messages in Kafka are consumed in the order they are produced, unless there is an explicit seek to a new offset or a new consumer reading from the latest offset. A consumer will only read the record in question after it reads all previously published records to the same topic partition. Suppose that at the time the message was committed, the consumer’s offset was N messages behind the committed message. Catch-up time, in this case, is the time it takes to consume N messages.
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/Kafka_latency_catch-up.png?raw=true)
When you build real-time use cases, it is better to aim for zero catch-up time where consumers read messages as soon as they are committed. If consumers consistently fall behind, end-to-end latency may become unbounded. Thus, catch-up time mostly depends on the ability of consumers to keep up with the produce throughput.

#### Fetch time: Fetching the record from the broker
The consumer subscribed to a topic partition continuously polls the leader broker for more data. Fetch time is the time it takes to fetch the record in question from the leader broker, potentially waiting for enough data to form the response to the fetch request and return the record in the response from KafkaConsumer.poll(). The default consumer configuration is optimized for latency (fetch.min.bytes is set to one), where the response to a fetch request is returned as soon as a single byte of data is available, or after a timeout (fetch.max.wait.ms).

#### Producer latency depends on ACK policy
 The record is acknowledged based on acks configuration, which controls the durability of records:

 - Immediately without waiting for a reply from a broker (acks = 0)
 - After appending the message to the leader (acks = 1)
 - After all in-sync replicas receive the message (acks = all)

![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/kafka_ack-latency.png?raw=true)

### Impact of durability configuration on latency
Some level of durability is often required because of the criticality of your data. Optimizing for durability increases end-to-end latency because it adds replication overhead (commit time) to latency and adds replication load to the brokers, which increases queueing delays.

#### Replication factor
Replication factor is at the core of Kafka’s durability guarantees, which defines the number of copies of the topic in a Kafka cluster. A replication factor of rf allows you to lose (rf – 1) brokers without losing your data. A replication factor of one achieves minimal end-to-end latency while providing the weakest durability guarantees.

Increasing replication factor adds replication overhead and increases the load on brokers. If client bandwidth is distributed evenly among Kafka brokers, each broker will get rf * w write bandwidth and r + (rf – 1)*w read bandwidth, where rf is the replication factor, w is the client produce bandwidth per broker, and r is the client consume bandwidth per broker. Therefore, the best way to minimize the impact of replication factor on end-to-end latency is to ensure evenly loaded brokers. This will reduce commit time, which is defined by the slowest follower replica, and reduce tail latencies due to decreased variance among broker latencies.

If your Kafka brokers are highly utilized either on disk bandwidth or CPU, followers may start lagging behind the leaders, increasing commit times. We suggest configuring a separate listener for replication traffic on brokers (KIP-103) to reduce interference with client traffic. You can also increase the degree of I/O parallelism on the follower broker, and increase the number of replica fetcher threads per source broker (number.replica.fetchers), which is one by default.

#### Acks
Slower acknowledgements from brokers usually decrease the throughput that can be achieved by a single producer, which in turn increases the wait time in the producer. This is due to the producer limit (max.inflight.requests.per.connection) on the number of unacknowledged requests sent to the broker. For example, nine producers (ran with nine consumers) that were able to produce 195 MB/second (throughput of the produce logic) to Kafka with acks=1 in our setup, produced 161 MB/second to Kafka after changing acks to all. Increasing acks often requires additional scaling of producers to ensure they can keep up with your application logic in order to both achieve your required total throughput and minimize wait time in the internal Kafka producer.

#### Min.insync.replicas
Min.insync.replicas is an important durability configuration, because it defines the number of replicas that have to be in sync for the broker to accept writes for the partition. This configuration impacts availability, but it does not impact end-to-end latency. The message has to be replicated to all replicas that are in sync with the leader regardless of the min.insync.replicas configuration. Thus, choosing a smaller min.insync.replicas does not reduce commit time or latency as a result.
