# Archived: Omega HPC Memory v1 Design

**Archived Date:** 2026-04-10

**Reason:** 设计理念已更新为分层多文件架构

**Replacement:** `../prds/omega-hpc-memory-prd.md`

---

## Original Design (v1)

### Key Issues Identified

1. **单文件 `omega.omega` 格式问题** (v1 原始设计)
   - 写入放大：任何更新需要重写整个文件
   - 并发安全缺失
   - Git 不友好

2. **向量引擎选型矛盾**
   - PRD 写 HNSW-lite，Spec 退回到 flat index
   - 离线可用与高性能不可兼得

3. **文档检索 vs 智能记忆语义混淆**
   - 原设计试图用单一系统满足两种需求
   - 没有清晰的记忆写入机制

### v1 架构

```
omega.omega  (单文件)
├── Header
├── Metadata
├── Documents
├── BM25 Index
├── Vector Store
└── Checksum
```

### v2 改进

- 分层多文件架构，各司其职
- 索引与内容分离
- 明确 SDK/CLI 记忆写入接口
- 项目级知识库隔离
