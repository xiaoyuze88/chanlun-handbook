# 主会话执行指南

> 本文件是新会话的起点。打开新会话后，先读本文件，再读对应阶段的进度表，然后开始执行。

---

## 项目概览

- **目标**：将缠中说禅108篇博客课文整理为系统化的缠论教材（29个小节）
- **根目录**：`/Users/xyz/Sites/private/chzhshch-108-plus/systematic/`
- **详细工作流**：见 `WORKFLOW.md`

---

## 快速定位当前状态

```
1. 读 PHASE1-PROGRESS.md → 看 Phase 1 是否全部完成
2. 读 PHASE2-PROGRESS.md → 看 Phase 2 是否全部完成
3. 读 PHASE3-PROGRESS.md → 看 Phase 3 是否全部完成
```

哪个阶段有未完成项，就从那里继续。

---

## ⚠️ 会话开始前必须执行（无论哪个阶段）

**清理僵尸状态**：将 `PHASE1-PROGRESS.md` 中所有"进行中"状态的 BATCH 改为"失败（第N次）"，N = 当前失败次数 + 1。
上次会话中断时这些 agent 状态未知，必须重跑，不能跳过。

---

## Phase 0 执行（仅运行一次）

**前置条件**：`PHASE1-PROGRESS.md` 不存在

**执行步骤**：

```
1. 读 prompts/phase0-prep.md
2. 启动 1 个 subagent，将 phase0-prep.md 的完整内容作为 prompt
3. 等待 subagent 完成（前台运行，不用 background）
4. 确认以下文件存在：
   - systematic/00-CONCEPT-MAP.md
   - systematic/00-GLOSSARY.md（已由 Phase 0 填充）
   - systematic/PHASE1-PROGRESS.md
   - systematic/PHASE2-PROGRESS.md
   - systematic/PHASE3-PROGRESS.md
   - systematic/drafts/P1~P8.md（8个文件，第1行为 MERGED 标记）
   - systematic/final/01~29.md（29个文件）
```

---

## Phase 1 执行（滚动调度）

**前置条件**：Phase 0 已完成

**每次循环必须重新读取 PHASE1-PROGRESS.md，不得缓存状态。**

**调度伪代码**：

```python
while True:
    读取 PHASE1-PROGRESS.md  # 每次循环重新读，不缓存

    running = 状态为"进行中"的 BATCH 列表
    pending = 状态为"待处理"的 BATCH 列表
    failed  = 状态为"失败（第1次）"或"失败（第2次）"的 BATCH 列表

    if len(running) == 0 and len(pending) == 0 and len(failed) == 0:
        print("Phase 1 完成！")
        break

    # 填满5个槽位（优先派发失败批次）
    可派发数量 = 5 - len(running)
    待派发 = failed[:可派发数量] + pending[:max(0, 可派发数量 - len(failed))]

    for batch_id in 待派发:
        构造 prompt（见下方"构造 batch prompt"）
        启动 background subagent
        更新 PHASE1-PROGRESS.md：状态 → "进行中"，记录启动时间  # 先更新再继续

    等待任意 subagent 完成（收到通知）

    # 收到通知后：先更新进度表，再决定是否补充派发
    for 完成的 batch_id:
        校验 temp/BATCH_XX.md：存在 AND 非空 AND 包含至少一个 "## [P" 标签
        if 校验通过:
            更新状态 → "完成"
        else:
            失败次数 += 1
            if 失败次数 >= 2:
                更新状态 → "异常"
            else:
                更新状态 → "失败（第N次）"
```

**构造 batch prompt**：

读取 `prompts/phase1-batch.md`，替换占位符：
- `{{BATCH_ID}}` → 如 `BATCH_03`
- `{{FILE_LIST}}` → 从 `batches.json` 读取该 BATCH 的 `files` 数组，每个文件名前拼接绝对路径前缀 `/Users/xyz/Sites/private/chzhshch-108-plus/108/`，每行一个完整路径
- `{{LESSON_RANGE}}` → 如 `课016-020`

