# Phase 0 — 准备 Agent 提示词

**角色定义：** 你是缠论 AI 知识库构建项目的 Phase 0 准备 agent

**项目根目录：** `{PROJECT_ROOT}`

**模型：** Claude Sonnet 1M，上下文充足，无需分批读取

---

## 任务清单（7个子任务）

### 任务 1：一次性读取全部源材料

一次性读取以下全部文件（合计约 138K tokens，对 1M 模型无压力）：

- `graphify-out/GRAPH_REPORT.md`
- `graphify-out-108/GRAPH_REPORT.md`
- `organized-v1/` 下全部 29 个文件
- `systematic/00-GLOSSARY.md`（升级基础）
- `index.md`（v1 的8篇章结构参考）

读取完毕后执行：

- 提取两份图谱中的 god nodes、hyperedges、communities
- 对比两图谱差异，识别 `organized-v1` 遗漏的概念关系

---

### 任务 2：分析 organized-v1 的内容范围

- 为每个 v2 章节映射 `organized-v1` 的源文件
- 识别 v1 中的"归类存疑"内容，决定 v2 中的归属

---

### 任务 3：生成 `00-STRUCTURE.md`

输出路径：`{PROJECT_ROOT}/v2-systematic/00-STRUCTURE.md`

内容要求：

- v2 的 30 个章节列表（编号、标题、所属篇章）
- 每个章节的知识点清单（从图谱 god nodes 和 v1 内容提炼）
- 每个章节对应的 `organized-v1` 源文件列表
- 每个章节对应的 `108/` 原始课文编号列表

---

### 任务 4：生成 `00-GLOSSARY-V2.md`

输出路径：`{PROJECT_ROOT}/v2-systematic/00-GLOSSARY-V2.md`

内容要求：

- 从 `graphify-out` 的概念节点 + `systematic/00-GLOSSARY.md` 升级生成
- 每个定义条目包含：
  - **名称**
  - **精确表述**（不是描述性语言）
  - **来源课文**
  - **修订链**（如有）
  - **最终有效版本：课XX**（有演进历史的概念必须标注）
- 去除 v1 GLOSSARY 中的冗余条目和"见GLOSSARY"自引用

---

### 任务 5：生成 `00-DEPENDENCY.md`

输出路径：`{PROJECT_ROOT}/v2-systematic/00-DEPENDENCY.md`

内容要求：

- 章节依赖关系图（参考设计规格第7节）
- 并行波次划分（共 5 波）
- 每个章节的前置章节"核心定义摘要"（约 200 字/章节）
  > 这些摘要供 Phase 1 agent 作为上下文参考，**无需读取前置章节全文**

---

### 任务 6：生成 `PROGRESS.md`

输出路径：`{PROJECT_ROOT}/v2-systematic/PROGRESS.md`

表格格式规范：

**Phase 1 / Phase 3**（每章节一行）：

```
| Phase | 章节ID | 状态 | 失败次数 | 备注 |
```

初始状态：所有行的"状态"为 `待处理`，"失败次数"为 `0`

**Phase 2**（单行）：

```
| 状态 | 冲突数 | 缺失引用数 | 去重数 | 规则冲突数 | 备注 |
```

初始状态：`待处理`

**Phase 4**（单行状态）：

```
| 状态 | 备注 |
```

初始状态：`待处理`

---

### 任务 7：创建 30 个空草稿文件

输出目录：`{PROJECT_ROOT}/v2-systematic/drafts/`

- 文件命名格式：`{chapter-id}-draft.md`（例如：`P1-01-draft.md`）
- 每个文件首行内容：`<!-- CHAPTER: {chapter-id}, STATUS: pending -->`

---

## 约束

- `00-GLOSSARY-V2.md` 中的定义必须是**精确表述**，不是描述性语言
- `00-DEPENDENCY.md` 中的核心定义摘要必须**自包含**，Phase 1 agent 不读前置章节全文
- `PROGRESS.md` 的初始状态必须**全部为"待处理"**

---

## 完成后汇报

完成后通过 `SendMessage` 向主会话汇报以下内容：

1. 各任务完成状态（逐条列出）
2. `00-STRUCTURE.md` 中 30 个章节的知识点数量总计
3. `00-GLOSSARY-V2.md` 中的定义条目数
4. `00-DEPENDENCY.md` 中的并行波次和每波章节数
5. 是否发现任何异常（如图谱缺失节点、v1 内容冲突等）
