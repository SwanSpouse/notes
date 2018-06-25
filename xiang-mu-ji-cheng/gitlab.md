#### Gitlab webhook 应用

通过master，release两个分支来保证代码的开发和发布。

* release：合并到release分支上的代码必须是能够立即发外网的。
* master：合并到master分支上的代码有时候不能发布外网，比如现在1.0版本在自己的开发分支dev1.0已经测试并无问题，现在合并到master来进行内网测试。如果此时外网出现问题，由于此时master正在进行内网测试 ，不能立即发外网。所以可以提交hot-fix到release上面，测试后发出外网。然后把hot-fix分支cherry-pick到master上，继续进行内网测试。

master和release两个分支可以解决master在进行内网测试，不能及时发布release的尴尬。

#### merge-check

项目中添加了一个hook关联到jenkins的merge-check任务，当有新的mr打开时会执行merge-check：

1、如果目标分支是release：源分支是master时，要求master分支的最近一个CI任务必须为成功状态，否则无法接受mr；源分支不是master时（如紧急hot-fix），提示需要在feature分支上运行CI任务通过测试；

2、目标分支时其他：要求没有处于open状态且目标分支是release的mr。

以上两点保证了所有合并到release分支的代码（hot-fix除外）都要经过master先运行CI（且在内网测试过）。

当master代码在内网测试时，避免在合并到release前有新的代码并入master，可以预先打开一个master到release的mr从而锁住master分支。

##### Git2Jira

起一个单独的项目git2jira。在gitlab上添加一个webhook任务，当有分支合并到master的时候，向git2jira发送一个请求，git2jira接受到请求之后，将本次合并到master分支的情况通过API调用评论到JIRA对应的issue下面。这样可以将gitlab和jira关联起来。在后续追溯问题的时候，也可以通过评论中的jira号来查看当时这样修改的原因。



#### 参考

* zhiwang-gitbook



