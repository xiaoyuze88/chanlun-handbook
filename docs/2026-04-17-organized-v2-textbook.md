# organized-v2 缠论 AI 知识库构建 — 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 organized-v1 的原文聚合内容提炼为结构化 AI 知识库，生成 organized-v2。知识库的受众是 AI，用途是辅助人类在缠论相关提问和交易决策场景中获得精准、可操作的 AI 回答。

**定位说明：**
- **读者是 AI，不是人类**：内容不需要"循循善诱"，需要"自包含、无歧义、机器可解析"
- **使用场景**：用户向 AI 提问缠论理论 / AI 辅助实盘交易决策
- **核心要求**：定义可判断（充要条件式）、规则可执行（if-then 格式）、冲突有优先级

**Architecture:** 5阶段工作流 — Phase 0 准备（单 subagent）→ Phase 1 知识提炼（按章节并行 subagent，5波）→ Phase 2 交叉验证（单 subagent，全量一次）→ Phase 3 成文输出（按章节并行 subagent，5波）→ Phase 4 附录总论（单 subagent）。主会话只做调度和验证，通过 PROGRESS.md 追踪断点。模型：Claude Sonnet 1M，Phase 0/2/4 全量读入无上下文压力。

**Tech Stack:** Markdown 文件，Agent subagent 并行调度，PROGRESS.md 状态追踪，memory 系统跨会话断点恢复

**Spec:** `docs/superpowers/specs/2026-04-17-organized-v2-textbook-design.md`

---

## 断点恢复协议

本项目跨多个会话执行。每个会话可能中断（上下文窗口、超时、手动停止），必须能从断点恢复。

### 恢复步骤（每个新会话启动时执行）

1. **读取 memory**：读取 `memory/organized-v2-progress.md`，了解当前全局进度
2. **读取 PROGRESS.md**：读取 `v2-systematic/PROGRESS.md`，获取细粒度章节级状态
3. **僵尸清理**：将 PROGRESS.md 中所有"进行中"状态改为"失败"，失败次数+1
4. **从断点继续**：跳到最近一个未完成的 Task 继续执行
5. **更新 memory**：每次完成一个 Phase 或一波后，更新 `memory/organized-v2-progress.md`

### 检查点（Checkpoint）规则

- 每个 Phase 完成后必须 commit + 更新 memory
- Phase 1/3 每波完成后必须 commit + 更新 memory（每波是一个检查点）
- Phase 2 全量交叉验证完成后 commit + 更新 memory
- 单个章节失败不阻塞同波次中无依赖关系的其他章节；进入下一波前，确认下一波所需的前置章节均已完成（失败章节作为缺失依赖，在 DEPENDENCY_SUMMARY 中标注 ⚠️ 前置章节缺失）

### 幂等性保证

- Draft 文件首行标记 `<!-- CHAPTER: {id}, STATUS: completed -->` — 恢复时跳过已完成的
- Verified 文件首行标记 `<!-- CHAPTER: {id}, VERIFIED: yes, ISSUES: {N} -->`
- 最终文件首行标记 `<!-- V2-OUTPUT: {id}, VERIFIED: yes -->`
- 恢复时检查文件首行标记，跳过已完成的章节
- 文件存在但首行非完成态（半成品）→ 恢复时整体覆盖重写，不允许追加

---

## 文件结构

### 新建文件

| 文件 | 职责 |
|------|------|
| `v2-systematic/00-STRUCTURE.md` | v2 章节结构定义，每个章节的知识点清单 |
| `v2-systematic/00-GLOSSARY-V2.md` | v2 精确定义词典（充要条件式表述） |
| `v2-systematic/00-DEPENDENCY.md` | 章节间依赖关系图 + 核心定义摘要 |
| `v2-systematic/00-TEMPLATE.md` | v2 知识点 markdown 模板（面向 AI 知识库格式，供 Phase 1/3 agent 参考） |
| `v2-systematic/PROGRESS.md` | 统一进度表（Phase 1-4） |
| `v2-systematic/prompts/phase0-prep.md` | Phase 0 准备 prompt |
| `v2-systematic/prompts/phase1-extract.md` | Phase 1 知识提炼 prompt 模板 |
| `v2-systematic/prompts/phase2-verify.md` | Phase 2 交叉验证 prompt |
| `v2-systematic/prompts/phase3-output.md` | Phase 3 成文输出 prompt 模板 |
| `v2-systematic/prompts/phase4-appendix.md` | Phase 4 附录总论 prompt |
| `v2-systematic/drafts/{chapter-id}-draft.md` × 30 | Phase 1 输出的章节草稿 |
| `v2-systematic/drafts/{chapter-id}-verified.md` × 30 | Phase 2 输出的验证后草稿 |
| `organized-v2/00-总论.md` | 总论（AI 系统提示词备选：核心公理、查询路由表） |
| `organized-v2/P1-认知基础/01-市场本质与经济人假设.md` 等 × 30 | 最终输出文件 |
| `organized-v2/附录/A-定义速查.md` 等 × 5 | 附录文件（A定义速查、B定理索引、C操作规则总表、D状态条件速查、E原文索引） |

### 只读参考文件

| 文件 | 角色 |
|------|------|
| `organized-v1/*.md` | Phase 1 主要输入 |
| `108/*.md` | Phase 1 交叉验证原文来源 |
| `graphify-out/GRAPH_REPORT.md` | Phase 0 结构设计参考 |
| `graphify-out-108/GRAPH_REPORT.md` | Phase 0/1 概念溯源参考 |
| `systematic/prompts/*.md` | Prompt 设计模式参考 |

