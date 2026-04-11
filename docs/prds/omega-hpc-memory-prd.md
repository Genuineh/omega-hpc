# Omega HPC Memory: Unified Memory and Knowledge Base System

**状态**: Active

---

## Summary

为 omega-hpc 设计一套统一记忆与知识库系统，同时服务于：

1. **Agent 长期记忆** - 跨会话保留决策、偏好、上下文
2. **外置知识库** - 以项目为单位的文档索引与检索

**核心理念：索引即记忆**。保存结构化索引（类比大脑皮层/元认知），内容按需加载。

**集成方式**：仅通过 CLI 或 SDK 集成，不提供 MCP 服务。

---

## Problem

当前 omega-hpc 存在两层需求未被满足：

**记忆层缺失**
- Agent 每次会话从零开始，无法保留历史决策
- 跨会话无法复用经验
- 缺乏显式记忆写入机制

**知识库层缺失**
- 大量文档和技能指南无法快速检索
- 文档检索与 Agent 记忆是割裂的两套系统

---

## Design Principles

### 1. 索引即记忆

存储的是：
- **索引结构** - BM25、向量、实体槽位的组织方式
- **引用映射** - 内容在哪、如何加载（不存副本）
- **关联图谱** - 实体之间的关系

类比：大脑保存神经连接模式，而非完整经历录像。

### 2. 多文件分层

每个目录类型有明确职责，边界清晰：

| 目录 | 职责 | 内容类型 | 边界规则 |
|------|------|----------|----------|
| `.hpc/cortex/` | 元认知索引 | TOML + 二进制索引 | 索引可独立重建 |
| `.hpc/kb/` | 知识库内容 | Chunk 文件 | 按 content-hash 寻址 |
| `.hpc/mem/` | Agent 记忆 | 结构化 .mem 文件 | 按用户/会话隔离 |

### 3. 层间一致性

**Cortex 与 KB 的一致性**：
- KB 内容通过 content-hash 引用，hash 不变则内容不变
- Cortex 记录 `content_hash`，用于校验 KB 内容完整性
- 文件外部修改后，hash 变化，Cortex 标记该 chunk 为 stale

**Mem 与 KB 的关系**：
- Mem 是结构化记忆（facts/decisions），不是文档 chunks
- Mem 可引用 KB 中的文档（通过 doc_id）
- Mem 有独立的查询接口（recall 命令）

### 4. Git 友好

- Cortex 文件是 TOML，可 diff
- KB 使用 content-addressable 存储，每个 hash 对应确定内容
- Mem 文件是 TOML，可追踪变更历史

---

## Architecture

### 目录结构

```
.hpc/                          # 项目记忆根目录（隐藏目录）
├── cortex/                    # 元认知层（索引）
│   ├── index.toml             # 主索引清单
│   ├── bm25.bin               # BM25 结构（二进制）
│   ├── entities.toml          # 实体槽位映射
│   └── embeddings/            # 向量分片
│       └── shard_*.bin
│
├── kb/                        # 知识库层（内容）
│   └── {sha256}.chunk        # 内容分片，按 hash 寻址
│
├── mem/                       # 记忆层（Agent 私有）
│   ├── sessions/
│   │   └── {session-id}.mem  # 会话级记忆
│   └── users/
│       └── {user-id}.mem     # 用户级记忆
│
└── sync/
    └── manifest.toml          # 变更追踪
```

### 各层边界定义

#### Cortex Layer (`.hpc/cortex/`)

**存储内容**：
- 文档清单（哪些文件被索引）
- Chunk 映射（文档 → chunks 的映射关系）
- BM25 索引数据
- 向量索引数据
- 实体槽位表

**不存储**：原始内容（由 KB 层提供）

**一致性保证**：
- 每个 document 记录 `content_hash`
- 添加文档时同时写入 KB 层并更新 Cortex
- KB 内容 hash 校验失败时，Cortex 标记为 stale，需重建

#### Knowledge Layer (`.hpc/kb/`)

**存储内容**：
- 文档分片（原始内容的压缩块）
- 每个 chunk 独立校验

**边界规则**：
- 内容通过 hash 寻址，不可变
- 外部修改文件 → hash 变化 → 视为新 chunk
- 旧 chunk 保留直至手动 gc

#### Memory Layer (`.hpc/mem/`)

**存储内容**：
- 结构化记忆（facts、decisions）
- 会话元数据
- Agent 提取的摘要信息

