# Omega HPC TODO

## Active

- [ ] **omega-hpc-memory**: Unified memory and knowledge base system

### Phase 1: Foundation + Knowledge Index (MVP)
- [ ] Create project structure
- [ ] Implement `.hpc/cortex/` - Cortex layer (index)
  - meta.toml (全局元数据)
  - docs/{doc_id}.toml (按文档拆分的索引)
  - bm25/ (tantivy 索引目录)
  - entities.toml (实体槽位)
- [ ] Implement `.hpc/kb/` - Knowledge layer (content)
  - Content-addressable chunk 存储
  - Hash 校验
- [ ] `omega-hpc init` command
- [ ] `omega-hpc add` command + chunking
- [ ] BM25 indexing via tantivy
- [ ] `omega-hpc search --mode bm25`
- [ ] `omega-hpc rebuild` command

### Phase 2: Vector Search + Entities + Local Embedding
- [ ] Remote embedding API integration (OpenAI)
- [ ] Local CPU embedding via Candle (all-MiniLM-L6-v2, 384 dim)
- [ ] Embedding fallback strategy (remote → local auto-degradation)
- [ ] `omega-hpc search --mode vector`
- [ ] `omega-hpc search --mode hybrid`
- [ ] `omega-hpc stat` command
- [ ] HNSW vector store (optional)

### Phase 3: Memory Layer + SDK
- [ ] Implement `.hpc/mem/` - Memory layer
  - Session management
  - Fact/Decision storage
  - Independent BM25 index for recall
- [ ] `omega-hpc remember` command (write + index to Mem BM25)
- [ ] `omega-hpc recall` command (query via Mem BM25)
- [ ] `omega-hpc forget` command
- [ ] `omega-hpc gc` command (KB chunk garbage collection)
- [ ] `extract_facts_from_conversation` SDK method
- [ ] Rust SDK crate
- [ ] SDK documentation

### Phase 4: Advanced Features
- [ ] Incremental index updates
- [ ] Time-travel debugging
- [ ] `omega-hpc eval --benchmark locomo`
- [ ] `omega-hpc eval --benchmark knowledge`
- [ ] `omega-hpc eval --benchmark memory`

## Completed

- [x] Create project documentation structure
- [x] Write PRD (multi-file layered architecture)
- [x] Write Spec (layer boundaries defined)
- [x] Write Guide (commands + SDK)

## Key Design Decisions

### Architecture
- **Multi-file layered** instead of single file
- `.hpc/cortex/` - Index only (可独立重建)
  - 按文档拆分: meta.toml + docs/{doc_id}.toml
  - BM25: tantivy 索引目录（非单文件）
- `.hpc/kb/` - Content chunks (content-addressable)
- `.hpc/mem/` - Structured agent memories (TOML)
  - 独立 BM25 索引用于 recall

### Layer Boundaries
- **Cortex vs KB**: Cortex records `content_hash`, KB provides content by hash
- **Mem vs KB**: Mem = structured memories (facts/decisions), KB = document chunks
- **Query separation**: `search` for KB, `recall` for Mem

### Embedding Model
- **Dual mode: Remote API + Candle local fallback**
- Remote: OpenAI/Cohere (1536+ dim, 最高质量)
- Local: Candle + all-MiniLM-L6-v2 (384 dim, ~80MB, CPU only)
- Auto-fallback when remote unavailable

### Memory Writing
- **Explicit + SDK extraction**:
  - CLI: `remember` 命令显式写入
  - SDK: `extract_facts_from_conversation` 提取后显式写入
- Session snapshots on demand

### Garbage Collection
- `omega-hpc gc` 清理 KB 层未引用 chunk
- 默认 dry-run，需显式执行
- 同时支持清理 stale 索引条目 (`--all`)

### Benchmark
- **LoCoMo adopted** - dialogue memory recall across sessions
- **Knowledge retrieval** - document search accuracy
- **Memory recall** - agent memory accuracy
- Anti-gaming: closed-book, independent question bank, no shortcut optimization

## Deferred

- MCP service (explicitly not providing)
