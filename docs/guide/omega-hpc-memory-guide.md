# Omega HPC Memory CLI Guide

---

## Overview

omega-hpc-memory 是统一记忆与知识库系统，服务于：
- **Agent 长期记忆** - 跨会话保留决策、偏好
- **外部知识库** - 项目文档的索引与检索

**核心理念：索引即记忆**。保存索引结构，内容按需加载。

**认知架构**：系统模拟人类记忆的三层结构——短期记忆（碎片）→ 整理期（巩固）→ 长期记忆 + 认知图谱。记忆不是简单堆放，而是经过组织过程形成认知体系。

**集成方式**：仅 CLI 和 SDK，不提供 MCP。

**可配置数据层**：用户通过 `.hpc.toml` 定义数据层，每层有独立的索引策略和召回行为，支持继承。系统根据层配置提供个性化检索方案。

---

## Installation

```bash
cargo install --path crates/omega-hpc-memory-cli
cargo run --bin omega-hpc -- --help
```

---

## Quick Start

```bash
# 1. 初始化（使用代码项目模板）
omega-hpc init --template code

# 2. 查看层配置
omega-hpc layers

# 3. 添加内容到指定层
omega-hpc add ./src/ --layer rust --recursive
omega-hpc add ./docs/ --layer docs --recursive
omega-hpc add docs/decisions/ --layer decisions

# 4. 搜索（跨层自动融合）
omega-hpc search "为什么选择 Tantivy"

# 5. 搜索指定层
omega-hpc search "fn main" --layer rust

# 6. 写入记忆（进入短期记忆层）
omega-hpc remember "用户偏好 Rust" --type fact --layer memory
omega-hpc remember "选择了 Tantivy 因为无外部依赖" --type decision

# 7. 整理记忆（碎片 → 结构化 + 认知图谱）
omega-hpc organize --layer memory

# 8. 回忆记忆（结合认知图谱推理）
omega-hpc recall "上次为什么选了 Tantivy"

# 9. 查询认知图谱（决策链、概念关系）
omega-hpc graph --query "Tantivy"

# 10. 查看统计
omega-hpc stat
```

---

## Memory Space Structure

```
.hpc/
├── cortex/                    # 元认知索引
│   ├── meta.toml              # 全局元数据
│   ├── layers/                # 按层拆分的索引
│   │   ├── rust/
│   │   │   ├── index.toml     # 层级文档清单
│   │   │   ├── docs/          # 按文档拆分的索引
│   │   │   └── bm25/          # tantivy 索引
│   │   ├── docs/
│   │   ├── decisions/
│   │   └── memory/
│   ├── entities.toml          # 实体槽位
│   └── embeddings/            # 向量分片（跨层共享）
│
├── layers/                    # 用户定义的数据层
│   ├── rust/                  # content 层：chunk 存储
│   │   └── {sha256}.chunk
│   ├── docs/                  # content 层
│   ├── decisions/             # content 层
│   └── memory/                # memory 层：TOML 记忆
│       ├── sessions/
│       └── users/
│
└── sync/
    └── manifest.toml
```

---

## Layer Configuration

### 什么是 Layer

Layer 是用户可配置的存储与检索单元。每个 Layer 描述一类数据的：
- **索引方式** - 分块大小、策略、glob 模式
- **存储方式** - content（文件分片）或 memory（结构化记忆）
- **召回行为** - 优先级、boost 系数、检索策略

### 层类型

| 类型 | 存储方式 | 写入方式 | 查询方式 |
|------|----------|----------|----------|
| `content` | content-addressable chunk | `add` 命令 | `search` |
| `memory` | TOML 文件 | `remember` / SDK | `recall` |

### 定义层

在 `.hpc.toml` 中添加 `[[layer]]` 段：

```toml
[[layer]]
name = "code"
description = "项目源代码"
type = "content"
priority = 1

[layer.indexing]
chunk_size = 1024
glob = ["*.rs", "*.ts"]
overlap = 64
strategy = "line"
embed = true

[layer.recall]
boost = 1.5
max_results = 5
strategy = "hybrid"
weight_bm25 = 0.6
weight_vector = 0.4
```

