![](git常用命令/1.png)

### 获取与创建项目

```bash
# 在当前目录新建一个Git代码库
$ git init

# 在指定目录初始化为Git代码库
$ git init [project-name]
```

```bash
# 下载一个项目和它的整个代码历史到当前路径
$ git clone [url]

# 下载一个项目和它的整个代码历史到指定目录
$ git clone [url] [project-name]
```

### 配置

Git的设置文件为`.gitconfig`，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

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

```bash
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .
```

```bash
# 删除暂存区中的指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 在暂存区与工作区中删除指定文件
$ git rm -f [file]
```

```bash
# 在暂存区与工作区中重命名或移动文件
$ git mv [file-original] [file-renamed]
```

### 提交操作

```bash
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
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

```bash
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .
```

```bash
# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [HEAD] [file1] [file2] ...

# 重置暂存区的指定文件夹，与上一次commit保持一致，但工作区不变
$ git reset HEAD [dir]

# 重置暂存区，与上一次commit保持一致，但工作区不变
$ git reset [HEAD]

# 重置暂存区，与指定commit保持一致，但工作区不变
$ git reset [commit]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]
```

### 信息查看

```bash
# 显示文件信息变更，-s参数输出简短结果
$ git status [-s]
```

```bash
# 显示尚未缓存的改动
$ git diff

# 显示摘要而非整个 diff
$ git diff --stat

# 查看已缓存的改动
$ git diff --cached [file]

# 查看已缓存的与未缓存的所有改动
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

```bash
# 显示当前分支的版本历史
$ git log

# 显示当前分支的简洁版本历史
$ git log --oneline

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

```bash
# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn
```

```bash
# 显示指定文件是什么人在什么时间修改过
$ git blame [file]
```

```bash
# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示上次提交的元数据和内容变化
$ git show [HEAD]

# 显示上上次提交的元数据和内容变化
$ git show HEAD^

# 显示倒数第3次提交的元数据和内容变化
$ git show HEAD~2
```

**参考链接**

[常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)</br>
[菜鸟教程--Git 教程](http://www.runoob.com/git/git-basic-operations.html)</br>
[git常用命令解释](https://wenku.baidu.com/view/5a3f580fcf84b9d528ea7a69.html)
