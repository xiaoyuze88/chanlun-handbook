# Phase 0 准备 Agent 提示词

**角色定义**：你是缠论 AI 知识库构建项目的 Phase 0 准备 agent  
**项目根目录**：`{PROJECT_ROOT}`  
**模型**：Claude Sonnet 1M，上下文充足，无需分批读取

---

## 任务清单（7 个子任务）

### 任务 1：一次性读取全部源材料

一次性读取以下全部文件（合计约 138K tokens，对 1M 模型无压力）：

- `graphify-out/GRAPH_REPORT.md`
- `graphify-out-108/GRAPH_REPORT.md`
- `organized-v1/` 下全部 29 个文件
- `systematic/00-GLOSSARY.md`（升级基础）
- `index.md`（v1 的 8 篇章结构参考）

提取 god nodes、hyperedges、communities；对比两图谱差异，识别 `organized-v1` 遗漏的概念关系

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

升级来源：`graphify-out` 的概念节点 + `systematic/00-GLOSSARY.md`

每个定义条目须包含以下字段：

| 字段 | 说明 |
|------|------|
| 名称 | 术语名称 |
| 精确表述 | 严格定义，不得使用描述性语言 |
| 来源课文 | 出处课文编号 |
| 修订链 | 如有演进历史，列出各版本 |
| 最终有效版本 | **有演进历史的概念必须标注**，格式：`最终有效版本：课XX` |

额外要求：

- 去除 v1 GLOSSARY 中的冗余条目
- 去除"见 GLOSSARY"类自引用条目

---

### 任务 5：生成 `00-DEPENDENCY.md`

输出路径：`{PROJECT_ROOT}/v2-systematic/00-DEPENDENCY.md`

内容要求：

- 章节依赖关系图（参考设计规格第 7 节）
- 并行波次划分（共 5 波）
- 每个章节的前置章节"核心定义摘要"（约 200 字/章节）

> **摘要用途说明**：这些摘要供 Phase 1 agent 作为上下文参考，无需读取前置章节全文。摘要须自包含，确保 Phase 1 agent 仅凭摘要即可理解所有前置概念。

---

### 任务 6：生成 `PROGRESS.md`

输出路径：`{PROJECT_ROOT}/v2-systematic/PROGRESS.md`

表格格式规范：

**Phase 1 / Phase 3**（每章节一行）：

```
| Phase | 章节ID | 状态 | 失败次数 | 备注 |
```

初始值：状态 = `待处理`，失败次数 = `0`

**Phase 2**（单行汇总）：

```
| 状态 | 冲突数 | 缺失引用数 | 去重数 | 规则冲突数 | 备注 |
```

初始值：状态 = `待处理`，其余数值 = `0`

**Phase 4**（单行状态）：

```
| 状态 | 备注 |
```

初始值：状态 = `待处理`

---

### 任务 7：创建 30 个空草稿文件

在 `{PROJECT_ROOT}/v2-systematic/drafts/` 下创建 30 个空草稿文件。

**命名规则**：`{chapter-id}-draft.md`（例如 `P1-01-draft.md`）

**首行格式**：

```
<!-- CHAPTER: {chapter-id}, STATUS: pending -->
```

---

## 约束

- `00-GLOSSARY-V2.md` 中的定义**必须是精确表述**，不得使用描述性语言
- `00-DEPENDENCY.md` 中的核心定义摘要**必须自包含**，Phase 1 agent 不读取前置章节全文
- `PROGRESS.md` 的初始状态**必须全部为"待处理"**

---

## 完成后汇报

完成所有任务后，通过 `SendMessage` 向主会话汇报以下内容：

1. 各任务完成状态（逐一列明）
2. `00-STRUCTURE.md` 中 30 个章节的知识点数量总计
3. `00-GLOSSARY-V2.md` 中的定义条目数
4. `00-DEPENDENCY.md` 中的并行波次和每波章节数
5. 是否发现任何异常（若有，详细描述）
