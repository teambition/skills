# Teambition 企业统计 MCP 工具 API 参考

## MCP Server 连接信息

三个工具均为 Teambition 开放平台的 MCP Tool，需要先配置 MCP Server 才能调用。

- **协议**：Streamable HTTP
- **URL**：`https://open.teambition.com/api/mcp?userToken=<TB_USER_TOKEN>`
- **Token 获取**：访问 [Teambition 开放平台](https://open.teambition.com/user-mcp) 获取用户 Token（格式为 `u-xxxx`）

### 标准 MCP 配置

在支持 MCP 的客户端（如 Claude Desktop、Cursor、Aone Copilot 等）中添加以下配置：

```json
{
  "mcpServers": {
    "teambition": {
      "url": "https://open.teambition.com/api/mcp?userToken=<你的TB_USER_TOKEN>"
    }
  }
}
```

将 `<你的TB_USER_TOKEN>` 替换为实际的用户 Token。

配置完成后，确认 `OrganizationColumnMap`、`OrganizationFilterSeacherOptions`、`OrganizationTaskStatistics` 三个工具出现在可用工具列表中。

---

## 工具 1：OrganizationColumnMap

获取企业的字段映射关系，包括可用的指标、维度、筛选字段和排序字段。

- **operationId**: `OrganizationColumnMap`
- **HTTP**: `POST /report/cross/task/providedInfo`
- **API 文档**: https://open.teambition.com/docs/apis/69b3b1f978cfd6c1b86d3fb0

### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| organizationId | string | 是 | 企业 ID |
| projectIds | string[] | 否 | 项目 ID 列表，限定统计范围 |

### 请求 Header

| Header | 必填 | 说明 |
|--------|------|------|
| Authorization | 是 | app_access_token |
| X-Tenant-Id | 是 | 租户 ID |
| X-Tenant-Type | 是 | 租户类型，默认为 organization |
| X-Operator-Id | 是 | 操作者 ID |

### 返回结构

返回 `task` 对象（`task.dataType = "task"`），包含以下子列表：

#### task.sections — 指标列表

每个元素包含：
- `section`（string）：指标字段 column，用于 OrganizationTaskStatistics 的 indicators[].column
- `name`（string）：指标显示名称
- `source`（string | null）：指标来源（如 `worktime-cf` 表示工时自定义字段）
- `layouts`（object | null）：布局信息

常见内置指标：`taskcount`（任务数）、`storypoint`（Story Points）、`completiontime`（完成时长/天）、`staytime`（停留时长/天）、`overduetime`（逾期时长/天）。以 `cf:` 开头的为自定义字段指标。

#### task.adimension — 维度列表

每个元素包含：
- `column`（string）：维度字段名，用于 OrganizationTaskStatistics 的 dimensions[].column
- `name`（string）：维度显示名称
- `dataType`（string）：字段类型
- `subType`（string | null）：子类型

dataType 枚举值：`string` / `boolean` / `datetime` / `cascading` / `parentId` / `text` / `dropDown` / `lookup.member` / `number`

#### task.filter — 筛选字段列表

每个元素包含：
- `column`（string）：筛选字段名，用于构建 filterTql
- `name`（string）：筛选字段显示名称
- `dataType`（string）：字段类型（同 adimension 的 dataType 枚举）
- `component`（string）：UI 组件类型
- `choices`（array | null）：下拉选项列表（仅 dropDown 类型有值），每个选项包含 `_id` 和 `value`
- `instanceId`（string | null）：实例 ID
- `subType`（string）：子类型

component 枚举值：`text` / `dropDown` / `singleRadio` / `crossMember` / `multiText` / `datetime` / `executor` / `team` / `parentTask` / `taskflowstatus` / `taskflow` / `multiPriority` / `number` / `work`

#### task.orderBys — 排序字段列表

每个元素包含：
- `column`（string）：排序字段名
- `name`（string）：排序字段显示名称
- `type`（string | null）：字段类型（boolean / string / datetime / text / dropDown / lookup.member / parentId / null）

#### task.cdimension — 对比维度列表

结构同 adimension，但不包含 `t_taskflowstatusId`（工作流状态按工作流区分）、`accomplished`/`created`/`updated`/`dueDate`/`startDate`/`statusUpdated`（时间类维度）、`executorTeamId`/`creatorTeamId`（部门类维度）。

#### task.userDefinedColumns — 用户自定义字段列表

结构同 filter，包含企业自定义的所有字段定义（以 `cf:` 开头的字段）。注意 userDefinedColumns 可能包含 filter 中没有的字段（如 `component: work` 类型的附件字段）。

---

## 工具 2：OrganizationFilterSeacherOptions

获取筛选条件字段的可选值列表。用于构建 filterTql 时获取 value。

- **operationId**: `OrganizationFilterSeacherOptions`
- **HTTP**: `POST /report/cross/task/searcherOptions`
- **API 文档**: https://open.teambition.com/docs/apis/69b3b32f78cfd6c1b86d4401

### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| organizationId | string | 是 | 企业 ID |
| key | string | 是 | 筛选字段的 column 名称（来自 OrganizationColumnMap 返回的 filter[].column） |
| projectIds | string[] | 否 | 项目 ID 列表，限定统计范围 |

### 请求 Header

| Header | 必填 | 说明 |
|--------|------|------|
| Authorization | 是 | app_access_token |
| X-Tenant-Id | 是 | 租户 ID |
| X-Tenant-Type | 是 | 租户类型，默认为 organization |
| X-Operator-Id | 是 | 操作者 ID |

### 返回结构

返回 JSON 数组，每个元素包含：
- `_id`（string）：选项唯一标识
- `name`（string）：选项显示名称
- `value`（string | number）：选项值（用于 filterTql）
- `color`（string）：选项颜色标识（如 `red` / `orange` / `blue` / `grey` / `green`，部分字段可能无此属性）

示例（key = `priority` 时的返回）：

```json
[
  { "_id": "654da5c9...", "name": "P1", "value": 7, "color": "red" },
  { "_id": "5e870bc3...", "name": "High", "value": 3, "color": "orange" },
  { "_id": "5e870bc3...", "name": "Medium", "value": 2, "color": "blue" },
  { "_id": "5e870bc3...", "name": "Low", "value": 1, "color": "grey" }
]
```

---

## 工具 3：OrganizationTaskStatistics

查询企业统计数据。

### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| organizationId | string | 是 | 企业 ID |
| indicators | object[] | 是 | 指标列表 |
| indicators[].column | string | 是 | 指标字段名（从 sections 获取） |
| indicators[].function | string | 是 | 聚合函数：count / sum / avg / max / min |
| dimensions | object[] | 否 | 维度列表 |
| dimensions[].column | string | 是 | 维度字段名（从 adimension 获取） |
| dimensions[].dataType | string | 是 | 字段类型：string / boolean / datetime |
| filterTql | string | 否 | 筛选条件表达式 |
| orderBys | object[] | 否 | 排序字段列表 |
| orderBys[].column | string | 是 | 排序字段名 |
| orderBys[].direction | string | 是 | 排序方向：desc / asc |
| projectIds | string[] | 否 | 项目 ID 列表 |

### 返回结构

```json
{
  "graphData": {
    "cols": [
      { "name": "创建者", "baseType": "type/Task", "column": "creatorId" },
      { "name": "任务数 (计数)", "baseType": "type/Task", "column": "taskcount" }
    ],
    "rows": [
      ["游凡", "67"],
      ["张三", "42"]
    ],
    "ids": [...]
  },
  "chartType": "bar",
  "filterTreeNode": { ... },
  "dimensions": [ ... ],
  "selectedSections": [ ... ]
}
```

### filterTql 语法

格式：`column op (value) AND column = value`

**操作符**：`in`、`=`、`>=`、`<=`

**value 格式**：

| 场景 | 格式 | 示例 |
|------|------|------|
| 字符串值 | 单引号包围 | `sfcsource in ('application.bug')` |
| ID 值（column 以 Id 结尾） | 不用引号 | `creatorId in (64a403ad,61de768e)` |
| 布尔值 | true / false | `isDone = true` |
| 数值 | 数字 | `priority in (1,2,3)` |
| 日期时间 | yyyy-MM-ddTHH:mm:ss+08:00 | `created >= 2025-01-01T00:00:00+08:00` |
| 空值 | null | `executorId = null` |

**日期时间**只支持 `>=` 和 `<=` 操作符。
