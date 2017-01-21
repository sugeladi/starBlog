title: git revert 和 git reset 的区别
tags:
  - git
categories: git
date: 2017-01-20 11:52:11
---
### 主要用于会退git版本

##### 在日常的工作中,我们偶尔会遇到需要在远程服务器上回退几个版本的需求,git reset和gitrevert 都可以处理这类需求,个人比较推荐使用git revert。
* ###### git reset 是删除本次提交，git revert是撤销某次commit并且会把这个撤销动作当成一次新的commit.
* ###### git reset是把HEAD向后移动了一下，而git revert是向前移动了一下
* ###### git reset时直接删除制定commit，而gitrevert是用新的commit来回滚之前的commit

#### 1.git reset

git reset 处理本地提交很好用，直接使用.


```
$ git reset --hard commit_id //commit_id 版本号 本地代码回退  
$ git push origin HEAD --force //强制提交到远程

```
或者

```
$ git reset --hard HEAR~1
$ git push --force //强制提交到远程

```

#### 2.git revert

```
$ git revert commit_id //commit_id 版本号 本地代码回退
$ git push

```