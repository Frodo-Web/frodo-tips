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

### Unacknowledged messages
In RabbitMQ, unacknowledged messages (often referred to as "unacked messages" or "unacknowledged deliveries") represent messages that have been delivered to a consumer but have not yet been acknowledged by that consumer. Acknowledgment is a mechanism that allows consumers to inform the broker that a message has been successfully processed and can be safely removed from the queue. <br>

When a message is sent to a consumer, it enters an unacknowledged state until the consumer explicitly acknowledges it. The acknowledgment process is a critical part of message reliability and ensures that messages are not lost, even in the presence of consumer failures. <br>

Here is an overview of the life cycle of a message in RabbitMQ related to acknowledgment:

    Message Delivery:
        A message is sent to a consumer for processing.

    Unacknowledged State:
        The message enters an unacknowledged state until the consumer explicitly acknowledges it.
        During this period, the message is considered "in-flight" or "delivered" but not yet confirmed as successfully processed.

    Acknowledgment:
        The consumer acknowledges the message to confirm its successful processing.
        Acknowledgment can be automatic (auto ack) or manual (manual ack) based on the acknowledgment mode used by the consumer.

    Acknowledged State:
        Once acknowledged, the message is considered successfully processed, and RabbitMQ removes it from the queue.
#### Code example
Auto ack:
````python
import pika

def callback(ch, method, properties, body):
    # Simulate message processing
    print(f"Received message: {body}")

# Connect to RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Declare a queue named "my_queue"
channel.queue_declare(queue='my_queue', durable=True)

# Set up the consumer with automatic acknowledgment (auto ack)
channel.basic_consume(queue='my_queue', on_message_callback=callback, auto_ack=True)

print(' [*] Waiting for messages. To exit, press CTRL+C')
channel.start_consuming()
````
Manual ack:
````python
import pika

def callback(ch, method, properties, body):
    # Simulate message processing
    print(f"Received message: {body}")

    # Explicit acknowledgment (manual ack)
    ch.basic_ack(delivery_tag=method.delivery_tag)
    print("Acknowledged message")

# Connect to RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Declare a durable queue named "my_queue" with manual acknowledgment
channel.queue_declare(queue='my_queue', durable=True)

# Set up the consumer with manual acknowledgment
channel.basic_consume(queue='my_queue', on_message_callback=callback)

print(' [*] Waiting for messages. To exit, press CTRL+C')
channel.start_consuming()
````
### Durable vs Transient
In RabbitMQ, the concepts of "durable" and "transient" are related to the durability of queues, exchanges, and messages.
#### Durable
    Durable Queue:
        A durable queue is a queue that survives broker restarts. When a queue is declared as durable, its definition and the state of its messages are persisted to disk, ensuring that the queue and its contents          are not lost in case of a broker restart.
        Example: channel.queue_declare(queue='my_queue', durable=True)

    Durable Exchange:
        A durable exchange is an exchange that survives broker restarts. Similar to durable queues, a durable exchange's definition is persisted to disk, making sure that the exchange is not lost during a broker          restart.
        Example: channel.exchange_declare(exchange='my_exchange', exchange_type='direct', durable=True)

    Durable Message:
        A durable message is a message that survives a broker restart. For a message to be durable, it must be published to a durable exchange and routed to a durable queue. Additionally, the message itself must be marked as persistent.
        Example: channel.basic_publish(exchange='my_exchange', routing_key='my_queue', body='Hello, RabbitMQ!', properties=pika.BasicProperties(delivery_mode=2))
#### Transient (Non-Durable)
    Transient Queue:
        A transient (non-durable) queue exists only in memory and does not survive a broker restart. Transient queues are faster than durable queues but may lose their state (definition and messages) in case of a         broker restart.
        Example: channel.queue_declare(queue='my_queue', durable=False)

    Transient Exchange:
        A transient (non-durable) exchange exists only in memory and does not survive a broker restart. Similar to transient queues, transient exchanges are faster but may lose their state during a broker restart.
        Example: channel.exchange_declare(exchange='my_exchange', exchange_type='direct', durable=False)

    Transient Message:
        A transient (non-durable) message is a message that is not marked as persistent. Such messages are not guaranteed to survive a broker restart and may be lost if the broker goes down.
        Example: channel.basic_publish(exchange='my_exchange', routing_key='my_queue', body='Hello, RabbitMQ!', properties=pika.BasicProperties(delivery_mode=1))

### Queue mirroring
Queue mirroring in RabbitMQ is a feature that provides high availability and fault tolerance for message queues. When a queue is mirrored, its contents are replicated across multiple nodes in a RabbitMQ cluster. This means that even if a RabbitMQ node hosting a mirrored queue fails, the queue and its messages are still accessible from another node in the cluster. <br>

High Availability:

    Queue mirroring enhances the availability of queues in case of node failures.
    If a node goes down, a mirrored queue remains accessible on another node, preventing disruptions in message processing.

Mirrored Queue Structure:

    A mirrored queue consists of the master and its replicas.
    The master is the primary node where the queue is declared and where writes (publishing messages) are performed.
    Replicas are copies of the queue stored on other nodes in the RabbitMQ cluster.

Automatic Failover:

    If the master node fails, one of the replicas is automatically promoted to be the new master.
    The cluster maintains continuous service, and consumers can continue consuming messages without interruption.

Synchronous Replication:

    Mirroring involves synchronous replication of messages to the replicas.
    The master node waits for acknowledgments from the replicas before confirming message receipt to the publisher.

Mirrored Queue Declaration:

    To create a mirrored queue, the x-ha-policy argument is set to "all" when declaring the queue.
Example:
```python
channel.queue_declare(queue='my_mirrored_queue', durable=True, arguments={'x-ha-policy': 'all'})
```
Network Partitions:

    RabbitMQ is designed to handle network partitions and ensure that only one set of nodes is active for a mirrored queue at any given time.

