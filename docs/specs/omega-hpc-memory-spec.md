# Omega HPC Memory Technical Specification

## Overview

omega-hpc-memory 是统一记忆与知识库系统，同时服务于 Agent 长期记忆和外部知识库检索。

**核心理念：索引即记忆**。系统保存的是结构化索引（类比大脑皮层/元认知），内容按需加载。

## Architecture

### File Hierarchy

```
.hpc/                        # 项目记忆根目录
├── cortex/                    # 元认知层（索引）
│   ├── index.toml             # 主索引清单
│   ├── bm25.bin               # BM25 结构
│   ├── entities.toml          # 实体槽位
│   └── embeddings/            # 向量分片
│       └── shard_*.vec
│
├── kb/                        # 知识库层（内容）
│   └── {hash}.chunk          # 内容分片
│
├── mem/                       # 记忆层（Agent 私有）
│   ├── sessions/
│   │   └── {session-id}.mem
│   └── users/
│       └── {user-id}.mem
│
└── sync/
    └── manifest.toml          # 变更追踪
```

### Cortex Layer Files

#### index.toml

```toml
version = "1.0"

[meta]
created_at = "2026-04-10T00:00:00Z"
updated_at = "2026-04-10T00:00:00Z"
total_chunks = 1247
total_entities = 156

[[documents]]
id = "doc_001"
path = "docs/TODO.md"
mime_type = "text/markdown"
content_hash = "sha256:abc123..."
chunk_ids = ["chunk_001", "chunk_002"]
chunk_count = 2

[[entities]]
name = "omega-hpc"
type = "project"
slots = { description = "AI Agent 操作平台" }
related_chunks = ["chunk_010"]
```

#### bm25.bin

BM25 索引结构，使用 `tantivy` 生成：
- 可从 index.toml 重建
- 二进制格式，高效压缩

#### entities.toml

```toml
[[slots]]
name = "PostgreSQL"
type = "technology"
values = { role = "数据库", selected_at = "2026-04-01" }

[[slots]]
name = "Tantivy"
type = "library"
values = { language = "Rust", purpose = "BM25 引擎" }
```

### Knowledge Layer Files

#### {hash}.chunk

Content-addressable 内容分片：

```
+------------------+
| Header           |  Magic (4B) + Hash (32B) + Size (4B)
+------------------+
| Content          |  原始文本/压缩内容
+------------------+
| Checksum         |  SHA-256
+------------------+
```

### Memory Layer Files

#### {session-id}.mem

```toml
version = "1.0"

[session]
id = "sess_20260410_001"
agent = "claude-code"
user = "jerryg"
started_at = "2026-04-10T10:00:00Z"
ended_at = "2026-04-10T12:00:00Z"

[[facts]]
id = "fact_001"
content = "用户偏好 Rust 作为主要开发语言"
extracted_at = "2026-04-10T10:30:00Z"
confidence = 0.95
source = "conversation"

[[decisions]]
id = "dec_001"
content = "选择 Tantivy 作为 BM25 引擎因为纯 Rust 实现"
context = "需要避免外部依赖"
decided_at = "2026-04-10T11:00:00Z"
revised = false

[[references]]
doc_id = "doc_001"
relevance = 0.85
last_accessed = "2026-04-10T11:30:00Z"
```

## Core Components

### 1. Indexer

```rust
pub struct Indexer {
    cortex_path: PathBuf,
    kb_path: PathBuf,
}

impl Indexer {
    pub fn add_document(&mut self, path: &Path) -> Result<DocId>;
    pub fn add_directory(&mut self, path: &Path, glob: &Glob) -> Result<()>;
    pub fn rebuild_index(&mut self) -> Result<()>;
}
```

### 2. Search Engine

```rust
pub struct SearchEngine {
    bm25_index: tantivy::Index,
    vector_store: Box<dyn VectorStore>,
    entity_index: HashMap<String, EntitySlot>,
}

impl SearchEngine {
    pub fn search(&self, query: &str, mode: SearchMode) -> Result<Vec<SearchHit>>;
    pub fn search_hybrid(&self, query: &str, limit: usize) -> Result<Vec<SearchHit>>;
    pub fn search_bm25(&self, query: &str, limit: usize) -> Result<Vec<SearchHit>>;
    pub fn search_vector(&self, query: &str, limit: usize) -> Result<Vec<SearchHit>>;
}

pub enum SearchMode {
    Hybrid { alpha: f32 },  // BM25 weight
    BM25,
    Vector,
}
```

### 3. Memory Store

