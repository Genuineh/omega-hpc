# Omega HPC TODO

## Active

- [ ] **omega-hpc-memory**: Cognitive memory and knowledge base system with configurable layers

---

## Phase 1: Foundation — Layer Config + Content Index (MVP)

### 1.1 Project Setup
- [ ] Create `crates/` workspace structure
- [ ] Create `omega-hpc-memory-cli` crate (CLI entry point)
- [ ] Create `omega-hpc-memory-core` crate (shared core library)
- [ ] Create `omega-hpc-memory-sdk` crate (public SDK)

### 1.2 Layer Configuration System
- [ ] **Layer config parser**: `.hpc.toml` → `HpcConfig` struct
- [ ] **Layer inheritance resolver**: `extends` + `abstract` + circular detection
- [ ] **LayerRegistry**: `from_config()`, `get()`, `content_layers()`, `memory_layers()`, `layers_by_priority()`
- [ ] **Template loader**: `default`, `code`, `minimal` templates
- [ ] **`omega-hpc layers`** command (list, --verbose, --validate)

### 1.3 Storage Layer
- [ ] **`.hpc/cortex/`** implementation
  - `meta.toml` (global metadata)
  - `layers/{name}/index.toml` (per-layer doc manifest)
  - `layers/{name}/docs/{doc_id}.toml` (per-doc index)
  - `entities.toml`
- [ ] **`.hpc/layers/{name}/`** implementation
  - Content layer: `{sha256}.chunk` files + hash verification
  - Create directory structure from layer config on `omega-hpc init`

### 1.4 Indexing (Content Layers)
- [ ] **Chunking**: fixed/line/paragraph strategies per layer config
- [ ] **Content-addressable storage**: write chunk, verify hash
- [ ] **BM25 indexing**: per-layer tantivy index in `layers/{name}/bm25/`
- [ ] **Document tracking**: `add_document()` → update `index.toml` + `docs/{id}.toml`
- [ ] **`omega-hpc init --template <name>`** command
- [ ] **`omega-hpc add --layer <name>`** command
- [ ] **`omega-hpc rebuild --layer <name>`** command

### 1.5 Search (Content Layers)
- [ ] **BM25 search**: `omega-hpc search --mode bm25`
- [ ] **Recall Fusion**: cross-layer result merging with boost + priority
- [ ] **`omega-hpc search --layer <name>`** (single layer)
- [ ] **`omega-hpc search <query>`** (all content layers, fusion)

---

## Phase 2: Vector Search + Embeddings

### 2.1 Embedding System
- [ ] **EmbeddingClient trait**: `embed(texts) -> Vec<Vec<f32>>`
- [ ] **OpenAI embedding**: `OpenAIEmbedding` implementation
- [ ] **Cohere embedding**: `CohereEmbedding` implementation
- [ ] **Candle local embedding**: `CandleEmbedding` (all-MiniLM-L6-v2, CPU, ~80MB)
- [ ] **Fallback strategy**: remote → local auto-degradation
- [ ] **Config**: `[embedding]` + `[embedding.local]` in `.hpc.toml`

### 2.2 Vector Search
- [ ] **Per-layer vector storage**: `embeddings/` directory
- [ ] **`omega-hpc search --mode vector`**: vector similarity search
- [ ] **`omega-hpc search --mode hybrid`**: BM25 + vector fusion
- [ ] **HNSW vector store** (optional): for large-scale vectors

### 2.3 Search Utilities
- [ ] **`omega-hpc stat --layer <name>`**: per-layer statistics
- [ ] **`omega-hpc gc --layer <name>`**: garbage collect orphaned chunks
- [ ] **Per-layer stats in stat output**: doc count, chunk count, index size

---

## Phase 3: Memory System — Remember + Recall

### 3.1 RawMemoryStore (Short-Term Memory)
- [ ] **`layers/{name}/raw/`** directory structure
- [ ] **Raw memory format**: `{timestamp}_{uuid}.raw` TOML files
- [ ] **`save_raw()`**: write to raw/, return id
- [ ] **`list_unorganized()`**: get all raw memories with `organized=false`
- [ ] **`mark_organized()`**: set `organized=true`
- [ ] **`delete_raw()`**: remove raw memory file
- [ ] **`raw_count()`**: count unorganized raw memories
- [ ] **`raw_retention` enforcement**: reject写入 when at capacity

