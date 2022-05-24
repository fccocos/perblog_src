## Git配置

`git config`  配置或读取相应的工作环境变量

### 配置信息的存放位置

- `/etc/gitconfig`文件
  - 该文件存放的是系统中所有用户都适用的配置信息
  - `git config --system` 这个命令就是对`/etc/gitconfig`文件进行读写操作
- `~/.gitconfig`文件
  - 这个文件是存放当前用户的配置信息
  - `git config --global`  这个命令就是对`~/.gitconfig`文件进行读写操作
- git一般情况下优先读写`~/.gitconfig`文件，然后再读取`/etc/gitconfig`文件。

### 配置用户信息

- `git config --global user.name "user_name"`
- `git config --global user.emial "your_email"`
- 使用上述两条命令，就是在你的配置文件中写入你的用户信息，此后所有的git项目都会默认的使用你的配置文件中的用户信息
- 当然你也可以单独的为你的每一个git项目配置用户信息，只需要将`--global`选项去掉即可, 此时新的设定将会保存在你的当前git项目的`.git/config`文件里



### 为git配置你熟悉的文本编辑器

`git config --global core.editor "your_editor_name"`

如果你的编辑器名字有空格，请用双引号""将其括起来

### 配置差异分析工具

用于解决合并冲突时的分析工具

`git config --global merge.tool "diff_tool_name"`

使用什么差异分析工具就使用它的名字即可,如`vimdiff`

git可以理解的差异分析工具有

-  `kdiff3`
- `tkdiff`
- `meld`
- `xxdiff`
- `emerge`
- `vimdiff`
- `gvimdiff`
- `ecmerge`
- `opendiff`

### 查看配置信息

`git config --list`

## Git工作区、暂存区和版本库

### 基本概念

- 工作区：本地git目录，即`.git`所在目录为git工作区
- 暂存区:  又叫索引区，存放于.git目录下的index文件中
- 版本库：工作区中的一个隐藏目录`.git`, `.git`不属于工作区，它仅仅是一个Git的版本库

### 工作区、暂存区和版本库的关系

1. 工作区和版本库是独立的两个部分，虽然版本库存在于工作区中
2. 版本库中包含了暂存区、对象库、分支等信息
3. 分支是一个目录树，用一个"HEAD"的游标指向这个分支
4. 对象库存放于`.git/objects`目录中，它包含了创建的各种对象以及内容

