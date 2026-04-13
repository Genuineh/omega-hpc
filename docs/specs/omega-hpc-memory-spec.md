# Omega HPC Memory Technical Specification

**状态**: Active

---

## Overview

omega-hpc-memory 是统一记忆与知识库系统，同时服务于：
- Agent 长期记忆（跨会话保留决策、偏好）
- 外部知识库（项目文档的索引与检索）

**核心理念：索引即记忆**。系统保存结构化索引，内容按需加载。

**认知架构**：系统模拟人类记忆的三层结构——短期记忆（碎片）→ 整理期（巩固）→ 长期记忆 + 认知图谱（结构化知识）。记忆不是简单堆放，而是经过组织过程形成认知体系。

**集成方式**：仅 CLI 和 SDK，不提供 MCP。

---

## Architecture

### Configurable Layer System

系统通过 **用户可配置的数据层（Layer）** 组织存储与检索。每个 Layer 描述一类数据的索引策略、存储方式和召回行为。Layer 通过 `.hpc.toml` 定义，支持继承。

**Layer 类型**：

| 类型 | 说明 | 存储方式 | 写入方式 |
|------|------|----------|----------|
| `content` | 外部文件内容 | content-addressable chunk | `add` 命令 |
| `memory` | Agent 结构化记忆 | TOML 文件 | `remember` 命令 / SDK |

**Layer 属性**：

| 属性 | 说明 | 必填 |
|------|------|------|
| `name` | 唯一标识符 | 是 |
| `description` | 人类可读描述 | 否 |
| `type` | `content` 或 `memory` | 是 |
| `abstract` | 是否为抽象模板层（无存储） | 否 |
| `extends` | 继承父层配置 | 否 |
| `priority` | 召回优先级（数字越小越优先） | 是 |
| `indexing` | 索引配置（分块、glob 等） | 否 |
| `recall` | 召回配置（boost、策略等） | 是 |

**Layer 继承**：

子层通过 `extends` 继承父层全部配置，仅覆盖差异字段。抽象层（`abstract = true`）仅作模板，不创建存储目录。

```
解析规则：
  1. 找到 extends 指向的父层
  2. 深度合并：父层配置为底，子层字段覆盖
  3. 标量：子覆盖父
  4. 数组：子替换父（不合并）
  5. 表：递归合并
  6. 继承链支持多级（A → B → C），禁止循环
```

### Directory Structure

```
.hpc/                          # 项目记忆根目录
├── cortex/                    # 元认知层（全局索引）
│   ├── meta.toml              # 全局元数据
│   ├── layers/                # 按层拆分的索引
│   │   ├── {layer_name}/
│   │   │   ├── index.toml     # 层级文档清单
│   │   │   └── bm25/          # 层级 BM25 索引（tantivy）
│   │   └── ...
│   ├── entities.toml          # 实体槽位
│   └── embeddings/            # 向量分片（跨层共享）
│       └── shard_*.bin
│
├── layers/                    # 用户定义的数据层
│   ├── {content-layer}/       # content 类型：chunk 存储
│   │   └── {sha256}.chunk
│   ├── {memory-layer}/        # memory 类型：三层记忆结构
│   │   ├── raw/               # 短期记忆（碎片，未整理）
│   │   │   └── {timestamp}_{uuid}.raw
│   │   ├── organized/         # 长期记忆（已整理）
│   │   │   ├── facts/
│   │   │   │   └── {fact_id}.toml
│   │   │   ├── decisions/
│   │   │   │   └── {decision_id}.toml
│   │   │   └── sessions/
│   │   │       └── {session-id}.toml
│   │   └── graph/             # 认知图谱（概念 + 关系）
│   │       └── {graph_id}.json
│   └── ...
│
└── sync/
    └── manifest.toml          # 变更追踪
```

**注意**：抽象层（`abstract = true`）不创建 `layers/` 下的存储目录，也不创建 `cortex/layers/` 下的索引目录。

### Default Layers

当 `.hpc.toml` 未定义任何 layer 时，系统使用内置默认配置，行为与早期版本兼容：

```toml
[[layer]]
name = "kb"
description = "知识库"
type = "content"
priority = 1

[layer.indexing]
chunk_size = 512

[layer.recall]
boost = 1.0
max_results = 10
strategy = "hybrid"

[[layer]]
name = "mem"
description = "Agent 记忆（三层结构）"
type = "memory"
priority = 2

[layer.memory]
raw_retention = 100       # raw memory 保留上限
organize_on_count = 50    # raw 达到此数量时建议整理

[layer.recall]
boost = 1.0
max_results = 5
strategy = "hybrid"
weight_bm25 = 0.3
weight_graph = 0.7
```