---

## Task 1: 创建项目骨架与目录

**Files:**
- Create: `v2-systematic/` 目录结构
- Create: `organized-v2/` 目录结构

- [ ] **Step 1: 创建 v2-systematic 目录**

```bash
mkdir -p v2-systematic/drafts v2-systematic/prompts
```

- [ ] **Step 2: 创建 organized-v2 目录**

```bash
mkdir -p organized-v2/P1-认知基础 organized-v2/P2-形态学 organized-v2/P3-中枢理论 organized-v2/P4-走势与级别 organized-v2/P5-背驰理论 organized-v2/P6-买卖点体系 organized-v2/P7-操作系统 organized-v2/P8-高阶应用 organized-v2/附录
```

- [ ] **Step 3: 验证目录结构**

```bash
ls -R v2-systematic/ organized-v2/
```

Expected: 两个目录树均存在，子目录完整

- [ ] **Step 4: Commit**

```bash
git add v2-systematic/ organized-v2/
git commit -m "chore: create organized-v2 project skeleton directories"
```

---

## Task 2: 编写 v2 模板文件 (00-TEMPLATE.md)

**Files:**
- Create: `v2-systematic/00-TEMPLATE.md`

这是所有 Phase 1/3 agent 共用的知识点模板。**面向 AI 知识库设计，不是教科书叙述格式。**

- [ ] **Step 1: 编写模板文件**

写入 `v2-systematic/00-TEMPLATE.md`，包含以下段落（所有占位内容用 `{说明}` 格式标注）：

- **文件头元数据**：所属篇章、前置知识、核心概念
- **概述**：1-3句，点明本章节在缠论体系中的位置和用途
- **📖 定义**：充要条件式或 if-then 结构，而非描述性语言
  - 格式：`定义：当[条件A] 且 [条件B] 时，称为 X；不满足任一条件则不构成 X`
  - 每条定义附原文引用（1-2句关键原文）
- **📐 定理**：适用条件 + 结论，类似规则库格式
  - 格式：`前提：[状态描述] → 结论：[可推导的市场含义]`
  - 每条定理附原文引用
- **🔍 边界判断**（原"要点辨析"）：用判断树而非叙述段落
  - 格式：遇到 X 情况 → 先判断[条件1]，是则 A，否则判断[条件2]...
  - 重点覆盖：相邻概念的精确区分、定义的最终版本（"以课XX为准"）
- **💡 操作规则**（原"操作指引"）：AI 可直接匹配执行的规则格式
  - 格式：`前提：[状态A] AND [状态B] | 触发：[事件C] | 操作：[步骤] | 冲突优先级：[当X时本规则优先/让位于Y]`
  - 每个操作规则必须有明确的前提状态，无前提的"注意事项"不写入此段
- **📜 经验判断**（可选）：适用于原文无充要条件支撑的经验性知识（如背驰目测判断、走势感觉描述）
  - 格式：`场景：[在什么情况下] | 判断倾向：[原文指引] | 边界：[什么情况下不适用]`
  - 当原文对某概念的定义本身就不是充要的，使用此格式而非强行构造充要条件
- **📎 关联**：前置章节 / 扩展章节 / 对比概念（表格）
- **🖼️ 图示说明**：如涉及原文图片，补写图片的文字描述（"此图描述了 X 情况下的 K 线形态：..."），保留原始图片引用
- **出处索引**：`| 内容要点 | 来源课文 |` 表格

- [ ] **Step 2: 验证模板完整性**

读取 `v2-systematic/00-TEMPLATE.md`，确认包含所有必须段落，且每个段落的格式要求（充要条件式、if-then、判断树）均有示例占位。

- [ ] **Step 3: Commit**

```bash
git add v2-systematic/00-TEMPLATE.md
git commit -m "docs: add v2 knowledge point template"
```

---

## Task 3: 编写 Phase 0 Prompt

**Files:**
- Create: `v2-systematic/prompts/phase0-prep.md`

Phase 0 agent 负责：一次性读取两个知识图谱 + organized-v1 全部文件 + GLOSSARY 基础，生成 v2-systematic/ 下的所有准备文件。

- [ ] **Step 1: 编写 phase0-prep.md**

Prompt 结构参考 `systematic/prompts/phase0-prep.md` 的模式，包含：

