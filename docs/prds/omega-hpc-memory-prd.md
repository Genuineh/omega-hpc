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

**个性化检索缺失**
- 固定分层无法适应不同项目需求
- 代码项目需要源代码专用索引策略（语言感知分块、高召回优先级）
- 文档项目需要不同分块大小和权重
- 用户无法自定义召回维度和优先级

---

## Design Principles

### 1. 索引即记忆

存储的是：
- **索引结构** - BM25、向量、实体槽位的组织方式
- **引用映射** - 内容在哪、如何加载（不存副本）
- **关联图谱** - 实体之间的关系

类比：大脑保存神经连接模式，而非完整经历录像。

### 2. 可配置数据层

数据层（Layer）是用户可配置的存储与检索单元。每个 Layer 描述一类数据的索引策略、存储方式和召回行为。

**核心能力**：
- 用户通过 `.hpc.toml` 定义任意数量的数据层
- 每层独立配置索引参数（分块大小、glob 模式、分块策略）
- 每层独立配置召回行为（优先级、boost、检索策略）
- 层间支持继承，避免重复配置
- 跨层搜索使用 Recall Fusion 合并结果

**层类型**：
- `content` - 外部文件内容，自动分块索引
- `memory` - Agent 结构化记忆，手动/SDK 写入

### 3. 多文件分层

每个目录类型有明确职责，边界清晰：

| 目录 | 职责 | 内容类型 | 边界规则 |
|------|------|----------|----------|
| `.hpc/cortex/` | 元认知索引 | TOML + 二进制索引 | 索引可独立重建 |
| `.hpc/layers/{name}/` | 用户定义的数据层 | content 或 memory | 按层配置管理 |

### 4. 层间一致性

**Cortex 与 Layer 的一致性**：
- Content 层内容通过 content-hash 引用，hash 不变则内容不变
- Cortex 按层组织索引，每层有独立的 BM25 索引
- 文件外部修改后，hash 变化，Cortex 标记该 chunk 为 stale

**跨层 Recall Fusion**：
- 搜索时多层结果按 boost 调分后合并
- Priority 决定同分时的排列顺序
- 每层 max_results 限制单层返回数量

### 5. Git 友好

- Cortex 文件是 TOML，可 diff
- Content 层使用 content-addressable 存储
- Memory 层文件是 TOML，可追踪变更历史
- `.hpc.toml` 层配置可纳入版本控制，团队共享

---

## Architecture

### 目录结构

```
.hpc/                          # 项目记忆根目录
├── cortex/                    # 元认知层（全局索引）
│   ├── meta.toml              # 全局元数据
│   ├── layers/                # 按层拆分的索引
│   │   ├── {name}/
│   │   │   ├── index.toml     # 层级文档清单
│   │   │   ├── docs/          # 按文档拆分的索引
│   │   │   └── bm25/          # tantivy 索引
│   │   └── ...
│   ├── entities.toml          # 实体槽位
│   └── embeddings/            # 向量分片
│
├── layers/                    # 用户定义的数据层
│   ├── {content-layer}/       # content: chunk 存储
│   ├── {memory-layer}/        # memory: TOML 记忆
│   └── ...
│
└── sync/
    └── manifest.toml
```

### 层配置示例

**代码项目**（多语言，通过继承减少重复）：

```toml
[[layer]]
name = "code"
type = "content"
abstract = true              # 模板层，不创建存储

[layer.indexing]
chunk_size = 1024
overlap = 64
strategy = "line"
embed = true

[layer.recall]
boost = 1.5
max_results = 5
strategy = "hybrid"

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
name = "decisions"
type = "content"
priority = 0                # 最高优先级

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
strategy = "bm25"
```

### 层继承机制

```
resolve(name):
  layer = config[name]
  if layer.extends:
    parent = resolve(layer.extends)
    return deep_merge(parent, layer)
  return layer
```

**规则**：
- 标量：子覆盖父
- 数组：子替换父
- 表：递归合并
- 支持多级继承链（A → B → C）
- 禁止循环继承
- `abstract = true` 的层仅作模板，不创建存储目录

---

## Memory Writing Mechanism

### 写入触发方式

