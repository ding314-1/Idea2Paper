# File Organization Guide

This document explains the file organization of the Idea2Paper codebase and clarifies the purpose of different directories.

## 📁 Directory Structure

```
Idea2Paper/
├── Paper-KG-Pipeline/
│   ├── src/                           # Core source code (importable Python package)
│   │   └── idea2paper/
│   │       ├── infra/                 # Infrastructure (LLM, embeddings, logging, config)
│   │       │   ├── llm.py            # LLM API client (supports GitHub Models, OpenAI, etc.)
│   │       │   ├── embeddings.py     # Embedding API client
│   │       │   ├── run_context.py    # Runtime context management
│   │       │   ├── run_logger.py     # Structured logging system
│   │       │   └── ...
│   │       ├── recall/                # Recall system (retrieval)
│   │       │   ├── recall_system.py  # Main recall interface
│   │       │   └── ...
│   │       ├── novelty/               # Novelty checking system
│   │       │   ├── novelty_checker.py
│   │       │   └── novelty_index.py
│   │       ├── review/                # Review system
│   │       │   ├── critic.py         # Anchored multi-agent review
│   │       │   └── review_index.py
│   │       ├── pipeline/              # Core pipeline modules
│   │       │   ├── manager.py        # Pipeline orchestration
│   │       │   ├── pattern_selector.py
│   │       │   ├── story_generator.py
│   │       │   ├── idea_fusion.py
│   │       │   ├── story_reflector.py
│   │       │   ├── refinement.py
│   │       │   ├── planner.py
│   │       │   └── ...
│   │       ├── application/           # Application layer (delegates to core modules)
│   │       │   ├── pipeline/         # Pipeline wrappers
│   │       │   ├── review/           # Review wrappers
│   │       │   ├── novelty/          # Novelty wrappers
│   │       │   └── verification/     # Verification wrappers
│   │       └── config.py             # Unified configuration
│   │
│   ├── scripts/                       # Executable scripts and entry points
│   │   ├── tools/                    # Build and maintenance tools
│   │   │   ├── build_entity_v3.py   # Build knowledge graph nodes (RECOMMENDED)
│   │   │   ├── build_entity.py      # Legacy entity builder (v2)
│   │   │   ├── build_edges.py       # Build knowledge graph edges
│   │   │   ├── generate_patterns.py # Generate pattern clusters
│   │   │   ├── build_recall_index.py
│   │   │   ├── build_novelty_index.py
│   │   │   └── ...
│   │   ├── demos/                    # Demo and example scripts
│   │   │   ├── demo_pipeline.py     # Pipeline usage examples
│   │   │   ├── run_pipeline.py      # Basic pipeline runner
│   │   │   └── simple_recall_demo.py
│   │   ├── dev/                      # Development utilities
│   │   │   ├── compare_pipeline_result.py
│   │   │   └── verify_recall_equivalence.py
│   │   ├── legacy/                   # Deprecated scripts
│   │   │   └── generate_patterns_old.py
│   │   │
│   │   ├── idea2story_pipeline.py   # MAIN ENTRY POINT for end-to-end pipeline
│   │   │
│   │   └── [wrapper files]          # Backward compatibility wrappers
│   │       ├── build_edges.py       # → tools/build_edges.py
│   │       ├── build_entity.py      # → tools/build_entity.py
│   │       ├── demo_pipeline.py     # → demos/demo_pipeline.py
│   │       └── ...
│   │
│   ├── output/                       # Generated files (knowledge graph + results)
│   │   ├── nodes_paper.json         # Paper nodes
│   │   ├── nodes_pattern.json       # Pattern nodes
│   │   ├── nodes_domain.json        # Domain nodes
│   │   ├── nodes_idea.json          # Idea nodes
│   │   ├── edges.json               # Graph edges
│   │   ├── patterns_structured.json # Clustered patterns
│   │   ├── pipeline_result.json     # Full pipeline output
│   │   ├── final_story.json         # Generated research story
│   │   └── ...
│   │
│   └── docs/                         # Documentation
│       ├── OUTPUT_FILES_EXPLAINED.md    # THIS IS NEW! Explains output JSON files
│       ├── 00_PROJECT_OVERVIEW.md
│       ├── 01_KG_CONSTRUCTION.md
│       ├── 02_RECALL_SYSTEM.md
│       ├── 03_IDEA2STORY_PIPELINE.md
│       └── archive/                  # Archived documentation
│
├── frontend/                          # Web UI (experimental)
├── papers/                            # Raw paper data
└── .env                               # API keys and secrets (DO NOT COMMIT)
```

