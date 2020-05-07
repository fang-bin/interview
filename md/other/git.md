# GIT
###### 撤销之前的缓存操作

`git add` --> `git reset`

`git commit` --> `git rest --soft HEAD^`

git reset的几个参数：

--mixed 
意思是：不删除工作空间改动代码，撤销commit，并且撤销git add . 操作
这个为默认参数,git reset --mixed HEAD^ 和 git reset HEAD^ 效果是一样的。


--soft  
不删除工作空间改动代码，撤销commit，不撤销git add . 

--hard
删除工作空间改动代码，撤销commit，撤销git add . 


###### git rebase
变基

###### git cherrypick