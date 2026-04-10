# AI Coding System 对 superpowers skills 的参考映射

## 文档信息
- 状态: V1.0
- 日期: 2026-04-09
- 目的: 说明 `ai-coding-system` 方案中哪些方法论参考了 `../superpowers/skills`，以及做了哪些本地化改造
- 适用范围: 解释方案来源、对外讲解、后续方法演进对照

---

## 1. 结论

1. `ai-coding-system` 不是对 `superpowers/skills` 的照搬。
2. 参考的是其**流程纪律、Skill 协议、验证机制、任务切片思路**，不是具体业务方案。
3. 当前方案是在 superpowers 方法层基础上，扩展出面向企业迭代开发的：
   - 状态机流程治理
   - 跨状态不明确项/风险寄存
   - 主设计 + 迭代设计双层文档
   - 面向 AI coding 的知识基座与检索体系

---

## 2. 参考来源总览

| superpowers skill | 借鉴点 | 在 ai-coding-system 中的映射 |
|-------------------|--------|------------------------------|
| `verification-before-completion` | 完成前必须验证，证据先于结论 | Gate 机制、阶段出口校验、AI 先校验 + 人放行 |
| `writing-skills` | Skill frontmatter、`Use when...` 触发描述、结构化协议 | Gate Check Skill / 知识包 Skill / 主设计反向生成 Skill 的设计方式 |
| `brainstorming` | 先设计、后实现；未完成设计不进入编码 | Requirement/Design 分层与 Gate 限制 |
| `writing-plans` | 可执行计划、切片、输入足够细 | Design 到可编码级、Implementation 不再临场定义业务口径 |
| `executing-plans` | 按计划执行、阻塞即停 | Design 通过后进入 Implementation，失败则回退 |
| `subagent-driven-development` | 任务切片、逐步 review、独立可验证 | `implementation.slices.md`、后续多 Skill 串联方向 |
| `dispatching-parallel-agents` | 独立问题独立处理、并行化 | 知识生成/链路分析/后续子任务拆解思路 |

---

## 3. 重点参考一：verification-before-completion → Gate 机制

### 3.1 superpowers 原始思想
来源：`../superpowers/skills/verification-before-completion/SKILL.md`

核心铁律：

