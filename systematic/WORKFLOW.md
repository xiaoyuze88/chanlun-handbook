# 缠论整理工作流

## 目录结构

```
systematic/
  WORKFLOW.md              ← 本文件，工作流总说明
  PHASE1-PROGRESS.md       ← Phase 1 进度表（由 Phase 0 生成）
  PHASE2-PROGRESS.md       ← Phase 2 进度表（由 Phase 0 生成）
  PHASE3-PROGRESS.md       ← Phase 3 进度表（由 Phase 0 生成）
  00-GLOSSARY.md           ← 精确定义词典（由 Phase 0 从 graphify 填充，人工维护，只读参考）
  00-CONCEPT-MAP.md        ← 概念-课文索引，由 Phase 0 从 graphify 生成
  batches.json             ← 分批配置（由 Phase 0 生成，机器读）
  prompts/
    phase0-prep.md         ← Phase 0 prep agent 完整 prompt
    phase1-batch.md        ← Phase 1 batch agent prompt 模板
    phase2-merge.md        ← Phase 2 merge agent 完整 prompt
    phase3a-structure.md   ← Phase 3 第一步：去重+重排 agent prompt 模板
    phase3b-output.md      ← Phase 3 第二步：拆分+引用+标注 agent prompt 模板
  temp/
    BATCH_01.md            ← batch agent 输出（Phase 1 产物）
    BATCH_02.md
    ...
  drafts/
    P1-哲学与心理.md        ← merge 后的大章节草稿（Phase 2 产物）
    P1-哲学与心理-cleaned.md ← Phase 3a 去重重排后（Phase 3a 产物）
    P2-形态学.md
    P2-形态学-cleaned.md
    ...（每个章节一对）
  final/
    01-市场本质与经济人原则.md    ← Phase 3b 产物（29个小节文件）
    02-当下性哲学.md
    03-贪嗔痴与人性陷阱.md
    04-分型.md
    05-笔.md
    06-线段.md
    07-中枢定义与计算.md
    08-中枢级别扩张.md
    09-中枢震荡与延伸.md
    10-中枢中心定理.md
    11-走势三大类型.md
    12-走势分解定理.md
    13-级别自组织与自相似性.md
    14-同级别分解.md
    15-背驰定义.md
    16-盘整背驰vs趋势背驰.md
    17-MACD辅助判断.md
    18-背驰级别与转折.md
    19-第一类买卖点.md
    20-第二类买卖点.md
    21-第三类买卖点.md
    22-三类买卖点完备性.md
    23-均线吻理论.md
    24-区间套精确定位.md
    25-操作级别选择.md
    26-资金管理与止损.md
    27-牛熊市判断与大资金策略.md
    28-反庄操作.md
    29-实战案例精解.md
```

---

## 分批说明

共 19 批（BATCH_01 ~ BATCH_19），按文件字节大小均衡分组，每批约 80~120KB。
详见 `batches.json`。

---

## 章节路由标签

Phase 1 的 batch agent 按以下 8 个大类标签提取内容：

| 标签 | 内容范围 | Phase 3 后拆分为 |
|------|---------|----------------|
| [P1-哲学与心理] | 市场本质、经济人原则、当下性哲学、交易心理、人性陷阱、修炼自己 | final/01, 02, 03 |
| [P2-形态学] | 顶底分型、笔的构成、线段划分标准（含各版本修订）、特征序列 | final/04, 05, 06 |
| [P3-中枢理论] | 中枢定义、中枢中心定理、中枢级别扩张、中枢震荡与延伸 | final/07, 08, 09, 10 |
| [P4-走势类型] | 走势三分类、走势分解定理、级别体系、同级别分解 | final/11, 12, 13, 14 |
| [P5-背驰理论] | 背驰定义（均线面积法）、盘整背驰、MACD辅助、背驰级别与转折 | final/15, 16, 17, 18 |
| [P6-买卖点体系] | 三类买卖点定义、完备性定理 | final/19, 20, 21, 22 |
| [P7-操作系统] | 均线吻、区间套精确定位、操作级别、资金管理、止损、买卖程序 | final/23, 24, 25, 26 |
| [P8-高阶应用] | 牛熊市大资金策略、反庄操作、实战图解案例 | final/27, 28, 29 |
| [SKIP] | 纯诗歌、纯行情数字点评、无技术内容的纯闲聊 | 不进入 drafts |

---

## 四个阶段

### Phase 0：准备
**执行者**：1 个 prep agent（串行，一次性）
**输入**：graphify-out/ + 108/ 文件列表
**输出**：
- `00-CONCEPT-MAP.md`：概念-课文索引 + 版本修订链 + 超边关系 + 社区聚类
- `00-GLOSSARY.md`：从 graphify 提取的精确定义词典（静态，后续只读）
- `PHASE1-PROGRESS.md`：19 个 BATCH 条目，全部初始为"待处理"
- `PHASE2-PROGRESS.md`：19 个 BATCH 条目，全部初始为"待处理"
- `PHASE3-PROGRESS.md`：8 个章节条目，全部初始为"待处理"
- `drafts/P1~P8.md`：8 个大章节草稿文件（MERGED 标记固定在第1行）
- `final/01~29.md`：29 个小节文件（空模板）

**触发方式**：新会话，手动运行 prompt `prompts/phase0-prep.md`