### 层继承

子层通过 `extends` 继承父层配置，仅覆盖差异：

```toml
# 父层：抽象模板（abstract = true，不创建存储）
[[layer]]
name = "code"
type = "content"
abstract = true

[layer.indexing]
chunk_size = 1024
overlap = 64
strategy = "line"
embed = true

[layer.recall]
boost = 1.5
max_results = 5
strategy = "hybrid"

# 子层：仅覆盖 glob
[[layer]]
name = "rust"
extends = "code"
priority = 1

[layer.indexing]
glob = ["*.rs"]

[[layer]]
name = "python"
extends = "code"
priority = 1

[layer.indexing]
glob = ["*.py", "*.pyi"]
```

**继承规则**：
- 标量：子覆盖父
- 数组：子替换父
- 表：递归合并
- `abstract = true` 的层仅作模板，不可直接 `add` 或 `search`
- 禁止循环继承

### Init Templates

```bash
# 通用项目（kb + mem）
omega-hpc init --template default

# 代码密集项目（code→rust/python + docs + decisions + memory）
omega-hpc init --template code

# 轻量项目（single content + memory）
omega-hpc init --template minimal
```

**`code` 模板生成的 `.hpc.toml`**：

```toml
[[layer]]
name = "code"
type = "content"
abstract = true

[layer.indexing]
chunk_size = 1024
overlap = 64
strategy = "line"
embed = true

[layer.recall]
boost = 1.5
max_results = 5
strategy = "hybrid"
weight_bm25 = 0.6
weight_vector = 0.4

[[layer]]
name = "rust"
extends = "code"
priority = 1

[layer.indexing]
glob = ["*.rs"]

[[layer]]
name = "python"
extends = "code"
priority = 1

[layer.indexing]
glob = ["*.py", "*.pyi"]

[[layer]]
name = "docs"
type = "content"
priority = 2

[layer.indexing]
chunk_size = 512
glob = ["*.md", "*.txt"]
embed = true

[layer.recall]
boost = 1.0
max_results = 10
strategy = "hybrid"

[[layer]]
name = "decisions"
type = "content"
priority = 0

[layer.indexing]
chunk_size = 256
glob = ["docs/decisions/*.md"]
embed = false

[layer.recall]
boost = 2.0
max_results = 3
strategy = "bm25"

[[layer]]
name = "memory"
type = "memory"
priority = 3

[layer.recall]
boost = 0.8
max_results = 5
strategy = "bm25"
```

### Recall Fusion

跨层搜索时，系统自动合并多层结果：

```
1. 对每个目标层并行搜索
2. 每层结果应用 boost 调分：hit.score *= layer.boost
3. 每层截取至 max_results
4. 合并所有层结果，按 score 降序排列
5. score 相近（差距 < 5%）时按 priority 升序排
6. 截取至全局 limit
```

**示例**：

```
$ omega-hpc search "为什么选择 Tantivy"

Layer decisions (boost=2.0):
  [0] 选择 Tantivy 作为 BM25 引擎 (score: 1.2)

Layer rust (boost=1.5):
  [1] use tantivy::Index; (score: 1.05)

Layer docs (boost=1.0):
  [2] Tantivy 是 Rust 全文搜索引擎 (score: 0.8)
```

---

## Three-Tier Memory Architecture

Memory 层采用三层架构，模拟人类记忆的整理过程：

```
短期记忆(raw/) ──整理期──→ 长期记忆(organized/) ──提取──→ 认知图谱(graph/)
     ↓                                              ↑
  碎片、矛盾、                          概念节点 + 关系边
  低置信度                              支持推理查询
```

### 为什么要整理？

`remember` 写入的是**短期记忆**——快速、不加筛选。可能矛盾、低置信度、冗余。

整理期将碎片转化为**结构化知识**，并从中提取**概念关系图**。

### 短期记忆层（raw/）

