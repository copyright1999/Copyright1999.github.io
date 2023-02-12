---
layout:     post   				   
title:     pciutils交叉编译			
subtitle:  
date:       2023-02-12				
author:     婷                               
header-img: img/81.png 	
catalog: true 						
tags:								

- linux
- 交叉编译
- pcie
---



## 简介

本文主要是简单介绍`pciutils`工具如何交叉编译



## 过程

下载地址，选择`3.8`版本

```bash
https://mirrors.edge.kernel.org/pub/software/utils/pciutils/
```



![image-20220615225638578](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/pciutils/image-20220615225638578.png)



解压`tar -zxvf pciutils-3.8.0.tar.gz`

![image-20230212152914490](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/pciutils/image-20230212152914490.png)



修改`Makefile`

![image-20230212153041822](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/pciutils/image-20230212153041822.png)



红框为主要修改点

![image-20230212153416787](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/pciutils/image-20230212153416787-16761872571752.png)



```bash
# Host OS and release (override if you are cross-compiling)
HOST=arm-linux
RELEASE=
CROSS_COMPILE=aarch64-linux-gnu-

# Support for compressed pci.ids (yes/no, default: detect)
ZLIB=no

# Support for resolving ID's by DNS (yes/no, default: detect)
DNS=no

```



最后`make -j12`

![image-20230212153442572](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/pciutils/image-20230212153442572.png)



就看到成果物了

![image-20230212153924951](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/pciutils/image-20230212153924951.png)
