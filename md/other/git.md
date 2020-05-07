# GIT
##### 撤销之前的缓存操作

`git add` --> `git reset`

`git commit` --> `git rest --soft HEAD^`

**`git reset`的几个参数：**

--mixed 
意思是：不删除工作空间改动代码，撤销`commit`，并且撤销`git add` 操作
这个为默认参数,`git reset --mixed HEAD^` 和 `git reset HEAD^` 效果是一样的。


--soft  
不删除工作空间改动代码，撤销`commit`，不撤销`git add` . 

--hard
删除工作空间改动代码，撤销`commit`，撤销`git add` 

##### git commit --amend
`commit`注释写错了，上述命令可以进入默认 vim 编辑器，修改注释完毕后保存就好了。

##### git rebase
变基

**合并N次commit**
`git rebase -i HEAD~${N}` 不过不能合并已经推到远程的`commit`

**分支合并**
`merge`合并分支，会在记录里发现一些 `merge` 的信息，但是我们觉得这样污染了 `commit` 记录，想要保持一份干净的 `commit`，可以使用`git rebase`

`rebase` 做了什么操作呢？

首先，`git` 会把 `feature1` 分支里面的每个 `commit` 取消掉；
其次，把上面的操作临时保存成 `patch` 文件，存在 `.git/rebase` 目录下；
然后，把 `feature1` 分支更新到最新的 `master` 分支；
最后，把上面保存的 `patch` 文件应用到 `feature1` 分支上；

在 `rebase` 的过程中，也许会出现冲突 `conflict`。在这种情况，`git` 会停止 `rebase` 并会让你去解决冲突。在解决完冲突后，用 `git add` 命令去更新这些内容。

`git rebase --continue`

这样 `git` 会继续应用余下的 `patch` 补丁文件。

在任何时候，我们都可以用 `--abort` 参数来终止 `rebase` 的行动，并且分支会回到 `rebase` 开始前的状态。

`git rebase —abort`

**注意** `git rebase`是一个非常危险的操作，因为它改变了历史，除非你可以肯定该分支只有你自己使用，否则请谨慎操作。

##### git cherry-pick

git cherry-pick可以理解为”挑拣”提交，它会获取某一个分支的单笔提交，并作为一个新的提交引入到你当前分支上。 当我们需要在本地合入其他分支的提交时，如果我们不想对整个分支进行合并，而是只想将某一次提交合入到本地当前分支上，那么就要使用git cherry-pick了。

寻常没什么用...

###### 参考

[git-rebase参考](http://jartto.wang/2018/12/11/git-rebase/)
