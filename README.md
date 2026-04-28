# Teambition Skills 仓库

本仓库包含了 Teambition 相关的所有 AI 助手技能（Skills），用于增强 AI 助手在项目管理、数据统计等场景下的能力。

## 📦 包含的 Skills

### 1. teambition - 项目管理助手

**用途**: Teambition 项目管理的全能助手，支持任务和项目的增删改查操作。

**核心功能**:
- 📋 **项目管理**: 查询项目列表、创建项目、查询项目内任务和标签
- ✅ **任务管理**: 创建、查询、更新、删除任务
- 👥 **成员管理**: 查询企业成员信息
- 👤 **用户管理**: 获取当前用户信息
- 🔍 **TQL 查询**: 支持强大的 TQL 语法进行任务和项目查询

**使用方式**: 通过 CLI 工具 `teambition_cli` 与 Teambition API 交互

**文档**: [`skills/teambition/SKILL.md`](skills/teambition/SKILL.md)

---

### 2. teambition-statistics - 企业数据统计专家

**用途**: 企业级任务数据统计分析，回答各类数据相关问题。

**核心功能**:
- 📊 **多维度统计**: 按执行者、创建者、时间等维度分析任务数据
- 🐛 **缺陷统计**: 统计缺陷数量、完成率等
- 📝 **需求统计**: 统计需求数量、完成情况等
- 📈 **灵活查询**: 支持自定义指标、维度和筛选条件

**使用方式**: 通过 Teambition 开放平台 MCP 工具查询统计数据

**文档**: [`skills/teambition-statistics/SKILL.md`](skills/teambition-statistics/SKILL.md)

---

## 🚀 快速开始

### 前置条件

使用任何 Skill 前，需要先配置 Teambition UserToken：

1. 访问 [Teambition UserToken 自助申请](https://open.teambition.com/user-mcp) 完成申请
2. 配置环境变量：
   ```bash
   # teambition skill 使用
   export TEAMBITION_MCP_HOST="https://open.teambition.com/api"
   export TEAMBITION_MCP_TOKEN="your_token_here"
   
   # teambition-statistics skill 使用
   export TEAMBITION_USER_TOKEN="your_token_here"
   ```

### 安装 CLI 工具（teambition skill）

**Linux / macOS**:
```bash
curl -fsSL https://file.teambition.net/public/teambition-cli/install.sh | bash
```

**Windows (PowerShell)**:
```powershell
Invoke-WebRequest -Uri "https://file.teambition.net/public/teambition-cli/install.ps1" -UseBasicParsing -OutFile "install.ps1"
powershell -ExecutionPolicy Bypass -File .\install.ps1
```

## 📁 仓库结构

```
.
├── README.md                           # 本文档
└── skills/
    ├── teambition/                     # 项目管理 Skill
    │   ├── SKILL.md                    # Skill 主文档
    │   └── references/
    │       ├── tql.md                  # TQL 语法参考
    │       └── project-tql.md          # 项目 TQL 参考
    └── teambition-statistics/          # 数据统计 Skill
        ├── SKILL.md                    # Skill 主文档
        └── references/
            └── api-spec.md             # API 规范文档
```

## 🔗 相关链接

- [Teambition 官网](https://www.teambition.com/)
- [Teambition 开放平台](https://open.teambition.com/)
- [Teambition UserToken 申请](https://open.teambition.com/user-mcp)

## 📝 版本信息

当前版本: `v12.220-0.0.1`

## 📄 许可证

内部使用

---

> 💡 **提示**: 如需了解每个 Skill 的详细使用方法，请查看对应的 `SKILL.md` 文档。