### Cortex Layer (`.hpc/cortex/`)

**存储**：索引数据，不存储原始内容。

```toml
# meta.toml
version = "1.0"
created_at = "2026-04-10T00:00:00Z"
updated_at = "2026-04-10T00:00:00Z"
layers = ["code", "docs", "decisions", "memory"]
total_docs = 42
total_chunks = 1247
total_entities = 156
```

```toml
# layers/{layer_name}/index.toml
[layer]
name = "code"
total_docs = 15
total_chunks = 342
updated_at = "2026-04-11T10:00:00Z"

# layers/{layer_name}/bm25/ 由 tantivy 自管理
```

```toml
# layers/{layer_name}/docs/doc_001.toml
id = "doc_001"
layer = "code"
path = "src/main.rs"
mime_type = "text/rust"
content_hash = "sha256:abc123..."
chunk_count = 3
status = "active"
chunks = ["chunk_001", "chunk_002", "chunk_003"]
```

### Content Layer (`.hpc/layers/{name}/`)

Content-addressable 内容分片：

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

### Memory Layer (`.hpc/layers/{name}/`)

Memory 层采用**三层记忆架构**，模拟人类记忆的短期→整理→长期过程：

```
remember → raw/ (碎片) ──整理期──→ organized/ (结构化) ──提取──→ graph/ (认知)
                                                     ↑
                                             recall 时可组合查询
```

#### 1. Raw Memory（短期记忆）

`raw/` 目录存储未整理的原始记忆碎片：

```toml
# {timestamp}_{uuid}.raw
version = "1.0"
layer = "memory"
stored_at = "2026-04-11T10:00:00Z"

[memory]
type = "fact"          # fact | decision | observation
content = "用户好像偏好 Rust"
confidence = 0.6        # 原始置信度，可能有误
source = "conversation"
session_id = "sess_001"

[tags]
raw = true
organized = false
```

**特性**：
- 快速写入，不做结构化处理
- 可能矛盾、低置信度
- 积累到一定数量后需要整理

#### 2. Organized Memory（长期记忆）

`organized/` 存储整理后的结构化记忆：

```toml
# facts/{fact_id}.toml
version = "1.0"
layer = "memory"
type = "fact"
id = "fact_002"

[content]
text = "用户偏好 Rust 作为主要开发语言"
summary = "Rust 是用户的首选语言"

[metadata]
confidence = 0.95       # 整理后提升的置信度
source = "conversation"
organized_at = "2026-04-12T08:00:00Z"
raw_refs = ["raw_001", "raw_003"]  # 来源的 raw memory

[project_context]
related_layers = ["code"]
relevant_files = ["src/main.rs"]
```

```toml
# decisions/{decision_id}.toml
version = "1.0"
layer = "memory"
type = "decision"
id = "dec_002"

[content]
text = "选择 Tantivy 作为 BM25 引擎"
rationale = "因为纯 Rust 实现，无外部依赖"

[context]
triggered_by = "需要为项目选择检索引擎"
alternatives_considered = ["whoosh", "bluge"]
outcome = "采用 Tantivy"

[metadata]
confidence = 0.98
decided_at = "2026-04-10T11:00:00Z"
revised = false
organized_at = "2026-04-12T08:00:00Z"
raw_refs = ["raw_005"]
```

#### 3. Concept Graph（认知图谱）

`graph/` 存储从 organized memory 中提取的概念关系图：

```json
// {graph_id}.json
{
  "version": "1.0",
  "layer": "memory",
  "updated_at": "2026-04-12T08:00:00Z",
  "nodes": [
    {
      "id": "concept_rust",
      "type": "technology",
      "name": "Rust",
      "description": "用户的首选开发语言",
      "confidence": 0.95
    },
    {
      "id": "concept_tantivy",
      "type": "technology",
      "name": "Tantivy",
      "description": "纯 Rust 实现的 BM25 搜索引擎"
    },
    {
      "id": "decision_002",
      "type": "decision",
      "name": "选择 Tantivy",
      "description": "选用 Tantivy 而非其他搜索引擎"
    },
    {
      "id": "concept_bm25",
      "type": "concept",
      "name": "BM25",
      "description": "检索算法"
    }
  ],
  "edges": [
    {
      "from": "decision_002",
      "to": "concept_tantivy",
      "relation": "selects"
    },
    {
      "from": "concept_tantivy",
      "to": "concept_bm25",
      "relation": "implements"
    },
    {
      "from": "concept_rust",
      "to": "concept_tantivy",
      "relation": "preferred_for",
      "weight": 0.9
    },
    {
      "from": "decision_002",
      "to": "concept_rust",
      "relation": "influenced_by"
    }
  ]
}
```

