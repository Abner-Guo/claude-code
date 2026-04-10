# Human + AI 协同 Coding 系统方案（讨论稿）

## 文档信息
- 状态: Draft
- 目标: 作为我们共同讨论和持续更新的单一文档
- 更新时间: 2026-04-03

## 1. 当前已对齐目标（初稿）
- 方向: 提升迭代交付效率，并推动开发流程规范化
- 长期目标: 让 AI 承担高比例 coding 工作（目标值后续细化）
- 讨论方式: 先定流程与架构，再细化指标和阶段目标

### 1.1 整体流程设计思想（总纲）

#### 1.1.1 核心定位
1. 人负责决策和准入，AI 负责高强度执行与文档化沉淀。
2. 系统目标不是“做完功能”，而是“可持续迭代地做对功能”。
3. 所有流程设计围绕三件事：可验证、可追溯、可持续更新。

#### 1.1.2 统一方法论（状态机思路）
1. 采用统一状态机推进工作流：
   - `Requirement -> Design -> Implementation -> Verification -> Knowledge Sync`
2. 每个状态必须定义:
   - 明确输入（输入契约）
   - 明确输出（产物契约）
   - 明确门禁（通过条件）
3. 未通过当前门禁，不得进入下一状态。

#### 1.1.3 四个流程原则
1. 阶段化（Stage-based）
   - 先定义问题，再定义方案，再进入执行，避免边做边改方向。
2. 门禁化（Gate-driven）
   - 需求、设计、代码、知识四类门禁前置，禁止“先合并后补材料”。
3. 证据化（Evidence-first）
   - 结论必须带代码证据、测试证据、提交证据，避免“经验性正确”。
4. 闭环化（Closed-loop）
   - 交付后必须回灌知识基座，确保下一次 AI 检索可复用、可置信。

#### 1.1.4 与后续章节映射关系
1. `2.1 历史知识建基流程` 对应“Knowledge Sync”的历史补基与治理。
2. `2.2 新需求处理流程` 对应“Requirement -> Design -> Implementation -> Verification”主链路。
3. `2.3 质量与治理闸口` 对应全流程统一 Gate 系统（跨阶段共用）。
4. `3/4/10 章` 对应该方法论的工具化落地（KB + Skill + 自动化规则）。

### 1.2 跨状态不明确项管理（Uncertainty Register）

#### 1.2.1 机制目的
1. 不明确项必须显式化，禁止“隐式假设推进”。
2. Human 与 AI 基于同一不明确项清单进行决策与准入。

#### 1.2.2 强制登记规则
1. 在任一状态（Requirement/Design/Implementation/Verification/Knowledge Sync）发现不明确点，必须登记到 Uncertainty Register。
2. 每条不明确项至少包含以下字段：
   - `id`
   - `state`
   - `question`
   - `impact`
   - `default_assumption`
   - `owner`
   - `due_at`
   - `status`（`open|answered|accepted-risk|rejected`）

#### 1.2.3 通过与放行规则
1. `P0/P1` 级不明确项未关闭时，不得通过当前状态 Gate。
2. `P2/P3` 级可带风险放行，但必须满足：
   - `status=accepted-risk`
   - 明确 `owner`
   - 明确 `due_at`
3. Human 可进行“带风险放行”，但必须记录决策人、时间与理由。

## 2. 流程主线（已确认）

### 2.1 历史知识建基流程（重点细化）

#### 2.1.1 总体原则
1. 先建立“可用知识基座”，再追求“完整知识基座”
2. 先框架后细节，先主干后分支，避免一次性做大全导致失败
3. 每份文档都要可追溯到代码证据（路径、行号、提交）
4. 文档必须可持续更新，不做一次性静态产物

#### 2.1.2 分层生成顺序（结合你的想法）
1. 框架层（系统/仓库级）
   - 基于现有代码框架生成 `architecture-overview.md`
   - 目标: 识别系统边界、公共模块、依赖关系、主调用链
   - 说明: 该层会依赖公共模块信息，作为后续所有模块文档的上位上下文
2. 功能模块层（业务域/子系统级）
   - 为每个功能模块生成 `module-<name>.md`
   - 目标: 明确模块职责、输入输出、对外接口、关键流程
3. Controller 层（接口入口级）
   - 按 controller 维度生成 `controller-<name>.md`
   - 目标: 明确请求入口、参数校验、调用链路、错误处理、鉴权逻辑
4. 数据表层（数据模型级）
   - 按数据库表生成 `table-<name>.md`
   - 目标: 说明表结构用途、被哪些功能点使用、读写路径、一致性约束

#### 2.1.3 推荐执行顺序（可落地）
1. 第 1 轮: 框架层
   - 产出系统框架总览文档
   - 组织一次架构评审，确认模块边界和公共能力定义
2. 第 2 轮: 功能模块层
   - 优先高频迭代模块和核心业务模块
   - 输出模块清单和优先级
3. 第 3 轮: Controller 层
   - 优先外部调用频繁、变更频繁的接口
   - 补齐接口到服务的真实调用链
   - **强制要求**：Controller 层知识文档必须包含字段级接口契约（请求/响应 JSON 结构 + 字段说明），否则不具备 Design 阶段输入资格
4. 第 4 轮: 数据表层
   - 优先核心交易表、主数据表、审计表
   - 建立”表 -> 功能点 -> 接口 -> 代码路径”的映射

