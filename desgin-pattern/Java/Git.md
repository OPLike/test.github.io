# Git

## 一、Git基础

### 1、Git简介

Git是一种分布式版本控制系统

所谓分布式版本系统，就是以远程服务器上的仓库为基石在每个机器上创建一个完整的仓库，每个仓库独立控制其版本，最终汇总到远程仓库。



![image-20201229212250725](..\picture\20201229212252.png)

### 2、Git与SVN的区别

其与SVN不同，SVN是一种集中式版本控制系统。所谓集中式版本控制系统，就是将所有版本库集中存储中央仓库，本地无法控制版本。

无法控制其版本的缺点就是每次必须先联网从中央仓库得到最新版本，改动之后推送到中央仓库。

![kza3hza5ew](..\picture\kza3hza5ew.png)

### 3、Git工作原理

Git的四个区域

![image-20210324095716587](..\picture\image-20210324095716587.png)



基本的Git工作流程

![image-20210324100520720](..\picture\image-20210324100520720.png)



以上的工作原理可用下图表示

![图片描述](..\picture\59c31e4400013bc911720340.png)

### 4、Git分支

分支是为了将修改记录的整个流程分开存储，让分开的分支不受其它分支的影响，所以在同一个数据库里可以同时进行多个不同的修改。

![](..\picture\20201229221702.png)

Git 为我们自动创建的第一个分支，也叫主分支，一般其它分支开发完成后都要合并到 master

![](..\picture\20201229221800.png)



### 5、Git文件状态

在git中，文件主要有四种状态：

![](..\picture\20201229222227.png)



- **Untracked**: 未跟踪, 此文件在文件夹中, 但并没有加入到git库, 不参与版本控制. 通过`git add` 状态变为`Staged`.
- **Unmodify**（Committed): 文件已经入库, 未修改, 即版本库中的文件快照内容与文件夹中完全一致. 这种类型的文件有两种去处, 如果它被修改, 而变为`Modified`. 如果使用`git rm`移出版本库, 则成为`Untracked`文件
- **Modified**: 文件已修改, 仅仅是修改, 并没有进行其他的操作. 这个文件也有两个去处, 通过`git add`可进入暂存`staged`状态, 使用`git checkout` 则丢弃修改过, 返回到`unmodify`状态, 这个`git checkout`即从库中取出文件, 覆盖当前修改
- **Staged**: 暂存状态. 执行`git commit`则将修改同步到库中, 这时库中的文件和本地文件又变为一致, 文件为`Unmodify`状态. 执行`git reset HEAD filename`取消暂存, 文件状态为`Modified`



## 二、Git应用

### 1、首次提交代码到远程仓库

在本地工作区间创建新工程：newPro

在远程仓库创建新仓库：newPro

将工程newPro提交至远程仓库命令如下：

```shell
# 在当前工程目录创建本地版本库和git管理
git init 

# 添加README.md文件至缓存区
git add -A README.md

# 提交：将新增、修改、删除文件从缓存区提交至本地仓库保存，一般使用`git commit -am "说明的文字"`
git commit -m "first commit"

# 将本地仓库与远程仓库创建连接
git remote add origin https://github.com/newPro.git`将本地仓库与指定的远程仓库创建 联系；

# 将本地仓库代码推送至远程仓库，实际开发中，需要输入github 账号以及密码。（首次提交注意别遗漏`-u`指定默认主机）
git push -u origin master
```

![img](..\picture\211919426917967.jpg)

### 2、远程仓库更新到本地



更新版本可以选择update Project整个工程更新

也可以选择pull

![img](..\picture\211920094419072.jpg)



### 3、冲突解决

![img](..\picture\211933513474570.jpg)



![img](..\picture\211934044109593.jpg)

### 4、合并分支

最终我们的提交都是要合并到`master`分支的，首先切换到`master`分支，接着通过命令，`git merge develop`命令，将`develop`分支合并到`master`。

如果在合并的过程中出现冲突的情况，可以通过 `git status` 查看冲突的文件，手动解决冲突。



### 5、版本回退

假如说，我们发现这次提交不是我们想要的，可以通过 `git reset --hard HEAD^` 回退到上一次提交



### 6、打标签

假如我们要发布一个版本，我们通常会给这次提交打一个标签 `git tag publish/0.0.1`

可以通过`git tag`命令来查看我们打的标签。



## 三、Git常用命令

### 1、初始化仓库

```shell
    # 在当前目录新建一个Git代码库
    $ git init

    # 新建一个目录，将其初始化为Git代码库
    $ git init [project-name]

    # 下载一个项目和它的整个代码历史
    $ git clone [url]
```



### 2、配置

Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

```shell
    # 显示当前的Git配置
    $ git config --list

    # 显示 Git 的某一项配置
    $ git config <key>

    # 编辑Git配置文件
    $ git config -e [--global]

    # 设置提交代码时的用户信息，选择global即全局配置
    $ git config [--global] user.name "[name]"
    $ git config [--global] user.email "[email address]"
