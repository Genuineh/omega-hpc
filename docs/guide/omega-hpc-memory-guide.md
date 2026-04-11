# Omega HPC Memory CLI Guide

---

## Overview

omega-hpc-memory 是统一记忆与知识库系统，服务于：
- **Agent 长期记忆** - 跨会话保留决策、偏好
- **外部知识库** - 项目文档的索引与检索

**核心理念：索引即记忆**。保存索引结构，内容按需加载。

**集成方式**：仅 CLI 和 SDK，不提供 MCP。

---

## Installation

```bash
cargo install --path crates/omega-hpc-memory-cli
cargo run --bin omega-hpc -- --help
```

---

## Quick Start

```bash
# 1. 初始化项目记忆空间
omega-hpc init

# 2. 添加知识库内容
omega-hpc add ./docs/ --recursive
omega-hpc add . --glob "*.md"

# 3. 搜索知识库
omega-hpc search "omega-hpc 的设计原则"

# 4. 写入记忆
omega-hpc remember "用户偏好 Rust" --type fact
omega-hpc remember "选择了 Tantivy 因为零外部依赖" --type decision

# 5. 回忆记忆
omega-hpc recall "上次为什么选了 Tantivy"

# 6. 查看统计
omega-hpc stat
```

---

## Memory Space Structure

```
.hpc/
├── cortex/            # 元认知索引
│   ├── index.toml
│   ├── bm25.bin
│   └── embeddings/
├── kb/               # 知识库内容
│   └── *.chunk
├── mem/              # Agent 记忆
│   ├── sessions/
│   └── users/
└── sync/
    └── manifest.toml
```

---

## Commands Reference

### init

```bash
omega-hpc init [OPTIONS] [PATH]
```

初始化项目记忆空间。

Options:
- `--path <PATH>` 路径 (default: `.hpc`)
- `--force` 强制覆盖

```bash
omega-hpc init
omega-hpc init --path /path/to/project/.hpc
```

### add

```bash
omega-hpc add [OPTIONS] <PATH>
```

添加知识库内容。

Options:
- `--recursive, -r` 递归
- `--glob <PATTERN>` 文件过滤
- `--chunk-size <N>` 分块大小 (default: 512)

```bash
omega-hpc add ./docs/ --recursive
omega-hpc add . --glob "*.md" --glob "*.txt"
```

### search

```bash
omega-hpc search [OPTIONS] <QUERY>
```

混合检索知识库。

Options:
- `--limit, -n <N>` 结果数量 (default: 10)
- `--mode <MODE>` 模式: `hybrid`, `bm25`, `vector`
- `--format <FMT>` 格式: `json`, `table`, `simple`

```bash
omega-hpc search "持久化记忆系统设计"
omega-hpc search "Rust" --mode bm25
omega-hpc search "架构设计" --mode vector
```

### recall

```bash
omega-hpc recall [OPTIONS] <QUERY>
```

回忆 Agent 记忆。

Options:
- `--session <ID>` 限定会话
- `--user <ID>` 限定用户
- `--type <TYPE>` 类型: `fact`, `decision`, `all`

```bash
omega-hpc recall "用户偏好"
omega-hpc recall "架构决策" --type decision
```

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

```bash
omega-hpc remember "用户偏好 Rust" --type fact
omega-hpc remember "选择了 Tantivy" --type decision --confidence 0.95
```

### find

```bash
omega-hpc find --exact <PATTERN>
omega-hpc find --regex <PATTERN>
```

精确匹配。

Options:
- `--case-sensitive` 区分大小写
- `--context <N>` 上下文行数

```bash
omega-hpc find --exact "omega-hpc"
omega-hpc find --regex "omega-.*-prd"
```

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

```bash
omega-hpc forget --fact fact_001
omega-hpc forget --session sess_20260410_001
```

### stat

```bash
omega-hpc stat
```

显示统计。

```
Omega HPC Memory Statistics
===========================
Cortex:
  Documents: 42
  Chunks: 1,247
  Entities: 156
  Index Size: 2.4 MB

Knowledge Base:
  Content Chunks: 1,247
  Total Size: 4.8 MB

Memory:
  Sessions: 12
  Facts: 89
  Decisions: 34
```

### rebuild

```bash
omega-hpc rebuild [OPTIONS]
```

重建索引。