#### 2.1.4 每层文档最小必填字段（MVP）
1. 基本信息: 名称、owner、最后更新时间、状态（draft/verified）
2. 职责边界: 做什么、不做什么、上下游依赖
3. 关键流程: 正常流 + 异常流
4. 代码证据: 文件路径、关键函数、行号、最近提交
5. 风险与约束: 性能、事务、一致性、权限、安全
6. **接口契约（Controller 层强制）**: 每个接口的请求/响应字段级结构（JSON 样例 + 字段说明表），作为 Design 阶段契约变更的基线；缺失此项不得作为 Design 阶段输入

#### 2.1.5 审校与准入机制
1. AI 初稿生成后，不直接入库，必须经过人审
2. 审校角色建议:
   - 框架层: 架构师/Tech Lead
   - 模块层: 模块 owner
   - Controller 层: 接口 owner
   - 表层: 数据库 owner + 业务 owner
3. 审校完成后标记 `verified`
4. 未 `verified` 的文档可被检索但不作为高置信输入

#### 2.1.6 持续更新机制（防过期）
1. PR 触发“文档漂移检测”
2. 命中以下变化时强制补文档:
   - controller 路由/参数/返回结构变化
   - service 核心流程变化
   - 数据库 schema 变化
   - 公共模块契约变化
3. CI 门禁:
   - 代码变化命中规则但无文档更新时阻断合并
   - 或者至少创建文档更新任务并设置时限（过期阻断）

#### 2.1.7 你提出方案的评价结论
1. 方向正确，尤其“按 controller 维度”和“按表映射功能点”非常适合迭代型团队
2. 需要补充“先框架层”的前置步骤，否则会出现局部文档互相矛盾
3. 建议采用“4 层递进 + 分批审校”，比一次性全量生成更稳

### 2.2 新需求处理流程（已确认）
#### 2.2.0 Requirement（需求态）执行规范（统一方法论）
1. Requirement 阶段必须先做上下文检索与澄清，不允许直接进入方案设计或编码。
2. 检索采用“两级检索 + 双源校验”：
   - 一级检索（入口文档）: 优先读取历史 `design.md`/`solution.design.md`
   - 二级检索（证据文档）: 自动读取关联 `module-*.md`/`controller-*.md`/`table-*.md`
   - 双源校验: 文档结论必须回查代码证据（file + line + commit）与近期 PR/测试基线
3. Requirement 阶段采用双路径规则：
   - Path A（有历史设计）:
     - 必须完成 `design -> module/controller/table -> 代码证据回查` 全链路
   - Path B（无历史设计）:
     - 必须先生成 `design-baseline.md`（最小基线设计）
     - 基线至少包含: 边界、术语、模块草图、关键流程、验收口径、风险假设
4. Requirement Gate（通过条件）统一为：
   - 验收标准可测试
   - 范围/非范围明确
   - 依赖与风险显式化
   - 证据来源明确（Path A: 历史设计+代码回查；Path B: 基线设计+当前代码扫描）
5. Gate 失败处理：
   - 不允许进入 Design 阶段
   - 回退到 Requirement 补充与澄清，直至满足通过条件
6. 本阶段发现的不明确项必须登记到 Uncertainty Register，并按全局规则处理。

1. 需求输入标准化
   - 将原始需求整理为结构化 `requirement.spec.md`
   - 至少包含: 背景、范围、约束、验收标准、非功能要求
2. 上下文检索与冲突分析
   - 检索历史 `design.md`、接口契约、近期 PR、测试基线
   - 输出 `impact.analysis.md`（影响模块、改动点、兼容性风险）
3. 方案生成与决策
   - 生成 `solution.design.md`（方案对比、选型理由、数据/接口变更、回滚策略）
   - 由人做最终架构决策
4. 任务拆解与执行
   - 将方案拆解为可执行任务（编码、测试、迁移脚本、文档更新）
   - AI 完成主要 coding，人负责关键审查与合并闸口
5. 交付与知识回灌
   - 合并后自动回写模块 `design.md`、变更日志、FAQ
   - 将本次需求沉淀为可检索案例

#### 2.2.1 Design（设计态）执行规范（统一方法论）
1. Design 阶段定位为“决策态”，不是“写文档态”；目标是把 Requirement 转成可实施、可验证、可回滚的方案。
2. 输入契约：
   - 已通过 Requirement Gate 的 `requirement.spec.md`
   - Requirement 阶段产出的上下文包（历史设计/知识文档/代码回查证据）
   - 不明确项清单（Uncertainty Register）
3. 核心步骤：
   - 影响面建模：模块、接口、数据表、兼容性、迁移范围
   - 方案分叉：至少给出 2 个可实施方案
   - 方案收敛：给出推荐方案，并写明“不选理由”
   - 实施切片：拆分为可独立交付、可独立验证的任务包
   - 风险与回滚：明确技术风险、发布风险、数据风险与回滚路径
4. 输出契约：
   - `impact.analysis.md`
   - `solution.design.md`
   - `implementation.slices.md`（建议，作为 Implementation 输入）
   - `decision.log.md`（建议，记录最终决策与 tradeoff）
5. Design Gate（通过条件）：
   - 方案边界明确（做什么/不做什么）
   - 推荐方案已落锤（含不选理由）
   - 兼容性与迁移策略明确
   - 回滚路径可执行
   - 每个实施切片具备可验证验收标准
   - 不明确项已完成“分级处置 + 风险记录 + 责任绑定”
   - `P0/P1` 不明确项为 0（禁止带风险通过）
   - `P2/P3` 仅允许 `accepted-risk + owner + due_at` 后放行
6. Gate 失败回退：
   - 需求歧义导致失败：回 Requirement 补充澄清
   - 技术不可行导致失败：回 Design 重新分叉方案
   - 风险不可控导致失败：缩范围重切片后重审
