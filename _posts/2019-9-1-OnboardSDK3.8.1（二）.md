---
layout:     post   				    # 使用的布局（不需要改）
title:     OnboardSDK3.8.1(二)			# 标题 
subtitle:  代码上修改串口配置  #副标题
date:       2019-09-01				# 时间
author:     婷                               # 作者
header-img: img/desk_head.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签

- stm32
- OSDK
- 飞控
- 串口
---



# 引言

因为OSDK输出调试信息用的是串口2，然而自己手头上的板子并没有将串口2的接口引出来，所以我将调试串口改成串口4，方便看输出的信息，记得之前自己改这里的时候踩过不少坑，所以想记录一下

# 修改配置

首先新建一个文件夹，我命名为Code,主要是放置自己的代码

![1.png](https://i.loli.net/2019/09/01/dSbo32Ar1MGC6JW.png)

在魔术棒里面添加这个文件夹，添加文件

![2.png](https://i.loli.net/2019/09/01/OBX9VuJtnqKgl1v.png)

头文件的路径也要添加

![3.png](https://i.loli.net/2019/09/01/AE96Ukr3uZm2Ic4.png)

然后开始写代码  `ctrl+c` `ctrl+v`      ~~纠正一下应该是uart4（自己写错了懒得改了）~~

记得在usart4.h里面说明  `using namespace  DJI::OSDK`这样子就可以使用DJI的类

![4.png](https://i.loli.net/2019/09/01/eo8WtQ4GKCRcMvD.png)

一开始我是这样子写的，没注意到一个小地方就报了个错

![5.png](https://i.loli.net/2019/09/01/QPlCpJmMG43goEn.png)

```c
..\Code\usart4.h(6): error:  #276: name followed by "::" must be a class or namespace name  using namespace DJI::OSDK;
```



报错是因为在包含头文件前就使用了他的命名空间，将它放在头文件之后就可以了

![6.png](https://i.loli.net/2019/09/01/umwEe3kWUFLHt7i.png)

再来到main.cpp这里  

![7.png](https://i.loli.net/2019/09/01/F8nbJNdhk4f9OVp.png)

`F12`进入BSPinit()这个函数里面

![8.png](https://i.loli.net/2019/09/01/rc1PCXJAimv5lBU.png)

进入Usartconfig()这个函数

![9.png](https://i.loli.net/2019/09/01/RTWxBY3dlqobO7M.png)

注释掉串口2的初始化函数，然后依葫芦画瓢，将串口2的初始化代码复制到自己的串口4初始化代码，   然后改一下一些对应的配置就行，详细过程就不贴图说明了

最后记得改中断分组里面的串口2换成串口4(改红框出的代码)

![10.png](https://i.loli.net/2019/09/01/bl4YrDGdZ582SnK.png)

接下来就是重定向printf函数的问题

来到`cppforstm32.cpp`,改动红色框里面的位置，换成UART4

![11.png](https://i.loli.net/2019/09/01/lyJPazuxWq5VXsO.png)

改动后如图

![12.png](https://i.loli.net/2019/09/01/MQFyDwxs56GSUKm.png)

然后来到Receive.cpp这里，将串口2中断注释掉

![13.png](https://i.loli.net/2019/09/01/u9g6nc1r38qyHGp.png)

同时将中断里面的内容复制到串口4中断

![14.png](https://i.loli.net/2019/09/01/hmL8fvwTRI5nOXd.png)



# 结尾

今天先暂时写到这里吧，大概修改调试串口的配置就这几个地方要注意的，明天还要早起为祖国升国旗，下一次就准备开始将代码烧进板子喽