Resource Intensive:

    Queue mirroring can be resource-intensive, especially for high-throughput queues with a large number of messages.
    It is essential to consider the impact on system resources when enabling mirroring for queues.

Mirror All Queues or Selectively:

    Queues can be selectively mirrored based on the requirements of the application.
    For critical queues, you may choose to mirror them, while for others, mirroring may be unnecessary.
Example:
```python
# Declare a mirrored queue named 'my_mirrored_queue'
channel.queue_declare(queue='my_mirrored_queue', durable=True, arguments={'x-ha-policy': 'all'})
```
In this example, the x-ha-policy argument is set to "all", indicating that the queue should be mirrored across all nodes in the RabbitMQ cluster.
### Quorum Queues 
The RabbitMQ quorum queue is a modern queue type, which implements a durable, replicated FIFO queue based on the Raft consensus algorithm.

Quorum queues are designed to be safer and provide simpler, well defined failure handling semantics that users should find easier to reason about when designing and operating their systems.

Quorum queues and streams now replace the original, replicated mirrored classic queue. Mirrored classic queues are now deprecated and scheduled for removal.

Quorum queues are optimized for set of use cases where data safety is a top priority. This is covered in Motivation. Quorum queues should be considered the default option for a replicated queue type.

Quorum queues share most of the fundamentals with other queue types. A client library that can use regular mirrored queues will be able to use quorum queues.

The following operations work the same way for quorum queues as they do for regular queues:

    Consumption (subscription)
    Consumer acknowledgements (except for global QoS and prefetch)
    Cancelling consumers
    Purging
    Deletion

With some queue operations there are minor differences:

    Declaration
    Setting prefetch for consumers
#### When Not to Use Quorum Queues

    Temporary nature of queues: transient or exclusive queues, high queue churn (declaration and deletion rates)
    Lowest possible latency: the underlying consensus algorithm has an inherently higher latency due to its data safety features
    When data safety is not a priority (e.g. applications do not use manual acknowledgements and publisher confirms are not used)
    Very long queue backlogs (streams are likely to be a better fit)

#### Quorum Queue Replication and Data Locality
When a quorum queue is declared, an initial number of replicas for it must be started in the cluster. By default the number of replicas to be started is up to three, one per RabbitMQ node in the cluster.

Three nodes is the practical minimum of replicas for a quorum queue. In RabbitMQ clusters with a larger number of nodes, adding more replicas than a quorum (majority) will not provide any improvements in terms of quorum queue availability but it will consume more cluster resources.

Therefore the recommended number of replicas for a quorum queue is the quorum of cluster nodes (but no fewer than three). This assumes a fully formed cluster of at least three nodes.
#### Controlling the Initial Replication Factor
For example, a cluster of three nodes will have three replicas, one on each node. In a cluster of seven nodes, three nodes will have one replica each but four more nodes won't host any replicas of the newly declared queue.

Like with classic mirrored queues, the replication factor (number of replicas a queue has) can be configured for quorum queues.

The minimum factor value that makes practical sense is three. It is highly recommended for the factor to be an odd number. This way a clear quorum (majority) of nodes can be computed. For example, there is no "majority" of nodes in a two node cluster. This is covered with more examples below in the Fault Tolerance and Minimum Number of Replicas Online section.

This may not be desirable for larger clusters or for cluster with an even number of nodes. To control the number of quorum queue members set the x-quorum-initial-group-size queue argument when declaring the queue. The group size argument provided should be an integer that is greater than zero and smaller or equal to the current RabbitMQ cluster size. The quorum queue will be launched to run on a random subset of RabbitMQ nodes present in the cluster at declaration time.

In case a quorum queue is declared before all cluster nodes have joined the cluster, and the initial replica count is greater than the total number of cluster members, the effective value used will be equal to the total number of cluster nodes. When more nodes join the cluster, the replica count will not be automatically increased but it can be increased by the operator.

#### Queue Leader Location
Every quorum queue has a primary replica. That replica is called queue leader. All queue operations go through the leader first and then are replicated to followers (mirrors). This is necessary to guarantee FIFO ordering of messages.

To avoid some nodes in a cluster hosting the majority of queue leader replicas and thus handling most of the load, queue leaders should be reasonably evenly distributed across cluster nodes.

When a new quorum queue is declared, the set of nodes that will host its replicas is randomly picked, but will always include the node the client that declares the queue is connected to.

Which replica becomes the initial leader can controlled using three options:

    Setting the queue-leader-locator policy key (recommended)
    By defining the queue_leader_locator key in the configuration file (recommended)
    Using the x-queue-leader-locator optional queue argument

Supported queue leader locator values are

    client-local: Pick the node the client that declares the queue is connected to. This is the default value.
    balanced: If there are overall less than 1000 queues (classic queues, quorum queues, and streams), pick the node hosting the minimum number of quorum queue leaders. If there are overall more than 1000 queues, pick a random node.
    
#### Managing Replicas (Quorum Group Members)
Replicas of a quorum queue are explicitly managed by the operator. When a new node is added to the cluster, it will host no quorum queue replicas unless the operator explicitly adds it to a member (replica) list of a quorum queue or a set of quorum queues.

When a node has to be decommissioned (permanently removed from the cluster), it must be explicitly removed from the member list of all quorum queues it currently hosts replicas for.

Several CLI commands are provided to perform the above operations:
````
rabbitmq-queues add_member [-p <vhost>] <queue-name> <node>

rabbitmq-queues delete_member [-p <vhost>] <queue-name> <node>

rabbitmq-queues grow <node> <all | even> [--vhost-pattern <pattern>] [--queue-pattern <pattern>]

rabbitmq-queues shrink <node> [--errors-only]
````
When replacing a cluster node, it is safer to first add a new node and then decomission the node it replaces.

