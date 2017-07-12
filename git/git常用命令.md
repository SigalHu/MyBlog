![](git常用命令/1.png)

### 获取与创建项目

#### git init
```bash
# 在当前目录新建一个Git代码库
$ git init

# 在指定目录初始化为Git代码库
$ git init [project-name]
```
#### git clone
```bash
# 下载一个项目和它的整个代码历史到当前路径
$ git clone [url]

# 下载一个项目和它的整个代码历史到指定目录
$ git clone [url] [project-name]
```

### 配置

Git的设置文件为`.gitconfig`，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

#### git config
```bash
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```

### 暂存区操作

#### git add
```bash
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .
```
#### git rm
```bash
# 删除暂存区中的指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 在暂存区与工作区中删除指定文件
$ git rm -f [file]
```
#### git mv
```bash
# 在暂存区与工作区中重命名或移动文件
$ git mv [file-original] [file-renamed]
```

### 提交操作

#### git commit
```bash
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区所有已跟踪文件的变化到仓库区
$ git commit -am [message]

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

### 撤销操作

#### git checkout
```bash
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .
```
#### git reset
```bash
# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [HEAD] [file1] [file2] ...

# 重置暂存区的指定文件夹，与上一次commit保持一致，但工作区不变
$ git reset HEAD [dir]

# 重置暂存区，与上一次commit保持一致，但工作区不变
$ git reset [HEAD]

# 重置当前分支的HEAD为指定commit，同时重置暂存区，与指定commit保持一致，但工作区不变
$ git reset [commit]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
# 慎用，指定commit之后的提交会消失
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]
```
#### git revert
```bash
# 撤销上次提交，并把这次撤销作为一次最新的提交
$ git revert HEAD

# 撤销指定的版本，撤销也会作为一次提交进行保存
$ git revert [commit]
```

### 贮藏操作

#### git stash
```bash
# 备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中
$ git stash

# 从Git栈中读取最近一次保存的内容，恢复工作区的相关内容
$ git stash pop

# 读取指定版本号为stash@{1}的保存内容，恢复工作区的相关内容
$ git stash apply stash@{1}

# 显示Git栈内的所有备份，可以利用这个列表来决定从那个地方恢复
$ git stash list

# 清空Git栈
$ git stash clear
```

### 信息查看

#### git status
```bash
# 显示文件信息变更，-s参数输出简短结果
$ git status [-s]
```
#### git diff
```bash
# 显示工作区相比于暂存区中文件的改动
$ git diff

# 显示摘要而非改动的详细信息
$ git diff --stat

# 查看暂存区中指定文件相比于上次提交的改动
$ git diff --cached [file]

# 查看工作区文件相比于上次提交的改动
$ git diff HEAD

# 显示当前分支与其他分支之间的差异
$ git diff [other-branch]

# 显示两个分支之间的差异
$ git diff [first-branch] [second-branch]

# 比较上次提交commit和上上次提交
$ git diff HEAD HEAD^

# 显示两次提交之间的差异
$ git diff [first-commit] [second-commit]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"
```
#### git log
```bash
# 显示当前分支的版本历史
$ git log

# 显示指定分支的版本历史
$  git log [branch]

# 显示在第一个分支且不在第二个分支的提交信息
$ git log [first-branch] ^[second-branch]

# 显示当前分支的简洁版本历史
$ git log --oneline

# 显示commit历史，以及标签信息
$ git log --decorate

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 显示commit历史，以及每次commit修改的内容
$ git log -p

# 以拓扑结构显示当前分支的简洁版本历史
$ git log --oneline --graph

# 显示指定用户过去5次的提交日志
$ git log --author=[author] --oneline -5

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]
```
#### git shortlog
```bash
# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn
```
#### git blame
```bash
# 显示指定文件是什么人在什么时间修改过
$ git blame [file]
```
#### git show
```bash
# 显示指定提交的元数据和内容变化
$ git show [commit]

# 显示指定标签的元数据和内容变化
$ git show [tag]

# 显示指定提交发生变化的文件
$ git show --name-only [commit]

# 显示指定提交的指定文件内容
$ git show [commit]:[filename]

