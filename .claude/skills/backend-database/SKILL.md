---
name: backend-database
description: Database Design and Selection Guide. Provides guidance for database selection and schema design for different use cases.
version: 1.0.0
---

# Database Design and Selection

Guide for database selection and schema design.

---

## Database Selection

### Decision Matrix

| Use Case | Recommended | Alternative |
|----------|-----------|-------------|
| **Transactional (OLTP)** | PostgreSQL, MySQL | CockroachDB |
| **Analytical (OLAP)** | ClickHouse, BigQuery | PostgreSQL + extensions |
| **Document store** | MongoDB, PostgreSQL JSON | CouchDB |
| **Key-Value** | Redis, etcd | DynamoDB |
| **Graph** | Neo4j | PostgreSQL + graph |
| **Time series** | InfluxDB, TimescaleDB | PostgreSQL |
| **Cache** | Redis | Memcached |
| **Search** | Elasticsearch, Meilisearch | PostgreSQL full-text |

### Selection Criteria

1. **Data Model**
   - Structured (SQL) vs Unstructured (NoSQL)
   - Relations required?

2. **Consistency Needs**
   - ACID required? → SQL
   - Eventual OK? → NoSQL

3. **Scale**
   - Single node? → PostgreSQL/MySQL
   - Distributed? → CockroachDB, MongoDB

4. **Latency**
   - Sub-ms? → Redis
   - ms OK? → PostgreSQL

---

## Schema Design Principles

### Normalization

| Level | Description | Use When |
|-------|-------------|----------|
| 1NF | Atomic values | Always |
| 2NF | No partial dependencies | With composite keys |
| 3NF | No transitive dependencies | Most cases |
| BCNF | Boyce-Codd | When strict |

### Denormalization

When to denormalize:
- Read-heavy workloads
- Performance-critical queries
- Reporting/analytics

### Common Patterns

```sql
-- Star schema for analytics
CREATE TABLE fact_orders (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    product_id BIGINT REFERENCES products(id),
    store_id BIGINT REFERENCES stores(id),
    quantity INT,
    total_amount DECIMAL(10,2),
    created_at TIMESTAMP
);

-- Dimension tables
CREATE TABLE dim_users (...);
CREATE TABLE dim_products (...);
CREATE TABLE dim_stores (...);
```

---

## SQL Best Practices

### Indexing

```sql
-- Composite index for common query
CREATE INDEX idx_orders_user_date
ON orders(user_id, created_at DESC);

-- Partial index
CREATE INDEX idx_active_users
ON users(id) WHERE status = 'active';

-- Avoid over-indexing
-- Each index slows writes
```

### Query Patterns

```sql
-- Good: Use EXISTS instead of IN
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- Avoid: SELECT *
SELECT id, name, email FROM users;

-- Good: Batch operations
INSERT INTO items (name) VALUES
('a'), ('b'), ('c');

-- Pagination
SELECT * FROM users
ORDER BY id LIMIT 10 OFFSET 100;
```

---

## NoSQL Patterns

### Document (MongoDB)

```javascript
// Embedding vs Referencing
// Embed: One-to-few
{
  name: "Order",
  items: [
    { product: "Book", qty: 2 },
    { product: "Pen", qty: 1 }
  ]
}

// Reference: One-to-many
{
  user_id: 123,
  orders: [1, 2, 3]  // Array of IDs
}
```

### Key-Value (Redis)

```rust
// Use for:
// - Session cache
// - Rate limiting
// - Leaderboards

// Example: Rate limiter
async fn check_rate_limit(key: &str, limit: u32, window: Duration) -> bool {
    let count = redis.incr(key).await?;
    if count == 1 {
        redis.expire(key, window).await?;
    }
    Ok(count <= limit)
}
```

---

## Data Modeling Checklist

- [ ] Data model selected
- [ ] Normalization level decided
- [ ] Indexes planned
- [ ] Query patterns analyzed
- [ ] Migration strategy defined
- [ ] Backup strategy defined
- [ ] Performance requirements set
