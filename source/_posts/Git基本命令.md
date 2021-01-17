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

```
git checkout .
```

### 场景2：已经 commit，但是未 push 到远端
#### 回退到上一个版本 

```
git reset --hard HEAD^
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
* 删除本地分支
    * git branch -D branchName
* 删除远端分支
    * git push origin --delete branchName
    * git push origin :branchName
* 查看本地和远程分支
    * git branch -a
    
## git 删除文件
git rm -r --cached 文件/文件夹名称
git commit -m "提交说明"
git push origin master

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

## 多个 Git 切换
* gitlab 和 github 切换使用

### 新建 ~/.ssh/config
* id_rsa 为 GitLab 密钥
* github_rsa 为 GitHub 密钥

```
# gitlab
Host gitlab
	HostName git.zerozero.cn
	User git
	IdentityFile ~/.ssh/id_rsa
# githab
Host github.com
	HostName github.com
	User git
	IdentityFile ~/.ssh/github_rsa
```
### 切换 Git 命令
```
ssh -T github.com
ssh -T github
```