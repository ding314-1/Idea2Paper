# 问题解答汇总

本文档回答用户提出的三个问题。

---

## 1. 把 LLM API 改成 GitHub Model 的 API

### ✅ 已完成

已经更新系统以支持 GitHub Models API 作为默认选项。

### 配置方式

#### 方案 1：使用 GitHub Models（推荐）

1. **创建 GitHub Personal Access Token (PAT)**
   - 访问 [GitHub Settings → Developer settings → Personal access tokens → Fine-grained tokens](https://github.com/settings/tokens?type=beta)
   - 创建新令牌，选择 `models:read` 权限
   - 复制生成的令牌

2. **配置环境变量**
   
   复制 `.env.example` 到 `.env`，配置如下：
   ```bash
   GITHUB_TOKEN=your_github_pat_here
   LLM_API_URL=https://models.github.ai/inference/chat/completions
   LLM_MODEL=openai/gpt-4o-mini
   ```

3. **运行**
   ```bash
   python Paper-KG-Pipeline/scripts/idea2story_pipeline.py "你的研究想法"
   ```

#### 方案 2：继续使用 SiliconFlow

如果想继续使用 SiliconFlow，配置如下：
```bash
SILICONFLOW_API_KEY=your_key_here
LLM_API_URL=https://api.siliconflow.cn/v1/chat/completions
LLM_MODEL=Pro/zai-org/GLM-4.7
```

#### 方案 3：使用 OpenAI

```bash
LLM_API_KEY=your_openai_key
LLM_API_URL=https://api.openai.com/v1/chat/completions
LLM_MODEL=gpt-4o-mini
```

### API 密钥优先级

系统会按以下顺序查找 API 密钥：
1. `LLM_API_KEY`（如果设置）
2. `GITHUB_TOKEN`（用于 GitHub Models）
3. `SILICONFLOW_API_KEY`（兜底）

### 变更内容

1. **config.py**：更新了默认 API 端点和模型
   - 默认 URL：`https://models.github.ai/inference/chat/completions`
   - 默认模型：`openai/gpt-4o-mini`
   - 支持多个 API 密钥源

2. **.env.example**：添加了详细的配置说明
   - 三种配置方案的示例
   - 清晰的注释说明

3. **README.md/README-zh_CN.md**：更新了配置文档
   - 添加了 GitHub Models 配置步骤
   - 保留了其他提供商的配置方式

### GitHub Models API 特点

- **免费额度**：用于实验和原型开发
- **兼容 OpenAI 格式**：无需修改代码
- **多模型支持**：支持 OpenAI、DeepSeek、Llama 等模型
- **企业级**：支持组织归属和使用跟踪
- **付费升级**：可升级到生产级别额度

---

## 2. 文件组织优化

### 问题分析

原有文件组织存在以下问题：

1. **大量路由文件**：`scripts/` 根目录下有 9 个纯路由文件，只是简单转发到其他目录
2. **功能重复**：多个目录有相同功能的文件
3. **结构混乱**：`scripts/`、`scripts/tools/`、`scripts/demos/` 之间职责不清

### 解决方案

#### 已完成的工作

1. **创建了详细的文档**
   
   新建文件：`Paper-KG-Pipeline/docs/FILE_ORGANIZATION.md`
   
   该文档说明了：
   - 完整的目录结构
   - 每个目录的用途
   - 文件依赖关系
   - 路由文件的存在原因
   - 最佳实践

2. **明确了文件职责**

   ```
   src/idea2paper/        ← 核心代码（可导入的 Python 包）
       ├── infra/         ← 基础设施（LLM、配置、日志）
       ├── pipeline/      ← 核心 Pipeline 逻辑
       ├── recall/        ← 召回系统
       ├── novelty/       ← 新颖性检查
       └── review/        ← 评审系统
   
   scripts/               ← 可执行脚本
       ├── tools/         ← 知识图谱构建和维护工具
       ├── demos/         ← 示例和演示
       ├── dev/           ← 开发工具
       └── legacy/        ← 已废弃的脚本
   ```

3. **路由文件说明**

   以下文件是向后兼容的路由文件（可在文档更新后删除）：
   
   - `scripts/build_edges.py` → `scripts/tools/build_edges.py`
   - `scripts/build_entity.py` → `scripts/tools/build_entity.py`
   - `scripts/build_entity_v3.py` → `scripts/tools/build_entity_v3.py`
   - `scripts/generate_patterns.py` → `scripts/tools/generate_patterns.py`
   - `scripts/demo_pipeline.py` → `scripts/demos/demo_pipeline.py`
   - `scripts/run_pipeline.py` → `scripts/demos/run_pipeline.py`
   - 等等...

#### 推荐的使用方式

**使用规范路径**（而不是路由文件）：

```bash
# ✅ 推荐：使用完整路径
python Paper-KG-Pipeline/scripts/tools/build_entity_v3.py
python Paper-KG-Pipeline/scripts/tools/build_edges.py

# ❌ 避免：使用路由文件（虽然仍可用）
python Paper-KG-Pipeline/scripts/build_edges.py
```

#### 构建流程

```bash
# 1. 构建知识图谱节点（一次性设置）
python Paper-KG-Pipeline/scripts/tools/build_entity_v3.py

# 2. 构建知识图谱边（一次性设置）
python Paper-KG-Pipeline/scripts/tools/build_edges.py

# 3. 构建索引（可选，会自动构建）
python Paper-KG-Pipeline/scripts/tools/build_recall_index.py
python Paper-KG-Pipeline/scripts/tools/build_novelty_index.py

# 4. 运行 Pipeline
python Paper-KG-Pipeline/scripts/idea2story_pipeline.py "你的研究想法"
```

---

## 3. /output 中的 JSON 文件是怎么得来的

### 详细文档

已创建完整的说明文档：
- **英文**：`Paper-KG-Pipeline/docs/OUTPUT_FILES_EXPLAINED.md`
- **中文**：`Paper-KG-Pipeline/docs/OUTPUT_FILES_EXPLAINED_zh.md`

### 快速概览

#### 知识图谱节点文件（4个）

| 文件 | 生成工具 | 内容 |
|------|---------|------|
| `nodes_paper.json` | `build_entity_v3.py` | 论文节点：包含标题、摘要、skeleton、tricks、评审信息 |
| `nodes_pattern.json` | `build_entity_v3.py` | 模式节点：写作模式模板和元数据 |
| `nodes_domain.json` | `build_entity_v3.py` | 领域节点：研究领域和关键词 |
| `nodes_idea.json` | `build_entity_v3.py` | 想法节点：核心创新想法 |

#### 知识图谱边文件

| 文件 | 生成工具 | 内容 |
|------|---------|------|
| `edges.json` | `build_edges.py` | 所有图边：7种边类型，支持三路召回 |

**边类型**：
1. Paper → Idea（论文到想法）
2. Idea → Domain（想法到领域）
3. Paper → Pattern（论文到模式）
4. Domain → Pattern（领域到模式）
5. Pattern → Idea（模式到想法）
6. Idea → Idea（想法相似度）
7. Paper → Paper（论文相似度）

#### Pipeline 输出文件

| 文件 | 生成工具 | 内容 |
|------|---------|------|
| `pipeline_result.json` | `idea2story_pipeline.py` | 完整的 Pipeline 执行跟踪 |
| `final_story.json` | `idea2story_pipeline.py` | 最终生成的研究故事 |
| `log.json` | `idea2story_pipeline.py` | 详细的执行日志 |

#### 模式文件

| 文件 | 生成工具 | 内容 |
|------|---------|------|
| `patterns_structured.json` | `generate_patterns.py` | 聚类后的模式数据 |
| `patterns_guide.txt` | `generate_patterns.py` | 模式使用指南 |
| `patterns_statistics.json` | `generate_patterns.py` | 模式统计信息 |

### 生成流程图

```
原始论文数据 (papers/)
    ↓
【步骤 1】build_entity_v3.py
    ↓
nodes_paper.json  (论文)
nodes_pattern.json (模式)
nodes_domain.json (领域)
nodes_idea.json   (想法)
    ↓
【步骤 2】build_edges.py（读取节点文件）
    ↓
edges.json (图边)
    ↓
【步骤 3】idea2story_pipeline.py（读取图谱）
    ↓
pipeline_result.json (完整结果)
final_story.json     (研究故事)
log.json            (执行日志)
```

### 文件大小参考

- `nodes_paper.json`: ~15MB（包含完整论文元数据）
- `edges.json`: ~90MB（包含所有图边）
- `nodes_idea.json`: ~10MB
- `nodes_pattern.json`: ~0.5MB
- `nodes_domain.json`: ~0.3MB
- `pipeline_result.json`: ~40KB（每次运行）

### 依赖关系

```
必须先构建：
  nodes_*.json (4个文件)
    ↓
然后构建：
  edges.json
    ↓
最后运行：
  idea2story_pipeline.py
```

---

## 总结

### 已完成的改进

1. ✅ **LLM API 迁移到 GitHub Models**
   - 更新了默认配置
   - 添加了多提供商支持
   - 更新了文档和示例

2. ✅ **文件组织优化**
   - 创建了完整的组织文档
   - 明确了文件职责
   - 说明了路由文件的原因
   - 提供了最佳实践指南

3. ✅ **Output 文件说明**
   - 创建了详细的文档（中英文）
   - 解释了所有 JSON 文件的来源
   - 提供了构建流程图
   - 添加了故障排除指南

### 相关文档

- **配置说明**：`.env.example`
- **文件组织**：`Paper-KG-Pipeline/docs/FILE_ORGANIZATION.md`
- **Output 文件说明**：`Paper-KG-Pipeline/docs/OUTPUT_FILES_EXPLAINED_zh.md`
- **项目概览**：`Paper-KG-Pipeline/docs/00_PROJECT_OVERVIEW_zh.md`
- **知识图谱构建**：`Paper-KG-Pipeline/docs/01_KG_CONSTRUCTION_zh.md`
- **召回系统**：`Paper-KG-Pipeline/docs/02_RECALL_SYSTEM_zh.md`
- **Pipeline 说明**：`Paper-KG-Pipeline/docs/03_IDEA2STORY_PIPELINE_zh.md`

---

**更新时间**：2024-02-04  
**完成状态**：所有三个任务已完成
