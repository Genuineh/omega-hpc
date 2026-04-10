# Omega HPC Memory Technical Specification

**版本**: v2  
**状态**: Active

---

## Overview

omega-hpc-memory 是统一记忆与知识库系统，同时服务于：
- Agent 长期记忆（跨会话保留决策、偏好）
- 外部知识库（项目文档的索引与检索）

**核心理念：索引即记忆**。系统保存结构化索引，内容按需加载。

**集成方式**：仅 CLI 和 SDK，不提供 MCP。

---

## Architecture

### Directory Structure

```
.hpc/                          # 项目记忆根目录
├── cortex/                    # 元认知层（索引）
│   ├── index.toml             # 主索引清单
│   ├── bm25.bin               # BM25 结构
│   ├── entities.toml          # 实体槽位
│   └── embeddings/            # 向量分片
│       └── shard_*.bin
│
├── kb/                        # 知识库层（内容）
│   └── {sha256}.chunk        # 内容分片
│
├── mem/                       # 记忆层
│   ├── sessions/
│   │   └── {session-id}.mem
│   └── users/
│       └── {user-id}.mem
│
└── sync/
    └── manifest.toml
```

### Layer Boundaries

#### Cortex Layer (`.hpc/cortex/`)

**存储**：索引数据，不存储原始内容

```toml
# index.toml
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
content_hash = "sha256:abc123..."  # 指向 KB 层
chunk_ids = ["chunk_001", "chunk_002"]
chunk_count = 2
status = "active"  # active | stale

[[entities]]
name = "omega-hpc"
type = "project"
slots = { description = "AI Agent 操作平台" }
related_chunks = ["chunk_010"]
```

#### Knowledge Layer (`.hpc/kb/`)

**存储**：Content-addressable 内容分片

```
+------------------+
| Header           |  Magic (4B) + Hash (32B) + Size (4B)
+------------------+
| Content          |  原始文本/压缩内容
+------------------+
| Checksum         |  SHA-256
+------------------+
```

**规则**：
- Hash 不变则内容不变
- 外部修改 → 新 hash → 新 chunk
- 旧 chunk 保留直至 gc

#### Memory Layer (`.hpc/mem/`)

**存储**：结构化 Agent 记忆

```toml
# {session-id}.mem
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
source = "explicit"  # explicit | conversation

[[decisions]]
id = "dec_001"
content = "选择 Tantivy 作为 BM25 引擎因为纯 Rust 实现"
context = "需要避免外部依赖"
decided_at = "2026-04-10T11:00:00Z"
revised = false

[[references]]
doc_id = "doc_001"  # 可引用 KB 中的文档
relevance = 0.85
last_accessed = "2026-04-10T11:30:00Z"
```

**Mem vs KB 区别**：

| 维度 | KB 层 | Mem 层 |
|------|-------|--------|
| 内容类型 | 文档分片 | 结构化记忆 |
| 来源 | 外部文件 | Agent 写入 |
| 格式 | 二进制 chunk | TOML |
| 查询命令 | `search` | `recall` |

---

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
    pub fn mark_stale(&mut self, doc_id: &str) -> Result<()>;
}
```

### 2. SearchEngine

```rust
pub struct SearchEngine {
    cortex_path: PathBuf,
    kb_path: PathBuf,
    bm25_index: tantivy::Index,
    embedding_client: Box<dyn EmbeddingClient>,
}

impl SearchEngine {
    pub fn search(&self, query: &str, mode: SearchMode) -> Result<Vec<SearchHit>>;
    pub fn search_hybrid(&self, query: &str, limit: usize) -> Result<Vec<SearchHit>>;
    pub fn search_bm25(&self, query: &str, limit: usize) -> Result<Vec<SearchHit>>;
    pub fn search_vector(&self, query: &str, limit: usize) -> Result<Vec<SearchHit>>;
}

pub enum SearchMode {
    Hybrid { alpha: f32 },
    BM25,
    Vector,
}
```

### 3. MemoryStore

```rust
pub struct MemoryStore {
    mem_path: PathBuf,
}

impl MemoryStore {
    pub fn save_fact(&mut self, fact: &Fact) -> Result<()>;
    pub fn save_decision(&mut self, decision: &Decision) -> Result<()>;
    pub fn recall(&self, query: &str) -> Result<Vec<MemoryHit>>;
    pub fn start_session(&self, agent: &str, user: &str) -> Result<Session>;
    pub fn get_session(&self, session_id: &str) -> Result<Session>;
}
```

### 4. EmbeddingClient Trait

```rust
pub trait EmbeddingClient: Send + Sync {
    fn embed(&self, texts: &[&str]) -> Result<Vec<Vec<f32>>>;
}