```
角色定义：你是缠论 AI 知识库构建项目的 Phase 0 准备 agent
项目根目录：{PROJECT_ROOT}
模型：Claude Sonnet 1M，上下文充足，无需分批读取

任务清单（7个子任务）：

1. 一次性读取以下全部文件（合计约 138K tokens，对 1M 模型无压力）
   - graphify-out/GRAPH_REPORT.md
   - graphify-out-108/GRAPH_REPORT.md
   - organized-v1/ 下全部 29 个文件
   - systematic/00-GLOSSARY.md（升级基础）
   - index.md（v1 的8篇章结构参考）
   提取 god nodes、hyperedges、communities；对比两图谱差异，识别 organized-v1 遗漏的概念关系

2. 分析 organized-v1 的内容范围
   - 为每个 v2 章节映射 organized-v1 的源文件
   - 识别 v1 中的"归类存疑"内容，决定 v2 中的归属

3. 生成 00-STRUCTURE.md
   - v2 的30个章节列表（编号、标题、所属篇章）
   - 每个章节的知识点清单（从图谱 god nodes 和 v1 内容提炼）
   - 每个章节对应的 organized-v1 源文件列表
   - 每个章节对应的 108/ 原始课文编号列表

4. 生成 00-GLOSSARY-V2.md
   - 从 graphify-out 的概念节点 + systematic/00-GLOSSARY.md 升级
   - 每个定义：名称、精确表述、来源课文、修订链（如有）、**最终有效版本：课XX**（有演进历史的概念必须标注）
   - 去除 v1 GLOSSARY 中的冗余和"见GLOSSARY"自引用

5. 生成 00-DEPENDENCY.md
   - 章节依赖关系图（参考设计规格第7节）
   - 并行波次划分（5波）
   - 每个章节的前置章节"核心定义摘要"（约200字/章节）
     这些摘要供 Phase 1 agent 作为上下文参考，无需读取前置章节全文

6. 生成 PROGRESS.md，表格格式如下：
   - Phase 1/3：每行格式 `| Phase | 章节ID | 状态 | 失败次数 | 备注 |`，初始状态全为"待处理"，失败次数全为0
   - Phase 2：单行格式 `| 状态 | 冲突数 | 缺失引用数 | 去重数 | 规则冲突数 | 备注 |`，初始状态为"待处理"
   - Phase 4：单行状态

7. 在 v2-systematic/drafts/ 下创建30个空草稿文件
   文件名：{chapter-id}-draft.md（如 P1-01-draft.md）
   首行：<!-- CHAPTER: {chapter-id}, STATUS: pending -->

约束：
- 00-GLOSSARY-V2.md 中的定义必须是精确表述，不是描述性语言
- 00-DEPENDENCY.md 中的核心定义摘要必须自包含，Phase 1 agent 不读前置章节全文
- PROGRESS.md 的初始状态必须全部为"待处理"

完成后通过 SendMessage 向主会话汇报：
- 各任务完成状态
- 00-STRUCTURE.md 中30个章节的知识点数量总计
- 00-GLOSSARY-V2.md 中的定义条目数
- 00-DEPENDENCY.md 中的并行波次和每波章节数
- 是否发现任何异常
```

- [ ] **Step 2: Commit**

```bash
git add v2-systematic/prompts/phase0-prep.md
git commit -m "docs: add Phase 0 prep prompt"
```

---

## Task 4: 编写 Phase 1 Prompt 模板

**Files:**
- Create: `v2-systematic/prompts/phase1-extract.md`

Phase 1 是核心提炼阶段，每个章节由独立 agent 处理。Prompt 需要 `{{VARIABLE}}` 占位。

- [ ] **Step 1: 编写 phase1-extract.md**

Prompt 结构参考 `systematic/prompts/phase1-batch.md`，包含：

```
角色定义：你是缠论 AI 知识库构建专家，负责将散落的原文聚合提炼为 AI 可直接读取和执行的结构化知识
项目根目录：{PROJECT_ROOT}
受众说明：本知识库的读者是 AI，不是人类。内容不需要"循循善诱"，需要"自包含、无歧义、机器可解析"

模板变量：
- {{CHAPTER_ID}}：章节编号（如 P1-01）
- {{CHAPTER_TITLE}}：章节标题（如 市场本质与经济人假设）
- {{PART_NAME}}：所属篇章（如 P1-认知基础）
- {{V1_SOURCE_FILES}}：organized-v1 中对应的源文件路径列表
- {{LESSON_NUMBERS}}：涉及的108课编号列表（如 课001,课005,课010）
- {{DEPENDENCY_SUMMARY}}：前置章节的核心定义摘要（来自 00-DEPENDENCY.md）

步骤：

1. 读取参考文件
   - 读取 v2-systematic/00-TEMPLATE.md（输出格式规范）
   - 读取 v2-systematic/00-GLOSSARY-V2.md（精确定义参考）
   - 读取 {{DEPENDENCY_SUMMARY}}（前置知识上下文）

2. 读取源材料
   - 读取 {{V1_SOURCE_FILES}} 中列出的 organized-v1 文件
   - 这些文件包含按主题聚合的原文摘录和问答

3. 按需交叉验证
   - 对于定义或定理有疑问时，读取 108/ 中 {{LESSON_NUMBERS}} 对应的原始课文
   - 使用 Read 工具读取，只读相关课文，不读全部
   - 当 organized-v1 的摘录不够完整或存在"归类存疑"标记时，务必读原文

4. 提炼知识（核心步骤）
   按照 00-TEMPLATE.md 的格式，逐段落提炼：

   a. 📖 定义：充要条件式或 if-then 结构
      - 格式：「当[条件A] 且 [条件B] 时，称为 X；不满足任一条件则不构成 X」
      - 每个定义附原文引用（1-2句关键原文）
      - 如果定义已在 GLOSSARY-V2 中，引用 GLOSSARY 而不重复
      - 禁止使用描述性语言，禁止"大概是"、"通常表现为"等模糊表述

   b. 📐 定理：前提 + 结论的规则库格式
      - 格式：「前提：[状态描述] → 结论：[可推导的市场含义]」
      - 每条定理附原文引用（1-2句）
      - 如果原文对同一定理有多课补充，合并为一条完整陈述（注明"以课XX为准"）

   c. 🔍 边界判断：判断树格式，而非叙述段落
      - 格式：「遇到 X 情况 → 先判断[条件1]：是→A，否→判断[条件2]：是→B，否→C」
      - 必须覆盖：与相邻概念的精确区分边界、定义的最终版本说明

   d. 💡 操作规则：AI 可直接匹配执行的规则格式
      - 格式：
        ```
        前提：[状态A] AND [状态B]
        触发：[事件C]
        操作：[步骤1] → [步骤2] → ...
        冲突优先级：当[冲突情况]时，本规则[优先/让位于][规则X]
        ```
      - 每条操作规则必须有明确的前提状态
      - 禁止写无前提的"注意"、"感觉"等模糊指引
      - 如果原文缺乏足够素材，写"本节暂缺可执行操作规则"而不是编造

   e. 🖼️ 图示说明：原文涉及图片时
      - 保留原始图片引用 ![...](../../108/pic/XXX.png)
      - 紧随图片引用，补写文字描述：「此图描述了[X情况]下的K线形态：[描述]」
      - AI 无法解析图片，文字描述是关键

   f. 📎 关联：基于 00-DEPENDENCY.md 和知识图谱中的关系，填写前置/扩展/对比

5. 写入草稿文件
   - 写入 v2-systematic/drafts/{{CHAPTER_ID}}-draft.md
   - 首行标记：<!-- CHAPTER: {{CHAPTER_ID}}, STATUS: completed -->
   - 底部出处索引表：列出所有引用的课文编号

约束（必须遵守）：
- 不要照搬原文大段引用，每个原文引用不超过2句
- 不要引入原文未提及的概念
- 不要编造原文中不存在的定理或推导
- 定义必须是充要条件式，不能是描述性语言；若原文对该概念无充要条件定义，使用📜经验判断格式，标注"原文未给出充要条件"
- 当一个概念在多课中有演进时，以 GLOSSARY-V2 中"最终有效版本"字段所指的课文为准提炼充要条件；历史版本降级写入🔍边界判断，标注"以课XX为准，本条为历史版本"
- 定理必须有明确的前提条件，不能是无条件结论
- 操作规则必须有明确的触发前提，无法确定前提时标记 ⚠️

完成报告：
- 定义数量、定理数量、操作规则数量
- 引用的课文编号列表
- 任何无法确认的存疑点（标记 ⚠️）
- 是否有因缺乏原文素材而省略的段落
```

