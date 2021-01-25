# Git Patch

Git 可以通过创建 patch 的方式将自己的提交离线提供给别人

> 在 Github 兴起之前，开源项目主要就是通过邮件发送 patch 来进行贡献

## 适用场景

- 协作双方没有共享的代码库
- 临时代码共享

## 创建 Patch

有三种方式可以创建 patch

- git diff 生成标准的 patch

    ```bash
    # 生成两个 commit 之间的差异作为 patch
    git diff HEAD~..HEAD > temp.patch
    ```

- git format-patch 生成 Git 专用 patch

    ```bash
    # 生成 commit2 和 commit3 两个 patch
    git format-patch commit1..commit3
    ```

    > git format-patch 创建的 patch 包含提交信息和提交者信息

- Linux 本身的 patch 命令

    > TODO


## 应用 Patch

### git apply

git apply 可以应用使用 git diff 和 git format-patch 生成的 2 种 patch 来打补丁.

```bash
# 检查是否能正常应用
git apply --check 【path/to/xxx.patch】
# 应用 patch
git apply patch1.patch
# 遇到冲突时解决冲突
git apply --reject patch1.patch
```

!> git apply 只会合入修改，不会自动提交

### git am

git am 只能应用 git format-patch 生成的 patch 文件

```bash
git am patch1.patch
```

!> git am 会直接使用 patch 文件中的 diff 的信息，还有提交者，时间等等来自动提交,不需要我们再去提交 commit

## References

- [Git 打补丁-- patch 和 diff 的使用（详细）](https://www.jianshu.com/p/ec04de3f95cc)





