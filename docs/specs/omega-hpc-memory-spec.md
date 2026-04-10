# Omega HPC Memory Technical Specification

## Overview

omega-hpc-memory 是 omega-hpc 的持久化记忆层，提供基于 Rust 的高性能存储和检索能力。

## File Format: .omega

### Version: 1.0

```
Offset  Size    Field           Description
------  ----    -----           -----------
0       8       magic           "OMEGA001"
8       2       version         0x0001
10      4       metadata_len    metadata JSON length
14      N       metadata        JSON (见下)
14+N    M       documents       Document storage
...     K       bm25_index      BM25 index data
...     L       vectors         Vector store (HNSW-lite)
...     P       entities        Entity slot index
...     32      checksum        SHA-256 of all above
```

### Metadata Schema

```json
{
  "version": "1.0",
  "created_at": "2026-04-10T00:00:00Z",
  "updated_at": "2026-04-10T00:00:00Z",
  "doc_count": 100,
  "chunk_count": 1500,
  "embedding_model": "sentence-transformers/all-MiniLM-L6-v2",
  "vector_dim": 384
}
```

## Core Components

### 1. Document Store

```rust
struct Document {
    id: u64,
    path: String,
    mime_type: String,
    chunks: Vec<Chunk>,
    metadata: HashMap<String, String>,
}

struct Chunk {
    id: u64,
    content: String,
    start_line: u32,
    end_line: u32,
}
```

### 2. BM25 Index

- Indexer: `tantivy` (pure Rust)
- Parameters: `k1=1.5, b=0.75`
- Field: content (tokenized, lowercased)

### 3. Vector Store

- Pure Rust implementation
- Flat index with quantization
- Default dim: 384 (configurable)
- Similarity: Cosine

### 4. Entity Extractor

```rust
struct EntitySlot {
    name: String,
    entity_type: String,  // "person", "concept", "technology"
    values: HashMap<String, String>,
}
```

## RCLI Commands

### init

```bash
omega-hpc init [--path <.omega_path>]
```

创建新的记忆库文件。

### add

```bash
omega-hpc add <file_or_directory> [--recursive] [--glob <pattern>]
```

添加文件或目录到记忆库。

### search

```bash
omega-hpc search <query> [--limit <n>] [--mode <hybrid|bm25|vector>]
```

混合语义搜索。

### find

```bash
omega-hpc find --exact <pattern> [--case-sensitive]
omega-hpc find --regex <pattern>
```

精确匹配搜索。

### stat

```bash
omega-hpc stat
```

显示记忆库统计信息。

### export

```bash
omega-hpc export [--format <json|markdown>] [--output <path>]
```

导出记忆内容。

## Embedding Integration

### Local (Default)

```toml
[memory]
embedding = "local"
model = "sentence-transformers/all-MiniLM-L6-v2"
```

### Remote API

```toml
[memory]
embedding = "openai"
model = "text-embedding-3-small"
api_key = "${OPENAI_API_KEY}"
```

## Evaluation Framework

### Benchmark Integration

#### 1. LoCoMo Benchmark Protocol

参考 [Mem0 Evaluation](https://github.com/mem0ai/mem0/tree/main/evaluation) 实现。

**评测执行**:
```bash
omega-hpc eval --benchmark locomo --dataset ./benchmarks/locomo/
```

**输出格式**:
```json
{
  "benchmark": "locomo",
  "timestamp": "2026-04-10T00:00:00Z",
  "results": {
    "single_hop": { "llm_score": 0.85, "bleu": 0.72, "f1": 0.78 },
    "temporal": { "llm_score": 0.72, "bleu": 0.65, "f1": 0.68 },
    "multi_hop": { "llm_score": 0.68, "bleu": 0.58, "f1": 0.62 },
    "open_domain": { "llm_score": 0.75, "bleu": 0.66, "f1": 0.70 }
  },
  "performance": {
    "token_cost_avg": 1847,
    "p50_latency_ms": 45,
    "p95_latency_ms": 120
  }
}
```

#### 2. Anti-Gaming Measures

**闭卷评测约束**:
```rust
struct EvalConfig {
    // 评测数据集不得包含在索引构建中
    exclude_eval_from_index: true,
    
    // 使用随机采样防止题目泄露
    random_seed: Option<u64>,
    
    // 评测时禁用特定优化
    disable_shortcut_optimizations: true,
}
```

**防止作弊机制**:
1. 评测数据集独立存储，使用时加载
2. 题目在评测前对系统保持隐藏
3. 不允许针对特定题目添加人工索引
4. 支持第三方独立评测接口

#### 3. Evaluation Commands

```bash
# 运行 LoCoMo 评测
omega-hpc eval --benchmark locomo

# 运行 Omega HPC 场景评测
omega-hpc eval --benchmark omega-hpc

# 运行鲁棒性测试
omega-hpc eval --benchmark robustness

# 生成评测报告
omega-hpc eval --report --format html

# 对比评测 (vs 基线)
omega-hpc eval --compare --baseline mem0
```

### Evaluation Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| LLM-as-Judge | LLM 评判答案正确性 (0-1) | > 0.65 |
| BLEU Score | n-gram 相似度 | > 0.60 |
| F1 Score | 精确率/召回率调和平均 | > 0.65 |
| Token Cost | 单次查询 Token 消耗 | < 2K |
| P50 Latency | 中位数延迟 | < 50ms |
| P95 Latency | 95分位延迟 | < 200ms |

## Performance Targets

| Metric | Target |
|--------|--------|
| Index speed | 10,000 chunks/sec |
| Search latency (p50) | < 50ms |
| Search latency (p99) | < 200ms |
| Memory usage | < 500MB for 1M chunks |
| File size | ~1KB per chunk |

## Error Handling

| Error | Code | Handling |
|-------|------|----------|
| Invalid magic | E0001 | 拒绝打开，显示帮助 |
| Corrupted index | E0002 | 提供修复模式 |
| Out of memory | E0003 | 优雅降级，分批处理 |
| Eval dataset not found | E0101 | 提示下载或检查路径 |
| Eval dimension mismatch | E0102 | 配置检查 |

## Testing Strategy

- Unit tests for each module
- Integration tests for CLI
- Benchmark suite for search quality
- Fuzz testing for parser
- Evaluation framework validation against LoCoMo