#### Rebalancing Replicas for Quorum Queues
Once declared, the RabbitMQ quorum queue leaders may be unevenly distributed across the RabbitMQ cluster. To re-balance use the rabbitmq-queues rebalance command. It is important to know that this does not change the nodes which the quorum queues span. To modify the membership instead see managing replicas.
````
# rebalances all quorum queues
rabbitmq-queues rebalance quorum
````
it is possible to rebalance a subset of queues selected by name:
````
# rebalances a subset of quorum queues
rabbitmq-queues rebalance quorum --queue-pattern "orders.*"
````
or quorum queues in a particular set of virtual hosts:
````
# rebalances a subset of quorum queues
rabbitmq-queues rebalance quorum --vhost-pattern "production.*"
````
### Streams
RabbitMQ Streams is a persistent replicated data structure that can complete the same tasks as queues: they buffer messages from producers that are read by consumers. However, streams differ from queues in two important ways: how messages are stored and consumed.

Streams model an append-only log of messages that can be repeatedly read until they expire. Streams are always persistent and replicated. A more technical description of this stream behavior is “non-destructive consumer semantics”.

To read messages from a stream in RabbitMQ, one or more consumers subscribe to it and read the same messages as many times as they want.

Data in a stream can be used via a RabbitMQ client library or through a dedicated binary protocol plugin and associated client(s). The latter option is highly recommended as it provides access to all stream-specific features and offers best possible throughput (performance).

Now, you might be asking the following questions:

    Do streams replace queues then?
    Should I move away from using queues?

To answer these questions, streams were not introduced to replace queues but to complement them. Streams open up many opportunities for new RabbitMQ use cases which are described in Use Cases for Using Streams.

#### Use Cases for Using Streams
Streams were developed to initially cover 4 messaging use-cases that existing queue types either can not provide or provide with downsides:

    Large fan-outs

    When wanting to deliver the same message to multiple subscribers users currently have to bind a dedicated queue for each consumer. If the number of consumers is large this becomes potentially inefficient, especially when wanting persistence and/or replication. Streams will allow any number of consumers to consume the same messages from the same queue in a non-destructive manner, negating the need to bind multiple queues. Stream consumers will also be able to read from replicas allowing read load to be spread across the cluster.

    Replay (Time-travelling)

    As all current RabbitMQ queue types have destructive consume behaviour, i.e. messages are deleted from the queue when a consumer is finished with them, it is not possible to re-read messages that have been consumed. Streams will allow consumers to attach at any point in the log and read from there.

    Throughput Performance

    No persistent queue types are able to deliver throughput that can compete with any of the existing log based messaging systems. Streams have been designed with performance as a major goal.

    Large backlogs

    Most RabbitMQ queues are designed to converge towards the empty state and are optimised as such and can perform worse when there are millions of messages on a given queue. Streams are designed to store larger amounts of data in an efficient manner with minimal in-memory overhead.
#### Performance Characteristics
As streams persist all data to disks before doing anything it is recommended to use the fastest disks possible.

Due to the disk I/O-heavy nature of streams, their throughput decreases as message sizes increase.

Just like quorum queues, streams are also affected by cluster sizes. The more replicas a stream has, the lower its throughput generally will be since more work has to be done to replicate data and achieve consensus.

#### Controlling the Initial Replication Factor
The x-initial-cluster-size queue argument controls how many rabbit nodes the initial stream cluster should span.
#### Managing Stream Replicas
Replicas of a stream are explicitly managed by the operator. When a new node is added to the cluster, it will host no stream replicas unless the operator explicitly adds it to a replica set of a stream.

When a node has to be decommissioned (permanently removed from the cluster), it must be explicitly removed from the replica list of all streams it currently hosts replicas for.

Two CLI commands are provided to perform the above operations, rabbitmq-streams add_replica and rabbitmq-streams delete_replica:
````
rabbitmq-streams add_replica [-p <vhost>] <stream-name> <node>
rabbitmq-streams delete_replica [-p <vhost>] <stream-name> <node>
````
To successfully add and remove replicas the stream coordinator must be available in the cluster.

When replacing a cluster node, it is safer to first add a new node, wait for it to become in-sync and then de-comission the node it replaces.

The replication status of a stream can be queried using the following command:
````
rabbitmq-streams stream_status [-p <vhost>] <stream-name>
````
### Channels 
In RabbitMQ, a channel is a virtual connection inside a real TCP connection. When you're working with RabbitMQ, it's common to open a single TCP connection to the broker and then use multiple channels on this connection for different operations. This approach is more efficient than opening multiple TCP connections, as channels are much lighter on resources and can be created and destroyed without the overhead associated with TCP connections.

Channels are particularly useful because they allow for concurrent operations like publishing or consuming messages in isolation, without interfering with each other. Each channel can be thought of as a lightweight connection that shares the main TCP connection's socket, but operates independently in terms of the AMQP commands it processes.

#### Key Points About Channels:

    Concurrency: Multiple channels can operate concurrently over a single connection, making it efficient for multi-threaded applications.
    Isolation: Channels provide a way to segregate operations. If an error occurs on one channel, it can be closed without affecting the connection or other channels.
    Resource Efficiency: Using channels reduces the networking and resource overhead compared to using multiple connections.

Example of publisher (pika):
```python
import pika

# Establish a connection to RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Declare a queue named 'hello'
channel.queue_declare(queue='hello')

# Publish a message to the 'hello' queue
channel.basic_publish(exchange='',
                      routing_key='hello',
                      body='Hello World!')

print(" [x] Sent 'Hello World!'")

# Close the connection
connection.close()
```
Example of consumer (pika):
```python
import pika

def callback(ch, method, properties, body):
    print(f" [x] Received {body}")

# Establish a connection to RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Declare the same queue as the sender to make sure it exists
channel.queue_declare(queue='hello')

# Subscribe to the queue
channel.basic_consume(queue='hello',
                      auto_ack=True,
                      on_message_callback=callback)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```
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
````shell
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
```bash
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
rabbitmqctl -n rabbit@rabbitmq-02 -p / list_queues name messages_ready messages_unacknowledged messages
..
Timeout: 60.0 seconds ...
Listing queues for vhost / ...
name	messages_ready	messages_unacknowledged	messages
hello	0	            0	                    0
QueueA	3	            0	                    3
QueueB	3	            0	                    3
QueueC	3	            0	                    3

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
```
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
````python
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
````python
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
````python
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
````python
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
### TODO
Quorum Queues: Use quorum queues for highly available and consistent queues, which are an improvement over mirrored queues.

