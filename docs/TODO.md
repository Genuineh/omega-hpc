# Omega HPC TODO

## Active

- [ ] **omega-hpc-memory**: Unified memory and knowledge base system

### Phase 1: Foundation + Knowledge Index (MVP)
- [ ] Create project structure
- [ ] Implement `.hpc/cortex/` - Cortex layer (index)
  - index.toml (document清单)
  - bm25.bin (BM25结构)
  - entities.toml (实体槽位)
- [ ] Implement `.hpc/kb/` - Knowledge layer (content)
  - Content-addressable chunk 存储
  - Hash 校验
- [ ] `omega-hpc init` command
- [ ] `omega-hpc add` command + chunking
- [ ] BM25 indexing via tantivy
- [ ] `omega-hpc search --mode bm25`
- [ ] `omega-hpc rebuild` command

### Phase 2: Vector Search + Entities
- [ ] Remote embedding API integration (OpenAI only)
- [ ] `omega-hpc search --mode vector`
- [ ] `omega-hpc search --mode hybrid`
- [ ] `omega-hpc stat` command
- [ ] HNSW vector store (optional)

### Phase 3: Memory Layer + SDK
- [ ] Implement `.hpc/mem/` - Memory layer
  - Session management
  - Fact/Decision storage
- [ ] `omega-hpc remember` command
- [ ] `omega-hpc recall` command
- [ ] `omega-hpc forget` command
- [ ] Rust SDK crate
- [ ] SDK documentation

### Phase 4: Advanced Features
- [ ] Incremental index updates
- [ ] Time-travel debugging
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
- `.hpc/kb/` - Content chunks (content-addressable)
- `.hpc/mem/` - Structured agent memories (TOML)

### Layer Boundaries
- **Cortex vs KB**: Cortex records `content_hash`, KB provides content by hash
- **Mem vs KB**: Mem = structured memories (facts/decisions), KB = document chunks
- **Query separation**: `search` for KB, `recall` for Mem

### Embedding Model
- **Remote API only** (Phase 2)
- No ONNX (has C library deps)
- No Candle/burn (ecosystem immature)
- OpenAI/Cohere only

### Memory Writing
- **Explicit only** - no auto-extraction
- Manual via CLI (`remember`) or SDK
- Session snapshots on demand

### Benchmark
- **LoCoMo NOT adopted** - it's for dialogue memory auto-extraction
- Use knowledge retrieval + memory recall benchmarks instead

## Deferred

- MCP service (explicitly not providing)
- Local embedding (ONNX deps conflict with zero-dependency goal)
