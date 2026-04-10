# Omega HPC TODO

## Active

- [ ] **omega-hpc-memory**: Unified memory and knowledge base system
  - [ ] Phase 1: Foundation + Knowledge Index (MVP)
    - [ ] Create project structure
    - [ ] Implement `.hpc/cortex/` - Cortex layer (index)
    - [ ] Implement `.hpc/kb/` - Knowledge layer (content)
    - [ ] `omega-hpc init` command
    - [ ] `omega-hpc add` command
    - [ ] BM25 indexing via tantivy
    - [ ] `omega-hpc search` command
  - [ ] Phase 2: Retrieval Enhancement + Vector Search
    - [ ] Vector store implementation (Flat/HNSW)
    - [ ] Embedding integration (ONNX local or remote)
    - [ ] `omega-hpc find` command
    - [ ] Entity extraction
    - [ ] `omega-hpc stat` command
  - [ ] Phase 3: Memory Layer + MCP Integration
    - [ ] Implement `.hpc/mem/` - Memory layer
    - [ ] `omega-hpc recall` command
    - [ ] MCP protocol support
    - [ ] `omega-hpc forget` command
    - [ ] Agent memory auto-extraction
  - [ ] Phase 4: Advanced Features
    - [ ] Incremental index updates
    - [ ] Time-travel debugging
    - [ ] `omega-hpc eval` command
  - [ ] Phase 5: Evaluation System
    - [ ] Benchmark framework
    - [ ] Knowledge retrieval benchmarks
    - [ ] Memory recall benchmarks
    - [ ] Anti-gaming measures

## Completed

- [x] Create project documentation structure
- [x] Write `docs/prds/omega-hpc-memory-prd.md` (revised)
- [x] Write `docs/specs/omega-hpc-memory-spec.md` (revised)
- [x] Write `docs/guide/omega-hpc-memory-guide.md` (revised)

## Key Design Changes (v2)

- **Multi-file layered architecture** instead of single .hpc file
  - `.hpc/cortex/` - Metacognition index
  - `.hpc/kb/` - Knowledge base content
  - `.hpc/mem/` - Agent memory layer
- **Index-first design** - Store indexes, load content on demand
- **Unified memory + knowledge base** - Both agent memory and document retrieval
- **Project-scoped knowledge base** - User decides boundaries

## Deferred

- None

## Archive

- [v1 design](./archive/omega-hpc-memory-prd-v1.md) - Initial monolithic single-file design
