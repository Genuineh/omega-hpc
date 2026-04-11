# Omega HPC

AI Agent 操作平台 - 统一记忆与知识库系统

## 核心理念

**索引即记忆**。保存结构化索引（类比大脑皮层/元认知），内容按需加载。记忆分层通过多文件实现，各司其职，不无限扩张。

**集成方式**：仅通过 CLI 或 SDK 集成，不提供 MCP 服务。

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
- 多文件分层架构（`.hpc/cortex/` + `.hpc/kb/` + `.hpc/mem/`）
- 索引与内容分离
- 项目级知识库隔离
- CLI + SDK 接口（无 MCP）

**文档：**
- [设计文档](docs/prds/omega-hpc-memory-prd.md)
- [技术规格](docs/specs/omega-hpc-memory-spec.md)
- [使用指南](docs/guide/omega-hpc-memory-guide.md)

**目录结构：**
```
.hpc/
├── cortex/          # 元认知层（索引）
├── kb/             # 知识库层（内容分片）
├── mem/            # 记忆层（Agent 私有）
└── sync/           # 同步层
```

**CLI 命令：**
```bash
omega-hpc init              # 初始化记忆空间
omega-hpc add ./docs/       # 添加知识库
omega-hpc search <query>    # 检索知识库
omega-hpc remember "记忆"   # 写入记忆
omega-hpc recall <query>    # 回忆记忆
```

## 技术参考

- [Tantivy](https://github.com/quickwit-oss/tantivy) - Rust 全文搜索引擎
- [Mem0](https://github.com/mem0ai/mem0) - AI Agent 记忆层
- [Memvid](https://memvid.com) - 单文件 AI 记忆系统

## 状态

项目规划阶段 - 参见 [docs/TODO.md](docs/TODO.md)