```rust
pub struct MemoryStore {
    mem_path: PathBuf,
}

impl MemoryStore {
    pub fn save_fact(&mut self, fact: &Fact) -> Result<()>;
    pub fn save_decision(&mut self, decision: &Decision) -> Result<()>;
    pub fn recall(&self, query: &str) -> Result<Vec<MemoryHit>>;
    pub fn get_session(&self, session_id: &str) -> Result<Session>;
}
```

### 4. Vector Store Trait

```rust
pub trait VectorStore: Send + Sync {
    fn add(&mut self, id: &str, embedding: &[f32]) -> Result<()>;
    fn search(&self, query: &[f32], k: usize) -> Result<Vec<ScoredId>>;
    fn save(&self, path: &Path) -> Result<()>;
    fn load(&mut self, path: &Path) -> Result<()>;
}

// 内置实现
pub struct FlatVectorStore;
pub struct HNSWVectorStore;
```

## RCLI Commands

### init

```bash
omega-hpc init [OPTIONS] [PATH]
```

初始化项目记忆空间，创建 `.hpc/` 目录结构。

Options:
- `--path <PATH>` 记忆空间根目录 (default: `.hpc`)
- `--force` 强制覆盖

### add

```bash
omega-hpc add [OPTIONS] <PATH>
```

添加知识库内容到索引。

Options:
- `--recursive, -r` 递归添加目录
- `--glob <PATTERN>` 文件过滤
- `--chunk-size <N>` 分块大小 (default: 512)
- `--no-content` 仅添加索引，不存储内容

### search

```bash
omega-hpc search [OPTIONS] <QUERY>
```

混合检索。

Options:
- `--limit, -n <N>` 结果数量 (default: 10)
- `--mode <MODE>` 搜索模式: `hybrid`, `bm25`, `vector`
- `--format <FMT>` 输出格式: `json`, `table`, `simple`

### recall

```bash
omega-hpc recall [OPTIONS] <QUERY>
```

回忆相关 Agent 记忆。

Options:
- `--session <ID>` 限定会话
- `--user <ID>` 限定用户
- `--type <TYPE>` 记忆类型: `fact`, `decision`, `all`

### find

```bash
omega-hpc find --exact <PATTERN>
omega-hpc find --regex <PATTERN>
```

精确匹配搜索。

### forget

```bash
omega-hpc forget [OPTIONS]
```

删除记忆或索引。

Options:
- `--doc <ID>` 删除文档索引
- `--fact <ID>` 删除事实
- `--session <ID>` 删除会话记忆
- `--entity <NAME>` 删除实体

### stat

```bash
omega-hpc stat
```

显示记忆空间统计。

### eval

```bash
omega-hpc eval [OPTIONS]
```

运行评测。

Options:
- `--benchmark <NAME>` 基准: `locomo`, `knowledge`, `memory`
- `--dataset <PATH>` 数据集路径
- `--output <PATH>` 输出路径

## Embedding Integration

### Local (Default - ONNX)

```toml
[embedding]
provider = "onnx"
model = "sentence-transformers/all-MiniLM-L6-v2"
```

使用 `mistral.rs` 或 ` candle` 加载 ONNX 模型，纯 Rust 无外部依赖。

### Remote API

```toml
[embedding]
provider = "openai"
model = "text-embedding-3-small"
api_key = "${OPENAI_API_KEY}"
```

### Vector Store Configuration

```toml
[vector]
store = "hnsw"           # flat | hnsw
dim = 384
hnsw_m = 16
hnsw_ef = 128
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Index speed | 5,000 chunks/sec |
| Search P50 | < 30ms (index only) |
| Search P95 | < 150ms (with content) |
| Memory (1M chunks) | < 1GB |
| Chunk size | ~1KB avg |

## Error Handling

| Error | Code | Handling |
|-------|------|----------|
| Invalid cortex | E0001 | 引导重建 |
| Chunk not found | E0002 | 提示重新添加 |
| Corrupted index | E0003 | 提供修复命令 |
| Out of memory | E0004 | 降级到 flat 搜索 |

## Evaluation

### Benchmark Integration

```bash
# Knowledge Retrieval Benchmark
omega-hpc eval --benchmark knowledge --dataset ./benchmarks/knowledge/

# Memory Recall Benchmark  
omega-hpc eval --benchmark memory --dataset ./benchmarks/memory/

# Combined
omega-hpc eval --benchmark locomo
```

### Metrics

| Metric | Description |
|--------|-------------|
| Recall@K | Relevant docs in top K |
| MRR | Mean Reciprocal Rank |
| Memory Precision | Agent memory accuracy |
| Latency P50/P95 | Search latency percentiles |
