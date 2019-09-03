---
layout:     post   				    # 使用的布局（不需要改）
title:     OnboardSDK3.8.1(一)			# 标题 
subtitle:  stm32下开发sdk的准备工作  #副标题
date:       2019-08-30				# 时间
author:     婷                               # 作者
header-img: img/18.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签

- stm32
- OSDK
- 飞控
---



# 引言

&#160;&#160;&#160;&#160;今年六月份因为比赛需求大概看了一个月大疆N3飞控的SDK代码吧，那时候还没学C++，又是第一次接触，而OnboardSDK3.3版本的代码对前面的代码进行了大改很多写法命名也不一样，八月份比完赛后官方开源了一个室内定位的方案，而且据说guidance也要停产，现在也即将大三了，我想把我大二没做完的事情做完，说不定今年比赛就真的要用上SDK了，也可以帮助到新队员。

# 准备

## 硬件配置

板子用的是stm32 F405RG，飞控用的是N3飞控

## 修改代码

### 修改晶振

&#160;&#160;&#160;&#160;首先自己的板子是用的F405RG,晶振是25Mhz，需要对官方的例程做点修改,改外部晶振,有两个地方要改一个是头文件`stm32f4xx.h`，一个是源文件`system_stm32f4xx.c`

①修改HSE_VALUE，修改为25Mhz

![1.png](https://i.loli.net/2019/08/30/X32C8RB6Uh9JokI.png)

②修改PLL_M  修改为25

![2.png](https://i.loli.net/2019/08/30/gKr4ifQVvq86tPD.png)

③选芯片型号

![3.png](https://i.loli.net/2019/08/30/Ymh58SbADiPtEl3.png)



### 删繁就简

接下来我会把大疆的库osdk-core这个文件夹里面的.c  .h弄成一个只包含src  inc 的文件夹(只能说是个人习惯吧，用不上的东西我只想删，~~断舍离~~)

![4.png](https://i.loli.net/2019/08/30/crGqYbK3mzo9eNg.png)



下面的这几个被我选中的文件我是直接删掉的，然后在剩下的的文件夹里面头文件放在inc，源文件放在src

![5.png](https://i.loli.net/2019/08/30/dLkqSxpVyYhzv5l.png)

有一点要注意的是这个platform文件夹，如下图所示，被我选中的都删掉，最后只需要剩下default跟STM32两个文件夹里面的内容就好

![6.png](https://i.loli.net/2019/08/30/BL8ozvE5jkfR1xQ.png)

处理之后的osdk-core文件夹的内部就是这样的啦

![0000.png](https://i.loli.net/2019/08/30/jl6aqfgDmLVzceK.png)

接下来在keil5里面的选项卡那一栏将DJIlib删掉，然后再重新添加文件

![7.png](https://i.loli.net/2019/08/30/qlOc8aJNiEK2Meu.png)

`点击那个 X 移除懂得都懂吧`

再重新建一个DJIlib

![8.png](https://i.loli.net/2019/08/30/NUXocKq1xeQr8n9.png)

添加osdkcore/src里面的所有c++源文件

![8.png](https://i.loli.net/2019/08/30/NUXocKq1xeQr8n9.png)

![9.png](https://i.loli.net/2019/08/30/zU6y7OZcdBkKXnp.png)



添加完成之后修改包含的头文件的路径

![11111.png](https://i.loli.net/2019/08/30/lkzmaTihJAU18WG.png)



编译`0 Error(s),0 Warning(s).`

还有说明的一点是，编译之前会有一个这个编译器的warning

![10.png](https://i.loli.net/2019/08/30/sMHNZ9vVGcnrQWp.png)



```c
*** Warning: You are compiling one or more files of source type C++ and have selected 'use MicroLIB'. MicroLIB does not support C++!
```

这句话的意思是指MicoLIB不支持C++，但是不用担心这一部分，因为勾选MicroLIB只是为了重定向printf函数，而那部分代码已经用`extern "C"`修饰。

`   extern "C"` 的作用是让 C++ 编译器将 `extern "C"` 声明的代码当作 C 语言代码处理，可以避免 C++ 因符号修饰导致代码不能和C语言库中的符号进行链接的问题。





