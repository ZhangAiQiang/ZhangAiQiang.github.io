---
layout: post
title: flask-session身份伪造
categories: [web]
tags: [web]
fullview: false
comments: true
---

想研究很久了，这次终于初步了解了flask session伪造(得知道密钥)。  

参考文章:https://www.cnblogs.com/apossin/p/10083937.html  

## ①首先需要知道的信息   

flask是把session存在客户端的，而且只经过base64编码和用密钥签名，虽然没有有签名不可以伪造session，但是有很多信息我们可以直接从session解码找出来。  

## ②session伪造源码(https://github.com/ZhangAiQiang/Flask/tree/master/flask-session%E8%BA%AB%E4%BB%BD%E4%BC%AA%E9%80%A0%E7%AE%80%E5%8D%95%E6%BA%90%E7%A0%81)  ##


![](https://i.imgur.com/yGUkmP0.png)    

## ③简单的说一下代码的功能: ##  




- 除了admin需要密码，其他用户爱咋地咋地。
- 直接输入ip一开始没有session所以跳转登录界面，有了session就可以直接访问ip看到欢迎信息
- 想尽办法伪造admin用户  

## ④先随便用个qaq账户登陆下试试，登陆成功后##
    
**打开控制台:**  

![](https://i.imgur.com/jy5yv3T.png)

>     session的值为eyJ1c2VybmFtZSI6InFhcSJ9.Dxclgg.FMNAqa5Zk2wqg6S6WPyOQm-nU68  
>     其中eyJ1c2VybmFtZSI6InFhcSJ9为base64编码后session的内容。  

**解码：**  
 
![](https://i.imgur.com/S1h2W1v.png)  

## ⑤如果我直接把qaq改成admin然后base64编码一下不就可以直接登陆上去了？   

不是的，后边的内容是签名，一开始说了，没有密钥没法伪造身份就是因为这个签名防篡改的作用。  

## ⑥获取密钥的办法:   
ssti注入，信息泄露。  

这里ssti怎么获取就不说了，获取密钥要紧，以后再讨论ssti，输入**ip/{{config}}**  
![](https://i.imgur.com/wSZi5T1.png)  

可以看到密钥是**HELLO WORLD!**  

## ⑦伪造session脚本   ##
https://github.com/noraj/flask-session-cookie-manager    

![](https://i.imgur.com/vsjGbYc.png)  

## ⑧burp抓包，替换session   ##
  

![](https://i.imgur.com/CH1mE76.png)  

可以看到成功的以admin身份登陆  

![](https://i.imgur.com/CWKOd8E.png)