**节点类型**：`technology` | `concept` | `decision` | `person` | `project` | `rule`

**边关系**：`depends_on` | `implements` | `selects` | `related_to` | `influenced_by` | `contradicts` | `temporal_before` | `preferred_for`

#### Organization Process（整理期）

整理期是定期将 raw memory 转化为 organized memory 并更新 graph 的过程：

```
整理触发条件（满足任一）：
  - raw memory 数量 >= layer.memory.raw_retention
  - 手动执行 `omega-hpc organize`
  - 定时触发（每日/每周）

整理算法：
  1. 读取所有未整理的 raw memory
  2. LLM 去重、合并矛盾、提取关键信息
  3. 生成 structured facts + decisions
  4. 从 structured facts/decisions 提取概念和关系
  5. 更新/创建 graph 节点和边
  6. 标记 raw memory 为已整理（可选删除）
  7. 记录整理日志
```

**Memory vs Content Layer 区别**：

| 维度 | Content Layer | Memory Layer |
|------|---------------|--------------|
| 内容类型 | 文档分片 | 三层记忆（raw/organized/graph） |
| 来源 | 外部文件 | Agent 写入 + 整理提取 |
| 格式 | 二进制 chunk | TOML + JSON |
| 写入命令 | `add` | `remember` |
| 整理命令 | 无 | `organize` |
| 查询命令 | `search` | `recall` |
| 图谱查询 | 无 | `omega-hpc graph query` |

---

## Core Components

### 1. LayerRegistry

管理所有层的配置、继承解析和生命周期。

```rust
pub struct LayerRegistry {
    layers: HashMap<String, ResolvedLayer>,
}

pub struct ResolvedLayer {
    name: String,
    description: String,
    layer_type: LayerType,
    abstract_layer: bool,
    priority: u32,
    storage_path: PathBuf,
    indexing: IndexingConfig,
    recall: RecallConfig,
}

pub enum LayerType {
    Content,
    Memory,
}

impl LayerRegistry {
    pub fn from_config(config: &HpcConfig) -> Result<Self>;
    pub fn get(&self, name: &str) -> Option<&ResolvedLayer>;
    pub fn content_layers(&self) -> Vec<&ResolvedLayer>;
    pub fn memory_layers(&self) -> Vec<&ResolvedLayer>;
    pub fn layers_by_priority(&self) -> Vec<&ResolvedLayer>;
}
```

### 2. Indexer

```rust
pub struct Indexer {
    cortex_path: PathBuf,
    registry: LayerRegistry,
}

impl Indexer {
    pub fn add_document(&mut self, path: &Path, layer: &str) -> Result<DocId>;
    pub fn add_directory(&mut self, path: &Path, layer: &str) -> Result<()>;
    pub fn rebuild_index(&mut self, layer: Option<&str>) -> Result<()>;
    pub fn mark_stale(&mut self, layer: &str, doc_id: &str) -> Result<()>;
}
```

### 3. SearchEngine

支持跨层搜索和单层搜索，跨层时使用 Recall Fusion 策略。

```rust
pub struct SearchEngine {
    cortex_path: PathBuf,
    registry: LayerRegistry,
    embedding_client: Box<dyn EmbeddingClient>,
}

impl SearchEngine {
    pub fn search(&self, query: &str, layers: &[&str], mode: SearchMode) -> Result<Vec<SearchHit>>;
    pub fn search_layer(&self, query: &str, layer: &str, limit: usize) -> Result<Vec<SearchHit>>;
    pub fn search_all(&self, query: &str, limit: usize) -> Result<Vec<SearchHit>>;
}

pub enum SearchMode {
    Hybrid { alpha: f32 },
    BM25,
    Vector,
}
```

### 4. RawMemoryStore

短期记忆存储，负责接收 `remember` 写入并直接存入 raw 层：