```text
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

本质：
- 不能在没有最新验证证据时声称完成
- 不能靠“应该可以”“看起来没问题”推进
- 结论之前必须先跑验证，再读输出，再下判断

### 3.2 在 ai-coding-system 中的本地化改造

从“代码完成前验证”升级为“阶段完成前验证”：

| superpowers 语境 | ai-coding-system 语境 |
|------------------|-----------------------|
| 提交前 / PR 前验证 | Requirement 结束前 Gate |
| 代码修完再验证 | Design 结束前 Gate |
| 测试/构建作为证据 | 文档、寄存、代码回查、风险关闭状态作为证据 |

### 3.3 产物化结果
1. Requirement Gate
2. Design Gate
3. AI 生成结构化 Gate 报告
4. 人工最终放行

结论：这是当前方案中最明显、也最成功的一次借鉴。

---

## 4. 重点参考二：writing-skills → Skill 协议设计

### 4.1 superpowers 原始思想
来源：`../superpowers/skills/writing-skills/SKILL.md`

关键借鉴点：
1. Skill 用 YAML frontmatter 描述
2. `description` 强调“什么时候触发（Use when...）”
3. Skill 正文强调结构化：Overview / Process / Common Mistakes / Red Flags
4. Skill 要可搜索、可发现、可稳定调用

### 4.2 在 ai-coding-system 中的映射

当前讨论稿中以下 Skill 设计方式都受其影响：
1. `requirement-gate-check`
2. 知识包生成 Skill
3. 历史主设计反向生成 Skill
4. `kb search / trace / context / drift` 命令契约设计

### 4.3 本地化改造

superpowers 更偏“技能文档规范”；当前方案把它继续往前推了一层：
- 不只定义 Skill 说明文档
- 还定义了 Skill 的**输入/输出契约**
- 并把 Skill 接入到整个 Requirement / Design / Knowledge Base 体系

结论：你借到的不是格式，而是“让 AI 能稳定发现、调用、执行流程能力”的协议化思路。

---

## 5. 重点参考三：brainstorming + writing-plans + executing-plans → 先设计后实现

### 5.1 superpowers 原始思想
来源：
- `../superpowers/skills/brainstorming/SKILL.md`
- `../superpowers/skills/writing-plans/SKILL.md`
- `../superpowers/skills/executing-plans/SKILL.md`

共通思想：
1. 创造性工作先澄清和设计
2. 设计批准前不能进入实现
3. 计划必须足够细，执行阶段不要靠临场猜测
4. 执行时严格按计划走，遇阻塞就停下来回退

### 5.2 在 ai-coding-system 中的映射

| superpowers | ai-coding-system |
|-------------|------------------|
| brainstorming | Requirement 阶段的上下文检索、澄清、范围定义 |
| writing-plans | Design 阶段输出可编码级方案 |
| executing-plans | Implementation 只负责落地，不再定义业务口径 |

### 5.3 本地化改造

1. 引入 `Requirement -> Design -> Implementation` 阶段分层
2. 把“设计批准”进一步结构化为 Gate
3. 增加 `TODO-DES-*`、契约矩阵、SQL 粒度策略、兼容与回滚要求

结论：superpowers 给的是方法纪律；当前方案把它升级成了企业需求迭代流。

---

## 6. 重点参考四：subagent-driven-development + dispatching-parallel-agents → 切片与并行思维

### 6.1 superpowers 原始思想
来源：
- `../superpowers/skills/subagent-driven-development/SKILL.md`
- `../superpowers/skills/dispatching-parallel-agents/SKILL.md`

关键点：
1. 任务要切成独立块
2. 每块都可单独验证
3. review 要分阶段进行
4. 独立问题可以并行处理

### 6.2 在 ai-coding-system 中的映射

1. `implementation.slices.md`
2. 按 controller / module / table 的知识生成单元
3. `kb trace` 这类链路追踪能力
4. 后续 Gate Skill、知识包 Skill、主设计 Skill 串联的自动化方向

### 6.3 当前还未完全落地的部分

这一块在你的方案里还是“方向性借鉴”，还没有像 Gate 那样固化到完整执行闭环。后续若继续做自动化，这部分可以演进为：
- 多阶段 AI reviewer
- 多 Skill 串联执行
- 按功能切片并行建基座

---

## 7. 哪些是借鉴，哪些已经是你自己的体系

### 7.1 借鉴来的“方法层”
1. 完成前必须验证
2. 先设计后实现
3. 文档要结构化、可检索、可执行
4. 任务应切片、可独立验证

### 7.2 你自己扩展出来的“治理层”
1. `Requirement -> Design -> Implementation -> Verification -> Knowledge Sync` 状态机
2. `uncertainty/register.md` 跨状态寄存
3. `risk/register.md` 跨状态寄存
4. Human 决策 + AI 校验的双主体放行机制

### 7.3 你自己扩展出来的“文档层”
1. 主设计 `design/design.md`
2. 迭代设计 `design/iterations/.../design.md`
3. Requirement 迭代文档 + 索引
4. 按功能聚合目录结构

### 7.4 你自己扩展出来的“知识层”
1. `knowledge-base/` 多层文档体系
2. `relation-*.md`
3. `index.md + index.json` 双入口
4. `kb search / trace / context / drift` 命令面

结论：现在这套东西已经不是“superpowers 的副本”，而是基于其方法论进一步演化出的本地体系。

---

## 8. 为什么需要单独保留这份参考文档

原因有三：

1. **对外讲解时能说清来源**
   - 哪些是参考的
   - 哪些是自己扩展的

2. **避免讨论稿越写越混**
   - 讨论稿保留增量思考
   - 参考关系单独维护
   - 总纲方案只保留结论和入口

3. **方便后续演进**
   - 如果以后继续参考 superpowers 新 skill，可以直接追加到这份文档
   - 如果本地方案偏离原始思路，也能回头对照

---

## 9. 建议使用方式

1. `ai-coding-system-discussion.md`
   - 保留讨论历史、修订过程、阶段性判断
2. `ai-coding-system-scheme.md`
   - 作为稳定总纲，只保留结论和入口
3. `ai-coding-system-superpowers-reference.md`
   - 专门解释方法借鉴来源与本地化改造

这样三份文档分工明确：
- 讨论稿 = 思考过程
- 总纲 = 稳定方案
- 参考映射 = 方法来源说明

---

## 10. 关联文档

- [ai-coding-system-discussion.md](ai-coding-system-discussion.md)
- [ai-coding-system-scheme.md](ai-coding-system-scheme.md)
- [knowledge-base-generation-spec-v1.1.md](knowledge-base-generation-spec-v1.1.md)
- [requirement-phase-spec.md](requirement-phase-spec.md)
- [design-phase-spec.md](design-phase-spec.md)