7. 本阶段发现的不明确项必须登记到 Uncertainty Register，并按全局规则处理。
8. Design 阶段的不明确项处理细则（强制）：
   - 在本阶段结束前，`status=open` 的不明确项必须完成状态转移（`answered|accepted-risk|rejected`）
   - 每条不明确项必须写入风险分级（`P0|P1|P2|P3`）与风险记录
   - 风险记录最小字段：
     - `risk_id`
     - `uncertainty_id`
     - `severity`
     - `scenario`（可能出现的问题场景）
     - `impact_scope`（影响模块/接口/数据）
     - `trigger_condition`（触发条件）
     - `mitigation`（缓解措施）
     - `rollback_plan`（失败后的回退策略）
     - `owner`
     - `review_decision`（human 决策结论）
   - 未完成风险记录的不明确项，视为未关闭，不得通过 Design Gate

#### 2.2.2 Design 与 Implementation 分界（本轮新增）
1. 结论：
   - 具体“业务实现逻辑”必须在 Design 阶段明确到可编码级；
   - Implementation 阶段不再讨论业务口径，只做代码实现与工程化落地。
2. Design 必须明确内容（最低要求）：
   - 接口入参/出参与可空语义（契约级）
   - 关键业务规则与分支（正常流/异常流）
   - 查询过滤与聚合口径（至少到 SQL 粒度策略）
   - 场景分流规则（同接口不同场景如何区分）
   - 可对账一致性约束（例如卡片值与总计值一致）
3. Implementation 负责内容：
   - 类/方法/SQL 的具体编码实现
   - 性能优化与重构细节
   - 自动化测试、联调修正、发布执行
4. Design 阶段人工 TODO 机制（强制）：
   - 对于需要人工补充的信息，必须在 Design 文档中登记 `TODO-DES-*`
   - 每条 TODO 必须包含 `owner`，并在进入 Implementation 前完成或明确放行决策
   - 未登记 owner 的 TODO 视为未完成，不得通过 Design Gate

### 2.3 质量与治理闸口（流程内置控制点）
1. 需求闸口: 结构化需求是否完整
2. 设计闸口: 是否完成影响分析与方案评审
3. 代码闸口: 测试、静态检查、安全扫描是否通过
4. 知识闸口: 设计文档是否已同步更新
5. 不明确项闸口: `P0/P1` 未关闭不得过 Gate；`P2/P3` 仅允许 `accepted-risk + owner + due_at` 后放行

## 3. 可用知识基座规范 V1（新增）

### 3.1 目标
1. 标准化沉淀: 所有知识可追溯、可更新、可审校
2. 快速检索: 支持关键词 + 语义 + 关系链路混合召回
3. 高置信输入: AI 默认优先使用 `verified` 且证据完整的知识

### 3.2 目录结构（建议混合式）
```text
knowledge-base/
  architecture/
  modules/
  controllers/
  tables/
  features/
  relations/
  indexes/
  changelog/
```

### 3.3 文档命名规范
1. `architecture-overview.md`
2. `module-<module_name>.md`
3. `controller-<controller_name>.md`
4. `table-<table_name>.md`
5. `feature-<feature_name>.md`
6. `relation-<domain_or_module>.md`

### 3.4 YAML 头（每个 md 必填）
```yaml
id: kb-module-user-profile
layer: module
module: user-profile
owner: team-a
status: draft|verified|deprecated
updated_at: 2026-04-03
source_commit: abcdef1
tags: [user, profile, auth]
confidence: 0.0-1.0
```

### 3.5 正文最小结构（统一模板）
1. 职责与边界
2. 关键流程（正常流/异常流）
3. 上游下游依赖
4. 关键代码入口
5. 数据对象/表映射
6. 风险与约束
7. 证据清单

### 3.6 证据规范（强制）
1. 每条关键结论附: 文件路径 + 行号 + 提交哈希
2. 示例: `src/user/UserController.ts:88 (commit abcdef1)`
3. 无证据条目不得进入高置信检索结果

### 3.7 关系模型规范（用于链路检索）
1. `FEATURE_USES_CONTROLLER`
2. `CONTROLLER_CALLS_SERVICE`
3. `SERVICE_READS_TABLE`
4. `SERVICE_WRITES_TABLE`
5. `MODULE_OWNS_TABLE`
6. `MODULE_DEPENDS_ON_MODULE`
7. 关系字段: `from_id`、`to_id`、`relation_type`、`evidence`、`updated_at`

### 3.8 生命周期与门禁
1. 新生成文档默认 `draft`
2. owner 审核后改为 `verified`
3. PR 命中漂移规则时必须更新对应文档
4. 过期文档标记 `stale` 并在检索中降权

## 4. Skill 检索设想（新增，作为基座设计佐证）

### 4.1 为什么要这样设计知识基座
1. Skill 要做到“快速可用”，必须有稳定目录和统一命名
2. Skill 要做到“结果可信”，必须有状态字段和证据字段
3. Skill 要做到“定位改动链路”，必须有标准关系模型
4. Skill 要做到“持续有效”，必须依赖 PR 漂移检测机制

### 4.2 Skill 命令面（先定义，不实现）
1. 建索引
   - `kb index`
   - `kb index --scope modules,controllers,tables`
   - `kb index --incremental --from-commit <sha>`
2. 通用检索
   - `kb search "退款接口超时"`
   - `kb search "用户注销" --module user --top-k 8`
   - `kb search "订单状态流转" --layer controller,table --status verified`
