# Omega HPC TODO

## Active

- [ ] **omega-hpc-memory**: Configurable layer-based memory and knowledge base system

### Phase 1: Configurable Layer System + Knowledge Index (MVP)
- [ ] Create project structure
- [ ] Implement `.hpc.toml` layer configuration parser
  - `[[layer]]` definition parsing
  - Layer inheritance resolution (`extends` + `abstract`)
  - Circular inheritance detection
  - Layer type validation (content/memory)
- [ ] Implement `LayerRegistry` - central layer management
  - `from_config()` - parse and resolve all layers
  - `content_layers()` / `memory_layers()` / `layers_by_priority()`
  - Template loading: `default`, `code`, `minimal`
- [ ] Implement `.hpc/cortex/` - Cortex layer (per-layer index)
  - `meta.toml` (全局元数据)
  - `layers/{name}/index.toml` (层级文档清单)
  - `layers/{name}/docs/{doc_id}.toml` (按文档拆分)
  - `layers/{name}/bm25/` (tantivy 索引目录，按层独立)
  - `entities.toml` (实体槽位)
- [ ] Implement `.hpc/layers/{name}/` - Per-layer storage
  - Content type: content-addressable chunk 存储 + Hash 校验
  - Memory type: TOML sessions/users 目录
- [ ] `omega-hpc init --template <name>` command
- [ ] `omega-hpc layers` command (list, --verbose, --validate)
- [ ] `omega-hpc add --layer <name>` command + per-layer chunking
- [ ] BM25 indexing via tantivy (per-layer independent)
- [ ] `omega-hpc search --layer <name> --mode bm25`
- [ ] `omega-hpc rebuild --layer <name>` command

### Phase 2: Vector Search + Recall Fusion + Local Embedding
- [ ] Remote embedding API integration (OpenAI)
- [ ] Local CPU embedding via Candle (all-MiniLM-L6-v2, 384 dim)
- [ ] Embedding fallback strategy (remote → local auto-degradation)
- [ ] `omega-hpc search --mode vector` (per-layer)
- [ ] `omega-hpc search --mode hybrid` (per-layer)
- [ ] Recall Fusion: cross-layer result merging (boost + priority)
- [ ] `omega-hpc stat --layer <name>` command (per-layer statistics)
- [ ] HNSW vector store (optional)

### Phase 3: Memory Layer + SDK
- [ ] Memory type layer implementation
  - Session management
  - Fact/Decision storage
  - Per-layer independent BM25 index for recall
- [ ] `omega-hpc remember --layer <name>` command
- [ ] `omega-hpc recall --layer <name>` command
- [ ] `omega-hpc forget --layer <name>` command
- [ ] `omega-hpc gc --layer <name>` command (per-layer chunk garbage collection)
- [ ] `extract_facts_from_conversation` SDK method
- [ ] Rust SDK crate (`LayerRegistry`, `MemoryStore`, `SearchEngine`, `Indexer`)
- [ ] SDK documentation

### Phase 4: Advanced Features
- [ ] Incremental index updates
- [ ] Time-travel debugging
- [ ] `omega-hpc eval --benchmark locomo`
- [ ] `omega-hpc eval --benchmark knowledge`
- [ ] `omega-hpc eval --benchmark memory`

## Completed

- [x] Create project documentation structure
- [x] Write PRD (configurable layer system with inheritance)
- [x] Write Spec (layer types, Recall Fusion, per-layer indexing)
- [x] Write Guide (layer-aware CLI, templates, SDK usage)

## Key Design Decisions

### Architecture: Configurable Data Layers
- **User-defined layers** via `.hpc.toml` instead of hardcoded kb/mem
- **Layer types**: `content` (file indexing) and `memory` (agent memories)
- **Layer inheritance**: `abstract = true` + `extends` for DRY config
- **Recall Fusion**: cross-layer search with boost + priority
- **Per-layer indexing**: each layer has independent BM25, independent config

### Directory Layout
- `.hpc/cortex/layers/{name}/` - per-layer index (index.toml + docs/ + bm25/)
- `.hpc/layers/{name}/` - per-layer storage (chunks or TOML)
- `.hpc/cortex/embeddings/` - shared vector store
- `.hpc/cortex/entities.toml` - shared entity slots

### Layer Boundaries
- **Cortex vs Layer**: Cortex records `content_hash`, layer provides content by hash
- **Content vs Memory**: content = file chunks, memory = structured facts/decisions
- **Query separation**: `search` for content layers, `recall` for memory layers
- **Cross-layer**: Recall Fusion merges results from multiple layers

### Embedding Model
- **Dual mode: Remote API + Candle local fallback**
- Remote: OpenAI/Cohere (1536+ dim, 最高质量)
- Local: Candle + all-MiniLM-L6-v2 (384 dim, ~80MB, CPU only)
- Auto-fallback when remote unavailable

### Memory Writing
- **Explicit + SDK extraction**:
  - CLI: `remember --layer <name>` 命令显式写入
  - SDK: `extract_facts_from_conversation` 提取后显式写入
- Session snapshots on demand

### Garbage Collection
- `omega-hpc gc --layer <name>` 清理指定 content 层未引用 chunk
- 默认 dry-run，需显式执行
- 同时支持清理 stale 索引条目 (`--all`)

### Init Templates
- `default` - kb (content) + mem (memory)
- `code` - code(abstract) → rust/python + docs + decisions + memory
- `minimal` - single content + memory

### Benchmark
- **LoCoMo adopted** - dialogue memory recall across sessions
- **Knowledge retrieval** - document search accuracy
- **Memory recall** - agent memory accuracy
- Anti-gaming: closed-book, independent question bank, no shortcut optimization

## Deferred

- MCP service (explicitly not providing)
- Layer config hot-reload (Phase 2+)
- Layer groups (grouping layers for combined operations)