- [ ] **Step 2: Commit**

```bash
git add v2-systematic/prompts/phase1-extract.md
git commit -m "docs: add Phase 1 extraction prompt template"
```

---

## Task 5: 编写 Phase 2 Prompt

**Files:**
- Create: `v2-systematic/prompts/phase2-verify.md`

- [ ] **Step 1: 编写 phase2-verify.md**

```
角色定义：你是缠论 AI 知识库交叉验证专员，负责检查30个章节草稿之间的一致性、完整性和 AI 可用性
项目根目录：{PROJECT_ROOT}
模型：Claude Sonnet 1M，上下文充足，全部 30 个 draft 约 22K tokens，一次性全量读入

输入：v2-systematic/drafts/ 下全部 {chapter-id}-draft.md 文件（30个）+ 00-GLOSSARY-V2.md
输出：v2-systematic/drafts/ 下对应的 {chapter-id}-verified.md 文件（30个）

步骤：

1. 一次性读取全部 30 个 draft 文件 + 00-GLOSSARY-V2.md + 00-STRUCTURE.md
   无需分批，全量读入后在同一上下文中完成所有验证，有利于发现跨篇章的定义冲突

2. 定义一致性检查
   - 扫描所有 draft 中的"📖 定义"段落
   - 同一概念在不同章节中出现时，定义是否一致
   - 与 GLOSSARY-V2 中的定义是否矛盾
   - 不一致处标记 ⚠️ 定义冲突：{概念}在{章节A}表述为...，在{章节B}表述为...

3. 定理引用检查
   - 扫描所有"📐 定理"段落，确认每条定理有原文引用
   - 无原文引用标记 ⚠️ 定理缺少引用：定理{n}.{m}未标注原文来源

4. 操作规则冲突检查（AI 知识库关键验证）
   - 扫描所有"💡 操作规则"段落
   - 识别在相同前提条件下、不同章节给出不同操作的规则
   - 标记 ⚠️ 规则冲突：在[前提X]下，{章节A}指示[操作A]，{章节B}指示[操作B]，需补充优先级
   - 如果原文有明确优先级，直接写入；无法判断则保留标记供人工确认

5. 跨章节去重（仅限非核心段落）
   - **不对📖定义、📐定理、💡操作规则做链接替换**——这三类核心知识允许适度冗余，以维持章节自包含性
   - 仅对纯重复的解释性段落（非核心背景描述）做链接替换：→ 详见 [{章节标题}]({文件路径})
   - 不去除互补视角的重复（同一概念从不同角度论述）

6. 模板合规性检查
   - 每个 draft 必须包含：概述、📖 定义、📐 定理、🔍 边界判断、💡 操作规则、📎 关联、出处索引
   - 缺失段落标记 ⚠️ 缺失段落：{章节}缺少{段落名}
   - 检查定义是否为充要条件式（发现描述性定义标记 ⚠️ 定义格式不合规）
   - 检查操作规则是否有明确前提（发现无前提规则标记 ⚠️ 操作规则缺少前提）

7. 写入 verified 文件
   - 对每个 draft，将验证修改应用后写入 {chapter-id}-verified.md
   - 保留所有 ⚠️ 标记供人工审阅
   - 首行标记：<!-- CHAPTER: {id}, VERIFIED: yes, ISSUES: {N} -->

8. 更新 PROGRESS.md Phase 2 状态
   - 填写：状态=完成、冲突数、缺失引用数、去重数、规则冲突数

9. 完成后通过 SendMessage 向主会话汇报验证报告摘要
   - 定义冲突数量、规则冲突数量、缺失引用数量、去重数量、缺失段落数量
   - 格式不合规的定义和操作规则数量
   - 需要人工处理的 ⚠️ 列表

约束：
- 不重写内容，只标记问题和做链接替换
- 不删除任何内容，只标记或替换
- ⚠️ 标记必须包含具体描述，不能只写"有问题"
- 规则冲突标记必须说明具体的冲突前提条件
```