```rust
pub struct RawMemoryStore {
    layer_name: String,
    raw_path: PathBuf,
}

impl RawMemoryStore {
    pub fn save_raw(&mut self, memory: &RawMemory) -> Result<String>;  // returns id
    pub fn list_unorganized(&self) -> Result<Vec<RawMemory>>;
    pub fn mark_organized(&mut self, ids: &[String]) -> Result<()>;
    pub fn delete_raw(&mut self, id: &str) -> Result<()>;
    pub fn raw_count(&self) -> Result<usize>;
}
```

### 5. Organizer

整理器，执行整理期逻辑，将 raw memory 转化为 organized memory 和 graph：

```rust
pub struct Organizer {
    layer_name: String,
    raw_store: RawMemoryStore,
    organized_store: OrganizedMemoryStore,
    graph_store: ConceptGraphStore,
    llm_client: Box<dyn LlmClient>,
}

pub struct OrganizeOptions {
    pub force: bool,         // 即使未达阈值也整理
    pub delete_raw: bool,    // 整理后删除 raw memory
    pub dry_run: bool,       // 仅模拟，不写入
}

impl Organizer {
    pub fn organize(&mut self, options: OrganizeOptions) -> Result<OrganizeReport>;
    pub fn needs_organization(&self) -> Result<bool>;
}

pub struct OrganizeReport {
    pub facts_created: usize,
    pub decisions_created: usize,
    pub nodes_created: usize,
    pub edges_created: usize,
    pub raw_processed: usize,
    pub conflicts_resolved: usize,
}
```

**整理算法流程**：

```
organize():
  1. raw_store.list_unorganized() → 获取所有未整理 raw memory
  2. 对 raw memories 分组（按 type: fact/decision）
  3. LLM 调用（批量）：
     - 去重：识别相似的 raw memories，合并
     - 矛盾检测：识别相互矛盾的 memories，标记
     - 提精华：生成高置信度的 structured 输出
     - 关系提取：从 facts/decisions 中提取概念和关系
  4. organized_store.save_fact() / save_decision() 写入
  5. graph_store.upsert_nodes() / upsert_edges() 更新图谱
  6. raw_store.mark_organized() 标记（或 delete_raw）
  7. 返回 OrganizeReport
```

### 6. ConceptGraphStore

认知图谱存储，维护项目的概念关系网络：

```rust
pub struct ConceptGraphStore {
    layer_name: String,
    graph_path: PathBuf,
    index: GraphIndex,  // 用于快速图查询的索引
}

#[derive(Serialize, Deserialize)]
pub struct ConceptGraph {
    pub nodes: Vec<ConceptNode>,
    pub edges: Vec<ConceptEdge>,
}

pub struct ConceptNode {
    pub id: String,
    pub node_type: NodeType,  // technology | concept | decision | person | project | rule
    pub name: String,
    pub description: String,
    pub confidence: f32,
    pub metadata: HashMap<String, String>,
}

pub struct ConceptEdge {
    pub from: String,
    pub to: String,
    pub relation: EdgeRelation,  // depends_on | implements | selects | related_to | ...
    pub weight: f32,
}

impl ConceptGraphStore {
    pub fn upsert_node(&mut self, node: &ConceptNode) -> Result<()>;
    pub fn upsert_edge(&mut self, edge: &ConceptEdge) -> Result<()>;
    pub fn query(&self, query: &GraphQuery) -> Result<Vec<GraphResult>>;
    pub fn get_node(&self, id: &str) -> Result<Option<ConceptNode>>;
    pub fn get_neighbors(&self, node_id: &str, relation: Option<EdgeRelation>) -> Result<Vec<ConceptNode>>;
    pub fn traverse(&self, start: &str, relation: EdgeRelation, depth: usize) -> Result<Vec<PathResult>>;
}

pub enum GraphQuery {
    Traverse { start: String, relation: EdgeRelation, depth: usize },
    Pattern { nodes: Vec<String>, edges: Vec<EdgeRelation> },
    Search { concept: String },
}

pub struct PathResult {
    pub path: Vec<String>,  // node ids
    pub total_weight: f32,
}
```

### 7. OrganizedMemoryStore

长期记忆存储，负责整理后记忆的 CRUD：