```bash
# 写入记忆（进入 raw/，不整理）
omega-hpc remember "用户好像偏好 Rust" --type fact --confidence 0.6
omega-hpc remember "Rust 好像不错" --type observation

# 查看 raw memory 数量
omega-hpc stat --layer memory

# Raw memory 积累后建议：
#   omega-hpc organize --layer memory
```

### 整理期（organize）

```bash
# 整理（碎片 → 结构化 + 图谱）
omega-hpc organize --layer memory

# 仅模拟，不写入
omega-hpc organize --layer memory --dry-run

# 整理并清理 raw
omega-hpc organize --layer memory --delete-raw
```

**整理过程**：
1. LLM 读取所有 raw memories
2. 去重、合并矛盾、提精华
3. 生成 structured facts/decisions
4. 提取概念和关系，更新认知图谱
5. 输出整理报告

### 长期记忆层（organized/）

整理后的结构化记忆，高置信度，有上下文：

```bash
# 回忆（查询 organized/ + graph）
omega-hpc recall "用户偏好什么语言"
omega-hpc recall "为什么选择 Tantivy"
```

### 认知图谱（graph/）

从整理后的记忆中提取的概念关系网络：

```bash
# 搜索概念
omega-hpc graph --query "Rust"

# 查看节点
omega-hpc graph --node concept_tantivy

# 沿关系遍历
omega-hpc graph --traverse decision_002 --relation selects --depth 2
```

**图谱节点类型**：`technology`、`concept`、`decision`、`person`、`rule`

**图谱关系类型**：`selects`、`depends_on`、`implements`、`related_to`、`influenced_by`、`contradicts`

---

## Commands Reference

### init

```bash
omega-hpc init [OPTIONS] [PATH]
```

初始化项目记忆空间。根据模板创建 `.hpc.toml` 和 `.hpc/` 目录。

Options:
- `--path <PATH>` 路径 (default: `.hpc`)
- `--force` 强制覆盖
- `--template <NAME>` 模板: `default`, `code`, `minimal`

```bash
omega-hpc init
omega-hpc init --template code
omega-hpc init --path /path/to/project/.hpc --template minimal
```

### layers

```bash
omega-hpc layers [OPTIONS]
```

列出或验证层配置。

Options:
- （无参数）列出所有层及其配置摘要
- `--verbose` 显示完整配置（含继承解析后的值）
- `--validate` 验证层配置完整性

```bash
$ omega-hpc layers

Layer           Type     Priority  Boost  Strategy  Chunk Size
code            content  -         1.5    hybrid    1024       (abstract)
rust            content  1         1.5    hybrid    1024
python          content  1         1.5    hybrid    1024
docs            content  2         1.0    hybrid    512
decisions       content  0         2.0    bm25      256
memory          memory   3         0.8    bm25      -

$ omega-hpc layers --verbose

[rust]
  description: (none)
  type: content
  priority: 1
  extends: code
  abstract: false
  indexing:
    chunk_size: 1024
    glob: ["*.rs"]
    overlap: 64
    strategy: line
    embed: true
  recall:
    boost: 1.5
    max_results: 5
    strategy: hybrid
    weight_bm25: 0.6
    weight_vector: 0.4
...
```

### add

```bash
omega-hpc add [OPTIONS] <PATH>
```

添加内容到指定层。

Options:
- `--layer <NAME>` 目标层 (default: 按 glob 匹配第一个 content 层)
- `--recursive, -r` 递归
- `--glob <PATTERN>` 文件过滤（覆盖层配置中的 glob）
- `--chunk-size <N>` 分块大小（覆盖层配置）

```bash
omega-hpc add ./src/ --layer rust --recursive
omega-hpc add ./docs/ --layer docs --recursive
omega-hpc add . --layer docs --glob "*.md" --glob "*.txt"
```

**注意**：不能向 `abstract = true` 或 `memory` 类型的层添加内容。

### search

```bash
omega-hpc search [OPTIONS] <QUERY>
```

检索知识库。跨层搜索时使用 Recall Fusion 合并结果。