3. 链路追踪
   - `kb trace --feature "退款"`
   - `kb trace --controller "RefundController"`
   - `kb trace --table "t_order_refund"`
4. 需求上下文打包
   - `kb context --requirement docs/req/refund-v2.md --top-k 12`
5. 漂移检测
   - `kb drift --pr 1234`
   - `kb drift --from-commit <sha1> --to-commit <sha2>`

### 4.3 Skill 输出标准（给 AI 使用）
1. `context-pack` 内容:
   - `hits`: 命中文档片段
   - `evidence`: 命中证据
   - `relations`: 关联链路
   - `conflicts`: 冲突结论
   - `confidence_score`: 总置信度
2. 排序策略:
   - `verified` 优先
   - 更新时间近优先
   - 证据完整度高优先

### 4.4 `kb` 最小命令契约 V1（新增）
1. `kb search`
   - 输入字段:
     - `query`: string
     - `scope`: `all|architecture|modules|controllers|tables|features`
     - `top_k`: number（默认 8）
     - `filters`: `{ status?: verified|draft|stale, module?: string, layer?: string[] }`
     - `offset`: number（默认 0）
   - 输出字段:
     - `hits`: `[{ doc_id, title, snippet, layer, status, updated_at }]`
     - `evidence`: `[{ doc_id, file, line, commit }]`
     - `applied_limit`: number
     - `applied_offset`: number
     - `truncated`: boolean
2. `kb trace`
   - 输入字段:
     - 三选一: `feature|controller|table`
     - `depth`: number（默认 3）
     - `status`: `verified|all`
   - 输出字段:
     - `nodes`: `[{ id, type, name, status }]`
     - `edges`: `[{ from_id, to_id, relation_type, evidence }]`
     - `paths`: `string[]`（示例: `feature->controller->service->table`）
3. `kb read`
   - 输入字段:
     - `doc_id` 或 `path`
     - `offset`: number（按行）
     - `limit`: number（按行）
   - 输出字段:
     - `content`: string
     - `line_start`: number
     - `line_count`: number
     - `total_lines`: number
     - `file_unchanged`: boolean（命中去重缓存时）
4. `kb context`
   - 输入字段:
     - `requirement_path` 或 `requirement_text`
     - `top_k`: number（默认 12）
     - `scope`: 同 `kb search`
     - `strict_verified`: boolean（默认 true）
   - 输出字段:
     - `context_pack`:
       - `hits`
       - `evidence`
       - `relations`
       - `conflicts`
       - `confidence_score`
       - `missing_context`（缺失上下文列表）

## 5. 参考 Claude Code 源码的可借鉴设计（新增）
1. 工具分层设计（检索工具与读取工具分离）
   - 参考文件:
     - `src/tools/GrepTool/GrepTool.ts`
     - `src/tools/GlobTool/GlobTool.ts`
     - `src/tools/FileReadTool/FileReadTool.ts`
   - 借鉴点:
     - 先 `search/glob` 缩小范围，再 `read` 精读，避免全量读取
     - 工具输入输出 schema 化，便于稳定调用和审计
2. 统一权限控制与规则优先级
   - 参考文件:
     - `src/utils/permissions/filesystem.ts`
   - 借鉴点:
     - 统一入口 `checkReadPermissionForTool`
     - `deny/ask` 优先于 `allow`
     - 把 deny 规则转为 ignore pattern，检索阶段就降噪
3. 检索性能与稳定性
   - 参考文件:
     - `src/utils/ripgrep.ts`
     - `src/utils/glob.ts`
     - `src/tools/GrepTool/GrepTool.ts`
   - 借鉴点:
     - ripgrep 作为底层检索引擎
     - 默认 `head_limit + offset` 分页
     - 超时、缓冲、重试（EAGAIN 单线程降级）机制
4. 读取限额与防失控机制
   - 参考文件:
     - `src/tools/FileReadTool/limits.ts`
     - `src/utils/readFileInRange.ts`
     - `src/tools/FileReadTool/FileReadTool.ts`
   - 借鉴点:
     - `maxSizeBytes + maxTokens` 双阈值
     - `offset/limit` 行级读取
     - 设备文件黑名单与路径标准化校验
5. 上下文控制与去重缓存
   - 参考文件:
     - `src/tools/FileReadTool/FileReadTool.ts`
     - `src/context.ts`
   - 借鉴点:
     - 同文件同范围未变化时返回 `file_unchanged` stub
     - 上下文预取 + memoize 缓存，降低首轮延迟
6. 对本项目知识基座的落地映射
   - `kb search` 对应 Grep/Glob 思路
   - `kb read` 对应 FileReadTool 的分段读取与限额机制
   - `kb context` 对应 context 聚合与 token 控制
   - `kb drift` 对应规则驱动的持续更新与门禁机制

## 6. 对“历史设计 MD 怎么来”的阶段结论
采用三段式机制:
1. 一次性 AI 反向生成（从 0 到 1）
2. 关键人审校（保证可信度）
3. PR 驱动持续同步（防止再次过期）

## 7. 下一步讨论建议
建议优先细化以下内容:
1. `design.md` 标准模板（章节、粒度、证据格式）
2. 自动生成规则（按模块/按服务/按目录）
3. 文档漂移检测规则（哪些变化必须更新文档）
4. 人工审校规则（谁审、审什么、多久完成）
5. `kb context` 的输入格式和输出格式（便于后续接入 AI coding 流程）

## 8. 待确认问题清单
- 文档体系目录约定（每个模块一个 `design.md` 还是集中式）
- 证据链接格式约定（文件路径、行号、提交哈希）
- 哪些高风险模块需要更严格的人审和发布门禁
- 需求到设计到代码的工单字段映射规范
- `verified` 的时效策略（例如 30/60/90 天）

