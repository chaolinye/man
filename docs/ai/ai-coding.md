# AI Coding

## Tips

- 输出的好坏取决于 Context 的质量（**Context is King**）
    - 提供足够的背景知识
    - 尽量详细
    - 保持 Context 的简单，不要放置无关的内容（**新鲜& 精炼**）
        - 关注Context的内容
        - 新任务开启新的 session
- Agent 走偏要及时纠正（不要害怕中断和重新开始）
    - 手动中断，添加信息再往下走
    - 回到之前的环节，添加信息，重新开始
    - 开启新 Session，优化prompt，重新开始
- 复杂任务先计划，调整计划，再行动(`give me a plan/optimize plan/agree/do it`)
- 所有事情都让 agent 完成，不要自己上手(构建自己独特的ai workflow)  `Turn this into a skill I can reuse `
- second brain (.claude/xxxx.md)
    - 重复的步骤，让 agent 总结生成 skills
    - 完成任务，让 agent 把做的事情输出 project note `update project note by above`
    - 让 agent 更新 CLAUDE.md

### 领域知识累积 -- Build a local second Brain

```bash
# 更新到项目的 .claude/NOTES.md
Update the project notes with what we worked on
```

让 Coding Agent 了解该项目的历史记录和决策，无需重新解释

下次启动获取最新状态

```bash
Give me the latest status report
```

### Build Custom Workflow -- Skill

```bash
Turn this into a skill I can reuse 
```

方向：
- PR Spliting

### Before You Code，Reach A Shared Design Concept

> No one exactly what they want

对齐 Design Concept（概念空间）

```bash
# https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md
# 比 plan mode 更好用 
/grill-me
# https://github.com/mattpocock/skills/blob/main/skills/engineering/grill-with-docs/SKILL.md
# /grill-me 升级版，记录和适用Concept文档及架构决策文档
# 有 codebase 时使用，无 codebase 时使用 grill-me
/grill-with-docs
# https://github.com/mattpocock/skills/blob/main/skills/engineering/to-prd/SKILL.md
/write-a-prd
# https://github.com/mattpocock/skills/blob/main/skills/engineering/to-issues/SKILL.md
/write-issues
# https://github.com/mattpocock/skills/blob/main/skills/engineering/to-issues/SKILL.md
/prd-to-issues
```

> AI Coding 也要每天投入精力到系统设计，而不能仅仅是 spec2code

### 让 AI 写出好的代码

- 易测试的
- 层次分明的

```bash
# https://github.com/mattpocock/skills/blob/main/skills/engineering/tdd/SKILL.md
/tdd
# https://github.com/mattpocock/skills/blob/main/skills/engineering/improve-codebase-architecture/SKILL.md 
/improve-codebase-architecture
```

### Design the interface, delegate the implemenntatiton

人脑跟不上，要记忆太多东西，太累

对于非关键模块，只关注接口，不关注实现，只需要接口能按预期通过测试和验证即可


## Hermess Engineering

### Framework

