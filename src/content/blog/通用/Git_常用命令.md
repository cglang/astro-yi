---
title: Git 常用命令
description: ""
date: 2020-11-04
tags:
---

### 创建本地版本库

```bash
git init
```

<!-- more -->

### 添加远程仓库

```bash
git remote add <remoteName> <仓库地址>
```

### 将修改添加到暂存区

添加指定文件或者是所有的文件；

```bash
git add {<filename1,...,filename2> | --all}
```

### 提交修改

将修改提交到版本库

```bash
git commit -m "commit message"
```

### 分支管理

查看分支；切换到指定分支；删除指定分支；查看远程分支；查看所有分支；

```bash
git branch [{<branchName> | -d <branchName> | -r | -v }]
```

创建分支；创建并切换分支；

```bash
git checkout { <branchName> | -b <branchName> }
```

创建分支；切换分支；

```bash
git switch { -c <branchName> | <branchName> }
```

合并指定分支到当前分支

```bash
git merge <branchName>
```

### 暂存区存储

存储暂存区修改；查看暂存区记录列表；恢复|删除|恢复并删除某次记录；

```bash
git stash { -m <"commitMeaasge"> | list | { apply | drop | pop } <stash@{0}> }
```

### 推送与拉取

推送默认仓库；推送到指定仓库；

```bash
git push [-u <remoteName>]
```

拉取；拉取指定分支并合并；

```bash
git pull [<远程主机名> <远程分支名>[:<本地分支名>]]
```

### 提交记录管理

查看提交记录 [单行模式] [指定条数]；查看提交记录简洁模式查看；

```bash
git log [--oneline] [-{n}]
git reflog [-{n}]
```

删除当前分支的最后 {n} 次提交；

```bash
git reset --hard HEAD~{n}
```


# Git 配置

> ~/.gitconfig 文件为全局配置；.git/config 文件为仓库配置；默认优先选择仓库配置；

#### 用户信息配置

```ini
[user]
	name = name
	email = email@email.com
```



#### 核心配置

```ini
[core]
	配置ssh提交时默认使用的秘钥
	sshCommand = ssh -i ~/.ssh/key
```