### 3.2 OrganizedMemoryStore (Long-Term Memory)
- [ ] **`layers/{name}/organized/facts/`** directory
- [ ] **`layers/{name}/organized/decisions/`** directory
- [ ] **`layers/{name}/organized/sessions/`** directory
- [ ] **Fact format**: `{fact_id}.toml` with confidence, context, raw_refs
- [ ] **Decision format**: `{decision_id}.toml` with rationale, context, alternatives
- [ ] **`save_fact()`**, **`save_decision()`**: write to organized/
- [ ] **`get_fact()`**, **`get_decision()`**: read from organized/
- [ ] **`list_facts()`**, **`list_decisions()`**: list all
- [ ] **BM25 index for organized memories**: separate from content BM25
- [ ] **`recall()`**: search organized memories (BM25)

### 3.3 Memory CLI Commands
- [ ] **`omega-hpc remember <content>`**: write to raw/ (short-term)
- [ ] **`omega-hpc recall <query>`**: search organized/ + graph
- [ ] **`omega-hpc forget --fact <id>`**: delete from organized/
- [ ] **`omega-hpc forget --session <id>`**: delete session memories
- [ ] **Session management**: `start_session()`, `end_session()`
- [ ] **Memory config**: `raw_retention`, `organize_on_count`, `auto_organize`

---

## Phase 4: Cognitive System — Organize + Graph

### 4.1 ConceptGraphStore
- [ ] **`layers/{name}/graph/`** directory
- [ ] **Graph format**: JSON with nodes + edges
- [ ] **Node types**: `technology`, `concept`, `decision`, `person`, `project`, `rule`
- [ ] **Edge relations**: `depends_on`, `implements`, `selects`, `related_to`, `influences`, `contradicts`, `temporal_before`, `preferred_for`
- [ ] **`upsert_node()`**: create or update node
- [ ] **`upsert_edge()`**: create or update edge
- [ ] **`get_node()`**: retrieve node by id
- [ ] **`get_neighbors()`**: nodes connected to given node
- [ ] **`traverse()`**: follow relation edges at depth N
- [ ] **Graph index**: for fast node lookup by name/type

### 4.2 Organizer (整理期)
- [ ] **LLM deduplication**: group similar raw memories
- [ ] **LLM conflict detection**: identify contradictory memories
- [ ] **LLM fact extraction**: generate structured facts from raw
- [ ] **LLM decision extraction**: generate decisions with rationale
- [ ] **LLM concept extraction**: extract concepts from facts/decisions
- [ ] **LLM relation extraction**: extract relations between concepts
- [ ] **`needs_organization()`**: check against `organize_on_count`
- [ ] **`organize()`**: full consolidation pipeline
- [ ] **`OrganizeReport`**: facts_created, decisions_created, nodes_created, edges_created, raw_processed, conflicts_resolved
- [ ] **`OrganizeOptions`**: `force`, `dry_run`, `delete_raw`

### 4.3 Cognitive CLI Commands
- [ ] **`omega-hpc organize --layer <name>`**: trigger organization
  - [ ] `--force`: organize even if below threshold
  - [ ] `--dry-run`: simulate without writing
  - [ ] `--delete-raw`: delete processed raw memories
- [ ] **`omega-hpc graph --query <expr>`**: search concepts
- [ ] **`omega-hpc graph --node <id>`**: view node details
- [ ] **`omega-hpc graph --neighbors <id>`**: view adjacent nodes
- [ ] **`omega-hpc graph --traverse <id> --relation <rel> --depth <N>`**: graph traversal
- [ ] **Graph query syntax**: `--[relation]->` for path queries

---

## Phase 5: SDK + Evaluation + Advanced Features

### 5.1 Rust SDK
- [ ] **SDK crate public API**: `LayerRegistry`, `MemoryStore`, `SearchEngine`, `Indexer`
- [ ] **Builder pattern** for configuration
- [ ] **Error types**: `OmegaHpcError` with error codes matching spec
- [ ] **`extract_facts_from_conversation()`**: SDK method for LLM extraction
- [ ] **Publish to crates.io**
- [ ] **SDK documentation**

