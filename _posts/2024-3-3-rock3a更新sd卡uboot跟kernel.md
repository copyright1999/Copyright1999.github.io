---
layout:     post   				    
title:     rock3a更新sd卡uboot跟kernel			
subtitle:  
date:       2024-03-03				
author:     婷                               
header-img: img/122.png 	
catalog: true 						
tags:								

- rock3a
- sd

---



## 简介

简单的记录下`rock3a`在linux下怎么更新`uboot`，`kernel`

先上一张[分区表](https://wiki.radxa.com/Rock3/partitions)

![image-20240303203557002](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240303203557002.png)



`dd`命令参数

- `skip=xxx`是在备份时对`if `后面的部分也就是原文件跳过多少`bs`的大小再开始备份
- `seek=xxx`则是在备份时对`of `后面的部分也就是目标文件跳过多少`bs`的大小再开始写
- `dd`命令默认的`bs=512K`





## 更新uboot

`sync`命令非常非常重要！！！

```bash
sudo dd if=idbloader.img of=/dev/mmcblk1 seek=64   conv=notrunc 
sudo dd if=u-boot.itb of=/dev/mmcblk1 seek=16384  conv=notrunc

sync
sync
sync
```



![image-20240303203827776](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240303203827776.png)





## 更新kernel

`sync`命令非常非常重要！！！



```bash
dd if=boot.img of=/dev/mmcblk1 seek=32768  conv=notrunc 

sync
sync
sync
```



![image-20240303200029500](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240303200029500.png)



![image-20240303200043507](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240303200043507.png)

如果觉得太慢了，可以设置`bs=1M`，`seek为16`

```bash
dd if=boot.img of=/dev/mmcblk1 seek=16  conv=notrunc bs=1M

sync
sync
sync
```



![image-20240303202310284](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240303202310284.png)



![image-20240303200043507](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rock3a/image-20240303200043507.png)



## 参考链接

- [分区表](https://wiki.radxa.com/Rock3/partitions)



## 其他

不知道为什么同样的`sd`卡，在`wsl`下将同样的文件`dd`写进去，`wsl`下总是会把一些字节给篡改了，难道是因为`dd`命令对于`/dev/sdx`跟`/dev/mmcblkx`有不一样的地方？？？





















