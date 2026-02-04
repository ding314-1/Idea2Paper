# Output 目录文件说明

本文档解释了 `Paper-KG-Pipeline/output/` 目录中 JSON 文件的生成方式和内容。

## 📋 目录

- [知识图谱节点文件](#知识图谱节点文件)
- [知识图谱边文件](#知识图谱边文件)
- [模式文件](#模式文件)
- [Pipeline 输出文件](#pipeline-输出文件)
- [构建流程概览](#构建流程概览)

---

## 知识图谱节点文件

知识图谱包含四种类型的节点，每种节点存储在独立的 JSON 文件中。

### 📄 `nodes_paper.json`

**生成工具**: `scripts/tools/build_entity.py` 或 `scripts/tools/build_entity_v3.py`

**用途**: 包含论文节点及其元数据，包括 skeleton（摘要结构）、tricks（技术细节）和评审信息。

**结构示例**:
```json
{
  "paper_id": {
    "paper_id": "唯一标识符",
    "title": "论文标题",
    "authors": ["作者1", "作者2"],
    "abstract": "论文摘要文本...",
    "skeleton": "结构化摘要表示",
    "tricks": "技术实现细节",
    "review_stats": {
      "rating": 7.5,
      "confidence": 4.0,
      ...
    },
    ...
  }
}
```

**数据来源**:
- v2: `data/{conference}/*_paper_node.json`
- v3: ICLR 数据，来自 `assignments.jsonl` 和集群数据

---

### 📄 `nodes_pattern.json`

**生成工具**: `scripts/tools/build_entity.py` 或 `scripts/tools/build_entity_v3.py`

**用途**: 包含写作模式节点，表示从论文中提取的常见研究范式。

**结构示例**:
```json
{
  "pattern_id": {
    "pattern_id": "唯一标识符",
    "pattern_name": "模式名称",
    "description": "模式描述",
    "skeleton_template": "摘要结构模板",
    "trick_template": "技术方法模板",
    "example_papers": ["paper_id1", "paper_id2"],
    "cluster_size": 42,
    "rank": 5,
    ...
  }
}
```

**数据来源**:
- 从 `patterns_structured.json` 生成，该文件通过聚类论文 skeleton 和 tricks 创建

---

### 📄 `nodes_domain.json`

**生成工具**: `scripts/tools/build_entity.py` 或 `scripts/tools/build_entity_v3.py`

**用途**: 包含研究领域节点，表示广泛的研究方向。

**结构示例**:
```json
{
  "domain_id": {
    "domain_id": "唯一标识符",
    "domain_name": "领域名称",
    "description": "领域描述",
    "related_papers": ["paper_id1", "paper_id2"],
    "keywords": ["关键词1", "关键词2"],
    ...
  }
}
```

**数据来源**:
- 从论文元数据和会议主题提取
- 通过主题建模或论文聚类派生

---

### 📄 `nodes_idea.json`

**生成工具**: `scripts/tools/build_entity.py` 或 `scripts/tools/build_entity_v3.py`

**用途**: 包含从论文中提取的核心创新 Idea 节点，表示关键研究贡献。

**结构示例**:
```json
{
  "idea_id": {
    "idea_id": "唯一标识符",
    "idea_text": "核心想法描述",
    "source_papers": ["paper_id1", "paper_id2"],
    "related_patterns": ["pattern_id1"],
    "novelty_score": 0.85,
    ...
  }
}
```

**数据来源**:
- 从论文摘要、贡献部分提取
- 通过 LLM 分析论文内容生成

---

## 知识图谱边文件

### 📄 `edges.json`

**生成工具**: `scripts/tools/build_edges.py`

**用途**: 包含连接知识图谱中节点的所有边，支持三路召回系统。

**边类型**:
1. **Paper → Idea**: 论文到核心想法
2. **Idea → Domain**: 想法到研究领域
3. **Paper → Pattern**: 论文到写作模式
4. **Domain → Pattern**: 领域到常用模式
5. **Pattern → Idea**: 模式到相关想法
6. **Idea → Idea**: 相似想法（运行时计算）
7. **Paper → Paper**: 相似论文（通过内容相似度计算）

**结构示例**:
```json
{
  "edges": [
    {
      "source": "node_id_1",
      "target": "node_id_2",
      "edge_type": "paper_to_idea",
      "weight": 0.95,
      "metadata": {...}
    },
    ...
  ]
}
```

**支持的召回路径**:
- **路径 1**: Idea → Idea → Pattern（相似想法召回）
- **路径 2**: Idea → Domain → Pattern（基于领域召回）
- **路径 3**: Idea → Paper → Pattern（相似论文召回）

---

## 模式文件

### 📄 `patterns_structured.json`

**生成工具**: `scripts/tools/generate_patterns.py`

**用途**: 包含通过聚类论文 skeleton 和 tricks 创建的结构化模式数据。

**创建过程**:
1. 从论文中提取 skeleton（摘要结构）和 tricks（技术细节）
2. 使用 LLM 聚类相似的 skeleton 和 tricks
3. 生成模式模板和元数据
4. 按集群大小和质量对模式进行排序

**结构示例**:
```json
{
  "patterns": [
    {
      "pattern_id": "pattern_001",
      "skeleton_cluster": {
        "representative": "摘要结构模板",
        "members": ["paper_id1", "paper_id2"],
        "size": 42
      },
      "trick_cluster": {
        "representative": "技术方法模板",
        "members": ["paper_id3", "paper_id4"],
        "size": 38
      },
      "rank": 5,
      ...
    }
  ]
}
```

---

### 📄 `patterns_guide.txt`

**生成工具**: `scripts/tools/generate_patterns.py`

**用途**: 所有模式的可读性指南，包含示例。

**内容**: 每个模式的文本描述，包含使用示例和论文引用。

---

### 📄 `patterns_statistics.json`

**生成工具**: `scripts/tools/generate_patterns.py`

**用途**: 模式分布和质量指标的统计摘要。

---

## Pipeline 输出文件

### 📄 `pipeline_result.json`

**生成工具**: `scripts/idea2story_pipeline.py`（主 pipeline 入口）

**用途**: Pipeline 执行的完整跟踪，包含所有阶段、评审、修正和审计。

**结构示例**:
```json
{
  "run_id": "run_20240204_123456",
  "input_idea": "原始研究想法文本",
  "config": {...},
  "stages": {
    "phase1_pattern_selection": {
      "selected_patterns": [...],
      "recall_candidates": [...],
      ...
    },
    "phase2_story_generation": {
      "generated_stories": [...],
      ...
    },
    "phase3_review": {
      "reviews": [...],
      "scores": {...},
      ...
    },
    "phase4_verification": {
      "collision_check": {...},
      ...
    }
  },
  "final_story": {...},
  "metadata": {
    "duration_seconds": 245.3,
    "llm_calls": 42,
    ...
  }
}
```

---

### 📄 `final_story.json`

**生成工具**: `scripts/idea2story_pipeline.py`（从 pipeline result 提取）

**用途**: 最终生成的研究故事，包含结构化字段，可用于论文写作。

**结构示例**:
```json
{
  "title": "生成的论文标题",
  "abstract": "生成的摘要文本...",
  "problem": "研究问题陈述",
  "method": "提出的方法描述",
  "contributions": ["贡献1", "贡献2"],
  "experiments": "实验设置和结果",
  "pattern_used": "pattern_id",
  "novelty_score": 0.85,
  "review_scores": {
    "relevance": 8.5,
    "clarity": 7.8,
    "novelty": 8.2
  }
}
```

---

### 📄 `log.json`

**生成工具**: 启用日志记录的 Pipeline 执行

**用途**: 详细的执行日志，用于调试和审计。

**结构**: JSONL 格式（每行一个 JSON 对象），带时间戳的事件。

---

## 构建流程概览

### 分步创建流程

```
原始论文数据
    ↓
build_entity_v3.py
    ↓
nodes_paper.json
nodes_pattern.json
nodes_domain.json
nodes_idea.json
    ↓
build_edges.py（读取 nodes_*.json）
    ↓
edges.json
    ↓
idea2story_pipeline.py（读取 edges.json + nodes_*.json）
    ↓
pipeline_result.json
final_story.json
log.json
```

### 构建命令

1. **构建知识图谱节点**（一次性设置）:
   ```bash
   python Paper-KG-Pipeline/scripts/tools/build_entity_v3.py
   ```
   
   创建: `nodes_paper.json`, `nodes_pattern.json`, `nodes_domain.json`, `nodes_idea.json`

2. **构建知识图谱边**（一次性设置）:
   ```bash
   python Paper-KG-Pipeline/scripts/tools/build_edges.py
   ```
   
   创建: `edges.json`

3. **生成模式**（可选，如果步骤1未包含）:
   ```bash
   python Paper-KG-Pipeline/scripts/tools/generate_patterns.py
   ```
   
   创建: `patterns_structured.json`, `patterns_guide.txt`, `patterns_statistics.json`

4. **运行 Pipeline**（生成研究故事）:
   ```bash
   python Paper-KG-Pipeline/scripts/idea2story_pipeline.py "你的研究想法"
   ```
   
   创建: `pipeline_result.json`, `final_story.json`, `log.json`

---

## 文件依赖关系

```
原始数据 (papers/)
    ↓
nodes_*.json (4个文件) ← build_entity_v3.py
    ↓
edges.json ← build_edges.py (读取 nodes_*.json)
    ↓
pipeline 输出 ← idea2story_pipeline.py (读取 edges.json + nodes_*.json)
    ├── pipeline_result.json
    ├── final_story.json
    └── log.json
```

---

## 重要提示

1. **实体构建器版本**:
   - `build_entity.py` (v2): 使用通用会议数据格式
   - `build_entity_v3.py` (v3): ICLR 专用，推荐用于当前设置
   - 仅使用一个版本以避免冲突

2. **索引文件**:
   - `novelty_index/` 和 `recall_index/` 目录包含预计算的 embeddings
   - 通过 `build_novelty_index.py` 和 `build_recall_index.py` 自动构建或手动构建
   - 如果 embedding 模型更改，必须重建

3. **增量更新**:
   - 节点文件: 可以独立重新生成
   - 边文件: 依赖节点文件，节点更新后需重新生成
   - Pipeline 输出: 每次运行生成，非累积

---

## 故障排除

**Q: 运行 pipeline 后缺少输出文件？**
- 检查知识图谱文件是否存在: `nodes_*.json` 和 `edges.json`
- 按顺序运行构建脚本: 先实体，后边
- 检查日志中的错误消息

**Q: Pipeline 结果在不同运行之间有差异？**
- LLM 响应是非确定性的（temperature > 0）
- 设置 `seed` 参数以获得更一致的结果
- 检查索引是否未更改

**Q: 文件大小很大？**
- `nodes_paper.json`: ~15MB（包含完整论文元数据）
- `edges.json`: ~90MB（包含所有图边）
- 这些文件在 pipeline 执行期间加载到内存中

---

## 参考文档

- [知识图谱构建指南](01_KG_CONSTRUCTION_zh.md)
- [召回系统文档](02_RECALL_SYSTEM_zh.md)
- [Idea2Story Pipeline 指南](03_IDEA2STORY_PIPELINE_zh.md)
- [项目概览](00_PROJECT_OVERVIEW_zh.md)