- [ ] **Step 2: Commit**

```bash
git add v2-systematic/prompts/phase2-verify.md
git commit -m "docs: add Phase 2 verification prompt"
```

---

## Task 6: 编写 Phase 3 Prompt 模板

**Files:**
- Create: `v2-systematic/prompts/phase3-output.md`

- [ ] **Step 1: 编写 phase3-output.md**

```
角色定义：你是缠论 AI 知识库成文格式化专员，负责将验证后的草稿转为最终输出文件
项目根目录：{PROJECT_ROOT}

模板变量：
- {{CHAPTER_ID}}：章节编号
- {{CHAPTER_TITLE}}：章节标题
- {{PART_NAME}}：所属篇章
- {{OUTPUT_PATH}}：最终输出文件路径（如 organized-v2/P1-认知基础/01-市场本质与经济人假设.md）
- {{DEPENDENCY_SUMMARY}}：前置章节的核心定义摘要

步骤：

1. 读取参考文件
   - 读取 v2-systematic/00-TEMPLATE.md
   - 读取 v2-systematic/drafts/{{CHAPTER_ID}}-verified.md

2. 格式化输出
   - 严格按照 00-TEMPLATE.md 格式
   - 确保 markdown 语法正确（标题层级、表格、引用块）
   - 将 ⚠️ 标记保留为 HTML 注释 <!-- ⚠️ ... --> 不在正文中显示

3. 补充关联链接
   - 📎 关联表格中的概念链接指向具体文件路径
   - 格式：[{概念}](相对路径) 如 [中枢定义](../P3-中枢理论/08-中枢定义.md)

4. 生成出处索引表
   - 从 verified draft 底部提取所有引用的课文编号
   - 格式：| 内容要点 | 来源课文 |

5. 写入最终文件
   - 写入 {{OUTPUT_PATH}}
   - 首行标记：<!-- V2-OUTPUT: {{CHAPTER_ID}}, VERIFIED: yes -->

6. 更新 PROGRESS.md
   - 将 Phase 3 中该章节状态改为"完成"

约束：
- 不修改 verified draft 中的知识内容，只做格式化和链接补充
- 图片路径统一为 ../../108/pic/XXX.png
- 关联链接必须是有效路径
```

- [ ] **Step 2: Commit**

```bash
git add v2-systematic/prompts/phase3-output.md
git commit -m "docs: add Phase 3 output prompt template"
```

---

## Task 7: 编写 Phase 4 Prompt

**Files:**
- Create: `v2-systematic/prompts/phase4-appendix.md`

- [ ] **Step 1: 编写 phase4-appendix.md**

```
角色定义：你是缠论 AI 知识库附录与总论编写专员
项目根目录：{PROJECT_ROOT}

前置条件：Phase 3 全部完成，organized-v2/ 下所有章节文件已生成

步骤：

1. 读取 organized-v2/ 下所有最终文件（一次性全量读入，约 30K tokens，无需分批）

2. 编写 00-总论.md（定位：AI 系统提示词的备选来源，高密度精确内容）
   - 缠论体系全景架构（文本/mermaid，说明各模块间的逻辑依赖关系）
   - 核心公理列表（走势终完美、自同构性等，每条精确表述，不超过1句）
   - 查询路由表：用户问题类型 → 对应章节映射
     格式：| 问题类型 | 首查章节 | 辅助章节 |
     例：| 判断当前是否构成中枢 | P3-中枢定义 | P2-线段 |
   - 概念层次结构：哪些概念是其他概念的前置（供 AI 回答"X 依赖 Y 吗"）
   - 各篇章概要与章节列表（每篇1-2句 + 所含章节索引）

3. 编写 附录/A-定义速查.md
   - 从所有章节文件中提取"📖 定义"段落
   - 按所属篇章顺序排列（P1→P8），同篇章内按章节编号
   - 每条：定义名称 | 精确表述（充要条件式） | 所在章节链接 | 来源课文

4. 编写 附录/B-定理索引.md
   - 从所有章节文件中提取"📐 定理"段落
   - 按所属篇章顺序排列（P1→P8），同篇章内按章节编号
   - 每条：定理名称 | 前提条件 | 结论 | 所在章节链接 | 来源课文

5. 编写 附录/C-操作规则总表.md（新增，AI 辅助决策核心入口）
   - 从所有章节文件中提取"💡 操作规则"段落
   - 按操作场景分组（如：建仓场景、离场场景、风控场景）
   - 每条格式：
     ```
     规则ID：{篇章}-{序号}
     前提：[状态描述]
     触发：[事件]
     操作：[步骤]
     冲突优先级：[如有]
     来源章节：[链接]
     ```
   - 如有已标记的 ⚠️ 规则冲突，在此附录中单独列出"待确认冲突"区域

6. 编写 附录/D-状态条件速查.md（新增，实盘辅助决策入口）
   - 覆盖常见市场状态组合，映射到适用的操作规则
   - 格式：给定[当前状态组合] → 可能适用的操作规则列表（含优先级）
   - 状态维度包括：走势类型、级别、中枢状态、背驰情况、买卖点位置
   - 每个状态组合的操作规则引用到附录C的规则ID

7. 编写 附录/E-原文索引.md
   - 从所有章节的出处索引表汇总
   - 格式：| 课文编号 | 课文标题 | 涉及v2章节列表 |
   - 按课文编号排序

约束：
- 总论字数不限，以信息密度优先（原"不超过2000字"限制取消，AI 读者不怕长）
- 附录A、B中的表述必须与正文中完全一致，不可改写
- 附录C中必须包含所有"💡 操作规则"，不可遗漏
- 附录D的状态组合必须来自原文场景，不可编造
```

