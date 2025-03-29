---
layout:     post   				    
title:      wsl中使用qemu模拟riscv平台
subtitle:  
date:       2025-03-29				
author:     婷                               
header-img: img/147.png
catalog: true 						
tags:								

- qemu
- riscv
- wsl

---





## 简介

简单记录下`qemu`模拟`riscv`平台的过程，以前有模拟`arm64`平台的[文章](https://copyright1999.github.io/2023/07/30/wsl%E4%B8%8A%E4%BD%BF%E7%94%A8qemu/)



## qemu安装

首先安装`qemu`以及相关的软件

```bash
sudo apt-get install qemu-system-misc libncurses5-dev gcc-riscv64-linux-gnu build-essential bison flex libssl-dev
```

![image-20250326215241624](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326215241624.png)





## 编译代码

这里使用如下链接的代码

```
https://github.com/runninglinuxkernel/riscv_programming_practice/tree/master/chapter_2/benos
```

代码内容目录如下，`sbi`目录的代码编译出来是运行在**M模式**下的固件，为运行在**S模式**下的操作系统（这里是src目录的代码编译出来的镜像）提供引导和统一的

接口服务，业界流行的是用`OPENSBI`固件

![image-20250326215335763](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326215335763.png)



编译命令如下

```bash
export board=qemu
make clean
make
```

![image-20250326215410856](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326215410856.png)





## qemu运行

在上一步的目录，执行命令`make run`即可用`qemu`把编译出来的`riscv`的`baremental`程序跑起来了

![image-20250326215503159](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326215503159.png)







## qemu gdb调试

执行命令`make debug`即可让`qemu`进入`debug`模式

![image-20250326215703911](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326215703911.png)



然后再开一个终端，打开`gdb`终端

```bash
gdb-multiarch --tui /d/riscv_learn/benos/benos.elf
```



![image-20250326215841285](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326215841285.png)

![image-20250326215918638](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326215918638.png)



连接`gdb`的端口

```bash
target remote localhost:1234
```

![image-20250326215944085](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326215944085.png)

在程序开始的地方打断点

```bash
b _start
```

![image-20250326220009344](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326220009344.png)



运行

![image-20250326220048948](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326220048948.png)



如果想调试`sbi.elf`得退出前面占用的`1234`端口

```
gdb-multiarch --tui /d/riscv_learn/benos/mysbi.elf
```

![image-20250326220240903](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/riscv00/image-20250326220240903.png)