// 内置实现
pub struct OpenAIEmbedding {
    api_key: String,
    model: String,
}
```

---

## RCLI Commands

### init

```bash
omega-hpc init [OPTIONS] [PATH]
```

初始化项目记忆空间，创建 `.hpc/` 目录结构。

Options:
- `--path <PATH>` 记忆空间路径 (default: `.hpc`)
- `--force` 强制覆盖

### add

```bash
omega-hpc add [OPTIONS] <PATH>
```

添加知识库内容。

Options:
- `--recursive, -r` 递归添加
- `--glob <PATTERN>` 文件过滤
- `--chunk-size <N>` 分块大小 (default: 512)

### search

```bash
omega-hpc search [OPTIONS] <QUERY>
```

混合检索。

Options:
- `--limit, -n <N>` 结果数量 (default: 10)
- `--mode <MODE>` 模式: `hybrid`, `bm25`, `vector`
- `--format <FMT>` 格式: `json`, `table`, `simple`

### recall

```bash
omega-hpc recall [OPTIONS] <QUERY>
```

回忆 Agent 记忆。

Options:
- `--session <ID>` 限定会话
- `--user <ID>` 限定用户
- `--type <TYPE>` 类型: `fact`, `decision`, `all`

### remember

```bash
omega-hpc remember <CONTENT> [OPTIONS]
```

写入记忆。

Options:
- `--type <TYPE>` 类型: `fact`, `decision`
- `--confidence <N>` 置信度 (default: 0.9)
- `--session <ID>` 关联会话
- `--user <ID>` 关联用户

### forget

```bash
omega-hpc forget [OPTIONS]
```

删除记忆或索引。

Options:
- `--doc <ID>` 删除文档索引
- `--fact <ID>` 删除事实
- `--session <ID>` 删除会话
- `--entity <NAME>` 删除实体

### stat

```bash
omega-hpc stat
```

显示统计。

### rebuild

```bash
omega-hpc rebuild [OPTIONS]
```

重建索引。

Options:
- `--cortex` 仅重建 cortex
- `--kb` 仅重建 kb
- `--full` 全部重建

### eval

```bash
omega-hpc eval [OPTIONS]
```

运行评测。

Options:
- `--benchmark <NAME>` 基准: `knowledge`, `memory`
- `--dataset <PATH>` 数据集路径
- `--output <PATH>` 输出文件

---

## Embedding Integration

### Remote API Only (Phase 2)

```toml
[embedding]
provider = "openai"
model = "text-embedding-3-small"
api_key = "${OPENAI_API_KEY}"
```

**明确**：
- 不支持本地 ONNX（ort crate 有 C 库依赖）
- 不支持 Candle/burn（生态不成熟）
- 仅支持远程 API

### Supported Providers

| Provider | Status | Models |
|----------|--------|--------|
| OpenAI | Phase 1 | text-embedding-3-small, text-embedding-3-large |
| Cohere | Phase 2 | embed-english-v3.0, embed-multilingual-v3.0 |
| Local | **不支持** | - |

---

## Configuration

`.hpc.toml`:

```toml
[hpc]
cortex_path = ".hpc"

[indexing]
chunk_size = 512

[embedding]
provider = "openai"
model = "text-embedding-3-small"
api_key = "${OPENAI_API_KEY}"

[vector]
store = "flat"  # flat | hnsw (Phase 2)
dim = 1536

[search]
default_limit = 10
hybrid_alpha = 0.7
```

---

## Performance Targets

| Metric | Phase 1 | Phase 2 |
|--------|---------|---------|
| Index speed | 5,000 chunks/sec | 5,000 chunks/sec |
| Search P50 (BM25) | < 20ms | < 20ms |
| Search P50 (vector) | - | < 100ms (网络延迟) |
| Search P95 | < 100ms | < 300ms |
| Memory (1M chunks) | < 500MB | < 500MB |

---

## Error Handling

| Error | Code | Handling |
|-------|------|----------|
| Invalid cortex | E0001 | 引导 `omega-hpc rebuild` |
| Chunk not found | E0002 | 提示重新 `add` |
| KB hash mismatch | E0003 | 标记 chunk 为 stale |
| Embedding API error | E0101 | 返回错误，不阻塞搜索 |
| Out of memory | E0004 | 降级到 BM25 only |

---

## Evaluation

### Knowledge Retrieval Benchmark (Phase 4)

```bash
omega-hpc eval --benchmark knowledge --dataset ./benchmarks/knowledge/
```

**指标**：
- Recall@K
- MRR
- P50/P95 Latency

### Memory Recall Benchmark (Phase 4)

```bash
omega-hpc eval --benchmark memory --dataset ./benchmarks/memory/
```

**指标**：
- Memory Recall
- Memory Precision

**不采用 LoCoMo**：LoCoMo 测试对话记忆自动提取，与本项目手动记忆写入机制不匹配。
