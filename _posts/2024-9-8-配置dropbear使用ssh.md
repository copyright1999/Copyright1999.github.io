---
layout:     post   				    
title:     配置dropbear使用ssh			
subtitle:  
date:       2024-09-08				
author:     婷                               
header-img: img/132.png 	
catalog: true 						
tags:								

- buildroot
- dropbear
- ssh
- raspi

---





## 简介

树莓派上使用`buildroot`编译的`dropbear`，开启`ssh`功能



## 网络配置

### 编译dropbear

使用`buildroot`构建根文件系统，开启`dropbear`选项，更新`rootfs`

更新后查看是否有`/etc/dropbear`文件夹，如果没有则需要自己创建



### dropbear启动

启动命令如下

```
dropbear -R -I 1800
```

我们可以把这个命令加到`/etc/init.d/S50eth0`文件中，这样就可以开机自启了

![image-20240907225856323](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/dropbear/image-20240907225856323.png)

`netstat`查看，已生效

![image-20240907230123235](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/dropbear/image-20240907230123235.png)







### 空密码，新增用户

因为树莓派是`root`用户，密码是空，`dropbear`似乎不支持空密码登录

于是我们添加`admin`用户，密码为`abcd1234`，在`/etc/passwd`文件最后一行加入

```
admin:$5$4dd1e101bd12bbd9$T7jsbPkk0SVWjqpcicC1t3dXGVn9wQ7EllL.GBRN3m/:0:0:root:/:/bin/sh
```



![image-20240907204741639](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/dropbear/image-20240907204741639.png)



然后即可用用户名`admin`密码`abcd1234`登录`ssh`登录树莓派了





## 遇到的问题

一开始是懒得更新整个`rootfs`，然后就把`dropbear`可执行文件复制过来了，然后总是显示下面的错误

![image-20240907173942640](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/dropbear/image-20240907173942640.png)

最后想到可以查看`syslog`看问题，于是查看`/var/log/messages`

![image-20240907181810398](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/dropbear/image-20240907181810398.png)

可以看到是缺少`/etc/dropbear`这个文件夹

![image-20240907181712542](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/dropbear/image-20240907181712542.png)





## 代办

有时间的话整理下以前分析`/etc/passwd`中密码算法的文档

