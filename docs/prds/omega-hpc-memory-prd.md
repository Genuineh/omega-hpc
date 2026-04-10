# Omega HPC Memory: Unified Memory and Knowledge Base System

## Summary

为 omega-hpc 设计一套统一记忆与知识库系统，同时服务于：
1. **Agent 长期记忆** - 跨会话保留决策、偏好、上下文
2. **外置知识库** - 以项目为单位的文档索引与检索

核心理念：**索引即记忆**。保存的是结构化索引（类比大脑皮层/元认知），内容按需加载。记忆的分层通过多个单文件实现，各司其职，不无限扩张。

## Problem

当前 omega-hpc 存在两层需求未被满足：

**记忆层缺失**
- Agent 每次会话从零开始，无法保留历史决策
- 跨会话无法复用经验（如"上次因为 X 原因选了 Y 方案"）
- 缺乏主动记忆写入机制

**知识库层缺失**
- 大量文档和技能指南无法快速检索
- 文档检索与 Agent 记忆是割裂的两套系统
- 缺乏统一的索引管理和版本追踪

## Design Principles

### 1. 索引即记忆

不存储原始内容副本，而是存储：
- **索引结构** - BM25、向量、实体槽位的组织方式
- **引用映射** - 内容在哪、如何加载
- **关联图谱** - 实体之间的关系

类比：大脑保存的是神经连接模式，而非完整经历录像。

### 2. 多文件分层

不搞单一大文件，每个文件类型有明确职责：

| 文件类型 | 职责 | 可否合并 |
|----------|------|----------|
| `.hpc.cortex` | 元认知索引 | 多个可合并 |
| `.hpc.kb/` | 知识库内容分片 | 按项目隔离 |
| `.hpc.mem` | Agent 记忆层 | 按用户/会话隔离 |
| `.hpc.sync` | 同步元数据 | 自动管理 |

### 3. Git 友好

- 索引文件是结构化文本（TOML/JSON），可 diff
- 内容通过 content-addressable 引用（hash 寻址）
- 每个文件变更可独立追踪和 review

## Architecture

### 文件层次

```
project/
├── .hpc/
│   ├── cortex/                    # 元认知层（索引）
│   │   ├── index.toml            # 主索引清单
│   │   ├── bm25.idx              # BM25 结构（二进制，可重建）
│   │   ├── entities.toml         # 实体槽位映射
│   │   └── embeddings/           # 向量分片（可外部化）
│   │       └── shard_001.vec
│   │
│   ├── kb/                       # 知识库层（内容）
│   │   └── {content-hash}.chunk # 分片内容块
│   │
│   ├── mem/                      # 记忆层（Agent 私有）
│   │   ├── sessions/            # 按会话组织
│   │   │   └── {session-id}.mem
│   │   └── users/               # 按用户组织
│   │       └── {user-id}.mem
│   │
│   └── sync/                     # 同步层
│       └── manifest.toml        # 变更追踪
```

### 检索流程

```
查询 "上次为什么选了 PostgreSQL"
    ↓
1. 查询 .hpc/cortex（索引层）
   - 找到相关实体 "PostgreSQL"
   - 找到相关决策记忆引用
    ↓
2. 如需内容，按引用加载 .hpc/kb/ 中的 chunk
    ↓
3. 如需 Agent 记忆，加载 .hpc/mem/ 中对应 .mem 文件
    ↓
返回：索引片段 + 关联记忆 + 原始文档引用
```

## Requirements

### Must Have

1. **多格式知识库索引**
   - 支持 Markdown、纯文本、代码、结构化文档
   - 自动分块（512 tokens 默认）
   - Content-addressable 存储

2. **混合检索引擎**
   - BM25 词项搜索（精确）
   - 向量相似搜索（语义）
   - 实体槽位 O(1) 查询
   - 支持组合查询

3. **RCLI 命令行**
   - `omega-hpc init` - 初始化项目记忆空间
   - `omega-hpc add` - 添加知识库内容
   - `omega-hpc search` - 混合检索
   - `omega-hpc recall` - 回忆相关记忆
   - `omega-hpc forget` - 删除记忆/索引

4. **记忆写入接口**
   - CLI 命令行写入
   - SDK 编程接口写入
   - 会话级别的记忆快照

### Should Have

5. **增量索引更新**
   - 文件变化时只更新受影响部分
   - 索引可独立重建

6. **多项目隔离**
   - 不同项目使用不同 .hpc 目录
   - 项目间可显式共享知识库

7. **时间旅行**
   - 查看历史索引版本
   - 对比不同时间点的记忆状态

### Nice to Have

8. **记忆总结与压缩**
   - LLM 生成记忆摘要
   - 过期记忆归档

9. **跨记忆库关联**
   - 建立项目间知识引用
   - 形成知识图谱

## Memory Layer Specification

### Cortex Layer (元认知)

`.hpc.cortex/` 是整个系统的"大脑皮层"，保存索引结构：