**1. 手动写入（CLI）**
```bash
omega-hpc remember "用户偏好 Rust" --type fact --layer memory
omega-hpc remember "选择 Tantivy" --type decision
```

**2. 手动写入（SDK）**
```rust
let mem = MemoryStore::new("memory", &registry)?;
mem.save_fact(Fact {
    content: "用户偏好 Rust".into(),
    confidence: 0.9,
})?;
```

**3. 会话快照（CLI）**
```bash
omega-hpc session end
```

**4. SDK 会话管理**
```rust
let session = mem.start_session("claude-code", "jerryg")?;
session.extract_facts_from_conversation(&messages)?;
session.save_decision("选择了 X 方案", "因为 Y 原因")?;
session.end()?;
```

### 何时触发记忆提取

- **显式触发**：用户或 Agent 调用 `remember` 命令
- **会话结束**：自动生成会话摘要（如果启用）
- **对话提取**：SDK 提供 `extract_facts_from_conversation` 方法，由调用方决定何时调用

### 对话记忆提取管道

为支持 LoCoMo 等对话记忆评测场景，SDK 提供对话 → 结构化记忆的提取接口：

```
对话历史 → [LLM 提取] → 结构化 facts/decisions → 写入 memory 层
```

**提取流程**：
1. 调用方传入对话消息列表
2. SDK 使用 LLM 提取关键 facts 和 decisions
3. 返回提取结果供调用方确认或修改
4. 调用方确认后写入指定 memory 层

**LLM 依赖**：
- 提取质量取决于 LLM 能力
- 支持配置不同 LLM 提供者（OpenAI/本地）
- 本地模式可使用 Candle 运行小型模型（质量降低但离线可用）

### 不做的事

- 不会自动监听对话并提取记忆（需要显式调用）
- 不会在后台运行 LLM 进行摘要（由调用方负责）

---

## Technical Decisions

### 决策点

| 决策点 | 选择 | 理由 |
|--------|------|------|
| 存储架构 | 可配置数据层 | 适应不同项目需求，个性化检索 |
| 索引格式 | TOML + 二进制 | TOML 可 diff，二进制高效 |
| 内容存储 | Content-addressable | Hash 寻址，防篡改，可去重 |
| 层间继承 | deep_merge | 避免重复配置，DRY |
| 向量存储 | 可插拔 | 支持 flat/HNSW |
| 嵌入模型 | 远程优先 + 本地 fallback | 远程高质量，Candle 本地离线可用 |
| CLI/SDK | 仅这两者 | 不提供 MCP 服务 |

### 嵌入模型选择

**双模式：远程 API 优先 + 本地 CPU fallback**

| 方案 | 状态 | 说明 |
|------|------|------|
| 远程 API | **主模式** | OpenAI/Cohere，最高质量，需网络 |
| Candle 本地 | **fallback** | 纯 Rust，CPU 运行，离线可用 |

```toml
[embedding]
provider = "openai"           # openai | cohere | local
model = "text-embedding-3-small"
api_key = "${OPENAI_API_KEY}"

[embedding.local]
model = "sentence-transformers/all-MiniLM-L6-v2"  # Candle 加载
dim = 384
```

