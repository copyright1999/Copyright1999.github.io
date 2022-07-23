---
layout:     post   				    
title:     linux USB学习（三）			 
subtitle:  一些基本的概念
date:       2022-07-23				
author:     婷                               
header-img: img/74.jpg 
catalog: true 						
tags:								

- USB

---



## 简介

这里主要介绍`USB`一些会涉及到的基本概念，后续会慢慢的补充跟完善。



## USB主控制器种类



| 种类 | 对应的USB的协议和支持的速率   | 创立者                                    | 功能划分                                                     | 常用于                        |
| :--: | ----------------------------- | ----------------------------------------- | ------------------------------------------------------------ | ----------------------------- |
| OHCI | USB 1.1=Low Speed和Full Speed | Compaq，Microsoft和National Semiconductor | 硬件功能 > 软件功能⇒硬件做的事情更多，所以实现对应的软件驱动的任务，就相对较简单 | 扩展卡，嵌入式开发板的USB主控 |
| UHCI | USB 1.1=Low Speed和Full Speed | Intel                                     | 软件功能 > 硬件功能⇒软件的任务重，可以使用较便宜的硬件的USB控制器 | PC端的主板上的USB主控         |
| ECHI | USB 2.0=High Speed            | Intel                                     | 定义了USB 2.0主控中所要实现何种功能，以及如何实现            | 各种USB 2.0主控               |
| xHCI | USB 3.0=Super Speed           | Intel                                     | 定义了USB 3.0主控中所要实现何种功能，以及如何实现            | 各种USB 3.0主控               |



关于速度，内核中`include/uapi/linux/usb/ch9.h`有相关的枚举变量来表示

![image-20220719004229197](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719004229197.png)





## USB的线缆

- `USB2.0`只有4根线，分别是`VBUS`，`D+`,`D-`和`GND`。
- `USB3.0`的线缆是9根线，除过包括`USB2.0`的四根线外，还包括` RX−` ，`RX+` ，`TX− `，`TX+ `和`GND_DRAIN`五根线。



### USB1.x/2.0接口的引脚定义及颜色

| 引脚 | 名称 | 电缆颜色 | 描述           |
| ---- | ---- | -------- | -------------- |
| 1    | VBUS | Red      | +5 V，电源     |
| 2    | D−   | White    | Data −，数据线 |
| 3    | D+   | Green    | Data +，数据线 |
| 4    | GND  | Black    | Ground，接地   |

**注意：**引脚号跟下面要介绍的**USB物理接口**是相对应的。





### USB3.0接口及以上的引脚定义及颜色

| 引脚 | A型连接器  | B型连接器  | 线缆颜色 | 描述                |
| ---- | ---------- | ---------- | -------- | ------------------- |
| 1    | VBUS       | 红色       | 红色     | 供电                |
| 2    | D-         | D-         | 白色     | 2.0数据差分对       |
| 3    | D+         | D+         | 绿色     | 2.0数据差分对       |
| 4    | GND        | GND        | 黑色     | 电源地              |
| 5    | StdA_SSRX− | StdB_SSTX− | 蓝色     | 高速数据差分对-接收 |
| 6    | StdA_SSRX+ | StdB_SSTX+ | 黄色     | 高速数据差分对-接收 |
| 7    | GND_DRAIN  | N/A        | N/A      | 信号地              |
| 8    | StdA_SSTX− | StdB_SSRX− | 紫色     | 高速数据差分对-发送 |
| 9    | StdA_SSTX+ | StdB_SSRX+ | 橙色     | 高速数据差分对-发送 |

**注意：**可以看到这里的`1 2 3 4`引脚其实是兼容了之前的`USB2.0`



### USB2.0 OTG 

`OTG` 是` On-The-Go` 的缩写，支持 `USB OTG `功能的` USB` 接口既可以做 `HOST`， 也可以做 `DEVICE`。而接口怎么知道自己是主是从，则是通过`ID`线判断。像`Mini USB`或者是`Micro USB`有带`ID`线的接口就可以做`otg`。

`Mini/Micro-A/B`引脚定义以及颜色

| 引脚 | 名称 | 线缆颜色 | 描述                             |
| ---- | ---- | -------- | -------------------------------- |
| 1    | VBUS | 红色     | +5V供电                          |
| 2    | D-   | 白色     | 差分数据-                        |
| 3    | D+   | 绿色     | 差分数据+                        |
| 4    | ID   | N/A      | ID高电平：从机<br>ID低电平：主机 |
| 5    | GND  | 黑色     | 地                               |





## USB的物理接口

`USB`的接口类型，根据接口形状不同，主要可以分为三大类：

- `TYPE`类型：普通的硬件直接叫做`Type`
- `Mini`类型：小型版本的叫`Mini`迷你的
- `Micro`类型：更加小的，叫做`Micro`微小的