[![Og2FL4.png](https://s1.ax1x.com/2022/05/15/Og2FL4.png)](https://imgtu.com/i/Og2FL4)

#### `git add`的运行过程

当使用`git add`命令后，暂存区的目录树会被更新，同时工作区修改的文件内容会被写入到对象库中的一个新对象中，而该对象的ID被记录在暂存区的文件索引中

**将工作区修改的内容写入到暂存区**

#### `git commit`的运行过程

执行`git commit`命令时，暂存区的目录树写入到版本库（对象库）中，master分支会做相应的更新，即master指向的目录树就是提交时暂时缓存的目录树。

**将暂存区中的内容写入到master指向的分支**

#### `git reset HEAD`的运行过程

执行`git reset HEAD`命令时，暂存区的目录树会被重写，被master分支指向的目录所替换，但是工作区不受影响。

**将master分支中的目录树写入到暂存区中**





`git rm --cached <file>` **直接从暂存区删除文件，工作区则不做出改变**

`git checkout .`或`git checkout -- <file>`  **用暂存区全部或指定的文件替换工作区的文件**。这个操作很危险，会清除工作区中未添加到暂存区中的改动。

`git checkout HEAD .`或者`git checkout HEAD <file>` **用 HEAD 指向的 master 分支中的全部或者部分文件替换暂存区和以及工作区中的文件。**这个命令也是极具危险性的，因为不但会清除工作区中未提交的改动，也会清除暂存区中未提交的改动。

## Git创建仓库

```shell
# 初始化
git init #在当前目录中初始化一个仓库
git init repo_name #指定一个目录名创建仓库
# 克隆远程项目
git clone <远程仓库链接> [<存放目录路径>]
```

## Git基本操作

### Git常用命令

#### 创建仓库命令

```shell
# 初始化一个仓库
git init [<direction>]
# 拷贝一个远程仓库
git clone <repository-link> [<direction>]
```

#### 提交与修改

```shell
# 添加文件到暂存区
git add
git add * # 将工作区中的所有文件都添加到暂存区
git add . # 将工作区中修改过的文件添加到暂存区
git add <file> # 指定工作区的文件添加到暂存区

# 查看仓库当前状态，显示有变更的文件
git status

# 比较文件的不同，即暂存区和工作区的差异
git diff <file>

# 提交暂存区内容到分支中
git commit [-m "提示(promt)"]

# 回退版本 
git reset

# 删除工作区文件
git rm <file>

# 移动或重命名工作区文件
git mv <oldfile> <newfile>

```

#### 提交日志

```shell
# 查看日志记录
git log

# 以列表的形式查看指定文件的历史修改记录
git blame <file>
```

#### 远程操作

```shell
# 远程仓库操作
git remote

# 从远程获取代码库
git fetch

# 下载远程代码并合并
git pull

# 上传远程代码并合并
```

## Git分支管理

### 分支管理常用命令

```shell
# 创建分支
git branch <branch_name>
# 切换分支
git checkout <brach_name>
# 合并分支
git merge
```

### 分支管理操作

#### 列出分支

`git branch` 列出分支

例子：使用`git branch`命令创建一个分支并切换到新的分支上，提交当前修该的内容

```shell
# 创建分支
git branch newbranch
# 列出所有分支，查看是否分支创建成功
git branch
# 切换到新建分支中
git checkout newbranch

# 在当前工作区修改内容
# ...
# ...

# 提交修改到暂存区
git add .
# 将暂存区的内容提到newbranch中
git commit -m "promt"
```

我们也可以使用`git branch -b <branch_name>` 来创建一个新分支并切换到新建的分支上

#### 删除分支

`git branch -d <branch_name>` 删除分支

例子：删除上述添加的分支

```shell
git branch -d newbrach
```

#### 分支合并

`git merge <branch_name>` 合并分支

例子：将上述的`newbranch`分支合并到主分支中然后删除分支

```shell
# 列出当前分支
git branch
# 合并分支
git merge newbranch
# 删除nerbranch分支
git branch -d newbranch
# 列出分支，查看是否删除成功
git brach
```

#### 合并冲突的解决

```shell
# 合并产生冲突时，会提示我们是那些文件产生的合并冲突
# 因此我们可以根据提示打开文件，查看冲突内容,然后修改即可

cat 产生冲突的文件 # 查看冲突
vim 产生冲突的文件 # 修改文件
git diff # 查看文件修改前后的差异
git status -s # 查看git当前状态
git add 产生冲突的文件 # 告诉git冲突已经解决
git status -s # 查看当前状态
git commit # 提交到master分支


```

## 查看提交历史

1. `git log`
2. `git blame <file>`  以列表的形式查看指定文件的历史修改记录

```shell
git log --oneline（简洁） # 查看历史记录的简洁版本
git log --graph(拓扑图)  # 查看历史中什么时候出现分支、合并
git log --reverse --onlie # 逆向显示所有日志
git log --author="name" # 查找指定用户的提高日志
git log --oneline --before={3.weeks.ago} --after={2021-01-01} --no-merges # 查看三周以前，2021年01月01日之后的没有合并的日志
# 还有--until --since可用 
```

## Git标签

`git tag`就是给你当前项目的某一个阶段打标签

`git tag -a 注释` 创建一个带有注释的标签

`git log --decorate` 可以查看历史标签

`git tag -a 注释  提交的序列号`

`git tag` 查看所有标签

`git tag -a <tagname> -m "注释"` 指定标签信息

`git tag -s <tagname> -m "注释" ` PGP签名标签

## Github

```shell
# 生成ssh key
ssh-keygen -t rsa -C "your-email@example" # 邮件为关联了github的邮件
# 进入到生成的'~/.ssh'目录中将公钥id_rsa.pub中的内容复制到你的github上，即`SSH and GPG keys`中
# 验证是否成功
ssh -T git@github.com
# 初始化
git init
# 添加文件
git add .
# 提交
git commit -m "添加文件"

# 提交到github
git remote add origin git@github.com:github_user_name/reporsity.git
git push -u origin master

# 查看当前远程库
git remote
git remote -v 查看每个别名的实际地址

# 从远程仓库下载新分支与数据
git fetch # 该命令执行完后需要执行git merge 远程分支到你所在的分支
# 从远端仓库提取数据并尝试合并到当前分支
git merge
# 当你想要提取更新的数据的时候
git fetch [alias] # 告诉Git去获取它没有的数据
git merge [alias]/[branch] #将远程仓库的任何更新合并到你的当前分支

# alias为远程仓库的别名

# 将新分支和数据推送到你的远程仓库
git push [alias] [branch]

# 删除远程仓库
git remote rm [alias]

# 添加仓库
git remote add [alias] git@github.com:user_name/repo.git


```

## Github高级用法

### 搜索技巧

【s】-->聚焦搜索-->【enter】-->进入搜索界面-->点击`advanced search`-->进入高级搜索表单

### 查找仓库中的文件

【t】-->查找文件

【l】-->跳转到某一行

【b】-->查看文件修改记录

【C-v】-->打开控制面板

在仓库详情页按下【。】-->打开一个网页版的vscode来查看项目

在项目的链接中的github.com前加一个gitpod.io/#/来打开项目：此时会根据项目的类型自用添加环境依赖