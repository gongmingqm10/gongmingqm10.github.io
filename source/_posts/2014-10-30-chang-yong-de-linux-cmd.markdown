---
layout: post
title: "常用的Linux Cmd"
date: 2014-10-30 15:47:40 +0800
comments: true
categories:
description: Linux常用命令

---

ls：查看目录下所有可见文件  
ls -a ： 查看目录下所有文件，包括隐藏文件    
ls -l : 查看目录下可见文件的详细信息  
ls -al 查看目录下所有文件的详细信息

<!-- more -->
cat file: 查看文件内容  
cat -n file: 查看文件内容，并显示出行号  
cat file1 file2 >> file3：将file1, file2的文件内容以此加到file3中，如果file3存在，则会在file3后面继续追加，不会覆盖file3原来的内容，如果file3不存在，则会自动创建。若需要强行覆盖，使用一个  > 即可  


touch file: 新建文件，当文件存在时则会更新文件的时间戳  

pwd: 查看当前目录  
cd {path} : 进入目录    

mkdir {directory}: 新建目录，若某子目录不存在则可能新建失败  
mkdir -p {directory}: 新建目录，对于不存在的子目录可以一并创建    

tree: 查看当前目录的文件树结构  

cp file1 file1-copy: 复制file1到file1-copy  
cp -r dir1 dir2: 复制文件夹dir1到dir2    

mv: 移动文件，也可用来重命名文件，具体用法和cp一样  

rmdir directory1: 移除一个空的文件夹  
rm file: 移除文件  
rm -r directory1: 移除文件夹    

chmod：修改文件的权限
u : user, g : Group, o : others    增加权限“＋”，删除权限“－”   r (Read), w (Write), x (Execute).  
chmod u+x file1, 给当前用户增加对file1的执行权限  

wc -l file1: 计算file1的行数  
wc -w file1: 计算file1单词的个数  
wc -c file1: 计算file1中字符的个数  

history: 查看历史命令纪录  
!-1 : 执行倒数第一条命令  
!345: 执行第345条命令  

cat file1 | grep A : 从file1中找到带有‘A’的行  
find . -name *.png 在当前文件夹下查找后缀名为.png的文件，子文件夹中的文件也可以被找出。  

更多命令，欢迎大家一起总结与补充
