---
name: backend-principles
description: Backend General Technical Principles. Covers async design, decoupling, microservices consistency, cloud-native robustness, and observability. Use for backend architecture decisions.
version: 1.0.0
---

# Backend Technical Principles

Comprehensive guide for backend technical principles.

---

## 1. Async Design

### Core Principles

- **Non-blocking I/O**: Use async APIs to handle I/O without blocking threads
- **Event-driven**: Design around events for loose coupling
- **Backpressure**: Handle slow consumers gracefully

### Best Practices

```rust
// Good: Async handler
async fn get_user(id: i64) -> Result<User, Error> {
    let user = db.find_user(id).await?;
    Ok(user)
}

// Avoid: Blocking in async context
async fn bad_example() {
    let result = blocking_call(); // Blocks the runtime
}
```

### When to Use Async

| Scenario | Recommendation |
|----------|---------------|
| I/O bound (DB, HTTP) | Use async |
| CPU bound | Use blocking or dedicated thread |
| Simple scripts | Sync is fine |

---

## 2. Decoupling & Separation of Concerns

### Principles

- **Single Responsibility**: Each service/component has one job
- **Dependency Inversion**: Depend on abstractions, not concretions
- **Interface Segregation**: Small, focused interfaces

### Patterns

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Layer A   │ ──▶ │   Layer B   │ ──▶ │   Layer C   │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │
   Business Logic      Data Access       Presentation
```

### Coupling Types

| Type | Description | How to Avoid |
|------|-------------|--------------|
| Temporal | A waits for B to complete | Event-driven, async |
| Data | Shared data structures | API contracts |
| Domain | Shared business rules | Bounded contexts |

---

## 3. Microservices & Distributed Consistency

### Consistency Patterns

| Pattern | Use Case | Trade-off |
|---------|----------|-----------|
| **Strong Consistency** | Financial transactions | Availability ↓ |
| **Eventual Consistency** | Social feeds | Temporary inconsistency |
| **Saga Pattern** | Cross-service transactions | Complexity ↑ |
| **CQRS** | Read/write separation | Consistency challenges |

### Common Problems

1. **Distributed Transactions**
   - Use Saga pattern
   - Implement compensation logic
   - Consider idempotency

2. **Network Partitions**
   - Design for failure
   - Implement retry logic
   - Use circuit breakers

3. **Data Consistency**
   - Event sourcing
   - Change data capture
   - Dual writes avoidance

---

## 4. Cloud-Native Robustness

### Principles

- **Design for Failure**: Assume anything can fail
- **Graceful Degradation**: Partial functionality > complete failure
- **Horizontal Scaling**: Stateless services
- **Self-Healing**: Automatic recovery

### Patterns

| Pattern | Description |
|---------|-------------|
| **Circuit Breaker** | Prevent cascading failures |
| **Bulkhead** | Isolate failures |
| **Retry with Backoff** | Handle transient failures |
| **Timeout** | Prevent infinite waits |
| **Rate Limiting** | Protect against overload |

### Implementation

```rust
// Circuit breaker concept
struct CircuitBreaker {
    failures: u32,
    state: CircuitState,
    threshold: u32,
}

impl CircuitBreaker {
    fn call<T>(&mut self, op: impl FnOnce() -> T) -> Result<T, Error> {
        match self.state {
            CircuitState::Open => Err(Error::CircuitOpen),
            CircuitState::HalfOpen => {
                // Try the call
                match op() {
                    Ok(r) => {
                        self.state = CircuitState::Closed;
                        Ok(r)
                    }
                    Err(e) => {
                        self.failures += 1;
                        if self.failures >= self.threshold {
                            self.state = CircuitState::Open;
                        }
                        Err(e)
                    }
                }
            }
            CircuitState::Closed => op(),
        }
    }
}
```

---

## 5. Observability

### Three Pillars

| Pillar | Metrics | Tools |
|--------|---------|-------|
| **Logs** | Structured events | ELK, Loki |
| **Metrics** | Numeric aggregations | Prometheus, Grafana |
| **Traces** | Request flow | Jaeger, Zipkin |

### Best Practices

1. **Structured Logging**
   ```json
   {
     "timestamp": "2024-01-01T00:00:00Z",
     "level": "INFO",
     "message": "User logged in",
     "user_id": "123",
     "trace_id": "abc"
   }
   ```

2. **Metric Naming**
   - Use suffixes: `_total`, `_seconds`, `_bytes`
   - Include units
   - Be consistent

3. **Distributed Tracing**
   - Always include trace_id
   - Add context propagation
   - Instrument at boundaries

### Key Metrics to Track

- Latency (P50, P95, P99)
- Error rate
- Throughput (RPS)
- Resource usage (CPU, Memory)
- Queue depth

---

## Validation Checklist

- [ ] Async used appropriately for I/O
- [ ] Components have clear responsibilities
- [ ] Consistency model chosen for data
- [ ] Circuit breakers implemented
- [ ] Timeouts configured
- [ ] Logs structured
- [ ] Metrics exposed
- [ ] Traces propagated
