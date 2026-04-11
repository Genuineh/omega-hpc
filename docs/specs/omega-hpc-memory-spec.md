# Omega HPC Memory Technical Specification

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

目录结构（按文档拆分，避免单文件膨胀）：

```
cortex/
├── meta.toml              # 全局元数据
├── docs/
│   ├── doc_001.toml       # 单文档索引
│   └── doc_002.toml
├── bm25/                  # tantivy 索引目录
│   └── ...                # tantivy 自管理
├── entities.toml          # 实体槽位
└── embeddings/
    └── shard_*.bin
```

```toml
# meta.toml
version = "1.0"
created_at = "2026-04-10T00:00:00Z"
updated_at = "2026-04-10T00:00:00Z"
total_docs = 42
total_chunks = 1247
total_entities = 156
```

```toml
# docs/doc_001.toml
id = "doc_001"
path = "docs/TODO.md"
mime_type = "text/markdown"
content_hash = "sha256:abc123..."
chunk_count = 2
status = "active"
chunks = ["chunk_001", "chunk_002"]
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

Mem 层使用独立的 BM25 索引（轻量 tantivy index）实现 recall 查询：

**recall 实现机制**：
1. 每次 `remember` 写入时，同时将内容索引到 Mem 层的 BM25
2. `recall` 查询先经过 BM25 检索候选记忆
3. Phase 2 后可为 Mem 层添加向量索引，提升语义匹配

```rust
pub struct MemoryStore {
    mem_path: PathBuf,
    bm25_index: tantivy::Index,  // Mem 层独立 BM25 索引
}

impl MemoryStore {
    pub fn save_fact(&mut self, fact: &Fact) -> Result<()>;
    pub fn save_decision(&mut self, decision: &Decision) -> Result<()>;
    pub fn recall(&self, query: &str) -> Result<Vec<MemoryHit>>;
    pub fn start_session(&self, agent: &str, user: &str) -> Result<Session>;
    pub fn get_session(&self, session_id: &str) -> Result<Session>;
}
```

### 4. extract_facts_from_conversation 行为契约

**输入**：`Vec<Message>` - 对话消息列表（role + content）

**输出**：`ExtractedMemories` - 提取的结构化记忆

```rust
pub struct ExtractedMemories {
    pub facts: Vec<ExtractedFact>,
    pub decisions: Vec<ExtractedDecision>,
}

pub struct ExtractedFact {
    pub content: String,
    pub confidence: f32,
}

pub struct ExtractedDecision {
    pub content: String,
    pub context: String,
}
```

**LLM 配置**：
- 使用配置中的 LLM 提供者（复用 embedding 或独立配置）
- 内置提取 prompt 模板，支持自定义覆盖
- 提取结果返回给调用方，不自动写入（调用方需显式 `save_fact` / `save_decision`）

**离线模式**：
- 远程 LLM 不可用时，可降级到基于规则的关键句提取（质量降低但可用）

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

### gc

```bash
omega-hpc gc [OPTIONS]
```

垃圾回收，清理 KB 层中未被 Cortex 索引引用的孤立 chunk 文件。

Options:
- `--dry-run` 仅报告可回收空间，不执行删除
- `--all` 同时清理 Cortex 中标记为 stale 的索引条目

**回收策略**：
1. 扫描 `.hpc/kb/` 中所有 chunk 文件
2. 交叉比对 Cortex 中 `content_hash` 引用
3. 无引用的 chunk 标记为可回收
4. 删除可回收 chunk 并报告释放空间

**安全保证**：
- 默认 dry-run 模式，需显式去掉 `--dry-run` 才执行删除
- 被活跃索引引用的 chunk 不会被误删
- 删除前记录日志到 `.hpc/sync/manifest.toml`

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

### Dual Mode: Remote API + Local CPU Fallback

```toml
[embedding]
provider = "openai"           # openai | cohere | local
model = "text-embedding-3-small"
api_key = "${OPENAI_API_KEY}"

