# 3RD运营-评价管理-评价汇总：Requirement -> Design 过程记录（实操版）

## 文档信息
- 记录日期: 2026-04-09
- 适用迭代: `2026-04-08-03-评价汇总新增全国均值卡片`
- 目的: 复盘本次从 Requirement 进入 Design 的真实过程，便于后续讲解复用

## 1. 输入材料
1. 新迭代需求来源（飞书）
   - `https://yumchina.feishu.cn/wiki/DIOZwX649iAvHzka6TOcT4msnCb`
2. 历史功能设计主文档
   - `docs/features/3RD运营-评价管理-评价汇总/design/design.md`
3. 历史知识基座（Controller）
   - `3rd.bi.platform.management/knowledge-base/controllers/controller-appraise-summary.md`
   - `3rd.bi.platform.management/knowledge-base/controllers/controller-target-config.md`

## 2. Requirement 阶段实际产出
1. 创建本次迭代需求文档
   - `docs/features/3RD运营-评价管理-评价汇总/requirement/iterations/2026-04-08-03-评价汇总新增全国均值卡片/requirement.spec.md`
2. Requirement 阶段明确的核心结论
   - 新增“全国评分指标均值”卡片
   - 指标选择与同环比作为全局控件
   - 新增“有效订单门店”过滤（默认 >=10，范围 1~10000）
   - 美团指标适配（旧三档显式化 + 综合体验分三档新增）
3. Requirement 阶段确定边界
   - 本轮有接口契约变更
   - 字段级契约明细下沉到 Design 阶段

## 3. 从 Requirement 进入 Design 的关键动作
1. 新建本次迭代 Design 文档
   - `docs/features/3RD运营-评价管理-评价汇总/design/iterations/2026-04-08-03-评价汇总新增全国均值卡片/design.md`
2. 基于 Requirement 结论补全 Design 的契约变化与实现方向
   - `/summary` 增加 `summaryScene` 区分 `detail_total` 与 `national_card`
   - 所有数据接口统一新增公共参数 `validOrderMin`
   - 全国卡片口径复用评分明细全国总计主链路
3. 对齐业务澄清并回写 Design
   - `validOrderMin` 作用于所有数据接口
   - 不要求回补最早生效日期
   - 查询不到数据时允许返回 `null`
   - 指标真实计算为 0 时允许返回 `0`
4. 明确指标来源
   - 美团指标适配以 `/api/target/view`（target-config）结果驱动，不在评价汇总侧硬编码

## 4. Design 阶段落地结果（本次）
1. 已明确并写入的设计约束
   - `nationalAvgCard` 与“评分明细全国总计”同结构返回
   - 前端负责渲染，不在前端重算口径
   - 统计门店组件仅作用评分明细
2. 已补充“可编码级逻辑”章节
   - 请求入参归一化
   - 场景分流规则
   - 查询与聚合规则
   - 指标配置驱动规则
   - 返回结构规则
3. 已补充“人工待补 TODO”
   - `TODO-DES-001 ~ TODO-DES-005`（均已挂 owner）

## 5. 同步更新的跟踪文档
1. 不明确项寄存
   - `docs/features/3RD运营-评价管理-评价汇总/uncertainty/register.md`
2. 风险寄存
   - `docs/features/3RD运营-评价管理-评价汇总/risk/register.md`
3. 本轮状态同步
   - `U-DES-101 / R-DES-101` 已由待定改为 `answered`

## 6. 当前停留点（便于讲解）
1. 本轮 Design 主体已成型，已达到“可继续 Implementation”的输入质量。
2. 当前按你的决定：Design 暂不继续改动，等待人工补充 TODO 明细后再推进编码。

## 7. 讲解时可用的一句话版本
- 本次是先用飞书需求 + 历史设计/知识基座完成 Requirement 定义，再把契约变化与口径规则下沉到 Design，并通过不明确项/风险寄存完成闭环，最后在 Design 停在”人工 TODO 待补”节点。

---

## 8. Design 阶段实际过程复盘（含澄清过程）

> 本节记录 Design 文档从草稿到 Gate PASS 的完整澄清迭代过程，便于讲解”设计是怎么一步步被逼出来的”。

### 8.1 不确定项的解决过程

**U-DES-104：综合体验分三档阈值从哪来？**

