

#### Gitlab webhook 应用

通过master，release两个分支来保证代码的开发和发布。

#### merge-check

项目中添加了一个hook关联到jenkins的merge-check任务，当有新的mr打开时会执行merge-check：

1、如果目标分支是release：源分支是master时，要求master分支的最近一个CI任务必须为成功状态，否则无法接受mr；源分支不是master时（如紧急hot-fix），提示需要在feature分支上运行CI任务通过测试；

2、目标分支时其他：要求没有处于open状态且目标分支是release的mr。

以上两点保证了所有合并到release分支的代码（hot-fix除外）都要经过master先运行CI（且在内网测试过）。

当master代码在内网测试时，避免在合并到release前有新的代码并入master，可以预先打开一个master到release的mr从而锁住master分支。



