---
name: backend-microservices
description: Microservices Design and Communication Guide. Covers service design principles, protocol selection, and use cases.
version: 1.0.0
---

# Microservices Design

Guide for microservices architecture and inter-service communication.

---

## Service Design Principles

### Bounded Context

Each service should have:
- Clear boundaries
- Own data
- Independent deployment

### Service Size Guidelines

| Metric | Target |
|--------|--------|
| Team size | 5-9 people |
| Code size | 10K-50K lines |
| Deployment | < 1 hour |
| Startup time | < 30 seconds |

### Coupling

- **High Cohesion**: Related functionality together
- **Low Coupling**: Services depend on contracts, not internals

---

## Communication Patterns

### 1. Synchronous (Request-Response)

**When to Use:**
- Need immediate response
- User-facing operations
- Simple queries

**Protocols:**

| Protocol | Use Case | Pros | Cons |
|----------|---------|------|------|
| **REST** | CRUD, simple APIs | Ubiquitous, familiar | Over-fetching |
| **gRPC** | High performance | Fast, type-safe | Learning curve |
| **GraphQL** | Flexible queries | Client-defined queries | Complexity |

### 2. Asynchronous (Event-Driven)

**When to Use:**
- Long-running operations
- Cross-service updates
- Decoupling required

**Patterns:**

| Pattern | Description |
|---------|-------------|
| **Pub/Sub** | One-to-many |
| **Event Sourcing** | State as event log |
| **CQRS** | Separate read/write |

---

## Protocol Selection Guide

### Decision Tree

```
Need immediate response?
├─ Yes → REST or gRPC
│         └─ High performance? → gRPC
│         └─ Standard CRUD → REST
│
└─ No → Use Message Queue
          ├─ One-to-many → Pub/Sub
          ├─ Transactional → Saga
          └─ High throughput → Kafka
```

### REST Design

```yaml
/resources:
  /users:
    GET /users          # List
    POST /users         # Create
    GET /users/{id}     # Read
    PUT /users/{id}     # Update
    DELETE /users/{id}  # Delete
```

### gRPC Design

```protobuf
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc StreamUsers(StreamRequest) returns (stream User);
}
```

---

## Service Mesh

### When to Use

- > 10 services
- Complex routing needs
- Advanced observability

### Options

| Solution | Features | Complexity |
|----------|----------|------------|
| **Istio** | Full-featured | High |
| **Linkerd** | Simpler than Istio | Medium |
| **Envoy** | Data plane | Low |

---

## Inter-Service Communication Checklist

- [ ] Communication pattern selected (sync/async)
- [ ] Protocol chosen
- [ ] Contract defined (OpenAPI/Proto)
- [ ] Error handling defined
- [ ] Timeout configured
- [ ] Retry policy defined
- [ ] Circuit breaker configured
