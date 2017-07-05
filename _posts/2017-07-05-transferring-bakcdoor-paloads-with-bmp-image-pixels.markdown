---
layout:post 
title:  用BMP图像像素传输后门有效载荷[翻译]
data:   2017-07-05 16:12:16 +0800
categories: security
---

网上无意间看到老外写的这篇文章，写的挺好，自己跟着操作了一下，顺便翻译一下记录下来。只翻译了文章的主要结构，想看原文的请参见文章底部连接。

现在我们讨论的通过BMP图像像素传输后门有效载荷的方法，你可以测试它是否可以bypass反病毒软件或者IPS/IDS


正如你所看到的图片1，图片是黑色的背景加上红色的线，你有从图片中发现有什么错误的或者不正常的地方吗？
![图片1](../images/bmp1.png)  
图片1

图片2我向你展示了图片中不正常的地方
![图片2](../images/bmp2.png)  
图片2 恶意软件的有效载荷注入在图片像素的后面


看到这里我想讨论一下为什么这么做是有危害的和你怎样才能做到这一点？

重要的问题！
    1.为什么通过图片传输有效载荷或数据是危险的？
        很不幸的是,没有人认为这种方式是重要的威胁  
    2.在此之前你是否用反病毒软件扫描过bmp文件？  
    3.你是否用反病毒软件实时检测和扫描过bmp文件？  
    4.多个反病毒软件可以检测到这种威胁？  
    5.我们应该怎样检测这么威胁，当一个bmp文件放在一个目标网站或者被感染的网站？  
    6.这种方式可以用作web攻击吗？或者我们可以用这么方式bypass WAF(web应用防火墙)从bmp中读取有效载荷从而进行web攻击？  
    7.对于web或者网络渗透，传输有效载荷或数据的最好方式之一就是通过80或443端口,尤其是80端口，在bmp文件中有没有有效载荷加密(重要)  
    8.有多少防火墙或者IPS/IDS工具可以检测到这种技术  
    9.如果用这种技术将后门的有效载荷加密在图片中，谁可以检测出来，怎样检测处理？或者如果我使用这种技术通过分块BMP文件，这意味着将有效载荷分割成多于1个图片文件，那么哪种AV可以检测到这种有效载荷的传递？  



原文连接[Transferring Backdoor Payloads with BMP Image Pixels][src-url]

[src-url]:  https://www.peerlyst.com/posts/transferring-backdoor-payloads-with-bmp-image-pixels-damon-mohammadbagher?utm_source=twitter&utm_medium=social&utm_content=peerlyst_post&utm_campaign=peerlyst_resource