```rust
pub struct OrganizedMemoryStore {
    layer_name: String,
    organized_path: PathBuf,
    bm25_index: tantivy::Index,
}

impl OrganizedMemoryStore {
    pub fn save_fact(&mut self, fact: &OrganizedFact) -> Result<()>;
    pub fn save_decision(&mut self, decision: &OrganizedDecision) -> Result<()>;
    pub fn get_fact(&self, id: &str) -> Result<Option<OrganizedFact>>;
    pub fn get_decision(&self, id: &str) -> Result<Option<OrganizedDecision>>;
    pub fn list_facts(&self) -> Result<Vec<OrganizedFact>>;
    pub fn list_decisions(&self) -> Result<Vec<OrganizedDecision>>;
    pub fn recall(&self, query: &str, limit: usize) -> Result<Vec<MemoryHit>>;
}
```

### 8. MemoryStore（组合层）

组合 RawMemoryStore + Organizer + OrganizedMemoryStore + ConceptGraphStore，提供统一接口：

```rust
pub struct MemoryStore {
    layer_name: String,
    raw_store: RawMemoryStore,
    organized_store: OrganizedMemoryStore,
    graph_store: ConceptGraphStore,
    organizer: Organizer,
}

impl MemoryStore {
    pub fn remember(&mut self, content: &str, memory_type: MemoryType, session: Option<&str>) -> Result<String> {
        // 写入 raw memory
    }

    pub fn recall(&self, query: &str, limit: usize) -> Result<Vec<MemoryHit>> {
        // 从 organized memory + graph 组合查询
    }

    pub fn organize(&mut self, options: OrganizeOptions) -> Result<OrganizeReport> {
        self.organizer.organize(options)
    }

    pub fn graph_query(&self, query: &GraphQuery) -> Result<Vec<GraphResult>> {
        self.graph_store.query(query)
    }

    pub fn needs_organization(&self) -> Result<bool> {
        self.organizer.needs_organization()
    }
}

### 5. Recall Fusion

跨层搜索时，合并多层结果的策略。

```rust
pub struct RecallFusion {
    registry: LayerRegistry,
}

impl RecallFusion {
    pub fn merge(&self, per_layer: HashMap<String, Vec<SearchHit>>, global_limit: usize) -> Vec<SearchHit>;
}
```

**Fusion 流程**：

```
search(query, layers=all):
  1. 对每个目标层并行执行搜索
  2. 每层结果应用 recall.boost 调分：hit.score *= layer.boost
  3. 每层截取至 recall.max_results
  4. 合并所有层结果
  5. 按 score 降序排列，score 相近（差距 < 5%）时按 priority 升序排
  6. 截取至 global_limit
```

**Boost 语义**：
- `boost > 1.0`：放大该层结果分数，提升排名
- `boost = 1.0`：原始分数
- `boost < 1.0`：缩小该层结果分数，降低排名

**示例**（code boost=1.5, docs boost=1.0, decisions boost=2.0）：

```
Query: "为什么选择 Tantivy"

Layer decisions (boost=2.0):
  hit_a: score=0.6 → 0.6 × 2.0 = 1.2

Layer code (boost=1.5):
  hit_b: score=0.7 → 0.7 × 1.5 = 1.05

Layer docs (boost=1.0):
  hit_c: score=0.8 → 0.8 × 1.0 = 0.8

Merged: [hit_a(1.2), hit_b(1.05), hit_c(0.8)]
```

### 6. extract_facts_from_conversation 行为契约

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

### 7. EmbeddingClient Trait

```rust
pub trait EmbeddingClient: Send + Sync {
    fn embed(&self, texts: &[&str]) -> Result<Vec<Vec<f32>>>;
}

