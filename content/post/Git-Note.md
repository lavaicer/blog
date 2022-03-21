---
title: "Git Note"
date: 2022-03-19T13:26:53+08:00
draft: false
weight: 9
categories: ["Git"]
tags: ["Git"]
author: ["lavaicer"]
# author: ["Me", "You"] # multiple authors
---
# Note OF Git

## 始

- 安装git

  搜索引擎
- 配置文件

  ```bash
  /etc/gitconfig 系统
  ~/.gitconfig 或 ~/.config/git/config 只针对当前用户 可传参 ‘global’让Git读写此文件
  .git/config 针对当前仓库的配置文件
  上面三个，越下面级别越高
  ```
- 配置修改

  ```bash
  - 用户信息
  git config --global user.name "John Doe"
  git config --global user.email johndoe@example.com
  查看配置信息：'git config --list'/'git config -l'

  - 设置/删除代理
  git config --global https.proxy http://127.0.0.1:1080
  git config --global https.proxy https://127.0.0.1:1080
  git config --global --unset http.proxy
  git config --global --unset https.proxy

  - 默认编辑器是vi，更改为vim
  git config --global core.editor "vim"

  - 设置终端带颜色
  git config --global color.ui true

  ```
- 4.帮助手册

  ```bash
  git help <verb>
  git <verb> --help
  man git-<verb>
  ```

---

## 基本命令

```bash
- 1.git init
    初始化git仓库
  
- 2.git status
    查看git仓库的状态
  
- 3.git add
    缓存添加文件（夹）到git仓库
  
- 4.git rm -cached
    移除添加的缓存
  
- 5.git commit -m 'something'
    commit 代表提交
    -m 表示提交附属信息，对此次提交的描述
  
- 6.git log
    查看提交日志
  
- 7.git branch
    查看当前分支	git branch
    新建分支a	git branch a
    新建a分支并切换到a分支	git checkout -b a
	删除无用的a分支 git branch -d a
	若a还没合并到main，则上一条会失败，-D可强制删除	git branch -D
  
- 8.git merge
    要先切换到main分支，然后git merge a就可将a合并到main分支
  
- 11.git tag
    添加标签，方便切换和查找
    查看标签 git tag
    切换 git checkout v1.0
```

---

## 链接远程仓库

### SSH

```bash
生成rsa秘钥 ssh-keygen -t rsa 
\ 生成的文件在 ~/.ssh 目录下，将pub中的内容复制到GitHub上。
\ 设置 >> SSh >> new SSH key 
\ 输入ssh -T git@github.com 进行测试是否添加成功
```

### push & pull

```bash
git push origin main 本地代码推到远程分支
git pull origin main 远程代码拉到本地
一般push之前会先pull
```

### 仓库克隆到本地

```bash
git clone + url
```

### 查看远程仓库

```bash
git remote -v
```

### git diff

```bash
git diff <$id1> <$id2>  比较两次提交之间的差异
git diff `<branch1>`..`<branch2>`  在两个分支之间比较
git diff --staged  比较暂存区和版本库差异

```
### git checkout tag/id/branch

可以撤销还没add进暂存区的文件

### git stash

代码还未commit之前暂时切换到另外的分支

```bash
git stash list 暂存区记录
git stash apply 还原
git stash drop 删除这次stash记录
git stash pop 还原并删除
git stash clear 清空

```

### merge&rebase

```bash
git checkout master
git merge featureA

```
等价于

```bash
git checkout master
git rebase featureA
rebase 会按时序合并

```
---

## git 分支合作

查看本地分支列表
`git branch`

查看远程分支列表
`git branch -r`

删除本地分支
`git branch -d develop`
`git branch -D develop (强制删除)`

删除远程分支
`git push origin :develop`

如果远程分支有个 develop ，而本地没有，你想把远程的 develop 分支迁到本地：

`git checkout develop origin/develop`

同样的把远程分支迁到本地顺便切换到该分支：

`git checkout -b develop origin/develop`