[embedding.local]
model = "sentence-transformers/all-MiniLM-L6-v2"
dim = 384
cache_dir = "~/.cache/omega-hpc/models"
```

**Fallback 策略**：
1. 优先使用配置的远程 provider
2. 远程不可用时（网络错误/API 限流）自动 fallback 到 local
3. 可通过 `--provider local` 强制使用本地模式

### Local Mode Details

使用 [Candle](https://github.com/huggingface/candle) (HuggingFace 纯 Rust ML 框架)：

| 属性 | 值 |
|------|-----|
| 默认模型 | all-MiniLM-L6-v2 |
| 向量维度 | 384 |
| 模型大小 | ~80MB |
| 依赖 | 纯 Rust，无 CUDA/MKL/ONNX |
| 首次运行 | 下载模型到 cache_dir |
| CPU 性能 | ~50 embeddings/sec |

### Supported Providers

| Provider | Status | Models | Dim |
|----------|--------|--------|-----|
| OpenAI | Phase 2 | text-embedding-3-small/large | 1536/3072 |
| Cohere | Phase 2 | embed-english-v3.0 | 1024 |
| Local (Candle) | Phase 2 | all-MiniLM-L6-v2 | 384 |

**已知限制**：
- 本地模型 (384 dim) vs 远程 (1536 dim) 语义质量有差距
- 切换 provider 导致向量维度变化，需 `omega-hpc rebuild --full`

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

### LoCoMo Benchmark (对话记忆)

来源: [Mem0 论文](https://arxiv.org/abs/2504.19413)，评测多轮多会话对话中的记忆检索能力。

**四类问题**:
| 类别 | 描述 |
|------|------|
| Single-hop | 单点事实查询 |
| Temporal | 时间相关查询 |
| Multi-hop | 多跳推理查询 |
| Open-domain | 开放域问答 |

**评测指标**:
| 指标 | 说明 |
|------|------|
| LLM-as-Judge | LLM 评判答案正确性 (0-1) |
| BLEU | n-gram 相似度 |
| F1 | 精确率/召回率调和平均 |
| Token Cost | 单次查询 Token 消耗 |
| P50/P95 Latency | 检索延迟 |

**参考基线**:
| 系统 | LLM Score | Token | P95 延迟 |
|------|-----------|-------|----------|
| Full-context | ~72.9% | ~26K | ~17.12s |
| OpenAI Memory | ~52.9% | ~8K | ~4.2s |
| Mem0 | ~66.9% | ~1.8K | ~1.44s |
| Mem0+Graph | ~68.4% | ~2.5K | ~2.59s |

```bash
omega-hpc eval --benchmark locomo --dataset ./benchmarks/locomo/
```

### Knowledge Retrieval Benchmark

```bash
omega-hpc eval --benchmark knowledge --dataset ./benchmarks/knowledge/
```

**指标**: Recall@K, MRR, P50/P95 Latency

### Memory Recall Benchmark

```bash
omega-hpc eval --benchmark memory --dataset ./benchmarks/memory/
```

**指标**: Memory Recall, Memory Precision

### Anti-Gaming Measures

| 规则 | 说明 |
|------|------|
| 闭卷索引 | 评测数据集不参与索引构建 |
| 独立题库 | 题库与代码库分离，定期轮换 |
| 无捷径优化 | 禁止针对评测题目添加人工规则 |
| 不可预测性 | 评测题目在系统设计时未知 |
| 可复现性 | 评测结果可被独立复现 |

### eval 命令

```bash
omega-hpc eval [OPTIONS]
```

Options:
- `--benchmark <NAME>` 基准: `locomo`, `knowledge`, `memory`
- `--dataset <PATH>` 数据集路径
- `--output <PATH>` 输出文件
- `--format <FMT>` 格式: `json`, `table`, `html`
