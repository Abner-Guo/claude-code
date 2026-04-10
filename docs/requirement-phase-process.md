# Requirement 阶段过程记录 - 3RD运营/评价管理/评价汇总（2026-04-08）

## 1. 目的
1. 记录本次迭代在 Requirement（需求态）如何从原始需求走到 Gate 可通过状态。
2. 作为后续对外讲解与团队复用的标准案例。

## 2. 输入材料
1. 需求原文（飞书）：
   - `https://yumchina.feishu.cn/wiki/DIOZwX649iAvHzka6TOcT4msnCb`
2. 历史功能主设计：
   - `docs/features/3RD运营-评价管理-评价汇总/design/design.md`
3. 历史代码知识基座：
   - `3rd.bi.platform.management/knowledge-base/controllers/controller-appraise-summary.md`

## 3. 执行过程（按统一方法论）

### Step 1：需求接入与结构化草案
1. 先创建本次迭代需求文档：
   - `docs/features/3RD运营-评价管理-评价汇总/requirement/iterations/2026-04-08-3RD运营-评价管理-评价汇总/requirement.spec.md`
2. 在无法直接读取飞书正文时，先基于历史设计与基座输出“可评审草案”。

### Step 2：两级检索 + 双源校验
1. 一级检索：读取历史主设计，明确模块长期边界与稳定约束。
2. 二级检索：读取 controller 知识基座，确认接口簇与关键风险点。
3. 双源校验：将需求草案的结论与历史代码证据进行对齐。

### Step 3：飞书内容拉取
1. 使用 `feishu-doc-integration` 技能读取需求文档。
2. 处理 wiki 链接时采用两段式：
   - `wiki token -> docx token`
   - 再使用 `docx raw_content` 拉正文
3. 读取结果标题：
   - `03 评价汇总新增全国均值卡片`

### Step 4：需求文档回填
1. 回填本次目标、范围、非范围、验收标准。
2. 重点回填项：
   - 新增“全国评分指标均值”卡片
   - 指标选择/同环比改为全局控件
   - 新增有效订单门店过滤
   - 美团三档指标名称调整与新增三档指标

### Step 5：跨状态不明确项管理
1. 将不明确项统一登记到：
   - `docs/features/3RD运营-评价管理-评价汇总/uncertainty/register.md`
2. 将对应风险统一登记到：
   - `docs/features/3RD运营-评价管理-评价汇总/risk/register.md`
3. 对每个不明确项进行状态流转（answered/rejected/open）。

### Step 6：阶段边界校正
1. 已确认“存在契约变更”。
2. 将“字段级契约明细”从 Requirement 下沉到 Design，避免阶段混淆。
3. Requirement 仅保留：
   - 是否有契约变更
   - 影响范围
   - 兼容策略方向

## 4. 不明确项处理结果（Requirement 结束时）
1. `U-REQ-001`：answered（目标与成功指标已明确）
2. `U-REQ-002`：answered（已确认有契约变更）
3. `U-REQ-003`：answered（口径扩展已确认）
4. `U-REQ-004`：rejected（本次不涉及 configDetail 权限策略调整）
5. `U-REQ-005`：rejected（本次不涉及 export 协议改造）
6. `U-REQ-006`：answered（Requirement 仅确认范围与兼容策略，字段级明细下沉 Design）

## 5. Gate 结论
1. Requirement 阶段判定：`通过`
2. 通过依据：
   - 目标、范围、非范围明确
   - 验收标准可测试
   - Requirement 维度 `P0/P1` 不明确项已关闭

## 6. 产出清单
1. Requirement 迭代文档：
   - `docs/features/3RD运营-评价管理-评价汇总/requirement/iterations/2026-04-08-3RD运营-评价管理-评价汇总/requirement.spec.md`
2. 不明确项寄存：
   - `docs/features/3RD运营-评价管理-评价汇总/uncertainty/register.md`
3. 风险寄存：
   - `docs/features/3RD运营-评价管理-评价汇总/risk/register.md`

## 7. 可复用讲解要点（给团队）
1. Requirement 不是写“完整设计”，而是把问题定义清楚并关掉高优不确定性。
2. 契约变更只需在 Requirement 确认“有无 + 影响范围 + 兼容方向”。
3. 字段级 before/after 矩阵必须在 Design 阶段输出。
4. 不明确项和风险必须跨状态同源管理，不能散落在单文档里。
