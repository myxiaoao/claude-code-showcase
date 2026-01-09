# Claude Code 项目配置展示

> 大多数软件工程师严重低估了当前 LLM Agent 的能力，尤其是像 Claude Code 这样的工具。

一旦配置好 Claude Code，你可以将它指向你的代码库，让它学习你的代码规范，引入最佳实践，并不断优化，直到它基本上像一个超级强力的队友一样运作。**真正的突破在于构建一套可复用的"[技能](#技能---领域知识)"以及一些用于日常工作的"[Agent](#agent---专业助手)"。**

### 实际应用场景

**自定义 UI 库？** 我们有一个[技能文档详细解释如何使用它](.claude/skills/core-components/SKILL.md)。[如何编写测试](.claude/skills/testing-patterns/SKILL.md)、[如何组织 GraphQL](.claude/skills/graphql-schema/SKILL.md)，以及我们在仓库中希望的所有实现方式都有对应的技能文档。因此，当 Claude 生成代码时，它已经符合我们的模式和标准。

**自动化质量门禁？** 我们使用 [hooks](.claude/settings.json) 来自动格式化代码、在测试文件更改时运行测试、检查 TypeScript 类型，甚至[阻止在主分支上的编辑](.claude/settings.md)。Claude Code 还创建了大量 ESLint 自动化，包括自定义规则和 lint 检查，在代码进入审查之前就捕获问题。

**深度代码审查？** 我们有一个[代码审查 Agent](.claude/agents/code-reviewer.md)，Claude 在代码更改后运行它。它遵循详细的检查清单，涵盖 TypeScript 严格模式、错误处理、加载状态、数据变更模式等。当 PR 提交时，我们有一个 [GitHub Action](.github/workflows/pr-claude-code-review.yml) 自动进行完整的 PR 审查。

**定时维护？** 我们有按计划运行的 GitHub workflow Agent：
- [每月文档同步](.github/workflows/scheduled-claude-code-docs-sync.yml) - 读取上个月的提交，确保文档仍然一致
- [每周代码质量检查](.github/workflows/scheduled-claude-code-quality.yml) - 审查随机目录并自动修复问题
- [双周依赖审计](.github/workflows/scheduled-claude-code-dependency-audit.yml) - 安全的依赖更新并验证测试

