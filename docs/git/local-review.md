# 本地检视

在 Pull Request 代码比较复杂的情况下，需要拉取代码到本地 IDE 进行 Review

- 拉取 Pull Request 提交者代码分支

```bash
# 拉取提交的分支，并把 FETCH_HEAD 指针指向它
git fetch git@gitee.com:somebody/someproject.git develop

# 基于 FETCH_HEAD 创建一个新的本地分支用于检视
git checkout -b somebody-develop FETCH_HEAD
```