Load Balancing: Implement load balancers in front of RabbitMQ nodes for equal distribution of client connections.

Flow Control: Understand RabbitMQ's flow control mechanisms to prevent memory and disk overloads.

Authentication and Authorization: Use RabbitMQ's internal database or integrate with external authentication mechanisms (LDAP) for managing user access.

Disaster Recovery Plan: Have a disaster recovery plan in place, including procedures for restoring from backups and switching to a standby setup if necessary.

Dead Letter Exchanges: Utilize dead letter exchanges to handle undeliverable messages properly.

Queue Length and Message TTL: Set appropriate queue lengths and message TTLs to avoid excessive memory usage and ensure timely processing.

Consumer Scaling: Scale your consumers adequately to handle the workload without causing backlogs in the queues.

Staying Updated: Keep RabbitMQ and Erlang/OTP up to date with the latest stable versions to benefit from performance improvements and bug fixes.
## RabbitMQ configuration files

    rabbitmq.conf: This is the main configuration file for RabbitMQ. It is typically located in the etc/rabbitmq directory. The file may not exist by default, but you can create it to override default settings.

    advanced.config: This file is used for advanced configuration settings. It allows you to specify more detailed configurations that are not covered by the main rabbitmq.conf file. Like rabbitmq.conf, this file is also located in the etc/rabbitmq directory.

    enabled_plugins: This file lists the plugins that should be enabled when RabbitMQ starts. It is located in the etc/rabbitmq directory. Each line corresponds to a plugin that RabbitMQ will load.

    definitions.json: This JSON file contains definitions for exchanges, queues, bindings, and other RabbitMQ entities. It is typically used for defining the initial state of RabbitMQ when it starts. The file may be found in the etc/rabbitmq directory.

    rabbitmq-env.conf: This file allows you to set environment variables for RabbitMQ. It is used to configure system-level settings and is often found in the same directory as the other configuration files.

Note that the file locations mentioned here are general and may differ based on your installation method (e.g., package manager, manual installation) and operating system (e.g., Linux, Windows).

### rabbitmq.conf
````
# Example rabbitmq.conf file

# Define the RabbitMQ node name
node.name = rabbit@localhost

# Specify the location of the RabbitMQ log files
log.dir = /var/log/rabbitmq

# Set the default user for RabbitMQ management plugin
default_user = guest

# Set the default password for RabbitMQ management plugin
default_pass = guest

# Enable RabbitMQ management plugin
management.load_definitions = /etc/rabbitmq/definitions.json
management.tcp.port = 15672

# Set the RabbitMQ server port
listeners.tcp.default = 5672

# Configure the RabbitMQ memory-based flow control threshold
vm_memory_high_watermark.relative = 0.4

## Alternatively, we can set a limit (in bytes) of RAM used by the node.
##
# vm_memory_high_watermark.absolute = 1073741824
# vm_memory_high_watermark.absolute = 2GB
## Supported unit symbols:
##
## k, kiB: kibibytes (2^10 - 1,024 bytes)
## M, MiB: mebibytes (2^20 - 1,048,576 bytes)
## G, GiB: gibibytes (2^30 - 1,073,741,824 bytes)
## kB: kilobytes (10^3 - 1,000 bytes)
## MB: megabytes (10^6 - 1,000,000 bytes)
## GB: gigabytes (10^9 - 1,000,000,000 bytes)

## Fraction of the high watermark limit at which queues start to
## page message out to disc in order to free up memory.
## For example, when vm_memory_high_watermark is set to 0.4 and this value is set to 0.5,
## paging can begin as early as when 20% of total available RAM is used by the node.
##
## Values greater than 1.0 can be dangerous and should be used carefully.
##
## One alternative to this is to use durable queues and publish messages
## as persistent (delivery mode = 2). With this combination queues will
## move messages to disk much more rapidly.
##
## Another alternative is to configure queues to page all messages (both
## persistent and transient) to disk as quickly
## as possible, see https://rabbitmq.com/lazy-queues.html.
##
# vm_memory_high_watermark_paging_ratio = 0.5

## Selects Erlang VM memory consumption calculation strategy. Can be `allocated`, `rss` or `legacy` (aliased as `erlang`),
## Introduced in 3.6.11. `rss` is the default as of 3.6.12.
## See https://github.com/rabbitmq/rabbitmq-server/issues/1223 and rabbitmq/rabbitmq-common#224 for background.
# vm_memory_calculation_strategy = rss

## Interval (in milliseconds) at which we perform the check of the memory
## levels against the watermarks.
##
# memory_monitor_interval = 2500

## The total memory available can be calculated from the OS resources
## - default option - or provided as a configuration parameter.
# total_memory_available_override_value = 2GB

## Set disk free limit (in bytes). Once free disk space reaches this
## lower bound, a disk alarm will be set - see the documentation
## listed above for more details.
##
## Absolute watermark will be ignored if relative is defined!
# disk_free_limit.absolute = 50000

## Or you can set it using memory units (same as in vm_memory_high_watermark)
## with RabbitMQ 3.6.0+.
# disk_free_limit.absolute = 500KB
# disk_free_limit.absolute = 50mb
# disk_free_limit.absolute = 5GB

