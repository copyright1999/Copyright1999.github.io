---
layout:     post   				   
title:     rock3a使用usb-wifi网卡			
subtitle:  
date:       2024-01-28				
author:     婷                              
header-img: img/110.png 	
catalog: true 						
tags:								

- rock3a
- wifi
- 网络
---



## 简介

这篇文章主要记录`rock3a`上`usb wifi`网卡的使用方法，用的网卡是这款，以前实习的时候上家公司给的，刚好派上用场。

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240127194319668.png" alt="image-20240127194319668" style="zoom: 50%;" />





<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240127194341449.png" alt="image-20240127194341449" style="zoom:50%;" />



## 加载wifi驱动

担心网卡冗余的问题，所以下面测试的时候，都没用`eth0`网卡，`eth0`网卡直接给`down`掉了。（因为我的`eth0`网卡跟`wifi`在同个网段）

插入网卡后，系统自动加载了`rtl8192cu`相关的驱动

![image-20240114225724686](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240114225724686.png)



同时，`dmesg`中打印的内容如下，提示下载固件失败

![image-20240114230935574](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240114230935574.png)



针对固件问题，执行命令`sudo apt-get install firmware-realtek`

![image-20240114231142628](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240114231142628.png)



在`/lib/firmware/rtlwifi`路径下可以看到下载了许多`wifi`相关的固件，其中就有我们需要的`rtl8192`相关的固件

![image-20240114231344846](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240114231344846.png)



这时候重新插拔网卡，可以看到`dmesg`中固件下载失败的提示消失了

![image-20240114231301650](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240114231301650.png)



不过这个时候还是没有`link`

![image-20240114224050470](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240114224050470.png)



先检查`debian`的网络管理工具是否禁用了无线网络功能，这里看是没有的

![image-20240114232033008](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240114232033008.png)



![image-20240114232106692](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240114232106692.png)



如果被禁用可以用这个命令启动

```bash
nmcli radio wifi on
```





## 连接wifi

使用`wpa_cli`命令来连接`wifi`，首先进行扫描

```bash
scan
```



![image-20240127201143497](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240127201143497.png)



打印扫描到的`wifi`，比如`TP-LINK_905C`就是我们需要连接的`wifi`

```bash
scan_results
```



![image-20240127201206565](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240127201206565.png)



然后添加我们要连接的`wifi`，并使能，使能后可以看到`wifi`网卡打印`link becomes ready`

```bash
add_network
set_network 0 ssid "TP-LINK_905C"
set_network 0 psk "密码"
enable_network 0
```



![image-20240127201503502](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240127201503502.png)



最后退出

```bash
quit
```



![image-20240127201526202](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240127201526202.png)



`ifconfig`可以看到`wifi`网卡是`RUNNING`状态

![image-20240127201620249](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240127201620249.png)





## 其他问题

### iwconfig失败

一开始我用`iwconfig`链接`wifi`失败

![image-20240114230456875](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240114230456875.png)



简单的查了下，可能是下面的原因

>在实际使用中，如果要连接到受 WPA 或 WPA2 加密保护的无线网络，通常会使用 `wpa_cli` 进行连接配置和管理。而对于简单的无线网络连接，例如连接到开放的无线网络或者 WEP 加密的网络，可以使用 `iwconfig` 进行基本配置。





### 改网卡名字

`wifi`连接后，想给网卡改成`wlan0`比较好操作，但是改了后，就一直`up`不起来，用`wpa_cli`重新配置也不行，然后把名字改回来后就好了，不理解。

```bash
sudo ip link set wlx08beac133448 down
sudo ip link set wlx08beac133448 name wlan0
sudo ip link set wlan0 up
```



![image-20240127200802111](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240127200802111.png)







