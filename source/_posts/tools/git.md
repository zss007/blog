---
title: tools 之 git
categories:
- tools
---
现在项目托管一般是使用 git，而之前则是 svn，那么两者之前的区别在哪呢。git 为什么能取代 svn，成为代码托管的首选呢。
其实 svn 是集中式版本管理，需要一个中央服务器，每次工作的时候先从服务器获取最新数据，然后提交自己的修改到服务器，而集中式版本管理最大的毛病是必须联网才能工作，而且提交一个较大的文件需要特别多的时间。而 git 则是分布式版本管理，每个人的电脑都是一个版本库，通常需要一台充当 "中央服务器" 的电脑，作用仅限于方便交换修改，而且他的分支管理把 svn 甩在了后面。
<!--more-->
### 一、常用命令及工作流程
#### 1、创建版本库 
创建版本有两种情况，一种是本地新建的项目新建一个 git 代码库，另外一种则是从远程服务器下载项目(即 git 代码库)，创建成功后会有一个 .git 的隐藏文件，请不要去修改它，否则会破坏 git 仓库。
```
# 在当前目录新建一个 Git 代码库
$ git init
    
# 下载一个项目和它的整个代码历史
$ git clone [url]
```
#### 2、增加/删除文件   
这里的 add 和 rm 并不是提交到仓库上，而是提交到缓存区。
```
# 添加指定文件到暂存区
$ git add [file1] [file2] ...
    
# 添加指定目录到暂存区，包括子目录
$ git add [dir]
    
# 添加当前目录的所有文件到暂存区
$ git add .
    
# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...
```
#### 3、代码提交  
这里的 commit 才是把本地代码修改提交到仓库上。
```
# 提交暂存区到仓库区
$ git commit -m [message]
    
# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]
```
#### 4、分支    
```
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
    
# 合并指定分支到当前分支
$ git merge [branch]
    
# 删除分支
$ git branch -d [branch-name]
    
# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```
#### 5、查看信息  
```
# 显示有变更的文件
$ git status
    
# 显示当前分支的版本历史
$ git log
```
#### 6、远程同步  
``` 
# 显示某个远程仓库的信息
$ git remote show [remote]
    
# 增加一个新的远程仓库，并命名(为本地代码指定远程服务器)
$ git remote add [shortname] [url]
    
# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]
    
# 上传本地指定分支到远程仓库(提交本地仓库变化到服务器仓库中)
$ git push [remote] [branch]
    
# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force
    
# 推送所有分支到远程仓库
$ git push [remote] --all

# 添加远程仓库地址
git remote add origin [url]

# 修改远程仓库地址
git remote set-url origin [url]

#从远程服务器上获取分支信息并更新到本地(有时同事在服务器上新增、删除分支，但是本地看不到分支变化，可用该命令从服务器拉取分支信息刷新本地分支)
git remote update origin --prune
```
#### 7、撤销
```
# 重置暂存区的指定文件，与上一次 commit 保持一致，但工作区(即当前电脑看到的目录)不变
$ git reset [file]
    
# 重置暂存区与工作区，与上一次 commit 保持一致
$ git reset --hard
    
# 重置当前分支的指针为指定 commit，同时重置暂存区，但工作区不变
$ git reset [commit]
    
# 重置当前分支的 HEAD 为指定 commit，同时重置暂存区和工作区，与指定 commit 一致
$ git reset --hard [commit]
    
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]
    
# 暂时将未提交的变化移除
$ git stash
```
### 二、.gitignore
如果我们有些文件不需要提交到远程服务器，比如执行特定命令后生成的文件，这个时候 .gitignore 文件就可以用来配置这些。
```
# 1、忽略目录 fd1 下的全部内容；注意，不管是根目录下的 /fd1/ 目录，还是某个子目录 /child/fd1/ 目录，都会被忽略；
fd1/*

# 2、忽略根目录下的 /fd1/ 目录的全部内容；
/fd1/*

# 3、忽略全部内容，但是不忽略 .gitignore 文件、根目录下的 /fw/bin/ 和 /fw/sf/ 目录；
/*
!.gitignore
!/fw/bin/
!/fw/sf/
```
### 三、.editorconfig
既然我们这里是讲 git，为什么需要讲到 .editorconfig，它只是与编码规范有关。
让我们先来了解下它吧。当多人开发共同开发一个项目时，往往会使用不同的编辑器，比如 Sublime、Atom、Webstorm 等，而且不同人会在不同系统环境下开发，这就会导致一个问题，那就是不同编辑器编码规范是不同的，而且不同系统环境下也不一样，很有可能就会导致代码冲突，而 .editorconfig 则是用来统一配置这些。如果编译器不支持，可能需要一个 editorconfig 插件。以 Webstorm 为例，点开 Preferences，点击 Plugins，搜索 EditorConfig，如果不存在则到 repositories 下载。常用编码规范如下：
```
# http://editorconfig.org
root = true

# 对所有文件生效
[*]
# 文件编码
charset = utf-8
# 缩进类型
indent_style = space
# 缩进数量
indent_size = 2
# 换行符格式
end_of_line = lf
# 是否在文件的最后插入一个空行
insert_final_newline = true
# 是否删除行尾的空格
trim_trailing_whitespace = true

[*.md]
insert_final_newline = false
trim_trailing_whitespace = false
```
### 四、补充
Git 会忽略空的文件夹。如果你想版本控制包括空文件夹，根据惯例会在空文件夹下放置 .gitkeep 文件。其实对文件名没有特定的要求，一旦一个空文件夹下有文件后，这个文件夹就会在版本控制范围内。