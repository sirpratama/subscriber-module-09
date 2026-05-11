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

![Multiple Spikes Queue](images/multiple%20spikes.png)

### Why is the total number of queued messages as such?

The queue builds up because the **publisher produces messages much faster than the subscriber can consume them**. Each publisher run sends 5 messages almost instantly. However, the subscriber now takes 1 second to process each message. When the publisher is run several times rapidly, the broker accumulates a backlog of unprocessed messages in the queue. The total number of queued messages grows with each publisher run and only decreases slowly as the subscriber works through them one by one at its 1-second-per-message rate. This demonstrates a classic producer–consumer imbalance: when the consumer is slower than the producer, the queue acts as a buffer, absorbing the excess load instead of dropping or blocking messages.

## Reflection and Running at Least Three Subscribers

By running three or more subscriber instances all connected to the same `user_created` queue, RabbitMQ distributes the incoming messages across all of them in a round-robin fashion. Each subscriber picks up a different message, so they process the backlog in parallel instead of sequentially.

**Multiple subscriber consoles** — messages are split across instances:

![Multiple Subscribers](images/slow%20start.png)

**RabbitMQ chart** — queue drains significantly faster than with a single slow subscriber:

![Spike Result](images/slow%20start%20spike%20result.png)

### Why does the queue reduce quicker?

With a single slow subscriber taking 1 second per message, 5 messages from one publisher run take 5 seconds to fully process. With three subscribers running simultaneously, those same 5 messages are distributed across them — each subscriber handles roughly 1–2 messages — bringing the total processing time down to about 2 seconds. The more subscribers connected, the faster the queue drains, because the workload is parallelised across multiple consumers all reading from the same queue.

### What could be improved in the code?

Looking at both the publisher and subscriber code, there are a few things worth improving:

1. **Busy-wait loop in subscriber** — the `loop {}` at the end of `main()` spins the CPU at 100% doing nothing. It should be replaced with `std::thread::park()` or a proper blocking wait, or a signal handler that shuts down gracefully on Ctrl-C.
2. **Unused variable `now`** — `time::Instant::now()` is captured but never used. It was likely intended to measure elapsed processing time, but the measurement is never completed or logged.
3. **Ignored errors** — both publisher and subscriber use `_ = p.publish_event(...)` and `_ = listener.listen(...)`, silently discarding any errors. Proper error handling or at least logging would make failures visible.
4. **Hard-coded connection URL** — the AMQP URL `amqp://guest:guest@localhost:5672` is duplicated in both programs. It should be read from an environment variable or config file so it can be changed without recompiling.
