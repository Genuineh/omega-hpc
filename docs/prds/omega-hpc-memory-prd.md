# Omega HPC Memory: Unified Memory and Knowledge Base System

**状态**: Active

---

## Summary

为 omega-hpc 设计一套统一记忆与知识库系统，同时服务于：

1. **Agent 长期记忆** - 跨会话保留决策、偏好、上下文
2. **外置知识库** - 以项目为单位的文档索引与检索

**核心理念：索引即记忆**。保存结构化索引（类比大脑皮层/元认知），内容按需加载。

**认知架构**：系统模拟人类记忆的三层结构——短期记忆（碎片）→ 整理期（巩固）→ 长期记忆 + 认知图谱（结构化知识）。记忆不是简单堆放，而是经过组织过程形成项目的认知体系。

**集成方式**：仅通过 CLI 或 SDK 集成，不提供 MCP 服务。

---

## Problem

当前 omega-hpc 存在三层需求未被满足：

**记忆层缺失**
- Agent 每次会话从零开始，无法保留历史决策
- 跨会话无法复用经验
- 缺乏显式记忆写入机制

**知识库层缺失**
- 大量文档和技能指南无法快速检索
- 文档检索与 Agent 记忆是割裂的两套系统

**认知体系缺失**
- 记忆碎片没有整理过程，无法形成结构化知识
- 决策没有关联因果链，无法推理"为什么"
- 缺乏项目级的概念图谱，无法理解实体关系

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

### 6. 三层记忆与认知体系

Memory 层采用三层架构，模拟人类记忆的整理过程：

```
短期记忆(raw/) ──整理期──→ 长期记忆(organized/) ──提取──→ 认知图谱(graph/)
     ↓                                              ↑
  碎片、矛盾、                          概念节点 + 关系边
  低置信度                              支持推理查询
```

**为什么需要整理期（Organization Period）**：

`remember` 写入的是原始碎片——快速、未经筛选。类比人类睡眠时的记忆巩固过程，系统需要定期整理：
- **去重**：合并相似的记忆碎片
- **矛盾检测**：识别相互矛盾的信息
- **结构化**：将碎片转化为高置信度的事实和决策
- **图谱提取**：从结构化记忆中提取概念和关系

**认知图谱作为元认知**：

认知图谱是项目的"理解层"——不是记忆本身，而是关于记忆的知识。它使系统能够：
- 推理"为什么做了这个决策"（决策链遍历）
- 理解"这个概念和其他概念的关系"（关系网络）
- 追踪"这个决定的演变过程"（时间线）

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
│   ├── {memory-layer}/        # memory: 三层记忆结构
│   │   ├── raw/              # 短期记忆（碎片）
│   │   ├── organized/         # 长期记忆（已整理）
│   │   │   ├── facts/
│   │   │   ├── decisions/
│   │   │   └── sessions/
│   │   └── graph/            # 认知图谱
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

### 三层记忆写入流程

```
remember → raw/ (短期) → organize → organized/ (长期) + graph/ (认知)
                              ↓
                         手动/定时触发
```

### 写入触发方式

**1. 写入短期记忆（CLI）**
```bash
omega-hpc remember "用户偏好 Rust" --type fact --layer memory
omega-hpc remember "选择 Tantivy 因为无外部依赖" --type decision
```
→ 写入 `layers/memory/raw/`，不做整理

**2. 写入短期记忆（SDK）**
```rust
let mem = MemoryStore::new("memory", &registry)?;
mem.remember("用户偏好 Rust", MemoryType::Fact, None)?;
```
→ 写入 raw/，返回 raw memory id

**3. 触发整理（CLI）**
```bash
omega-hpc organize --layer memory
```
→ 读取所有 raw memories，整理后写入 organized/ + 更新 graph/

