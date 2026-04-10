# Skill 体系详细设计（V1.0）

## 文档信息
- 状态: 设计阶段，本轮均不实现
- 日期: 2026-04-09
- 来源: 从 `ai-coding-system-scheme.md` 第 7 章拆出
- 定位: Skill 的完整输入/输出/流程/契约定义；总纲只保留纵览表

---

## 1. 知识包生成 Skill

### 1.1 定位
输入 controller 文件路径，自动生成该 controller 的完整知识包。

### 1.2 输入
- 必填: `controller_path`
- 可选: `module_name_override`、`owner`、`strict_mode`（缺关键依赖时是否失败）

### 1.3 核心流程
1. 解析 controller（路由、请求类型、service 注入、方法映射、注解）
2. 解析 service（接口与实现、方法级逻辑、关键分支、异步行为）
3. 解析 mapper + xml（方法映射、SQL 段、表依赖、数据源依赖）
4. 抽取请求模型（关键字段和业务含义）
5. 生成完整知识包（controller/service/module/feature/relation/table/index.md/index.json）
6. 命名冲突处理（按 V1.1 规则自动升级文件名）
7. 写入证据（所有关键结论附文件路径 + 行号）
8. DDL 占位区（所有 table 文档自动写入 `DDL（人工补充，必填）` 模板）
9. 状态控制（新文档统一 `draft`）

### 1.4 输出
1. 文件产物清单（绝对路径）
2. 依赖摘要: controller → service → mapper → table
3. 风险摘要: 缺失实现、跨数据源、异步语义、可能命名冲突
4. 审核待办清单: DDL 补齐项、高风险规则复核项

### 1.5 规范约束
必须遵循 `knowledge-base-generation-spec-v1.1.md`。

### 1.6 实现边界（本轮不实现）
1. 本轮仅固化设计，不编写脚本。
2. 落地时再确定: 技术实现方式（脚本/模板引擎/AST+正则）、失败重试与错误分级策略、批量模式（多 controller 一次生成）。

---

## 2. Gate Check Skill

### 2.1 定位
阶段完成前的强制校验，对应 `verification-before-completion` 原则。

与 superpowers 的对应关系：
```text
verification-before-completion : 代码完成前验证
= requirement-gate-check       : Requirement 阶段完成前验证
```

### 2.2 核心逻辑（以 Requirement Gate 为例）

```markdown
---
name: requirement-gate-check
description: Use when Requirement phase is complete, before entering Design -
             checks all Gate conditions, outputs a pass/fail report, and handles human sign-off
---
```

**执行步骤：**
1. 读取当前功能的 `requirement.spec.md`
2. 读取 `uncertainty/register.md`
3. 逐条核对 Gate 条件（5条）
4. 输出 Gate 报告表格（见总纲 6.4 规范）并追加到 `requirement.spec.md` 末尾
5. 明确声明 PASS 或 FAIL

### 2.3 分支处理

**FAIL 分支：**
- 列出必须解决的项
- 不得进入 Design，不得询问放行

**PASS 分支：**
- 主动询问人是否放行："Gate 检查通过，是否确认放行进入 Design？"
- 若人确认放行：
  1. 将 `requirement.spec.md` 文档信息中 `status` 改为 `approved`
  2. 将 Gate 报告中放行选项改为 `[x] 确认进入 Design`
  3. 告知放行完成，可进入 Design 阶段
- 若人拒绝放行或要求补充：
  - 记录回退原因，不修改文件状态

### 2.4 铁律
- 未输出 Gate 报告前，不得声称 Requirement 阶段完成。
- PASS 后必须主动询问放行，不得跳过。
- 未经人确认，不得自行修改 status 为 approved。

### 2.5 扩展
Design Gate / Implementation Gate 可按同一模式扩展为独立 Skill。

### 2.6 实现边界（本轮不实现）
1. 本轮仅固化设计，不编写脚本。
2. 后续实现时参考 superpowers skill 文件结构与格式规范。

---

## 3. 主设计反向生成 Skill

### 3.1 背景与痛点
1. 项目历史功能已积累较多，但均无主设计文档（`design/design.md`）。
2. 直接进入新需求 Design 阶段时，AI 缺少接口契约基线，只能写增量描述，无法输出变更后的完整契约，导致联调时字段对不上。
3. 人工从零补写主设计成本高，且容易遗漏接口细节。
4. 根本解法：提供一个 Skill，输入 controller 文件路径，从现有代码反向生成该功能的初始主设计 `design.md`。

