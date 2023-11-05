---
layout:     post   				   
title:     linux内核网络相关api			 
subtitle:  
date:       2023-11-5			
author:     婷                              
header-img: img/97.png	
catalog: true 						
tags:								

- gmac
- linux
- 网络
---



### 前言

最近在看`gmac`驱动代码，发现很多`linux`内核网络的`api`，暂时这里先整理出来，先大概知道什么用法，后续再深入了解



### netif_rx

```c
 void netif_rx(struct sk_buff *skb);
```

调用（包括中断期间）这个函数可以通知内核已经收到一个数据包，并封装入一个套接字缓冲区。



### netif_rx_schedule

```c
void netif_rx_schedule(dev);
```

调用该函数通知内核数据包已经存在，并且在接口上启动轮询机制，它只在`NAPI`驱动程序中使用。



### netif_receive_skb与netif_rx_complete

```c
int netif_receive_skb(struct sk_buff *skb)；
void netif_rx_complete(dev);
```

这两函数只在`NAPI`驱动程序中使用；`NAPI`中的`netif_receive_skb`函数与`netif_rx`等价，它将数据包发送给内核。当`NAPI`驱动程序耗尽了为接收数据包准备的内存时，则它将重新启动中断，然后调用`netif_rx_complete`终止轮询函数。 



### skb_shinfo

通过这个宏来判断数据包是由一个数据片段组成，还是由大量数据片段组成。例如海思的网卡代码

![image-20231105103228717](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gmac/image-20231105103228717.png)





### 参考链接

- [参考链接一](https://www.cnblogs.com/zxc2man/p/4105652.html)

- [参考链接二](https://www.cnblogs.com/debruyne/p/9133439.html)
