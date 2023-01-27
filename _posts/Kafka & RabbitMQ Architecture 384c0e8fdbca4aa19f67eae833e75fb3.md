# Kafka and RabbitMQ

## **Apache Kafka: pull based approach**

Consumers request batches of messages from a specific offset. Kafka permits long-pooling, which prevents high loops when there is no message past the offset, and aggressively batches messages to support this.

A pull model is logical for Kafka because of partitioned data structure. Kafka provides message order in a partition with no contending consumers. This allows users to leverage the batching of messages for effective message delivery and higher throughput

## **RabbitMQ: Push-based approach**

RabbitMQ uses a push model and stops overwhelming consumers through a prefetch limit defined on the consumer. This can be used for low latency messaging.

The aim of the push model is to distribute messages individually and quickly, to ensure that work is parallelized evenly and that messages are processed approximately in the order in the order in which they arrived in the queue.. However, this can also cause issues in cases where one or more consumers have `died` and are no longer receiving messages.

## **How do they handle messaging?**

| Tool | Apache Kafka | RabbitMQ |
| --- | --- | --- |
| Message ordering | provides message ordering. thanks to its partitioning. Messages are sent to topics by message key | Not supported |
| Message life time | Kafka is a log, which means that it retains messages by default. You. can manage this by specifying a retention policy. | RabbitMQ is a queue, so messages are done away with once consumed, and acknowledgment is provided. |
| Delivery Guarantees | Retains. order only inside a partition. IN a partition, Kafka guarantees that the whole batch of messages either fails or passes | Doesn’t guarantee atomicity, even in relation to transactions involving a single queue. |
| Message priorities | N/A | In RabbitMQ, you can specify message priorities and consume message with high priority first. |

## Kafka vs RabbitMQ Performance

### Apache Kafka

Kafka offers much higher. performance than message brokers like RabbitMQ. It uses sequential disk I/O to boost performance,  making it. a suitable option for implementing queues. It can achieve high throughput (millions of message per second) with limited resources, a necessity for big. data use cases.

### RabbitMQ:

RabbitMQ can also process a million messages per second but requires more resources (around 30 nodes). You can use RabbitMQ for. many of the same use case as Kafka, but you’ll need to combine it. with other tools like Apache Cassandra

## What are the best use cases?

### Apache Kafka Use cases

Apache Kafka provides the broker itself and has been design. towards stream processing scenarios. Recently, it has added Kafka Streams, a client library for building applications and Microservices. Apache Kafka supports use cases such as metrics, activity tracking, log aggregation, stream processing, commit logs, and event sourcing.

The following message scenarios are especially. suited for Kafka:

- Streams with complex routing, throughput of 100K/sec events or more, with at least once partitioned ordering
- Applications requiring a stream history, delivered in “at least once” partitioned ordering. Clients can see a “replay” of the event stream.
- Event sourcing, modeling changes to a system as a sequence of events.
- Stream processing data in multi-stage pipelines. The pipelines generate graphs of real-time data flows.

### RabbitMQ use cases

RabbitMQ can be used when web servers need to quickly respond to requests. This eliminates the need to perform resource-intensive activities while the user wait for a result. RabbitMQ is also used to convey a message to various recipients for consumption or to share loads between workers under high load (20K+ messages/second).

Scenarios that RabbitMQ can be used for:

- Applications that need to support legacy protocols, such as STOMP, MQTT, AMQP, 0-9-1
- Granular control over consistency/set of guarantees on a per-message basis.
- Complex routing to consumers.
- Applications that need a variety of publish/subscribe point-to-point request/reply messaging capabilities.

Kafka with Data Lake

![Untitled](Kafka%20&%20RabbitMQ%20Architecture%20384c0e8fdbca4aa19f67eae833e75fb3/Untitled.png)
