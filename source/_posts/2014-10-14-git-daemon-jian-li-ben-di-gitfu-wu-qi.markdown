---
layout: post
title: "Git daemon 建立本地Git服务器"
date: 2014-10-14 10:51:41 +0800
comments: true
categories: git

---

平时项目中我们常遇到这样的需求，需要在自己电脑上clone台式机上的git代码，从而省略了向Github远程服务器上传代码的过程。免除了一些授权Access问题。幸好有Git Daemon神器：

**前提**

代码是通过Git托管的，如果使用SVN的话，多增加一个Git repo也是可以的。

**Step by Step**

1. 进入到自己工程所在的目录下，以Users/mingong/project/Sheldon为例：`cd  project/Sheldon`

2. 在Git项目根目录下开启Daemon服务器  `git daemon --base-path=. --export-all --reuseaddr --informative-errors  —verbose`

3. 使用ifconfig查看本机电脑内网IP地址：`ifconfig`，以我的电脑为例，查看到的IP地址为 `10.113.241.150`

4. `clone/pull`代码到另外一台电脑（两台电脑需在同一个内网下即可），为模拟需要，直接到我的Downloads目录下面模拟进行后续步骤。  
首先我需要在Doanloads文件夹中`clone project: git clone git://10.113.241.150/sheldon` ，
ls可以看到我的Downloads中已经成功clone了sheldon工程，clone了工程的下一步就是pull, 当A电脑上的代码变动之后，我需要直接更新代码，首先进入sheldon根目录中：`git pull git://10.113.241.150/`

5. 增加push权限  
在B电脑上clone代码后，B电脑上的伙伴修改完代码会需要将代码push回A电脑的工程中，
在第2步中开启的git daemon给可访问的客户端read权限，在需要开启write权限时，我们需要给git daemon增加 ` —enable=receive-pack`，执行命令如下，    
`git daemon --base-path=. --export-all --enable=receive-pack --reuseaddr --informative-errors  —verbose`  
There is one quirk: client can’t push into your active git branch. Before pushing, user on the server should change the branch, if client wants to push to this branch.  
客户端B不能直接提交master到当前活跃分支，所以可以在B新建一个分支，只要提交分支到A的repo中即可，由A决定是否进行分支merge.  

```
$ git checkout -b sharing
# made some changes ...
$ git commit -am "add sharing feature"
$ git push -u origin sharing
...
* [new branch]      sharing -> sharing
...


```

**在服务器端merge分支**

```
$ git checkout master
$ git merge sharing

```

至此，push工作也就基本完成了。Git 还是很强大的。


参考网址： [taming-the-git-daemon-to-quickly-share-git-repository](http://railsware.com/blog/2013/09/19/taming-the-git-daemon-to-quickly-share-git-repository/)