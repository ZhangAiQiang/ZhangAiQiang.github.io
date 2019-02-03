---
layout: post
title: volatility的使用
categories: [wp]
tags: [wp]
fullview: false
comments: true
---

记录下题解  
  
 
volatility取证的使用----windows内存  

##简介  

kali下默认安装  

可以对windows，linux，mac,android的内存进行分析
    
##内存文件的准备  

>     Win2003SP2x86下使用工具dumpit获取到了内存文件，保存为ROOT-6B78B0CA4D-20190202-044824.raw
>     其实不光.raw .vmem .img都是可以的  

##获取基本信息  

>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw imageinfo  
     
![](https://i.imgur.com/ewuQSNk.png)  

>     这里最关键的就是获取profile的类型，因为不同的系统数据结构啥的不一样，所以得用--profile=来指定。
>     这里自动猜解可能的系统类型，一般情况下第一个是正确的  

##列出所有进程  

>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw --profile=Win2003SP0x86 pslist
>     psxview可以查看隐藏进程  


```  

No suitable address space mapping found
Tried to open image as:
 MachOAddressSpace: mac: need base
 LimeAddressSpace: lime: need base
 WindowsHiberFileSpace32: No base Address Space
 WindowsCrashDumpSpace64BitMap: No base Address Space
 VMWareMetaAddressSpace: No base Address Space
 WindowsCrashDumpSpace64: No base Address Space
 HPAKAddressSpace: No base Address Space
 VirtualBoxCoreDumpElf64: No base Address Space
 VMWareAddressSpace: No base Address Space
 QemuCoreDumpElf: No base Address Space
 WindowsCrashDumpSpace32: No base Address Space
 SkipDuplicatesAMD64PagedMemory: No base Address Space
 WindowsAMD64PagedMemory: No base Address Space


```  

这是因为**profile指定错误**的原因，因为我自己提取的内存文件，确实是win2003sp2，而他第一个猜解的profile是sp0，没有关系，挨个试试就好了，重新列出进程  

>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw --profile=Win2003SP0x86 pslist    




![](https://i.imgur.com/JI2FJ6H.png)  

##提取出某个进程的内容  

我在win2003记事本和notepad++里面藏了个 "flag{123456}",我要找出来。  

>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw --profile=Win2003SP2x86 pslist|grep notepad  
>       
>     grep可以很快的在众多进程找出我们想要的进程 
>     我们需要找的就是进程的pid  
  
 
![](https://i.imgur.com/kO5chSa.png)  

>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw --profile=Win2003SP2x86 memdump -p 1448 -D /root    
>     
>     这条指令是把pid为1448进程的数据保存为dmp格式，保存到/root目录下边。  

最后一步用winhex打开dmp文件，搜索flag就能找到保存的内容了。  

![](https://i.imgur.com/JDjVFtv.png)  

##内存中注册表以及位置  

>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw --profile=Win2003SP2x86 hivelist  

![](https://i.imgur.com/Nd1CVLt.png)  

##得到用户密码的哈希值    

>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw --profile=Win2003SP2x86 -y （system的virtual地址） -s （sam的virtual地址）  

![](https://i.imgur.com/ksa7a0P.png)

##cmd执行的命令  
>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw --profile=Win2003SP2x86 cmdscan  

##网络连接情况  
>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw --profile=Win2003SP2x86 netscan  

 
##IE使用情况  
>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw --profile=Win2003SP2x86 iehistory   

##filescan查看文件    

>     volatility -f ROOT-6B78B0CA4D-20190202-044824.raw --profile=Win2003SP2x86 filescan
  
![](https://i.imgur.com/ED1Z9fw.png)
  
##提取filescan的文件,利用dumpfiles  

>     volatility -f memory  --profile=WinXPSP2x86 dumpfiles -Q 0x00000000053e9658 --dump-dir=./  
>     -Q制定了文件物理位置的开始，另一个参数制定了保存的位置。