- [ ] **Step 2: Commit**

```bash
git add v2-systematic/prompts/phase4-appendix.md
git commit -m "docs: add Phase 4 appendix prompt"
```

---

## Task 8: 执行 Phase 0 — 准备

**Files:**
- Create: `v2-systematic/00-STRUCTURE.md`
- Create: `v2-systematic/00-GLOSSARY-V2.md`
- Create: `v2-systematic/00-DEPENDENCY.md`
- Create: `v2-systematic/PROGRESS.md`
- Create: `v2-systematic/drafts/*.md` (30个空草稿文件)

- [ ] **Step 1: 派发 Phase 0 agent**

使用 Agent 工具派发 1 个独立 subagent（`run_in_background: true`），prompt 来自 `v2-systematic/prompts/phase0-prep.md`（手动替换 `{PROJECT_ROOT}` 为 `/Users/xyz/Sites/private/chzhshch-108-plus`）。

Agent 一次性读入全部输入（合计约 138K tokens，对 1M 模型容量无压力，无需拆分）：
- `graphify-out/GRAPH_REPORT.md`
- `graphify-out-108/GRAPH_REPORT.md`
- `organized-v1/` 全部 29 个文件
- `systematic/00-GLOSSARY.md`（作为 GLOSSARY-V2 的基础）
- `index.md`（v1 的8篇章结构）

Agent 完成后通过 SendMessage 汇报，主会话等待消息后再继续。

- [ ] **Step 2: 验证 Phase 0 输出**

检查以下文件是否存在且非空：
- `v2-systematic/00-STRUCTURE.md` — 应包含30个章节定义
- `v2-systematic/00-GLOSSARY-V2.md` — 应包含精确定义条目
- `v2-systematic/00-DEPENDENCY.md` — 应包含5波并行划分和核心定义摘要
- `v2-systematic/PROGRESS.md` — 应包含 Phase 1-4 的进度表
- `v2-systematic/drafts/` — 应有30个空草稿文件

- [ ] **Step 3: 更新 PROGRESS.md Phase 0 状态**

手动将 PROGRESS.md 中 Phase 0 状态改为"完成"。

- [ ] **Step 4: 更新 memory 进度**

更新 `memory/organized-v2-progress.md`：
- 当前状态改为 "Phase 0 已完成"
- 已完成 Phase 增加 Phase 0
- 下一步改为 "Task 9 - Phase 1 第一波 P1+P2"

- [ ] **Step 5: Commit**

```bash
git add v2-systematic/
git commit -m "feat: Phase 0 complete - structure, glossary, dependency, progress files"
```

---

## Task 9: 执行 Phase 1 第一波 — P1 + P2 并行

**Files:**
- Create: `v2-systematic/drafts/P1-01-draft.md` ~ `P1-03-draft.md`
- Create: `v2-systematic/drafts/P2-04-draft.md` ~ `P2-07-draft.md`

Phase 1 按依赖关系分5波执行。第一波 P1（3章节）和 P2（4章节）可并行，共 7 个 agent。

- [ ] **Step 1: 恢复检查**

读取 PROGRESS.md，检查 P1/P2 各章节状态。跳过已标记"完成"的章节。将"进行中"全改为"失败"。

- [ ] **Step 2: 准备每个章节的 prompt 参数**

从 `00-STRUCTURE.md` 和 `00-DEPENDENCY.md` 中提取每个章节的：
- `{{CHAPTER_ID}}`：如 P1-01
- `{{CHAPTER_TITLE}}`：如 市场本质与经济人假设
- `{{PART_NAME}}`：如 P1-认知基础
- `{{V1_SOURCE_FILES}}`：从 00-STRUCTURE.md 中读取映射
- `{{LESSON_NUMBERS}}`：从 00-STRUCTURE.md 中读取映射
- `{{DEPENDENCY_SUMMARY}}`：从 00-DEPENDENCY.md 中读取（P1/P2 无前置依赖，摘要为空）

- [ ] **Step 3: 并行派发 Phase 1 agent**（仅未完成的章节）

使用 Agent 工具，并行派发 agent 处理 P1-01, P1-02, P1-03, P2-04, P2-05, P2-06, P2-07 中未完成的章节。
每个 agent 的 prompt 来自 `phase1-extract.md`，变量替换后使用。

设置 `run_in_background: true`，监控 agent 完成。

- [ ] **Step 4: 等待 agent 完成，验证输出**

对每个完成的 agent：
- 检查 draft 文件是否存在且非空
- 检查首行标记 `<!-- CHAPTER: ..., STATUS: completed -->`
- 检查是否包含 📖 定义 和 📐 定理 段落
- 更新 PROGRESS.md 中对应章节状态为"完成"

- [ ] **Step 5: 处理失败 agent**

对失败的章节：
- 更新 PROGRESS.md 失败次数+1
- 重新派发 agent（最多重试2次）
- 超过2次失败标记为"跳过"

