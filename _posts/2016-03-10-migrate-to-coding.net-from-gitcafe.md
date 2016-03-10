---
title: 墙内博客镜像从 Gitcafe 迁移到 Coding.net
layout: post
guid: urn:uuid:1c762fbd-16d9-4657-a1a7-5984a895f46b
comments: true
tags:
  - blog migrate
---

### 动机
前些日子折腾把博客同时托管到 Gitcafe 和 Github 时遇到不少问题，在谷歌的帮助下全部搞定。怎奈本次迁移又遇到不少之前的问题，还要继续搜来搜去。
故纪录下容易踩坑的地方，希望对大家有所帮助。

#### 我的 SSH 配置

```java
#============================== GitHub ===========================

#johnwatsondev
Host github-jwdev
    HostName github.com
    User git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_jwdev
    IdentitiesOnly yes

#============================== Coding.net ===========================

#johnwatsondev
Host coding-jwdev
    HostName git.coding.net
    User git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_coding_jwdev
    IdentitiesOnly yes
```

由于我自定义了私钥名称，所以需要添加到 `ssh-agent`。

```java
$ ssh-add ~/.ssh/id_rsa_jwdev
$ ssh-add ~/.ssh/id_rsa_coding_jwdev
```

现在分别测试连接到 Github 和 Coding。

```java
$ ssh -T git@github.com
$ ssh -T git@git.coding.net
```

别忘了在 “系统偏好设置”–>”共享”–>”远程登录“ 中打开远程链接

### 为本地仓库设置多个 remote

在本地仓库的 `.git/config` 中添加如下设置：

```java
[remote "github"]
          url = git@github-jwdev:johnwatsondev/johnwatsondev.github.io.git
              fetch = +refs/heads/*:refs/remotes/origin/*
  [remote "coding"]
          url = git@coding-jwdev:johnwatsondev/johnwatsondev.git
              fetch = +refs/heads/*:refs/remotes/coding/*
  [remote "all"]
          url = git@github-jwdev:johnwatsondev/johnwatsondev.github.io.git
          url = git@coding-jwdev:johnwatsondev/johnwatsondev.git
  [alias]
          pushall= !sh -c \"git push github master:master && git push coding master:coding-pages\"
```

#### 容易混淆的点

分支分为三种：本地分支、远程分支、为本地设置的远程跟踪分支。

我们上面的 fetch 中添加的就是第三种。

请大家用 `git branch -r` 命令查看。

### 反思总结
这次之所以又在同样的问题上卡壳，根本在于上次只是照猫画虎式的快速解决问题，没有彻底弄清楚一些命令的用法和含义，导致解决重复问题。

### 参考资源
[git-add-remote-branch](https://stackoverflow.com/questions/11266478/git-add-remote-branch)  
[delete-a-git-branch-both-locally-and-remotely](https://stackoverflow.com/questions/2003505/delete-a-git-branch-both-locally-and-remotely)  
[Mac下搭建Hadoop环境](http://arccode.net/2014/07/13/Mac%E4%B8%8B%E6%90%AD%E5%BB%BAHadoop%E7%8E%AF%E5%A2%83/)