Options:
- `--layer <NAME>` 指定层（可多次使用），默认搜索所有 content 层
- `--limit, -n <N>` 结果数量 (default: 10)
- `--mode <MODE>` 模式: `hybrid`, `bm25`, `vector`
- `--format <FMT>` 格式: `json`, `table`, `simple`

```bash
# 跨层搜索（自动融合）
omega-hpc search "持久化记忆系统设计"

# 指定层搜索
omega-hpc search "fn main" --layer rust
omega-hpc search "架构决策" --layer decisions --layer docs

# 搜索模式
omega-hpc search "Rust" --mode bm25
omega-hpc search "架构设计" --mode vector
```

### recall

```bash
omega-hpc recall [OPTIONS] <QUERY>
```

回忆 Agent 记忆。

Options:
- `--layer <NAME>` 指定 memory 层（可多次使用），默认搜索所有 memory 层
- `--session <ID>` 限定会话
- `--user <ID>` 限定用户
- `--type <TYPE>` 类型: `fact`, `decision`, `all`

```bash
omega-hpc recall "用户偏好"
omega-hpc recall "架构决策" --type decision
omega-hpc recall "上次讨论" --layer memory
```

### remember

```bash
omega-hpc remember <CONTENT> [OPTIONS]
```

写入记忆到指定 memory 层。

Options:
- `--layer <NAME>` 目标 memory 层 (default: 第一个 memory 层)
- `--type <TYPE>` 类型: `fact`, `decision`
- `--confidence <N>` 置信度 (default: 0.9)
- `--session <ID>` 关联会话
- `--user <ID>` 关联用户

```bash
omega-hpc remember "用户偏好 Rust" --type fact --layer memory
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
- `--layer <NAME>` 指定层

```bash
omega-hpc find --exact "omega-hpc"
omega-hpc find --regex "omega-.*-prd" --layer docs
```

### forget

```bash
omega-hpc forget [OPTIONS]
```

删除记忆或索引。

Options:
- `--layer <NAME>` 指定层
- `--doc <ID>` 删除文档索引
- `--fact <ID>` 删除事实
- `--session <ID>` 删除会话
- `--entity <NAME>` 删除实体

```bash
omega-hpc forget --fact fact_001 --layer memory
omega-hpc forget --doc doc_001 --layer rust
omega-hpc forget --session sess_20260410_001
```

### stat

```bash
omega-hpc stat [OPTIONS]
```

显示统计。

Options:
- `--layer <NAME>` 指定层（默认显示所有层）

```bash
$ omega-hpc stat

Omega HPC Memory Statistics
===========================
Cortex:
  Layers: 5 (3 content, 1 memory, 1 abstract)
  Total Docs: 42
  Total Chunks: 1,247
  Entities: 156

Layer: rust (content, priority 1)
  Documents: 15
  Chunks: 342
  Index Size: 1.2 MB

Layer: docs (content, priority 2)
  Documents: 20
  Chunks: 580
  Index Size: 0.8 MB

Layer: decisions (content, priority 0)
  Documents: 7
  Chunks: 45
  Index Size: 0.2 MB

Layer: memory (memory, priority 3)
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
- `--layer <NAME>` 仅重建指定层
- `--full` 全部重建

```bash
omega-hpc rebuild --layer rust
omega-hpc rebuild --full
```

### gc

```bash
omega-hpc gc [OPTIONS]
```

垃圾回收，清理各 content 层中未被索引引用的孤立 chunk 文件。

Options:
- `--layer <NAME>` 仅清理指定层（默认所有 content 层）
- `--dry-run` 仅报告可回收空间，不执行删除（默认行为）
- `--all` 同时清理 Cortex 中 stale 索引条目

```bash
omega-hpc gc --dry-run
omega-hpc gc
omega-hpc gc --layer rust
omega-hpc gc --all
```

### organize

```bash
omega-hpc organize [OPTIONS]
```

执行整理期，将 raw memory 转化为 organized memory 并更新认知图谱。

Options:
- `--layer <NAME>` 指定 memory 层（默认第一个）
- `--force` 即使未达阈值也强制整理
- `--dry-run` 仅模拟，不写入
- `--delete-raw` 整理后删除已处理的 raw memory

