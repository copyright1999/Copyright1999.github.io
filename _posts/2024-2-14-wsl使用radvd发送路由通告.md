---
layout:     post   				    
title:     wsl使用radvd发送路由通告			 
subtitle:  
date:       2024-02-14				
author:     婷                               
header-img: img/114.png 	
catalog: true 					
tags:								

- 网络
- wsl
- ipv6
- radvd

---





## 简介

最近在学习`ipv6`相关内容，在`wsl`上使用`radvd`来实现发送路由通告（**RA报文**）分配地址前缀跟`dns`服务器的功能。





## radvd分配地址前缀

安装

```bash
sudo apt-get install radvd
```

![image-20240210153309022](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240210153309022.png)



编辑 `/etc/radvd.conf` 文件来定义路由器广告的参数

![image-20240210153847821](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240210153847821.png)

我使用的是我的`tap0`网卡，发送的路由前缀是`2001:db8:1234:5678::/64`

```bash
interface tap0 {
    AdvSendAdvert on;
    MinRtrAdvInterval 3;
    MaxRtrAdvInterval 10;
    prefix 2001:db8:1234:5678::/64 {
        AdvOnLink on;
        AdvAutonomous on;
        AdvRouterAddr on;
    };
};

```



![image-20240210153831591](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240210153831591.png)



`wsl`打开`ipv6`转发功能（显得自己是个路由器？）

```bash
sudo sysctl -w net.ipv6.conf.all.forwarding=1
```



![image-20240212163521363](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240212163521363.png)



启动 `radvd` 服务

```bash
sudo systemctl start radvd
sudo systemctl enable radvd
```



![image-20240212154035554](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240212154035554.png)



运行`radvd`

```bash
sudo /etc/init.d/radvd  start
```

![image-20240212163452493](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240212163452493.png)



在`wsl`这边可以用`radvdump`命令查看发送的路由通告消息的具体内容

```bash
sudo apt install radvdump
sudo radvdump
```



安装

![image-20240212155238943](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240212155238943.png)



运行

![image-20240212163618548](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240212163618548.png)



![image-20240212163600421](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240212163600421.png)



这时候可以看到`qemu`中的`enp0s1`网卡收到了此`RA`消息，并生成了此前缀的`ipv6`地址



![image-20240214153406075](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214153406075.png)



如果想停止运行`radvd`，输入下列命令

```bash
sudo /etc/init.d/radvd stop
```

![image-20240214153633943](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214153633943.png)



`wsl`这边的抓到的`RA`消息如下，这里就不多解释，等下篇文章再做解释，抓包文件已放此[链接](https://github.com/copyright1999/image-typora-markdown/tree/main/ipv6)

![image-20240214153813114](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214153813114.png)



![image-20240214135033609](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214135033609.png)



## radvd发送dns信息

编辑 `/etc/radvd.conf` 文件，增加红框所示的内容

> RDNSS 是一种用于在 SLAAC 中提供域名解析服务的方式。它通过在 Router Advertisement 报文中包含 DNS 服务器地址，为主机提供域名解析服务，而不需要额外的 DHCPv6 服务器。在RFC6106中在5.1章节标题为Recursive DNS Server Option。

![image-20240214161750863](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214161750863.png)



```bash
interface tap0 {
    AdvSendAdvert on;
    MinRtrAdvInterval 3;
    MaxRtrAdvInterval 10;
    prefix 2001:db8:1234:5678::/64 {
        AdvOnLink on;
        AdvAutonomous on;
        AdvRouterAddr on;
    };

    RDNSS fd00:2020:2019::100 fd00:2020:2019::200 {
        AdvRDNSSLifetime 300;
    };

};
```



运行

![image-20240214161913166](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214161913166.png)



查看`qemu`这边的域名服务器，可以看到增加了信的`ipv6`域名服务器

```bash
cat /etc/resolv.conf
```



![image-20240214161939962](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214161939962.png)



在`wsl`这边进行抓包，抓包文件已放此[链接](https://github.com/copyright1999/image-typora-markdown/tree/main/ipv6)

![image-20240214162940882](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214162940882.png)

![image-20240214162536591](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214162536591.png)



![image-20240214162841501](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214162841501.png)









## 源码

源码下载链接

```bash
https://radvd.litech.org/
```

`RA`报文发送的实现在`send.c`中的`really_send`函数

![image-20240214160645529](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214160645529.png)



![image-20240214160922350](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/ipv6/image-20240214160922350.png)





## 参考链接

- [抓包文件链接](https://github.com/copyright1999/image-typora-markdown/tree/main/ipv6)
- [radvd配置介绍](https://blog.csdn.net/ttood/article/details/119213677)
- [radvd配置dns信息](https://blog.csdn.net/zdl244/article/details/109140794)
- [RFC6106](https://datatracker.ietf.org/doc/html/rfc6106)