---

## 🎯 Key Principles

### 1. **Source Code Location: `src/idea2paper/`**

All **importable Python code** lives in `src/idea2paper/`. This is installed as a package and imported by scripts.

**Why?**
- Clean separation between library code and executable scripts
- Enables proper module imports: `from idea2paper.infra.llm import call_llm`
- Facilitates testing and reuse

### 2. **Executable Scripts: `scripts/`**

All **command-line entry points** live in `scripts/` (and subdirectories).

**Subdirectory Organization**:
- `scripts/tools/` → Knowledge graph building and maintenance
- `scripts/demos/` → Examples and demonstrations
- `scripts/dev/` → Development and debugging utilities
- `scripts/legacy/` → Deprecated scripts kept for reference

### 3. **Backward Compatibility Wrappers**

Several files in `scripts/` root are **thin wrappers** that redirect to the real implementations:

```python
# scripts/build_edges.py (wrapper)
#!/usr/bin/env python3
from pathlib import Path
import runpy

# Compatibility wrapper (scripts/ -> scripts/tools)
runpy.run_path(str(Path(__file__).parent / "tools" / "build_edges.py"), run_name="__main__")
```

**Purpose**:
- Maintain backward compatibility with old documentation and user scripts
- Allow gradual migration to new structure
- All wrappers can be safely removed once documentation is updated

**Wrapper Files** (can be deleted after documentation update):
- `scripts/build_edges.py` → `scripts/tools/build_edges.py`
- `scripts/build_entity.py` → `scripts/tools/build_entity.py`
- `scripts/build_entity_v3.py` → `scripts/tools/build_entity_v3.py`
- `scripts/generate_patterns.py` → `scripts/tools/generate_patterns.py`
- `scripts/extract_paper_review.py` → `scripts/tools/extract_paper_review.py`
- `scripts/demo_pipeline.py` → `scripts/demos/demo_pipeline.py`
- `scripts/run_pipeline.py` → `scripts/demos/run_pipeline.py`
- `scripts/simple_recall_demo.py` → `scripts/demos/simple_recall_demo.py`
- `scripts/generate_patterns_old.py` → `scripts/legacy/generate_patterns_old.py`

---

## 📝 File Purposes

### Core Entry Point

- **`scripts/idea2story_pipeline.py`**  
  Main command-line interface for running the complete Idea2Story pipeline.
  
  ```bash
  python Paper-KG-Pipeline/scripts/idea2story_pipeline.py "Your research idea"
  ```

### Knowledge Graph Building (One-Time Setup)

- **`scripts/tools/build_entity_v3.py`** ⭐ **RECOMMENDED**  
  Builds all four node types: paper, pattern, domain, idea.  
  Uses ICLR-specific data format.
  
- **`scripts/tools/build_entity.py`** (Legacy v2)  
  Older version using generic conference format.  
  **Use v3 instead.**

- **`scripts/tools/build_edges.py`**  
  Builds graph edges connecting nodes (7 edge types).  
  Must run after entity building.

- **`scripts/tools/generate_patterns.py`**  
  Clusters paper skeletons and tricks into patterns.  
  Creates `patterns_structured.json`.

### Index Building (One-Time Setup)

- **`scripts/tools/build_recall_index.py`**  
  Pre-computes embeddings for recall system.
  
- **`scripts/tools/build_novelty_index.py`**  
  Pre-computes embeddings for novelty checking.

### Demos and Examples

- **`scripts/demos/demo_pipeline.py`**  
  Shows how to use the pipeline programmatically.
  
- **`scripts/demos/simple_recall_demo.py`**  
  Demonstrates recall system usage.

### Development Tools

- **`scripts/dev/compare_pipeline_result.py`**  
  Compares two pipeline runs for debugging.
  
- **`scripts/dev/verify_recall_equivalence.py`**  
  Tests recall system consistency.

---

## 🔧 How Things Work

### Import Flow

```python
# In any script (e.g., scripts/idea2story_pipeline.py):
from idea2paper.infra.llm import call_llm              # LLM client
from idea2paper.config import LLM_MODEL, LLM_API_URL   # Configuration
from idea2paper.pipeline.manager import PipelineManager # Pipeline logic
from idea2paper.recall.recall_system import RecallSystem # Recall system
```