- [ ] **Step 6: 更新 memory 进度**

更新 `memory/organized-v2-progress.md`：
- Phase 1 状态改为"第一波完成"
- 记录已完成的章节列表

- [ ] **Step 7: Commit**

```bash
git add v2-systematic/drafts/
git commit -m "feat: Phase 1 wave 1 - P1 + P2 drafts complete"
```

---

## Task 10: 执行 Phase 1 第二波 — P3

**Files:**
- Create: `v2-systematic/drafts/P3-08-draft.md` ~ `P3-11-draft.md`

P3 依赖 P2，必须等第一波完成。

- [ ] **Step 1: 恢复检查**

读取 PROGRESS.md，检查 P3 各章节状态。跳过已完成的。将"进行中"全改为"失败"。

- [ ] **Step 2: 准备 P3 的 prompt 参数**

- `{{DEPENDENCY_SUMMARY}}`：从 00-DEPENDENCY.md 中提取 P2 的核心定义摘要（分型、笔、线段的关键定义）

- [ ] **Step 3: 并行派发4个 Phase 1 agent**（仅 P3 中未完成的章节）

- [ ] **Step 4: 验证输出，处理失败，更新 PROGRESS.md**

- [ ] **Step 5: 更新 memory 进度 + Commit**

```bash
git add v2-systematic/drafts/
git commit -m "feat: Phase 1 wave 2 - P3 drafts complete"
```

---

## Task 11: 执行 Phase 1 第三波 — P4 + P5 并行

**Files:**
- Create: `v2-systematic/drafts/P4-12-draft.md` ~ `P4-15-draft.md`
- Create: `v2-systematic/drafts/P5-16-draft.md` ~ `P5-19-draft.md`

P4 和 P5 均依赖 P3，可并行。

- [ ] **Step 1: 恢复检查**

读取 PROGRESS.md，检查 P4/P5 各章节状态。跳过已完成的。将"进行中"全改为"失败"。

- [ ] **Step 2: 准备 P4/P5 的 prompt 参数**

- `{{DEPENDENCY_SUMMARY}}`：从 00-DEPENDENCY.md 中提取 P3 的核心定义摘要（中枢定义、中枢定理的关键结论）

- [ ] **Step 3: 并行派发8个 Phase 1 agent**（仅 P4/P5 中未完成的章节）

- [ ] **Step 4: 验证输出，处理失败，更新 PROGRESS.md**

- [ ] **Step 5: 更新 memory 进度 + Commit**

---

## Task 12: 执行 Phase 1 第四波 — P6

**Files:**
- Create: `v2-systematic/drafts/P6-20-draft.md` ~ `P6-23-draft.md`

P6 依赖 P4 + P5。

- [ ] **Step 1: 恢复检查**

读取 PROGRESS.md，检查 P6 各章节状态。跳过已完成的。将"进行中"全改为"失败"。

- [ ] **Step 2: 准备 P6 的 prompt 参数**

- `{{DEPENDENCY_SUMMARY}}`：从 00-DEPENDENCY.md 中提取 P4（走势类型、级别）和 P5（背驰）的核心定义摘要

- [ ] **Step 3: 并行派发4个 Phase 1 agent**（仅 P6 中未完成的章节）

- [ ] **Step 4: 验证输出，处理失败，更新 PROGRESS.md**

- [ ] **Step 5: 更新 memory 进度 + Commit**

---

## Task 13: 执行 Phase 1 第五波 — P7 + P8

**Files:**
- Create: `v2-systematic/drafts/P7-24-draft.md` ~ `P7-27-draft.md`
- Create: `v2-systematic/drafts/P8-28-draft.md` ~ `P8-30-draft.md`

P7 依赖 P6，P8 依赖 P6+P7。先并行执行 P7 全部4章节，P7 全部完成后再并行执行 P8 全部3章节。

- [ ] **Step 1: 恢复检查**

读取 PROGRESS.md，检查 P7/P8 各章节状态。跳过已完成的。将"进行中"全改为"失败"。

- [ ] **Step 2: 准备 P7/P8 的 prompt 参数**

- `{{DEPENDENCY_SUMMARY}}`：从 00-DEPENDENCY.md 中提取 P6 的核心定义摘要（买卖点体系）

- [ ] **Step 3: 分两批派发 Phase 1 agent**

第一批：并行派发 P7 的4个章节（P7-24, P7-25, P7-26, P7-27 中未完成的），等待全部完成。
第二批：P7 全部完成后，并行派发 P8 的3个章节（P8-28, P8-29, P8-30 中未完成的）。

- [ ] **Step 4: 验证输出，处理失败，更新 PROGRESS.md**

- [ ] **Step 5: 更新 memory 进度 + Commit**

---

## Task 14: 执行 Phase 2 — 交叉验证

**Files:**
- Create: `v2-systematic/drafts/{chapter-id}-verified.md` × 30
- Modify: `v2-systematic/PROGRESS.md`

Phase 2 由单个 subagent 一次性读入全部 30 个 draft（约 22K tokens，对 1M 模型容量无压力）进行全局验证，无需分批。全量比对能发现跨篇章的定义冲突，质量优于逐对分批验证。

- [ ] **Step 1: 恢复检查**

读取 PROGRESS.md，检查 Phase 2 状态。如果已完成，跳过此 Task。
检查 verified 文件是否已存在（若存在则 Phase 2 可能已部分完成，需确认是否全部 30 个均已生成）。

