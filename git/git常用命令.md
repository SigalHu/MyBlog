![](git常用命令/1.png)

### 获取与创建项目

```bash
# 在当前目录新建一个Git代码库
$ git init

# 在指定目录初始化为Git代码库
$ git init [project-name]

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

### 文件操作

```bash
# 添加指定文件到暂存区
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
```

**参考链接**

[常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)</br>
[菜鸟教程--Git 教程](http://www.runoob.com/git/git-basic-operations.html)
