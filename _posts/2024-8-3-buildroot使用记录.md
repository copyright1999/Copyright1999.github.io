---
layout:     post   				    
title:     buildroot使用记录			 
subtitle:  
date:       2024-08-03				
author:     婷                               
header-img: img/128.png 	
catalog: true 						
tags:								

- buildroot
- rootfs
- ext4


---





## 简介

简单列下如何用`buildroot`第一次配置自己的根文件系统，且制作用于`emmc`或者`sd`卡的根文件系统



## 下载

来到[官网](https://buildroot.org/download.html)，选择`buildroot-2024.02.4.tar.gz`

![image-20240803103211476](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240803103211476.png)

解压 

```bash
tar -zxvf buildroot-2024.02.4.tar.gz
```



![image-20240718215738085](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718215738085.png)



## 配置

先`make menuconfig`

![image-20240718220210517](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220210517.png)



### target options

![image-20240718220309671](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220309671-1721311389959-3.png)

可以看到默认的架构，我们这里修改下`Target Architeceture`

![image-20240718220325146](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220325146.png)

修改为` Arm (little endian)`

![image-20240718220404707](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220404707.png)

选完第一个之后，下面的其他选项都自动选成默认了

![image-20240718220425345](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220425345.png)

`Target Architecture Variant`选择为`cortex-A7`

![image-20240718220514626](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220514626.png)



![image-20240718220501282](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220501282.png)



接下来是`Target ABI`

![image-20240718220603929](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220603929.png)

根据我们的工具链选择为`EABIhf`

![image-20240718220548545](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220548545.png)



### toolchain

![image-20230615195741269](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20230615195741269.png)

**Toolchain type**：将`buildroot_toolchain`改成`external_toolchain`，这里就不用buildroot自己下载具体的工具链

![image-20240718220643799](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220643799.png)



![image-20240718220712067](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220712067.png)



![image-20240718220738290](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220738290.png)

**Toolchain **改为`Custom toolchain`

![image-20240718220757358](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220757358.png)



![image-20240718220811520](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220811520.png)

选完后自动变成了  **Toolchain origin (Pre-installed toolchain)**

![image-20240718220830826](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220830826.png)

然后填**Toolchain path**

![image-20240718220842797](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220842797.png)



如下是我们工具链的路径

![image-20240718220919127](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220919127.png)

![image-20240718220936185](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220936185.png)

![image-20240718220947950](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718220947950.png)





然后填写  **Toolchain prefix** 

![image-20240718221002963](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221002963.png)



![image-20240718221040815](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221040815.png)

![image-20240718221110074](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221110074.png)



接下来是**这三个选项**

![image-20240718221139454](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221139454.png)



**gcc version**选择`4.9.x`

![image-20240718221202457](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221202457.png)



![image-20240718221232017](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221232017.png)



**kernel headers series**是内核版本，根据自己用的内核版本来选择就可以

![image-20240718221315259](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221315259.png)

C库我们选择`glibc`，因为工具链是`glibc`类型，如果是`uclibc`或者是`musl`，`libc`后缀会带`uclibc`或者`musl`

![image-20240803105136823](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240803105136823.png)

![image-20240803105328212](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240803105328212.png)



![image-20240718221339305](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221339305.png)



![image-20240718221353247](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221353247.png)





**Toolchain has xx support**基础这三个选项都选上

![image-20240718221424590](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221424590.png)



 **Target Optimizations**这个是选填的，相当于编译优化的一个选项

![image-20240803110039097](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240803110039097.png)

这里我们我们填上我们的`CPU`型号

![image-20240718221508875](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221508875.png)

![image-20240718221521436](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221521436.png)



修改后就保存配置

![image-20240718221550225](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718221550225.png)





## 编译

直接输入如下命令，记住一定要`sudo`

```bash
sudo make -j16
```



一般如果编译过程没啥问题，不过我遇到了几个小问题，在后面小节记录了，这里就先看结果吧

最后就是在`output/images`下得到我们需要的产物`rootfs.tar`

![image-20240718231932639](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718231932639.png)



## 制作镜像

创建`release`文件夹，把`rootfs.tar`复制过来

![image-20240719002749014](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240719002749014.png)

制作空镜像，并格式化为`ext4`格式，这里我们制作的根文件系统为`256M`的大小，当然后面可以在`linux`下进行扩容

```
dd if=/dev/zero of=rootfs.ext4 bs=256M count=1
mkfs.ext4 rootfs.ext4
```



![image-20240719003710682](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240719003710682.png)



创建临时文件夹，供我们刚刚格式化的镜像可以进行挂载

```bash
mkdir tmp_rootfs
sudo mount -t ext4 rootfs.ext4 ./tmp_rootfs/
tar -xf rootfs.tar -C ./tmp_rootfs/
```



![image-20240719002821263](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240719002821263.png)



![image-20240719003249052](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240719003249052.png)



![image-20240719003447944](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240719003447944.png)



当然在`output/target`目录下也可以看到我们的`rootfs`里的内容，也可以直接把`target`下的内容直接复制也可以的，不过还是使用推荐做法啦

![image-20240803110622858](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240803110622858.png)



## 技巧 

### 打补丁



### 镜像源配置 







### 取消root登录





### 设置root密码



### rootfs扩容





### 更新







## 其他

这里记录下如何解决编译过程中的小问题

编译碰到`kernel`头文件版本不对的问题，这里我们按照出错提示选择`4.0.x`，同时删除`output/build/toolchain-external-custom`目录，再重新编译

![image-20240718231100318](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718231100318.png)



![image-20240718230650876](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718230650876.png)



![image-20240718231044503](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718231044503.png)



接着就是两个报错，根据提示将`Fortran`跟`OpenMP`都勾选上

![image-20240718231314669](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718231314669.png)



![image-20240718231355417](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718231355417.png)



![image-20240718231251338](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718231251338.png)



![image-20240718231416532](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/buildroot/image-20240718231416532.png)







