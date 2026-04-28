---
name: teambition
description: >
  Teambition 项目管理。用于一切与 Teambition 任务和项目相关的增删改查操作，通过 CLI 工具与 Teambition API 交互。
metadata:
  version: v12.220-0.0.1
---

## 前置条件

使用前必须设置以下环境变量：

```bash
export TEAMBITION_MCP_HOST="https://open.teambition.com/api"
export TEAMBITION_MCP_TOKEN=""
```
访问 [Teambition UserToken 自助申请](https://open.teambition.com/user-mcp) 获取 `TEAMBITION_MCP_TOKEN`
如果未设置，所有 CLI 命令将无法正常工作。

---

## CLI 工具安装

本 skill 通过 `teambition_cli` 命令行工具操作 Teambition 数据。首次使用前需要安装 CLI 工具。

### 安装方式

**Linux / macOS**：

```bash
curl -fsSL https://file.teambition.net/public/teambition-cli/install.sh | bash
```

**Windows (PowerShell)**：

```powershell
Invoke-WebRequest -Uri "https://file.teambition.net/public/teambition-cli/install.ps1" -UseBasicParsing -OutFile "install.ps1"
powershell -ExecutionPolicy Bypass -File .\install.ps1
```

安装脚本会自动检测系统架构并下载对应版本。安装完成后可通过 `teambition_cli --help` 验证。

### 常见安装问题

- **macOS 安全提示**（"无法验证开发者"）：执行 `xattr -cr /usr/local/bin/teambition_cli`
- **权限被拒绝**：执行 `chmod +x /usr/local/bin/teambition_cli`
- **命令未找到**：确认安装目录在 PATH 中，或重新打开终端

以下示例统一使用 `teambition` 代替完整命令名。

**全局参数**：

| 参数 | 说明 |
|------|------|
| `--json` | 以 JSON 格式输出原始响应 |
| `-d, --debug` | 启用调试模式，打印完整的 HTTP 请求和响应信息 |

**支持的功能**：
member      企业成员管理，查询当前企业成员信息
project     项目管理，包括查询项目列表、创建项目、查询项目内的任务、查询项目内的标签等
task        任务管理
user        用户管理，返回当前用户信息
---

### TQL 快速参考
查询任务和项目时，可以使用 TQL 语法查询
> 完整 TQL 语法（字段、运算符、时间函数）→ `references/tql.md`

### 项目 TQL 常用场景

> 完整项目 TQL → `references/project-tql.md`

---

## 分页查询
查询任务和项目时，可以使用分页参数设定分页大小和分页 token 获取全部数据
`teambition task query` 和 `teambition project query` 均支持分页：

| 参数 | 说明 |
|------|------|
| `--page-size <N>` | 每页记录数（默认由 API 决定） |
| `--page-token <T>` | 传入上次返回的 `nextPageToken` 获取下一页，只有nextPageToken 为空才表示已查询完所有数据，nextPageToken不变不代表已查询完所有数据 |

```bash
# 第一页
teambition task query --tql "executorId = me()" --page-size 50

# 下一页（使用上次输出中的 nextPageToken）
teambition task query --tql "executorId = me()" --page-size 50 --page-token "上次返回的TOKEN"
```
## 核心规则

1. **环境变量检查**：执行任何命令前，确认 `TEAMBITION_MCP_HOST` 和 `TEAMBITION_MCP_TOKEN` 已设置。若未设置，提示用户先配置环境变量
2. **任务的 content = 标题**：API 返回的 `content` 字段就是任务的**标题**，向用户展示时统一使用"标题"而非"内容"，避免混淆
3. **空字段隐藏**：展示任务或项目信息时，**值为空、null、0、false 或空数组的字段应当隐藏，不向用户展示**，只展示有实际值的字段
4. **批量任务表格输出**：当查询返回**多个任务**（2个及以上）时，**默认使用表格形式展示**，表格列应包含：标题、状态、优先级、执行人、截止时间。单行任务仍使用文本段落形式展示
5. **破坏性操作需确认**：执行移动任务、替换参与者/标签等不可逆或影响较大的操作前，**必须先向用户确认**
6. **ID 链接渲染**：当回复中涉及任务 ID 或项目 ID 时，必须将其渲染为可点击的链接：
    - 任务链接：`https://www.teambition.com/task/{taskId}`
    - 项目链接：`https://www.teambition.com/project/{projectId}`
7. **项目智能推断**：创建任务时，若用户未指定项目：
    1. 执行 `teambition project query --my` 查询用户参与的项目
    2. 若只有 **1 个**项目，自动使用该项目，并告知用户
    3. 若有 **多个**项目，展示项目列表（名称+ID），询问用户选择哪个项目后再创建
8. **创建项目**：用户请求创建项目时，**默认直接创建**，只有用户明确指定使用某个模板时才用 `--template-id` 从模板创建
9. **ID→名称**：API 返回的各类 ID 字段均为原始 ID 字符串，**展示给用户前必须转换为可读名称，禁止直接展示原始 ID**。
   - 操作方法：`teambition task query --enrich-fields all` 补充 ID 对应的名称，多个用逗号分隔。可选值: all(全部), project(项目), tfs(状态), sfc(任务类型), tag(标签), priority(优先级), members(执行人/参与人/创建人), customfields( 自定义字段)
   - 如果无法确定某个 ID 对应的名称，可展示为"未知"或保留该字段不展示，不要直接展示原始 ID
10. **优先级**：数值含义为 `0=紧急 1=高 2=中 3=低`，但**企业通常会自定义优先级名称**，脚本输出的 `priorityLabel` 仅为系统默认值，不代表该企业的真实配置。
   - **展示优先级名称前必须先查询企业配置**， `teambition project priority-query` 获取企业的优先级配置（ID和名称），将任务中的 `priority` 数值转换为对应的名称后再展示给用户
   - **更新优先级前同样需要先查询**，将用户描述的优先级名称与企业配置匹配后，再用对应的 `priority` 数值调用更新接口