Options:
- `--cortex` 仅重建 cortex
- `--kb` 仅重建 kb
- `--full` 全部重建

```bash
omega-hpc rebuild --full
```

### eval

```bash
omega-hpc eval [OPTIONS]
```

运行评测。

Options:
- `--benchmark <NAME>` 基准: `knowledge`, `memory`
- `--dataset <PATH>` 数据集路径
- `--output <PATH>` 输出文件
- `--format <FMT>` 格式: `json`, `table`

```bash
omega-hpc eval --benchmark knowledge
omega-hpc eval --benchmark memory --output results.json
omega-hpc eval --benchmark locomo --dataset ./benchmarks/locomo/
```

### LoCoMo Benchmark

测试多轮多会话对话中的记忆检索能力。

**四类问题**:
| 类别 | 示例 |
|------|------|
| Single-hop | "用户的职位是什么？" |
| Temporal | "用户上周说想去哪旅行？" |
| Multi-hop | "用户的经理偏好什么工作方式？" |
| Open-domain | "总结用户的职业背景" |

**评测指标**: LLM-as-Judge, BLEU, F1, Token 消耗, P50/P95 延迟

**参考基线** (来自 [Mem0 论文](https://arxiv.org/abs/2504.19413)):
| 系统 | LLM Score | Token | P95 延迟 |
|------|-----------|-------|----------|
| Full-context | ~72.9% | ~26K | ~17.12s |
| Mem0 | ~66.9% | ~1.8K | ~1.44s |
| Mem0+Graph | ~68.4% | ~2.5K | ~2.59s |

---

## SDK Usage

### Rust SDK

```rust
use omega_hpc_memory::{MemoryStore, SearchEngine, Indexer};

fn main() -> Result<()> {
    // 初始化
    let mut indexer = Indexer::new(".hpc")?;
    let search = SearchEngine::new(".hpc")?;
    let mut mem = MemoryStore::new(".hpc/mem")?;

    // 添加文档
    indexer.add_directory("./docs/", &glob("*.md"))?;

    // 搜索
    let results = search.search("架构设计", SearchMode::Hybrid)?;
    for hit in results {
        println!("{:?}", hit);
    }

    // 写入记忆
    mem.save_fact(Fact {
        content: "用户偏好 Rust".into(),
        confidence: 0.95,
    })?;

    // 回忆记忆
    let memories = mem.recall("用户偏好")?;
    for mem in memories {
        println!("{:?}", mem);
    }

    Ok(())
}
```

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
store = "flat"
dim = 1536

[search]
default_limit = 10
hybrid_alpha = 0.7
```

---

## Tips

### 性能

1. **批量添加**: 多次 `add` 增量索引
2. **BM25 vs Vector**: 关键词用 BM25，语义用 Vector
3. **网络依赖**: Vector 搜索依赖远程 API

### 调试

1. `omega-hpc stat` 查看状态
2. `omega-hpc rebuild --full` 重建索引
3. `--format json` 获取完整数据

### 记忆管理

1. **显式写入**: `remember` 命令需要显式调用
2. **会话关联**: 用 `--session` 关联记忆到会话
3. **手动清理**: `forget` 命令删除过期记忆

---

## Troubleshooting

### 搜索无结果

1. `omega-hpc stat` 确认已添加文档
2. `omega-hpc add ./docs/` 重新添加
3. 尝试 `--mode bm25`

### 索引损坏

```bash
omega-hpc rebuild --full
```

### 向量搜索失败

向量搜索依赖远程 API，检查：
- `OPENAI_API_KEY` 环境变量
- 网络连接

---

## Memory Layer vs Knowledge Layer

| 命令 | 作用于 | 用途 |
|------|--------|------|
| `add` | KB 层 | 添加文档到知识库 |
| `search` | KB 层 | 检索知识库文档 |
| `remember` | Mem 层 | 写入 Agent 记忆 |
| `recall` | Mem 层 | 回忆 Agent 记忆 |

**示例**：

```bash
# 添加文档到知识库
omega-hpc add ./docs/

# 搜索知识库（问项目有什么）
omega-hpc search "架构设计原则"

# 写入记忆（记下 Agent 的决策）
omega-hpc remember "选择了 Rust 因为性能"

# 回忆记忆（问 Agent 上次怎么决策的）
omega-hpc recall "为什么选择 Rust"
```
