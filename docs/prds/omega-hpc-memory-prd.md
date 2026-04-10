# Omega HPC Memory: Persistent Memory System for AI Agents

## Summary

为 omega-hpc 设计一套基于 Rust 的持久化记忆系统，通过单一 `.omega` 文件存储多元项目数据，提供 RCLI 命令行检索接口，实现快速、高效、可移植的 AI Agent 记忆层。

## Problem

当前 omega-hpc 作为 AI Agent 操作指南项目，缺乏有效的知识持久化和检索能力：
- Agent 每次会话无法保留上下文记忆
- 跨会话无法复用历史决策和经验
- 大量文档和技能指南无法快速检索
- 缺乏统一的记忆存储和查询接口

## Users

- AI Agents (Claude Code, OpenCode 等)
- 开发者 (通过 RCLI 查询项目知识)
- 其他工具集成 (通过 SDK)

## Requirements

### Must Have

1. **单文件持久化存储**
   - 使用 `.omega` 格式存储所有记忆数据
   - 零外部依赖，离线可用
   - 支持 Git 提交和同步

2. **混合检索引擎**
   - BM25 词项搜索 (快速、精确)
   - 向量相似度搜索 (语义理解)
   - 混合模式自动平衡

3. **RCLI 命令行接口**
   - `omega-hpc init` - 初始化记忆库
   - `omega-hpc add <file>` - 添加文档
   - `omega-hpc search <query>` - 语义搜索
   - `omega-hpc find --exact <pattern>` - 精确匹配
   - `omega-hpc stat` - 查看统计信息

4. **多格式文件支持**
   - Markdown (.md)
   - 纯文本 (.txt)
   - 代码文件 (.rs, .ts, .py, .js 等)
   - 结构化文档 (.yaml, .json)

### Should Have

5. **实体提取与 O(1) 查询**
   - 自动提取实体 (人名、概念、技术)
   - 槽位存储实现常量时间查找

6. **时间旅行调试**
   - 记录检索会话历史
   - 支持回放和参数调优

7. **增量索引更新**
   - 文件变化时自动增量更新
   - 无需全量重建索引

### Nice to Have

8. **LLM 驱动的记忆总结**
   - 自动生成文档摘要
   - 关键信息抽取

9. **多记忆空间**
   - User Memory / Session Memory / Agent Memory 分层

## Architecture

```
omega-hpc-memory/
├── omega-hpc-memory-core/      # 核心库 (无外部依赖)
│   ├── src/
│   │   ├── lib.rs
│   │   ├── storage/        # .omega 文件格式
│   │   ├── index/         # BM25 + Vector 索引
│   │   ├── entity/        # 实体提取与槽位索引
│   │   └── search/        # 混合搜索引擎
│   └── Cargo.toml
│
├── omega-hpc-memory-cli/       # RCLI 应用
│   ├── src/
│   │   ├── main.rs
│   │   └── commands/      # 子命令实现
│   └── Cargo.toml
│
└── omega-hpc-memory-sdk/       # SDK (可选 Rust crate)
    ├── src/
    └── Cargo.toml
```

### .omega 文件格式

```
+------------------+
| Header (Magic)   |  8 bytes: "OMEGA001"
+------------------+
| Version          |  2 bytes
+------------------+
| Metadata Size    |  4 bytes
+------------------+
| Metadata JSON    |  N bytes
+------------------+
| Documents        |  N bytes (ID + Content + Chunks)
+------------------+
| BM25 Index       |  N bytes
+------------------+
| Vector Store     |  N bytes (扁平向量 + HNSW)
+------------------+
| Entity Index     |  N bytes (槽位映射)
+------------------+
| Checksum         |  32 bytes (SHA-256)
+------------------+
```

## Evaluation Framework

### 评估原则

评估体系必须满足以下原则，防止通过取巧方式通过评测：

1. **闭卷评测** - 评测数据集不得包含在训练集或索引中
2. **真实任务** - 题目必须来自真实使用场景，而非人工构造
3. **不可预测性** - 评测题目在系统设计时未知，防止针对特定题目优化
4. **多维度度量** - 同时评估准确性、延迟、Token 消耗等多个维度
5. **可复现性** - 评测结果可被独立复现