## 9. 已落地修订（V1.1）

### 9.1 单 Controller 完整知识包（实践结论）
1. 单个 controller 不应只生成 `controller.md`，至少应生成一组完整产物：
   - `controller-*.md`
   - `service-*.md`
   - `module-*.md`
   - `feature-*.md`
   - `relation-*.md`
   - `table-*.md`（按实际依赖表）
   - `indexes/index-*.md`
   - `indexes/index-*.json`
2. `index.md` 用于人工审阅入口，`index.json` 用于机器检索入口（推荐双轨并存）。

### 9.2 命名冲突规则升级（已验证）
1. `service`（含你提到的 server 语义）同样存在命名冲突风险，规则如下：
   - 默认: `service-<service_name>.md`
   - 冲突: `service-<module>-<service_name>.md`
2. `table` 命名冲突规则：
   - 默认: `table-<table_name>.md`
   - 冲突（跨 datasource/schema）:
     - `table-<datasource>-<schema>-<table_name>.md`
3. 强制唯一标识字段（YAML）：
   - service: `service_key = <module>.<service_name>`
   - table: `table_key = <datasource>.<schema>.<table_name>`

### 9.3 Table 文档 DDL 规范（新增强制项）
1. 所有 `table-*.md` 必须包含 `DDL（人工补充，必填）` 区块。
2. DDL 区块至少包含：
   - `CREATE TABLE`
   - 主键/唯一键/索引
   - 分区策略
   - 字段语义说明（字段级）
   - 数据保留策略与口径负责人
3. 未补齐 DDL 的 table 文档状态必须保持 `draft`，不得标记为 `verified`。

### 9.4 人工审核导向的文档粒度（新增）
1. `feature/module/service` 文档应从“简要说明”升级为“可审核说明”，至少包含：
   - 方法级/流程级说明
   - 关键分支规则（如渠道分流、状态机）
   - 异步语义说明（接口成功 != 任务完成）
   - 风险点与审核清单
2. 结论：代码注释不足不应成为文档解释不足的理由；应通过“代码证据 + 业务归纳”补齐。

### 9.5 人工审核标记字段（检索依据）
1. 所有文档审核通过后统一更新 YAML:
   - `status: verified`
   - `verified_by`
   - `verified_at`
   - `review_round`
   - `review_notes`
2. table 文档额外要求:
   - `ddl_status`
   - `ddl_verified_by`
   - `ddl_verified_at`
3. 检索默认使用规则:
   - 普通文档: `status=verified`
   - table 文档: `status=verified` 且 `ddl_status=verified`
4. 未补齐以上字段的文档保持 `draft`，不得作为高置信依据。

## 10. Skill 设计思路（先设计，后落地）

### 10.1 目标
1. 仅输入 `controller` 文件路径，自动生成该 controller 的完整知识包。
2. 输出结果默认 `draft`，保留人工审校和 DDL 补充环节。

### 10.2 Skill 输入（最小化）
1. 必填:
   - `controller_path`
2. 可选:
   - `module_name_override`
   - `owner`
   - `strict_mode`（缺关键依赖时是否失败）

### 10.3 Skill 核心流程
1. 解析 controller:
   - 路由、请求类型、service 注入、方法映射、注解（鉴权/统计）
2. 解析 service:
   - 接口与实现、方法级逻辑、关键分支、异步行为
3. 解析 mapper + xml:
   - 方法映射、SQL 段、表依赖、数据源依赖
4. 抽取请求模型:
   - 关键字段和业务含义（用于 feature/module 文档）
5. 生成完整知识包:
   - controller/service/module/feature/relation/table/index.md/index.json
6. 命名冲突处理:
   - service/table 按 V1.1 规则自动升级文件名
7. 写入证据:
   - 所有关键结论附文件路径 + 行号
8. DDL 占位区:
   - 所有 table 文档自动写入 `DDL（人工补充，必填）` 模板
9. 状态控制:
   - 新生成文档状态统一为 `draft`

### 10.4 Skill 输出
1. 文件产物清单（绝对路径）
2. 依赖摘要:
   - controller -> service -> mapper -> table
3. 风险摘要:
   - 缺失实现、跨数据源、异步语义、可能命名冲突
4. 审核待办清单:
   - DDL 补齐项
   - 高风险规则复核项

### 10.5 与规范对齐点
1. 必须遵循:
   - `knowledge-base-generation-spec-v1.1.md`
2. 关键强约束:
   - 单 controller 完整知识包
   - service/table 命名冲突规则
   - table 文档 DDL 必填区
   - `draft -> verified` 人工准入机制

### 10.6 后续实现边界（本轮不实现）
1. 本轮仅固化 Skill 设计，不编写 Skill 脚本。
2. Skill 落地时再确定:
   - 技术实现方式（脚本/模板引擎/AST+正则）
   - 失败重试与错误分级策略
   - 批量模式（多 controller 一次生成）

## 11. 已落地修订（V1.2）

### 11.1 文档目录升级为“按功能聚合”
1. 从“按状态分目录”升级为“按功能分目录，状态作为子目录”。
2. 目标：让跨状态资产（不明确项/风险）与 Requirement/Design 在同一功能根路径下闭环管理。
3. 示例功能路径：
   - `docs/features/3RD运营-评价管理-评价汇总/`

### 11.2 Requirement / Design 双层文档（主文档 + 迭代文档）
1. Requirement：
   - 不设主文档，仅保留迭代文档：`...\requirement\iterations\<date-feature>\requirement.spec.md`
   - 迭代索引：`...\requirement\index.md`
