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
