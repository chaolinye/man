# AI Coding

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

## References

- [Anatomy of the .claude/ Folder](https://blog.dailydoseofds.com/p/anatomy-of-the-claude-folder)
- [Learn Claude Code by doing, not reading.](https://claude.nagdy.me/)
- [Claude Code Unpacked](https://ccunpacked.dev/)