- [ECC](https://github.com/affaan-m/everything-claude-code): everything-claude-code

## Claude Code 

### Slash Command

Context Management:
- `/context`: shows a visual grid of your context usage
- `/compact`: compresses the conversation — pass instructions to control what’s preserved: `/compact focus on xxx`
- `/clear`: starts completely fresh.

Session Tools:
- `/rename my-feature`: gives the session a readable name.
- `/resume`: picks up a previous session
- `/branch`: creates a parallel conversation to explore an alternative without losing your current state
- `/rewind`: rolls back to an earlier point
- `/export`: saves the session to a file or clipboard.

Configuration:
- `/model`: switches between available models 
- `/effort`: sets reasoning depth — low, medium, high, max, or auto
- `/permission` & `/allowed-tools`:  manage what Claude can do without asking
- `/config`: opens the settings menu.

Diagnostics:
- `/cost`: shows session cost, duration, code changes, and token usage.
- `/status`: shows version, model, and account info
- `/doctor`: checks installation health
- `/diff`: opens an interactive viewer for uncommitted changes — useful for reviewing what Claude has done before committing.
- `/debug`: 开启详细日志排错

Other:
- `/insights`: 生成会话分析报告
- `/simplify`：多维度并行审查代码质量
- `/batch`：批量跨文件大规模修改
- `/loop`：定时循环执行指令（监控部署 / 构建）
- `/btw`(by the way): 侧边提问，不污染对话历史

> Slash Command 支持 Tab 键补全

### 快捷键

- `Shift+Tab`：快速切换权限模式（default/acceptEdits/plan）
- `Option/Alt+T`：切换深度思考模式
- `Ctrl+O`：查看工具调用与思考过程
- `Ctrl+B`：后台运行子任务
- `Ctrl+X Ctrl+K`：终止所有后台任务

### Memory

Memory in Claude Code means context that persists across sessions， are loaded automatically every time Claude Code starts. 

分为手动记忆和自动记忆。

手动记忆：
- 组织级记忆：团队全局统一规则
- 项目级记忆：最常用，为项目根目录`./CLAUDE.md`或`.claude/CLAUDE.md`，可提交 Git 共享，存放技术栈、规范、命令、坑点等
- 用户级记忆：`~/.claude/CLAUDE.md`，存放个人偏好与通用习惯

> 大型项目可拆分规则到.claude/rules/*.md，支持用paths指定生效路径，实现按文件类型精准触发。

创建与更新记忆:
- 快速初始化：执行`/init`自动生成基础`CLAUDE.md`；交互式初始化用`CLAUDE_CODE_NEW_INIT=true claude`.这个文件会通过阅读目录结构、文档、配置文件来获取项目的技术栈、关键命令、初始规范。
- 编辑管理：`/memory`调用系统编辑器修改，保存后自动重载
- 引用文档：用@文件路径导入现有文档，支持最多五层嵌套
- 指令记忆：自然语言告知 “记住 XX”，或明确要求写入`CLAUDE.md`

自动记忆:
- 机制：Claude 自动记录项目规律、调试思路等，存于`~/.claude/projects/<项目>/memory/`
- 加载：会话自动加载`首 200 行 / 25KB`，其他主题文件按需加载
- 管理：可手动修改纠错；支持开关、会话级禁用、自定义存储路径

!> Monorepo 优化：用 `claudeMdExcludes` 排除无关CLAUDE.md，避免上下文干扰


## Pi

特点：只内置 AI Coding Agent 的最小功能集，需要其它功能就让Agent自己构建，定位类似于 nvim 这样的完全个性化工具

优点：没有冗余的功能，需要什么功能用户让Agent构建，保证用户对每个功能都足够掌握。

### Shortcut

| 快捷键/指令 | 功能说明 |
| --- | --- |
| `Ctrl+L` | 选择模型 |
| `Ctrl+P` | 切换模型 |
| `Shift+Tab` | 调整思考等级 |
| `Ctrl+G` | 打开外部编辑器 |
| `Ctrl+-` | 撤销编辑器中的文本操作 |
| `Enter` | 提交/引导（指令） |
| `Alt+Enter` | 跟进对话/续写 |
| `Escape` | 中止当前操作 |
| `Alt+Up` | 出队/取消排队 |
| `/settings` | 打开通用配置 |

### Prompt Stack

#### 一、Prompt 叠加顺序（栈结构，由外到内/由基础到补充）
| 顺序 | 层级内容 | 作用说明 |
| :--- | :--- | :--- |
| 1（最外层/基础） | `default prompt` | Pi 自带的默认基础提示词，作为整个提示词栈的底层框架 |
| 2 | `APPEND_SYSTEM.md` | 在默认提示词基础上追加补充的系统规则 |
| 3 | `AGENTS.md / CLAUDE.md` | 代理/项目级别的配置文件，定义模型行为规则 |
| 4 | `skills list` | 模型可用的技能列表，明确能力边界 |
| 5（最内层/补充） | `date + working dir` | 动态注入当前日期和工作目录信息，提供环境上下文 |

#### 二、配置文件路径与作用映射
| 文件路径 | 作用说明 | 备注 |
| :--- | :--- | :--- |
| `~/.pi/agent/AGENTS.md` | 全局级配置（# global） | 对所有项目生效的通用规则 |
| `./AGENTS.md` / `./CLAUDE.md` | 项目级配置（# project tree） | 仅对当前项目生效的规则，优先级高于全局配置 |
| `.pi/SYSTEM.md` | 替换默认提示词（# replace default） | 直接覆盖 Pi 的 `default prompt`，完全自定义基础框架 |
| `.pi/APPEND_SYSTEM.md` | 追加系统提示（# append） | 在原有的默认/替换后的系统提示词末尾，补充额外规则 |

这张图介绍了 Pi 中两种可复用工作流工具：**Skills（技能包）** 和 **Prompt Templates（提示词模板）**，核心是二者的定位差异与适用场景，总结如下：

### Skills vs Prompt Templates

| 类别 | 核心定位 | 用途说明 | 示例 |
| :--- | :--- | :--- | :--- |
| **Skills（技能包）** | 可复用的能力包 | 封装工作流、环境配置、脚本、参考资料等，先加载描述，再按需加载完整指令，提供固定能力 | `/skill:brave-search`、`/skill:pdf-tools extract` |
| **Prompt Templates（提示词模板）** | 可复用的提示词模板 | 通过斜杠命令快速调用预设提示词，适合代码审查、重构、问题分类、项目重复性任务 | `/.pi/prompts/review.md`（模板内容：`Review $@ for bugs and missing tests.`） |

关键区别一句话概括:
- **Skills**：给模型加“固定能力”，侧重底层功能/工具调用
- **Prompt Templates**：给模型加“固定话术/任务指令”，侧重上层任务执行，可以理解为最简单的Skills

### Extensions are where Pi becomes your agent

扩展是让 Pi 从对话工具升级为自动化智能体的关键。

#### 六大扩展模块详解

| 模块 | 核心功能 | 作用说明 |
| :--- | :--- | :--- |
| **tools** | `LLM-callable functions` | 给大模型提供可直接调用的外部函数，让模型能执行代码、调用API等操作 |
| **commands** | `your own /slash flows` | 自定义斜杠命令工作流，通过`/`命令快速触发预设任务流程 |
| **events** | `intercept turns and tools` | 拦截对话轮次和工具调用，实现流程控制、钩子逻辑与自定义行为 |
| **UI** | `prompts, confirms, widgets` | 自定义交互界面，包括提示框、确认弹窗、组件小部件等交互元素 |
| **providers** | `dynamic model sources` | 支持接入不同的大语言模型，动态切换模型源 |
| **state** | `session persistence` | 实现会话状态持久化，保存对话上下文和运行状态，支持跨会话记忆 |

简单来说，这些扩展覆盖了工具调用、自定义命令、流程拦截、界面交互、多模型接入和状态管理，让 Pi 可以被深度定制为适配不同场景的专属智能体。

这张图是关于 **Pi 扩展包（Packages）** 的获取与发布渠道，核心信息总结如下：

#### 扩展资源渠道

如果不想自己动手实现或者想学习extension怎么编写，可以通过以下的资源渠道获得 extensions

| 渠道 | 说明 | 地址 |
| :--- | :--- | :--- |
| **OFFICIAL EXAMPLES（官方示例）** | 官方提供的扩展示例代码，可参考学习 | `github.com/badlogic/pi-mono/.../examples/extensions` |
| **PACKAGE GALLERY（扩展包画廊）** | 官方的扩展包平台，可获取/发布社区扩展 | `pi.dev/packages` |

### Pi 刻意不内置的功能（Limits）

Pi 刻意不内置的功能，本身就是它设计理念的一部分。

####  Pi 不内置的功能列表
| 不内置的功能 |
| :--- |
| built-in MCP（内置的模型控制协议支持） |
| sub-agents（子代理） |
| permission popups（权限弹窗） |
| plan mode（规划模式） |
| to-dos（待办事项） |
| background bash（后台Bash执行） |

#### 核心设计规则

**设计原则**：如果你每天都需要某个工作流，就把它做成模板（template）、技能（skill）、扩展（extension）或包（package），而不是依赖工具自带。

### 解决问题时，优先选择最简单、最轻量化的配置方式

#### 决策层级对照表（从最简单到最复杂）
| 目标场景 | 对应配置方式 | 说明 |
| :--- | :--- | :--- |
| **修改默认设置** | `settings.json` | 调整基础参数，是最轻量化的配置方式 |
| **定义项目规则** | `AGENTS.md` | 给项目添加通用行为规则，比如代码规范、项目约定 |
| **替换智能体身份/基础系统提示** | `SYSTEM.md` | 完全覆盖默认的系统提示词，重塑智能体的基础设定 |
| **重复使用固定提示词** | `prompt template` | 保存常用指令模板，通过斜杠命令快速调用 |
| **新增可复用能力** | `skill` | 封装可复用的工具/工作流，按需加载使用 |
| **深度改变行为逻辑** | `extension` | 开发扩展模块，实现自定义工具、事件拦截、界面交互等高级功能 |
| **分享/打包配置集合** | `package` | 将模板、技能、扩展打包成完整的包，方便分享和复用 |

#### 一句话核心逻辑

能改配置文件就不用写模板，能用模板就不用写技能，能用技能就不用开发扩展——始终用最低成本的方式解决问题。

## References

- [Anatomy of the .claude/ Folder](https://blog.dailydoseofds.com/p/anatomy-of-the-claude-folder)
- [Learn Claude Code by doing, not reading.](https://claude.nagdy.me/)
- [Claude Code Unpacked](https://ccunpacked.dev/)
- [Claude Code Analysis](https://github.com/liuup/claude-code-analysis)
- [How I use Claude Code (Meta Staff Engineer Tips)](https://www.youtube.com/watch?v=mZzhfPle9QU)
- [Claude Code Workflows That Will 10x Your Productivity](https://www.youtube.com/watch?v=yZvDo_n12ns)
- [How I Turned Pi Into the Ultimate Coding Agent](https://www.youtube.com/watch?v=6xXjHM3V1zM&t=1102s)
- [Learn 90% Of Pi Agent in Under 17 Minutes](https://www.youtube.com/watch?v=8Dt0HM8HIq4&t=94s)
- [Pi Document](https://pi.dev/docs/latest)
- [I Hated Every Coding Agent, So I Built My Own — Mario Zechner (Pi)](https://www.youtube.com/watch?v=Dli5slNaJu0)
- ["Software Fundamentals Matter More Than Ever" — Matt Pocock](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=580s)
- [WeiXin iLink API: Bot](https://www.wechatbot.dev/en/protocol)
- [Project Note Skill](https://github.com/a3lem/my-claude-plugins)