---
layout:     post   				    
title:     	解决picgo图床上传github失败问题 
subtitle:   骂骂咧咧的写下这篇文章
date:       2025-04-21				
author:     婷                               
header-img: img/155.png
catalog: true 						
tags:								

- dns
- github
- picgo
- 图床

---





## 简介

事情是这样的，本来昨天莫名其妙的`github.com`被解析了`localhost`，最后通过修改`host`文件救了回来，详情可以查看这个文章[链接](https://copyright1999.github.io/2025/04/20/%E8%A7%A3%E5%86%B3github%E8%BF%9E%E6%8E%A5%E4%B8%8D%E4%B8%8A%E7%9A%84%E9%97%AE%E9%A2%98/)。结果昨天的`picgo`图床上传到`github`也用不了，一开始以为是折腾了电脑的问题，今天电脑重启后，`cmd`去`ping`是对的地址，但是`picgo`发现还是不行。最后发现还是老问题，还是`dns`被污染的原因，但是为什么`host`文件已经设置了`github.com`的`IP`地址了，还是不行，这个就解释不了，下面简单记录下`picgo`用代理的方法





## 过程

首先问题的发现是当我试图直接上传图片的时候

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/raspi4b/06/202504212345115.png" alt="image-20250421233807550" style="zoom:67%;" />

从这个提示发现又是`localhost`，真的是绷不住了

![image-20250421215929249](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/raspi4b/06/202504212346113.png)

然而`CMD`去`ping`的时候，`github`是正常解析的，`host`文件也没被篡改

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/raspi4b/06/202504212346462.png" alt="image-20250421234005406" style="zoom:67%;" />

但是转念一想，如果我不去配置`host`文件，`github.com`还是会被解析成`localhost`，所以元凶还是`dns`被污染了，然后问了一样使用电信运营商的小老弟，果然他在广州的网络也是，`github.com`还是会被解析成`localhost`，所以元凶就是运营商墙了。但是为啥`host`配置了，`picgo`还是不行，就解释不了了。不过`picgo`可以设置代理，这一点非常的棒



<img src="D:\code_for_github\Copyright1999_Blog\_posts\tbd\00-riscv\2029-5-12-模板.assets\image-20250421225244738.png" alt="image-20250421225244738" style="zoom:67%;" />

上传代理填入我们代理的端口号

<img src="D:\code_for_github\Copyright1999_Blog\_posts\tbd\00-riscv\2029-5-12-模板.assets\image-20250421225254527-1745247175224-1.png" alt="image-20250421225254527" style="zoom:67%;" />



端口号怎么查询？一般点开自己的代理软件都能查的到

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/raspi4b/06/202504212346436.png" alt="image-20250421225304574" style="zoom:67%;" />



## 总结

不知道是不是最近关税战的问题还是啥，为啥把`github`也墙了呢？？？？