- [ ] **Step 2: 派发 Phase 2 agent — 全量一次**

使用 Agent 工具派发 1 个独立 subagent（`run_in_background: true`），prompt 来自 `v2-systematic/prompts/phase2-verify.md`。

Agent 一次性读入全部 30 个 draft 文件 + GLOSSARY-V2，完成后通过 SendMessage 汇报验证报告摘要（冲突数、缺失引用数、去重数）。主会话等待消息后再继续。

- [ ] **Step 3: 验证 Phase 2 输出**

检查：
- 30个 verified 文件是否全部生成
- 每个 verified 文件首行标记 `<!-- CHAPTER: ..., VERIFIED: yes, ISSUES: {N} -->`
- 统计所有 ⚠️ 标记的数量和类型

- [ ] **Step 4: 更新 memory 进度 + Commit**

```bash
git add v2-systematic/drafts/
git commit -m "feat: Phase 2 - cross-validation complete"
```

---

## Task 15: 执行 Phase 3 — 成文输出（5波并行）

**Files:**
- Create: `organized-v2/P1-认知基础/01-市场本质与经济人假设.md` 等 × 30
- Modify: `v2-systematic/PROGRESS.md`

Phase 3 的并行波次与 Phase 1 相同，按依赖关系分5波。每波是一个独立的检查点。

- [ ] **Step 1: 恢复检查**

读取 PROGRESS.md，检查 Phase 3 各章节状态。跳过已标记"完成"的章节。将"进行中"全改为"失败"。

- [ ] **Step 2: 第一波 — P1 + P2 并行**（仅未完成的章节）

派发 agent（P1×3 + P2×4 中未完成的），每个 agent 的 prompt 来自 `phase3-output.md`，变量替换后使用。

- [ ] **Step 3: 验证第一波输出**

检查 organized-v2/ 下对应文件是否存在且非空，首行标记 `<!-- V2-OUTPUT: ..., VERIFIED: yes -->`。

- [ ] **Step 4: 第二波 — P3**（仅未完成的章节）

- [ ] **Step 5: 第三波 — P4 + P5**（仅未完成的章节）

- [ ] **Step 6: 第四波 — P6**（仅未完成的章节）

- [ ] **Step 7: 第五波 — P7 + P8**（仅未完成的章节）

每波完成后 commit，作为检查点。

- [ ] **Step 8: 全量验证**

检查 organized-v2/ 下30个文件是否全部生成且非空。

- [ ] **Step 9: 更新 memory 进度 + Commit**

```bash
git add organized-v2/
git commit -m "feat: Phase 3 - all 30 chapter output files complete"
```

---

## Task 16: 执行 Phase 4 — 附录与总论

**Files:**
- Create: `organized-v2/00-总论.md`
- Create: `organized-v2/附录/A-定义速查.md`
- Create: `organized-v2/附录/B-定理索引.md`
- Create: `organized-v2/附录/C-操作规则总表.md`
- Create: `organized-v2/附录/D-状态条件速查.md`
- Create: `organized-v2/附录/E-原文索引.md`

- [ ] **Step 1: 恢复检查**

读取 PROGRESS.md，检查 Phase 4 状态。如果已完成，跳过此 Task。
检查附录文件是否已存在。

- [ ] **Step 2: 派发 Phase 4 agent**

使用 Agent 工具派发 1 个独立 subagent（`run_in_background: true`），prompt 来自 `v2-systematic/prompts/phase4-appendix.md`。

Agent 一次性读入全部 30 个最终章节文件（约 30K tokens，对 1M 模型容量无压力，无需分批提取）。完成后通过 SendMessage 汇报，主会话等待消息后再继续。

- [ ] **Step 3: 验证 Phase 4 输出**

检查：
- `00-总论.md` 是否包含体系架构、核心公理列表、查询路由表、概念层次结构
- `A-定义速查.md` 条目数是否与所有章节定义数一致
- `B-定理索引.md` 条目数是否与所有章节定理数一致，且每条有前提+结论
- `C-操作规则总表.md` 是否覆盖所有章节的操作规则，是否有待确认冲突区域
- `D-状态条件速查.md` 是否覆盖主要市场状态组合
- `E-原文索引.md` 是否覆盖所有30个章节的出处

- [ ] **Step 4: 更新 PROGRESS.md + memory 进度**

- [ ] **Step 5: Commit**

```bash
git add organized-v2/
git commit -m "feat: Phase 4 - appendix and overview complete"
```

---

## Task 17: 最终质量检查

- [ ] **Step 1: 统计验证**

运行统计脚本，检查：
- organized-v2/ 下文件总数（应为 30 章节 + 1总论 + 5附录 = 36）
- 每个文件是否包含所有必须段落（概述、定义、定理、边界判断、操作规则、关联、出处索引）
- 所有 markdown 链接是否有效

- [ ] **Step 2: 人工抽检**

从每篇章各抽1个文件，检查：
- 定义格式：是否为充要条件式（"当A且B时称为X，不满足则不构成X"），而非描述性语言
- 定理格式：是否有明确前提条件 + 结论，而非孤立结论
- 操作规则：前提是否可供 AI 做条件匹配，是否有冲突优先级字段
- 附录C是否可以独立作为决策规则表使用（不依赖正文也能理解）

- [ ] **Step 3: 更新 PROGRESS.md 标记全部完成 + 更新 memory 为"项目完成"**

- [ ] **Step 4: Final commit**

```bash
git add .
git commit -m "feat: organized-v2 AI knowledge base construction complete"
```