**智能技能建议？** 我们构建了一个[技能评估系统](#技能评估-hooks)，它分析每个提示并根据关键词、文件路径和意图模式自动建议 Claude 应该激活哪些技能。

大量的维护和质量工作就这样...自动化了。运行得非常流畅。

**JIRA/Linear 集成？** 我们通过 [MCP 服务器](.mcp.json)将 Claude Code 连接到我们的工单系统。现在 Claude 可以读取工单、理解需求、实现功能、更新工单状态，甚至在发现 bug 时创建新工单。[`/ticket` 命令](.claude/commands/ticket.md)处理整个工作流程——从读取验收标准到将 PR 链接回工单。

我们甚至使用 Claude Code 进行工单分类。它读取工单，深入代码库，并留下评论说明它认为应该做什么。因此，当工程师接手时，他们基本上已经完成了一半的工作。

**这里有太多唾手可得的成果，老实说，让我惊讶的是人们还没有全面采用它。**

---

## 目录

- [目录结构](#目录结构)
- [快速开始](#快速开始)
- [配置参考](#配置参考)
  - [CLAUDE.md - 项目记忆](#claudemd---项目记忆)
  - [settings.json - Hooks 与环境](#settingsjson---hooks-与环境)
  - [MCP 服务器 - 外部集成](#mcp-服务器---外部集成)
  - [LSP 服务器 - 实时代码智能](#lsp-服务器---实时代码智能)
  - [技能评估 Hooks](#技能评估-hooks)
  - [技能 - 领域知识](#技能---领域知识)
  - [Agent - 专业助手](#agent---专业助手)
  - [命令 - 斜杠命令](#命令---斜杠命令)
- [GitHub Actions 工作流](#github-actions-工作流)
- [最佳实践](#最佳实践)
- [本仓库示例](#本仓库示例)

---

## 目录结构

```
your-project/
├── CLAUDE.md                      # 项目记忆（备选位置）
├── .mcp.json                      # MCP 服务器配置（JIRA、GitHub 等）
├── .claude/
│   ├── settings.json              # Hooks、环境、权限
│   ├── settings.local.json        # 个人覆盖配置（gitignored）
│   ├── settings.md                # 人类可读的 hook 文档
│   ├── .gitignore                 # 忽略本地/个人文件
│   │
│   ├── agents/                    # 自定义 AI Agent
│   │   └── code-reviewer.md       # 主动代码审查 Agent
│   │
│   ├── commands/                  # 斜杠命令（/command-name）
│   │   ├── onboard.md             # 深度任务探索
│   │   ├── pr-review.md           # PR 审查工作流
│   │   └── ...
│   │
│   ├── hooks/                     # Hook 脚本
│   │   ├── skill-eval.sh          # 提示提交时的技能匹配
│   │   ├── skill-eval.js          # Node.js 技能匹配引擎
│   │   └── skill-rules.json       # 模式匹配配置
│   │
│   ├── skills/                    # 领域知识文档
│   │   ├── README.md              # 技能概述
│   │   ├── testing-patterns/
│   │   │   └── SKILL.md
│   │   ├── graphql-schema/
│   │   │   └── SKILL.md
│   │   └── ...
│   │
│   └── rules/                     # 模块化指令（可选）
│       ├── code-style.md
│       └── security.md
│
└── .github/
    └── workflows/
        ├── pr-claude-code-review.yml           # 自动 PR 审查
        ├── scheduled-claude-code-docs-sync.yml # 每月文档同步
        ├── scheduled-claude-code-quality.yml   # 每周质量审查
        └── scheduled-claude-code-dependency-audit.yml
```

---

## 快速开始

### 1. 创建 `.claude` 目录

```bash
mkdir -p .claude/{agents,commands,hooks,skills}
```

### 2. 添加 CLAUDE.md 文件

在项目根目录创建 `CLAUDE.md`，包含项目的关键信息。完整示例请参见 [CLAUDE.md](CLAUDE.md)。

```markdown
# 项目名称

## 快速信息
- **技术栈**: React, TypeScript, Node.js
- **测试命令**: `npm run test`
- **Lint 命令**: `npm run lint`

## 关键目录
- `src/components/` - React 组件
- `src/api/` - API 层
- `tests/` - 测试文件

## 代码风格
- TypeScript 严格模式
- 优先使用 interface 而非 type
- 禁止使用 `any` - 使用 `unknown`
```

### 3. 添加带 hooks 的 settings.json

创建 `.claude/settings.json`。完整示例请参见 [settings.json](.claude/settings.json)，包含自动格式化、测试等功能。

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "[ \"$(git branch --show-current)\" != \"main\" ] || { echo '{\"block\": true, \"message\": \"Cannot edit on main branch\"}' >&2; exit 2; }",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

### 4. 添加你的第一个技能

创建 `.claude/skills/testing-patterns/SKILL.md`。完整示例请参见 [testing-patterns/SKILL.md](.claude/skills/testing-patterns/SKILL.md)。

```markdown
---
name: testing-patterns
description: 本项目的 Jest 测试模式。在编写测试、创建 mock 或遵循 TDD 工作流时使用。
---

# 测试模式

## 测试结构
- 使用 `describe` 块进行分组
- 使用 `it` 进行单个测试
- 遵循 AAA 模式：Arrange、Act、Assert

## Mock
- 使用工厂函数：`getMockUser(overrides)`
- Mock 外部依赖，而非内部模块
```

> **提示：** `description` 字段至关重要——Claude 使用它来决定何时应用该技能。包含用户自然会提到的关键词。

---

## 配置参考

### CLAUDE.md - 项目记忆

CLAUDE.md 是 Claude 的持久记忆，在会话开始时自动加载。

**位置（按优先级顺序）：**
1. `.claude/CLAUDE.md`（项目，在 .claude 文件夹中）
2. `./CLAUDE.md`（项目根目录）
3. `~/.claude/CLAUDE.md`（用户级别，所有项目）

**应包含的内容：**
- 项目技术栈和架构概述
- 关键命令（测试、构建、lint、部署）
- 代码风格指南
- 重要目录及其用途
- 关键规则和约束

**📄 示例：** [CLAUDE.md](CLAUDE.md)

---

### settings.json - Hooks 与环境

hooks、环境变量和权限的主配置文件。

**位置：** `.claude/settings.json`

**📄 示例：** [settings.json](.claude/settings.json) | [人类可读文档](.claude/settings.md)

#### Hook 事件

| 事件 | 触发时机 | 用例 |
|------|----------|------|
| `PreToolUse` | 工具执行前 | 阻止在 main 分支编辑、验证命令 |
| `PostToolUse` | 工具完成后 | 自动格式化、运行测试、lint |
| `UserPromptSubmit` | 用户提交提示时 | 添加上下文、建议技能 |
| `Stop` | Agent 完成时 | 决定 Claude 是否应继续 |

#### Hook 响应格式

```json
{
  "block": true,           // 阻止操作（仅 PreToolUse）
  "message": "原因",       // 显示给用户的消息
  "feedback": "信息",      // 非阻止性反馈
  "suppressOutput": true,  // 隐藏命令输出
  "continue": false        // 是否继续
}
```

#### 退出码
- `0` - 成功
- `2` - 阻止性错误（仅 PreToolUse，阻止工具）
- 其他 - 非阻止性错误

---

### MCP 服务器 - 外部集成

MCP（Model Context Protocol，模型上下文协议）服务器让 Claude Code 可以连接到外部工具，如 JIRA、GitHub、Slack、数据库等。这是启用"读取工单、实现功能、更新工单状态"等工作流的方式。

**位置：** `.mcp.json`（项目根目录，提交到 git 以便团队共享）

**📄 示例：** [.mcp.json](.mcp.json)

#### MCP 工作原理

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Claude Code   │────▶│   MCP 服务器    │────▶│    外部 API     │
│                 │◀────│  （本地桥接）   │◀────│ （JIRA、GitHub）│
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

MCP 服务器在本地运行，为 Claude 提供与外部服务交互的工具。当你配置 JIRA MCP 服务器时，Claude 会获得 `jira_get_issue`、`jira_update_issue`、`jira_create_issue` 等工具。

#### .mcp.json 格式

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-name"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

**字段：**

| 字段 | 必需 | 描述 |
|------|------|------|
| `type` | 是 | 服务器类型：`stdio`（本地进程）或 `http`（远程） |
| `command` | stdio 需要 | 要运行的可执行文件（如 `npx`、`python`） |
| `args` | 否 | 命令行参数 |
| `env` | 否 | 环境变量（支持 `${VAR}` 展开） |
| `url` | http 需要 | 远程服务器 URL |
| `headers` | http 需要 | 用于认证的 HTTP 头 |

#### 示例：JIRA 集成

```json
{
  "mcpServers": {
    "jira": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-jira"],
      "env": {
        "JIRA_HOST": "${JIRA_HOST}",
        "JIRA_EMAIL": "${JIRA_EMAIL}",
        "JIRA_API_TOKEN": "${JIRA_API_TOKEN}"
      }
    }
  }
}
```

**这能实现什么：**
- 读取工单详情、验收标准和评论
- 更新工单状态（待办 → 进行中 → 审查中）
- 添加进度更新评论
- 为开发过程中发现的 bug 创建新工单
- 将 PR 链接到工单

**使用 [`/ticket` 命令](.claude/commands/ticket.md) 的示例工作流：**
```
你：/ticket PROJ-123

Claude：
1. 从 JIRA 获取 PROJ-123...
   "添加用户头像上传功能"

2. 读取验收标准...
   - 个人资料页面上的上传按钮
   - 支持 JPG/PNG，最大 5MB
   - 显示加载状态

3. 搜索代码库中的相关文件...
   找到：src/screens/Profile/ProfileScreen.tsx

4. 创建分支：cw/PROJ-123-avatar-upload

5. [实现功能...]

6. 更新 JIRA 状态为"审查中"
   添加评论："PR #456 已准备好审查"

7. 创建链接到 PROJ-123 的 PR...
```

#### 常见 MCP 服务器配置

**问题跟踪：**
```json
{
  "jira": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-jira"],
    "env": {
      "JIRA_HOST": "${JIRA_HOST}",
      "JIRA_EMAIL": "${JIRA_EMAIL}",
      "JIRA_API_TOKEN": "${JIRA_API_TOKEN}"
    }
  },
  "linear": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-linear"],
    "env": { "LINEAR_API_KEY": "${LINEAR_API_KEY}" }
  }
}
```

**代码与 DevOps：**
```json
{
  "github": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-github"],
    "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
  },
  "sentry": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-sentry"],
    "env": {
      "SENTRY_AUTH_TOKEN": "${SENTRY_AUTH_TOKEN}",
      "SENTRY_ORG": "${SENTRY_ORG}"
    }
  }
}
```

**通信：**
```json
{
  "slack": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-slack"],
    "env": {
      "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
      "SLACK_TEAM_ID": "${SLACK_TEAM_ID}"
    }
  }
}
```

**数据库：**
```json
{
  "postgres": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-postgres"],
    "env": { "DATABASE_URL": "${DATABASE_URL}" }
  }
}
```

#### 环境变量

MCP 配置支持变量展开：
- `${VAR}` - 展开为环境变量（如未设置则失败）
- `${VAR:-default}` - 如果 VAR 未设置则使用默认值

在你的 shell 配置文件或 `.env` 文件中设置这些（不要提交密钥！）：
```bash
export JIRA_HOST="https://yourcompany.atlassian.net"
export JIRA_EMAIL="you@company.com"
export JIRA_API_TOKEN="your-api-token"
```

#### MCP 设置

在 `settings.json` 中，你可以自动批准 MCP 服务器：

```json
{
  "enableAllProjectMcpServers": true
}
```

或批准特定服务器：
```json
{
  "enabledMcpjsonServers": ["jira", "github", "slack"]
}
```

---

### LSP 服务器 - 实时代码智能

LSP（Language Server Protocol，语言服务器协议）为 Claude 提供对代码的实时理解——类型信息、错误、补全和导航。Claude 不再只是阅读文本，而是能像你的 IDE 一样"看到"你的代码。

**为什么这很重要：** 当你编辑 TypeScript 时，Claude 会立即知道你是否引入了类型错误。当你引用一个函数时，Claude 可以跳转到它的定义。这大大提高了代码生成质量。

#### 启用 LSP

通过 `settings.json` 中的插件启用 LSP 支持：

```json
{
  "enabledPlugins": {
    "typescript-lsp@claude-plugins-official": true,
    "pyright-lsp@claude-plugins-official": true
  }
}
```

#### Claude 从 LSP 获得什么

| 功能 | 描述 |
|------|------|
| **诊断** | 每次编辑后的实时错误和警告 |
| **类型信息** | 悬停信息、函数签名、类型定义 |
| **代码导航** | 跳转到定义、查找引用 |
| **补全** | 上下文感知的符号建议 |

#### 可用的 LSP 插件

| 插件 | 语言 | 首先安装二进制文件 |
|------|------|-------------------|
| `typescript-lsp` | TypeScript/JavaScript | `npm install -g typescript-language-server typescript` |
| `pyright-lsp` | Python | `pip install pyright` |
| `rust-lsp` | Rust | `rustup component add rust-analyzer` |

#### 自定义 LSP 配置

对于高级设置，创建 `.lsp.json`：

```json
{
  "typescript": {
    "command": "typescript-language-server",
    "args": ["--stdio"],
    "extensionToLanguage": {
      ".ts": "typescript",
      ".tsx": "typescriptreact"
    },
    "initializationOptions": {
      "preferences": {
        "quotePreference": "single"
      }
    }
  }
}
```

#### 故障排除

如果 LSP 不工作：

1. **检查二进制文件是否已安装：**
   ```bash
   which typescript-language-server  # 应返回路径
   ```

2. **启用调试日志：**
   ```bash
   claude --enable-lsp-logging
   ```

3. **检查插件状态：**
   ```bash
   claude /plugin  # 查看错误标签
   ```

---

### 技能评估 Hooks

我们最强大的自动化之一是**技能评估系统**。它在每次提示提交时运行，智能建议 Claude 应该激活哪些技能。

**📄 文件：** [skill-eval.sh](.claude/hooks/skill-eval.sh) | [skill-eval.js](.claude/hooks/skill-eval.js) | [skill-rules.json](.claude/hooks/skill-rules.json)

#### 工作原理

当你提交提示时，`UserPromptSubmit` hook 触发我们的技能评估引擎：

1. **提示分析** - 引擎分析你的提示：
   - **关键词**：简单词匹配（`test`、`form`、`graphql`、`bug`）
   - **模式**：正则匹配（`\btest(?:s|ing)?\b`、`\.stories\.`）
   - **文件路径**：提取提到的文件（`src/components/Button.tsx`）
   - **意图**：检测你想做什么（`create.*test`、`fix.*bug`）

2. **目录映射** - 文件路径被映射到相关技能：
   ```json
   {
     "src/components/core": "core-components",
     "src/graphql": "graphql-schema",
     ".github/workflows": "github-actions",
     "src/hooks": "react-ui-patterns"
   }
   ```

3. **置信度评分** - 每种触发类型都有分值：
   ```json
   {
     "keyword": 2,
     "keywordPattern": 3,
     "pathPattern": 4,
     "directoryMatch": 5,
     "intentPattern": 4
   }
   ```

4. **技能建议** - 超过置信度阈值的技能会附带原因被建议：
   ```
   需要激活技能

   检测到的文件路径：src/components/UserForm.tsx

   匹配的技能（按相关性排名）：
   1. formik-patterns（高置信度）
      匹配：关键词 "form"，路径 "src/components/UserForm.tsx"
   2. react-ui-patterns（中等置信度）
      匹配：目录映射，关键词 "component"
   ```

#### 配置

技能在 [skill-rules.json](.claude/hooks/skill-rules.json) 中定义：

```json
{
  "testing-patterns": {
    "description": "Jest 测试模式和 TDD 工作流",
    "priority": 9,
    "triggers": {
      "keywords": ["test", "jest", "spec", "tdd", "mock"],
      "keywordPatterns": ["\\btest(?:s|ing)?\\b", "\\bspec\\b"],
      "pathPatterns": ["**/*.test.ts", "**/*.test.tsx"],
      "intentPatterns": [
        "(?:write|add|create|fix).*(?:test|spec)",
        "(?:test|spec).*(?:for|of|the)"
      ]
    },
    "excludePatterns": ["e2e", "maestro", "end-to-end"]
  }
}
```

#### 添加到你的项目

1. 将 hooks 复制到你的项目：
   ```bash
   cp -r .claude/hooks/ your-project/.claude/hooks/
   ```

2. 将 hook 添加到你的 `settings.json`：
   ```json
   {
     "hooks": {
       "UserPromptSubmit": [
         {
           "hooks": [
             {
               "type": "command",
               "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/skill-eval.sh",
               "timeout": 5
             }
           ]
         }
       ]
     }
   }
   ```

3. 用你项目的技能和触发器自定义 [skill-rules.json](.claude/hooks/skill-rules.json)。

---

### 技能 - 领域知识

技能是教会 Claude 项目特定模式和约定的 markdown 文档。

**位置：** `.claude/skills/{skill-name}/SKILL.md`

**📄 示例：**
- [testing-patterns](.claude/skills/testing-patterns/SKILL.md) - TDD、工厂函数、mock
- [systematic-debugging](.claude/skills/systematic-debugging/SKILL.md) - 四阶段调试方法
- [react-ui-patterns](.claude/skills/react-ui-patterns/SKILL.md) - 加载状态、错误处理
- [graphql-schema](.claude/skills/graphql-schema/SKILL.md) - 查询、变更、代码生成
- [core-components](.claude/skills/core-components/SKILL.md) - 设计系统、token
- [formik-patterns](.claude/skills/formik-patterns/SKILL.md) - 表单处理、验证

#### SKILL.md 前置字段

| 字段 | 必需 | 最大长度 | 描述 |
|------|------|----------|------|
| `name` | **是** | 64 字符 | 仅小写字母、数字和连字符。应与目录名匹配。 |
| `description` | **是** | 1024 字符 | 技能的作用和使用时机。Claude 使用此字段决定何时应用技能。 |
| `allowed-tools` | 否 | - | Claude 可以使用的工具的逗号分隔列表（如 `Read, Grep, Bash(npm:*)`）。 |
| `model` | 否 | - | 要使用的特定模型（如 `claude-sonnet-4-20250514`）。 |

#### SKILL.md 格式

```markdown
---
name: skill-name
description: 此技能的作用和使用时机。包含用户会提到的关键词。
allowed-tools: Read, Grep, Glob
model: claude-sonnet-4-20250514
---

# 技能标题

## 何时使用
- 触发条件 1
- 触发条件 2

## 核心模式

### 模式名称
```typescript
// 示例代码
```

## 反模式

### 不要做什么
```typescript
// 错误示例
```

## 集成
- 相关技能：`other-skill`
```

#### 技能最佳实践

1. **保持 SKILL.md 专注** - 500 行以内；将详细文档放在单独的引用文件中
2. **编写触发丰富的描述** - Claude 使用描述的语义匹配来决定何时应用技能
3. **包含示例** - 用代码展示好的和坏的模式
4. **引用其他技能** - 展示技能如何协同工作
5. **使用准确的文件名** - 必须是 `SKILL.md`（区分大小写）

---

### Agent - 专业助手

Agent 是具有特定目的和自己提示的 AI 助手。

**位置：** `.claude/agents/{agent-name}.md`

**📄 示例：**
- [code-reviewer.md](.claude/agents/code-reviewer.md) - 带检查清单的综合代码审查
- [github-workflow.md](.claude/agents/github-workflow.md) - Git 提交、分支、PR

#### Agent 格式

```markdown
---
name: code-reviewer
description: 审查代码的质量、安全性和约定。在编写或修改代码后使用。
model: opus
---

# Agent 系统提示

你是一位高级代码审查员...

## 你的流程
1. 运行 `git diff` 查看更改
2. 应用审查检查清单
3. 提供反馈

## 检查清单
- [ ] 无 TypeScript `any`
- [ ] 存在错误处理
- [ ] 包含测试
```

#### Agent 配置字段

| 字段 | 必需 | 描述 |
|------|------|------|
| `name` | 是 | 小写加连字符 |
| `description` | 是 | 何时/为何使用（最多 1024 字符） |
| `model` | 否 | `sonnet`、`opus` 或 `haiku` |
| `tools` | 否 | 逗号分隔的工具列表 |

---

### 命令 - 斜杠命令

使用 `/command-name` 调用的自定义命令。

**位置：** `.claude/commands/{command-name}.md`

**📄 示例：**
- [onboard.md](.claude/commands/onboard.md) - 深度任务探索
- [pr-review.md](.claude/commands/pr-review.md) - PR 审查工作流
- [pr-summary.md](.claude/commands/pr-summary.md) - 生成 PR 描述
- [code-quality.md](.claude/commands/code-quality.md) - 质量检查
- [docs-sync.md](.claude/commands/docs-sync.md) - 文档对齐

#### 命令格式

```markdown
---
description: 命令列表中显示的简要描述
allowed-tools: Bash(git:*), Read, Grep
---

# 命令指令

你的任务是：$ARGUMENTS

## 步骤
1. 首先做这个
2. 然后做这个
```

#### 变量

- `$ARGUMENTS` - 所有参数作为单个字符串
- `$1`、`$2`、`$3` - 各个位置参数

#### 内联 Bash

```markdown
当前分支：!`git branch --show-current`
最近提交：!`git log --oneline -5`
```

---

## GitHub Actions 工作流

使用 Claude Code 自动化代码审查、质量检查和维护。

**📄 示例：**
- [pr-claude-code-review.yml](.github/workflows/pr-claude-code-review.yml) - 自动 PR 审查
- [scheduled-claude-code-docs-sync.yml](.github/workflows/scheduled-claude-code-docs-sync.yml) - 每月文档同步
- [scheduled-claude-code-quality.yml](.github/workflows/scheduled-claude-code-quality.yml) - 每周质量审查
- [scheduled-claude-code-dependency-audit.yml](.github/workflows/scheduled-claude-code-dependency-audit.yml) - 双周依赖更新

### PR 代码审查

自动审查 PR 并响应 `@claude` 提及。

```yaml
name: PR - Claude Code Review
on:
  pull_request:
    types: [opened, synchronize, reopened]
  issue_comment:
    types: [created]

jobs:
  review:
    if: |
      github.event_name == 'pull_request' ||
      (github.event_name == 'issue_comment' &&
       github.event.issue.pull_request &&
       contains(github.event.comment.body, '@claude'))
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@beta
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: claude-opus-4-5-20251101
          prompt: |
            使用 .claude/agents/code-reviewer.md 标准审查此 PR。
            运行 `git diff origin/main...HEAD` 查看更改。
```

### 定时工作流

| 工作流 | 计划 | 目的 |
|--------|------|------|
| [代码质量](.github/workflows/scheduled-claude-code-quality.yml) | 每周（周日） | 审查随机目录，自动修复问题 |
| [文档同步](.github/workflows/scheduled-claude-code-docs-sync.yml) | 每月（1日） | 确保文档与代码更改一致 |
| [依赖审计](.github/workflows/scheduled-claude-code-dependency-audit.yml) | 双周（1日和15日） | 安全的依赖更新并测试 |

### 所需设置

将 `ANTHROPIC_API_KEY` 添加到你的仓库密钥：
- Settings → Secrets and variables → Actions → New repository secret

### 成本估算

| 工作流 | 频率 | 估算成本 |
|--------|------|----------|
| PR 审查 | 每个 PR | ~$0.05 - $0.50 |
| 文档同步 | 每月 | ~$0.50 - $2.00 |
| 依赖审计 | 双周 | ~$0.20 - $1.00 |
| 代码质量 | 每周 | ~$1.00 - $5.00 |

**估算每月总计：** ~$10 - $50（取决于 PR 数量）

---

## 最佳实践

### 1. 从 CLAUDE.md 开始

你的 `CLAUDE.md` 是基础。包含：
- 技术栈概述
- 关键命令
- 关键规则
- 目录结构

### 2. 逐步构建技能

不要试图一次性记录所有内容：
1. 从最常见的模式开始
2. 随着痛点出现添加技能
3. 每个技能专注于一个领域

### 3. 使用 Hooks 自动化

让 hooks 处理重复性任务：
- 保存时自动格式化
- 测试文件更改时运行测试
- schema 更改时重新生成类型
- 阻止在受保护分支上编辑

### 4. 为复杂工作流创建 Agent

Agent 适合：
- 代码审查（使用你团队的检查清单）
- PR 创建和管理
- 调试工作流
- 任务入门

### 5. 利用 GitHub Actions

自动化维护：
- 每个 PR 的 PR 审查
- 每周质量扫描
- 每月文档对齐
- 依赖更新

### 6. 版本控制你的配置

提交所有内容，除了：
- `settings.local.json`（个人偏好）
- `CLAUDE.local.md`（个人笔记）
- 用户特定的凭据

---

## 本仓库示例

| 文件 | 描述 |
|------|------|
| [CLAUDE.md](CLAUDE.md) | 示例项目记忆文件 |
| [.claude/settings.json](.claude/settings.json) | 完整的 hooks 配置 |
| [.claude/settings.md](.claude/settings.md) | 人类可读的 hooks 文档 |
| [.mcp.json](.mcp.json) | MCP 服务器配置（JIRA、GitHub、Slack 等） |
| **Agent** | |
| [.claude/agents/code-reviewer.md](.claude/agents/code-reviewer.md) | 综合代码审查 Agent |
| [.claude/agents/github-workflow.md](.claude/agents/github-workflow.md) | Git 工作流 Agent |
| **命令** | |
| [.claude/commands/onboard.md](.claude/commands/onboard.md) | 深度任务探索 |
| [.claude/commands/ticket.md](.claude/commands/ticket.md) | **JIRA/Linear 工单工作流（读取 → 实现 → 更新）** |
| [.claude/commands/pr-review.md](.claude/commands/pr-review.md) | PR 审查工作流 |
| [.claude/commands/pr-summary.md](.claude/commands/pr-summary.md) | 生成 PR 摘要 |
| [.claude/commands/code-quality.md](.claude/commands/code-quality.md) | 质量检查 |
| [.claude/commands/docs-sync.md](.claude/commands/docs-sync.md) | 文档同步 |
| **Hooks** | |
| [.claude/hooks/skill-eval.sh](.claude/hooks/skill-eval.sh) | 技能评估包装器 |
| [.claude/hooks/skill-eval.js](.claude/hooks/skill-eval.js) | Node.js 技能匹配引擎 |
| [.claude/hooks/skill-rules.json](.claude/hooks/skill-rules.json) | 模式匹配规则 |
| **技能** | |
| [.claude/skills/testing-patterns/SKILL.md](.claude/skills/testing-patterns/SKILL.md) | TDD、工厂函数、mock |
| [.claude/skills/systematic-debugging/SKILL.md](.claude/skills/systematic-debugging/SKILL.md) | 四阶段调试 |
| [.claude/skills/react-ui-patterns/SKILL.md](.claude/skills/react-ui-patterns/SKILL.md) | 加载/错误/空状态 |
| [.claude/skills/graphql-schema/SKILL.md](.claude/skills/graphql-schema/SKILL.md) | 查询、变更、代码生成 |
| [.claude/skills/core-components/SKILL.md](.claude/skills/core-components/SKILL.md) | 设计系统、token |
| [.claude/skills/formik-patterns/SKILL.md](.claude/skills/formik-patterns/SKILL.md) | 表单处理、验证 |
| **GitHub 工作流** | |
| [.github/workflows/pr-claude-code-review.yml](.github/workflows/pr-claude-code-review.yml) | 自动 PR 审查 |
| [.github/workflows/scheduled-claude-code-docs-sync.yml](.github/workflows/scheduled-claude-code-docs-sync.yml) | 每月文档同步 |
| [.github/workflows/scheduled-claude-code-quality.yml](.github/workflows/scheduled-claude-code-quality.yml) | 每周质量审查 |
| [.github/workflows/scheduled-claude-code-dependency-audit.yml](.github/workflows/scheduled-claude-code-dependency-audit.yml) | 双周依赖审计 |

---

## 了解更多

- [Claude Code 文档](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code Action](https://github.com/anthropics/claude-code-action) - GitHub Action
- [Anthropic API](https://docs.anthropic.com/en/api)

---

## 许可证

MIT - 将此作为你自己项目的模板使用。
