---
layout: post
title: "War Game for Hacker"
date: 2015-03-14 18:58:35 -0500
description: code game, hacker game
comments: true
categories: Security
published: false

---

##Bandit
###Level0

使用给定的用户名和密码SSH登录到固定主机中。

username: bandit0, password: bandit0, host: bandit.labs.overthewire.org。

```
XiaoMing:~ minggong$ ssh bandit0@bandit.labs.overthewire.org
The authenticity of host 'bandit.labs.overthewire.org (178.79.134.250)' can't be established.
RSA key fingerprint is 85:e5:7b:e2:87:74:18:c5:f4:d0:50:9f:13:56:9c:a7.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'bandit.labs.overthewire.org,178.79.134.250' (RSA) to the list of known hosts.
bandit0@bandit.labs.overthewire.org's password:

```

<!-- more -->
###Level0 - Level1
在当前用户目录中查找readme文件，读取起存储的密码，然后作为 bandit1的密码登录到主机中。

```
bandit0@melinda:~$ find ~/ -name "readme"
/home/bandit0/readme
bandit0@melinda:~$ cat /home/bandit0/readme
boJ9jbbUNNfktd78OOpsqOltutMc3MY1

XiaoMing:~ minggong$ ssh bandit1@bandit.labs.overthewire.org
bandit1@bandit.labs.overthewire.org's password:
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.18.5-x86_64-linode52 x86_64)

```

###Level1 - Level2
找出home目录下名为`-`的文件，下一关的密码存储在这个文件中。

```
bandit1@melinda:~$ ls
-
bandit1@melinda:~$ cat ./-
CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9
```
###Level2 - Level3
利用上一关找到的密码，进入bandit2用户home目录下，找到`spaces in filename`文件中的通向下一关的密码。

```
XiaoMing:~ minggong$ ssh bandit2@bandit.labs.overthewire.org

bandit2@bandit.labs.overthewire.org's password:
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.18.5-x86_64-linode52 x86_64)

bandit2@melinda:~$ ls
spaces in this filename
bandit2@melinda:~$ cat "spaces in this filename"
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK

```

###Level3 - Level4
找到bandit3用户目录下`inhere`文件夹下的隐藏文件。

```
XiaoMing:~ minggong$ ssh bandit3@bandit.labs.overthewire.org

bandit3@bandit.labs.overthewire.org's password:
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.18.5-x86_64-linode52 x86_64)

bandit3@melinda:~$ ls
inhere
bandit3@melinda:~$ cd inhere/
bandit3@melinda:~/inhere$ ls -a
.  ..  .hidden
bandit3@melinda:~/inhere$ cat .hidden
pIwrPrtPN36QITSp3EQaw936yaFoFgAB

```
###Level4 - Level5
找到inhere文件夹下的可读文件，密码就藏在可读文件中。

```
bandit4@melinda:~$ ls
inhere
bandit4@melinda:~$ cd inhere/
bandit4@melinda:~/inhere$ ls
-file00  -file02  -file04  -file06  -file08
-file01  -file03  -file05  -file07  -file09
bandit4@melinda:~/inhere$ file ./-*
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
bandit4@melinda:~/inhere$ cat ./-file07
koReBOKuIDDepwhWk7jZC0RTdopnAYKh
bandit4@melinda:~/inhere$

```

###Level5 - Level6
密码藏在inhere文件夹中的可读，大小为1033b，无执行权限的文件中。

```
bandit5@melinda:~/inhere$ find . -type f -size 1033c ! -executable
./maybehere07/.file2
bandit5@melinda:~/inhere$ cat ./maybehere07/.file2
DXjZPULLxYr17uwoI01bNLQbtFemEgo7
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        bandit5@melinda:~/inhere$

```

###Level6 - Level7

找到密码文件，文件被用户bandit7拥有，被bandit6组拥有，文件大小 33 bytes。

```
bandit6@melinda:~$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
bandit6@melinda:~$ cat /var/lib/dpkg/info/bandit7.password
HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs

```
其中 2>/dev/null 表示不显示错误信息到终端。参考[/dev/null](http://blog.csdn.net/ithomer/article/details/9288353)的解释。

### Level7 - Level8

密码藏在data.txt文件的`millionth`单词后面。

```

MEiyoMeiyoumia;0~!@#$
```
