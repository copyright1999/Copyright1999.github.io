---
layout:     post   				    # 使用的布局（不需要改）
title:     OnboardSDK3.8.1(三)			# 标题 
subtitle:  开始在板子上跑代码喽  #副标题
date:       2019-09-03				# 时间
author:     婷                               # 作者
header-img: img/21.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签

- stm32
- OSDK
- 飞控
- 串口
---

今天初步让代码在板子上跑，先看看打印出来的调试信息。

### 激活飞机

在Receive.cpp里面，需要你们自己去注册一个APP key跟APP ID。

![2.png](https://i.loli.net/2019/09/02/2tzF5oQlRuMVTPB.png)

进入[大疆开发者的网站](https://developer.dji.com/user/apps/#onboard),点击`CREATE APP`

![3.png](https://i.loli.net/2019/09/02/HwsQSTj8YLpfZFn.png)

如图

![4.png](https://i.loli.net/2019/09/02/ruJedBSpMGLKPoR.png)

创建之后就可以看到自己创建的一个APP，点进去就有你的APP key 跟 APP ID。我们需要 key 跟  ID来激活飞机，这一步是必不可缺的。

![5.png](https://i.loli.net/2019/09/02/AMsdHT9y64KoFQq.png)

得到key跟ID之后就将他们填入代码对应的地方就好啦。

### 开启API控制

要使用SDK，比如我手上用的是N3飞控，那么你就应该打开对应的[调参助手](<https://www.dji.com/cn/n3/info#downloads>)，使能API的控制，这样才能进行SDK的开发。（记得之前一开始接触的是m100，以为N3飞控的调参助手跟m100是同样的，最后发现是两个不同的版本，所以一定要找到对应的调参助手）。

打开调参助手之后选择SDK那一栏，然后记得勾选启动`启动API控制`，然后波特率的选择因为我代码里面是设置的115200，所以对应这边也应该是115200。

![01.png](https://i.loli.net/2019/09/02/QhowpNeT1UMFaYy.png)

![02.png](https://i.loli.net/2019/09/02/WIj6XOymTA52YnC.png)

### 接线

打开N3飞控的用户手册，查看其API的接线顺序

![04.png](https://i.loli.net/2019/09/02/HT3k846PqfmgOBs.png)

注意不要搞错线序，实物连接图如下

![001.png](https://i.loli.net/2019/09/03/MfcgiZpT5CojIP6.png)

板子上的logo是我们队伍的队徽，嘻嘻，虽然被我拍反了

![116.png](https://i.loli.net/2019/09/03/M4fXTUYZuhKdnWt.png)



### 烧录代码

然后打开串口调试助手，串口模块插入电脑，得到打印出来的调试信息

这是开头打印出来的调试信息

![003.png](https://i.loli.net/2019/09/03/fv2Vl38nYqM49Ga.png)

这是打印自己的飞控的SN，固件的一些信息

![004.png](https://i.loli.net/2019/09/03/DyZ7RMb8Juokc2Y.png)

然后是各种初始化之后，跑`telemetry sample`遥测样本（跑哪个样本可以自己选择，main.cpp开头的`sample_flag`可以自己选择。）

![005.png](https://i.loli.net/2019/09/03/HYBQ8wmXbgyvERs.png)

然后就是一些输出的信息，因为是N3飞控,所以只支持订阅功能，这个可以去[大疆开发者](<https://developer.dji.com/onboard-sdk/documentation/development-workflow/environment-setup.html#stm32>)的网站上去仔细阅读一下。

![007.png](https://i.loli.net/2019/09/03/ejcFamS1JIWroNq.png)

开始输出

![006.png](https://i.loli.net/2019/09/03/fAx8e2SlkuFNh7Y.png)

![008.png](https://i.loli.net/2019/09/03/ba6FxKQBsZiq74Y.png)

订阅信息发完之后`remove Package`

![009.png](https://i.loli.net/2019/09/03/sRpmFDenhWqP3jZ.png)



![010.png](https://i.loli.net/2019/09/03/C8YOnRWbUVo26T1.png)



### 注意

- 最好就是飞控先上电，等待飞控初始化完成之后再给板子上电。如果板子先上电的话可能飞控那边初始化没完成，这边板子程序上串口检测不到飞控发过来的信息，可能会把包含SDK信息的结构体从内存里面释放掉，这样子就没法用SDK了。可以再自己的代码里面初始化外设之后加一段延时再跑死循环来解决上电先后顺序的问题。

![011.png](https://i.loli.net/2019/09/03/ruC43kqRUIB5Qjx.png)

- 多阅读[大疆板载SDK](<https://developer.dji.com/onboard-sdk/documentation/development-workflow/environment-setup.html#stm32>)（OSDK）stm32开发平台给的文档









