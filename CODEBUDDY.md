本仓库108中是缠中说禅当年的博客原文,因为比较零散,organized-v1是我们通过内容整理后的第一个版本

## graphify

This project has two graphify knowledge graphs:

- **graphify-out/** — organized-v1 的体系化图谱（主图谱，日常查询用）
- **graphify-out-108/** — 108 原始博客的碎片化图谱（回溯原文时用）

Rules:
- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- For questions about original blog content or lesson-level references, check graphify-out-108/GRAPH_REPORT.md
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- After modifying files in organized-v1/, run `/graphify organized-v1 --update` to keep the main graph current

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
