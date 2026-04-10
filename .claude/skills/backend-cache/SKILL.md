---
name: backend-cache
description: Backend Cache Design Guide. Provides guidance for cache strategies, patterns, and implementation.
version: 1.0.0
---

# Cache Design Guide

Guide for backend caching strategies and implementation.

---

## Cache Strategies

### 1. Cache-Aside (Recommended)

```rust
async fn get_user(id: i64) -> Result<User, Error> {
    // 1. Check cache
    let key = format!("user:{}", id);
    if let Some(cached) = redis.get(&key).await? {
        return Ok(serde_json::from_str(&cached)?);
    }

    // 2. Fetch from DB
    let user = db.find_user(id).await?;

    // 3. Store in cache
    redis.set(&key, &serde_json::to_string(&user)?).await?;

    Ok(user)
}
```

### 2. Write-Through

```rust
async fn create_user(user: User) -> Result<User, Error> {
    // 1. Write to DB
    let user = db.create_user(user).await?;

    // 2. Write to cache
    let key = format!("user:{}", user.id);
    redis.set(&key, &serde_json::to_string(&user)?).await?;

    Ok(user)
}
```

### 3. Write-Behind

```rust
async fn update_user(id: i64, data: UserUpdate) -> Result<(), Error> {
    // 1. Update cache immediately
    let key = format!("user:{}", id);
    redis.update(&key, &data).await?;

    // 2. Async write to DB (eventually)
    queue.push(UpdateUserEvent { id, data }).await?;

    Ok(())
}
```

---

## Cache Patterns

### Pattern Selection

| Pattern | Write | Read | Use Case |
|---------|-------|------|----------|
| Cache-Aside | DB | Cache → DB | Read-heavy |
| Write-Through | Cache → DB | Cache | Write-heavy |
| Write-Behind | Cache → async DB | Cache | High write |
| Refresh-Ahead | Background | Cache → refresh | Predictable |

---

## Invalidation Strategies

### Time-based (TTL)

```rust
// Cache for 5 minutes
redis.set_ex("key", "value", 300);
```

### Event-based

```rust
// Invalidate on update
async fn update_user(id: i64, data: UserUpdate) -> Result<(), Error> {
    db.update_user(id, data).await?;

    // Invalidate cache
    redis.del(format!("user:{}", id)).await?;

    // Publish event for distributed cache
    publisher.publish("user.updated", id).await?;
}
```

### Pattern: Distributed Invalidation

```
Service A updates user
    ↓
Publishes "user:123:updated" to message queue
    ↓
All services invalidate local/user cache
```

---

## Cache Types

| Type | Use Case | Example |
|------|----------|---------|
| **Local (In-Memory)** | Hot data, low latency | HashMap, RwLock |
| **Distributed** | Shared cache, scaling | Redis, Memcached |
| **HTTP/CDN** | Static assets | CloudFront, S3 |

---

## Implementation Checklist

- [ ] Strategy selected
- [ ] TTL configured
- [ ] Invalidation defined
- [ ] Cache key scheme designed
- [ ] Monitoring set up
- [ ] Fallback defined