```



### 3、增加/删除文件

这里的增加/删除文件指的是向暂存区增加/删除文件。

```shell
    # 查看文件状态，查看当前工作区新增、更改或删除的文件
    $ git status
     
    # 添加指定文件到暂存区，可以添加多个文件，中间以空格隔开
    $ git add [file1] [file2] ...

    # 添加指定目录到暂存区，包括子目录
    $ git add [dir]

    # 添加当前目录的所有文件到暂存区
    $ git add .

    # 添加每个变化前，都会要求确认
    # 对于同一个文件的多处变化，可以实现分次提交
    $ git add -p

    # 删除工作区文件，并且将这次删除放入暂存区
    $ git rm [file1] [file2] ...

    # 停止追踪指定文件，但该文件会保留在工作区
    $ git rm --cached [file]

    # 改名文件，并且将这个改名放入暂存区
    $ git mv [file-original] [file-renamed]
    
    # 临时保存修改，可跨分支
    # save为可选项
    $ git stash [save message]
    # 所有保存的记录列表
    $ git stash list
    # 恢复工作进度到工作区，此命令的stash@{num}是可选项，在多个工作进度中可以选择恢复，不带此项则默认恢复最近的一次进度相当于git stash pop stash@{0}
    $ git stash pop [stash@{num}]
    # 恢复工作进度到工作区且该工作进度可重复恢复，此命令的stash@{num}是可选项，在多个工作进度中可以选择恢复，不带此项则默认恢复最近的一次进度相当于git stash apply stash@{0}
    $ git stash apply [stash@{num}]
    # 删除一条保存的工作进度，此命令的stash@{num}是可选项，在多个工作进度中可以选择删除，不带此项则默认删除最近的一次进度相当于git stash drop stash@{0}
    $ git stash drop stash@{num} 
    # 删除所有保存
    $ git stash clear
```



### 4、代码提交

```shell
    # 提交暂存区到仓库区，如果不加-m，会进入vim编辑器
    $ git commit -m [message]

    # 提交暂存区的指定文件到仓库区
    $ git commit [file1] [file2] ... -m [message]

    # 提交工作区自上次commit之后的变化，直接到仓库区
    $ git commit -a

    # 提交时显示所有diff信息
    $ git commit -v

    # 使用一次新的commit，替代上一次提交
    # 如果代码没有任何新变化，则用来改写上一次commit的提交信息
    $ git commit --amend -m [message]

    # 重做上一次commit，并包括指定文件的新变化
    $ git commit --amend [file1] [file2] ...
```



### 5、分支

```shell
    # 列出所有本地分支
    $ git branch

    # 列出所有远程分支
    $ git branch -r

    # 列出所有本地分支和远程分支
    $ git branch -a

    # 新建一个分支，但依然停留在当前分支
    $ git branch [branch-name]

    # 新建一个分支，并切换到该分支
    $ git checkout -b [branch]

    # 新建一个分支，指向指定commit
    $ git branch [branch] [commit]

    # 新建一个分支，与指定的远程分支建立追踪关系
    $ git branch --track [branch] [remote-branch]

    # 切换到指定分支，并更新工作区
    $ git checkout [branch-name]

    # 切换到上一个分支
    $ git checkout -

    # 建立追踪关系，在现有分支与指定的远程分支之间
    $ git branch --set-upstream [branch] [remote-branch]

    # 合并指定分支到当前分支
    $ git merge [branch]
   
    # 查看分支合并状态
    $  git rerere status
   
    # 显示合并冲突解决方案的当前状态——开始解决前与解决后的样子
    $  git rerere diff

    # 选择一个commit，合并进当前分支
    $ git cherry-pick [commit]
    
    #分支重命名
    $ git git branch -m [oldName]  [newName]
    # 删除分支
    $ git branch -d [branch-name]

    # 删除远程分支
    $ git push origin --delete [branch-name]
    $ git branch -dr [remote/branch]
```



### 6、标签

```shell
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```



### 7、查看信息

```shell
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示当前分支的最近几次提交
$ git reflog
```



### 8、远程同步

```shell
# 克隆远程仓库
$ git clone [url]

# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 拉取远程分支，同时创建本地分支
$ git fetch [remote] 远程分支名x:本地分支名x
# 拉取远程分支，同时创建本地分支，且切换到该分支
$ git checkout -b [branch] [origin/远程分支名]

# git pull相当于以下两步
# 1、拉取远程分支
$ git fetch [remote] [branch]
# 2、合并到当前分支
git merge [remote/branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all

# 更新远程分支列表
$ git remote update [remote] --prune
$ git remote update [remote] -p

# 删除和远程仓库的关联
$ git remote rm [remote]
```



### 9、撤销

```shell
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

#  回退到上一次提交
$ git reset --hard HEAD^
# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop
```