pub struct OpenAIEmbedding {
    api_key: String,
    model: String,
}
```

---

## CLI Commands

### init

```bash
omega-hpc init [OPTIONS] [PATH]
```

初始化项目记忆空间，创建 `.hpc/` 目录结构。根据 `.hpc.toml` 中的 layer 定义创建对应目录。

Options:
- `--path <PATH>` 记忆空间路径 (default: `.hpc`)
- `--force` 强制覆盖
- `--template <NAME>` 使用预设模板: `default`, `code`, `minimal`

### add

```bash
omega-hpc add [OPTIONS] <PATH>
```

添加内容到指定层。

Options:
- `--layer <NAME>` 目标层名称 (default: 按 glob 匹配第一个 content 层)
- `--recursive, -r` 递归添加
- `--glob <PATTERN>` 文件过滤（覆盖层配置中的 glob）
- `--chunk-size <N>` 分块大小（覆盖层配置）

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
code            content  1         1.5    hybrid    1024
docs            content  2         1.0    hybrid    512
decisions       content  3         2.0    bm25      256
memory          memory   4         0.8    bm25      -

$ omega-hpc layers --verbose

[code]
  description: "项目源代码"
  type: content
  priority: 1
  extends: (none)
  abstract: false
  indexing:
    chunk_size: 1024
    glob: ["*.rs", "*.ts", "*.py"]
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

### stat

```bash
omega-hpc stat [OPTIONS]
```

显示统计。

Options:
- `--layer <NAME>` 指定层（默认显示所有层）

### rebuild

```bash
omega-hpc rebuild [OPTIONS]
```

重建索引。

Options:
- `--layer <NAME>` 仅重建指定层
- `--full` 全部重建

### gc

```bash
omega-hpc gc [OPTIONS]
```

垃圾回收，清理各 content 层中未被索引引用的孤立 chunk 文件。

Options:
- `--layer <NAME>` 仅清理指定层（默认所有 content 层）
- `--dry-run` 仅报告可回收空间，不执行删除
- `--all` 同时清理 Cortex 中标记为 stale 的索引条目

**回收策略**：
1. 遍历每个 content 层的 chunk 文件
2. 交叉比对 Cortex 中该层的 `content_hash` 引用
3. 无引用的 chunk 标记为可回收
4. 删除可回收 chunk 并报告释放空间

**安全保证**：
- 默认 dry-run 模式，需显式去掉 `--dry-run` 才执行删除
- 被活跃索引引用的 chunk 不会被误删
- 删除前记录日志到 `.hpc/sync/manifest.toml`

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
$ omega-hpc organize --layer memory

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
- `--query <QUERY>` 图查询表达式
- `--node <ID>` 查看指定节点
- `--neighbors <ID>` 查看节点的邻居
- `--traverse <ID> --relation <REL> --depth <N>` 按关系遍历

```bash
# 搜索概念
omega-hpc graph --query "Rust"

# 查看节点详情
omega-hpc graph --node concept_tantivy

# 查看某节点的邻居
omega-hpc graph --neighbors concept_tantivy

# 沿 selects 关系遍历 2 层
omega-hpc graph --traverse decision_002 --relation selects --depth 2
```

**图查询表达式语法**：

```
omega-hpc graph --query "<concept>"
  # 模糊搜索节点名称

omega-hpc graph --query "concept_a --[relation]-> concept_b"
  # 查找从 A 沿 relation 指向 B 的路径

omega-hpc graph --query "<concept> --[]-> *"
  # 查找与 concept 直接相连的所有节点
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

### `.hpc.toml` 完整结构

```toml
[hpc]
cortex_path = ".hpc"

[embedding]
provider = "openai"
model = "text-embedding-3-small"
api_key = "${OPENAI_API_KEY}"

[embedding.local]
model = "sentence-transformers/all-MiniLM-L6-v2"
dim = 384
cache_dir = "~/.cache/omega-hpc/models"

[vector]
store = "flat"  # flat | hnsw (Phase 2)
dim = 1536

[search]
default_limit = 10
hybrid_alpha = 0.7

# ── Layer 定义 ──

[[layer]]
name = "code"
description = "项目源代码基础模板"
type = "content"
abstract = true

[layer.indexing]
chunk_size = 1024
overlap = 64
strategy = "line"    # line | paragraph | fixed
embed = true

[layer.recall]
boost = 1.5
max_results = 5
strategy = "hybrid"
weight_bm25 = 0.6
weight_vector = 0.4

[[layer]]
name = "rust"
description = "Rust 源代码"
extends = "code"
priority = 1

[layer.indexing]
glob = ["*.rs"]

[[layer]]
name = "python"
description = "Python 源代码"
extends = "code"
priority = 1

[layer.indexing]
glob = ["*.py", "*.pyi", "*.pyx"]

[[layer]]
name = "docs"
description = "项目文档"
type = "content"
priority = 2

[layer.indexing]
chunk_size = 512
glob = ["*.md", "*.txt", "*.rst"]
overlap = 32
embed = true

[layer.recall]
boost = 1.0
max_results = 10
strategy = "hybrid"
weight_bm25 = 0.4
weight_vector = 0.6

[[layer]]
name = "decisions"
description = "架构决策记录"
type = "content"
priority = 0

[layer.indexing]
chunk_size = 256
glob = ["docs/decisions/*.md"]
overlap = 0
embed = false

[layer.recall]
boost = 2.0
max_results = 3
strategy = "bm25"

[[layer]]
name = "memory"
description = "Agent 认知记忆（三层结构）"
type = "memory"
priority = 3

[layer.memory]
raw_retention = 100       # raw memory 保留上限
organize_on_count = 50    # raw 达到此数量时建议整理

[layer.recall]
boost = 0.8
max_results = 5
strategy = "hybrid"
weight_bm25 = 0.3
weight_graph = 0.7         # recall 时结合图谱推理
```