### USB 2.0 中的 Type-A

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723152500746.png" alt="image-20220723152500746" style="zoom: 50%;" />

上面这种就是最常见的 `USB Type-A` 接口, 我们一般直接叫做`USB`接口.

![image-20220719000231585](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719000231585.png)



### USB 2.0 中的 Type-B

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723152542328.png" alt="image-20220723152542328" style="zoom: 67%;" />

上面这种是`USB 2.0 Type-B`, 常见于打印机连接计算机所采用的数据线, 通常一端是 `Type-B `连接打印机, 另一端 `Type-A `连接计算机.

![image-20220719000407316](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719000407316.png)



### USB 2.0 中的 Mini USB

`Mini USB` 是我们通常叫法, 其实严谨的叫法应该是 `USB 2.0 Mini-B`, 因为还有一个 `USB 2.0 Mini-A`, 不过因为还未推广便已被淘汰, 所以市面上所有的` Mini USB `都是`USB 2.0 Mini-B `。

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723152618926.png" alt="image-20220723152618926" style="zoom:67%;" />



这种连接线常见于早期的移动硬盘, `MP3`, 智能手机等。



![image-20220719000449732](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719000449732.png)

 （这里五个引脚前面介绍`OTG`的时候已经讲过了）



### USB 2.0 中的 Micro USB

同样` Micro USB `是我们通常叫法, 严谨的叫法是 `USB 2.0 Micro-B`, 也是因为还有一个 `USB 2.0 Micro-A`, 不过几乎看不到.



<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723152640503.png" alt="image-20220723152640503" style="zoom:50%;" />

这种接口相比 `Mini USB` 要更加小巧, 在 `USB Type-C` 普及之前, 几乎市面上所有的**安卓机**采用此接口。

 

![image-20220719000514850](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719000514850.png)



### USB 3.0 中的 Type-A

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723152707316.png" alt="image-20220723152707316" style="zoom:50%;" />



上面这种也是` USB TYPE-A `接口, 不过因为` USB 3.0 `标准的推出, 为了方便区分 `USB 2.0`, 所以接口的颜色通常使用**蓝色**. 所以如果你的主板背板有黑色和蓝色两种` USB` 接口, 通常表示黑色的是 `USB2.0`, 而蓝色的是` USB3.0`或更高标准。



![image-20220719000541908](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719000541908.png)

 



### USB 3.0 中的 Type-B

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/usb-3.0-type-b.jpg" alt="USB接口详细读解, Gen2和Gen1到底是个什么鬼?" style="zoom: 50%;" />

`USB 3.0 Type-B `常见于连接 `USB 3.0 `的` HUB`, 和` USB 3.0 `的移动硬盘盒。



![image-20220719000653876](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719000653876.png)

 



### USB 3.0 中的 Micro-B

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/usb-3.0-micro-b.jpg" alt="USB接口详细读解, Gen2和Gen1到底是个什么鬼?" style="zoom:33%;" />

`USB 3.0 Micro-B `常见于 `USB 3.0 `的移动硬盘, 例如**希捷的3.5外置移动硬盘**。

 

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719000922684.png" alt="image-20220719000922684" style="zoom:200%;" />





### USB Type-C

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/usbtypec.jpg" alt="USB接口详细读解, Gen2和Gen1到底是个什么鬼?" style="zoom:33%;" />

上面这种就是` USB Type-C`了, 方便好用, 正插反插都可以。**这里还要提醒一点**, `USB Type-C` 只是一个物理接口, 并不等于它的传输速度, 例如最早的` NOKIA N1 `平板电脑所使用的` USB Type-C `传输速率仅相当于` USB 2.0`。但是目前市面上采用` USB Type-C` 接口的设备一般至少都是` USB3.1 Gen1 `传输标准。

 

![typec.png](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/typec-pin-define-connect.png)



### 其他被抛弃的接口

看看就好

|   类型   |                             图片                             |
| :------: | :----------------------------------------------------------: |
|  Mini-A  | ![](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719001020810.png) |
| Mini-AB  | ![image-20220719001039570](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719001039570.png) |
| Micro-A  | ![image-20220719001049422](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719001049422.png) |
| Micro-AB | ![image-20220719001111275](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719001111275.png) |







## USB2.0 与 USB3.0的辨认方法

### 颜色

`USB3.0`通常是蓝色基座

### 图标

`USB2.0`的标志就是和`USB1.1`的标志基本上没啥区别，还是以前的那个样子，使用黑色颜色用标识

![USB2.0图标](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/1617346863065.png)

`USB3.0`它有一个`SS`标志，意思是`SuperSpeed`，部分的设备上面会有这个标志,且使用蓝色进行标识：

![USB3.0图标](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/1617346879827.png)

比如上面提到的



<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220719002911948.png" alt="image-20220719002911948" style="zoom: 80%;" />



