---
layout: post
title: Docker与CTF
categories: [docker]
tags: [docker]
fullview: false
comments: true
---

Docker与CTF

主要是用来搭建环境，漏洞环境,CTF比赛题目复现。  

----------

docker你可以把它理解为一个vmware.

>     iamges:vmware需要的iso镜像  
>     container:vmware运行中的虚拟机。  

 

----------

我该去哪里用docker？  

>     1. 有个linux，64位，内核版本大于3.10，有阿里云linux主机的话可以直接用，也可以用vmware安装个最新版本的ubuntu，centos啥的都可以。
>     2. windows不推荐，用了这个会和vmware冲突。  
>     3. 安装的话http://www.runoob.com/docker/ubuntu-docker-install.html，很详细了。


----------
  

docker 简单的指令  



>     1.  docker images:查看已经下载的镜像
>     2.  docker ps:查看正在运行的contains
>     3.  docker stop

----------

怎么用别人的docker镜像  

>     1. 一种是没有docker-compose.yml，而是有dockerfile这个文件，我们先用指令建立一个images，进入dockerfile的目录下。
>     2. docker build -t  name:name1 .
>     3. 有镜像后，让它运行(变成container)
>     4. docker run -i -d -p 20000:80 name:name1
>     5. 注意这个20000:80，20000代表着外网访问你的端口号，80是你ctf题目镜像要用到的端口号(如果不知道的话，用记事本打开看看),就是一个端口映射的关系，name:name1是镜像的名字，和第二步要对应。
>     6. 用阿里云的话，安全组配置要开放这个端口号。访问 ip:端口  


>     1.另一种是有docker-compose.yml，这个更加简单，但是需要安装docker-compose，直接百度就可以了
>     2. 当前目录下  
>     3. docker-compose build  
>     4. docker-compose up -d  
>     5. docker ps查看运行中的container，可以看到开放的端口，直接ip：端口就行  


有的时候端口号会冲突，docker stop 停止其中一个container，开另一个就好了
----------
到这里环境就搭建起来了，两个漏洞
