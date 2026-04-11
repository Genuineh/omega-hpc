# Omega HPC

AI Agent 操作平台 - 统一记忆与知识库系统

## 核心理念

**索引即记忆**。保存结构化索引（类比大脑皮层/元认知），内容按需加载。记忆分层通过多文件实现，各司其职，不无限扩张。

**集成方式**：仅通过 CLI 或 SDK 集成，不提供 MCP 服务。

**可配置数据层**：用户通过 `.hpc.toml` 定义数据层，每层独立的索引策略和召回行为，支持继承和跨层融合检索，针对不同项目提供个性化检索方案。

## 项目结构

```
omega-hpc/
├── docs/                    # 项目文档
│   ├── prds/               # 产品需求文档
│   ├── specs/              # 技术规格
│   ├── guide/              # 使用指南
├── crates/                 # Rust workspace (规划中)
└── skills/                 # AI Agent 操作指南
```

## 核心组件

### Omega HPC Memory

统一记忆与知识库系统，同时服务于：
- **Agent 长期记忆** - 跨会话保留决策、偏好
- **外部知识库** - 以项目为单位的文档索引与检索

**架构特点：**
- 可配置数据层（`.hpc.toml` 定义，content/memory 两种类型）
- 层间继承（abstract + extends，避免重复配置）
- 跨层 Recall Fusion（boost + priority 自动合并）
- 每层独立索引（BM25、向量、分块策略）
- CLI + SDK 接口（无 MCP）

**文档：**
- [设计文档](docs/prds/omega-hpc-memory-prd.md)
- [技术规格](docs/specs/omega-hpc-memory-spec.md)
- [使用指南](docs/guide/omega-hpc-memory-guide.md)

**目录结构：**
```
.hpc/
├── cortex/                # 元认知层（索引）
│   ├── meta.toml          # 全局元数据
│   ├── layers/            # 按层拆分的索引
│   │   ├── rust/          # 层级 BM25 + 文档索引
│   │   ├── docs/
│   │   └── memory/
│   ├── entities.toml      # 实体槽位
│   └── embeddings/        # 向量分片（共享）
├── layers/                # 用户定义的数据层
│   ├── rust/              # content: chunk 存储
│   ├── docs/              # content: chunk 存储
│   └── memory/            # memory: TOML 记忆
└── sync/
    └── manifest.toml
```

**层配置示例：**
```toml
# 抽象模板（不创建存储）
[[layer]]
name = "code"
type = "content"
abstract = true

[layer.indexing]
chunk_size = 1024
strategy = "line"

[layer.recall]
boost = 1.5

# 继承 code 模板，仅覆盖 glob
[[layer]]
name = "rust"
extends = "code"
priority = 1

[layer.indexing]
glob = ["*.rs"]

# 独立决策层，高优先级
[[layer]]
name = "decisions"
type = "content"
priority = 0

[layer.indexing]
glob = ["docs/decisions/*.md"]

[layer.recall]
boost = 2.0
```

**CLI 命令：**
```bash
omega-hpc init --template code    # 初始化（代码项目模板）
omega-hpc layers                   # 查看层配置
omega-hpc add ./src/ --layer rust  # 添加到指定层
omega-hpc search <query>           # 跨层融合检索
omega-hpc search <query> --layer rust  # 单层检索
omega-hpc remember "记忆" --layer memory  # 写入记忆
omega-hpc recall <query>           # 回忆记忆
```

## 技术参考

- [Tantivy](https://github.com/quickwit-oss/tantivy) - Rust 全文搜索引擎
- [Candle](https://github.com/huggingface/candle) - HuggingFace 纯 Rust ML 框架
- [Mem0](https://github.com/mem0ai/mem0) - AI Agent 记忆层

## Known Limitations

| 限制 | 缓解措施 |
|------|----------|
| 远程嵌入 API 需网络 | Candle 本地 fallback（384 dim） |
| 切换嵌入模型需重建索引 | `omega-hpc rebuild --full` |
| Memory 层 recall 依赖 BM25 | Phase 2 添加向量索引 |
| Content 层无自动 gc | `omega-hpc gc` 手动清理 |
| tantivy 索引跨 major 版本不兼容 | 文档标注版本要求 |
| 层继承仅在配置加载时解析 | 配置热重载为 Phase 2+ |

## 状态

项目规划阶段 - 参见 [docs/TODO.md](docs/TODO.md)
