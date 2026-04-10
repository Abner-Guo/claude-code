# Requirement 阶段规范（V1.0）

## 1. 目标
1. 把原始需求转成可评审、可进入 Design 的结构化输入。
2. 明确范围、验收、依赖、风险与不明确项，避免带歧义进入 Design。

## 2. 输入
1. 需求原文（PRD/飞书/wiki/口头整理）。
2. 历史功能主设计（若存在）：`docs/features/<feature>/design/design.md`。
3. 历史知识基座（若存在）：`knowledge-base/controllers|modules|tables/*.md`。
4. 相关代码与近期变更（用于证据回查）。

## 3. 输出（强制）
1. 迭代需求文档：
   - `docs/features/<feature>/requirement/iterations/<date-topic>/requirement.spec.md`
   
2. 跨状态寄存（同一功能共享）：
   - `docs/features/<feature>/uncertainty/register.md`
   - `docs/features/<feature>/risk/register.md`

## 4. 执行步骤
1. 需求归一化
   - 提炼背景、目标、范围/非范围、约束、验收标准。
2. 上下文检索
   - 先看历史 design，再看 controller/module/table 基座，再回查代码证据。
3. 冲突与影响识别
   - 识别接口契约是否变化、统计口径是否变化、是否影响历史能力。
4. 不明确项登记
   - 所有阻塞问题写入 `uncertainty/register.md` 并分级（P0~P3）。
5. 风险同步
   - 为不明确项登记风险记录到 `risk/register.md`。

## 5. Requirement 文档最小结构
1. 文档信息（iteration_key、owner、updated_at、关联路径）。
2. 背景。
3. 本次目标。
4. 范围（In Scope）。
5. 非范围（Out of Scope）。
6. 约束与依赖。
7. 验收标准。
8. 不明确项（Requirement 阶段）。
9. 历史基线证据。
10. 下一步（进入 Design 前）。
11. Gate 规则（Requirement）。

## 6. 分层边界（Requirement vs Design）
1. Requirement 必须明确：
   - 业务目标、范围边界、兼容方向、验收口径。
2. Requirement 不做：
   - 字段级接口契约细节、SQL 粒度、实现分支细节（下沉 Design）。

## 7. Gate（通过条件）
1. 目标、范围、非范围明确。
2. 验收标准可测试。
3. 已确认是否存在契约变更（可只到“有/无”层级）。
4. `P0/P1` 不明确项关闭。
5. `P2/P3` 若未关闭，必须 `accepted-risk + owner + due_at`。

## 8. 失败回退
1. 任一 Gate 不满足，不进入 Design。
2. 回 Requirement 补充澄清并更新寄存文档。

## 9. 本次迭代沉淀（可复用）
1. 新需求进入时可没有历史 design；此时 Requirement 仍可先通过，但需显式记录“历史设计缺失”并在 Design 阶段补基线。
2. Requirement 阶段确认“有契约变更”即可；字段级契约矩阵在 Design 输出。
3. 针对数据语义，Requirement 只定方向（例如允许空值），不定义接口字段级实现。
