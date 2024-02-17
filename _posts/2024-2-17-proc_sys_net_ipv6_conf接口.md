---
layout:     post   				   
title:      proc_sys_net_ipv6_conf接口			
subtitle:  
date:       2024-02-17				
author:     婷                               
header-img: img/116.png 	
catalog: true 						
tags:								

- 网络
- ipv6
---





## 简介

主要介绍`/proc/sys/net/ipv6/conf` 目录相关参数的含义说明，以及一部分的内核代码小小的分析



## 说明

`/proc/sys/net/ipv6/conf/xxx` 目录

| 选项                      | 描述                                               | 备注                                                         |
| ------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **accept_ra**             | 控制是否接受路由器通告（Router Advertisement）消息 | 1，接受路由器通告；0，不接受路由器通告                       |
| **accept_redirects**      | 控制是否接受 IPv6 重定向消息                       | 1，接受重定向消息； 0，不接受重定向消息                      |
| **autoconf**              | 控制是否自动配置 IPv6 地址                         | 1，允许自动配置地址；0，禁止自动配置地址。                   |
| **disable_ipv6**          | 控制是否完全禁用 IPv6                              | 1，禁用 IPv6；0，启用 IPv6                                   |
| **forwarding**            | 控制是否启用 IPv6 数据包转发功能                   | 1，启用数据包转发； 0，禁用数据包转发                        |
| **hop_limit**             | 这个选项控制 IPv6 数据包的最大跳数                 | 通常情况下，跳数限制是默认值 64。                            |
| **mtu**                   | 这个选项控制 IPv6 的最大传输单元                   | 指定 IPv6 数据包的最大大小。                                 |
| **proxy_ndp**             | 控制是否启用 IPv6 邻居代理功能                     | 1，启用邻居代理； 0，禁用邻居代理。                          |
| **router_probe_interval** | 这个选项控制路由器探测的时间间隔                   | 当一个路由器失效时，系统会定期发送路由器探测消息以检测路由器是否重新可用 |
| **router_solicitations**  | 控制是否发送路由器请求消息。                       | 0 ，禁止发送路由器请求消息；设置为大于 0 的值则启用路由器请求消息，并设置请求的数量。 |





## conf/all conf/default

`net/ipv6/addrconf.c`中有两个全局变量`ipv6_devconf`和`ipv6_devconf_dflt`分别对应如下两个目录

```
/proc/sys/net/ipv6/conf/all/   
/proc/sys/net/ipv6/conf/default/
```



![image-20240217025000209](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240217025000209.png)



这两个结构体主要是一些配置选项的初始化

![image-20240215232122589](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240215232122589.png)



![image-20240215232142976](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240215232142976.png)



而初始化的函数流程如下，重点在于`addrconf_init_net`函数

```c
addrconf_init
	register_pernet_subsys(&addrconf_ops);
		addrconf_ops.init = addrconf_init_net

            
addrconf_init_net
	kmemdup --> "all"
    kmemdup --> "default"
    __addrconf_sysctl_register  "all"
    __addrconf_sysctl_register  "default"
```





函数`addrconf_init_net`中，`kmemdup`复制了两个配置，第一个配置`ipv6_devconf`，对应于`all`，第二个配置对应于默认`dflt`。（这里的`kmemdup`可以简单理解为`kmalloc+memcpy`）

![image-20240215231614678](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240215231614678.png)



复制后，调用`__addrconf_sysctl_register`函数

![image-20240217025735995](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240217025735995.png)

而`__addrconf_sysctl_register`函数先是对`addrconf_sysctl`全局变量进行复制给新变量`table`，然后`table`再更新为入参的`struct ipv6_devconf *p`的相关参数，这里的`table[i].data += (char *)p - (char *)&ipv6_devconf`没看懂是啥意思

![image-20240217163033795](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240217163033795.png)

接着就是把`table`中的相关参数一个一个注册到路径`net/ipv6/confg/%s`下，然后把table的值都赋予了入参`struct ipv6_devconf *p`的`sysctl_header`中，相当于以后要改自己接口（比如`eth0`)的配置项就可以通过这个`sysctl_header`找出来

![image-20240217171336063](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240217171336063.png)



而全局变量`addrconf_sysctl`则是对应着`/proc/sys/net/ipv6/conf/xxx` 目录下的各种内容

![image-20240217163156854](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240217163156854.png)





## conf/eth0

那我们这种`/proc/sys/net/ipv6/conf/eth0`的目录，有自己网卡的单独配置项的是什么时候注册的呢

![image-20240217163931882](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240217163931882.png)

在函数`ipv6_add_dev`中创建`inet6_dev`时，调用`addrconf_sysctl_register`函数注册，然后兜兜转转还是到了前面分析的`__addrconf_sysctl_register`函数

```c
ipv6_add_dev
    err = addrconf_sysctl_register(ndev)
    	__addrconf_sysctl_register(dev_net(idev->dev), idev->dev->name,idev, &idev->cnf)
```







## 配置接口演示

以`forwarding`为例，可以看到其`proc_handler`是`addrconf_sysctl_forwarding`

![image-20240215234554751](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240215234554751.png)

从函数的内容可以大概分析入参`write`会显示这次是读还是写，而下面的`proc_dointvec`函数则是内核中的一个`API`，大概就是提取原来的目录下的数值大小。如果是`write`就会修改前面提到的`table`

![image-20240217170823229](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/proc_sys_net_ipv6_conf/image-20240217170823229.png)

因为对接口读写这块其实涉及到内核的`sysctl`子系统，所以就不多描述，反正就是出现问题的时候知道要去哪里找代码看即可







## 参考链接

- [参考链接一](https://blog.csdn.net/sinat_20184565/article/details/113731345)
- [参考链接二](https://tldp.org/HOWTO/Linux+IPv6-HOWTO/ch11s02.html)















