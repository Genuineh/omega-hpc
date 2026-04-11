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
├── cortex/                    # 元认知索引
│   ├── meta.toml              # 全局元数据
│   ├── docs/                  # 按文档拆分的索引
│   │   └── {doc_id}.toml
│   ├── bm25/                  # tantivy 索引目录
│   ├── entities.toml          # 实体槽位
│   └── embeddings/            # 向量分片
├── kb/                        # 知识库内容
│   └── {sha256}.chunk
├── mem/                       # Agent 记忆
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

### gc

```bash
omega-hpc gc [OPTIONS]
```

垃圾回收，清理 KB 层中未被引用的孤立 chunk 文件。

Options:
- `--dry-run` 仅报告可回收空间，不执行删除（默认行为）
- `--all` 同时清理 Cortex 中 stale 索引条目

```bash
# 查看可回收空间
omega-hpc gc --dry-run

# 执行清理
omega-hpc gc

# 同时清理 stale 索引
omega-hpc gc --all
```

### eval

```bash
omega-hpc eval [OPTIONS]
```

运行评测。

Options:
- `--benchmark <NAME>` 基准: `locomo`, `knowledge`, `memory`
- `--dataset <PATH>` 数据集路径
- `--output <PATH>` 输出文件
- `--format <FMT>` 格式: `json`, `table`, `html`

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
provider = "openai"           # openai | cohere | local
model = "text-embedding-3-small"
api_key = "${OPENAI_API_KEY}"

[embedding.local]
model = "sentence-transformers/all-MiniLM-L6-v2"  # Candle 加载
dim = 384
cache_dir = "~/.cache/omega-hpc/models"

[vector]
store = "flat"
dim = 1536

[search]
default_limit = 10
hybrid_alpha = 0.7
```

### Embedding Modes

**远程 API（默认）**：使用 OpenAI/Cohere 嵌入服务，质量最高，需网络。

**本地 CPU（Candle fallback）**：
- 远程不可用时自动降级
- 使用 `--provider local` 强制本地模式
- 默认模型 all-MiniLM-L6-v2（~80MB，384 dim，~50 embeddings/sec）
- 首次运行下载模型到 `cache_dir`

**模型迁移警告**：切换 provider 导致向量维度变化（如 1536 → 384），必须 `omega-hpc rebuild --full` 重建索引。

### Recall Mechanism

`recall` 命令使用 Mem 层独立的 BM25 索引：
1. `remember` 写入时同时索引到 Mem 层 BM25
2. `recall` 查询通过 BM25 检索候选记忆
3. Phase 2 后可为 Mem 层添加向量索引，提升语义匹配

### Conversation Fact Extraction (SDK)

```rust
use omega_hpc_memory::MemoryStore;

let mem = MemoryStore::new(".hpc/mem")?;
let extracted = mem.extract_facts_from_conversation(&messages)?;
// extracted.facts: Vec<ExtractedFact>
// extracted.decisions: Vec<ExtractedDecision>
// 调用方确认后显式写入
for fact in extracted.facts {
    mem.save_fact(&fact)?;
}
```

---

## Tips

### 性能

1. **批量添加**: 多次 `add` 增量索引
2. **BM25 vs Vector**: 关键词用 BM25，语义用 Vector
3. **离线使用**: `--provider local` 使用 Candle 本地嵌入

### 调试

1. `omega-hpc stat` 查看状态
2. `omega-hpc rebuild --full` 重建索引
3. `--format json` 获取完整数据
4. `omega-hpc gc --dry-run` 查看可回收空间

### 记忆管理

1. **显式写入**: `remember` 命令需要显式调用
2. **会话关联**: 用 `--session` 关联记忆到会话
3. **手动清理**: `forget` 命令删除过期记忆
4. **对话提取**: SDK 提供 `extract_facts_from_conversation`，由调用方决定何时调用

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

向量搜索默认使用远程 API，检查：
- `OPENAI_API_KEY` 环境变量
- 网络连接
- 或使用 `--provider local` 切换到本地 Candle 模式

### 磁盘空间持续增长

```bash
# 查看可回收空间
omega-hpc gc --dry-run

# 执行清理
omega-hpc gc
```

KB 层使用 content-addressable 存储，文件修改后产生新 chunk，旧 chunk 需通过 `gc` 清理。

### 切换嵌入模型后搜索异常

切换 provider（如 openai → local）导致向量维度变化，需重建：
```bash
omega-hpc rebuild --full
```

---

## Known Limitations

| 限制 | 影响 | 缓解措施 |
|------|------|----------|
| 远程 API 需网络 | 向量搜索离线不可用 | Candle 本地 fallback |
| 向量维度不可混用 | 切换模型需重建索引 | `omega-hpc rebuild --full` |
| Mem 层 recall 依赖 BM25 | 语义匹配不如向量 | Phase 2 为 Mem 添加向量索引 |
| KB 无自动 gc | 磁盘可能膨胀 | `omega-hpc gc` 手动清理 |
| tantivy 索引跨版本不兼容 | 升级 tantivy 需重建 | 文档标注版本要求 |
| eval 依赖外部 LLM | 评测需网络和 API key | 可配置本地 LLM |

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
