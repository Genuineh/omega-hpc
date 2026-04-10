---
name: backend-queue
description: Message Queue Selection and Usage Guide. Provides guidance for message queue selection and implementation patterns.
version: 1.0.0
---

# Message Queue Guide

Guide for message queue selection and usage.

---

## Queue Selection Matrix

| Queue | Throughput | Latency | Persistence | Use Case |
|-------|-----------|---------|-------------|----------|
| **RabbitMQ** | Medium | Low | Yes | General purpose |
| **Kafka** | Very High | Medium | Yes | Event streaming |
| **Redis Streams** | High | Very Low | Optional | Low latency |
| **SQS** | High | Low | Yes | AWS integration |
| **NATS** | Very High | Very Low | Optional | Cloud native |

### Selection Criteria

1. **Persistence needed?** → Kafka, RabbitMQ, SQS
2. **Low latency critical?** → Redis, NATS
3. **High throughput?** → Kafka, NATS
4. **AWS integration?** → SQS, SNS
5. **Complex routing?** → RabbitMQ

---

## Patterns

### 1. Point-to-Point

```
Producer → [Queue] → Consumer
```

Use for:
- Task processing
- Background jobs

### 2. Pub/Sub

```
Publisher → [Topic] → Subscriber 1
                  → Subscriber 2
```

Use for:
- Notifications
- Event broadcasting

### 3. Saga Pattern

```
Order Service → [Create Order] → Payment Service
                    ← [Payment OK] ←

Order Service → [Reserve Inventory] → Inventory Service
```

Use for:
- Distributed transactions
- Long-running workflows

---

## Implementation Examples

### RabbitMQ (Rust)

```rust
use lapin::{
    Connection, ConnectionPolicy,
    options::*, types::FieldTable,
};

async fn publish(queue: &str, message: &str) -> Result<(), Error> {
    let channel = connection.create_channel().await?;

    channel.queue_declare(queue, QueueDeclareOptions::default(), FieldTable::default()).await?;

    channel.basic_publish(
        "",
        queue,
        BasicPublishOptions::default(),
        message.as_bytes(),
        BasicProperties::default(),
    ).await?;

    Ok(())
}
```

### Kafka (Rust)

```rust
use kafka::producer::{Producer, Record};

async fn send_event(topic: &str, key: &str, value: &str) -> Result<(), Error> {
    let mut producer = Producer::from_hosts(vec![broker]).create()?;

    producer.send(&Record {
        topic,
        key: Some(key.as_bytes()),
        value: value.as_bytes(),
        ..Default::default()
    }).await?;

    Ok(())
}
```

---

## Best Practices

### Message Design

```json
{
  "event_id": "uuid",
  "event_type": "user.created",
  "timestamp": "2024-01-01T00:00:00Z",
  "payload": {
    "user_id": "123",
    "email": "user@example.com"
  },
  "metadata": {
    "correlation_id": "abc",
    "causation_id": "xyz"
  }
}
```

### Consumer Patterns

```rust
// Idempotent consumer
async fn process_message(msg: Message) -> Result<(), Error> {
    // Check if already processed
    if processed(msg.id).await? {
        return Ok(());
    }

    // Process
    do_work(msg).await?;

    // Mark as processed
    mark_processed(msg.id).await?;

    Ok(())
}
```

---

## Queue Checklist

- [ ] Queue selected
- [ ] Message format defined
- [ ] Retry policy configured
- [ ] Dead letter queue configured
- [ ] Monitoring set up
- [ ] Idempotency implemented