## Alternatively, we can set a limit relative to total available RAM.
##
## Values lower than 1.0 can be dangerous and should be used carefully.
# disk_free_limit.relative = 2.0

# Enable RabbitMQ SSL
listeners.ssl.default = 5671
ssl_options.cacertfile = /etc/rabbitmq/ca_certificate.pem
ssl_options.certfile = /etc/rabbitmq/server_certificate.pem
ssl_options.keyfile = /etc/rabbitmq/server_key.pem

# Set the RabbitMQ heartbeat interval
heartbeat = 60

# Configure RabbitMQ clustering
cluster_formation.peer_discovery_backend = rabbit_peer_discovery_classic_config
cluster_formation.classic_config.nodes.1 = rabbit@node1
cluster_formation.classic_config.nodes.2 = rabbit@node2
cluster_formation.classic_config.nodes.3 = rabbit@node3

# Enable RabbitMQ federation
federation-upstream = upstream
federation-upstream-set = upstream-set

# Enable RabbitMQ Shovel plugin
shovel.src-uri = amqp://user:password@source_host
shovel.src-queue = source_queue
shovel.dest-uri = amqp://user:password@destination_host
shovel.dest-queue = destination_queue


## Set the max permissible size of an AMQP frame (in bytes).
##
# frame_max = 131072

## Set the max frame size the server will accept before connection
## tuning occurs
##
# initial_frame_max = 4096

## Set the max permissible number of channels per connection.
## 0 means "no limit".
##
# channel_max = 128
````
### vm_memory_high_watermark.*
The RabbitMQ server detects the total amount of RAM installed in the computer on startup and when
rabbitmqctl set_vm_memory_high_watermark fraction is executed. By default, when the RabbitMQ server uses above 40% of the available RAM, it raises a memory alarm and blocks all connections that are publishing messages. Once the memory alarm has cleared (e.g. due to the server paging messages to disk or delivering them to clients that consume and acknowledge the deliveries) normal service resumes. <br>
The default memory threshold is set to 40% of installed RAM. Note that this does not prevent the RabbitMQ server from using more than 40%, it is merely the point at which publishers are throttled. Erlang's garbage collector can, in the worst case, cause double the amount of memory to be used (by default, 80% of RAM). It is strongly recommended that OS swap or page files are enabled. <br>

The memory limit is appended to the log file when the RabbitMQ node starts:
````
2019-06-10 23:17:05.976 [info] <0.308.0> Memory high watermark set to 1024 MiB (1073741824 bytes) of 8192 MiB (8589934592 bytes) total
````
The memory limit may also be queried using the rabbitmq-diagnostics memory_breakdown and rabbitmq-diagnostics status commands.

The threshold can be changed while the broker is running using the
````
rabbitmqctl set_vm_memory_high_watermark <fraction>
````
command or
````
rabbitmqctl set_vm_memory_high_watermark absolute <memory_limit>
````
For example:
````
rabbitmqctl set_vm_memory_high_watermark 0.6
````
and
````
rabbitmqctl set_vm_memory_high_watermark absolute "4G"
````

    Option Name: vm_memory_high_watermark.relative
    Description: This option controls the high watermark level for RabbitMQ's memory usage.
    Default Value: 0.4
    Possible Values: A floating-point number between 0.0 and 1.0.
    Purpose: RabbitMQ uses memory-based flow control to prevent excessive memory usage. When the memory usage reaches the level defined by vm_memory_high_watermark.relative, RabbitMQ will take actions to slow down or stop producers to avoid overwhelming the system with messages.
    Explanation: The value of vm_memory_high_watermark.relative is a fraction of the available system memory. The RabbitMQ node will start considering the system to have high memory usage when the used memory reaches this fraction of the total available memory. For example, if the total available memory is 1 GB and vm_memory_high_watermark.relative is set to 0.4, RabbitMQ will start taking actions when the used memory reaches 40% (0.4 * 1 GB) of the total available memory.

This setting is crucial for preventing RabbitMQ from exhausting system resources during periods of high message traffic. It helps maintain stability by applying backpressure when memory usage is approaching critical levels, allowing the system to recover or adjust accordingly. Adjusting this value may be necessary based on the specific requirements and available resources of your RabbitMQ deployment.
#### Stop All Publishing
When the threshold or absolute limit is set to 0, it makes the memory alarm go off immediately and thus eventually blocks all publishing connections. This may be useful if you wish to deactivate publishing globally:
````
rabbitmqctl set_vm_memory_high_watermark 0
````
#### Configuring the Paging Threshold
This section is obsolete or not applicable for quorum queues, streams and classic queues storage version 2 (CQv2). All of them actively move data to disk and do not generally accumulate a significant backlog of messages in memory. <br>
Before the broker hits the high watermark and blocks publishers, it will attempt to free up memory by instructing CQv1 queues to page their contents out to disc. Both persistent and transient messages will be paged out (the persistent messages will already be on disc but will be evicted from memory). <br>
By default this starts to happen when the broker is 50% of the way to the high watermark (i.e. with a default high watermark of 0.4, this is when 20% of memory is used). To change this value, modify the vm_memory_high_watermark_paging_ratio configuration from its default value of 0.5. For example:
````
vm_memory_high_watermark_paging_ratio = 0.75
vm_memory_high_watermark.relative = 0.4
````
The above configuration starts paging at 30% of memory used, and blocks publishers at 40%.
### disk_free_limit.*
When free disk space drops below a configured limit (50 MB by default), an alarm will be triggered and all producers will be blocked.

The goal is to avoid filling up the entire disk which will lead all write operations on the node to fail and can lead to RabbitMQ termination.

This setting is essential for preventing RabbitMQ from overwhelming the disk with persistent messages, especially in scenarios where disk space is limited. Adjusting this value allows you to configure the system's response to low disk space conditions, ensuring that RabbitMQ takes appropriate actions to maintain stability and avoid potential out-of-disk errors.