2. Design：
   - 主文档：`...\design\design.md`（功能唯一主设计）
   - 迭代文档：`...\design\iterations\<date-feature>\design.md`
3. 规则：
   - Requirement 评审通过后，更新 `requirement/index.md`。
   - Design 迭代文档评审通过后，必须回写 `design/design.md`。
   - 未完成更新，不得视为该功能状态完成。

### 11.3 跨状态不明确项与风险寄存（功能级）
1. 功能根目录新增：
   - `...\uncertainty\register.md`
   - `...\risk\register.md`
2. 作用：
   - Requirement、Design、Implementation、Verification 共享同一寄存区。
   - 避免状态切换时不明确项与风险信息丢失或重复登记。
3. 闸口约束延续：
   - `P0/P1` 不明确项未关闭不得过 Gate。
   - `P2/P3` 仅允许 `accepted-risk + owner + due_at` 放行。

### 11.4 当前已落地样例（3RD运营-评价管理-评价汇总）
1. `docs/features/3RD运营-评价管理-评价汇总/requirement/index.md`
2. `docs/features/3RD运营-评价管理-评价汇总/requirement/iterations/2026-04-08-3RD运营-评价管理-评价汇总/requirement.spec.md`
3. `docs/features/3RD运营-评价管理-评价汇总/design/design.md`
4. `docs/features/3RD运营-评价管理-评价汇总/design/iterations/2026-04-08-3RD运营-评价管理-评价汇总/design.md`
5. `docs/features/3RD运营-评价管理-评价汇总/uncertainty/register.md`
6. `docs/features/3RD运营-评价管理-评价汇总/risk/register.md`

### 11.5 路径引用规范（新增强制项）
1. 文档内路径一律使用“项目相对地址”，禁止使用本机绝对地址（如 `D:\...`）。
2. 推荐相对路径写法：
   - 项目文档：`docs/features/...`
   - 代码与知识基座：`3rd.bi.platform.management/...`
3. 跨文档引用必须可在仓库根目录下直接定位，避免环境耦合。
4. 发现绝对路径时，视为规范不通过项，必须修正后再进入下一状态。

## 12. Gate 校验机制与 Skill 设计（V1.3）

### 12.1 Gate 的本质定位

1. Gate 是**阶段出口的强制检查点**，不是可选操作。
2. 核心作用：禁止带着未解决的问题进入下一阶段，把风险暴露在前而不是后。
3. 设计参考：superpowers `verification-before-completion` 的核心原则：
   - `NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE`
   - Gate 是这个原则在流程层面的对应物——不是"我觉得需求清楚了"，而是"5条可验证的布尔条件都满足了"。

### 12.2 Gate 触发时机（固定，非可选）

| 场景 | Gate |
|------|------|
| Requirement 写完，准备进入 Design | Requirement Gate |
| Design 写完，准备进入 Implementation | Design Gate |
| P0/P1 不明确项发现时 | 当前阶段 Gate 自动失败 |
| 需求范围不清晰 | Gate 失败，回退重新澄清 |

### 12.3 Gate 校验主体（已确认分工）

**结论：AI 校验 + 人做最终放行。**

1. **AI 职责**：
   - 逐条核对 Gate 条件
   - 读取 `uncertainty/register.md` 核查不明确项状态
   - 输出结构化 Gate 报告（PASS/FAIL + 逐条说明）
   - FAIL 时列出必须解决的项，不得自行推进

2. **人的职责**：
   - 看 AI 的 Gate 报告，做最终放行决策
   - 显式确认"进入下一阶段"或"回退补充"

3. **为什么不允许 AI 自动通过**：
   - Gate 失败的核心风险是 P0/P1 不明确项，通常是业务口径或技术边界问题
   - AI 无法独立判断，只有人知道答案
   - AI 自写自审容易"自我说服"，把模糊写得像清晰

### 12.4 Gate 报告输出规范（强制）

AI 完成 Requirement 阶段后，必须在 `requirement.spec.md` 末尾输出 Gate 报告块：

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

**等待人工放行**: [ ] 确认进入 Design  [ ] 回退补充
```

未输出 Gate 报告前，不得声称 Requirement 阶段完成。

### 12.5 requirement-gate-check Skill 设计（待实现）

#### 12.5.1 定位
与 superpowers 的对应关系：
```
verification-before-completion : 代码完成前验证
= requirement-gate-check       : Requirement 阶段完成前验证
```

#### 12.5.2 Skill 核心逻辑（草稿）

```markdown
---
name: requirement-gate-check
description: Use when Requirement phase is complete, before entering Design -
             checks all Gate conditions, outputs a pass/fail report, and handles human sign-off
---

## 执行步骤
1. 读取当前功能的 requirement.spec.md
2. 读取 uncertainty/register.md
3. 逐条核对 Gate 条件（5条）
4. 输出 Gate 报告表格（见 12.4 规范）并追加到 requirement.spec.md 末尾
5. 明确声明 PASS 或 FAIL

### FAIL 分支
- 列出必须解决的项
- 不得进入 Design，不得询问放行

### PASS 分支
- 主动询问人是否放行：
  "Gate 检查通过，是否确认放行进入 Design？"
- 若人确认放行：
  1. 将 requirement.spec.md 文档信息中 `status` 改为 `approved`
  2. 将 Gate 报告中放行选项改为 `[x] 确认进入 Design`
  3. 告知放行完成，可进入 Design 阶段
- 若人拒绝放行或要求补充：
  - 记录回退原因，不修改文件状态

