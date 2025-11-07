# Message Brokers

## Overview

Inside a microservice architecture, each service can either by a Producer/Publisher (who request work), or a Consumer/Subscriber (who perform work). To manage the transmission of messages between producers and consumers, you will need a **message broker**. It receives messages from producers, stores them temporarily, and delivers them to consumers according to defined rules and protocols. Therefore, the whole process can be called "Message Queue".

Core attributes:

- Asynchronous execution (no thread).
- Scalability/Load balancing.
- Backpressure: the rate of message production by producers exceeds the rate of message consumption by consumers.

  + queue is building up gradually
  + messages sit in the queue for longer periods, leading to delays in their processing and delivery to consumers
  + unbounded queues can consume excessive memory/disk space on the message broker => system instability + crash

  => Limit the size of the queue (reject messages then retry later, or even better, enforcing a rate limiter on the publisher), scale the consumers to handle more message, ...

- Retrying.
- Ordering (determine the order of the message)
- Delivery strategy: at-most-once, at-least-once, exactly-once.
- Logging and monitoring.
- Scheduling.

## Redis - one of the most common Message Brokers

Redis design its native transport mechanism called _Redis Pub/Sub_ (generally known as *publish-subscribe model*) that leverages blocking list editing operations (such as `BLPOP`, `BRPOPLPUSH`, ...)

Its strategy is "fire-and-forget": if a message is published and there are no subscribers currently listening to it, the message is dropped and can't be recovered. This strategy ensures highest throughput in exchange for reliability.

Redis Stream is the new transport mechanism which:

- Specilied data structure designed for append-only logs.
- Relibility: store messages and create message history, native doesn't store anything.
- Distribution among Consumters inside a Consumer Group: multiple Consumers can share the processing of messages from a single Redis Stream, ensure each message is delivered to only one Consumer within the group (ensure no two Consumers handle the same message). Without consumer groups, every consumer would get every message, not only wasting computing resource but also duplicating processing, creating unexpected behavior.

=> More messages are handled in parallel, horizontal scaling by simply adding more consumers without needing to change stream's logic.

Overall: Redis as broker is dead simple and fast, but it has less robust message routing/features.

## BullMQ + Redis Stream

A popular stack that's widely seen in small-to-medium scale applications required microservrices and distributed architectures.


