# Managing RabbitMQ
## Terms
Understand the main components of RabbitMQ, including exchanges, queues, and bindings.
### AMQP
In the Advanced Message Queuing Protocol (AMQP), messages are structured data units that are exchanged between message producers (publishers) and message consumers (subscribers) through a message broker. The structure of an AMQP message includes both header and body sections, each containing various properties and payload data. <br>
#### AMQP Message Structure
    Message Header:
        The message header contains metadata and properties related to the message.
        Key properties in the header include:
            delivery_mode: Indicates whether the message is persistent (1) or transient (0).
            content_type: Describes the MIME type of the message payload.
            priority: Specifies the priority of the message.
            timestamp: The timestamp of when the message was sent.
            expiration: Specifies the message's expiration time.
            correlation_id: Used for correlating responses to requests.
            reply_to: Specifies the name of the reply-to queue.

    Message Properties:
        The message properties section provides additional application-specific properties.
        Properties include user-defined headers and other custom properties.
        Properties can be used to convey application-specific metadata about the message.

    Message Body:
        The message body contains the actual payload data of the message.
        The payload can be of any data type or format (e.g., JSON, XML, binary data).
        The interpretation of the payload is determined by the content_type property in the header.
#### Example of AMQP Message Structure (JSON Representation):
```json
{
  "header": {
    "delivery_mode": 1,
    "content_type": "application/json",
    "priority": 0,
    "timestamp": 1642126800000,
    "expiration": "3600000",
    "correlation_id": "abc123",
    "reply_to": "response_queue"
  },
  "properties": {
    "user_id": "user123",
    "custom_header": "value"
  },
  "body": {
    "message": "Hello, AMQP!"
  }
}
```
#### AMQP Message Flow
    Producer (Publisher):
        The producer creates an AMQP message and publishes it to an exchange.

    Exchange:
        The exchange receives the message and routes it to one or more queues based on routing rules and bindings.

    Queue:
        The message is stored in a queue until a consumer is ready to process it.

    Consumer (Subscriber):
        The consumer retrieves the message from the queue, processes it, and acknowledges its receipt.

### Message Acknowledgment
In RabbitMQ, acknowledgment (ack) is a mechanism that allows consumers to inform the broker that a message has been successfully processed and can be safely removed from the queue. There are two main acknowledgment modes: manual acknowledgment (manual ack) and automatic acknowledgment (auto ack).
#### Manual Acknowledgment (Manual Ack)
In manual acknowledgment mode, also known as explicit acknowledgment, the consumer is responsible for sending an acknowledgment signal to the broker after successfully processing a message. This gives the consumer more control over the acknowledgment process. <br>
- Messages are not removed from the queue until they are explicitly acknowledged.
- This can help avoid message loss in case of consumer failures or errors during message processing.
- Developers need to ensure that acknowledgments are sent at the appropriate time and handle potential errors.
- If a consumer fails to send an acknowledgment, the broker may redeliver the message to another consumer.
#### Automatic Acknowledgment (Auto Ack)
In automatic acknowledgment mode, also known as implicit acknowledgment or auto ack, the acknowledgment process is handled automatically by the broker as soon as the message is delivered to the consumer. The consumer does not need to send explicit acknowledgment signals.
- Auto acknowledgment simplifies the consumer code as it does not need to explicitly handle acknowledgments.
- The acknowledgment process is handled automatically in the background.
- Eliminates the need for developers to manage acknowledgment logic, reducing potential errors.
- If a consumer crashes or encounters an error during message processing, the message may be lost without acknowledgment.
- May lead to scenarios where a message is not processed successfully, but the broker assumes it has been.
### Virtual Hosts
In RabbitMQ, a virtual host is a way to partition and isolate resources such as exchanges, queues, and permissions within a RabbitMQ broker. Each virtual host operates independently of others, providing a logical separation of messaging entities and their associated configuration. <br>
Here are some key purposes and benefits of using RabbitMQ virtual hosts:
- Logical Isolation
- Security and Permissions
- Resource Partitioning
- Policy Isolation
- Message Routing Isolation
- Ease of Management and Monitoring <br>

In RabbitMQ, you can limit resources for a specific virtual host by setting various policies. Resource limitations can include constraints on message TTL (Time To Live), maximum length of queues, maximum number of connections, and more. These limitations help in preventing resource exhaustion and managing the behavior of messaging entities within a virtual host.