- 初始假设：阈值由指标配置系统（`/target/view` 接口）驱动，设计文档照此起草。
- 澄清动作：读取 `controller-target-config.md` + 查阅 `TargetConfigService`，发现 `/target/view` 只返回指标元数据（名称、字段名、说明），`t_data_target_config` 表没有阈值字段。
- 反查代码：在 `AppraiseSummaryMapper.xml` 中找到 `merchant_rate_score_low/middle/high` 的实现，确认阈值 4.6/4.8 是硬编码在 SQL CASE WHEN 语句中的。
- 结论：`overallExperienceScore` 三档采用完全相同的接入模式，阈值同样硬编码为 4.6/4.8。

**教训**：设计阶段对”指标配置系统”的理解是错的，靠读代码纠正了假设。Uncertainty Register 的价值在于把这个”不确定”显式登记出来，而不是靠经验猜。

---

### 8.2 SQL 实现从”理解意图”到”对齐格式”的过程

**第一轮：字段名错误**

- 初稿写的是 `overall_experience_score_latest`（参考了其他字段命名惯例）。
- 业务补充：实际字段是 `third_t_order_summary.overall_experience_score`，没有 `_latest` 后缀。
- 修正：字段名改为 `overall_experience_score`。

**第二轮：SQL 逻辑不对齐**

- 初稿 SQL 逻辑是自行推导的，格式和 NULL 处理与现有代码不一致。
- 要求：参考 `merchant_rate_score_low/middle/high` 的写法保持一致。
- 动作：精确读取 `AppraiseSummaryMapper.xml` 第 243~284 行，逐字对齐：
  - CASE WHEN 条件写在单行
  - IS NULL 时返回 0，不返回 NULL
  - Low 档额外排除 `is not null and != 0`（无评分门店排除）
  - 最前面加逗号（`,`），接在上一个字段后面

**第三轮：joinGradeStoreNumber 缺少字段**

- 发现 `grade` 子查询（`joinGradeStoreNumber` / `joinGradeStoreNumber2`）中没有 `overall_experience_score` 字段，CASE WHEN 引用 `grade.overall_experience_score` 会报错。
- 需要在两个 fragment 的内外层 SELECT 中各补一行，才能让 `targetColumn` 里的 CASE WHEN 正确引用。
- 这是一个”改动两处缺一不可”的依赖关系，设计文档第 8.4 节明确标注了这点。

---

### 8.3 validOrderMin 过滤规则的决策过程

**字段名澄清**

- 设计初稿写的是 `valid_order_cnt`（推测命名）。
- 业务补充：实际字段是 `third_t_order_summary.valid_order_number`。

**与 showOrderNumberSwitch 的关系**

- 初稿设计：`validOrderMin > 0` 时 `showOrderNumberSwitch` 不叠加生效（互斥，用 `if/else if`）。
- 理由：语义更清晰，避免双重过滤让人困惑。
- 实际决策：叠加无影响（`valid_order_number >= N` 已隐含 `total_order_cnt != 0`），改为两个独立 `if`，代码更简单。
- 修改：8.3 节伪代码从 `if/else if` 改为两个独立 `if`，5.1 节去掉互斥约束描述。

**教训**：设计约束不需要为了”看起来严谨”而增加复杂度。叠加是否有影响是个技术问题，想清楚了就可以简化，不用保留防御性约束。

---

### 8.4 Design Gate 通过时的状态

| 项 | 状态 |
|---|---|
| 所有 P1 不确定项 | answered |
| TODO-DES-001（阈值确认） | closed（4.6 / 4.8，与商户评分三档一致） |
| SQL 实现逻辑 | 可直接编码，含两处改动点说明 |
| validOrderMin 过滤字段 | 明确为 `third_t_order_summary.valid_order_number` |
| 与旧参数兼容策略 | 两个条件独立追加，可叠加 |

---

### 8.5 讲解要点

1. **Uncertainty Register 是发现问题的抓手，不是形式**：U-DES-104 的存在让”指标阈值从哪来”这个问题被显式提出，而不是靠经验猜，最终靠读代码确认，避免了错误实现。

2. **设计文档要能被直接编码**：第 8 节的实现逻辑细化达到”伪代码 + 精确 SQL”级别，开发拿到就能写，不需要二次澄清。

3. **澄清不是质疑，是减少返工**：SQL 字段名错了、格式没对齐、缺少关联字段，这些如果带进 Implementation 都是联调 bug，Design 阶段发现成本最低。
