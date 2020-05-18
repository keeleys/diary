title: git 常用命令
date: 2015-06-27 13:54:05
tags: git
categories: 运维优化
toc: true
---


#git 常用命令

## 查看配置

```
git config -l
```
##  添加别名配置
```
git config --global alias.ad "add --all"
git config --global alias.cm "commit -m 'commit' ""
git config --global alias.pb "push origin master" 
```

## 配置用户名邮箱
```
git config --global user.name "xxx"
git config --global user.email "xxx@gmail.com"
```


## 生成ssh
```
ssh-keygen -t rsa -C "xxx@gmail.com"
```
<!-- more -->
* * * 


## 撤消操作
### 修改最后一次提交
如果刚才提交时忘了暂存某些修改，可以先补上暂存操作，然后再运行 --amend 提交
```
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```
### 取消已经暂存的文件
假如已经commit了 还没有push 的时候 可以用

* 所有文件回到上次提交的状态,
```
$ git reset --hard HEAD
```
* 取消单个文件
```
git reset HEAD <file>
```
### 取消对文件的修改
对于已经commit过的文件，修改之后撤销它的修改
```
git checkout -- filename 
```

## 查看某个文件的修改历史

```
git log --pretty=oneline 文件名 # 显示修改历史
git show 356f6def9d3fb7f3b9032ff5aa4b9110d4cca87e # 查看更改
```

## 分支

```
git checkout -b new_branch #创建并切换分支
git branch #查看当前分支
git checkout master #切换到master分支
git merge new_branch #合并分支到当前分支
git branch -d new_branch #删除分支

```
## 查看修改
```
git status #查看当前状态 
git diff #对比修改
```


# .gitignore文件 

过滤不需要git托管的文件或目录
```
/target

*.bak
*.class
*.tmp
*.log
/bin
build.sh
integration-repo
build

# IDEA metadata and output dirs
*.iml
*.ipr
*.iws
*.idea
out
*.db
.svn
.classpath
.project
.settings
```

