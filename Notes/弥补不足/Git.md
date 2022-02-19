1. 命令
```git
git status                          查看git状态，包括工作区和暂存区
git add <file>                      保存指定文件修改到暂存区
git add .                           保存所有修改到暂存区
git commit -m "xxxx"                提交文件，从暂存区到版本库
git reset --hard <commit_id>        强制回退
git log                             查看提交日志，可以找到commitId
git reflog                          操作记录，可以找到commitId
git diff                            查看修改，可以确认修改内容
git checkout -- <file>              撤销工作区file文件的所有修改，其实是用版本库中的文件替换工作区的文件，可以用于恢复工作区误删的文件
git reset HEAD <file>               撤销暂存区file文件的所有修改，重新放回工作区
git rm <file>                       删除文件

git checkout -b dev                 创建并切换到dev分支
git switch -c dev                   创建并切换到dev分支（新版Git功能）    
git branch dev                      创建dev分支
git checkout dev                    切换到dev分支
git switch dev                      切换到dev分支（新版Git功能）
git branch                          查看分支
git merge dev                       合并dev分支到当前分支
git branch -d dev                   删除dev分支

git log --graph --pretty=oneline --abbrev-commit  查看分支图

git tag <tagname> [<commit_id>]     打标签到当前分支的Head(或者指定的commitId)
git tag -a <tagname> -m "xxx"       加标签并加描述信息
git tag                             查看所有标签
git show <tagname>                  显示某个标签具体信息
git push origin <tagname>           推送本地指定标签到远程
git push origin --tags              推送本地所有标签到远程
git tag -d <tagname>                删除一个本地标签
git push origin :refs/tags/<tagname>删除一个远程标签


git config --global color.ui true   显示颜色
```
2. 回退
未push的情况下回退直接使用reset命令 --hard强制切换head到某个commitId，就可以回退


个人的一些理解，可能大家都知道，但是本人一直云里雾里的，这次一次性弄清楚

Git仓库，在本地和远程的都是一个完整的代码库，可以进行一样的操作（不涉及远程，在本地操作本地仓库，在远程操作远程仓库，都是一样的命令和思路），但是要在两个代码库中进行同步，就涉及到了远程命令，所谓的远程命令其实都是一套同步命令，你在本地代码库做了任何修改都可以同步到远程代码库（push），同样你也可以把远程代码库做的任何修改同步到本地仓库（pull）。

仅仅针对面前的代码库而言，就包括三个区，工作区，暂存区，版本区，当下修改文件就是工作区，add之后就是将修改添加到暂存区，commit就是将暂存区的修改提交到版本区（对于当前代码库而言就是完成修改的最后一步了）。

这时如果要将我们本地仓库的修改同步到远程仓库，就需要将本地仓库的修改push到远程仓库，push到远程仓库后不是到达其工作区或者暂存区，即不必再执行一套远程仓库的的add和commit操作，而是直接到达远程代码库的版本区了。

远程仓库与本地仓库的关联也有两种方式，一种是本地先建立（init）代码库，然后再在远程（github）建立远程代码库，然后在本地运行一些命令将本地仓库与远程仓库直接关联。另外一种就是我们常见的，先在远程仓库建立远程仓库，然后通过clone命令克隆到本地，clone命令在本地创建的远程代码库的本地副本，这种方式是存在天然的关联关系的，不必再手动进行关联。

第一种方式的命令如下：
```txt
git remote add origin git@github.com:<GitHub账户名>/<代码仓库名>.git
```

有关回退命令
回退分为两种情况，一种是仅仅只在本地提交，并未同步到远程代码库，这种情况只需要执行get reset命令即可
```git
git reset <commit_id>           本地代码库回退到指定commit_id，其后做的修改也会恢复到工作区
git reset --soft <commit_id>    本地代码库回退到指定commit_id，其后做的修改也会恢复到暂存区
git reset --hard <commit_id>    本地代码库回退到指定commit_id，其后做的修改会被删除
```
如果已经同步到远程代码库，则需要使用git revert命令
```git
git revert <commit_id>
git push origin dev
```
两个步骤，第一步先回退本地，第二步将回退操作同步到远程仓库，保持两地一致

get revert 后面的commit_id表示要回退的提交，回退成功后该commit_id对应得修改将会被恢复

如果回退的commit_id是最近一次的提交，那么可以直接成功，但如果不是最近一次的提交，那么该命令会产生一个冲突，需要手动进行代码的恢复，然后执行git add,再执行git revert --continue来完成回退

二者的不同之处，reset会将要回退的提交（commit_id）删除，而revert不会，他是通过自动新建一个提交的方式来回退代码，因为会产生新的提交，所以需要push到远程

[三年 Git 使用心得 & 常见问题整理
](https://blog.csdn.net/liuyan19891230/article/details/106964749)

[三年 Git 使用心得 & 常见问题整理](https://juejin.cn/post/6844904191203213326)