The limit can be changed while the broker is running using the 
````
rabbitmqctl set_disk_free_limit
````
command. This command will have its effect until the next node restart.

To reduce the risk of filling up the disk, all incoming messages are blocked. Transient messages, which aren't normally persisted, are still paged out to disk when under memory pressure, and will use up the already limited disk space.

If the disk alarm is set too low and messages are paged out rapidly, it is possible to run out of disk space and crash RabbitMQ in between disk space checks (at least 10 seconds apart). A more conservative approach would be to set the limit to the same as the amount of memory installed on the system (see the configuration below).

Monitoring will begin on node start. It will leave a log entry like this:
````
2019-04-01 12:02:11.564 [info] <0.329.0> Enabling free disk space monitoring
2019-04-01 12:02:11.564 [info] <0.329.0> Disk free limit set to 950MB
````
Free disk space monitoring will be deactivated on unrecognised platforms, causing an entry such as the one below:
````
2019-04-01 11:04:54.002 [info] <0.329.0> Disabling disk free space monitoring
````
When running RabbitMQ in a cluster, the disk alarm is cluster-wide; if one node goes under the limit then all nodes will block incoming messages.

When free disk space drops below the configured limit, RabbitMQ will block producers and prevent memory-based messages from being paged to disk. This will reduce the likelihood of a crash due to disk space being exhausted, but will not eliminate it entirely. In particular, if messages are being paged out rapidly it is possible to run out of disk space and crash in the time between two runs of the disk space monitor. A more conservative approach would be to set the limit to the same as the amount of memory installed on the system (see the configuration section below).
### Heartbeat
 Set the server AMQP 0-9-1 heartbeat timeout in seconds.
 RabbitMQ nodes will send heartbeat frames at roughly
 the (timeout / 2) interval. Two missed heartbeats from
 a client will close its connection.

    Option Name: heartbeat
    Description: This option sets the interval, in seconds, between heartbeat messages exchanged between RabbitMQ clients and the server.
    Default Value: 60 seconds
    Possible Values: Any positive integer representing the interval in seconds.
    Purpose: The heartbeat mechanism is designed to detect network issues or unresponsive peers in a timely manner. If a connection's heartbeat interval passes without any communication, the connection is considered dead, and appropriate actions can be taken, such as closing and re-establishing the connection.
    Explanation: For example, if heartbeat is set to 60, RabbitMQ clients and the server will exchange heartbeat messages every 60 seconds. If, for any reason, a client or the server does not receive a heartbeat within the expected interval, it may assume that the connection is lost and take appropriate measures to handle the situation.

## Paging data from RAM to Disk
In a computer system, RAM is used to store actively used programs and data. However, if the amount of data being used exceeds the available physical RAM, the operating system may use a portion of the hard disk as virtual memory to compensate for the shortage.

    Paging: The operating system divides physical memory into fixed-size blocks known as pages. When the system runs out of available physical RAM, it moves less frequently used or inactive pages of data from RAM to the hard disk. This frees up space in RAM for more critical processes.

    Swapping: The act of moving pages between RAM and the hard disk is often referred to as swapping. The data that is swapped out to the hard disk is stored in a file known as the paging file or swap file.

    Paging File: The paging file is a reserved space on the hard disk that the operating system uses for virtual memory management. It is used to store pages of data that are not immediately needed in RAM.

The use of a paging file allows the operating system to handle situations where the demand for memory exceeds the physical capacity. However, accessing data from the hard disk is significantly slower than accessing data from RAM, so excessive paging can lead to performance degradation.

In the context of RabbitMQ or other server applications, it's important to monitor and manage paging carefully, as excessive paging can impact the overall performance of the system. RabbitMQ, for example, relies on sufficient system resources, including RAM, for efficient message processing. If the system is heavily paging, it may indicate a need for more physical RAM or adjustments to the system's configuration.

## Prefetch Count
The prefetch count setting in RabbitMQ is a crucial parameter for controlling how many messages are sent over a channel before an acknowledgment is received. It's part of the Quality of Service (QoS) settings and is used to limit the number of unacknowledged messages on a channel. This can help to distribute messages more evenly among consumers and prevent a single consumer from being overwhelmed, which is particularly useful in work queue scenarios where workload distribution among multiple workers is desired.
How Prefetch Count Works:

    When prefetch count is set to 1, RabbitMQ will deliver one message to a consumer at a time. The consumer must acknowledge this message before RabbitMQ will deliver the next one.
    Setting a higher prefetch count allows consumers to work on multiple messages at once, but increases the risk that messages might be re-delivered if a consumer crashes or becomes unavailable before acknowledging them.
    The optimal prefetch count value depends on the nature of the task and the processing time of each message.

### Setting Prefetch Count
The prefetch count can be set at the channel level or the consumer level, depending on the client library you're using. Here are examples for some common RabbitMQ client libraries.

Pika
```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.queue_declare(queue='task_queue')

# Set the prefetch count
channel.basic_qos(prefetch_count=1)

def callback(ch, method, properties, body):
    print(f" [x] Received {body}")
    # Simulate a task
    time.sleep(body.count(b'.'))
    print(" [x] Done")
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(queue='task_queue', on_message_callback=callback)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```
Java (Using Spring AMQP): <br>
In a Spring Boot application, you can configure the prefetch count in the application properties or through configuration classes. <br>
application.properties Configuration:
```
spring.rabbitmq.listener.simple.prefetch=1
```
```java
import org.springframework.amqp.core.AcknowledgeMode;
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {

    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(ConnectionFactory connectionFactory) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setPrefetchCount(1);
        factory.setAcknowledgeMode(AcknowledgeMode.AUTO);
        return factory;
    }
}

```

