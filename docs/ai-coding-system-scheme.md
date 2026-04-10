# Human + AI 协同 Coding 系统方案

## 文档信息
- 状态: V1.0（基于讨论稿整理）
- 整理日期: 2026-04-09
- 来源: `ai-coding-system-discussion.md` 全量内容条理化重组
- 定位: 系统方案总纲，各阶段详细规范见关联文档

---

## 目录

1. [系统定位与目标](#1-系统定位与目标)
2. [统一方法论](#2-统一方法论)
3. [流程主线](#3-流程主线)
4. [知识基座体系](#4-知识基座体系)
5. [文档管理体系](#5-文档管理体系)
6. [Gate 校验机制](#6-gate-校验机制)
7. [Skill 体系纵览](#7-skill-体系纵览)
8. [参考设计](#8-参考设计)
9. [待确认问题清单](#9-待确认问题清单)
10. [关联文档索引](#10-关联文档索引)

---

## 1. 系统定位与目标

### 1.0 方法借鉴来源（Superpowers）
1. 本方案的方法层参考了兄弟仓库 `../superpowers/skills/` 的部分流程纪律与 Skill 协议设计。
2. 借鉴重点不是业务内容，而是：
   - Gate 思维（参考 `verification-before-completion`）
   - Skill 文件结构与触发描述（参考 `writing-skills`）
   - 先设计后实现（参考 `brainstorming` / `writing-plans` / `executing-plans`）
   - 任务切片与可验证执行（参考 `subagent-driven-development` / `dispatching-parallel-agents`）
3. 本方案不是照搬 superpowers，而是在其方法层基础上，扩展出了面向企业迭代开发的状态机流程治理、知识基座、跨状态寄存与双层设计文档体系。
4. 详细映射见：`ai-coding-system-superpowers-reference.md`。

### 1.1 核心定位
1. 人负责决策和准入，AI 负责高强度执行与文档化沉淀。
2. 系统目标不是"做完功能"，而是"可持续迭代地做对功能"。
3. 所有流程设计围绕三件事：**可验证、可追溯、可持续更新**。

### 1.2 已对齐目标
- 方向: 提升迭代交付效率，并推动开发流程规范化。
- 长期目标: 让 AI 承担高比例 coding 工作（目标值后续细化）。
- 讨论方式: 先定流程与架构，再细化指标和阶段目标。

---

## 2. 统一方法论

### 2.1 状态机工作流

采用统一状态机推进工作流：

```
Requirement → Design → Implementation → Verification → Knowledge Sync
```

每个状态必须定义：
- **输入契约** — 明确输入
- **产物契约** — 明确输出
- **门禁条件** — 通过条件（未通过不得进入下一状态）

### 2.2 四个流程原则

| 原则 | 说明 |
|------|------|
| 阶段化（Stage-based） | 先定义问题，再定义方案，再进入执行，避免边做边改方向 |
| 门禁化（Gate-driven） | 需求、设计、代码、知识四类门禁前置，禁止"先合并后补材料" |
| 证据化（Evidence-first） | 结论必须带代码证据、测试证据、提交证据，避免"经验性正确" |
| 闭环化（Closed-loop） | 交付后必须回灌知识基座，确保下一次 AI 检索可复用、可置信 |

### 2.3 跨状态不明确项管理（Uncertainty Register）

#### 2.3.1 机制目的
- 不明确项必须显式化，禁止"隐式假设推进"。
- Human 与 AI 基于同一不明确项清单进行决策与准入。

#### 2.3.2 强制登记规则
在任一状态发现不明确点，必须登记到 Uncertainty Register。每条至少包含：

| 字段 | 说明 |
|------|------|
| `id` | 唯一标识 |
| `state` | 所处阶段 |
| `question` | 不明确问题 |
| `impact` | 影响范围 |
| `default_assumption` | 默认假设 |
| `owner` | 负责人 |
| `due_at` | 截止时间 |
| `status` | `open / answered / accepted-risk / rejected` |

#### 2.3.3 通过与放行规则
- `P0/P1` 级未关闭 → 不得通过当前状态 Gate。
- `P2/P3` 级可带风险放行，但必须满足：`status=accepted-risk` + 明确 `owner` + 明确 `due_at`。
- Human 可进行"带风险放行"，但必须记录决策人、时间与理由。

---

## 3. 流程主线

### 3.1 历史知识建基流程

#### 3.1.1 总体原则
1. 先建立"可用知识基座"，再追求"完整知识基座"。
2. 先框架后细节，先主干后分支，避免一次性做大全导致失败。
3. 每份文档都要可追溯到代码证据（路径、行号、提交）。
4. 文档必须可持续更新，不做一次性静态产物。

#### 3.1.2 分层生成顺序

| 层级 | 产物 | 目标 |
|------|------|------|
| 框架层（系统/仓库级） | `architecture-overview.md` | 识别系统边界、公共模块、依赖关系、主调用链 |
| 功能模块层（业务域/子系统级） | `module-<name>.md` | 明确模块职责、输入输出、对外接口、关键流程 |
| Controller 层（接口入口级） | `controller-<name>.md` | 明确请求入口、参数校验、调用链路、错误处理、鉴权逻辑 |
| 数据表层（数据模型级） | `table-<name>.md` | 说明表结构用途、被哪些功能点使用、读写路径、一致性约束 |

**强制要求**：Controller 层知识文档必须包含字段级接口契约（请求/响应 JSON 结构 + 字段说明），否则不具备 Design 阶段输入资格。

#### 3.1.3 推荐执行顺序

| 轮次 | 内容 | 说明 |
|------|------|------|
| 第 1 轮 | 框架层 | 产出系统框架总览，组织架构评审，确认模块边界和公共能力定义 |
| 第 2 轮 | 功能模块层 | 优先高频迭代模块和核心业务模块，输出模块清单和优先级 |
| 第 3 轮 | Controller 层 | 优先外部调用频繁、变更频繁的接口，补齐接口到服务的真实调用链 |
| 第 4 轮 | 数据表层 | 优先核心交易表、主数据表、审计表，建立"表→功能点→接口→代码路径"映射 |

#### 3.1.4 每层文档最小必填字段（MVP）
1. 基本信息: 名称、owner、最后更新时间、状态（draft/verified）
2. 职责边界: 做什么、不做什么、上下游依赖
3. 关键流程: 正常流 + 异常流
4. 代码证据: 文件路径、关键函数、行号、最近提交
5. 风险与约束: 性能、事务、一致性、权限、安全
6. 接口契约（Controller 层强制）: 每个接口的请求/响应字段级结构（JSON 样例 + 字段说明表）

#### 3.1.5 审校与准入机制
1. AI 初稿生成后，不直接入库，必须经过人审。
2. 审校角色：框架层→架构师/Tech Lead、模块层→模块 owner、Controller 层→接口 owner、表层→数据库 owner + 业务 owner。
3. 审校完成后标记 `verified`；未 `verified` 的文档可被检索但不作为高置信输入。

#### 3.1.6 持续更新机制（防过期）
1. PR 触发"文档漂移检测"。
2. 命中以下变化时强制补文档：controller 路由/参数/返回结构变化、service 核心流程变化、数据库 schema 变化、公共模块契约变化。
3. CI 门禁：代码变化命中规则但无文档更新时阻断合并，或至少创建文档更新任务并设置时限。

#### 3.1.7 三段式生成机制
1. 一次性 AI 反向生成（从 0 到 1）
2. 关键人审校（保证可信度）
3. PR 驱动持续同步（防止再次过期）

### 3.2 新需求处理流程

完整流程五步：需求输入标准化 → 上下文检索与冲突分析 → 方案生成与决策 → 任务拆解与执行 → 交付与知识回灌。

各阶段详细规范见独立文档：
- Requirement 阶段 → [requirement-phase-spec.md](requirement-phase-spec.md)
- Design 阶段 → [design-phase-spec.md](design-phase-spec.md)

#### 3.2.1 Requirement（需求态）核心要点

1. **先检索后动手**：必须先做上下文检索与澄清，不允许直接进入方案设计或编码。
2. **两级检索 + 双源校验**：
   - 一级检索: 优先读取历史 `design.md` / `solution.design.md`
   - 二级检索: 自动读取关联 `module-*.md` / `controller-*.md` / `table-*.md`
   - 双源校验: 文档结论必须回查代码证据（file + line + commit）与近期 PR/测试基线
3. **双路径规则**：
   - Path A（有历史设计）: 完成 `design → module/controller/table → 代码证据回查` 全链路
   - Path B（无历史设计）: 先生成 `design-baseline.md`（最小基线设计）
4. **Gate 通过条件**: 验收标准可测试 + 范围/非范围明确 + 依赖与风险显式化 + 证据来源明确

#### 3.2.2 Design（设计态）核心要点

1. **定位**："决策态"而非"写文档态"，目标是把 Requirement 转成可实施、可验证、可回滚的方案。
2. **核心步骤**: 影响面建模 → 方案分叉（至少2个）→ 方案收敛（含不选理由）→ 实施切片 → 风险与回滚
3. **产物**: `impact.analysis.md` + `solution.design.md` + `implementation.slices.md` + `decision.log.md`
4. **不明确项处理（强制）**: 在本阶段结束前，所有 `status=open` 项必须完成状态转移，并写入风险记录（含 `risk_id` / `severity` / `scenario` / `mitigation` / `rollback_plan` / `owner` 等字段）。

#### 3.2.3 Design 与 Implementation 分界

| Design 必须明确 | Implementation 负责 |
|----------------|---------------------|
| 接口入参/出参与可空语义（契约级） | 类/方法/SQL 的具体编码实现 |
| 关键业务规则与分支（正常流/异常流） | 性能优化与重构细节 |
| 查询过滤与聚合口径（至少到 SQL 粒度策略） | 自动化测试、联调修正、发布执行 |
| 场景分流规则 | |
| 可对账一致性约束 | |

**人工 TODO 机制**：需要人工补充的信息必须登记 `TODO-DES-*`，每条绑定 `owner`，进入 Implementation 前必须完成或明确放行决策。

### 3.3 质量与治理闸口（流程内置控制点）

| 闸口 | 校验内容 |
|------|----------|
| 需求闸口 | 结构化需求是否完整 |
| 设计闸口 | 是否完成影响分析与方案评审 |
| 代码闸口 | 测试、静态检查、安全扫描是否通过 |
| 知识闸口 | 设计文档是否已同步更新 |
| 不明确项闸口 | `P0/P1` 未关闭不得过 Gate；`P2/P3` 仅允许 `accepted-risk + owner + due_at` 后放行 |

---

## 4. 知识基座体系

### 4.1 基座规范

详细规范见 → [knowledge-base-generation-spec-v1.1.md](knowledge-base-generation-spec-v1.1.md)

核心要点：
- 以"单个 Controller"作为最小落地单元，一次产出完整知识包（8类交付物）。
- ���有文档必须包含 YAML 头（id/layer/module/owner/status/confidence 等）。
- 证据强制：每条关键结论附文件路径 + 行号 + 提交哈希。
- Table 文档 DDL 必填（未补齐保持 `draft`）。
- 人工审核标记：`verified_by` / `verified_at` / `review_round` / `review_notes`。

### 4.2 目录结构

```text
knowledge-base/
  architecture/       # 框架层
  controllers/        # Controller 文档
  services/           # Service 文档
  modules/            # 模块文档
  features/           # 功能文档
  relations/          # 关系链路文档
  tables/             # 数据表文档
  indexes/            # 检索索引（.md 人读 + .json 机读）
  changelog/          # 变更日志
```

### 4.3 命名与冲突规则

| 类型 | 默认命名 | 冲突规则 |
|------|----------|----------|
| Controller | `controller-<name>.md` | — |
| Service | `service-<service_name>.md` | 跨模块同名 → `service-<module>-<service_name>.md` |
| Table | `table-<table_name>.md` | 跨 datasource/schema → `table-<datasource>-<schema>-<table_name>.md` |
| Module | `module-<module_name>.md` | — |
| Feature | `feature-<feature_name>.md` | — |
| Relation | `relation-<domain_or_module>.md` | — |

强制唯一标识字段（YAML）：
- Service: `service_key = <module>.<service_name>`
- Table: `table_key = <datasource>.<schema>.<table_name>`

### 4.4 文档正文最小结构

所有知识文档统一包含：
1. 职责与边界
2. 关键流程（正常流/异常流）
3. 上游下游依赖
4. 关键代码入口
5. 数据对象/表映射
6. 风险与约束
7. 证据清单

### 4.5 关系模型规范

用于链路检索的标准关系类型：

| 关系 | 说明 |
|------|------|
| `FEATURE_USES_CONTROLLER` | 功能使用哪些 Controller |
| `CONTROLLER_CALLS_SERVICE` | Controller 调用哪些 Service |
| `SERVICE_READS_TABLE` | Service 读哪些表 |
| `SERVICE_WRITES_TABLE` | Service 写哪些表 |
| `MODULE_OWNS_TABLE` | 模块拥有哪些表 |
| `MODULE_DEPENDS_ON_MODULE` | 模块间依赖 |

关系字段：`from_id`、`to_id`、`relation_type`、`evidence`、`updated_at`

### 4.6 生命周期与门禁
1. 新生成文档默认 `draft`。
2. Owner 审核后改为 `verified`。
3. PR 命中漂移规则时必须更新对应文档。
4. 过期文档标记 `stale` 并在检索中降权。

### 4.7 文档粒度要求（人工审核导向）

`feature/module/service` 文���应从"简要说明"升级为"可审核说明"，至少包含：
- 方法级/流程级说明
- 关键分支规则（如渠道分流、状态机）
- 异步语义说明（接口成功 ≠ ���务完成）
- 风险点与审核清单

结论：代码注释不足不应成为文档解释不足的理由；应通过"代码证据 + 业务归纳"补齐。

---

## 5. 文档管理体系

### 5.1 目录结构：按功能聚合

从"按状态分目录"升级为"按功能分目录，状态作为子目录"。目标：让跨状态资产（不明确项/风险）与 Requirement/Design 在同一功能根路径下闭环管理。

```text
docs/features/<功能名>/
  requirement/
    index.md                                    # 迭代索引
    iterations/<date-feature>/
      requirement.spec.md                       # 迭代需求文档
  design/
    design.md                                   # 功能唯一主设计
    iterations/<date-feature>/
      design.md                                 # 迭代设计文档
  uncertainty/
    register.md                                 # 跨状态不明确项寄存
  risk/
    register.md                                 # 跨状态风险寄存
```

### 5.2 Requirement / Design 双层文档规则

| 文档类型 | 规则 |
|----------|------|
| Requirement | 不设主文档，仅保留迭代文档；评审通过后更新 `requirement/index.md` |
| Design | 主文档 `design/design.md`（功能唯一）+ 迭代文档；迭代评审通过后必须回写主设计 |

未完成回写，不得视为该功能状态完成。

### 5.3 跨状态不明确项与风险寄存

- 功能根目录下 `uncertainty/register.md` 和 `risk/register.md` 为 Requirement / Design / Implementation / Verification 共享。
- 避免状态切换时信息丢失或重复登记。
- 闸口约束：`P0/P1` 未关闭不得过 Gate；`P2/P3` 仅允��� `accepted-risk + owner + due_at` 放行。

### 5.4 路径引用规范（强制）

1. 文档内路径一律使用**项目相对地址**，禁止使用本机绝对地址（如 `D:\...`）。
2. 推荐写法：`docs/features/...`、`3rd.bi.platform.management/...`
3. 跨文档引用必须可在仓库根目录下直接定位。
4. 发现绝对路径时，视为规范不通过项，必须修正后再进入下一状态。

---

## 6. Gate 校验机制

### 6.1 Gate 的本质定位

1. Gate 是**阶段出口的强制检查点**，不是可选操作。
2. 核心作用：禁止带着未解决的问题进入下一阶段，把风险暴露在前而不是后。
3. 设计原则：`NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE`（参考 superpowers `verification-before-completion`）。

### 6.2 Gate 触发时机（固定）

| 场景 | 触发 Gate |
|------|-----------|
| Requirement 写完，准备进入 Design | Requirement Gate |
| Design 写完，准备进入 Implementation | Design Gate |
| P0/P1 不明确项发现时 | 当前阶段 Gate 自动失败 |
| 需求范围不清晰 | Gate 失败，回退重新澄清 |

### 6.3 Gate 校验分工

**AI 职责**：
- 逐条核对 Gate 条件
- 读取 `uncertainty/register.md` 核查不明确项状态
- 输出结构化 Gate 报告（PASS/FAIL + 逐条说明）
- FAIL 时列出必须解决的项，不得自行推进

**人的职责**：
- 看 AI 的 Gate 报告，做最终放行决策
- 显式确���"进入下一阶段"或"回退补充"

**不允许 AI 自动通过的原因**：Gate 失败的核心风险是 P0/P1 不明确项（通常是业务口径或技术边界问题），AI 无法独立判断；AI 自写自审容易"自我说服"。

### 6.4 Gate 报告输出规范（强制）

AI 完成阶段工作后，必须在对应文档末尾输出 Gate 报告块：

```markdown
## Gate 检查结果（AI 生成，需人工放行）

| 条件 | 状态 | 说明 |
|------|------|------|
| 目标/范围/非范围明确 | ✅/❌ | ... |
| 验收标准可测试 | ✅/❌ | ... |
| 契约变更确认 | ✅/❌ | 有/无契约变更 |
| P0/P1 不明确项关闭 | ✅/❌ | 若 ❌，列出未关闭项 ID |
| P2/P3 accepted-risk | ✅/N/A | 若有，确认字段完整 |

**AI 结论**: Gate PASS/FAIL — 原因说明

**等待人工放行**: [ ] 确认进入下一阶段  [ ] 回退补充
```

未输出 Gate 报告前，不得声称当前阶段完成。

---

## 7. Skill 体系纵览

> 以下 Skill 均为设计阶段，本轮不实现。各 Skill 的完整输入/输出/流程/契约定义见 → [skill-designs-overview.md](skill-designs-overview.md)

| Skill | 定位 | 输入 | 输出 | 状态 |
|-------|------|------|------|------|
| 知识包生成 | 输入 controller 路径，自动生成完整知识包（8类交付物） | `controller_path` | controller/service/module/feature/relation/table/index 文档 | 待实现 |
| Gate Check | 阶段完成前强制校验，输出 PASS/FAIL 报告，人工放行 | 当前阶段文档 + `uncertainty/register.md` | Gate 报告（追加到文档末尾） | 待实现 |
| 主设计反向生成 | 从现有代码反向生成功能主设计 `design.md`，含接口契约基线 | `controller_path` + 知识基座（可选） | `design/design.md`（`draft`） | 待实现 |
| kb 检索命令 | 面向 AI coding 的知识检索体系（search/trace/context/read/drift） | 查询/路径/需求文本 | 命中文档 + 证据 + 关系链路 + 置信度 | 待定义 |

### 推荐串联顺序

```text
知识包生成 ──→ 主设计反向生成 ──→ 日常迭代进入
                                    │
                            Requirement Gate Check
                                    │
                              Design Gate Check
                                    │
                         Implementation Gate Check
```

1. 新功能首次建基座：知识包生成 → 主设计反向生成 → 人工审校
2. 日常迭代：Requirement → Gate Check → Design → Gate Check → Implementation
3. 日常检索：`kb search / trace / context` 持续使用
4. 持续治理：`kb drift` 检测文档漂移

---

## 8. 参考设计

### 8.1 方法借鉴（Superpowers skills）

| 借鉴来源 | 核心思想 | 在本方案中的映射 |
|----------|----------|------------------|
| `verification-before-completion` | 完成前必须验证，证据先于结论 | Gate 机制、阶段出口校验、AI 先校验 + 人放行 |
| `writing-skills` | Skill frontmatter、`Use when...` 触发式描述、结构化协议 | Gate Check Skill / 知识包 Skill / 主设计反向生成 Skill 的设计方式 |
| `brainstorming` | 先设计、后实现；设计批准前不进入编码 | Requirement/Design 分层与 Gate 限制 |
| `writing-plans` | 计划必须足够细，执行不要临场猜 | Design 输出可编码级方案、`implementation.slices.md` |
| `executing-plans` | 按计划执行、遇阻塞回退 | Design 通过后才能进入 Implementation，失败则回退 |
| `subagent-driven-development` | 任务切片、逐步 review、独立可验证 | 多 Skill 串联、切片化实施、后续 AI reviewer 扩展方向 |
| `dispatching-parallel-agents` | 独立问题独立处理、并行化 | 模块/接口/知识生成单元拆分与后续并行处理思路 |

说明：本方案对 superpowers 的借鉴主要发生在**方法层**；状态机流程治理、知识基座、跨状态寄存、主设计/迭代设计双层文档体系属于本地化扩展。详细映射见：`ai-coding-system-superpowers-reference.md`。

### 8.2 Claude Code 源码借鉴

| 借鉴点 | 参考文件 | 落地映射 |
|--------|----------|----------|
| 工具分层（检索与读取分离） | `GrepTool.ts` / `GlobTool.ts` / `FileReadTool.ts` | `kb search` 对应 Grep/Glob；`kb read` 对应 FileReadTool |
| 统一权限控制与规则优先级 | `filesystem.ts` | `deny/ask` 优先于 `allow`，deny 规则转 ignore pattern |
| 检索性能与稳定性 | `ripgrep.ts` / `glob.ts` | ripgrep 底层引擎，`head_limit + offset` 分页，超时重试 |
| 读取限额与防失控 | `limits.ts` / `readFileInRange.ts` | `maxSizeBytes + maxTokens` 双阈值，行级读取 |
| 上下文控制与去重缓存 | `FileReadTool.ts` / `context.ts` | `file_unchanged` stub，上下文预取 + memoize |
| 规则驱动持续更新 | — | `kb drift` 对应门禁机制 |

---

## 9. 待确认问题清单

1. 文档体系目录约定（每个模块一个 `design.md` 还是集中式）
2. 证据链接格式约定（文件路径、行号、提交哈希）
3. 哪些高风险模块需要更严格的人审和发布门禁
4. 需求到设计到代码的工单字段映射规范
5. `verified` 的时效策略（例如 30/60/90 天）
6. `design.md` 标准模板（章节、粒度、证据格式）
7. 自动生成规则（按模块/按服务/按目录）
8. 文档漂移检测规则（哪些变化必须更新文档）
9. 人工审校规则（谁审、审什么、多久完成）
10. `kb context` 的输入格式和输出格式（便于后续接入 AI coding 流程）

---

## 10. 关联文档索引

| 文档 | 定位 |
|------|------|
| [ai-coding-system-superpowers-reference.md](ai-coding-system-superpowers-reference.md) | superpowers skills 方法借鉴与本地化改造映射 |
| [skill-designs-overview.md](skill-designs-overview.md) | Skill 体系详细设计（输入/输出/流程/契约） |
| [knowledge-base-generation-spec-v1.1.md](knowledge-base-generation-spec-v1.1.md) | 知识基座生成规范（详细版） |
| [requirement-phase-spec.md](requirement-phase-spec.md) | Requirement 阶段规范 V1.0 |
| [design-phase-spec.md](design-phase-spec.md) | Design 阶段规范 V1.0 |
| [requirement-phase-process.md](requirement-phase-process.md) | Requirement 阶段过程记录（实操案例） |
| [requirement-to-design-process-2026-04-09.md](requirement-to-design-process-2026-04-09.md) | Requirement → Design 过程记录（实操案例） |
| [ai-coding-system-discussion.md](ai-coding-system-discussion.md) | 原始讨论稿（完整历史记录） |
