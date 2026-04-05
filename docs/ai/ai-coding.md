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

> Slash Command 支持 Tab 键补全

### Memory

Memory in Claude Code means context that persists across sessions， are loaded automatically every time Claude Code starts. 



## References

- [Anatomy of the .claude/ Folder](https://blog.dailydoseofds.com/p/anatomy-of-the-claude-folder)
- [Learn Claude Code by doing, not reading.](https://claude.nagdy.me/)
- [Claude Code Unpacked](https://ccunpacked.dev/)