---
layout:     post   				    
title:      MLC,SLC,TLC概念		
subtitle:  
date:       2025-04-05				
author:     婷                              
header-img: img/149.png 	
catalog: true 						
tags:								

- emmc
- 存储

---





## 概念

在一些厂商`support`的`emmc`列表上，会有这些数据

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc00/1743843708758-1.png)

`SLC`，`MLC`，`TLC`这三个都是表示闪存的类型，最大的区别就是每个单元能存储的比特数

| 闪存类型            | SLC（single-level-cell) | MLC（multiple)     | TLC（triple)    也有Flash厂家叫8LC |
| ------------------- | ----------------------- | ------------------ | ---------------------------------- |
| 每单元比特数        | 1                       | 2                  | 3                                  |
| 可擦写次数          | 约10万次                | 约5千              | 约1千                              |
| 读取时间            | 25us                    | 50us               | 75us                               |
| 编程时间            | 300us                   | 600us              | 900us                              |
| 擦写时间            | 1500us                  | 3000us             | 4500us                             |
| 价格                | 最贵                    | 中等               | 最便宜                             |
| MOSSFET电压变化状态 | 2种（0,1）              | 4种（00,01,10,11） | 8种（000,001,010,...,111）         |



## 物理结构

`MOSFET`(金属氧化物半导体场效应晶体管)基本结构如下，在对一个闪存单元编程的时候，电压加到控制栅极(control gate)上，形成一个电场，让电子穿过硅氧化物栅栏，达到浮动栅极(floating gate)。穿越过程完成后，控制栅极上的电压会立刻降回零，硅氧化物就扮演了一个绝缘层的角色。单元的擦除过程类似，只不过电压加在硅基底(P-well)上。

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc00/1743843739921-4.png)



`SLC`、`MLC`、`TLC`三种闪存的`MOSFET`三种闪存的`MOSFET`是完全一样的，区别在于如何对单元进行编程。

- `SLC`要么编程，要么不编程，状态只能是0、1。
- `MLC`每个单元存储俩比特，状态就有四种00、01、10、11，电压状态对应也有四种。
- `TLC`每个单元三个比特，状态就有八种了(000、001、010、100、011、101、110、111)。



物理上差不多的结构储存了更多的信息，所以随之而来存储更多的信息就等于带来了更多不稳定。





## 怎么查看

目前好像只能是看手册了，但是我看了两份手册也没具体的内容





## 参考链接

- [参考链接一](https://mp.weixin.qq.com/s/9JbcvDXWEx6TuB9MnvMT-w)
- [参考链接二](https://mp.weixin.qq.com/s/9hQ1Y-c3h1oVmOvkIItQmA)



