---
layout: git
title: Git基本命令
date: 2020-12-29 14:57:00
tags: Git
---

# Git 基本命令
## zsh 简写

`gco == git checkout`
`ga . == git add .`
`gcmsg "add log" == git commit -m “addlog”`
`git checkout -b feature-branch-name`  // 切分支

## 新建并切换仓库
```
    git init
    git add .
    git commit -m ‘first commit’
    git remote add origin http/s://github.com/yourgithubID/
    (git pull --rebase origin master）
    git push -u origin master #将本地仓库push到远程仓库
```

## diff操作

在git提交环节，存在三大部分：`working tree`, `index file`, `commit`

### 这三大部分中：
* working tree：就是你所工作在的目录，每当你在代码中进行了修改，working tree的状态就改变了。
* index file：是索引文件，它是连接working tree和commit的桥梁，每当我们使用git-add命令来登记后，index file的内容就改变了，此时index file就和working tree同步了。
* commit：是最后的阶段，只有commit了，我们的代码才真正进入了git仓库。我们使用git-commit就是将index file里的内容提交到commit中。

### 总结：

```
git diff //查看 working tree 与 index file 的差别的。
git diff --cached //查看 index file 与 commit 的差别的。
git diff HEAD // 查看working tree和commit的差别的。（HEAD代表的是最近的一次commit的信息）

//git diff 旧 新
git diff ea83556 a0553e3
git diff HEAD~1 HEAD
git diff HEAD~1
    
```

## git merge 操作
https://blog.walterlv.com/post/git-merge-strategy.html#patience

### git fatch

```
git fetch origin master //从远程主机的master分支拉取最新内容 （不加master就是全部库）
git merge FETCH_HEAD    //将拉取下来的最新内容合并到当前所在的分支中

```
## git 回滚操作
### 场景1：未 add
撤回修改
```
git checkout .
```
删除没有 git add 的文件
```
git clean -d fx
```

### 场景2：已经 commit，但是未 push 到远端
#### 回退到上一个版本 

```
git reset --hard HEAD~
git reset --hard HEAD~2 //具有破坏性
git reset --mixed HEAD~2 // 有所保留
```

#### 更改 commit 内容（未 push）
```
git commit --amend
```

### 场景3：已经 push
```
git push -u origin master -f
```

### 场景4：想要把 cf2e245 嫁接到某个分支目录下：

```
git checkout feat/xxx
git cherry-pick cf2e245
```

## git checkout 操作

```
* git checkout -- filename // 撤销 filename 上次修改操作
```

## git branch 操作
* 拉取指定分支内容
    * git clone -b dev
    * git fetch origin dev
    * git checkout -b dev origin/dev
* 删除本地分支
    * git branch -D branchName
* 删除远端分支
    * git push origin --delete branchName
    * git push origin :branchName
* 查看本地和远程分支
    * git branch -a
* 强制更新个人分支
    * git reset --hard origin/develop
    * git push origin yourbranch --force
    
## git 删除文件
* git rm -r --cached 文件/文件夹名称
## git tag
* 新建 tag
    * 可添加描述： git tag <tagname> -a
    * 无描述：git tag <tagname>
* 查看 tag
    * git tag

## git 暂存
思想：
* 可以暂存到工作区，并恢复该分支到上一个commit后的状态。
* 可以再任意分支 pop 出来并继续更改。

```
git stash
git stash save "test-cmd-stash"  // 给 stash 加 message
git stash pop  // 第一个stash删除，并将对应修改应用到当前的工作目录
git stash apply // 不删除
git stash list // 查看缓存列表
git stash drop stash@{0} // 删除某stash
git stash pop stash@{1}  // 恢复指定stash
```
## git merge 和 git rebase
### git pull

```
git pull = git fetch + git merge FETCH_HEAD 
git pull --rebase =  git fetch + git rebase FETCH_HEAD 
```

### merge 和 rebase
现在我们有这样的两个分支,test和master，提交如下：

```
   D---E test
  /
A---B---C---F--- master
```
在master执行git merge test,然后会得到如下结果：

```
   D--------E
  /           \
A---B---C---F---G--- test, master
```
在master执行git rebase test，然后得到如下结果：

```
A---B---D---E---C‘---F‘--- test, master
```
merge操作会生成一个新的节点，之前的提交分开显示。
而rebase操作不会生成新的节点，是将两个分支融合成一个线性的提交。

## 多个 Git 切换
举例：gitlab 和 github 切换使用，配置步骤如下

### 生成多个密钥
* 先假设我有两个账号，一个是 github 上的，一个是公司 gitlab 上面的。首先为不同的账号生成不同的ssh-key
    * `ssh-keygen -t rsa -f ~/.ssh/id_rsa_work -c xxx@gmail.com` 然后根据提示连续回车即可在~/.ssh目录下得到id_rsa_work和id_rsa_work.pub两个文件，id_rsa_work.pub文件里存放的就是我们要使用的key 
    * `ssh-keygen -t rsa -f ~/.ssh/id_rsa_github -c xxx@gmail.com` 然后根据提示连续回车即可在~/.ssh目录下得到id_rsa_github和id_rsa_github.pub两个文件，id_rsa_gthub.pub文件里存放的就是我们要使用的key
* 把 id_rsa_xxx.pub 中的 key 添加到 github 或 gitlab 上，这一步在 github 或 gitlab 上都有帮助，不再赘述

### 添加密钥
* 默认 SSH 只会读取 id_rsa，所以为了让 SSH 识别新的私钥，需要将其添加到 SSH agent

```
ssh-add ~/.ssh/id_rsa_work
ssh-add ~/.ssh/id_rsa_github
```

### 新建 config 文件
* 新建 ~/.ssh/config，并写入以下内容，设定不同的git 服务器对应不同的key

config: 
```
# gitlab
Host workgit
	HostName workgit.cn
	User git
	IdentityFile ~/.ssh/id_rsa_work
# github
Host github
	HostName github.com
	User git
	IdentityFile ~/.ssh/id_rsa_github
```
### 切换 Git 命令
* 根据 config 配置的 `Host` 切换 git

```
ssh -T workgit
```

```
ssh -T github
```