**注意**：subagent 用 `run_in_background: true`

---

## Phase 2 执行（分轮 merge，每轮最多 5 个 BATCH）

**前置条件**：Phase 1 全部完成（无"待处理"、"进行中"、"失败"状态）

**执行步骤**：

```
while 有"待处理"的 BATCH:
    1. 读 PHASE2-PROGRESS.md
    2. 取最多 5 个"待处理"的 BATCH
    3. 读 prompts/phase2-merge.md，附上本轮待处理的 BATCH 列表
    4. 启动 1 个 merge agent（前台运行）
    5. 等待完成
    6. 确认本轮各 BATCH 状态已更新为"完成"或"跳过（来源缺失）"

全部完成后（所有 BATCH 为"完成"或"跳过"）：
    7. 验证 drafts/P1~P8.md 每个文件均有实质内容
    8. 读取 PHASE1-PROGRESS.md，找出所有状态为"异常"的 BATCH
    9. 对每个异常 BATCH，查询 batches.json 中其覆盖的课文范围（lessons 字段）
    10. 根据课文范围判断影响哪些章节（参考 WORKFLOW.md 的路由标签表）
    11. 更新 PHASE3-PROGRESS.md 对应章节的"⚠️ 来源缺失"列，写入：
        "BATCH_XX（课XXX-XXX）"
```

merge agent 自行维护 PHASE2-PROGRESS.md，主会话验收。

---

## Phase 3 执行（每章 3a+3b 两个 agent，串行）

**前置条件**：Phase 2 全部完成

**执行步骤**（对 P1~P8 依次执行）：

```
对每个章节 PX：
  【Phase 3a — 结构整理】
  1. 读 PHASE3-PROGRESS.md，确认该章节 3a 为"待处理"
  2. 读 prompts/phase3a-structure.md，替换占位符：
     - {{CHAPTER_ID}} → 如 P3
     - {{CHAPTER_NAME}} → 如 中枢理论
     - {{DRAFT_FILE}} → drafts/P3-中枢理论.md
     - {{CLEANED_FILE}} → drafts/P3-中枢理论-cleaned.md
  3. 启动 review agent 3a（前台运行）
  4. 等待完成，确认 drafts/PX-cleaned.md 存在且非空
  5. 更新 PHASE3-PROGRESS.md：3a 状态 → "完成"

  【Phase 3b — 输出生成】
  6. 读 prompts/phase3b-output.md，替换占位符：
     - {{CHAPTER_ID}}、{{CHAPTER_NAME}}、{{CLEANED_FILE}}
     - {{FINAL_FILES}} → 对应小节文件列表 + 关键词（见下方完整映射表）
     - {{MISSING_SOURCES}} → 从 PHASE3-PROGRESS.md 读取该章节的来源缺失信息
  7. 启动 review agent 3b（前台运行）
  8. 等待完成
  9. 确认 final/ 对应小节文件全部非空
  10. 更新 PHASE3-PROGRESS.md：3b 状态 → "完成"
  11. 继续下一章
```

---

## Phase 3 各章节完整映射表（含关键词）

### P1-哲学与心理 → final/01, 02, 03

```
01-市场本质与经济人原则.md
  关键词：经济人、赢家输家、资本市场本质、没有庄家、不会赢钱的废人

02-当下性哲学.md
  关键词：当下性、市场无须分析只要看和干、当下合力、理性是干出来的、不患

03-贪嗔痴与人性陷阱.md
  关键词：贪嗔痴疑慢、喜好死亡陷阱、赌徒心理、患得患失、修炼自己
```

### P2-形态学 → final/04, 05, 06

```
04-分型.md
  关键词：顶分型、底分型、K线形态、分型定义、顶底分型心理因素、中继分型

05-笔.md
  关键词：笔的定义、笔的构成标准、笔的唯一性、顶分型延续为笔、笔延伸条件

06-线段.md
  关键词：线段划分标准、特征序列、线段破坏、版本修订（课45→67→71→78→84）
```

### P3-中枢理论 → final/07, 08, 09, 10