# 显示上次提交的元数据和内容变化
$ git show [HEAD]

# 显示上上次提交的元数据和内容变化
$ git show HEAD^

# 显示倒数第3次提交的元数据和内容变化
$ git show HEAD~2
```

### 分支管理

#### git branch
```bash
# 新建分支
$ git branch [branch-name]

# 新建一个分支，指向指定commit
$ git branch [branch-name] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch-name] [remote-branch]

# 在现有分支与指定的远程分支之间建立追踪关系
$ git branch --set-upstream-to=origin/[remote-branch] [branch-name]

# 删除指定分支
$ git branch -d [branch-name]

# 删除与指定远程分支的追踪关系
$ git branch -dr origin/[remote-branch]

# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a
```
#### git checkout
```bash
# 查看当前分支信息
$ git checkout

# 切换到指定分支
$ git checkout [branch]

# 新建分支并切换到该分支
$ git checkout -b [branch]
```
#### git merge
```bash
# 合并指定分支到当前分支
$ git merge [branch]
```
#### git cherry-pick
```bash
# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]
```
#### 冲突解决

在 Git 中，可以用 git add 要告诉 Git 文件冲突已经解决
```bash
# 修改冲突文件
$ vim [file]

# 提交修改文件
$ git add [file]
$ git commit
```

### 标签操作

如果你达到一个重要的阶段，并希望永远记住那个特别的提交快照，你可以使用git tag给它打上标签。比如说，我们想为项目发布一个"1.0"版本。我们可以用`git tag -a v1.0`命令给最新一次提交打上（HEAD）"v1.0"的标签。

#### git tag
```bash
# 创建标签并指定标签信息
$ git tag -a v1.0 -m [message]

# 给指定提交创建标签并指定标签信息
$ git tag -a v0.9 -m [message] [commit]

# 查看所有标签
$ git tag

# 删除指定标签
$ git tag -d v1.0
```

### 远程仓库

#### git remote
```bash
# 给远程仓库指定别名
$ git remote add [shortname] [url]

#
$ git remote show origin
```
#### git push
```bash
# 将本地指定分支推送到指定远程分支(若不存在则新建)
$ git push origin [branch-name]:[remote-branch]

# 将本地指定分支推送到指定远程分支(若不存在则新建)，同时指定origin为默认主机，并建立追踪关系
$ git push origin -u [branch-name]:[remote-branch]

# 将本地指定分支推送到同名远程分支(若不存在则新建)
$ git push origin [branch-name]

# 将当前分支推送到存在追踪关系的远程分支
$ git push

# 不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机
$ git push --all

# 如果远程主机的版本比本地版本更新，推送时Git会报错，要求先在本地做git pull合并差异，然后再推送到远程主机。这时，如果你一定要推送，可以使用-–force选项
$ git push --force

# 删除远程分支
$ git push origin --delete [branch-name]
# 推送一个空分支到远程分支，相当于删除远程分支
$ git push origin :[branch-name]

# 推送本地tag
$ git push --tags

# 获取远程tag
git fetch origin tag [tag-name]

# 删除远程tag
$ git push origin --delete tag [tag-name]
# 推送一个空tag到远程tag，相当于删除远程tag
$ git push origin :refs/tags/[tag-name]
```

**参考链接**

[常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)</br>
[菜鸟教程--Git 教程](http://www.runoob.com/git/git-basic-operations.html)</br>
[git常用命令解释](https://wenku.baidu.com/view/5a3f580fcf84b9d528ea7a69.html)</br>
[Git push 常见用法](http://www.cnblogs.com/qianqiannian/p/6008140.html)</br>
[Git 远程分支常用管理--查看+删除+重命名](https://my.oschina.net/kimcerry/blog/702980)</br>
[Git查看、删除、重命名远程分支和tag](http://zengrong.net/post/1746.htm)</br>
[git常用命令之git push使用说明](http://blog.csdn.net/jo__yang/article/details/50972807)</br>
[Git Stash用法](http://www.cppblog.com/deercoder/archive/2011/11/13/160007.aspx)</br>
[git入门（5）-Git revert和git reset版本的回退](http://blog.csdn.net/codectq/article/details/50777934)
