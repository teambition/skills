---
name: teambition-statistics
description: Teambition 企业数据统计专家。当用户询问企业任务数据相关问题（如任务数量、完成率、缺陷统计、需求统计、按执行者/创建者/时间等维度的数据分析）时使用此技能。通过调用 Teambition 开放平台 MCP 工具查询企业级统计数据。
metadata:
  version: v12.220-0.0.1
---

# Teambition 企业数据统计

通过 Teambition 开放平台 MCP 工具查询企业任务统计数据。将用户的自然语言问题拆解为指标、维度和筛选条件，调用对应工具获取数据。

## 前置条件

使用本技能前，必须先配置 Teambition UserToken：

1. 访问 [Teambition UserToken 自助申请](https://open.teambition.com/user-mcp) 完成申请
2. 获取 Token 后，选择以下任一方式配置：
   ```bash
   # 方式 1：设置环境变量
   export TEAMBITION_USER_TOKEN="your_token_here"
   
   # 方式 2：写入配置文件（在 teambition/ 目录下创建）
   echo '{"userToken": "your_token_here"}' > user-token.json
   ```
3. 验证配置：`echo $TEAMBITION_USER_TOKEN` 或检查 `user-token.json` 是否存在且格式正确

> ⚠️ 若遇到 **401 Unauthorized** 错误，说明 Token 无效或过期，请重新申请并配置。

## 背景知识

- 所有数据都是企业的**任务数据**
- **缺陷** = 任务类型分类（sfcsource）是缺陷的任务
- **需求** = 任务类型分类（sfcsource）是需求的任务
- 字段映射关系因企业而异，必须先通过 `OrganizationColumnMap` 获取当前企业的字段定义

## 可用的 MCP 工具

| 工具 | 用途 | 何时调用 |
|------|------|---------|
| `OrganizationColumnMap` | 获取字段映射关系（指标、维度、筛选字段） | **每次查询前必须先调用** |
| `OrganizationFilterSeacherOptions` | 获取筛选条件的可选值 | 当需要构建 filterTql 时调用 |
| `OrganizationTaskStatistics` | 查询统计数据 | 拿到字段映射和筛选值后调用 |

## 执行流程

收到用户的数据问题后，严格按以下步骤执行：

### Step 1：拆解问题

将用户问题拆解为三个要素：

- **指标**（必需）：要统计的数值，如任务数、Story Points、完成时长等
- **维度**（可选）：数据的分组方式，如按执行者、按创建时间、按项目等
- **筛选条件**（可选）：数据的过滤条件，如只看已完成的、只看缺陷类型的

**维度规则**：当维度中有 datetime 类型的维度时，该维度必须作为第一个维度，且最多只能有一个 datetime 类型的维度。

### Step 2：获取字段映射

调用 `OrganizationColumnMap` 获取当前企业的字段定义。返回结果包含：

- **task.sections**：可用的指标列表（section 为字段 column，name 为显示名）
- **task.adimension**：可用的维度列表（column 为字段名，dataType 为类型）
- **task.filter**：可用的筛选字段列表（column 为字段名，dataType 为类型）

根据返回结果，将用户问题中的中文名称映射为对应的 column 值。

### Step 3：获取筛选值（按需）

当需要构建 filterTql 时，调用 `OrganizationFilterSeacherOptions` 获取筛选字段的可选值。

### Step 4：调用统计接口

使用 `OrganizationTaskStatistics` 查询数据。请求参数：

```json
{
  "organizationId": "企业ID（必填）",
  "indicators": [
    { "column": "taskcount", "function": "count" }
  ],
  "dimensions": [
    { "column": "executorId", "dataType": "string" }
  ],
  "filterTql": "isDone = true",
  "orderBys": [
    { "column": "taskcount", "direction": "desc" }
  ],
  "projectIds": ["项目ID"]
}
```

**indicators 参数说明**：
- `column`：指标字段名，从 sections 列表获取
- `function`：聚合函数，支持 `count`、`sum`、`avg`、`max`、`min`

**dimensions 参数说明**：
- `column`：维度字段名，从 adimension 列表获取
- `dataType`：字段类型，支持 `string`、`boolean`、`datetime`

### Step 5：返回结果

将返回的 graphData 数据整理为用户易读的格式呈现。

## filterTql 语法规范

filterTql 是字符串格式的筛选条件，多个条件用 `AND` 连接。

### 语法格式

```
column op (value) AND column = value AND column = null
```

### column 来源

column 必须从 `OrganizationColumnMap` 返回的 filter 列表中获取。

### value 格式规则

| 类型 | 格式 | 示例 |
|------|------|------|
| 字符串（column 不以 Id 结尾） | 单引号包围 | `sfcsource in ('application.bug','application.story')` |
| ID 类型（column 以 Id 结尾） | 不用引号 | `creatorId in (64a403ad26dfc4de152be2f9,61de768e58fd5bab1677a852)` |
| 布尔类型 | true 或 false | `isDone = true` |
| 数值类型 | 数字 | `priority in (1,2,3)` |
| 日期时间类型 | yyyy-MM-ddTHH:mm:ss+08:00 | `dueDate >= 2025-01-01T00:00:00+08:00 AND dueDate <= 2025-01-31T23:59:59+08:00` |
| 空值 | null | `executorId = null` |

**日期时间注意**：时间类型只支持 `>=` 和 `<=` 操作符。

### value 来源

value 需要从 `OrganizationFilterSeacherOptions` 工具获取，不能凭猜测填写。

## 示例

### 示例 1：查询企业总任务数

**用户问题**：企业一共有多少任务？

**拆解**：
- 指标：任务数（taskcount, count）
- 维度：无
- 筛选：无

**调用**：
```json
{
  "organizationId": "xxx",
  "indicators": [{ "column": "taskcount", "function": "count" }]
}
```

### 示例 2：按执行者统计已完成的缺陷数

**用户问题**：每个人完成了多少缺陷？

**拆解**：
- 指标：任务数（taskcount, count）
- 维度：执行者（executorId, string）
- 筛选：已完成（isDone = true）且是缺陷（sfcsource in ('application.bug')）

**调用**：
```json
{
  "organizationId": "xxx",
  "indicators": [{ "column": "taskcount", "function": "count" }],
  "dimensions": [{ "column": "executorId", "dataType": "string" }],
  "filterTql": "isDone = true AND sfcsource in ('application.bug')"
}
```

### 示例 3：按月统计创建的需求数

**用户问题**：2025年每个月创建了多少需求？

**拆解**：
- 指标：任务数（taskcount, count）
- 维度：创建时间（created, datetime）—— datetime 维度必须放第一个
- 筛选：是需求（sfcsource in ('application.story')）且创建时间在2025年

**调用**：
```json
{
  "organizationId": "xxx",
  "indicators": [{ "column": "taskcount", "function": "count" }],
  "dimensions": [{ "column": "created", "dataType": "datetime" }],
  "filterTql": "sfcsource in ('application.story') AND created >= 2025-01-01T00:00:00+08:00 AND created <= 2025-12-31T23:59:59+08:00"
}
```

## 关键约束

1. **必须先调用 OrganizationColumnMap** 获取字段映射，再进行后续操作
2. **filterTql 中的 value 必须从 OrganizationFilterSeacherOptions 获取**，不能凭猜测
3. **必须调用 OrganizationTaskStatistics 获取数据后再返回**，不能只返回中间结果
4. **严格按照工具参数格式调用**，不能有多余或缺少的符号
5. **根据用户问题选择合适的指标、维度和筛选条件**，不能随意添加或删除
6. **确保每次调用参数正确**，无语法错误或逻辑错误

## API 详细参考

如需查看三个 MCP 工具的完整 OpenAPI 定义，参见 [references/api-spec.md](references/api-spec.md)。
