---
title: Git版本控制
categories: 数据开发
tags:
  - Git
cover: 'https://bu.dusays.com/2023/06/13/648837aa5e334.png'
abbrlink: ca9298d7
date: 2023-06-13 17:31:41
---

# Git版本控制

## 什么是Git

`Git` 是一个开源的分布式`版本控制`系统,用于敏捷高效地处理任何或小或大的项目

- 可以实现软件的版本控制,在多个版本之间切换
- 可以实现多分支开发,提高开发效率
- 可以实现历史记录,历史状态的恢复
- 团队内部实现权限管理等

## Git工作流程

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114174552714.png" alt="Git工作流程" style="zoom:50%;" />

`Git`的工作流程分为四个模块:

- 工作目录:存放我们正在写的代码（当我们新版本开发完成之后,就可以进行新版本的提交）

- 暂存区:暂时保存待提交的内容（新版本提交后会存放到本地仓库）

- 本地仓库:位于我们电脑上的一个版本控制仓库（存放的就是当前项目各个版本代码的增删信息）

- 远程仓库:位于服务器上的版本控制仓库（服务器上的版本信息可以由本地仓库推送上去,也可以从服务器抓取到本地仓库）

## 安装Git

可以前往 Git 的官网进行下载安装 [https://git-scm.com](https://git-scm.com/download/win)

在安装完成后,需要设定全局用户名和邮箱来区分不同的用户:

```bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

## Git的基本命令介绍

### 创建本地仓库

#### 初始化本地仓库

```bash
git init
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114175659292.png" alt="image-20230114175659292" style="zoom:50%;" />

​	输入后,会自动生成一个`.git`目录,这个目录是一个隐藏目录,而当前目录就是我们的工作目录

#### 查看本地仓库状态

```bash
git status
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114175724136.png" alt="image-20230114175724136" style="zoom:50%;" />



### 添加和提交

#### 添加文件

​	这里我们添加一个文档,使用`git status`查看一下会发生什么

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114180000378.png" alt="image-20230114180000378" style="zoom:50%;" />

​	这里提示存在尚未跟踪的文件,意思就是`Git`不会记录它的变化,而是始终将其当做一个新创建的文件

​	这里我们将其添加到暂存区,那么它会自动变为被跟踪状态,`Git`就会记录它的变化

```bash
git add README.md
git status
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114180600586.png" alt="image-20230114180600586" style="zoom:50%;" />

​	现在文件名称的颜色变成了绿色,并且是处于`需要提交的变更`下面,因此,我们的`README.md`现在已经被添加到暂存区了

#### 提交文件

​	接下来我们来尝试将`README.md`文件提交到Git本地仓库中,这里注意需要输入提交的描述以便后续查看

​	例如你这次提交修改了或是新增了哪些内容

```bash
git commit -m 'add README.md'
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114180920120.png" alt="image-20230114180920120" style="zoom:50%;" />

​	接下来我们就可以查看我们的提交记录了

```bash
git log
git log --graph
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114181052671.png" alt="image-20230114181052671" style="zoom:50%;" />

​	或者说查看最近一次变更的详细内容,也可以查看指定的提交记录

```bash
git show 
git show [commit ID]
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114181129208.png" alt="image-20230114181129208" style="zoom:50%;" />

​	再次查看当前状态,发现已经是处于清空状态了

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114181310394.png" alt="image-20230114181310394" style="zoom:50%;" />

​	接下来我们可以尝试修改一下我们的`README.md`文件,由于当前文件已经是被跟踪状态

​	那么`Git`会去跟踪它的变化,如果说文件发生了修改,那么我们再次查看状态会得到下面的结果

```bash
echo "some change" >> README.md
git status
```
​	这里的`git add .`的意思是将当前目录下的所有文件添加到缓存区

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114181621724.png" alt="image-20230114181621724" style="zoom:50%;" />

​	因此现在`README.md`文件是处于已修改状态,我们如果修改好了,就可以提交我们的新版本到本地仓库中

```bash
git add .
git commit -m "some change in README.md"
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114181838495.png" alt="image-20230114181838495" style="zoom:50%;" />

接下来我们来查询一下提交记录,可以看到一共有两次提交记录

```bash
git log --graph
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114181955267.png" alt="image-20230114181955267" style="zoom:50%;" />

#### `.gitignore`文件

​	我们可以创建一个`.gitignore`文件来确定一个文件忽略列表,如果文件忽略列表中的文件存在且不是被跟踪状态的话

​	那么`Git`不会对其进行任何检查,同时也不会进行跟踪

```bash
vim .gitignore
```

```bash
# 这样就会匹配所有以txt结尾的文件
*.txt
# 虽然上面排除了所有txt结尾的文件,但是这个不排除
!666.txt
# 也可以直接指定一个文件夹,文件夹下的所有文件将全部忽略
test/
# 目录中所有以txt结尾的文件,但不包括子目录
xxx/*.txt
# 目录中所有以txt结尾的文件,包括子目录
xxx/**/*.txt
```

### 回滚

当我们想要回退到过去的版本时,就可以执行回滚操作,执行后可以将工作空间的内容恢复到指定提交的状态

```bash
git reset --hard 6c72fcf
git reset --hard commitID
```

执行后,会直接重置为那个时候的状态,再次查看提交日志,我们会发现之后的日志全部消失了

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114182959482.png" alt="image-20230114182959482" style="zoom:50%;" />

那么如果现在我又想回去呢？我们可以通过查看所有分支的所有操作记录

```bash
git reflog
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114183053812.png" alt="image-20230114183053812" style="zoom:50%;" />

这样就能找到之前的`commitID`,再次重置即可

## 分支

### 查看当前仓库存在的所有分支

```bash
git branch	
```

​	我们发现,默认情况下是有一个`master`分支的,并且我们使用的也是`master`分支

​	一般情况下master分支都是正式版本的更新,而其他分支一般是开发中才频繁更新的

​	我们接着来基于当前分支创建一个新的分支

### 创建分支

```bash
git branch test
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114192006328.png" alt="image-20230114192006328" style="zoom:67%;" />

### 删除分支

```bash
git branch -d test
```

### 切换分支

```bash
git checkout test
```

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/image-20230114192104583.png" alt="image-20230114192104583" style="zoom:67%;" />

### 合并分支

我们可以将两个分支更新的内容最终合并到同一个分支上

```bash
git checkout master
git merge test
```

### 变基分支

除了直接合并分支以外,我们还可以进行变基操作,它跟合并不同,合并是分支回到主干的过程,而变基是直接修改分支开始的位置

比如我们希望将`test`变基到`master`上,那么`test`会将分支起点移动到`master`最后一次提交位置

```bash
git checkout test
git rebase master
```

变基后的`test`分支相当于同步了此前`master`分支的全部提交

### 优选

我们还可以选择其将他分支上的提交作用于当前分支上,这种操作称为`cherrypick`

```bash
git cherry-pick <commit id>:单独合并一个提交
```

这里我们在`master`分支上创建一个新的文件,提交此次更新,接着通过`cherrypick`的方式将此次更新作用于test分支上

## 远程仓库

​	远程仓库实际上就是位于服务器上的仓库,它能在远端保存我们的版本历史,并且可以实现多人同时合作编写项目,每个人都能够同步他人的版本,能够看到他人的版本提交,相当于将我们的代码放在服务器上进行托管

### 远程账户认证和推送

```bash
git remote add 名称 远程仓库地址
git push 远程仓库名称 本地分支名称[:远端分支名称]
```

​	注意`push`后面两个参数,一个是远端名称,还有一个就是本地分支名称,但是如果本地分支名称和远端分支名称一致,那么不用指定远端分支名称,但是如果我们希望推送的分支在远端没有同名的,那么需要额外指定

​	推送前需要登陆账户,GitHub现在不允许使用用户名密码验证,只允许使用个人AccessToken来验证身份,所以我们需要先去生成一个Token才可以

​	推送后,我们发现远程仓库中的内容已经与我们本地仓库中的内容保持一致了,注意:远程仓库也可以有很多个分支

### 克隆项目

```bash
git clone 远程仓库地址
```

### 抓取、拉取和冲突解决

​	我们接着来看,如果这个时候,出现多个本地仓库对应一个远程仓库的情况下,比如一个团队里面,N个人都在使用同一个远程仓库,但是他们各自只负责编写和推送自己业务部分的代码,也就是我们常说的协同工作,那么这个时候,我们就需要协调

​	比如程序员A完成了他的模块,那么他就可以提交代码并推送到远程仓库,这时程序员B也要开始写代码了,由于远程仓库有其他程序员的提交记录,因此程序员B的本地仓库和远程仓库不一致,这时就需要有先进行pull操作,获取远程仓库中最新的提交

```bash
#抓取：只获取但不合并远端分支,后面需要我们手动合并才能提交
git fetch 远程仓库 
#拉取：获取+合并
git pull 远程仓库 
```