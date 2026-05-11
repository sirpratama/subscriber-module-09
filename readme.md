# Subscriber

## Questions

### a. What is AMQP?

AMQP stands for **Advanced Message Queuing Protocol**. It is an open standard application layer protocol designed for message-oriented middleware. AMQP defines how messages are formatted, routed, and reliably delivered between applications and a message broker (such as RabbitMQ). It enables different applications — potentially written in different programming languages and running on different systems — to communicate asynchronously through a common broker without being directly coupled to each other.

### b. What does `guest:guest@localhost:5672` mean?

The connection URL `amqp://guest:guest@localhost:5672` can be broken down as follows:

- **First `guest`** — This is the **username** used to authenticate with the RabbitMQ message broker. By default, RabbitMQ ships with a user named `guest`.
- **Second `guest`** — This is the **password** for that user. The default password for the `guest` user is also `guest`.
- **`localhost`** — This refers to the local machine (127.0.0.1), meaning RabbitMQ is running on the same machine as the subscriber program.
- **`5672`** — This is the **port number** that RabbitMQ listens on for incoming AMQP connections. Port 5672 is the standard default port for AMQP.

## Simulation Slow Subscriber

To simulate a slow subscriber, `thread::sleep(ten_millis)` was uncommented in the handler, introducing a 1-second delay for every message processed. The publisher was then run multiple times in quick succession.

![Multiple Spikes Queue](images/multiple spikes.png)

### Why is the total number of queued messages as such?

The queue builds up because the **publisher produces messages much faster than the subscriber can consume them**. Each publisher run sends 5 messages almost instantly. However, the subscriber now takes 1 second to process each message. When the publisher is run several times rapidly, the broker accumulates a backlog of unprocessed messages in the queue. The total number of queued messages grows with each publisher run and only decreases slowly as the subscriber works through them one by one at its 1-second-per-message rate. This demonstrates a classic producer–consumer imbalance: when the consumer is slower than the producer, the queue acts as a buffer, absorbing the excess load instead of dropping or blocking messages.
