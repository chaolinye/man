
Git 是当今最流行的分布式版本控制系统，掌握 Git 是每个开发者的必备技能

## 常用命令

```bash
# 查看两个分支的最近公共祖先
git merge-base branchA branchB
# 压缩历史文件，减少磁盘占用
git gc

## 交互式选择哪个文件来提交
git commit -p
```

## 学习资料

- [Pro Git 2](https://bingohuang.gitbooks.io/progit2/content/)

    > Git 官方推荐书籍，学习 Git 最好的书籍，没有之一

- [git 中文件的存储方式](https://jvns.ca/blog/2023/09/14/in-a-git-repository--where-do-your-files-live-/)


## 常用场景

### Git 的离线使用

[资料](https://www.gibbard.me/using_git_offline/)

思路1: 

- 通过 `git init --bare` 在 U 盘等存储上创建 git 仓库，
- 把本地的仓库通过 `git remote add origin /path/to/usb/repoName.git && git push origin master` 推送到 U 盘
- 其它电脑通过 `git clone /path/to/usb/repoName.git` 获取仓库内容

思路2:

- 通过 `git bundle create repoName.bundle --all` 直接从本地仓库创建一个单文件，copy 到 U 盘
- 其它电脑通过 `git clone repoName.bundle` 获取仓库内容