# CodeSage

<div align="center">

[![PyPI Version](https://img.shields.io/pypi/v/codesage.svg)](https://pypi.org/project/codesage/)
[![Python Support](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-alpha-orange.svg)](#)

</div>

轻量级终端 Coding Agent，基于 **ReAct** 与 **Plan Mode** 双模式驱动 LLM 自主完成编程任务。

## ✨ 特性

| 特性 | 说明 |
|------|------|
| 🤖 **多模型支持** | Anthropic Claude / OpenAI 双协议，统一接口，灵活切换 |
| 📋 **Plan Mode** | 编辑类操作需用户审批，支持 accept / deny / bypass 三种权限模式 |
| 🔌 **MCP 扩展** | 通过 Model Context Protocol 接入任意工具服务 |
| 🎯 **Skill 技能包** | 可插拔的技能定义，frontmatter 配置，支持自定义工作流 |
| 💾 **跨会话记忆** | 持久化记忆检索，项目上下文自动加载，智能回忆 |
| 👥 **多 Agent 协作** | 子 Agent 派生、Fork 隔离、Verification Agent 校验 |
| 🪝 **Hooks 钩子** | 事件驱动的自定义行为扩展，前后置自动化 |
| 🌿 **Worktree** | 独立分支工作区，隔离实验性改动，安全试错 |
| 💻 **Terminal UI** | 基于 Textual 构建的现代终端图形界面 |

## 🏗️ 架构

```
┌─────────────────────────────────────────────────┐
│              交互层 (Interaction)                 │
│  Terminal UI  ·  CLI Prompt  ·  MCP Server       │
├─────────────────────────────────────────────────┤
│              引擎层 (Engine)                      │
│  Agent Loop (ReAct)  ·  Plan Mode  ·  Multi-Agent│
├─────────────────────────────────────────────────┤
│              工具层 (Tools)                       │
│  Bash  ·  File Edit  ·  Grep  ·  Glob  ·  Search│
├─────────────────────────────────────────────────┤
│              记忆层 (Memory)                      │
│  Cross-session Recall  ·  Auto-memory  · Config  │
├─────────────────────────────────────────────────┤
│              安全层 (Security)                    │
│  Permission Checker  ·  Sandbox  ·  Hooks        │
└─────────────────────────────────────────────────┘
```

## 🚀 快速开始

### 安装

```bash
# 使用 pip
pip install -e .

# 或使用 uv（推荐）
uv pip install -e .
```

### 配置

创建配置文件 `~/.codesage/config.yaml`：

```yaml
providers:
  - name: claude
    protocol: anthropic
    model: claude-sonnet-4-5-20250514
    base_url: https://api.anthropic.com
    api_key: sk-ant-xxxxx
    thinking: false

permission_mode: acceptEdits     # default | acceptEdits | plan | bypassPermissions
```

> 环境变量中的密钥可通过 `${ANTHROPIC_API_KEY}` 引用，支持任意 provider。

### 运行

```bash
# 启动终端 UI（交互式）
codesage

# 非交互式单轮运行
codesage -p "Create a hello world Flask app"
```

## ⌨️ 命令

| 命令 | 说明 |
|------|------|
| `/help` | 显示帮助信息 |
| `/status` | 查看当前状态（模型、权限模式、上下文窗口） |
| `/clear` | 清除当前对话 |
| `/compact` | 压缩上下文，减少 token 消耗 |
| `/memory` | 管理跨会话记忆（增删查） |
| `/skill` | 管理技能包（加载/卸载/注册） |
| `/worktree` | 管理工作树（创建/切换/删除） |
| `/mcp` | 管理 MCP 服务器连接 |
| `/tasks` | 查看子任务执行状态 |

## ⚙️ 配置参考

### Provider 配置

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | ✅ | 提供者名称，用于标识 |
| `protocol` | string | ✅ | 协议类型：`anthropic` / `openai` / `openai-compat` |
| `model` | string | ✅ | 模型名称，如 `claude-sonnet-4-5-20250514` |
| `base_url` | string | ✅ | API 基础地址 |
| `api_key` | string | ✅ | API 密钥，支持 `${ENV_VAR}` 引用 |
| `thinking` | boolean | ❌ | 是否启用深度思考模式，默认 `false` |
| `context_window` | int | ❌ | 上下文窗口大小，自动检测时可不填 |
| `max_output_tokens` | int | ❌ | 最大输出 token 数，默认 8192（thinking 模式 64000） |

### 全局配置

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `permission_mode` | string | `default` | 权限模式 |
| `mcp_servers` | array | `[]` | MCP 服务器列表 |
| `enable_fork` | boolean | `false` | 启用 Fork 隔离模式 |
| `enable_verification_agent` | boolean | `false` | 启用验证 Agent |
| `teammate_mode` | string | `""` | 队友协作模式：`in-process` |
| `worktree.symlink_directories` | array | `["node_modules", ".venv", "vendor"]` | 符号链接排除目录 |

## 📁 项目结构

```
codesage/
├── agent.py              # Agent 核心循环 (ReAct Loop)
├── app.py                # Textual 终端 UI 主程序
├── client.py             # LLM 客户端 (Anthropic / OpenAI 双协议)
├── config.py             # 配置加载、合并与环境变量解析
├── conversation.py       # 对话管理与消息历史
├── driver.py             # 无 AltScreen 终端驱动
├── prompts.py            # 系统提示词构建器
├── validator.py          # 配置校验与模型上下文映射
├── __main__.py           # 入口点
├── agents/               # 多 Agent 系统
│   ├── loader.py         # Agent 动态加载器
│   ├── task_manager.py   # 异步任务管理
│   ├── trace.py          # 执行追踪与调试
│   ├── fork.py           # Fork 隔离机制
│   └── notification.py   # Agent 间通知
├── commands/             # 斜杠命令系统
│   ├── handlers/         # 各命令处理器
│   ├── registry.py       # 命令注册表
│   └── completion.py     # 命令补全
├── hooks/                # 钩子系统
│   ├── engine.py         # 条件评估与执行引擎
│   ├── models.py         # Hook/Action 数据模型
│   └── conditions.py     # 条件表达式解析
├── mcp/                  # MCP 协议支持
│   ├── client.py         # MCP JSON-RPC 客户端
│   ├── manager.py        # 服务器生命周期管理
│   └── tool_wrapper.py   # MCP 工具 → CodeSage 工具适配
├── memory/               # 跨会话记忆
│   ├── recall.py         # 语义记忆检索
│   ├── auto_memory.py    # 自动记忆提取
│   ├── instructions.py   # 指令加载与 @include 解析
│   └── session.py        # 会话摘要生成
├── permissions/          # 权限系统
│   ├── checker.py        # 权限检查器
│   ├── rules.py          # YAML 规则引擎
│   ├── sandbox.py        # 路径沙箱 & 危险命令检测
│   └── modes.py          # 权限模式枚举
├── skills/               # 技能包系统
│   ├── executor.py       # 技能执行器
│   ├── loader.py         # 项目/用户技能加载
│   ├── parser.py         # Frontmatter + SKILL.md 解析
│   └── directory.py      # 技能目录管理
├── teams/                # 多 Agent 协作
│   ├── coordinator.py    # 协调器模式
│   ├── manager.py        # 团队生命周期
│   ├── mailbox.py        # 异步邮箱通信
│   └── spawn_*.py        # 进程/终端.spawn 策略
├── tools/                # 内置工具集
│   ├── bash.py           # Shell 命令执行
│   ├── edit_file.py      # 智能文件编辑
│   ├── write_file.py     # 文件写入
│   ├── read_file.py      # 文件读取
│   ├── glob.py / grep.py # 文件搜索
│   ├── agent_tool.py     # 子 Agent 创建
│   └── task_*.py         # 任务管理工具
├── worktree/             # 工作树管理
│   ├── manager.py        # 创建/切换/删除
│   ├── changes.py        # 变更追踪
│   └── cleanup.py        # 过期清理
tests/                    # 测试套件
├── test_agent.py         # Agent 核心循环测试
├── test_permissions.py   # 权限系统测试
├── test_memory.py        # 记忆功能测试
└── test_worktree.py      # 工作树测试
```

## 🧪 开发

```bash
# 同步依赖
uv sync

# 运行全部测试
pytest tests/ -v

# 运行单个测试文件
pytest tests/test_agent.py -v

# 运行特定测试
pytest tests/test_agent.py::test_single_step_tool_call -v
```
![Uploading image.png…]()

## 📄 License

[MIT](LICENSE)