**4. SDK 会话管理 + 整理**
```rust
let session = mem.start_session("claude-code", "jerryg")?;
session.extract_facts_from_conversation(&messages)?;
// 调用 remember 多次后：
mem.organize(OrganizeOptions::default())?;
```

### 何时触发整理

- **手动触发**：`omega-hpc organize --layer memory`
- **定量触发**：raw memory 数量达到 `organize_on_count` 时建议整理
- **定时触发**：`auto_organize = true` 时按 `organize_interval` 自动整理

### 不做的事

- 不会自动将 remember 结果写入 organized memory（需要显式 organize）
- 不会自动监听对话并提取记忆（需要显式调用）
- 不会在后台运行 LLM 进行摘要（由调用方负责）
- 不会自动删除 raw memory（需要 `--delete-raw`）

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
| 记忆架构 | 三层（raw/organized/graph） | 类人记忆整理过程，形成认知体系 |
| 整理触发 | 手动 + 定量 + 定时 | 不过度自动化，保持用户控制 |

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
   - `omega-hpc remember --layer <name>` - 写入短期记忆
   - `omega-hpc recall --layer <name>` - 回忆记忆（结合图谱）
   - `omega-hpc organize --layer <name>` - 整理记忆
   - `omega-hpc graph --query <expr>` - 查询认知图谱
   - `omega-hpc forget --layer <name>` - 删除
   - `omega-hpc layers` - 列出层配置
   - `omega-hpc stat` - 统计
   - `omega-hpc gc` - 垃圾回收

5. **SDK 编程接口**
   - Rust crate 发布到 crates.io
   - 提供 `LayerRegistry`, `MemoryStore`, `SearchEngine`, `Indexer` 核心 API

### Should Have

6. **三层记忆架构**
   - RawMemoryStore: 短期记忆写入
   - Organizer: 整理期算法（去重、矛盾检测、关系提取）
   - OrganizedMemoryStore: 长期记忆存储
   - ConceptGraphStore: 认知图谱存储与遍历

7. **增量索引更新**
   - 文件变化时只更新受影响部分
   - `omega-hpc rebuild` 重建索引

8. **多项目隔离**
   - 不同项目使用不同 .hpc 目录

### Nice to Have

9. **时间旅行**
   - 查看历史索引版本

10. **记忆总结**
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

### Phase 3: 三层记忆架构

- [ ] RawMemoryStore 实现
- [ ] OrganizedMemoryStore 实现
- [ ] `remember --layer` → raw/
- [ ] `recall --layer` → organized/ + graph/
- [ ] `forget --layer` 命令

### Phase 4: 整理期 + 认知图谱

- [ ] ConceptGraphStore 实现（节点 + 边存储与遍历）
- [ ] Organizer 实现（LLM 去重、矛盾检测、关系提取）
- [ ] `omega-hpc organize --layer <name>` 命令
- [ ] `omega-hpc graph --query <expr>` 命令
- [ ] `extract_facts_from_conversation` SDK 方法

### Phase 5: SDK + 高级特性

- [ ] Rust SDK 发布
- [ ] `omega-hpc gc --layer <name>`
- [ ] 增量索引更新
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
| 远程 API 需网络 | 向量搜索/整理 离线不可用 | Candle 本地 fallback |
| 向量维度不可混用 | 切换模型需重建索引 | `omega-hpc rebuild --full` |
| Organization 依赖 LLM | 整理需网络和 API key | 使用本地 LLM 或手动整理 |
| Graph 遍历深度限制 | 深层推理可能超时 | 限制 depth 参数 |
| Content 层无自动 gc | 磁盘可能膨胀 | `omega-hpc gc` 手动清理 |
| tantivy 索引跨版本不兼容 | 升级 tantivy 需重建 | 文档标注版本要求 |
| 层继承仅在配置加载时解析 | 运行时修改继承需重启 | 配置热重载为 Phase 2+ |
| Raw memory 无限积累 | 磁盘膨胀 | 设置 `raw_retention` 并定期 organize |