An Example of Policy on all queues (.*) in virtual host my_host:
````
rabbitmqctl add_vhost my_vhost
rabbitmqctl set_policy -p my_vhost queue_length_limit '.*' '{"max-length": 100, "overflow": "reject-publish"}'
````
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/Screenshot%20from%202024-01-21%2014-10-52.png?raw=true)
### Messaging Patterns
````
    Publish/Subscribe:
        Description: In the publish/subscribe pattern, a message producer (publisher) sends messages to an exchange, and multiple message consumers (subscribers) receive messages from queues bound to that exchange. Each subscriber receives a copy of every message.
        RabbitMQ Entities: Exchange, Queues, Bindings

    Point-to-Point (Queues):
        Description: In the point-to-point pattern, messages are sent to a queue, and each message is consumed by a single consumer. Multiple consumers can be connected to the same queue, but each message is processed by only one of them.
        RabbitMQ Entities: Queue

    Direct Routing:
        Description: In this pattern, messages are sent to an exchange with a specific routing key. Queues are bound to the exchange with routing key criteria, and messages are routed to the queues based on these criteria.
        RabbitMQ Entities: Exchange, Queues, Bindings

    Topic Routing:
        Description: Similar to direct routing, but with more flexible routing patterns. Messages are sent to an exchange with a topic, and queues are bound to the exchange with topic patterns. Messages are routed to queues based on matching patterns.
        RabbitMQ Entities: Exchange, Queues, Bindings

    Headers Exchange:
        Description: Messages are sent to an exchange with a set of headers. Queues are bound to the exchange with header criteria, and messages are routed to queues based on matching header values.
        RabbitMQ Entities: Exchange, Queues, Bindings

    Request/Reply (RPC):
        Description: In the request/reply pattern, a client (requester) sends a message to a server (responder), and the server replies with a response. The client waits for the response, creating a synchronous communication pattern.
        RabbitMQ Entities: Queues, Correlation ID

    Dead Letter Exchange:
        Description: Messages that cannot be delivered to their intended queues are sent to a dead letter exchange. Dead letter exchanges are often used to handle messages that fail to be processed after multiple attempts.
        RabbitMQ Entities: Exchange, Queues, Bindings

    Fanout Exchange:
        Description: A fanout exchange broadcasts messages to all queues bound to it, ignoring routing keys. This pattern is useful for implementing broadcast-style communication where all subscribers receive the same message.
        RabbitMQ Entities: Exchange, Queues, Bindings
````
### Exchanges
In RabbitMQ, exchanges are routing mechanisms that determine how messages are routed from message producers (publishers) to message consumers (subscribers). Exchanges receive messages from producers and then route them to one or more queues based on specific rules, known as routing keys and binding rules.

    Routing Logic:
        Exchanges define the routing logic for messages within RabbitMQ.
        Messages sent to an exchange can be routed to one or more queues based on routing keys and binding rules.

    Message Routing:
        Exchanges determine how messages are distributed to queues based on routing keys and binding rules.
        Different types of exchanges use different routing algorithms.

    No Storage of Messages:
        Exchanges do not store messages. Their primary role is to route messages to queues.

Example of Topic Exchange:

Let's consider an example where we have a topic exchange named "logs" and several queues with different routing patterns:

    Queue A is bound to "logs" with the routing pattern *.error.*.
    Queue B is bound to "logs" with the routing pattern info.*.*.
    Queue C is bound to "logs" with the routing pattern #.critical.

In this setup, messages with routing keys like info.message.warning, error.message.debug, or warning.critical would be routed to the appropriate queues based on the defined patterns.

![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/Screenshot%20from%202024-01-21%2014-49-19.png?raw=true)

### Bindings
In RabbitMQ, a binding is a link between an exchange and a queue. It defines the rules for routing messages from the exchange to the queue. Bindings play a crucial role in determining how messages are delivered from producers to queues within the RabbitMQ message broker.

    Link Between Exchange and Queue:
        A binding establishes a connection between an exchange and a queue.
        Messages sent to an exchange are routed to queues based on the bindings that exist between the exchange and queues.

    Routing Key:
        When creating a binding, you specify a routing key or pattern.
        For some exchange types, the routing key is used to determine how messages should be routed to queues.

    Exchange Types:
        Different exchange types in RabbitMQ (direct, topic, fanout, headers) use bindings differently.
        For example, in a direct exchange, the routing key in the binding is compared to the routing key of the message for routing.

    Exchange-Queue Relationship:
        An exchange can be bound to multiple queues, and a queue can be bound to multiple exchanges.
        The same queue can have multiple bindings with different routing keys, allowing it to receive messages from different exchanges based on different criteria.

    Binding Parameters:
        In addition to the routing key, other parameters can be specified when creating a binding, depending on the exchange type.
        For example, headers can be specified in a headers exchange binding.

Example of creating binding:
````
import pika

# Connect to RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Declare a direct exchange named "direct_logs"
channel.exchange_declare(exchange='direct_logs', exchange_type='direct')

# Declare a queue named "my_queue"
channel.queue_declare(queue='my_queue')

# Bind the queue to the exchange with a specific routing key
channel.queue_bind(exchange='direct_logs', queue='my_queue', routing_key='info')

# Close the connection
connection.close()
````
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/Screenshot%20from%202024-01-21%2015-31-15.png?raw=true)