### Considerations:

    Auto Acknowledgment: If auto acknowledgment is enabled (not recommended for tasks that should not be lost), the prefetch count might not be as effective since RabbitMQ considers a message acknowledged once it's delivered.
    Multiple Consumers: If you have multiple consumers on the same channel, the prefetch count is divided among them.
    Message Size: For larger messages, a lower prefetch count can prevent memory issues, especially if the consumer processes messages slowly.
## Connection Pooling
1. Use Connection Pooling Libraries

    Implement or utilize existing connection pooling libraries specifically designed for RabbitMQ. Libraries such as rabbitmq-dotnet-client for .NET or pika for Python offer features to manage connections efficiently.

2. Minimize Connections

    Share Connections: Where possible, share connections between threads or application components instead of opening a new connection for each thread or task.
    Connection Per Application: Aim for a single connection per application (when feasible) and use multiple channels within that connection for different threads or tasks.

3. Optimize Channel Usage

    Channels over Connections: Use multiple channels within a single connection to parallelize tasks rather than multiple connections. Channels are lightweight and can be used by different threads to communicate with RabbitMQ.
    Close Channels Properly: Ensure that channels are properly closed when they are no longer needed to free up resources.

4. Monitor and Tune Connection Parameters

    Heartbeat Timeout: Configure the heartbeat timeout setting to ensure that idle connections are kept alive or closed appropriately, which helps in detecting and freeing up dead connections.
    Connection Timeout: Set appropriate connection timeouts to avoid hanging connections, which can occupy valuable resources.

5. Handle Connection Failures Gracefully

    Automatic Recovery: Utilize RabbitMQ client features that support automatic connection and channel recovery in case of connection failures.
    Retry Mechanisms: Implement intelligent retry mechanisms with exponential backoff to handle temporary network issues or RabbitMQ server unavailability.

6. Use High Availability and Load Balancing

    Cluster Configuration: In a clustered RabbitMQ setup, ensure connections are distributed evenly across the cluster nodes to balance the load.
    Client-Side Load Balancing: Use client-side load balancing to distribute connections and channels across multiple RabbitMQ nodes in a cluster.

7. Configure Resource Limits

    Limit the Number of Connections: Configure RabbitMQ to limit the number of connections and channels per connection to prevent overuse of resources.
    Memory and Disk Space Alarms: Set up memory and disk space alarms in RabbitMQ to get alerts when resource usage is high, which might indicate issues with connection or channel usage.

8. Regular Monitoring and Auditing

    Monitor Connection Metrics: Regularly monitor metrics related to connections and channels, such as the number of active connections, channels per connection, and message rates.
    Audit and Optimize: Periodically audit your connection usage and pooling strategy to identify inefficiencies and opportunities for optimization.

9. Adjust OS-Level Settings

    File Descriptors Limit: Ensure the operating system's file descriptor limit is high enough to support the number of connections and channels your application needs.
    Networking Configuration: Tune networking parameters such as TCP keepalive to ensure connections are managed efficiently at the OS level.

## Erlang VM and System optimizations
Erlang VM Options:

    Garbage Collection Tuning
        +hms <size>: Sets the default heap size for processes.
        +hmbs <size>: Sets the default heap block size for processes.

    Scheduler Settings
        +S <schedulers>:<schedulers_online>: Configures the number of schedulers and the number of schedulers online. This should align with the number of CPU cores.

    Async Threads
        +A <count>: Increases the number of asynchronous IO threads, improving file and socket IO performance.

    Network Settings
        +K true: Enables kernel poll, which can improve networking performance on supported platforms.

    Symmetric Multiprocessing (SMP) Support
        +S <cores>: Enables SMP support, setting the number of scheduler threads to the number of CPU cores.

    Memory Allocation
        +MMmcs <size>: Adjusts the maximum memory block size for multiblock carriers.
        +M<type> <option>: Fine-tunes memory allocators (e.g., +Ml, +Mu, +Ms) for different types of memory usage.

    Process Limits
        +P <number>: Sets the maximum number of concurrent Erlang processes.

    Low Latency Optimizations
        Specific options for low latency are more about tuning the above parameters (like garbage collection and scheduler settings) to favor responsiveness over throughput.

RabbitMQ Configuration Options:

    File Descriptors
        Increase the file descriptor limit at the OS level and configure RabbitMQ (ulimit -n <number>) to allow more concurrent connections and files.

    Memory and Disk Alarms
        Set vm_memory_high_watermark and disk_free_limit in the RabbitMQ configuration file to control memory and disk usage alarms.

    Network Configuration
        Adjust tcp_listeners and ssl_listeners to define ports and interfaces for RabbitMQ to accept client connections.

    High Availability and Clustering
        Configure cluster_partition_handling, ha_policy, and ha_sync_mode to manage behavior in clustered environments.

    Management Plugin
        Enable the RabbitMQ management plugin (rabbitmq-plugins enable rabbitmq_management) to monitor and manage RabbitMQ nodes.

    Logging and Tracing
        Configure log_levels to adjust the verbosity of RabbitMQ logs for better insights during optimization and troubleshooting.

    Message and Queue Limits
        Set queue_index_embed_msgs_below and queue_index_max_journal_entries to optimize message storage and retrieval.

System-Level Optimizations:

    File Descriptors Limit
        ulimit -n <number>: Sets the limit for open files, which is critical for RabbitMQ's ability to handle many simultaneous connections.

    Networking Parameters
        Adjust TCP/IP settings such as tcp_fin_timeout, tcp_tw_reuse, and tcp_keepalive_time to improve network performance and connection handling.

    Disk Performance
        Use faster disks (e.g., SSDs) and configure RAID levels appropriately for the balance of performance and redundancy.