### 触点

`USB2.0`仅具备4个金属触点 （除了支持`otg`功能的接口有一条`ID`线），`USB3.0`则是9个触点

![usb3.0和2.0的区别](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/1617347252802.png)



![USB3.0](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/1617346405958.png)



## USB速度

吐槽一句：关于3.0的叫法是真的乱

| 版本                        | 带宽    | 编码      | 理论速度 | 速率称号                | USB标识                                                      |
| --------------------------- | ------- | --------- | -------- | ----------------------- | ------------------------------------------------------------ |
| USB1.0                      | 1.5Mbps |           |          | 低速（Low-Speed)        | ![image-20220723141445033](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723141445033.png) |
| USB1.1                      | 12Mbps  | 8b/8b     | 1.5MB/s  | 全速（Full-Speed)       | ![image-20220723141448773](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723141448773.png) |
| USB2.0                      | 480Mbps | 8b/8b     | 60MB/s   | 高速（High-Speed)       | ![image-20220723141502874](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723141502874.png) |
| USB3.0<br>USB3.1 Gen1       | 5Gbps   | 8b/10b    | 500MB/s  | 超高速（Supper-Speed)   | ![image-20220723141741261](D:\code_for_github\Copyright1999_Blog\_posts\2022-7-17-USB学习记录三 主要是些基本概念.assets\image-20220723141741261.png) |
| USB3.1 Gen2<br/>USB3.2 Gen1 | 10Gbps  | 128b/132b | 1280MB/s | 超高速+（Supper-Speed+) | ![image-20220723141513030](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723141513030.png) |
| USB3.2 Gen2                 | 20Gbps  |           |          | 超高速+（Supper-Speed+) | ![image-20220723141728794](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723141728794.png) |





## USB4.0规范

`USB4`规范已于2019年9月3日正式公布, 规格和草案阶段相同。 归纳如下:

- 物理接口只有 `USB Type-C `一种
- 传输速率 `40 Gbps`
- 向下兼容 `USB 3.2 / USB 2.0 `和 `Thunderbolt 3`







## USB2.0硬件热插拔原理

`USB`主机是如何检测到设备的插入呢？

首先在`USB`集线器的每个下游端口的`D+ D-`上，分别接了一个`15K`欧姆的下拉电阻到地，这样在集线器的端口空悬的时候，就被这两个下拉电阻拉到了低电平。而在`USB`设备端，在`D+`或者`D-`上接了`1.5K`欧姆上拉电阻，对于全速和高速设备，上拉电阻是接在`D+`上,而低速设备则是上拉电阻接在`D-`上。

这样当设备插入到集线器时，由`1.5K`的上拉电阻和`15K`的下拉电阻分压，结果就将差分数据线中的一条拉高了，集线器检测到这个状态后，他就报告给`USB`主控制器（或者是通过他的上一层的集线器报告给`USB`主控制器），这样就检测到设备的插入了。`USB`高速设备先是被识别为全速设备，然后通过`host`和`device`两者之间的确认，再切换到高速模式的。

![image-20220723152130562](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb3/image-20220723152130562.png)





## RootHub

`RootHub`是`USB`主控制器内嵌的一个硬件，主要来做热插拔检测的工作。



## usb3.0和2.0的区别-协议速率电源等其性能对比

- `USB2.0`具备`480Mbit/s`的高速传输速率，向下兼容低速`1.5Mbit/s`和全速`12Mit/s`,对外提供供电电压为`5V`，最大电流`500mA`
- `USB3.0`提供更高的`5.0Gbit/s`的超高速传输速度，并向下兼容低速`1.5Mbits/s`、全速`12Mbit/s`和高速`480Mbit/s`传输速率,对外提供供电电压为`5V`，最大电流`900mA`
- `USB3.0`也增加了新的电源管理功能，支持待机、睡眠以及暂定模式，更加省电。
- `USB3.0`是全双工通讯，而`USB2.0`是半双工通讯。
- `USB2.0`采用的是`NRZI`编码，而`USB3.0`采用的是`8B/10B`编码。



## 参考链接

- [链接一](http://www.usbzh.com/article/detail-206.html)

- [链接二](http://www.usbzh.com/article/detail-258.html)

- [链接三](https://www.bybusa.com/community/usb-interface-detailed-explanation)

- [链接四](http://www.usbzh.com/article/detail-144.html)

- [链接五](http://www.usbzh.com/article/detail-138.html)

- [链接六](http://www.cn-bbj.com/news_view.aspx?TypeId=4&Id=490&Fid=t2:4:2)

- [总线速率计算链接一](https://blog.csdn.net/qq_31020665/article/details/105103677)

- [总线速率计算链接二](https://blog.csdn.net/ss343fd/article/details/54880037)

- [编码方式区别](https://blog.csdn.net/weiaipan1314/article/details/112543447)

  



