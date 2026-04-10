# Omega HPC Memory CLI Guide

## Installation

```bash
# From source
cargo install --path crates/omega-hpc-memory-cli

# Or use directly
cargo run --bin omega-hpc -- --help
```

## Quick Start

```bash
# 1. Initialize memory store
omega-hpc init omega.omega

# 2. Add project files
omega-hpc add ./docs/
omega-hpc add ./skills/
omega-hpc add . --glob "*.md"

# 3. Search
omega-hpc search "如何使用 omega-hpc"
omega-hpc search "omega-hpc 架构设计"

# 4. Exact find
omega-hpc find --exact "持久化"
omega-hpc find --regex "omega-.*-prd"
```

## Commands Reference

### init

```bash
omega-hpc init [OPTIONS] [PATH]
```

Options:
- `--path <PATH>` 指定 .omega 文件路径 (default: `omega.omega`)
- `--force` 强制覆盖已存在的文件

Example:
```bash
omega-hpc init --path myproject.omega
```

### add

```bash
omega-hpc add [OPTIONS] <PATH>
```

Options:
- `--recursive, -r` 递归添加目录
- `--glob <PATTERN>` 文件过滤模式
- `--chunk-size <N>` 文本分块大小 (default: 512)
- `--exclude <PATTERN>` 排除匹配的文件

Example:
```bash
# Add single file
omega-hpc add README.md

# Add entire directory
omega-hpc add ./docs/ --recursive

# Add with pattern
omega-hpc add . --glob "*.md" --glob "*.txt"

# Add but exclude certain patterns
omega-hpc add . --exclude "node_modules/" --exclude "target/"
```

### search

```bash
omega-hpc search [OPTIONS] <QUERY>
```

Options:
- `--limit, -n <N>` 返回结果数量 (default: 10)
- `--mode <MODE>` 搜索模式: `hybrid`, `bm25`, `vector` (default: `hybrid`)
- `--format <FMT>` 输出格式: `json`, `table`, `simple` (default: `table`)

Example:
```bash
# Basic search
omega-hpc search "记忆系统架构"

# Vector search only (semantic)
omega-hpc search "持久化存储方案" --mode vector

# BM25 only (keyword)
omega-hpc search "CLI 命令" --mode bm25

# Limit results
omega-hpc search "设计原则" --limit 5
```

### find

```bash
omega-hpc find [OPTIONS] --exact <PATTERN>
omega-hpc find [OPTIONS] --regex <PATTERN>
```

Options:
- `--exact <PATTERN>` 精确字符串匹配
- `--regex <PATTERN>` 正则表达式匹配
- `--case-sensitive` 区分大小写 (default: false)
- `--context <N>` 显示匹配上下文行数 (default: 2)

Example:
```bash
# Exact match
omega-hpc find --exact "omega-hpc"

# Regex match
omega-hpc find --regex "omega-.*-prd"

# Case sensitive
omega-hpc find --exact "Rust" --case-sensitive

# Show context
omega-hpc find --exact "memory" --context 3
```

### stat

```bash
omega-hpc stat
```

Display memory store statistics.

Example output:
```
Omega HPC Memory Statistics
==========================
File: omega.omega
Version: 1.0
Created: 2026-04-10
Updated: 2026-04-10

Documents: 42
  - Markdown: 28
  - Text: 10
  - Code: 4

Chunks: 1,247
Entities: 156

Index Status:
  - BM25: Ready
  - Vector: Ready (384 dims)

Size: 2.4 MB
```

### export

```bash
omega-hpc export [OPTIONS]
```

Options:
- `--format <FMT>` 导出格式: `json`, `markdown`, `text`
- `--output <PATH>` 输出文件路径

Example:
```bash
# Export all as JSON
omega-hpc export --format json --output backup.json

# Export as markdown
omega-hpc export --format markdown --output backup/
```

### doctor

```bash
omega-hpc doctor
```

诊断记忆库健康状态，检查损坏和修复建议。

### eval

```bash
omega-hpc eval [OPTIONS]
```

运行评测基准测试，评估记忆系统性能。

Options:
- `--benchmark <NAME>` 评测基准: `locomo`, `omega-hpc`, `robustness` (default: `locomo`)
- `--dataset <PATH>` 评测数据集路径
- `--output <PATH>` 结果输出文件路径
- `--format <FMT>` 输出格式: `json`, `table`, `html` (default: `table`)
- `--compare <BASELINE>` 与基线系统对比: `mem0`, `full-context`, `openai`

Example:
```bash
# Run LoCoMo benchmark
omega-hpc eval --benchmark locomo

# Run with custom dataset
omega-hpc eval --benchmark locomo --dataset ./benchmarks/locomo/

# Run and save results
omega-hpc eval --benchmark locomo --output results.json

# Compare with Mem0 baseline
omega-hpc eval --benchmark locomo --compare mem0

# Generate HTML report
omega-hpc eval --benchmark locomo --format html --output report.html
```

Example output:
```
Omega HPC Memory Evaluation
==========================
Benchmark: LoCoMo

Results:
┌─────────────┬───────────┬────────┬────────┐
│ Category    │ LLM Score │ BLEU   │ F1     │
├─────────────┼───────────┼────────┼────────┤
│ Single-hop  │ 0.85      │ 0.72   │ 0.78   │
│ Temporal    │ 0.72      │ 0.65   │ 0.68   │
│ Multi-hop   │ 0.68      │ 0.58   │ 0.62   │
│ Open-domain │ 0.75      │ 0.66   │ 0.70   │
└─────────────┴───────────┴────────┴────────┘

Performance:
  Token Cost (avg): 1,847
  P50 Latency: 45ms
  P95 Latency: 120ms

Overall LLM Score: 0.75
```

## Configuration

配置文件: `.omega-hpc-memory.toml` 或 `~/.config/omega-hpc-memory/config.toml`

```toml
[memory]
path = "omega.omega"

[indexing]
chunk_size = 512
overlap = 50

[embedding]
model = "local"
local_model = "sentence-transformers/all-MiniLM-L6-v2"

[search]
default_limit = 10
hybrid_alpha = 0.7  # BM25 weight
```

## Tips

### Performance

1. **批量添加**: 多次 `add` 会增量索引，无需重建
2. **选择模式**: 简单关键词用 `--mode bm25`，语义搜索用 `--mode vector`
3. **限制范围**: 使用 `--glob` 限制文件类型

### Debugging

1. 使用 `omega-hpc stat` 检查索引状态
2. 使用 `omega-hpc doctor` 诊断问题
3. 使用 `--format json` 获取完整结果数据

## Troubleshooting

### 搜索无结果

1. 检查索引是否就绪: `omega-hpc stat`
2. 尝试降低 limit 或更改搜索模式
3. 验证文档是否已添加

### 文件损坏

```bash
# Run diagnostics
omega-hpc doctor

# If recoverable, it will suggest a command
omega-hpc repair
```
