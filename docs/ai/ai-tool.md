# AI Tool

## herdr -- Tmux For AI Agent

### windows 下 herdr + git bash 新建 tab 工作目录没有 follow source tab

原因: git bash cd 切换工作目录没有修改进程的当前目录属性
解决方案：在 `~/.bashrc` 中增加下面的脚本

```bash
# >>> herdr shell integration: report cwd via OSC 7 >>>
# Git Bash (MSYS2) never updates the Windows process cwd on `cd`, so herdr
# cannot read the live directory from the PEB. Emit OSC 7 before every prompt
# so host terminals (herdr) can track this shell's real working directory.
__herdr_osc7_report() {
  local mixed
  mixed="$(cygpath -m -- "$PWD" 2>/dev/null)" || return 0
  [ -n "$mixed" ] || return 0
  mixed="${mixed// /%20}"
  printf '\033]7;file:///%s\033\\' "$mixed"
}
case "$PROMPT_COMMAND" in
  *__herdr_osc7_report*) ;;
  *)
    if [ -n "$PROMPT_COMMAND" ]; then
      PROMPT_COMMAND="$PROMPT_COMMAND;__herdr_osc7_report"
    else
      PROMPT_COMMAND="__herdr_osc7_report"
    fi
  ;;
esac
# <<< herdr shell integration <<<
```

## OpenCli

- 适用场景：将网站操作封装成 cli 命令，让网站操作更加高效精确，更容易自动化
- 工作原理：通过监测网站操作时的网络请求，让AI提取其中关键的请求及其关系，找到API请求链路，使用Cli封装。登录态通过浏览器cookie或者触发浏览器请求来获取。
- 官方文档：https://github.com/jackwener/OpenCLI/blob/main/README.zh-CN.md
- 使用方式：
```bash
# 全局安装命令行工具
npm install -g @jackwener/opencli
# 安装自定义adapter skill
npx skills add jackwener/opencli --skill opencli-adapter-author
# 打开 ai coding agent
# 提示词: /opencli-adapter-author intent: xxx  steps：1. 打开 xxxx 网站 2. 选择 xxx 3. 点击xxx  4. 获取到 xxx
```

## CLI-Anything

- 适用场景：有某个软件或者网站的源码，开发其 cli 工具
- 官方文档: https://github.com/HKUDS/CLI-Anything