```bash
# 标准整理
omega-hpc organize --layer memory

# 强制整理（即使 raw 很少）
omega-hpc organize --layer memory --force

# 仅模拟，查看会生成什么
omega-hpc organize --layer memory --dry-run

# 整理并清理 raw
omega-hpc organize --layer memory --delete-raw
```

**输出示例**：

```
整理报告
========================
Facts created:    12
Decisions created: 3
Graph nodes:      18
Graph edges:      24
Raw processed:    47
Conflicts resolved: 2
Duration: 4.2s

建议: 下次整理在 raw 数量达到 50 时进行
```

### graph

```bash
omega-hpc graph [OPTIONS]
```

查询认知图谱，支持遍历、搜索等图操作。

Options:
- `--layer <NAME>` 指定 memory 层（默认第一个）
- `--query <QUERY>` 模糊搜索节点名称
- `--node <ID>` 查看指定节点详情
- `--neighbors <ID>` 查看节点的邻居
- `--traverse <ID>` 按关系遍历
- `--relation <REL>` 指定关系类型
- `--depth <N>` 遍历深度

```bash
# 搜索概念
omega-hpc graph --query "Rust"
omega-hpc graph --query "Tantivy"

# 查看节点详情
omega-hpc graph --node concept_tantivy

# 查看某节点的邻居
omega-hpc graph --neighbors concept_tantivy

# 沿 selects 关系遍历 2 层
omega-hpc graph --traverse decision_002 --relation selects --depth 2
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
use omega_hpc_memory::{LayerRegistry, MemoryStore, SearchEngine, Indexer};

fn main() -> Result<()> {
    let config = HpcConfig::load(".hpc.toml")?;
    let registry = LayerRegistry::from_config(&config)?;

    // 按层添加文档
    let mut indexer = Indexer::new(".hpc", &registry)?;
    indexer.add_directory("./src/", "rust")?;
    indexer.add_directory("./docs/", "docs")?;

    // 跨层搜索（自动 fusion）
    let search = SearchEngine::new(".hpc", &registry)?;
    let results = search.search_all("架构设计", 10)?;
    for hit in results {
        println!("[{}] {} (score: {:.2})", hit.layer, hit.content, hit.score);
    }

    // 单层搜索
    let code_results = search.search_layer("fn main", "rust", 5)?;

    // 写入记忆到指定层
    let mut mem = MemoryStore::new("memory", &registry)?;
    mem.save_fact(Fact {
        content: "用户偏好 Rust".into(),
        confidence: 0.95,
    })?;

    // 回忆记忆
    let memories = mem.recall("用户偏好", 5)?;
    for m in memories {
        println!("{:?}", m);
    }

    // 对话提取
    let extracted = mem.extract_facts_from_conversation(&messages)?;
    for fact in extracted.facts {
        mem.save_fact(&fact)?;
    }

    Ok(())
}
```

---

## Configuration

### `.hpc.toml` 完整示例

```toml
[hpc]
cortex_path = ".hpc"

[embedding]
provider = "openai"           # openai | cohere | local
model = "text-embedding-3-small"
api_key = "${OPENAI_API_KEY}"

[embedding.local]
model = "sentence-transformers/all-MiniLM-L6-v2"
dim = 384
cache_dir = "~/.cache/omega-hpc/models"

[vector]
store = "flat"
dim = 1536

[search]
default_limit = 10
hybrid_alpha = 0.7

[[layer]]
name = "code"
type = "content"
abstract = true

[layer.indexing]
chunk_size = 1024
overlap = 64
strategy = "line"
embed = true

[layer.recall]
boost = 1.5
max_results = 5
strategy = "hybrid"
weight_bm25 = 0.6
weight_vector = 0.4

[[layer]]
name = "rust"
extends = "code"
priority = 1

[layer.indexing]
glob = ["*.rs"]

[[layer]]
name = "python"
extends = "code"
priority = 1

[layer.indexing]
glob = ["*.py", "*.pyi"]

[[layer]]
name = "docs"
type = "content"
priority = 2

[layer.indexing]
chunk_size = 512
glob = ["*.md", "*.txt"]
embed = true

[layer.recall]
boost = 1.0
max_results = 10
strategy = "hybrid"

[[layer]]
name = "decisions"
type = "content"
priority = 0

[layer.indexing]
chunk_size = 256
glob = ["docs/decisions/*.md"]
embed = false

[layer.recall]
boost = 2.0
max_results = 3
strategy = "bm25"

[[layer]]
name = "memory"
description = "Agent 认知记忆"
type = "memory"
priority = 3

[layer.memory]
raw_retention = 100       # raw memory 保留上限
organize_on_count = 50    # 建议整理的阈值

[layer.recall]
boost = 0.8
max_results = 5
strategy = "hybrid"
weight_bm25 = 0.3
weight_graph = 0.7         # recall 时结合图谱推理
```