---

### Phase 1：精读提取
**执行者**：主会话调度 + 最多 5 个并行 batch agent
**输入**：108/ 课文 + `00-CONCEPT-MAP.md` + `00-GLOSSARY.md`
**输出**：`temp/BATCH_XX.md`（每批一个文件）

**调度逻辑**：
1. 每次循环开始，**重新读取** `PHASE1-PROGRESS.md`（不缓存状态）
2. 找出所有"待处理"或"失败（<2次）"的 BATCH
3. 派发最多 5 个（优先失败批次），更新状态为"进行中"，记录启动时间
4. 每个 agent 完成后通知主会话
5. 收到通知后，**立即更新进度表**，再检查是否需要补充派发
6. 校验 `temp/BATCH_XX.md`：存在、非空、且包含至少一个 `## [P` 标签 → 标记"完成"；否则标记"失败（第N次）"
7. 失败次数 ≥ 2 → 标记"异常"，跳过
8. 有空位（运行中 < 5）→ 继续派发，直到全部完成或全部异常

**状态枚举**：`待处理` / `进行中` / `完成` / `失败（第1次）` / `失败（第2次）` / `异常`

**结束条件**：所有 BATCH 为"完成"或"异常"

---

### Phase 2：Merge
**执行者**：主会话调度 + 每轮最多 1 个 merge agent，每轮处理最多 5 个 BATCH
**输入**：`temp/BATCH_XX.md`（所有完成的批次）
**输出**：`drafts/P1~P8.md`（追加内容）

**调度逻辑**：
1. 主会话读 `PHASE2-PROGRESS.md`，找出状态为"待处理"的 BATCH
2. 每次取最多 5 个"待处理" BATCH，派发 1 个 merge agent 处理
3. merge agent 完成后汇报，主会话更新进度
4. 若还有"待处理"BATCH，继续派发新的 merge agent
5. 全部完成后，验证 drafts/P1~P8.md 每个文件均有实质内容

**幂等保证**：drafts/ 文件第1行的 `<!-- MERGED: BATCH_XX -->` 标记，已合并的批次跳过

**结束条件**：所有 BATCH 合并完成

---

### Phase 3：校对
**执行者**：主会话调度，每章节依次执行 2 个串行 agent（Phase 3a → Phase 3b）
**输入**：`drafts/PX.md` + `00-CONCEPT-MAP.md`
**输出**：`drafts/PX-cleaned.md`（3a 产物）→ `final/XX-小节名.md`（3b 产物）

**每章节执行顺序**：
1. **Phase 3a**（结构整理）：去重 + 重排 → 输出 `drafts/PX-cleaned.md`
2. **Phase 3b**（输出生成）：拆分小节 + 加交叉引用 + 标注疑问 → 写入 `final/`

**结束条件**：8 个章节全部 3a+3b 完成，所有 29 个 final 文件均非空

---

## 重要约束

1. **进度表是唯一真相来源**：所有状态必须在任务完成/失败时立即更新；每次调度循环开始时重新读取，不缓存
2. **图片链接全程原样保留**：原文共 43 张图片（28 篇课文含图），格式为 `![...](./pic/XXX.png)`。Phase 1/2/3 全程原样复制，不修改路径。**图片路径修复为人工收尾步骤**（见下方）
3. **temp/ 文件不可修改**：batch agent 写入后只读，Phase 2 全部完成后可清理
4. **merge 幂等**：MERGED 标记固定在 drafts 文件**第1行**，重跑不产生重复内容
5. **review agent 不改写原文**：只在 `### 课XX` 段落块级别移动内容，不拆分单篇课文的内部段落
6. **异常批次不阻塞流程**：Phase 2 跳过异常批次，在 PHASE3-PROGRESS 记录来源缺失
7. **GLOSSARY 只读**：Phase 1 提取前参考，任何 agent 不得写入

---

## 恢复流程（会话中断后）

**步骤0（最优先，必须先执行）**：
将 `PHASE1-PROGRESS.md` 中所有"进行中"状态的 BATCH，改为"失败（第N次）"（N = 当前失败次数 + 1）。这些 agent 在会话中断时状态未知，必须重跑。

**步骤1**：读 `WORKFLOW.md`（本文件）了解全局结构

**步骤2**：读对应阶段的进度表（`PHASE1/2/3-PROGRESS.md`）

**步骤3**：从上次中断处继续派发

无需重头开始，进度表记录了一切。

---

## Phase 3 完成后的人工收尾清单

Phase 3 完成后，final/ 文件尚有以下内容需要人工处理：

1. **图片路径修复**：在项目根目录执行批量替换，将所有 final/ 和 drafts/ 文件中的 `./pic/` 替换为正确的相对路径（如 `../../108/pic/`）或绝对路径
2. **处理 `❓ 归类存疑` 标注**：逐一确认或移动内容到正确章节
3. **处理 `⚠️ 待确认` 交叉引用**：核实 INFERRED 超边是否成立
4. **检查来源缺失警告**：若有异常 BATCH，对应章节头部有 `⚠️ 来源缺失` 标注，需补充阅读原文
5. **验证 29 个 final 文件均非空**：内容明显过少的小节需人工补充
6. **抽样核查出处**：对每个小节的"原始出处索引"抽几条核对，确认内容与来源课文一致
