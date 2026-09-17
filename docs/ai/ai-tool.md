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