### Layer 配置字段参考

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 唯一标识符，用于 CLI `--layer` 和目录名 |
| `description` | string | 否 | 人类可读描述 |
| `type` | enum | 是 | `content` 或 `memory` |
| `abstract` | bool | 否 | `true` 则不创建存储，仅作继承模板 |
| `extends` | string | 否 | 父层名称，继承全部配置 |
| `priority` | u32 | 是 | 召回优先级，数字越小越优先（0 最高） |

### Indexing 配置（content 类型）

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `chunk_size` | u32 | 512 | 分块大小（tokens） |
| `glob` | string[] | ["*"] | 文件匹配模式 |
| `overlap` | u32 | 0 | 分块重叠（tokens） |
| `strategy` | enum | "fixed" | `fixed`、`line`、`paragraph` |
| `embed` | bool | true | 是否生成向量嵌入 |

### Recall 配置

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `boost` | f32 | 1.0 | 分数放大系数 |
| `max_results` | u32 | 10 | 单层最大返回数 |
| `strategy` | enum | "hybrid" | `bm25`、`vector`、`hybrid` |
| `weight_bm25` | f32 | 0.5 | hybrid 模式 BM25 权重 |
| `weight_vector` | f32 | 0.5 | hybrid 模式向量权重 |
| `weight_graph` | f32 | 0.0 | recall 时图谱推理权重（memory 层） |

### Memory 配置（memory 类型层）

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `raw_retention` | u32 | 100 | raw memory 保留上限，超出后强制整理 |
| `organize_on_count` | u32 | 50 | 建议整理的 raw 数量阈值 |
| `auto_organize` | bool | false | 是否自动整理（定时） |

### Embedding Modes

**远程 API（默认）**：使用 OpenAI/Cohere 嵌入服务，质量最高，需网络。

**本地 CPU（Candle fallback）**：
- 远程不可用时自动降级
- 使用 `--provider local` 强制本地模式
- 默认模型 all-MiniLM-L6-v2（~80MB，384 dim，~50 embeddings/sec）
- 首次运行下载模型到 `cache_dir`

**模型迁移警告**：切换 provider 导致向量维度变化（如 1536 → 384），必须 `omega-hpc rebuild --full` 重建索引。

### Recall Mechanism

`recall` 命令使用 memory 层独立的 BM25 索引：
1. `remember` 写入时同时索引到该层的 BM25
2. `recall` 查询通过 BM25 检索候选记忆
3. Phase 2 后可为 memory 层添加向量索引，提升语义匹配

---

## Tips

### 层设计

1. **代码项目**：使用 `code` 模板，按语言拆分子层（rust/python），继承共享配置
2. **高优先级层**：将关键数据设为 priority 0 + 高 boost，确保排名靠前
3. **轻量层**：不嵌入的层设 `embed = false`，减少 API 调用
4. **抽象模板**：多项目共享基础配置时用 `abstract = true` + `extends`

### 性能

1. **单层搜索**：指定 `--layer` 比跨层搜索更快
2. **BM25 vs Vector**：关键词用 BM25，语义用 Vector
3. **离线使用**：`--provider local` 使用 Candle 本地嵌入

### 调试

