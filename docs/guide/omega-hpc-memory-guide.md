# Omega HPC Memory CLI Guide

## Overview

omega-hpc-memory 是统一记忆与知识库系统，服务于：
- **Agent 长期记忆** - 跨会话保留决策、偏好
- **外部知识库** - 项目文档的索引与检索

**核心理念：索引即记忆**。保存索引结构，内容按需加载。

## Installation

```bash
cargo install --path crates/omega-hpc-memory-cli
# 或
cargo run --bin omega-hpc -- --help
```

## Quick Start

```bash
# 1. 初始化项目记忆空间
omega-hpc init

# 2. 添加知识库内容
omega-hpc add ./docs/ --recursive
omega-hpc add . --glob "*.md"

# 3. 搜索知识库
omega-hpc search "omega-hpc 的设计原则"

# 4. 回忆相关记忆
omega-hpc recall "上次为什么选了 PostgreSQL"

# 5. 查看统计
omega-hpc stat
```

## Memory Space Structure

初始化后创建以下结构：

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

## Commands Reference

### init

```bash
omega-hpc init [OPTIONS] [PATH]
```

初始化项目记忆空间。

Options:
- `--path <PATH>` 记忆空间路径 (default: `.hpc`)
- `--force` 强制覆盖

```bash
# 在当前目录初始化
omega-hpc init

# 在指定路径初始化
omega-hpc init --path /path/to/project/.hpc
```

### add

```bash
omega-hpc add [OPTIONS] <PATH>
```

添加知识库内容。

Options:
- `--recursive, -r` 递归添加目录
- `--glob <PATTERN>` 文件过滤
- `--chunk-size <N>` 分块大小 (default: 512)
- `--exclude <PATTERN>` 排除模式
- `--no-content` 仅添加索引引用

```bash
# 添加整个目录
omega-hpc add ./docs/ --recursive

# 添加特定文件类型
omega-hpc add . --glob "*.md" --glob "*.txt"

# 仅添加索引（不存储内容副本）
omega-hpc add ./docs/ --no-content
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
# 混合检索
omega-hpc search "持久化记忆系统设计"

# 纯语义搜索
omega-hpc search "跨会话记忆方案" --mode vector

# 关键词搜索
omega-hpc search "BM25 Tantivy" --mode bm25

# JSON 输出
omega-hpc search "架构设计" --format json
```

### recall

```bash
omega-hpc recall [OPTIONS] <QUERY>
```

回忆 Agent 相关记忆。

Options:
- `--session <ID>` 限定会话
- `--user <ID>` 限定用户
- `--type <TYPE>` 类型: `fact`, `decision`, `all`

```bash
# 回忆所有相关记忆
omega-hpc recall "Rust"

# 仅回忆事实
omega-hpc recall "用户偏好" --type fact

# 回忆某会话的决策
omega-hpc recall "架构决策" --session sess_20260410_001
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
# 删除文档索引（保留内容）
omega-hpc forget --doc doc_001

# 删除特定记忆
omega-hpc forget --fact fact_001

# 删除整个会话
omega-hpc forget --session sess_20260410_001
```

### stat

```bash
omega-hpc stat
```

显示统计信息。

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

Vector Index:
  Dimensions: 384
  Status: Ready
```

### eval

```bash
omega-hpc eval [OPTIONS]
```

运行评测基准。

Options:
- `--benchmark <NAME>` 基准: `locomo`, `knowledge`, `memory`
- `--dataset <PATH>` 数据集路径
- `--output <PATH>` 输出文件
- `--format <FMT>` 格式: `json`, `table`, `html`

```bash
# 运行知识库检索基准
omega-hpc eval --benchmark knowledge

# 运行记忆召回基准
omega-hpc eval --benchmark memory

# 带输出
omega-hpc eval --benchmark locomo --output results.json
```

## Configuration

配置文件: `.hpc.toml`

```toml
[omega]
cortex_path = ".hpc"

[indexing]
chunk_size = 512
overlap = 50

[embedding]
provider = "onnx"          # onnx | openai
model = "sentence-transformers/all-MiniLM-L6-v2"

[vector]
store = "hnsw"             # flat | hnsw
dim = 384

[search]
default_limit = 10
hybrid_alpha = 0.7         # BM25 weight
```

## Tips

### 性能

1. **批量添加**: 多次 `add` 增量索引，无需全量重建
2. **模式选择**: 关键词用 `--mode bm25`，语义用 `--mode vector`
3. **内容按需**: 使用 `--no-content` 可只建索引，不存副本

### 调试

1. `omega-hpc stat` 查看索引状态
2. `omega-hpc find --exact <term>` 验证索引
3. `--format json` 获取完整返回数据

### 记忆管理

1. **主动记忆**: Agent 通过 MCP 接口主动写入 `omega-hpc.mem`
2. **自动提取**: 对话结束时自动提取 facts 和 decisions
3. **定期清理**: 使用 `omega-hpc forget` 删除过期记忆

## Troubleshooting

### 搜索无结果

1. 检查索引状态: `omega-hpc stat`
2. 确认文档已添加: `omega-hpc add ./docs/`
3. 尝试 `--mode bm25` 关键词搜索

### 索引损坏

```bash
# 诊断
omega-hpc doctor

# 重建索引
omega-hpc add . --recursive --glob "*.md"
```

### 内存不足

```toml
# 切换到 flat 向量
[vector]
store = "flat"
```
