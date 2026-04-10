---
name: backend-rust
description: Rust + Axum Backend Development Guide. Provides guidance for building REST APIs with Rust and Axum framework.
version: 1.0.0
---

# Rust + Axum Guide

Guide for building backend services with Rust and Axum.

---

## Project Setup

### Dependencies

```toml
[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tower = "0.4"
tower-http = { version = "0.5", features = ["trace", "cors"] }
tracing = "0.1"
tracing-subscriber = "0.3"
```

---

## Basic Structure

```rust
use axum::{
    routing::{get, post},
    Router,
    extract::State,
    http::StatusCode,
    response::Json,
};
use serde::{Deserialize, Serialize};
use std::sync::Arc;

// State shared across handlers
#[derive(Clone)]
struct AppState {
    db: Database,
}

#[derive(Serialize, Deserialize)]
struct User {
    id: i64,
    name: String,
    email: String,
}

// Handler
async fn get_user(
    State(state): State<Arc<AppState>>,
    Path(user_id): Path<i64>,
) -> Result<Json<User>, StatusCode> {
    state.db.find_user(user_id)
        .await
        .map(Json)
        .map_err(|_| StatusCode::NOT_FOUND)
}

// Router
let app = Router::new()
    .route("/users/:id", get(get_user))
    .route("/users", post(create_user))
    .with_state(Arc::new(AppState { db }));
```

---

## Request Handling

### Path Parameters

```rust
async fn get_user(Path(user_id): Path<i64>) -> Result<Json<User>, AppError> {
    // user_id extracted from path
}
```

### Query Parameters

```rust
async fn list_users(
    Query(params): Query<ListParams>,
) -> Result<Json<Vec<User>>, AppError> {
    // params.name, params.page, params.limit
}

#[derive(Deserialize)]
struct ListParams {
    name: Option<String>,
    page: Option<usize>,
    limit: Option<usize>,
}
```

### JSON Body

```rust
async fn create_user(
    Json(payload): Json<CreateUserRequest>,
) -> Result<Json<User>, AppError> {
    // payload.name, payload.email
}
```

### Form Data

```rust
async fn upload(
    Multipart(multipart): Multipart,
) -> Result<Json<UploadResponse>, AppError> {
    while let Some(field) = multipart.next_field().await? {
        let name = field.name().unwrap();
        let data = field.bytes().await?;
        // Process field
    }
    Ok(Json(UploadResponse { url: "...".to_string() }))
}
```

---

## Error Handling

### Custom Error Type

```rust
use axum::{
    response::IntoResponse,
    http::StatusCode,
};

#[derive(Debug)]
enum AppError {
    NotFound(String),
    BadRequest(String),
    Internal(String),
}

impl IntoResponse for AppError {
    fn into_response(self) -> axum::response::Response {
        let (status, message) = match self {
            AppError::NotFound(msg) => (StatusCode::NOT_FOUND, msg),
            AppError::BadRequest(msg) => (StatusCode::BAD_REQUEST, msg),
            AppError::Internal(msg) => (StatusCode::INTERNAL_SERVER_ERROR, msg),
        };
        (status, Json(serde_json::json!({ "error": message }))).into_response()
    }
}
```

---

## Middleware

### Logging

```rust
use tower_http::trace::TraceLayer;

let app = Router::new()
    .layer(TraceLayer::new_for_http());
```

### CORS

```rust
use tower_http::cors::{CorsLayer, Any};

let cors = CorsLayer::new()
    .allow_origin(Any)
    .allow_methods(Any)
    .allow_headers(Any);

let app = Router::new()
    .layer(cors);
```

### Rate Limiting

```rust
use tower::limit::RateLimitLayer;
use std::time::Duration;

let rate_limit = RateLimitLayer::new(
    100,
    Duration::from_secs(60),
);

let app = Router::new()
    .layer(rate_limit);
```

---

## State Management

### With Database

```rust
use sqlx::PgPool;

#[derive(Clone)]
struct AppState {
    db: PgPool,
}

async fn handler(State(state): State<Arc<AppState>>) {
    let result = sqlx::query_as::<_, User>("SELECT * FROM users")
        .fetch_all(&state.db)
        .await;
}
```

### With Redis

```rust
use redis::aio::RedisConnectionManager;

#[derive(Clone)]
struct AppState {
    redis: redis::aio::Pool<RedisConnectionManager>,
}
```

---

## Best Practices

1. **Use State**: Pass shared resources via State
2. **Async All**: Keep handlers async
3. **Error Types**: Implement custom error types
4. **Validation**: Use serde validation
5. **Tracing**: Add tracing middleware
6. **Timeouts**: Configure timeouts

```rust
// Timeout example
use std::time::Duration;
use tower::timeout::TimeoutLayer;

let app = Router::new()
    .layer(TimeoutLayer::new(Duration::from_secs(30)));
```

---

## Checklist

- [ ] Project structure defined
- [ ] State management configured
- [ ] Error handling implemented
- [ ] Middleware added (logging, cors)
- [ ] Validation added
- [ ] Tests written