```
07-中枢定义与计算.md
  关键词：中枢定义、三段重叠、次级别走势类型重叠、max/min公式、ZG/ZD/GG/DD

08-中枢级别扩张.md
  关键词：级别扩张定理、中枢升级、两个中枢合并条件、走势级别延续定理

09-中枢震荡与延伸.md
  关键词：中枢震荡、中枢延伸、围绕中枢运动、震荡不破中枢、延伸无限可能

10-中枢中心定理.md
  关键词：中枢中心定理一二、GG DD ZG ZD关系、中枢上移下移、利润模式
```

### P4-走势类型 → final/11, 12, 13, 14

```
11-走势三大类型.md
  关键词：上涨定义、下跌定义、盘整定义、趋势定义、走势三分类

12-走势分解定理.md
  关键词：走势分解定理一、走势分解定理二、任何走势终要完成、盘整上涨下跌连接

13-级别自组织与自相似性.md
  关键词：级别体系（1分5分30分日周月）、自相似性、级别自组织、分形哲学本质

14-同级别分解.md
  关键词：同级别分解、走势多义性、结合律、走势类型连接
```

### P5-背驰理论 → final/15, 16, 17, 18

```
15-背驰定义.md
  关键词：背驰定义、趋势力度、均线面积法、没有趋势没有背驰、缠中说禅趋势平均力度

16-盘整背驰vs趋势背驰.md
  关键词：盘整背驰、趋势背驰、背驰与转折关系、历史性底部、盘整中无背驰

17-MACD辅助判断.md
  关键词：MACD辅助、红绿柱面积对比、顶背驰、底背驰、MACD判断背驰规则

18-背驰级别与转折.md
  关键词：小级别背驰引发大级别转折、背驰力度、转折强度、级别放大效应
```

### P6-买卖点体系 → final/19, 20, 21, 22

```
19-第一类买卖点.md
  关键词：第一类买点、下跌趋势背驰买点、第一类卖点、上涨趋势背驰卖点

20-第二类买卖点.md
  关键词：第二类买点、第一次次级别回调低点、第二类卖点、回抽不过前高

21-第三类买卖点.md
  关键词：第三类买点、离开中枢后不回中枢、ZG突破、第三类卖点、ZD跌破

22-三类买卖点完备性.md
  关键词：三类买卖点完备性、买卖点体系、缠中说禅买卖点定律、买卖点的数学证明
```

### P7-操作系统 → final/23, 24, 25, 26

```
23-均线吻理论.md
  关键词：飞吻、唇吻、湿吻、均线吻定义、5周10周均线、均线买卖系统

24-区间套精确定位.md
  关键词：区间套、精确买卖点定位、多级别嵌套、最小操作级别

25-操作级别选择.md
  关键词：操作级别选择、资金量与级别、持股持币、节奏、中小资金高效操作

26-资金管理与止损.md
  关键词：资金管理、止损原则、三个独立程序、乘法原则、基于走势而非盈亏的止损
```

### P8-高阶应用 → final/27, 28, 29

```
27-牛熊市判断与大资金策略.md
  关键词：牛市操作、熊市判断、大资金阻击、0成本操作、中国特大型牛市三阶段

28-反庄操作.md
  关键词：逗庄家玩、阻击庄家、主力资金食物链、空间时间阻击、无庄家论

29-实战案例精解.md
  关键词：具体走势图解、530行情、工商银行日线、茅台周线、上证指数分析
```

---

## 常见异常处理

| 情况 | 处理方式 |
|------|---------|
| BATCH 状态为"异常" | Phase 2 跳过，Phase 3 在对应章节标注来源缺失 |
| merge 中途中断 | 重跑本轮 merge agent（≤5个BATCH），幂等机制保证不重复 |
| review agent 3a 输出为空 | 检查 draft 源文件是否有内容，重跑该章 3a |
| review agent 3b 某 final 文件为空 | 重跑该章 3b |
| 会话意外中断 | 执行本文件顶部的"会话开始前必须执行"步骤，再继续 |
