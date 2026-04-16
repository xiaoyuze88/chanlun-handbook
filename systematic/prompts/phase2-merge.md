# Phase 2 — Merge Agent Prompt

你是缠论整理项目的 **merge agent**。
任务：将本轮指定的 temp/BATCH_XX.md 文件内容，按章节标签机械追加到 drafts/ 对应文件。
**只做追加，不改写内容，不做任何润色。**

---

## 项目根目录

```
/Users/xyz/Sites/private/chzhshch-108-plus/systematic/
```

---

## 本轮待处理的 BATCH

由主会话在派发时指定（最多5个），格式如：
```
本轮处理：BATCH_01, BATCH_02, BATCH_03, BATCH_04, BATCH_05
```

---

## 你的任务

### 步骤1：确认本轮 BATCH 列表

读取 `PHASE2-PROGRESS.md`，对主会话指定的每个 BATCH，确认其状态为"待处理"。
若某 BATCH 已为"完成"（幂等检查），跳过。

### 步骤2：逐批合并

对每个待处理的 BATCH，按以下流程执行：

**2a. 幂等检查**
读取 drafts 文件的**第1行**（仅第1行），格式为 `<!-- MERGED: BATCH_01, BATCH_03 -->`。
如果该 BATCH 编号已出现在第1行的标记中，跳过（已合并）。
⚠️ 只读第1行，不读全文做检查。

**2b. 读取 temp 文件**
读取 `temp/BATCH_XX.md`。
如果文件不存在：
1. 在 `PHASE2-PROGRESS.md` 中将该 BATCH 状态改为"跳过（来源缺失）"
2. 在步骤3汇报中注明
3. 跳过后续步骤，处理下一个 BATCH

**2c. 按标签拆分追加**

从 temp 文件中找出各章节标签下的内容：
- `## [P1-哲学与心理]` 下的内容 → 追加到 `drafts/P1-哲学与心理.md`
- `## [P2-形态学]` 下的内容 → 追加到 `drafts/P2-形态学.md`
- `## [P3-中枢理论]` 下的内容 → 追加到 `drafts/P3-中枢理论.md`
- `## [P4-走势类型]` 下的内容 → 追加到 `drafts/P4-走势类型.md`
- `## [P5-背驰理论]` 下的内容 → 追加到 `drafts/P5-背驰理论.md`
- `## [P6-买卖点体系]` 下的内容 → 追加到 `drafts/P6-买卖点体系.md`
- `## [P7-操作系统]` 下的内容 → 追加到 `drafts/P7-操作系统.md`
- `## [P8-高阶应用]` 下的内容 → 追加到 `drafts/P8-高阶应用.md`
- `## [SKIP]` 下的内容 → 丢弃，不追加

追加时使用 **Edit 工具在文件末尾添加**（不要用 Write 工具整体覆盖），格式：

```markdown

<!-- FROM: BATCH_XX -->
[该 BATCH 在此标签下的所有内容，原样复制]
<!-- END: BATCH_XX -->
```

**2d. 更新第1行的 MERGED 标记**
将 drafts 文件的第1行从 `<!-- MERGED: 旧内容 -->` 更新为 `<!-- MERGED: 旧内容, BATCH_XX -->`。
⚠️ 只修改第1行，不动其他内容。

**2e. 更新进度表**
在 `PHASE2-PROGRESS.md` 中将该 BATCH 状态改为"完成"，记录当前时间。

### 步骤3：汇总汇报

全部处理完成后，向主会话报告：
- 成功合并了哪些 BATCH
- 跳过了哪些 BATCH（原因：幂等/来源缺失）
- 来源缺失的 BATCH 覆盖的课文范围（供主会话填写 PHASE3-PROGRESS 来源缺失列）

**Phase 2 循环终止条件**：所有 BATCH 状态为"完成"或"跳过（来源缺失）"时，Phase 2 结束。

---

## 重要约束

- **只做追加，使用 Edit 工具**：不要用 Write 工具整体覆盖 drafts 文件
- **图片链接原样保留**：`![...](./pic/XXX.png)` 语法一字不改，原样复制
- **MERGED 标记只在第1行**：只修改第1行，不移动、不重复写入
- **不修改 temp/ 文件**：temp 文件只读
- **不修改 PHASE1-PROGRESS.md**：只读
- **不修改 drafts 文件的第2行及以后的头部注释**：只改第1行
