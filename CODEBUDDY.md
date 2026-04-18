本仓库108中是缠中说禅当年的博客原文,因为比较零散,organized-v1是我们通过内容整理后的第一个版本

## graphify

This project has two graphify knowledge graphs:

- **graphify-out/** — organized-v2 的体系化图谱（主图谱，日常查询用）
- **graphify-out-108/** — 108 原始博客的碎片化图谱（回溯原文时用）

Rules:
- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- For questions about original blog content or lesson-level references, check graphify-out-108/GRAPH_REPORT.md
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- After modifying files in organized-v2/, run `/graphify organized-v2 --update` to keep the main graph current

## 缠论知识库查询规则

知识库位置：`organized-v2/`（36个章节文件，约2.8万字）
图谱数据：`graphify-out/graph.json`（供 graphify query 命令使用，不要直接 Read）

**当用户提问任何缠论相关问题时，执行以下流程：**

1. **用 graphify query 定位相关文件**：从问题中提取 2-4 个核心中文关键词，替换下方 `terms`，运行命令，获得 BFS 遍历后的相关文件列表：

```bash
cd /Users/xyz/Sites/private/chzhshch-108-plus && python3 -c "
import sys, json
from networkx.readwrite import json_graph
from pathlib import Path

data = json.loads(Path('graphify-out/graph.json').read_text())
G = json_graph.node_link_graph(data, edges='links')

# 从问题提取关键词（中文不能用 split()，由模型手动列出）
terms = ['关键词1', '关键词2']  # ← 替换为实际关键词

# 找匹配的起始节点（子串匹配，无长度限制）
scored = []
for nid, ndata in G.nodes(data=True):
    label = ndata.get('label', '').lower()
    score = sum(1 for t in terms if t in label)
    if score > 0:
        scored.append((score, nid))
scored.sort(reverse=True)
start_nodes = [nid for _, nid in scored[:3]]

if not start_nodes:
    print('No matching nodes found for terms:', terms)
    sys.exit(0)

# BFS depth=3 扩展邻居节点
frontier = set(start_nodes)
subgraph_nodes = set(start_nodes)
for _ in range(3):
    next_frontier = set()
    for n in frontier:
        for neighbor in G.neighbors(n):
            if neighbor not in subgraph_nodes:
                next_frontier.add(neighbor)
    subgraph_nodes.update(next_frontier)
    frontier = next_frontier

# 输出去重的 source_file 列表（按相关度排序）
def relevance(nid):
    return sum(1 for t in terms if t in G.nodes[nid].get('label', '').lower())

ranked = sorted(subgraph_nodes, key=relevance, reverse=True)
files = list(dict.fromkeys(
    G.nodes[n].get('source_file') for n in ranked
    if G.nodes[n].get('source_file')
))
for f in files[:8]:
    print(f)
"
```

2. **阅读原文**：用 Read 工具读取上一步返回的 `organized-v2/` 文件（通常 1-3 个即可）
   - 读每个文件时，先检查文件头的 `<!-- 前置知识：... -->` 注释
   - 若有前置知识，先读前置知识对应的文件，再读当前文件
   - 前置知识文件路径规则：同目录下同章节编号（如 P3-09 的前置 → 找 P3-08）

3. **理解后回答**：用自己的话回答，要求：
   - 先给结论，再给推导
   - 用日常语言，避免直接引用术语堆砌
   - 可以举例类比
   - 如果问题跨多个章节，先读再综合
   - 回复结尾附上"📎 引用来源"，列出本次阅读的文件路径及用到的具体段落标题（如"P3-08 § 定义一"），方便用户深入查阅

**禁止**：不要把图谱节点标签或边关系直接当答案输出。`graph.json` 不要用 Read 直接读，那是机器格式，只能通过上述 python3 命令查询。

## organized-v2 项目

正在将 organized-v1 的简单聚合内容提炼为教科书式知识体系。

**关键文件：**
- 设计规格: `docs/superpowers/specs/2026-04-17-organized-v2-textbook-design.md`
- 实施计划: `docs/superpowers/plans/2026-04-17-organized-v2-textbook.md`
- 进度表: `v2-systematic/PROGRESS.md`
- 工作目录: `v2-systematic/`（prompt、draft）
- 输出目录: `organized-v2/`

**当用户说"继续 organized-v2"或"恢复 v2 任务"时：**
1. 读取 memory 中的进度记录
2. 读取 `v2-systematic/PROGRESS.md` 获取细粒度状态
3. 僵尸清理：将"进行中"全改为"失败"
4. 从断点继续执行

## 目录结构

```
archived/108/              # 缠中说禅原始博客原文（零散）
archived/organized-v1/     # 第一版整理：通过 systematic/ 流程从 archived/108 整理而来
archived/graphify-out-108/ # archived/108 的知识图谱
archived/graphify-out-v1/  # archived/organized-v1 的知识图谱
organized-v2/              # 第二版整理：通过 v2-systematic/ 流程从 archived/organized-v1 提炼而来（当前主版本）
```

## 溯源路径

```
archived/108  →(archived/systematic/)→  archived/organized-v1  →(archived/v2-systematic)→  organized-v2
     ↓                                         ↓
archived/graphify-out-108            archived/graphify-out-v1
```

如需溯源某个知识点，可按此路径逐层回溯至原始博客。