## WAL
In the context of RabbitMQ, WAL stands for Write-Ahead Logging, a concept borrowed from database systems to enhance data integrity and durability. RabbitMQ uses WAL as part of its message store for durable queues to ensure that messages are not lost in the event of a crash or failure.
### How WAL Works in RabbitMQ
Operations first get written to memory and to a Write Ahead Log (WAL). There is a single WAL per broker, which serves all quorum queues on that broker. From there, operations are then written to per-queue segment files by the Segment Writer.
![](https://github.com/Frodo-Web/frodo-tips/blob/main/linux-admin/images/wal-and-segments.png?raw=true)

    Message Persistence: When a message is published to a durable queue with the delivery_mode property set to persistent (2), RabbitMQ writes the message to the WAL before it is actually stored in the queue's message store. This is done to ensure that, in case of a failure, the system can recover the message from the WAL.

    Data Integrity: The WAL records changes to the message store in a sequential manner. If RabbitMQ crashes or is restarted, it can use the WAL to replay and restore the state of the message store up to the last known good state, ensuring data integrity.

    Performance Consideration: While WAL enhances durability, it can also impact performance due to the additional disk I/O required for logging the message before storing it. RabbitMQ optimizes this process to balance durability with performance.

    Recovery Process: Upon restart, RabbitMQ checks the WAL for any unprocessed entries. If any are found, it replays them to ensure that the message store reflects all changes made before the crash.

    WAL and Mirrored Queues: In a clustered RabbitMQ setup with mirrored queues, WAL plays a crucial role in ensuring that messages are replicated across nodes in a consistent manner. Each node maintains its own WAL for the queues it hosts, ensuring durability across the cluster.

### Configurations Related to WAL:

    Queue Durability: As mentioned, for WAL to be utilized, the queue must be declared as durable.
    Message Persistence: Messages must be published with delivery_mode set to 2 (persistent).
    Disk Free Limit: RabbitMQ has a configurable disk free limit (disk_free_limit), which ensures that the node has enough disk space to operate safely. This setting is crucial for the WAL's operation, as running out of disk space could prevent new entries from being logged, affecting durability.
    Memory Thresholds: RabbitMQ also has memory thresholds that, when exceeded, trigger flow control or message paging to disk, involving WAL operations to ensure messages are not lost even when memory is constrained.

In summary, WAL in RabbitMQ is a mechanism to ensure that messages published to durable queues are not lost in the event of a system crash or failure, providing a higher level of data integrity and durability at the cost of some performance overhead.

The primary storage-related setting that can affect quorum queue resource use is the write-ahead log segment size limit, the limit at which WAL in-memory table will be moved to disk. In other words, every quorum queue would be able to keep up to this much message data in memory under steady load.
The limit can be controlled
```
# Flush current WAL file to a segment file on disk once it reaches 32 MiB in size
raft.wal_max_size_bytes = 32000000
```
Because memory deallocation may take some time, we recommend that the RabbitMQ node is allocated at least 3 times the memory of the default WAL file size limit. More will be required in high-throughput systems. 4 times is a good starting point for those.

## Mnesia
Mnesia is the distributed database management system that comes built into the Erlang/OTP platform, which RabbitMQ is built upon. RabbitMQ uses Mnesia for storing metadata about the broker itself, including information about users, queues, exchanges, bindings, and cluster configuration. Mnesia supports both in-memory and disk-based storage, making it suitable for high-throughput environments like those RabbitMQ often operates in.

Key Mnesia Options for RabbitMQ:

    Table Fragmentation:
        Mnesia tables can be fragmented to distribute the load and improve scalability and performance. RabbitMQ allows you to configure the degree of fragmentation for certain tables, like the queue table, to better support large deployments.

    RAM and Disk Nodes:
        Mnesia allows data to be stored in RAM for faster access or on disk for persistence. RabbitMQ nodes can be configured as disk nodes to store Mnesia data persistently, ensuring that critical metadata survives broker restarts.

    Backup and Restore:
        Mnesia provides built-in mechanisms for backing up and restoring its data. RabbitMQ leverages these capabilities to allow users to backup and restore the state of the broker, including its configuration and persistent message data.

    Schema Management:
        RabbitMQ allows for the management of the Mnesia schema, which includes creating, migrating, or updating the schema as necessary, especially during upgrades or when changing cluster configurations.

    Cluster Configuration:
        Mnesia's distributed nature supports RabbitMQ's clustering capabilities, allowing for broker metadata to be replicated across nodes in a RabbitMQ cluster. This includes support for automatic synchronization of schema and data across nodes.

    Table Locking and Transactions:
        Mnesia supports ACID transactions, which RabbitMQ uses to ensure data consistency, especially in operations that involve multiple steps or need to be atomic, such as creating a queue along with its bindings.

    Configurable Storage Locations:
        The location where Mnesia stores its data can be configured, allowing administrators to specify directories that might be on faster storage media or have more space available.

Configuring Mnesia for RabbitMQ:

RabbitMQ's use of Mnesia is largely managed internally by the application, but there are several environment variables and configuration options that can be set to tune Mnesia's behavior:

    RABBITMQ_MNESIA_DIR: Sets the directory where Mnesia data is stored.
    RABBITMQ_MNESIA_BASE: Specifies the base directory for Mnesia data, allowing for more granular control over where different types of Mnesia data are stored.

When adjusting Mnesia configurations, it's crucial to ensure that the changes are compatible with your RabbitMQ version and deployment architecture. Improper configurations can lead to issues with data consistency, cluster stability, and performance.

Best Practices:

    Regular Backups: Regularly back up your Mnesia database to recover from data loss or corruption.
    Monitor Disk Usage: Keep an eye on the disk space used by Mnesia, especially in environments with a high volume of changes to metadata.
    Cluster Considerations: In clustered environments, ensure that Mnesia is correctly configured to replicate data across all nodes, and be mindful of the implications of network partitions on Mnesia's data consistency.

RabbitMQ's use of Mnesia is a key part of its robustness and clustering capabilities, and understanding how to configure and manage Mnesia can help in optimizing RabbitMQ deployments for different scenarios.
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