1. `omega-hpc layers` 查看层配置和继承关系
2. `omega-hpc layers --validate` 检查配置错误
3. `omega-hpc stat` 查看各层统计
4. `omega-hpc rebuild --layer <name>` 重建单层索引
5. `omega-hpc gc --dry-run` 查看可回收空间

### 记忆管理

1. **显式写入**：`remember` 命令需要显式调用
2. **会话关联**：用 `--session` 关联记忆到会话
3. **手动清理**：`forget` 命令删除过期记忆
4. **对话提取**：SDK 提供 `extract_facts_from_conversation`，由调用方决定何时调用

---

## Troubleshooting

### 搜索无结果

1. `omega-hpc layers` 确认层配置
2. `omega-hpc stat` 确认已添加文档到对应层
3. `omega-hpc add ./src/ --layer rust` 重新添加
4. 尝试 `--mode bm25`

### 层配置错误

```bash
omega-hpc layers --validate
```

常见问题：
- `abstract` 层不能直接用于 `add` 或 `search`
- `extends` 指向不存在的层名
- 循环继承
- `memory` 类型的层没有 `indexing` 配置

### 索引损坏

```bash
omega-hpc rebuild --layer <name>
omega-hpc rebuild --full
```

### 向量搜索失败

向量搜索默认使用远程 API，检查：
- `OPENAI_API_KEY` 环境变量
- 网络连接
- 或使用 `--provider local` 切换到本地 Candle 模式

### 磁盘空间持续增长

```bash
omega-hpc gc --dry-run
omega-hpc gc
```

Content 层使用 content-addressable 存储，文件修改后产生新 chunk，旧 chunk 需通过 `gc` 清理。

### 切换嵌入模型后搜索异常

切换 provider（如 openai → local）导致向量维度变化，需重建：
```bash
omega-hpc rebuild --full
```

---

## Known Limitations

| 限制 | 影响 | 缓解措施 |
|------|------|----------|
| 远程 API 需网络 | 向量搜索/整理 离线不可用 | Candle 本地 fallback |
| 向量维度不可混用 | 切换模型需重建索引 | `omega-hpc rebuild --full` |
| Organization 依赖 LLM | 整理需网络和 API key | 使用本地 LLM 或手动整理 |
| Graph 遍历深度限制 | 深层推理可能超时 | 限制 depth 参数 |
| Content 层无自动 gc | 磁盘可能膨胀 | `omega-hpc gc` 手动清理 |
| tantivy 索引跨版本不兼容 | 升级 tantivy 需重建 | 文档标注版本要求 |
| 层继承仅在配置加载时解析 | 运行时修改继承需重启 | 配置热重载为 Phase 2+ |
| Raw memory 无限积累 | 磁盘膨胀 | 设置 `raw_retention` 并定期 organize |

---

## Content Layer vs Memory Layer

| 命令 | 作用于 | 用途 |
|------|--------|------|
| `add --layer <name>` | content 层 | 添加文档到知识库 |
| `search --layer <name>` | content 层 | 检索知识库文档 |
| `remember --layer <name>` | memory 层 raw/ | 写入短期记忆（碎片） |
| `recall --layer <name>` | memory 层 organized/ + graph/ | 回忆结构化记忆 |
| `organize --layer <name>` | memory 层 | 整理记忆，更新图谱 |
| `graph --query <expr>` | memory 层 graph/ | 查询认知图谱 |

**示例**：

```bash
# 添加源代码到 rust 层
omega-hpc add ./src/ --layer rust --recursive

# 搜索所有 content 层（自动融合）
omega-hpc search "架构设计原则"

# 搜索指定层
omega-hpc search "fn main" --layer rust

# 写入记忆到 memory 层（进入 raw/）
omega-hpc remember "选择了 Rust 因为性能" --layer memory

# 整理记忆（碎片 → 结构化 + 图谱）
omega-hpc organize --layer memory

# 回忆记忆（结合图谱推理）
omega-hpc recall "为什么选择 Rust"

# 查询认知图谱
omega-hpc graph --query "Rust"
omega-hpc graph --traverse decision_002 --relation selects --depth 2
```
