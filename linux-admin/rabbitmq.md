# Managing RabbitMQ
## Install the latest version on CentOS 7
````
1. Install Erlang OTP
sudo rpm --import https://packages.erlang-solutions.com/rpm/erlang_solutions.asc
// Browse for the latest package on https://www.erlang-solutions.com/downloads/ , this will install all the stuff
sudo yum install -y https://binaries2.erlang-solutions.com/centos/7/esl-erlang_26.2.1_1~centos~7_x86_64.rpm

2. Install RabbitMQ
// Browse for the latest package on https://github.com/rabbitmq/rabbitmq-server/releases/
curl -L -O https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.12.12/rabbitmq-server-3.12.12-1.el8.noarch.rpm
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

Set User tags:
sudo rabbitmqctl set_user_tags frodo administrator management

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
## RabbitMQ Web Management Console
Enable
````
sudo rabbitmq-plugins enable rabbitmq_management
````
Setting up access:
````
sudo chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/
sudo rabbitmqctl add_user admin password
sudo rabbitmqctl set_user_tags admin management
sudo rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"

sudo rabbitmqctl list_users
..
user    tags
admin   [management]
````
To access the RabbitMQ admin:
````
http://Your_Server_IP:15672
````
Using SSL is recommended. <br>
Create or update rabbitmq conf file at /etc/rabbitmq/rabbitmq.conf:
````
management.listener.port = 15672
management.listener.ssl  = true

management.listener.ssl_opts.cacertfile = /path/to/cacertfile.pem
management.listener.ssl_opts.certfile   = /path/to/certfile.pem
management.listener.ssl_opts.keyfile    = /path/to/keyfile.pem
````
## RabbitMQ Cluster
For example, we have 2 nodes, rabbitmq-01 and rabbitmq-02. <br>
We want to add rabbitmq-02 to rabbitmq-01 cluster <br>
These are the steps, on rabbitmq-01:
````
vim /etc/hosts
192.168.122.253  rabbitmq-01
192.168.122.254  rabbitmq-02

cat /var/lib/rabbitmq/.erlang.cookie
..
MAFSGVPPIDKTIRJHBQHY
````
On rabbitmq-02:
````
vim /etc/hosts
192.168.122.253  rabbitmq-01
192.168.122.254  rabbitmq-02

systemctl stop rabbitmq-server
echo -n "MAFSGVPPIDKTIRJHBQHY" > /var/lib/rabbitmq/.erlang.cookie
systemctl start rabbitmq-server

rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@rabbitmq-01
rabbitmqctl start_app

rabbitmqctl cluster_status
````
#### Pass erlang cookie
````
export RABBITMQ_ERLANG_COOKIE='XMSBWXUFRMWNOYHSDPER'
````
````
rabbitmqctl status --erlang-cookie MAFSGVPPIDKTIRJHBQHY
rabbitmq-diagnostics erlang_cookie_sources --erlang-cookie MAFSGVPPIDKTIRJHBQHY
````
#### Pass nodename
````
export RABBITMQ_NODENAME=rabbit@rabbitmq-02
````

## Example of using RabbitMQ with python
producer.py:
````
// This connects to rabbitmq-01 of our RabbitMQ cluster
import pika

# Connect to RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='192.168.122.253',
    port=5672,
    virtual_host='/',
    credentials=pika.PlainCredentials('frodo', 'test'),))
channel = connection.channel()

# Declare a queue named 'hello'
channel.queue_declare(queue='hello')

# Publish a message to the 'hello' queue
channel.basic_publish(exchange='', routing_key='hello', body='Hello, RabbitMQ!')

print(" [x] Sent 'Hello, RabbitMQ!'")

# Close the connection
connection.close()
````
consumer.py:
````
// This connects and reads messages from rabbitmq-02 of our RabbitMQ cluster
import pika

# Callback function to handle incoming messages
def callback(ch, method, properties, body):
    print(f" [x] Received {body}")

# Connect to RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='192.168.122.254',
    port=5672,
    virtual_host='/',
    credentials=pika.PlainCredentials('frodo', 'test'),))
channel = connection.channel()

# Declare a queue named 'hello'
channel.queue_declare(queue='hello')

# Set up the callback function to handle incoming messages
channel.basic_consume(queue='hello', on_message_callback=callback, auto_ack=True)

print(' [*] Waiting for messages. To exit, press CTRL+C')
# Start consuming messages
channel.start_consuming()
````
## RabbitMQ learning roadmap
Here's a roadmap to guide your learning journey:
1. Understand Messaging Concepts:

    Message Brokers:
        Learn what message brokers are and their role in distributed systems.
        Understand the advantages of using a message broker.

    Messaging Patterns:
        Explore common messaging patterns, such as publish/subscribe, point-to-point, and request/reply.

    Queues and Exchanges:
        Understand the concept of queues and exchanges in RabbitMQ.
        Learn how messages are routed and processed.

2. Install RabbitMQ:

    Install RabbitMQ:
        Install RabbitMQ on your local machine for development.
        Follow the official installation guides for your operating system.

3. RabbitMQ Basics:

    RabbitMQ Components:
        Understand the main components of RabbitMQ, including exchanges, queues, and bindings.

    AMQP Protocol:
        Learn about the Advanced Message Queuing Protocol (AMQP), which RabbitMQ supports.
        Understand how messages are structured in AMQP.

4. RabbitMQ Management:

    RabbitMQ Management Console:
        Explore the RabbitMQ Management Console for monitoring and managing your RabbitMQ server.
        Learn how to access the console and navigate its features.

5. Basic Operations:

    Publishing and Consuming:
        Write simple programs to publish and consume messages.
        Understand the basic API for interacting with RabbitMQ.

    Message Acknowledgment:
        Learn about message acknowledgment and how to handle message acknowledgment in your code.

    Durable Queues and Exchanges:
        Understand the concept of durability and how to make queues and exchanges durable.

6. Advanced Operations:

    Bindings and Routing:
        Explore advanced topics like bindings and routing in RabbitMQ.
        Understand how messages are routed based on routing keys and patterns.

    Exchanges Types:
        Learn about different types of exchanges, such as direct, topic, fanout, and headers exchanges.

    Dead Letter Exchanges:
        Understand the concept of dead letter exchanges for handling failed messages.

7. RabbitMQ in Real-world Scenarios:

    Message Reliability:
        Explore techniques for ensuring message reliability, including message acknowledgment, publisher confirms, and transactions.

    Clustering:
        Learn how to set up and configure RabbitMQ clusters for high availability and scalability.

    Security:
        Understand security best practices for RabbitMQ, including user management, permissions, and encryption.

8. Integration with Programming Languages and Frameworks:

    RabbitMQ with Java:
        Explore RabbitMQ integration with Java using the RabbitMQ Java client library.

    RabbitMQ with Other Languages:
        Investigate RabbitMQ client libraries for other programming languages, such as Python, Node.js, and .NET.

9. RabbitMQ Plugins:

    Management Plugin:
        Deepen your understanding of the RabbitMQ Management Plugin.
        Explore additional plugins for specific features or integrations.

10. Troubleshooting and Maintenance:

    Monitoring and Troubleshooting:
        Learn how to monitor RabbitMQ and troubleshoot common issues.
        Understand the importance of log files and how to analyze them.

    Backups and Recovery:
        Explore backup and recovery strategies for RabbitMQ.

11. Real-world Projects:

    Build Real-world Projects:
        Apply your knowledge to real-world projects.
        Consider building a small messaging system using RabbitMQ.
