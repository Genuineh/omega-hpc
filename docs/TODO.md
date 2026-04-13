# Omega HPC TODO

## Active

- [ ] **omega-hpc-memory**: Cognitive memory and knowledge base system with configurable layers

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

### Phase 3: Three-Tier Memory Architecture
- [ ] Implement `RawMemoryStore` - raw memory CRUD
  - `save_raw()` - writes directly to raw/ directory
  - `list_unorganized()` - get all unorganized raw memories
  - `mark_organized()` / `delete_raw()`
  - Raw count tracking + `raw_retention` enforcement
- [ ] Implement `OrganizedMemoryStore` - structured memory CRUD
  - `save_fact()` / `save_decision()` - writes to organized/
  - `get_fact()` / `get_decision()` / `list_facts()` / `list_decisions()`
  - BM25 index for organized memory recall
- [ ] `omega-hpc remember --layer <name>` → writes to raw/
- [ ] `omega-hpc recall --layer <name>` → queries organized/ + graph
- [ ] `omega-hpc forget --layer <name>` command (per-layer)

### Phase 4: Organization Period (整理期) + Cognitive Graph
- [ ] Implement `ConceptGraphStore` - graph storage + traversal
  - Node CRUD: technology/concept/decision/person/project/rule
  - Edge CRUD: depends_on/implements/selects/related_to/contradicts/...
  - `query()`, `get_neighbors()`, `traverse()` APIs
- [ ] Implement `Organizer` - memory consolidation engine
  - Deduplication (LLM-powered)
  - Conflict detection and resolution
  - Fact/Decision extraction from raw memories
  - Concept + relation extraction for graph
  - `organize()` with `OrganizeOptions` (force/dry_run/delete_raw)
  - `needs_organization()` check
- [ ] `omega-hpc organize --layer <name>` command
  - `--force`, `--dry-run`, `--delete-raw` flags
  - OrganizeReport output
- [ ] `omega-hpc graph --query <expr>` command
  - `--node`, `--neighbors`, `--traverse` options
  - Graph query expression syntax
- [ ] `extract_facts_from_conversation` SDK method (raw → organized pipeline)
- [ ] Memory layer config: `raw_retention`, `organize_on_count`, `auto_organize`

### Phase 5: SDK + Advanced Features
- [ ] Rust SDK crate: `LayerRegistry`, `MemoryStore`, `SearchEngine`, `Indexer`
- [ ] `omega-hpc gc --layer <name>` (per-layer chunk garbage collection)
- [ ] Incremental index updates
- [ ] Time-travel debugging
- [ ] `omega-hpc eval --benchmark locomo`
- [ ] `omega-hpc eval --benchmark knowledge`
- [ ] `omega-hpc eval --benchmark memory`
- [ ] SDK documentation

## Completed

- [x] Create project documentation structure
- [x] Write PRD (cognitive memory system with three-tier architecture)
- [x] Write Spec (layer types, Recall Fusion, three-tier memory, ConceptGraph)
- [x] Write Guide (layer-aware CLI, organize/graph commands, SDK usage)

## Key Design Decisions

### Architecture: Cognitive Memory System
- **Three-tier memory**: raw (碎片) → organized (结构化) → graph (认知)
- **整理期 (Organization Period)**: 定期/手动将 raw memory 转化为 organized + graph
- **Concept Graph as meta-cognition**: 项目认知体系的结构化表达
- **User-defined layers**: `.hpc.toml` 定义，支持继承

### Directory Layout
```
.hpc/layers/{memory-layer}/
  raw/                   # 短期记忆（碎片）
  organized/facts/        # 长期记忆 - facts
  organized/decisions/   # 长期记忆 - decisions
  organized/sessions/     # 会话
  graph/                 # 认知图谱 (nodes + edges)
```

### Layer Types
- **content**: file chunks, `add`, `search`
- **memory**: three-tier memory, `remember`, `recall`, `organize`, `graph`

### Organization Algorithm
1. Read unorganized raw memories
2. LLM: deduplicate, detect conflicts, extract key facts/decisions
3. LLM: extract concepts and relations from structured memories
4. Write to organized/ (facts + decisions)
5. Update graph (upsert nodes + edges)
6. Mark/delete raw memories
7. Return OrganizeReport

### Recall with Graph Reasoning
- Recall queries organized memory (BM25) AND graph (traversal)
- Graph traversal: follow decision chains, concept relations
- Combined result: structured memory + inferred context from graph

### Embedding Model
- **Dual mode: Remote API + Candle local fallback**
- Remote: OpenAI/Cohere (1536+ dim, 最高质量)
- Local: Candle + all-MiniLM-L6-v2 (384 dim, ~80MB, CPU only)
- Auto-fallback when remote unavailable

### Init Templates
- `default` - kb (content) + mem (memory, three-tier)
- `code` - code(abstract) → rust/python + docs + decisions + memory
- `minimal` - single content + memory

### Benchmark
- **LoCoMo adopted** - dialogue memory recall across sessions
- **Knowledge retrieval** - document search accuracy
- **Memory recall** - agent memory accuracy
- Anti-gaming: closed-book, independent question bank, no shortcut optimization

## Deferred

- MCP service (explicitly not providing)
- Layer config hot-reload
- Layer groups
- Auto-organize scheduling (定时整理)
