---
layout: post
title:  "开发经验漫谈 -- Git在开发流程中的运用"
---

这几期和大家分享下我自己在开发时的一些经验，这次来说说关于Git在开发流程中的运用。

## 目的

1. 保持提交历史整洁，俗称的一条线提交。
1. 让项目代码有迹可循，提供清晰的开发历程，亦称：取其精华，去其糟粕。
1. 帮助大家养成好的代码开发习惯，即：如何更好地和他人合作。
1. 提供一个思路，让大家更深刻的理解git。

## 请先花时间阅读

1. 如果你没有接触过git，请先阅读：http://git-scm.com/book/zh/v2
1. 必读：https://www.atlassian.com/git/tutorials/merging-vs-rebasing

## 示例系统

我们使用一个简单的项目作为示例来进行说明。

![sys](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/sys.png?raw=true)

该系统由用户管理模块和角色管理模块组成，该项目已经完成了角色模块的开发，现在在开发用户模块。

## 开发流程

#### 第一步：获取项目代码

```
ligang@vm-xubuntu ~/devspace $ git clone /home/ligang/repository/gitflow.git
Cloning into gitflow...
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (6/6), done.
```

当前的提交历史如下：

![init](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/init.png?raw=true)

#### 创建远程合作分支

我们把这个分支命名为user_admin

```
ligang@vm-xubuntu ~/devspace/gitflow $ git push -u origin user_admin
Total 0 (delta 0), reused 0 (delta 0)
To /home/ligang/repository/gitflow.git
 * [new branch]      user_admin -> user_admin
Branch user_admin set up to track remote branch user_admin from origin.
```

操作完成后，提交历史如下：

![user_admin_init](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/user_admin_init.png?raw=true)

说明：合作开发user_admin模块的同学，会在这个分支上合并彼此的代码。

#### 第三步：创建个人分支（真正的开发工作在这里进行）

我们把这个分支命名为：user_admin_ligang

```
ligang@vm-xubuntu ~/devspace/gitflow $ git push -u origin user_admin_ligang
Total 0 (delta 0), reused 0 (delta 0)
To /home/ligang/repository/gitflow.git
 * [new branch]      user_admin_ligang -> user_admin_ligang
Branch user_admin_ligang set up to track remote branch user_admin_ligang from origin.
```

![user_admin_ligang_init](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/user_admin_ligang_init.png?raw=true)

请注意：

1. 此分支是在远程合作分支（user_admin）的基础上创建的。
1. 此分支只可合并远程合作分支，不可以直接合并master。

### 分支合并流程

假设在user_admin_ligang的个人分支中已经完成了开发，现在需要把这部分代码提交给被人使用，那么请按照如下方式操作：

假设开发完成后的提交历史如下：

![user_admin_ligang_before_rebase](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/user_admin_ligang_before_rebase.png?raw=true)

#### 第一步：整理个人开发分支中待合并的提交，去掉无用的，仅保留有用的

###### 切到user_admin_ligang分支

```
ligang@vm-xubuntu  ~/devspace/gitflow $ git checkout user_admin_ligang
ligang@vm-xubuntu  ~/devspace/gitflow $ git branch
  master
  user_admin
* user_admin_ligang
ligang@vm-xubuntu  ~/devspace/gitflow $
```

###### 查找newbase

在这个清理过程中，我需要清理掉tmp1和tmp2这2个临时提交，所以newbase就是init，这里获得它的版本号：

![find_newbase](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/find_newbase.png?raw=true)
