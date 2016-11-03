---
title: 常用Git命令总结
date: 2016-09-20 19:52:42
categories: 工具
keywords: git, 版本管理,Android
tags: [Android,git]
---
http://www.ruanyifeng.com/blog/2014/06/git_remote.html

#### 删除远程分支
```
 git push origin :branch_name
 ```

#### 添加版本标签
```
 git tag -a 2.3.2 -m"description" // -a标签名字  -m 标签注释

 git push --tags

 删除标签的命令

 git tag -d 0.1.3

 删除远端服务器的标签

 git push origin :refs/tags/0.1.3
 ```

#### 本地仓库与远程主机管理
绑定
```
git remote add  
```
解绑
```
 git remote rm
 ```

重命名
> $ git remote rename <原主机名> <新主机名>

##### 放弃本地修改
```
git fetch --all

git reset --hard origin/master
```
