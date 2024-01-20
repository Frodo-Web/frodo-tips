# Managing RabbitMQ
## Install the latest version on CentOS 7
````
1. Install Erlang OTP
sudo rpm --import https://packages.erlang-solutions.com/rpm/erlang_solutions.asc
// Browse for the latest package on https://www.erlang-solutions.com/downloads/ , this will install all the stuff
sudo yum install -y https://binaries2.erlang-solutions.com/centos/7/esl-erlang_26.2.1_1~centos~7_x86_64.rpm

2. Install RabbitMQ
// Browse for the latest package on https://github.com/rabbitmq/rabbitmq-server/releases/
curl -L -O - https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.12.12/rabbitmq-server-3.12.12-1.el8.noarch.rpm
sudo rpm --import https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
sudo rpm -Uvh rabbitmq-server-3.12.12-1.el8.noarch.rpm

3. Run RabbitMQ
sudo systemctl start rabbitmq-server
````
## rabbitmqctl commands
````
Check RabbitMQ Node Status:
sudo rabbitmqctl status

Stop RabbitMQ Server:
sudo rabbitmqctl stop

Start RabbitMQ Server:
sudo rabbitmq-server

Add a User:
sudo rabbitmqctl add_user <username> <password>

Set User Permissions:
sudo rabbitmqctl set_permissions -p / <username> ".*" ".*" ".*"
This command sets permissions for a user. In the example above, the user has full permissions (".*").

Delete a User:
sudo rabbitmqctl delete_user <username>

List Queues:
sudo rabbitmqctl list_queues

Purge a Queue:
sudo rabbitmqctl purge_queue <queue_name>
This command removes all messages from a specified queue.

List Virtual Hosts:
sudo rabbitmqctl list_vhosts

Add Virtual Host:
sudo rabbitmqctl add_vhost <vhost_name>

Delete Virtual Host:
sudo rabbitmqctl delete_vhost <vhost_name>

Join a Cluster:
sudo rabbitmqctl join_cluster <cluster_name>
This command joins the current RabbitMQ node to a cluster.

List Cluster Nodes:
sudo rabbitmqctl cluster_status

List Exchanges:
sudo rabbitmqctl list_exchanges

List Connections:
sudo rabbitmqctl list_connections

List Channels:
sudo rabbitmqctl list_channels

Help:
rabbitmqctl help
rabbitmqctl help <command>
````
