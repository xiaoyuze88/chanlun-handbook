# Graph Report - organized-v2  (2026-04-18)

## Corpus Check
- Corpus is ~28,457 words - fits in a single context window. You may not need a graph.

## Summary
- 150 nodes · 260 edges · 16 communities detected
- Extraction: 97% EXTRACTED · 3% INFERRED · 0% AMBIGUOUS · INFERRED: 9 edges (avg confidence: 0.74)
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- [[_COMMUNITY_缠论体系总览|缠论体系总览]]
- [[_COMMUNITY_背驰与区间套精确定位|背驰与区间套精确定位]]
- [[_COMMUNITY_牛熊市与大资金策略|牛熊市与大资金策略]]
- [[_COMMUNITY_中枢价格体系|中枢价格体系]]
- [[_COMMUNITY_笔的构造与定理|笔的构造与定理]]
- [[_COMMUNITY_级别自组织与递归|级别自组织与递归]]
- [[_COMMUNITY_三类买卖点定律|三类买卖点定律]]
- [[_COMMUNITY_附录与完备性索引|附录与完备性索引]]
- [[_COMMUNITY_操作系统与资金管理|操作系统与资金管理]]
- [[_COMMUNITY_市场本质与经济人原则|市场本质与经济人原则]]
- [[_COMMUNITY_贪嗔痴与人性陷阱|贪嗔痴与人性陷阱]]
- [[_COMMUNITY_当下性哲学|当下性哲学]]
- [[_COMMUNITY_均线吻与趋势力度|均线吻与趋势力度]]
- [[_COMMUNITY_分型识别规则|分型识别规则]]
- [[_COMMUNITY_区间套定位法|区间套定位法]]
- [[_COMMUNITY_买卖不对等性|买卖不对等性]]

## God Nodes (most connected - your core abstractions)
1. `附录B：定理索引` - 16 edges
2. `缠论体系全景（总论）` - 15 edges
3. `附录A：定义速查` - 15 edges
4. `附录C：操作规则总表` - 15 edges
5. `附录E：原文索引` - 15 edges
6. `背驰（均线面积法）` - 11 edges
7. `P1-02 当下性哲学` - 10 edges
8. `MACD辅助判断背驰` - 9 edges
9. `牛熊市判断与大资金策略` - 9 edges
10. `反庄操作` - 9 edges

## Surprising Connections (you probably didn't know these)
- `缠中说禅趋势力度（均线面积）` --semantically_similar_to--> `MACD辅助判断背驰`  [INFERRED] [semantically similar]
  organized-v2/P5-背驰理论/P5-16-背驰.md → organized-v2/P5-背驰理论/P5-18-MACD辅助判断背驰.md
- `完全分类操作原则` --semantically_similar_to--> `风险的系统化处理`  [INFERRED] [semantically similar]
  organized-v2/P1-认知基础与交易哲学/P1-02-当下性哲学.md → organized-v2/P1-认知基础与交易哲学/P1-03-贪嗔痴与人性陷阱.md
- `分型（顶/底分型）` --conceptually_related_to--> `递归函数f1（起始程序）`  [EXTRACTED]
  organized-v2/P2-形态学基础/P2-04-分型.md → organized-v2/P2-形态学基础/P2-07-形态学总论与递归函数.md
- `自同构性结构` --semantically_similar_to--> `级别与时间无关定理`  [INFERRED] [semantically similar]
  organized-v2/P2-形态学基础/P2-07-形态学总论与递归函数.md → organized-v2/P4-走势与级别/P4-14-级别自组织与自相似性.md
- `走势多样性分解` --semantically_similar_to--> `走势连接结合律`  [INFERRED] [semantically similar]
  organized-v2/P8-高阶应用/P8-30-实战案例精解.md → organized-v2/P4-走势与级别/P4-13-走势分解定理.md

## Hyperedges (group relationships)
- **形态学构件的层级递进依赖关系（分型→笔→线段→中枢）** — p2_04_分型, p2_05_笔, p2_06_线段, p2_07_递归函数f1 [EXTRACTED 1.00]
- **背驰信号完整判断链（走势结构→趋势背驰→MACD辅助→级别与转折）** — p5_16_背驰, p5_17_趋势背驰, p5_18_MACD辅助判断, p5_19_背驰级别大于等于转折级别 [EXTRACTED 0.95]
- **P8高阶应用综合（牛熊市判断、反庄操作、实战案例协同）** — p8_28_牛熊市判断, p8_29_反庄操作, p8_30_实战案例, p8_30_当下图解分析法 [INFERRED 0.85]
- **三类买卖点完备体系（第一/二/三类联合构成完备操作体系）** — concept_first_buy, concept_second_buy, concept_third_buy, concept_completeness_theorem [EXTRACTED 0.95]
- **操作系统三位一体（操作级别+区间套定位+资金管理）** — concept_operation_level, concept_interval_nesting_method, concept_zero_cost [INFERRED 0.82]
- **中枢理论核心（ZG/ZD+Z走势段+三种命运）** — concept_pivot_zdg, concept_z_segment, concept_pivot_three_fates, concept_pivot_six_prices [EXTRACTED 0.90]

## Communities

### Community 0 - "缠论体系总览"
Cohesion: 0.1
Nodes (31): 缠论查询路由表, 缠论核心公理集, 缠论体系全景（总论）, 附录B：定理索引, 附录D：状态条件速查, 分型（顶/底分型）, 动力学（Dynamics）, 形态学（Morphology） (+23 more)