### Benchmark 评估体系

#### 1. LoCoMo Benchmark (Long-Term Conversation Memory)

**来源**: [Mem0 论文](https://arxiv.org/abs/2504.19413) 使用的基准测试

**评估维度**:
| 维度 | 描述 |
|------|------|
| Single-hop | 单点事实查询 |
| Temporal | 时间相关查询 |
| Multi-hop | 多跳推理查询 |
| Open-domain | 开放域问答 |

**评估指标**:
- **LLM-as-Judge Score** - 使用 LLM 评判答案正确性 (0-1)
- **BLEU Score** - 与标准答案的 n-gram 相似度
- **F1 Score** - 精确率和召回率的调和平均

**性能指标**:
- **Token 消耗** - 单次查询消耗的 Token 数
- **P50/P95 延迟** - 检索延迟分布

**参考基线**:
| 系统 | LLM Score | Token 消耗 | P95 延迟 |
|------|-----------|------------|----------|
| Full-context | ~72.9% | ~26K | ~17.12s |
| OpenAI Memory | ~52.9% | ~8K | ~4.2s |
| Mem0 | ~66.9% | ~1.8K | ~1.44s |

#### 2. Omega HPC Memory 评测集

针对 omega-hpc 特定场景的评测集：

**文档检索评测**:
- 跨文档引用检索
- 代码-文档关联检索
- 决策历史追溯

**记忆持久化评测**:
- 跨会话记忆保留率
- 增量更新一致性
- 实体记忆准确性

#### 3. 鲁棒性测试

| 测试类型 | 描述 |
|----------|------|
| 噪声注入 | 在输入中加入无关信息，测试检索抗干扰能力 |
| 长上下文 | 超长输入下的检索性能 |
| 部分匹配 | 输入与记忆不完全匹配时的召回能力 |
| 时序推理 | 基于时间顺序的推理查询 |

#### 4. 评测执行框架

```bash
# 评测命令
omega-hpc eval --benchmark locomo [--output results.json]
omega-hpc eval --benchmark omega-hpc [--output results.json]
omega-hpc eval --benchmark robustness [--output results.json]

# 生成评测报告
omega-hpc eval --report [--format html|markdown]
```

### 评测数据集管理

**约束**:
1. 评测数据集独立存储，不参与索引构建
2. 每次评测使用不同的随机种子
3. 评测题目在提交前保持隐藏

**防止作弊机制**:
- 评测数据集定期更新
- 支持第三方独立评测
- 公开评测子集用于本地验证

## Technical Decisions

| 决策点 | 选择 | 理由 |
|--------|------|------|
| 存储格式 | 单一 .omega 文件 | 便于 Git 追踪、复制、同步 |
| 向量引擎 | 扁平向量 + 简单索引 | 避免外部依赖，保持离线可用 |
| 嵌入模型 | 可配置 (本地/远程) | 支持离线 + 云端灵活切换 |
| 分词器 | Unicode aware + 代码专用 | 支持多语言和代码片段 |
| 评测标准 | LoCoMo + 自定义 | 行业标准 + 场景适配 |

## Reference Architecture

参考 [Memvid](https://memvid.com) 的设计理念：
- 单文件便携性
- 混合搜索 (BM25 + Vector)
- O(1) 实体查找
- 时间旅行调试

参考 [Mem0](https://github.com/mem0ai/mem0) 的设计理念：
- 多级记忆分层
- 开发者友好的 API
- CLI 优先
- LoCoMo Benchmark 评测体系

## Timeline

- **Phase 1**: 核心库 + 基本 CLI (MVP)
- **Phase 2**: 混合搜索 + 实体提取
- **Phase 3**: 增量索引 + 时间旅行
- **Phase 4**: SDK + 集成
- **Phase 5**: 评测体系 + Benchmark 适配

## Open Questions

1. 嵌入模型选择：本地 Embedding 还是调用远程 API？
2. 是否需要支持多用户/多租户？
3. 与现有 skills/ 系统如何集成？
4. 评测数据集的规模和覆盖范围？

## Related Documents

- `docs/specs/omega-hpc-memory-spec.md` - 详细技术规格
- `docs/guide/omega-hpc-memory-guide.md` - 使用指南