### 5.2 Evaluation
- [ ] **`omega-hpc eval --benchmark locomo`**: LoCoMo benchmark
- [ ] **`omega-hpc eval --benchmark knowledge`**: knowledge retrieval benchmark
- [ ] **`omega-hpc eval --benchmark memory`**: memory recall benchmark
- [ ] **LoCoMo dataset integration**: 4 question types, LLM-as-Judge
- [ ] **Anti-gaming measures**: closed-book evaluation

### 5.3 Advanced Features
- [ ] **Incremental index updates**: only re-index changed files
- [ ] **Time-travel debugging**: view historical index states
- [ ] **Auto-organize scheduling**: `organize_interval` (hourly/daily/weekly)
- [ ] **Layer config hot-reload**: update layer config without restart
- [ ] **Layer groups**: group layers for combined operations

---

## Completed

- [x] Create project documentation structure
- [x] Write PRD (cognitive memory system with three-tier architecture)
- [x] Write Spec (layer types, Recall Fusion, three-tier memory, ConceptGraph)
- [x] Write Guide (layer-aware CLI, organize/graph commands, SDK usage)

---

## Implementation Order Summary

```
Step 1: Project setup (1.1)
Step 2: Layer config + registry (1.2)
Step 3: Storage directories (1.3)
Step 4: Chunking + content storage (1.4)
Step 5: BM25 indexing (1.4)
Step 6: CLI: init, add, rebuild (1.4)
Step 7: CLI: search + Recall Fusion (1.5)
Step 8: Embeddings: remote API (2.1)
Step 9: Embeddings: Candle local (2.1)
Step 10: Vector search (2.2)
Step 11: stat, gc commands (2.3)
Step 12: RawMemoryStore (3.1)
Step 13: OrganizedMemoryStore (3.2)
Step 14: CLI: remember, recall, forget (3.3)
Step 15: ConceptGraphStore (4.1)
Step 16: Organizer (4.2)
Step 17: CLI: organize, graph (4.3)
Step 18: SDK crate (5.1)
Step 19: Evaluation commands (5.2)
Step 20: Advanced features (5.3)
```

---

## Design Decision Reference

### Architecture: Cognitive Memory System
- **Three-tier memory**: raw → organized → graph
- **整理期**: LLM-powered consolidation
- **Concept Graph**: project knowledge as nodes + edges
- **User-defined layers**: `.hpc.toml` with inheritance

### Directory Layout
```
.hpc/
├── cortex/
│   ├── meta.toml
│   ├── layers/{name}/
│   │   ├── index.toml
│   │   ├── docs/{doc_id}.toml
│   │   └── bm25/
│   ├── entities.toml
│   └── embeddings/
├── layers/
│   ├── {content-layer}/{sha256}.chunk
│   └── {memory-layer}/
│       ├── raw/{timestamp}_{uuid}.raw
│       ├── organized/facts/{id}.toml
│       ├── organized/decisions/{id}.toml
│       ├── organized/sessions/{id}.toml
│       └── graph/{id}.json
└── sync/manifest.toml
```

### Layer Types
- **content**: file chunks → `add` / `search`
- **memory**: three-tier → `remember` / `recall` / `organize` / `graph`

### Embedding
- Remote: OpenAI / Cohere (1536+ dim)
- Local: Candle + all-MiniLM-L6-v2 (384 dim, CPU, ~80MB)
- Auto-fallback when remote unavailable

### Init Templates
- `default`: kb (content) + mem (memory)
- `code`: code(abstract) → rust/python + docs + decisions + memory
- `minimal`: single content + memory

### Benchmark
- LoCoMo: dialogue memory recall (single-hop, temporal, multi-hop, open-domain)
- Knowledge retrieval: Recall@K, MRR, latency
- Memory recall: recall, precision

## Deferred

- MCP service (explicitly not providing)
- Auto-organize scheduling
- Layer config hot-reload
- Layer groups
