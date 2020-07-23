# Git常用命令

拉取upstream 分支

```text
git checkout  -b master-upstream remotes/origin/master
```

恢复还没有add 的文件

```text
git checkout
```

把add进来的文件变成非add状态的文件

```text
git reset

# 默认方式，不带任何参数的git reset，即时这种方式，它回退到某个版本，只保留源码，回退commit和index信息
git reset --mixed

# 回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可
git reset --soft

#彻底回退到某个版本，本地的源码也会变为上一个版本的内容，此命令 慎用！
git reset  --hard
```

撤销一次提交

```text
git revert
```

添加远程分支

```text
git remote add upstream
```

删除远端分支，下面的push 冒号是需要的。

```text
git branch -r -d origin/branch-name  
git push origin :branch-name
```

删除upstream 仓库

```text
git remote remove <repository-name>
```

查看分支的远程分支

```text
git branch -vv
```

查看这个分支的一些历史。比如从哪里checkout出来的。

```text
git reflog --date=local | grep <branchname>
```

