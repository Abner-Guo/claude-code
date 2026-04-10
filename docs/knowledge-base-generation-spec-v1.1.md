# 可用知识基座生成规范 V1.1

## 1. 目的与适用范围
- 目的: 为“人+AI 协同 coding”提供可检索、可审计、可持续更新的知识基座标准。
- 适用范围: 企业内部既有系统迭代开发，尤其是基于现有代码、接口、数据库的需求开发。
- 单位范围: 以“单个 Controller”作为最小落地单元，要求一次产出完整知识包。

## 2. 单 Controller 最小交付物（必选）
1. `controller-<name>.md`
2. `service-<name>.md`
3. `module-<name>.md`
4. `feature-<name>.md`
5. `relation-<name>.md`
6. `table-*.md`（该 controller 实际依赖到的表）
7. `indexes/index-<name>.md`
8. `indexes/index-<name>.json`

## 3. 目录结构规范
```text
knowledge-base/
  controllers/
  services/
  modules/
  features/
  relations/
  tables/
  indexes/
  changelog/
```

## 4. 命名规范
### 4.1 Controller / Service / Module / Feature / Relation
- `controller-<controller_name>.md`
- `service-<service_name>.md`
- `module-<module_name>.md`
- `feature-<feature_name>.md`
- `relation-<domain_or_module>.md`

### 4.2 Service 冲突规则
- 默认: `service-<service_name>.md`
- 冲突（同名 service 跨模块）: `service-<module>-<service_name>.md`

### 4.3 Table 冲突规则
- 默认: `table-<table_name>.md`
- 冲突（同名表跨 datasource/schema）: `table-<datasource>-<schema>-<table_name>.md`

## 5. YAML 头规范（所有文档必填）
```yaml
id: kb-xxx
layer: controller|service|module|feature|relation|table|index
module: xxx
owner: team-xxx
status: draft|verified|deprecated|stale
verified_by: <reviewer>
verified_at: YYYY-MM-DD
review_round: <number>
review_notes: <short note>
updated_at: YYYY-MM-DD
source_commit: <git sha or uncommitted>
tags: [a, b, c]
confidence: 0.0-1.0
```

### 5.1 唯一标识字段（强制）
- service 文档必须补充:
  - `service_name`
  - `service_key = <module>.<service_name>`
- table 文档必须补充:
  - `datasource`
  - `schema`
  - `table_name`
  - `table_key = <datasource>.<schema>.<table_name>`

## 6. 各类文档最小内容要求
### 6.1 `controller-*.md`
- 职责与边界
- 路由清单（method + path + request type + response type）
- 调用下游（service 方法映射）
- 横切能力（鉴权、埋点、耗时统计等）
- 风险与审核关注点
- 证据清单（文件 + 行号）

### 6.2 `service-*.md`
- 服务职责
- 方法级说明（输入、核心逻辑、输出）
- 关键分支规则（状态机/渠道分流/版本切换）
- 异步语义说明（接口成功与任务完成关系）
- 风险与补偿机制
- 证据清单

### 6.3 `module-*.md`
- 模块目标
- 端到端流程（controller -> service -> mapper -> table）
- 功能点与责任分配
- 数据源与访问边界
- 命名冲突策略摘要
- 人工审核建议
- 证据清单

### 6.4 `feature-*.md`
- 业务价值
- 用户可见能力清单
- 关键业务规则
- 人工审核清单
- 证据清单

### 6.5 `relation-*.md`
- 节点清单（feature/controller/service/mapper/table）
- 关系边（FEATURE_USES_CONTROLLER, CONTROLLER_CALLS_SERVICE, SERVICE_CALLS_MAPPER, MAPPER_READS_TABLE）
- 关键路径示例
- 证据清单

### 6.6 `table-*.md`
- 表用途
- 本模块使用场景
- 关键字段（基于 SQL 证据）
- 风险与口径说明
- 证据清单

## 7. Table 文档 DDL 规范（强制）
每个 `table-*.md` 必须包含以下固定区块:
- `DDL（人工补充，必填）`
- `ddl_status / ddl_verified_by / ddl_verified_at`
- `CREATE TABLE`（粘贴或链接）
- 字段说明表（column/type/nullable/default/meaning/example）
- 约束与治理（索引、分区、保留策略、口径负责人）

规则:
1. 未补齐 DDL 的 table 文档必须保持 `draft`。
2. 仅当 DDL 与字段语义完成人工审校后，才允许标记 `verified`。

## 7.1 人工审核标记规范（检索依据）
1. 所有文档审核通过后必须更新 YAML:
   - `status: verified`
   - `verified_by: <reviewer>`
   - `verified_at: YYYY-MM-DD`
   - `review_round: <number>`
   - `review_notes: <summary>`
2. table 文档除以上字段外，还必须更新:
   - `ddl_status: verified`
   - `ddl_verified_by: <reviewer>`
   - `ddl_verified_at: YYYY-MM-DD`
3. 检索默认过滤策略:
   - 文档层: `status=verified`
   - 表层: `ddl_status=verified`

## 8. 证据规范（强制）
- 每条关键结论必须附代码证据:
  - `文件路径 + 行号`（可加 commit）
- 无证据结论不得作为高置信输入。

## 9. 检索友好规范
- 每个知识包必须有:
  - `index-<name>.md`（人读入口）
  - `index-<name>.json`（机器检索入口）
- `index.json` 必须包含:
  - `docs` 分类清单
  - `entrypoints` 路由列表
  - `naming_policy`（service/table 冲突规则）

## 10. 生成流程（建议）
1. 读取 controller
2. 解析直接 service 依赖
3. 解析 mapper + XML
4. 抽取表依赖与数据源
5. 生成完整知识包初稿（全部 `draft`）
6. 人工补齐 table DDL
7. 人工审校并将通过文档标记为 `verified`

## 11. 验收标准（Definition of Done）
1. 单 controller 的 8 类交付物齐全。
2. 所有文档具备 YAML 头与证据清单。
3. table 文档全部带 DDL 区块。
4. 命名冲突规则在 index 中明确。
5. 至少完成一次人工审校记录（`verified_by` / `verified_at`）。

## 12. 模板建议（执行层）
- 新需求/新 controller 落地时，优先复制最近一个已验收知识包作为模板。
- 不建议从零写文档，建议“复制模板 + 证据替换 + DDL补齐”。