**本地模式说明**：
- 使用 [Candle](https://github.com/huggingface/candle) (HuggingFace 纯 Rust ML 框架) 运行小型 embedding 模型
- 仅需 CPU，无 CUDA/MKL/ONNX 依赖
- 模型约 80MB，首次下载后缓存本地
- 质量：384 维 vs 远程 1536 维，语义匹配略低但完全可用

### 向量存储选择

| 方案 | 状态 | 说明 |
|------|------|------|
| Flat | Phase 1 | 简单，够用 |
| HNSW | Phase 2 | 性能更好，可选 |

---

## Requirements

### Must Have

1. **可配置数据层系统**
   - 用户通过 `.hpc.toml` 定义数据层
   - 支持 content 和 memory 两种层类型
   - 每层独立配置索引参数和召回行为
   - 层间继承机制（abstract + extends）
   - 内置 default/code/minimal 模板

2. **多格式知识库索引**
   - 支持 Markdown、纯文本、代码、结构化文档
   - 按层配置分块（大小、策略、glob）
   - Content-addressable 存储

3. **混合检索引擎**
   - BM25 词项搜索（精确）
   - 向量相似搜索（语义，远程 API）
   - 跨层 Recall Fusion（boost + priority）

4. **RCLI 命令行**
   - `omega-hpc init` - 初始化（含模板选择）
   - `omega-hpc add --layer <name>` - 添加到指定层
   - `omega-hpc search --layer <name>` - 检索（跨层或单层）
   - `omega-hpc recall --layer <name>` - 回忆记忆
   - `omega-hpc remember --layer <name>` - 写入记忆
   - `omega-hpc forget --layer <name>` - 删除
   - `omega-hpc layers` - 列出层配置
   - `omega-hpc stat` - 统计
   - `omega-hpc gc` - 垃圾回收

5. **SDK 编程接口**
   - Rust crate 发布到 crates.io
   - 提供 `LayerRegistry`, `MemoryStore`, `SearchEngine`, `Indexer` 核心 API

### Should Have

6. **增量索引更新**
   - 文件变化时只更新受影响部分
   - `omega-hpc rebuild` 重建索引

7. **多项目隔离**
   - 不同项目使用不同 .hpc 目录

8. **时间旅行**
   - 查看历史索引版本

### Nice to Have

9. **记忆总结**
   - SDK 提供记忆序列化接口
   - LLM 摘要由调用方负责

---

## Timeline

### Phase 1: 基础框架 + 可配置层 + 知识库索引（MVP）

- [ ] 项目初始化 + `init` 命令（含模板）
- [ ] Layer 配置解析 + 继承解析
- [ ] `.hpc/cortex/` + `.hpc/layers/` 实现
- [ ] `add --layer` 命令 + 分块逻辑
- [ ] BM25 索引（via tantivy，按层独立）
- [ ] `search --layer` 命令（BM25 only）
- [ ] `rebuild` 命令
- [ ] `layers` 命令

### Phase 2: 向量搜索 + 召回融合

- [ ] 远程嵌入 API 集成
- [ ] Local CPU embedding via Candle
- [ ] `search --mode vector` / `search --mode hybrid`
- [ ] Recall Fusion（boost + priority 跨层合并）
- [ ] `stat` 命令（按层统计）
- [ ] HNSW vector store（可选）

### Phase 3: 记忆层 + SDK

- [ ] Memory 类型层实现
- [ ] `remember --layer` 命令
- [ ] `recall --layer` 命令
- [ ] `forget --layer` 命令
- [ ] `extract_facts_from_conversation` SDK 方法
- [ ] Rust SDK 发布
- [ ] Session 管理 API

### Phase 4: 高级特性

- [ ] 增量索引更新
- [ ] `gc` 命令（按层回收）
- [ ] 时间旅行
- [ ] `eval` 评测命令

---

## Benchmark & Evaluation

### 评测维度

系统同时服务于文档检索和 Agent 记忆，评测分为三个方向：

#### A. LoCoMo Benchmark（对话记忆）

**来源**: [Mem0 论文](https://arxiv.org/abs/2504.19413)，测试 AI Agent 在多轮多会话对话中记忆和检索用户信息的能力。

**适配方式**: 本项目通过 SDK 提供对话记忆接口，调用方负责将对话内容写入 memory 层，然后使用 LoCoMo 数据集评测 recall 的准确性。

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

---

## Known Limitations

| 限制 | 影响 | 缓解措施 |
|------|------|----------|
| 远程 API 需网络 | 向量搜索离线不可用 | Candle 本地 fallback |
| 向量维度不可混用 | 切换模型需重建索引 | `omega-hpc rebuild --full` |
| Memory 层 recall 依赖 BM25 | 语义匹配不如向量 | Phase 2 为 Memory 层添加向量索引 |
| Content 层无自动 gc | 磁盘可能膨胀 | `omega-hpc gc` 手动清理 |
| tantivy 索引跨版本不兼容 | 升级 tantivy 需重建 | 文档标注版本要求 |
| eval 依赖外部 LLM | 评测需网络和 API key | 可配置本地 LLM |
| 层继承仅在配置加载时解析 | 运行时修改继承需重启 | 配置热重载为 Phase 2+ |
