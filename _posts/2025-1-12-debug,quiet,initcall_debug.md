---
layout:     post   				    
title:      debug,quiet,initcall_debug			
subtitle:  	bootargs参数
date:       2025-01-12				
author:     婷                               
header-img: img/141.png 	
catalog: true 						
tags:								

- uboot
- initcall_debug
- bootargs

---





## 简介

简单演示下`uboot`设置内核`loglevel`的用法，以及结合`initcall_debug`参数的用法





## 用法

- `uboot`的`bootargs`中设置`debug`，则内核启动过程中会把不小于`debug`等级的所有打印都显示出来，如果设置为`quiet`，除非是`err`信息，否则则不打印内核启动信息。但是所有的信息都会存在`logbuf`中，`dmesg`可以查看
- 而`uboot`的`bootargs`中设置`initcall_debug`参数，则是让内核显示每个模块的加载时间





## debug

`uboot`中设置`debug`参数，启动过程中如果有`debug level`的打印会显示出来

![image-20250112122247085](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112122247085.png)

![image-20250112122326339](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112122326339.png)



## quiet

设置`quiet`参数，启动过程中只显示`err`信息

![image-20250112122143415](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112122143415.png)



## initcall_debug

设置`initcall_debug`参数

![image-20250112121631016](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112121631016.png)

启动过程中，并没有显示相关模块的加载时间

![image-20250112121834931](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112121834931.png)

而`dmesg`后，才能查看，说明此类信息为`debug level`的信息

![image-20250112121854148](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112121854148.png)





## initcall_debug debug

如果想要在系统启动过程查看时间或者查看是否因为某个模块加载卡主了，这时候就可以`initcall_debug`参数结合`debug`参数一起设置

![image-20250112121059064](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112121059064.png)

启动过程中打印如下

![image-20250112121140661](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112121140661.png)

`reboot`也会显示相关模块的`remove`

![image-20250112121239001](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112121239001.png)





## initcall_debug quiet

当然觉得全打印出来消息太多了，也可以`initcall_debug`跟`quiet`结合

![image-20250112121349336](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112121349336.png)

启动过程中就干净了

![image-20250112121444355](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/initcall_debug/image-20250112121444355.png)





