### Community 1 - "背驰与区间套精确定位"
Cohesion: 0.17
Nodes (13): 走势连接结合律, 走势多义性, 精确大转折点寻找程序定理（区间套）, 缠中说禅背驰-转折定理（三种后果）, 三重区间套嵌套背驰定理, 同级背驰必然回最后中枢, 缠中说禅小背驰-大转折定理（仅必要条件）, 背驰级别≥转折级别（绝对预测定理） (+5 more)

### Community 2 - "牛熊市与大资金策略"
Cohesion: 0.19
Nodes (13): 大资金操作级别原则（高出散户两重）, 政策面最大影响级别定理（周线级别）, 牛市一夜情（暴跌≠顶）, 牛市三阶段论, 牛熊市判断与大资金策略, 积累筹码降成本策略, 主力资金食物链, 买卖点不可逃避定理 (+5 more)

### Community 3 - "中枢价格体系"
Cohesion: 0.21
Nodes (13): 震荡中轴Z与Zn序列, 缠中说禅盘整, 中枢延伸, 中枢四指标GG/G/D/DD, 中枢六价体系ZG/ZD/GG/G/D/DD, 中枢三种命运（延伸/扩展/新生）, 缠中说禅趋势, 走势中枢[ZD,ZG] (+5 more)

### Community 4 - "笔的构造与定理"
Cohesion: 0.18
Nodes (12): 笔的四状态数组（D,E）, 笔（向上笔/向下笔）, 笔三条充要条件（C1/C2/C3）, 笔划分唯一性定理, 缠中说禅笔定理, 特征序列, 线段, 线段划分唯一性定理 (+4 more)

### Community 5 - "级别自组织与递归"
Cohesion: 0.24
Nodes (10): 自同构性结构, 递归函数f1（起始程序）, 递归函数f2（中枢过程规则）, 图表级别（K线时间周期）vs走势级别（递归层次）, 级别与时间无关定理, 走势级别体系（Xn/Yn/Wn/Sn/Rn/Zn/Mn/Jn/Nn）, a+A结构（同级别分解操作单元）, 同级别分解 (+2 more)

### Community 6 - "三类买卖点定律"
Cohesion: 0.36
Nodes (10): 三类买卖点完备性定理, 第一类买点, 第一类卖点, 缠中说禅买卖点定律一, 第二类买点, 第二类卖点, 缠中说禅趋势转折定律, P6-20 第一类买卖点 (+2 more)

### Community 7 - "附录与完备性索引"
Cohesion: 0.31
Nodes (10): 附录A：定义速查, 附录C：操作规则总表, a+A+b+B+c当下分析框架, 升跌完备性定理, 第三类买点, 三买100%获利性质, 第三类卖点, P3-11 中枢中心定理 (+2 more)

### Community 8 - "操作系统与资金管理"
Cohesion: 0.31
Nodes (9): 持币操作, 持股操作, 操作级别, 三个独立程序乘法原则, 走势止损条件, 零成本持股状态, P7-26 操作级别选择, P7-27 资金管理与止损 (+1 more)

### Community 9 - "市场本质与经济人原则"
Cohesion: 0.33
Nodes (6): 附录E：原文索引, 经济人原则, 股票废纸性, 缠论有效性前提, P1-01 市场本质与经济人原则, 理性：经济人原则的设计意图

### Community 10 - "贪嗔痴与人性陷阱"
Cohesion: 0.5
Nodes (5): 赌徒心理, 喜好陷阱, 风险的系统化处理, 技术指标评价系统, P1-03 贪嗔痴与人性陷阱

### Community 11 - "当下性哲学"
Cohesion: 0.4
Nodes (5): 完全分类操作原则, 价格唯一性公设, 分段函数操作模型, P1-02 当下性哲学, 理性：当下性哲学——为何拒绝预测

### Community 12 - "均线吻与趋势力度"
Cohesion: 0.6
Nodes (5): 体位（女上位/男上位）, 均线吻三类型（飞吻/唇吻/湿吻）, 缠中说禅板块强弱指标, 缠中说禅趋势力度（均线面积法）, P7-24 均线吻理论

### Community 13 - "分型识别规则"
Cohesion: 0.5
Nodes (4): 分型客观唯一性, 包含关系处理, 底分型, 顶分型

### Community 14 - "区间套定位法"
Cohesion: 1.0
Nodes (3): 背驰段（区间套）, 区间套定位法, P7-25 区间套精确定位

### Community 15 - "买卖不对等性"
Cohesion: 1.0
Nodes (1): 买卖不对等性

## Knowledge Gaps
- **51 isolated node(s):** `缠论核心公理集`, `底分型`, `分型客观唯一性`, `笔三条充要条件（C1/C2/C3）`, `笔划分唯一性定理` (+46 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **Thin community `买卖不对等性`** (1 nodes): `买卖不对等性`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `附录B：定理索引` connect `缠论体系总览` to `背驰与区间套精确定位`, `牛熊市与大资金策略`, `笔的构造与定理`, `级别自组织与递归`?**
  _High betweenness centrality (0.102) - this node is a cross-community bridge._
- **Why does `缠论体系全景（总论）` connect `缠论体系总览` to `背驰与区间套精确定位`, `牛熊市与大资金策略`, `笔的构造与定理`, `级别自组织与递归`?**
  _High betweenness centrality (0.096) - this node is a cross-community bridge._
- **What connects `缠论核心公理集`, `底分型`, `分型客观唯一性` to the rest of the system?**
  _51 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `缠论体系总览` be split into smaller, more focused modules?**
  _Cohesion score 0.1 - nodes in this community are weakly interconnected._