### Layer 配置字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 唯一标识符，用于 CLI `--layer` 和目录名 |
| `description` | string | 否 | 人类可读描述 |
| `type` | enum | 是 | `content` 或 `memory` |
| `abstract` | bool | 否 | `true` 则不创建存储，仅作继承模板 |
| `extends` | string | 否 | 父层名称，继承全部配置 |
| `priority` | u32 | 是 | 召回优先级，数字越小越优先（0 最高） |

### Indexing 配置（content 类型层）

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `chunk_size` | u32 | 512 | 分块大小（tokens） |
| `glob` | string[] | ["*"] | 文件匹配模式 |
| `overlap` | u32 | 0 | 分块重叠（tokens） |
| `strategy` | enum | "fixed" | 分块策略：`fixed`、`line`（按行边界）、`paragraph`（按段落边界） |
| `embed` | bool | true | 是否生成向量嵌入 |

### Recall 配置（所有层类型）

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `boost` | f32 | 1.0 | 分数放大系数，跨层搜索时应用 |
| `max_results` | u32 | 10 | 单层最大返回数 |
| `strategy` | enum | "hybrid" | 检索策略：`bm25`、`vector`、`hybrid` |
| `weight_bm25` | f32 | 0.5 | hybrid 模式下 BM25 权重 |
| `weight_vector` | f32 | 0.5 | hybrid 模式下向量权重 |
| `weight_graph` | f32 | 0.0 | recall 时图谱推理权重（memory 层） |

### Memory 配置（memory 类型层）

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `raw_retention` | u32 | 100 | raw memory 保留上限，超出后强制整理 |
| `organize_on_count` | u32 | 50 | 建议整理的 raw 数量阈值 |
| `auto_organize` | bool | false | 是否自动整理（定时） |
| `organize_interval` | string | "daily" | 自动整理间隔：`hourly`、`daily`、`weekly` |

### Init Templates

`omega-hpc init --template <NAME>` 提供预设层配置：

| 模板 | 层定义 | 适用场景 |
|------|--------|----------|
| `default` | kb + mem | 通用项目 |
| `code` | code(abstract) → rust/python + docs + decisions + memory | 代码密集项目 |
| `minimal` | single content + memory | 轻量项目 |

---

## Performance Targets

| Metric | Phase 1 | Phase 2 |
|--------|---------|---------|
| Index speed | 5,000 chunks/sec | 5,000 chunks/sec |
| Search P50 (BM25) | < 20ms | < 20ms |
| Search P50 (vector) | - | < 100ms (网络延迟) |
| Search P95 | < 100ms | < 300ms |
| Memory (1M chunks) | < 500MB | < 500MB |
| Cross-layer fusion | < 5ms overhead | < 5ms overhead |
| Organization (per raw) | < 500ms | < 200ms |
| Graph traversal (depth 3) | < 50ms | < 20ms |

---

## Error Handling

| Error | Code | Handling |
|-------|------|----------|
| Invalid cortex | E0001 | 引导 `omega-hpc rebuild` |
| Chunk not found | E0002 | 提示重新 `add` |
| Hash mismatch | E0003 | 标记 chunk 为 stale |
| Embedding API error | E0101 | 返回错误，不阻塞搜索 |
| Out of memory | E0004 | 降级到 BM25 only |
| Unknown layer name | E0201 | 提示可用层列表 |
| Abstract layer used | E0202 | 提示使用具体子层 |
| Circular inheritance | E0203 | 配置加载时检测并报错 |
| Organization in progress | E0301 | 拒绝并发 organize |
| Graph query failed | E0302 | 图查询语法错误或超时 |
| LLM organization failed | E0303 | 整理时 LLM 调用失败，保留 raw |

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