### 3.2 与知识包 Skill 的关系

| Skill | 输入 | 输出 | 定位 |
|-------|------|------|------|
| 知识包生成 | controller 路径 | 8 类知识文档 | 知识基座层，供 AI 检索 |
| 主设计反向生成 | controller 路径 + 知识基座（可选） | `design/design.md` | Design 层，供迭代设计输入与人工阅读 |

推荐串联：**先跑知识包 Skill 建基座 → 再跑主设计 Skill 生成 `design.md`**。

### 3.3 执行流程
1. 解析 controller → 提取路由、接口方法、注解（权限/统计）
2. 读取 Service 实现 → 提取每个接口实际使用的入参字段与响应字段
3. 读取 DTO/VO → 还原字段类型与语义
4. 识别内部注入字段（controller 注入的 `rankType`、service 内部设置的 `platform` 等）→ 标注说明
5. 识别异步语义接口 → 标注"接口成功 ≠ 业务完成"
6. 生成 `design.md` → 结构遵循主设计模板（模块定位、能力边界、历史基线、接口契约、稳定设计决策、Gate 约束、迭代接入规则）
7. 所有接口契约按"每接口独立完整展示"格式输出，不抽公共字段

### 3.4 输出
- `docs/features/<feature>/design/design.md`（初始主设计，状态 `draft`）
- 审核待办清单：高风险字段、异步语义、权限注解异常项

### 3.5 状态控制
生成文档默认 `status: draft`，人工审校通过后改为 `verified`，方可作为迭代 Design 的高置信输入。

### 3.6 实现边界（本轮不实现）
1. 本轮仅固化背景与设计思路，不编写脚本。
2. ���地时需确认：技术实现方式（AST 解析 vs 正则 + 代码阅读）、多 controller 批量模式。
3. 与知识包 Skill 共用命名冲突规则（`service/table` 按 V1.1 规范处理）。

---

## 4. kb 检索命令体系（先定义，不实现）

### 4.1 设计基座的必要性
- Skill 要"快速可用" → 需要稳定目录和统一命名
- Skill 要"结果可信" → 需要状态字段和证据字段
- Skill 要"定位改动链路" → 需要标准关系模型
- Skill 要"持续有效" → 需要 PR 漂移检测机制

### 4.2 命令面

| 命令 | 用途 | 示例 |
|------|------|------|
| `kb index` | 建索引 | `kb index --scope modules,controllers,tables` |
| `kb search` | 通用检索 | `kb search "退款接口超时" --module user --top-k 8` |
| `kb trace` | 链路追踪 | `kb trace --feature "退款"` |
| `kb context` | 需求上下文打包 | `kb context --requirement docs/req/refund-v2.md` |
| `kb read` | 文档读取 | `kb read --doc_id kb-module-user-profile` |
| `kb drift` | 漂移检测 | `kb drift --pr 1234` |

### 4.3 最小命令契约

**`kb search`**
- 输入: `query` / `scope` / `top_k` / `filters(status, module, layer)` / `offset`
- 输出: `hits[{doc_id, title, snippet, layer, status, updated_at}]` / `evidence` / `truncated`

**`kb trace`**
- 输入: `feature|controller|table`（三选一）/ `depth` / `status`
- 输出: `nodes` / `edges` / `paths`（如 `feature→controller→service→table`）

**`kb read`**
- 输入: `doc_id` 或 `path` / `offset` / `limit`
- 输出: `content` / `line_start` / `line_count` / `total_lines` / `file_unchanged`

**`kb context`**
- 输入: `requirement_path` 或 `requirement_text` / `top_k` / `scope` / `strict_verified`
- 输出: `context_pack{hits, evidence, relations, conflicts, confidence_score, missing_context}`

### 4.4 输出标准（给 AI 使用）
- 排序策略: `verified` 优先 → 更新时间近优先 → 证据完整度高优先

---

## 5. Skill 串联建议

```text
知识包生成 Skill ──→ 主设计反向生成 Skill ──→ 日常迭代进入
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

## 6. 关联文档

- [ai-coding-system-scheme.md](ai-coding-system-scheme.md) — 总纲（第 7 章纵览表）
- [ai-coding-system-superpowers-reference.md](ai-coding-system-superpowers-reference.md) — 方法借鉴来源
- [knowledge-base-generation-spec-v1.1.md](knowledge-base-generation-spec-v1.1.md) — 知识基座规范