**Key Point**: Scripts import from `idea2paper.*`, which is the package in `src/idea2paper/`.

### Configuration Hierarchy

1. **Environment variables** (`.env` file or shell exports)
2. **Config file** (`i2p_config.json`)
3. **Code defaults** (`src/idea2paper/config.py`)

### Execution Flow

```
User Command
    ↓
scripts/idea2story_pipeline.py (entry point)
    ↓
imports from src/idea2paper/
    ├── pipeline/manager.py (orchestration)
    ├── recall/recall_system.py (retrieval)
    ├── review/critic.py (evaluation)
    └── infra/llm.py (LLM calls)
    ↓
Output: pipeline_result.json, final_story.json
```

---

## 🧹 Cleanup Recommendations

### Files That Can Be Removed

1. **Wrapper scripts** in `scripts/` root (after updating documentation):
   - `build_edges.py`
   - `build_entity.py`
   - `build_entity_v3.py`
   - `generate_patterns.py`
   - `demo_pipeline.py`
   - `run_pipeline.py`
   - `simple_recall_demo.py`
   - `generate_patterns_old.py`

2. **Duplicate modules** (if confirmed unused):
   - `scripts/pipeline/` directory (entire folder is just config re-exports)

3. **Legacy scripts**:
   - `scripts/legacy/generate_patterns_old.py` (if not needed for reference)

### Action Plan

**Phase 1: Documentation Update** ✅ DONE
- [x] Create OUTPUT_FILES_EXPLAINED.md
- [x] Update README with new file paths

**Phase 2: Wrapper Deprecation** (Future)
- [ ] Add deprecation warnings to wrapper scripts
- [ ] Update all documentation references to use `scripts/tools/` paths
- [ ] Remove wrapper files after transition period

**Phase 3: Deep Cleanup** (Future)
- [ ] Consolidate `src/idea2paper/application/` into base modules
- [ ] Remove unused legacy code
- [ ] Standardize import patterns

---

## 🎓 Best Practices

### For Users

1. **Always use the canonical path** (e.g., `scripts/tools/build_edges.py` instead of `scripts/build_edges.py`)
2. **Run setup scripts in order**:
   ```bash
   # 1. Build knowledge graph
   python Paper-KG-Pipeline/scripts/tools/build_entity_v3.py
   python Paper-KG-Pipeline/scripts/tools/build_edges.py
   
   # 2. Build indexes (optional, auto-built if needed)
   python Paper-KG-Pipeline/scripts/tools/build_recall_index.py
   python Paper-KG-Pipeline/scripts/tools/build_novelty_index.py
   
   # 3. Run pipeline
   python Paper-KG-Pipeline/scripts/idea2story_pipeline.py "Your idea"
   ```

### For Developers

1. **New features go in `src/idea2paper/`**, not `scripts/`
2. **Scripts should be thin wrappers** around library functions
3. **Use absolute imports**: `from idea2paper.module import function`
4. **Follow the existing module structure**:
   - `infra/` → cross-cutting concerns (config, logging, APIs)
   - `recall/` → retrieval system
   - `novelty/` → novelty checking
   - `review/` → review and evaluation
   - `pipeline/` → pipeline orchestration

---

## 📚 Related Documentation

- [Output Files Explained](OUTPUT_FILES_EXPLAINED.md) ⭐ **NEW!**
- [Project Overview](00_PROJECT_OVERVIEW.md)
- [Knowledge Graph Construction](01_KG_CONSTRUCTION.md)
- [Recall System](02_RECALL_SYSTEM.md)
- [Idea2Story Pipeline](03_IDEA2STORY_PIPELINE.md)

---

## ❓ FAQ

**Q: Why are there so many wrapper files?**  
A: Historical reasons. The codebase was reorganized, but old file locations were kept for backward compatibility. These can be removed after documentation is updated.

**Q: Which entity builder should I use?**  
A: Use `scripts/tools/build_entity_v3.py` (the latest version). It's optimized for ICLR data.

**Q: Where should I add new functionality?**  
A: Add it to `src/idea2paper/` as a proper module. Then create a thin script in `scripts/` if command-line access is needed.

**Q: What's the difference between `src/idea2paper/pipeline/` and `src/idea2paper/application/pipeline/`?**  
A: `application/` is a legacy layer that wraps core modules. Most functionality should use the core modules directly. The application layer may be removed in a future cleanup.

---

**Last Updated**: 2024-02-04  
**Status**: Active - reflects current file organization with cleanup recommendations
