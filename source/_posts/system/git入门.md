---
title: git入门
date: 2022-08-16 14:16:25
categories: 系统
tags: git
---

[toc]

# 常用命令

```bash
# 克隆 -b指定分支克隆
git clone -b branch projectUrl
# 查看分支
# 不加参数表示查看本地分支，-r查看远程分支 -a查看所有分支
git branch
# 切换分支,-b创建并切换到该分支
git checkout branchName
```

## 查看分支来源

```bash
git reflog show branchName
git reflog --date=local | grep branchName
```

## 全局配置

```bash
# 查看全局配置
git config --global -l
```



### 配置账号

> 配置账号以后，每次pull、push或clone时就不用输入账号密码了。

```bash
# 保存账号密码
git config --global credential.helper store
# 重置账号密码
git config --global credential.helper reset
```

## 远程仓库

```bash
# 查看远程仓库
git remote
# 获取远程仓库地址
git remote get-url <name>
# 添加远程仓库
git remote add <name> <url>
# 提交到远程仓库
git push <repository> <refspec> # git push origin hexo 将hexo分支提交到origin仓库
# 更新远程仓库分支列表
git remote update origin -p
```

## 回退版本

### revert

语法：`git revert -n commitId/HEAD~1`;

`-n`表示`revert`后不提交commit信息，不使用`-n`的话会弹出编辑器用户编辑提交信息。

### reset

语法：`git reset [--soft | --mixed | --hard] [HEAD]`

`--mixed`表示回退`commit`和`index`，即执行后源代码保留，但修改的代码没有添加到暂存区，需要执行`git add`命令将其添加到暂存区；该类型为默认类型，可不显示指定；

`--soft`表示回退`commit`，即执行后源代码保留，且修改的代码仍在暂存区；

`--hard`表示彻底回退，源代码不再保留；

`HEAD`或`HEAD~0`表示当前版本；

`HEAD^`表示回退到上个版本，`HEAD^^`表示回退到上上个版本，依次类推；

`HEAD~1`表示回退到上个版本，`HEAD~2`表示回退到上上个版本，依次类推；

```bash
# 回退到上个版本
git reset --hard HEAD^
# 回退到上上个版本
git reset --hard HEAD~2
# 回退到指定版本,commitId为commit的版本号，可通过git log查看
git reset --hard [commitID]
```

### revert和reset的区别

- git reset 是回滚到对应的commit-id，相当于是删除了commit-id以后的所有的提交，并且不会产生新的commit-id记录，如果要推送到远程服务器的话，需要强制推送-f
- git revert 是反做撤销其中的commit-id，然后重新生成一个commit-id。本身不会对其他的提交commit-id产生影响，如果要推送到远程服务器的话，就是普通的操作git push就好了



# 常见问题

## 切换分支时当前分支更新的代码会同步到切换的分支

**原因：**当前分支更新的代码没有加入到版本管理中。

**解决方法：**切换分支前先`commit`一下更新的代码。