**与 KB 的区别**：

| 维度 | KB 层 | Mem 层 |
|------|-------|--------|
| 内容类型 | 文档分片 | 结构化记忆 |
| 来源 | 外部文件 | Agent 提取/写入 |
| 格式 | 二进制 chunk | TOML |
| 查询接口 | `search` | `recall` |
| 用途 | "这个项目有什么文档" | "Agent 上次怎么决策的" |

---

## Memory Writing Mechanism

### 写入触发方式

**1. 手动写入（CLI）**
```bash
omega-hpc remember "用户偏好 Rust" --type fact --confidence 0.9
omega-hpc remember "选择了 Tantivy 因为无外部依赖" --type decision
```

**2. 手动写入（SDK）**
```rust
mem_store.save_fact(Fact {
    content: "用户偏好 Rust".into(),
    confidence: 0.9,
})?;
```

**3. 会话快照（CLI）**
```bash
# 结束会话时自动生成快照
omega-hpc session end
```

**4. SDK 会话管理**
```rust
let session = mem_store.start_session("claude-code", "jerryg")?;
session.extract_facts_from_conversation(&messages)?;
session.save_decision("选择了 X 方案", "因为 Y 原因")?;
session.end()?;
```

### 何时触发记忆提取

- **显式触发**：用户或 Agent 调用 `remember` 命令
- **会话结束**：自动生成会话摘要（如果启用）
- **定时归档**：可选，定期将短期记忆归档为长期记忆

### 不做的事

- 不会自动监听对话并提取记忆（需要显式调用）
- 不会在后台运行 LLM 进行摘要（由调用方负责）
- 不会自动清理过期记忆（由用户决定）

---

## Technical Decisions

### 决策点

| 决策点 | 选择 | 理由 |
|--------|------|------|
| 存储架构 | 多文件分层 | 职责分离、Git 友好、可独立更新 |
| 索引格式 | TOML + 二进制 | TOML 可 diff，二进制高效 |
| KB 内容 | Content-addressable | Hash 寻址，防篡改，可去重 |
| 向量存储 | 可插拔 | 支持 flat/HNSW |
| 嵌入模型 | **仅支持远程 API** | 避免 ONNX C 库依赖 |
| CLI/SDK | 仅这两者 | 不提供 MCP 服务 |

### 嵌入模型选择

**明确：仅支持远程 API（Phase 2）**

| 方案 | 状态 | 说明 |
|------|------|------|
| ONNX 本地 | **不采用** | ort crate 需要 C 库依赖，违反"零外部依赖" |
| Candle/burn | **不采用** | 生态不成熟，功能有限 |
| 远程 API | **采用** | OpenAI/Cohere 等，零本地依赖 |

```toml
[embedding]
provider = "openai"           # 唯一选项
model = "text-embedding-3-small"
api_key = "${OPENAI_API_KEY}"
```

### 向量存储选择

| 方案 | 状态 | 说明 |
|------|------|------|
| Flat | Phase 1 | 简单，够用 |
| HNSW | Phase 2 | 性能更好，可选 |

---

## Requirements

### Must Have

1. **多格式知识库索引**
   - 支持 Markdown、纯文本、代码、结构化文档
   - 自动分块（512 tokens 默认）
   - Content-addressable 存储

2. **混合检索引擎**
   - BM25 词项搜索（精确）
   - 向量相似搜索（语义，远程 API）
   - 实体槽位 O(1) 查询

3. **RCLI 命令行**
   - `omega-hpc init` - 初始化项目记忆空间
   - `omega-hpc add` - 添加知识库内容
   - `omega-hpc search` - 混合检索
   - `omega-hpc recall` - 回忆相关记忆
   - `omega-hpc remember` - 写入记忆
   - `omega-hpc forget` - 删除记忆/索引

4. **SDK 编程接口**
   - Rust crate 发布到 crates.io
   - 提供 `MemoryStore`, `SearchEngine`, `Indexer` 核心 API

### Should Have

5. **增量索引更新**
   - 文件变化时只更新受影响部分
   - `omega-hpc rebuild` 重建索引

6. **多项目隔离**
   - 不同项目使用不同 .hpc 目录

7. **时间旅行**
   - 查看历史索引版本

### Nice to Have

8. **记忆总结**
   - SDK 提供记忆序列化接口
   - LLM 摘要由调用方负责

