---
layout: post
title: Diffie-Hellman密钥交换
categories: [wp]
tags: [wp]
fullview: false
comments: true
---

Diffie-Hellman密钥交换  
  
## 加密过程

有全局两个公开的参数:  
>     p,g
>     p是一个大素数，g是p的一个本原根  

现在有Alice和Bob两个人要交换密钥。  

>     ① Alice选定一个小于p的数a作为Alice私钥，并且计算Alice的公钥A=g^a mod p，公开Alice的公钥A
>     ② 与此同时Bob选定一个小于p的数b作为Bob私钥，并且计算Bob的公钥B=g^b mod p，公开Bob的公钥B
>     ③ Alice获得Bob的公钥B后，计算共享的私钥s=B^a mod p
>     ④ Bob获得Alice的公钥A后，计算共享的私钥s=A^b mod p
>     ⑤ 推导:s=A^b mod p=（g^a mod p）^b mod p=g^ab mod p
     
公式推导后，两人计算的s是相等的，并且双方各自的私钥没有被泄露，就协商出了共享的私钥。  


## 攻击方法
  

1.假设Eve为攻击者，Eve能够截取A发往B的数据包(包括A和B公钥)  
>     Eve可以获取p，g，A，B
>     Eve可以自己选定个私钥c，并且公布公钥C
>     Eve拦截alice和bob之间的信息，伪造数据并且互相转发，这样A实际和C通信，但是A认为是和B通信。B也这么认为，这样就控制了Alice和Bob之间的通信。  

2.解密通信过程中数据  
这里以安恒月赛的一个题目为例子:  

```  

Alice和Bob正在进行通信，作为中间人的Eve一直在窃听他们两人的通信。

Eve窃听到这样一段内容，主要内容如下：
p = 37
A = 17
B = 31

U2FsdGVkX1+mrbv3nUfzAjMY1kzM5P7ok/TzFCTFGs7ivutKLBLGbZxOfFebNdb2
l7V38e7I2ywU+BW/2dOTWIWnubAzhMN+jzlqbX6dD1rmGEd21sEAp40IQXmN/Y0O
K4nCu4xEuJsNsTJZhk50NaPTDk7J7J+wBsScdV0fIfe23pRg58qzdVljCOzosb62
7oPwxidBEPuxs4WYehm+15zjw2cw03qeOyaXnH/yeqytKUxKqe2L5fytlr6FybZw
HkYlPZ7JarNOIhO2OP3n53OZ1zFhwzTvjf7MVPsTAnZYc+OF2tqJS5mgWkWXnPal
+A2lWQgmVxCsjl1DLkQiWy+bFY3W/X59QZ1GEQFY1xqUFA4xCPkUgB+G6AC8DTpK
ix5+Grt91ie09Ye/SgBliKdt5BdPZplp0oJWdS8Iy0bqfF7voKX3VgTwRaCENgXl
VwhPEOslBJRh6Pk0cA0kUzyOQ+xFh82YTrNBX6xtucMhfoenc2XDCLp+qGVW9Kj6
m5lSYiFFd0E=

分析得知，他们是在公共信道上交换加密密钥，共同建立共享密钥。

而上面这段密文是Alice和Bob使用自己的密值和共享秘钥，组成一串字符的md5值的前16位字符作为密码使用另外一种加密算法加密明文得到的。

例如Alice的密值为3，Bob的密值为6，共享秘钥为35，那么密码为：

password = hashlib.md5("(3,6,35)").hexdigest()[0:16]



```  

解密过程:  
  
>     ① 第一步是获取共享密钥，题目中给出的参数p，A，B。
>     ② p=37，计算p的本原根=2  
>     ③ 计算a，17=2^a mod 37，a=7
>     ④ 计算b，31=2^b mod 37，b=9
>     ⑤ 计算s=g^ab mod p=6
>     ⑥ 计算password = hashlib.md5("(7,9,6)").hexdigest()[0:16]=a7ece9d133c9ec03

猜测password为密文的密钥，密文的加密方式未知，挨个尝试:  

https://www.alpertron.com.ar/DILOG.HTM  

![](https://i.imgur.com/AmSSHwF.png)