```toml
# index.toml
version = "1.0"

[meta]
created_at = "2026-04-10T00:00:00Z"
updated_at = "2026-04-10T00:00:00Z"
total_chunks = 1247
total_entities = 156

[[documents]]
id = "doc_001"
path = "docs/TODO.md"
mime_type = "text/markdown"
content_hash = "sha256:abc123..."
chunk_ids = ["chunk_001", "chunk_002"]

[[entities]]
name = "omega-hpc"
type = "project"
slots = { description = "AI Agent 操作平台" }
related_chunks = ["chunk_010"]
```

### Knowledge Layer (知识库)

`.hpc.kb/` 保存分片内容：

```
{content-hash}.chunk  # 二进制或压缩格式
```

每个 chunk 包含原始内容的片段，可通过 hash 校验完整性。

### Memory Layer (记忆)

`.hpc.mem/` 保存 Agent 的私有记忆：

```toml
# {session-id}.mem
version = "1.0"

[session]
id = "sess_20260410_001"
agent = "claude-code"
user = "jerryg"
started_at = "2026-04-10T10:00:00Z"

[[facts]]
content = "用户偏好 Rust 作为主要开发语言"
extracted_at = "2026-04-10T10:30:00Z"
confidence = 0.95

[[decisions]]
content = "选择 Tantivy 作为 BM25 引擎因为纯 Rust 实现"
context = "需要避免外部依赖"
decided_at = "2026-04-10T11:00:00Z"
```

## Evaluation Framework

### 评估维度

由于系统同时服务"文档检索"和"Agent 记忆"两种场景，评测也分两个方向：

#### A. 知识库检索评测

参考 [LoCoMo](https://arxiv.org/abs/2504.19413) 的四类问题：
| 维度 | 适用场景 |
|------|----------|
| Single-hop | 直接文档查询（"Tantivy 在哪用？"） |
| Temporal | 时间顺序查询（"上次更新是什么时候？"） |
| Multi-hop | 跨文档推理（"哪个决策导致了现在的架构？"） |
| Open-domain | 开放问答（"这个项目的设计原则是什么？"） |

#### B. 记忆保持评测

| 维度 | 描述 |
|------|------|
| 记忆召回率 | 相同上下文再次出现时能否正确回忆 |
| 干扰抵抗 | 新记忆是否覆盖/混淆旧记忆 |
| 摘要质量 | LLM 生成的记忆摘要是否准确 |

### 评测指标

| 指标 | 目标 | 说明 |
|------|------|------|
| Recall@K | > 0.85 | Top-K 结果中相关文档的比例 |
| MRR | > 0.75 | 平均倒数排名 |
| 记忆精度 | > 0.80 | Agent 记忆与事实的符合度 |
| P50 延迟 | < 30ms | 纯索引查询 |
| P95 延迟 | < 150ms | 含内容加载的查询 |

### 防作弊机制

1. **闭卷索引** - 评测数据集不得在构建索引时使用
2. **独立题库** - 题库与代码库分离，定期轮换
3. **无捷径优化** - 禁止针对评测题目添加人工规则

## Technical Decisions

| 决策点 | 选择 | 理由 |
|--------|------|------|
| 存储架构 | 多文件分层 | 职责分离、Git 友好、可独立更新 |
| 索引格式 | TOML + 二进制 | TOML 可 diff，二进制高效 |
| 向量存储 | 可插拔 | 本地（flat/HNSW）或远程 API |
| 嵌入模型 | 运行时选择 | 本地（Candle/ONNX）或远程（OpenAI） |
| 内容引用 | Content-addressable | Hash 寻址，防篡改，可去重 |

## Reference Architecture

**参考 [Memvid](https://memvid.com) 的设计理念：**
- 单文件便携性 → 多文件分层，索引独立
- 混合搜索 → 保留为核心能力

**参考 [Mem0](https://github.com/mem0ai/mem0) 的设计理念：**
- 多级记忆分层 → 实现 User/Session/Agent 三层
- LLM 驱动的记忆提取 → 作为 Should Have

**区别于两者：**
- 不是单一大文件
- 索引与内容分离
- 显式知识库层支持外部文档

## Timeline

- **Phase 1**: 基础框架 + 知识库索引（MVP）
  - 项目初始化 + add 命令
  - BM25 索引 + search 命令
  - Cortex 层实现

- **Phase 2**: 检索增强 + 向量搜索
  - 向量引擎集成
  - 混合搜索
  - 实体提取

- **Phase 3**: 记忆层 + SDK 集成
    - Memory 层实现
    - SDK 编程接口
    - recall 命令

- **Phase 4**: 高级特性
  - 增量更新
  - 时间旅行
  - 记忆总结

- **Phase 5**: 评测体系
  - 题库构建
  - 评测框架
  - 性能基准

## Open Questions

1. 向量引擎具体实现（本地 HNSW 还是远程？）
2. SDK 接口的记忆写入语义（何时触发记忆提取？）
3. 知识库是否需要版本分支？
4. 多 Agent 共享记忆的权限模型？

## Related Documents

- `docs/specs/omega-hpc-memory-spec.md` - 详细技术规格
- `docs/guide/omega-hpc-memory-guide.md` - 使用指南
