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

- 切到user_admin_ligang分支

```
ligang@vm-xubuntu  ~/devspace/gitflow $ git checkout user_admin_ligang
ligang@vm-xubuntu  ~/devspace/gitflow $ git branch
  master
  user_admin
* user_admin_ligang
ligang@vm-xubuntu  ~/devspace/gitflow $
```

- 查找newbase

在这个清理过程中，我需要清理掉tmp1和tmp2这2个临时提交，所以newbase就是init，这里获得它的版本号：

![find_newbase](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/find_newbase.png?raw=true)

- 执行清理

这一步是不可逆的，请谨慎操作，亦可先备份。

![user_admin_ligang_rebase_i](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/user_admin_ligang_rebase_i.png?raw=true)

这里按照提示，我们编辑rebase信息:

![user_admin_ligang_rebase_edit](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/user_admin_ligang_rebase_edit.png?raw=true)

保存退出，由于上面我们告诉rebase我们要重新编辑提交信息（r,reword），这里会进入提交信息编辑界面，我们修改最终提交信息如下：

![user_admin_ligang_rebase_reword](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/user_admin_ligang_rebase_reword.png?raw=true)

保存退出，清理过程结束，这里再次查看提交历史：

![user_admin_ligang_rebase_done](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/user_admin_ligang_rebase_done.png?raw=true)

这里可以看到，tmp1和tmp2已经被清理掉了，最终的done2是一个全新的提交。

#### 第二步：合并个人分支到远程合作分支

请确保此时只有你一个人操作远程合作分支

- 更新本地远程合作分支到最新

```
ligang@vm-xubuntu ~/devspace/gitflow $ git checkout user_admin
Switched to branch 'user_admin'
ligang@vm-xubuntu ~/devspace/gitflow $ git pull origin user_admin
From /home/ligang/repository/gitflow
 * branch            user_admin -> FETCH_HEAD
Already up to date.
```

- 切到user_admin_ligang分支

```
ligang@vm-xubuntu ~/devspace/gitflow $ git checkout user_admin_ligang
Switched to branch 'user_admin_ligang'
```

- 衍合个人开发分支

衍合前

![user_admin_before_rebase](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/user_admin_before_rebase.png?raw=true)

衍合

```
ligang@vm-xubuntu ~/devspace/gitflow $ git rebase user_admin
First, rewinding head to replay your work on top of it...
Applying: done2
Using index info to reconstruct a base tree...
M	show.txt
Falling back to patching base and 3-way merge...
Auto-merging show.txt
CONFLICT (content): Merge conflict in show.txt
error: Failed to merge in the changes.
Patch failed at 0001 done2
Use 'git am --show-current-patch' to see the failed patch

Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
```

这里有可能需要解决冲突，再continue完成整个衍合过程。

衍合后

![user_admin_after_rebase](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/user_admin_after_rebase.png?raw=true)

- 合并入远程合作分支

```
ligang@vm-xubuntu ~/devspace/gitflow $ git merge user_admin_ligang
Updating add0d89..9ed757b
Fast-forward
 show.txt | 1 +
 1 file changed, 1 insertion(+)
```

合并后的提交历史

![user_admin_merge](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/user_admin_merge.png?raw=true)

#### 删除个人开发分支

```
ligang@vm-xubuntu ~/devspace/gitflow $ git branch -d user_admin_ligang
Deleted branch user_admin_ligang (was 9ed757b).
ligang@vm-xubuntu ~/devspace/gitflow $ git push origin :user_admin_ligang
To /home/ligang/repository/gitflow.git
 - [deleted]         user_admin_ligang
```

## 合并到主干

#### 操作前确认

进行这一步操作前，请确认已经满足如下条件：

1. 请确认待合并的远程合作分支上的开发目标已全部完成。
1. 请确认即将把代码合并入主干进行发布上线。

如不能全部满足上述所有条件，请不要进行此操作。

#### 合并入主干

请确保此时只有你一个人操作远程分支及主干

- 更新本地主干到最新

```
ligang@vm-xubuntu ~/devspace/gitflow $ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
ligang@vm-xubuntu ~/devspace/gitflow $ git pull
Already up to date.
```

- 切到user_admin分支

```
ligang@vm-xubuntu ~/devspace/gitflow $ git checkout user_admin
Switched to branch 'user_admin'
```

- 衍合远程合作分支

```
ligang@vm-xubuntu ~/devspace/gitflow $ git rebase master
Current branch user_admin is up to date.
```

这里的过程和远程合作分支衍合时是一样的，有可能需要解决冲突，再continue完成整个衍合过程。

- 合并入主干

```
ligang@vm-xubuntu ~/devspace/gitflow $ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
ligang@vm-xubuntu ~/devspace/gitflow $ git merge user_admin
Updating 73180c1..9ed757b
Fast-forward
 show.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
```

- 删除远程合作分支

```
ligang@vm-xubuntu ~/devspace/gitflow $ git branch -d user_admin
Deleted branch user_admin (was 9ed757b).
ligang@vm-xubuntu ~/devspace/gitflow $ git push origin :user_admin
To /home/ligang/repository/gitflow.git
 - [deleted]         user_admin
```

## 最终的提交历史

![final](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-06-30-dev-exp-gitflow/final.png?raw=true)
