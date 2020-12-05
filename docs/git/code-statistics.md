# 代码统计

利用 Git 的 `git log` 命令可以实现一些简单的代码统计

比如: 

- 统计从 `2019-01-21` 这个日期以来每个人提交的代码量

    ```bash
    git log --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$name" --since="2019-01-21" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done
    ```

- 统计个人 `2020-12-03` 以来提交的代码量

    ```bash
    git log --author="username" --since="2020-12-03" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -
    ```