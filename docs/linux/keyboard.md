# 快捷键

`Bash` 内置了 `Readline` 库，具有这个库提供的很多“行操作”功能，比如命令的自动补全，可以大大加快操作速度。

这个库默认采用 `Emacs` 快捷键，也可以改成 `Vi` 快捷键。

```bash
# 切换成 vi 快捷键
set -o vi
# 切换成 emacs 快捷键
set -o emacs
```

> 本文介绍的快捷键都属于 `Emacs` 快捷键

| Action | Keyboard |
| :--: | :--: |
| 移动到行首 | `Ctrl + a` |
| 移动到行尾 | `Ctrl + e` |
| 左移一个字符 | `Ctrl + b` |
| 右移一个字符 | `Ctrl + f` |
| 移动到词尾 | `Alt + f` |
| 移动到词首 | `Alt + f` |
| 删除光标前面的单词 | `Ctrl + w` |

> 上面快捷键的 Alt 键，也可以用 ESC 键替代


