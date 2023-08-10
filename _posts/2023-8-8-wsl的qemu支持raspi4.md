---
layout:     post   				    
title:     wsl的qemu支持raspi4			
subtitle:  
date:       2023-08-08				
author:     婷                               
header-img: img/87.png 	
catalog: true 						
tags:								

- wsl
- qemu
- raspi4
---



## 前言

目前从`ubuntu`安装的`qemu-system-aarch64`命令还不支持`raspi4`，输入一下命令可以看到支持的硬件类型

```bash
sudo ./qemu-system-aarch64 -machine help
```



![image-20230809234636173](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230809234636173.png)





通过网上搜索到的结果，有个`raspi4`的`qemu`补丁，于是打算尝试下。`Github`链接：

```
https://github.com/0xMirasio/qemu-patch-raspberry4
```





## 过程

### 提前安装

#### ninja

解决后面`configure`出现的这个问题

![image-20230731224937561](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731224937561.png)



```bash
sudo apt install ninja-build
```



![image-20230731225034916](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731225034916.png)



#### pkg-config

解决后面`configure`出现的这个问题

![image-20230731225104676](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731225104676.png)

```bash
sudo apt install pkg-config
```

![image-20230731225134203](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731225134203.png)



#### glib

解决后面`configure`出现的这个问题

![image-20230731225153118](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731225153118.png)



```bash
sudo apt-get install libglib2.0-dev
```



![image-20230731225317322](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731225317322.png)





#### pixman-1

解决后面`configure`出现的这个问题

![image-20230731230428349](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731230428349.png)



```
sudo apt-get install libpixman-1-dev
```



![image-20230731230544452](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731230544452.png)



### 编译过程

克隆该仓库

![image-20230731223609275](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731223609275.png)



然后创建`build`文件夹，在`build`文件夹下，执行`../configure`

![image-20230731224817819](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731224817819.png)



![image-20230731225359236](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731225359236.png)



`configure`结束

![image-20230731230633940](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230731230633940.png)



接下来执行`sudo make`

![image-20230808002800605](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230808002800605.png)



然后大概一两小时的编译的时间

![image-20230808210813741](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230808210813741.png)



生成物在`build`文件夹下

![image-20230808210945344](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230808210945344.png)



执行命令查看是否支持`raspi4`

```bash
sudo ./qemu-system-aarch64 -machine help
```



![image-20230808211838122](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230808211838122-16916765853211.png)



可以看到这里是支持上的，成功

![image-20230808211903792](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_raspi4/image-20230808211903792.png)


