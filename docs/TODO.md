# Omega HPC TODO

## Active

- [ ] **omega-hpc-memory**: Persistent memory system for AI agents
  - [ ] Phase 1: Core library + basic CLI
    - [ ] Create `crates/omega-hpc-memory-core/` workspace member
    - [ ] Implement `.omega` file format
    - [ ] Implement BM25 index (via tantivy)
    - [ ] Implement vector store
    - [ ] Create `omega-hpc-memory-cli` binary
    - [ ] Implement `init` command
    - [ ] Implement `add` command
  - [ ] Phase 2: Search + entities
    - [ ] Implement `search` command (hybrid mode)
    - [ ] Implement `find` command (exact/regex)
    - [ ] Implement entity extractor
    - [ ] Implement `stat` command
  - [ ] Phase 3: Advanced features
    - [ ] Incremental indexing
    - [ ] Time-travel debugging
    - [ ] `export` command
  - [ ] Phase 4: SDK + integration
    - [ ] Create `omega-hpc-memory-sdk` crate
    - [ ] Integrate with skills system
  - [ ] Phase 5: Evaluation framework
    - [ ] Implement LoCoMo benchmark integration
    - [ ] Build anti-gaming evaluation constraints
    - [ ] Create omega-hpc scenario benchmarks
    - [ ] Implement robustness testing suite
    - [ ] Add `eval` command and reporting

## Completed

- [x] Create project documentation structure
- [x] Write `docs/prds/omega-hpc-memory-prd.md`
- [x] Write `docs/specs/omega-hpc-memory-spec.md`
- [x] Write `docs/guide/omega-hpc-memory-guide.md`
- [x] Add evaluation framework (LoCoMo benchmark, anti-gaming measures)

## Deferred

- None

## Archive

- None