### Queue Types
````
    Classic Queue:
        Description: Classic queues are the basic and most commonly used type of queue in RabbitMQ. They follow the standard queuing model where messages are delivered to consumers in the order they arrive in the         queue.
        Use Cases: Suitable for most messaging scenarios where messages should be processed in a first-in, first-out (FIFO) order.

    Durable Queue:
        Description: Durable queues persist their state to disk, ensuring that the queue and its messages survive broker restarts. Non-durable queues exist only in memory and may lose their state during a broker          restart.
        Use Cases: Recommended for scenarios where message durability and queue persistence are essential to prevent data loss.

    Transient Queue:
        Description: Transient queues (non-durable queues) exist only in memory and do not survive broker restarts. They are faster than durable queues but may lose their state in case of a broker restart.
        Use Cases: Suitable for scenarios where message durability is not critical, and fast, non-persistent messaging is preferred.

    Auto-Delete Queue:
        Description: Auto-delete queues are automatically deleted by RabbitMQ when the last consumer unsubscribes from the queue. This can be useful in scenarios where temporary queues are needed and can be               cleaned up automatically.
        Use Cases: Useful for temporary or short-lived queues that are no longer needed once consumers disconnect.

    Lazy Queue:
        Description: Lazy queues are designed to store large numbers of messages on disk efficiently. They are suitable for scenarios with a high message volume and where messages can be processed at a slower             pace.
        Use Cases: Ideal for scenarios with large message storage requirements, such as log aggregation or message buffering.

    Priority Queue:
        Description: Priority queues allow messages to be prioritized based on their priority level. Messages with higher priority levels are delivered before those with lower priority levels.
        Use Cases: Useful in scenarios where message prioritization is critical, and certain messages need to be processed with higher priority.

    Quorum Queue:
        Description: Quorum queues provide improved durability compared to classic queues. They use a quorum-based replication approach to ensure message persistence and availability even in the case of broker            failures.
        Use Cases: Recommended for scenarios where high durability and availability are crucial, such as critical business applications.
````
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/Screenshot%20from%202024-01-21%2015-43-09.png?raw=true)
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
rabbitmqctl -n rabbit@rabbitmq-02 -p / list_queues name messages_ready
..
Timeout: 60.0 seconds ...
Listing queues for vhost / ...
name	messages_ready
hello	0
QueueA	3
QueueB	3
QueueC	3

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
### Direct Exchange
direct_producer.py:
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
direct_consumer.py:
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
### Topic Exchange
topic_producer.py:
````
import pika

# Connect to RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='192.168.122.253',
    port=5672,
    virtual_host='/',
    credentials=pika.PlainCredentials('frodo', 'test'),))
channel = connection.channel()

# Declare a topic exchange named "logs"
channel.exchange_declare(exchange='logs', exchange_type='topic')

# Declare and bind queues with specific routing patterns
queues_and_patterns = [
    ('QueueA', 'error.message.debug'),
    ('QueueB', 'info.*.*'),
    ('QueueC', '#.critical')
]

for queue, routing_key_pattern in queues_and_patterns:
    channel.queue_declare(queue=queue)
    channel.queue_bind(exchange='logs', queue=queue, routing_key=routing_key_pattern)

# Publish some example messages with different routing keys
example_messages = [
    ('info.message.warning', 'This is an info message with a warning'),
    ('error.message.debug', 'This is an error message with debug information'),
    ('warning.critical', 'This is a warning message with critical importance')
]

for routing_key, message_body in example_messages:
    channel.basic_publish(exchange='logs', routing_key=routing_key, body=message_body)
    print(f" [x] Sent '{routing_key}':'{message_body}'")

# Close the connection
connection.close()
````
topic_consumer.py:
````
import pika

def callback(ch, method, properties, body):
    print(f" [x] Received {method.routing_key}: {body}")

# Connect to RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='192.168.122.254',
    port=5672,
    virtual_host='/',
    credentials=pika.PlainCredentials('frodo', 'test'),))
channel = connection.channel()

# Declare and bind queues with specific routing patterns (similar to the producer)
queues_and_patterns = [
    ('QueueA', 'error.message.debug'),
    ('QueueB', 'info.*.*'),
    ('QueueC', '#.critical')
]

for queue, routing_key_pattern in queues_and_patterns:
    channel.queue_declare(queue=queue)
    channel.queue_bind(exchange='logs', queue=queue, routing_key=routing_key_pattern)

# Set up the consumer callback
channel.basic_consume(queue='QueueA', on_message_callback=callback, auto_ack=True)
channel.basic_consume(queue='QueueB', on_message_callback=callback, auto_ack=True)
channel.basic_consume(queue='QueueC', on_message_callback=callback, auto_ack=True)

# Start consuming messages
print(' [*] Waiting for messages. To exit, press CTRL+C')
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
