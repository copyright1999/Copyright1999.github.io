---
layout:     post   				   
title:     wireshark实用技巧			
subtitle:  
date:       2024-02-14				
author:     婷                               
header-img: img/115.png 	
catalog: true 						
tags:								

- 网络
- wireshark
---



## 简介

不定时更新，介绍一些`wireshark`有用的配置



## 显示MAC地址

在列**Time**这里右击，选择**列首选项**

![image-20240214170500092](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wireshark/image-20240214170500092.png)



![image-20240214150628935](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wireshark/image-20240214150628935.png)



这里我增加两个自定义的`Title`，分别是`Src Mac`跟`Dest Mac`。类型后面就选择硬件地址就好。

![image-20240214170734965](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wireshark/image-20240214170734965.png)



然后就可以看到多出的这两列啦

![image-20240214150736680](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wireshark/image-20240214150736680.png)







## 单独导出特定分组

比如我们经常筛选出我们需要的包，把这些需要的再单独提取出来成一个`pcap`文件

![image-20240221234126433](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wireshark/image-20240221234126433.png)

点击**文件**，选择**导出特定分组**

![image-20240221234203947](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wireshark/image-20240221234203947.png)





界面如下，这里选择的是导出`Displayed`的所有包，也就是通过我们的过滤规则的所有包

![image-20240221233913772](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wireshark/image-20240221233913772.png)



而这个是指选择我们点击查看的那个包，比如导出特定分组前我点击了`No.6`的包进行查看，就导出这个

![image-20240221233927607](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wireshark/image-20240221233927607.png)

这里则是选择范围，这个范围就是所有包的序列号范围，如下我们需要的包序列号范围为`1-81`，而实际过滤后只剩下`75`个包

![image-20240221234042018](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wireshark/image-20240221234042018.png)

如果选择`Captured`就把`1-81`的所有包导出来

![image-20240221234020595](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wireshark/image-20240221234020595.png)













