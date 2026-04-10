# Omega HPC

AI Agent 操作平台 - 持久化记忆系统 + 多模型支持框架

## 项目结构

```
omega-hpc/
├── docs/               # 项目文档
│   ├── prds/          # 产品需求文档
│   ├── specs/         # 技术规格
│   └── guide/         # 使用指南
├── crates/            # Rust workspace (规划中)
└── skills/            # AI Agent 操作指南
```

## 核心组件

### Omega HPC Memory (持久化记忆系统)

为 AI Agent 提供持久化记忆存储和检索能力。

**特性:**
- 单文件 `.omega` 格式，零外部依赖
- 混合搜索 (BM25 + 向量)
- RCLI 命令行工具
- 多格式文档支持

**文档:**
- [设计文档](docs/prds/omega-hpc-memory-prd.md)
- [技术规格](docs/specs/omega-hpc-memory-spec.md)
- [使用指南](docs/guide/omega-hpc-memory-guide.md)

**CLI 命令:**
```bash
omega-hpc init              # 初始化记忆库
omega-hpc add ./docs/       # 添加文档
omega-hpc search <query>    # 语义搜索
omega-hpc find --exact x    # 精确匹配
```

## 技术参考

- [Memvid](https://memvid.com) - 单文件 AI 记忆系统
- [Mem0](https://github.com/mem0ai/mem0) - AI Agent 记忆层
- [Tantivy](https://github.com/quickwit-oss/tantivy) - Rust 全文搜索引擎

## 状态

项目规划阶段 - 参见 [docs/TODO.md](docs/TODO.md)