---

## Timeline

### Phase 1: 基础框架 + 知识库索引（MVP）

- [ ] 项目初始化 + `init` 命令
- [ ] `.hpc/cortex/` + `.hpc/kb/` 实现
- [ ] `add` 命令 + 分块逻辑
- [ ] BM25 索引（via tantivy）
- [ ] `search` 命令（BM25 only）

### Phase 2: 向量搜索 + 实体

- [ ] 远程嵌入 API 集成
- [ ] `search --mode vector`
- [ ] `search --mode hybrid`
- [ ] 实体提取 + entities.toml
- [ ] `omega-hpc stat`

### Phase 3: 记忆层 + SDK

- [ ] `.hpc/mem/` 实现
- [ ] `remember` 命令
- [ ] `recall` 命令
- [ ] `forget` 命令
- [ ] Rust SDK 发布
- [ ] Session 管理 API

### Phase 4: 高级特性

- [ ] 增量索引更新
- [ ] `rebuild` 命令
- [ ] 时间旅行
- [ ] `eval` 评测命令

---

## Benchmark & Evaluation

### 评测维度

系统同时服务于文档检索和 Agent 记忆，评测分为三个方向：

#### A. LoCoMo Benchmark（对话记忆）

**来源**: [Mem0 论文](https://arxiv.org/abs/2504.19413)，测试 AI Agent 在多轮多会话对话中记忆和检索用户信息的能力。

**适配方式**: 本项目通过 SDK 提供对话记忆接口，调用方负责将对话内容写入 Mem 层，然后使用 LoCoMo 数据集评测 recall 的准确性。

| 维度 | 说明 | 示例 |
|------|------|------|
| Single-hop | 单点事实查询 | "用户的职位是什么？" |
| Temporal | 时间相关查询 | "用户上周说想去哪旅行？" |
| Multi-hop | 多跳推理查询 | "用户的经理偏好什么工作方式？" |
| Open-domain | 开放域问答 | "总结用户的职业背景" |

**评测指标**:
- **LLM-as-Judge Score** - LLM 评判答案正确性 (0-1)
- **BLEU Score** - n-gram 相似度
- **F1 Score** - 精确率/召回率调和平均

**性能指标**:
- **Token 消耗** - 单次查询消耗的 Token 数
- **P50/P95 延迟** - 检索延迟分布

**参考基线** (来自 Mem0 论文):
| 系统 | LLM Score | Token 消耗 | P95 延迟 |
|------|-----------|------------|----------|
| Full-context | ~72.9% | ~26K | ~17.12s |
| OpenAI Memory | ~52.9% | ~8K | ~4.2s |
| Mem0 | ~66.9% | ~1.8K | ~1.44s |
| Mem0+Graph | ~68.4% | ~2.5K | ~2.59s |

#### B. 知识库检索评测

| 维度 | 说明 |
|------|------|
| Single-hop | 直接文档查询（"Tantivy 在哪用？"） |
| Multi-doc | 跨文档关联查询 |

**指标**：Recall@K, MRR, P50/P95 Latency

#### C. 记忆系统评测

| 维度 | 说明 |
|------|------|
| 记忆召回 | 写入的记忆能否被正确 recall |
| 干扰抵抗 | 新记忆是否影响旧记忆 |
| 时序推理 | 基于时间顺序的推理查询 |

**指标**：Recall, Precision

### 评测目标

| 指标 | 目标 | 说明 |
|------|------|------|
| LoCoMo LLM Score | > 0.60 | 对话记忆检索准确性 |
| Recall@K (知识库) | > 0.85 | 文档检索准确性 |
| 记忆精度 | > 0.80 | Agent 记忆与事实的符合度 |
| P50 延迟 | < 30ms | 纯索引查询 |
| P95 延迟 | < 150ms | 含内容加载的查询 |
| Token 消耗 | < 2K | 远低于 full-context |

### 防作弊机制

1. **闭卷索引** - 评测数据集不得在构建索引时使用
2. **独立题库** - 题库与代码库分离，定期轮换
3. **无捷径优化** - 禁止针对评测题目添加人工规则
4. **不可预测性** - 评测题目在系统设计时未知
5. **可复现性** - 评测结果可被独立复现

---

## Related Documents

- `docs/specs/omega-hpc-memory-spec.md` - 详细技术规格
- `docs/guide/omega-hpc-memory-guide.md` - 使用指南