## 铁律
- 未输出 Gate 报告前，不得声称 Requirement 阶段完成。
- PASS 后必须主动询问放行，不得跳过。
- 未经人确认，不得自行修改 status 为 approved。
```

#### 12.5.3 实现边界（本轮不实现）
1. 本轮仅固化 Skill 设计，不编写 Skill 脚本。
2. 后续实现时参考 superpowers skill 文件结构与格式规范。
3. 同理，Design Gate / Implementation Gate 可按同一模式扩展为独立 Skill。

## 13. 历史主设计反向生成 Skill 设计（待实现）

### 13.1 背景与痛点

1. 项目历史功能已积累较多，但均无主设计文档（`design/design.md`）。
2. 直接进入新需求 Design 阶段时，AI 缺少接口契约基线，只能写增量描述（"入参新增 xxx"），无法输出变更后的完整契约，导致联调时字段对不上。
3. 人工从零补写主设计成本高，且容易遗漏接口细节。
4. 根本解法：提供一个 Skill，输入 controller 文件路径，**从现有代码反向生成该功能的初始主设计 `design.md`**，包含完整的接口契约基线，人工审校后即可作为后续迭代 Design 的输入。

### 13.2 与已有 Skill 的关系

| Skill | 输入 | 输出 | 定位 |
|-------|------|------|------|
| 知识包生成（第10章） | controller 路径 | `controller/service/module/feature/relation/table/index.md` | 知识基座层，供 AI 检索 |
| 主设计反向生成（本章） | controller 路径 + 知识基座（可选） | `design/design.md`（含接口契约基线） | Design 层，供迭代设计输入与人工阅读 |

两个 Skill 可串联：**先跑知识包 Skill 建基座 → 再跑主设计 Skill 生成 `design.md`**。

### 13.3 Skill 核心设计思路

1. **输入**：
   - 必填：`controller_path`（controller 文件路径）
   - 可选：`knowledge_base_path`（已有知识基座路径，优先读取）
   - 可选：`owner`

2. **执行流程**：
   - Step 1：解析 controller，提取路由、接口方法、注解（权限/统计）
   - Step 2：读取 Service 实现，提取每个接口实际使用的入参字段与响应字段
   - Step 3：读取 DTO/VO，还原字段类型与语义
   - Step 4：识别内部注入字段（controller 注入的 `rankType`、service 内部设置的 `platform` 等），标注说明
   - Step 5：识别异步语义接口，标注"接口成功 ≠ 业务完成"
   - Step 6：生成 `design.md`，结构遵循主设计模板（模块定位、能力边界、历史基线、接口契约、稳定设计决策、Gate 约束、迭代接入规则）
   - Step 7：所有接口契约按"每接口独立完整展示"格式输出，不抽公共字段

3. **输出**：
   - `docs/features/<feature>/design/design.md`（初始主设计，状态 `draft`）
   - 审核待办清单：高风险字段、异步语义、权限注解异常项

4. **状态控制**：
   - 生成文档默认 `status: draft`
   - 人工审校通过后改为 `verified`，方可作为迭代 Design 的高置信输入

### 13.4 实现边界（本轮不实现）
1. 本轮仅固化背景与设计思路，不编写 Skill 脚本。
2. 落地时需确认：技术实现方式（AST 解析 vs 正则 + 代码阅读）、多 controller 批量模式。
3. 与知识包 Skill 共用命名冲突规则（`service/table` 按 V1.1 规范处理）。

## 14. 与 superpowers skills 的参考关系（本轮补充）

### 14.1 本轮结论
1. 当前这份讨论稿不是简单照搬兄弟仓库 `../superpowers/skills/`，而是抽取其中的“流程纪律”和“Skill 协议”后，嫁接到企业迭代开发场景。
2. 借鉴重点集中在方法层，而不是业务层。
3. 核心借鉴对象不是具体功能实现，而是：Gate 思维、Skill 文件结构、先设计后实现、可验证切片、文档即执行协议。

### 14.2 最直接的参考来源
1. `verification-before-completion`
   - 借鉴点：`NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE`
   - 在本稿中的体现：第 12 章 Gate 校验机制不是“可选检查”，而是阶段出口的强制验证点。
   - 本地化改造：从“代码完成前验证”升级为“Requirement/Design/Implementation 阶段完成前验证”。
2. `writing-skills`
   - 借鉴点：Skill 的 YAML frontmatter、`name/description` 结构、`Use when...` 触发式描述、面向检索的结构化写法。
   - 在本稿中的体现：第 10/12/13 章对 Skill 的定义方式、输入输出契约、实现边界说明。
3. `brainstorming` + `writing-plans` + `executing-plans`
   - 借鉴点：先设计、后实现；设计输出必须足够细，执行阶段不临场拍脑袋。
   - 在本稿中的体现：第 2.2.0 Requirement 执行规范、第 2.2.1 Design 执行规范、第 2.2.2 Design 与 Implementation 分界。
4. `subagent-driven-development` + `dispatching-parallel-agents`
   - 借鉴点：任务切片、可独立验证、独立问题独立处理、review loop。
   - 在本稿中的体现：`implementation.slices.md`、按功能/模块/接口拆分的知识生成与后续 Skill 扩展方向。

### 14.3 当前讨论稿中最明显的借鉴模块
1. Gate 机制
   - 最明显来源：`verification-before-completion`
   - 对应本稿：第 12 章。
2. Skill 设计方式
   - 最明显来源：`writing-skills`
   - 对应本稿：第 10/12/13 章。
3. Design 先于 Implementation
   - 最明显来源：`brainstorming` + `writing-plans`
   - 对应本稿：第 2.2 节。
4. 切片执行与可验证任务包
   - 最明显来源：`writing-plans` + `subagent-driven-development`
   - 对应本稿：`implementation.slices.md` 与多 Skill 串联设计。

### 14.4 不是照搬，而是本地化扩展
1. `Requirement -> Design -> Implementation -> Verification -> Knowledge Sync` 统一状态机，是在 superpowers 方法之上做的企业迭代治理扩展。
2. `uncertainty/register.md` 与 `risk/register.md` 的跨状态寄存，不是 superpowers 原生结构，而是为业务口径/技术边界澄清引入的显式管理机制。
3. 主设计 `design/design.md` + 迭代设计 `design/iterations/.../design.md` 的双层文档体系，也是面向长期迭代场景的本地化设计。
4. `knowledge-base/`、`relation-*.md`、`index.json`、`kb search/trace/context/drift` 这一整套知识检索体系，已经超出 superpowers 的 task workflow 范畴，属于本项目自己的 AI coding 基座设计。

### 14.5 阶段性判断
1. superpowers 更偏“Agent 工作流纪律”；当前方案更偏“企业迭代开发治理 + AI 知识基础设施”。
2. 当前稿子里，借鉴来的“方法层”和本项目自己的“规范层/治理层/知识层”内容混在一起，是造成文档阅读感受偏乱的原因之一。
3. 因此后续要保留两层文档：
   - 讨论稿：持续记录思考过程和版本修订；
   - 稳定文档：抽取出独立的 superpowers 参考映射文档，并在总纲方案文档中建立入口。

### 14.6 本轮后续动作
1. 抽取独立文档：`docs/ai-coding-system-superpowers-reference.md`。
2. 在总纲文档 `docs/ai-coding-system-scheme.md` 中补充“方法借鉴来源（Superpowers）”章节。

## 15. Implementation 阶段设计讨论（2026-04-10）

### 15.1 核心设计思路

Implementation 阶段采用**双角色编码**模式：
- **Claude Code**：负责精细化编码（Plan → Task 逐个实现）
- **Codex**（OpenAI codex-plugin-cc 插件）：负责编码校验

关键设计决策：
1. 基于 design.md 生成执行 Plan，**Plan 需人工 review 通过后**再生成 Task
2. Task 生成后同样需人工确认，确认后才进入编码
3. 编码完成后生成单元测试
4. 生成变更记录，标注人工重点 review 部分

### 15.2 节点链（讨论版）

```
1.  [S] 读取 design.md，生成执行 Plan
2.  [Q] Plan 人工 review（执行策略、顺序、依赖是否合理）
3.  [S] 基于确认的 Plan 生成 Task 列表（原子操作粒度）
4.  [Q] Task 列表人工确认（粒度、验证点是否合适）
5.  [S] Claude Code 按 Task 编码          ──┐
6.  [S] Codex 语法/编译/类型校验            │ 逐 Task
7.  [Q] 语法冲突项 → 双方理由 → 人工仲裁   │ 循环
8.  [S] Claude Code 按裁决修正              ──┘
9.  [S] Codex 统一校验（逻辑 + 对照 design.md 契约一致性）
10. [Q] 逻辑/契约冲突项 → 双方理由 → 人工仲裁
11. [S] Claude Code 按裁决修正
12. [S] 生成单元测试
13. [S] Codex 校验单元测试
14. [S] 生成变更记录 + 标注人工重点 review
15. [G] Implementation Gate（AI 校验 + 人工放行）
```

### 15.3 两轮 Codex 校验机制

| 轮次 | 时机 | 校验范围 | 原因 |
|------|------|----------|------|
| 第一轮 | 逐 Task 编码后 | 语法/编译/类型 | 代码未完整时校验逻辑会产生大量误报 |
| 第二轮 | 全部 Task 完成后 | 逻辑 + 契约一致性 | 全量代码完成后才有完整上下文，可对照 design.md 做字段名/返回结构/null-0 语义验证 |

### 15.4 冲突仲裁机制

- Claude Code 与 Codex 校验结果有分歧时，**逐项列出冲突点**
- **双方分别给出理由**
- **人工逐条裁决**，Claude Code 按裁决结果修正
- 不做纯 AI 循环仲裁，避免死循环

### 15.5 编码完成后的产出

1. **单元测试**：编码全部完成（含冲突仲裁修正）后生成，覆盖正常流/异常流/边界条件
2. **变更记录**：
   - 变更文件清单 + 摘要说明
   - 标注人工重点 review 部分：契约变更代码、复杂业务逻辑、冲突仲裁修正代码、新增/修改 SQL

### 15.6 阶段边界（已确认）

- Implementation 只管"编码 + 校验 + 测试 + 变更记录"
- Knowledge Sync（回写知识基座、更新接口契约）和主设计回写属于 **Verification 阶段**，不在 Implementation 内

### 15.7 待细化问题

1. Codex 校验的上下文投喂方式：如何把 design.md 的契约信息有效传递给 Codex？
2. codex-plugin-cc 插件的具体集成方式与 API 调用模式
3. Plan 的标准输出格式（便于人工 review 和后续 Task 拆解）
4. Task 的标准输出格式（便于 Claude Code 逐个执行和 Codex 逐个校验）
5. Implementation Gate 的具体校验条件（参考 Requirement/Design Gate 模式）
6. 变更记录的标准格式（便于人工 review 和后续追溯）

---

> 说明: 这是讨论稿，我们后续每轮讨论都直接在本文件增量更新。
