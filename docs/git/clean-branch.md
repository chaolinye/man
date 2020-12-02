# 清理分支

在团队开发过程中，Git 仓库的分支数量会不断增长，慢慢地会变得过于繁多，影响查看效率和性能。因此偶尔需要对分支进行清理。

## 删除本地分支

```bash
# 删除已合入当前分支的其他分支
git branch -d branchName

# 强制删除分支
git branch -D branchName
```

> 先用 `-d` 删除，如果删除失败，则确认后，再使用 `-D` 强制删除

## 删除远程分支

```bash
git push --delete origin branchName
```

## 同步清理远程分支

在其他人清理远程分支或者使用 `Gitlab` 等服务器界面内清理分支的情况下，被删除的远程分支对应的本地仓库的 `remoteName/branchName` 分支依旧会存在

> 即远程仓库分支被删除后，本地仓库 fetch 进行同步时，并不会删除 `remoteName/branchName` 分支，这也算是 Git 的一种安全机制

可通过运行以下命令进行同步删除

```bash
git remote prune originn
```