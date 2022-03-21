---
layout:     post   				   
title:      linux USB学习（一）			
subtitle:   USB驱动实验
date:       2022-03-15				
author:     婷                               
header-img: img/66.jpg 	
catalog: true 						
tags:								

- USB

---



## 前言

记录下自己做的`USB`驱动的实验。实验主要分两种，一种是板子做`USB`主机的实验，另一个种则是做从机的实验。



## USB主机实验

### USB鼠标实验 

`USB` 鼠标键盘属于 `HID `设备。`HID`，翻译过来就是人体学接口设备，主要是一些人机交互的设备，比如键盘，鼠标，摇杆，绘图板等等。如果想要实现`USB`鼠标功能，分两步走，需要内核**使能通用HID驱动**和**鼠标的驱动**。



#### 使能通用HID驱动

`make menuconfig`，然后按照此路径去使能**HID驱动**

```shell
Device Drivers -> HID support -> HID bus support -> Generic HID driver 
```



![image-20220227213116084](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227213116084.png)



![image-20220227213156890](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227213156890.png)



![image-20220227213251328](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227213251328.png)



#### 使能鼠标驱动

路径跟前面一样，这里选择`USB HID support->USB HID transport layer`

![image-20220227213515497](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227213515497.png)



![image-20220227213540962](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227213540962.png)



#### 鼠标测试

查看加载的驱动

![image-20220313213718735](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220313213718735.png)



查看热插拔信息

![image-20220227213925853](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227213925853.png)



```shell
[ 8740.902614] usb 2-1.1: new full-speed USB device number 3 using ci_hdrc
[ 8741.030362] input: 2.4G Mouse as /devices/platform/soc/2100000.aips-bus/2184200.usb/ci_hdrc.1/usb2/2-1/2-1.1/2-1.1:1.0/0003:1EA7:0064.0001/input/input3
[ 8741.062296] hid-generic 0003:1EA7:0064.0001: input: USB HID v1.10 Mouse [2.4G Mouse] on usb-ci_hdrc.1-1.1/input0
```

鼠标在板子上可以像`PC`那样使用，测试成功！



鼠标插入识别后，`/dev/input`目录下新增了`event3`和`mouse1`

![image-20220227215311424](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227215311424.png)



输入`hexdump event3`可以看到一些信息，至于是什么就没有去深究了，可以理解为是这些`HID`驱动的一个接口输出的信息。**（个人理解）**

![image-20220227215451596](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227215451596.png)









#### 特殊的HID驱动

既然前面说的**使能通用的HID驱动**，那肯定有**特殊的HID驱动**，点击`Special HID drivers`进去看看

![image-20220227215954708](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227215954708.png)

可以查看`Apple{i,Power,Mac}Books`这项内核描述

![image-20220227220055576](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227220055576.png)

主要还是针对苹果的产品，不得不感慨内核的强大

![image-20220227220123538](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227220123538.png)







### USB  U盘实验 

#### 使能SCSI协议

`U 盘`使用 `SCSI `协议，因此要先使能 `Linux `内核中的 `SCSI `协议，配置路径如下：` Device Drivers -> SCSI device support ->  SCSI disk support` 

![image-20220227234103371](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227234103371.png)

#### 使能U盘驱动

我们还需要使能 `USB Mass Storage`，也就是 `USB` 接口的大容量存储设备，配置路径如下： `Device Drivers -> USB support -> Support for Host-side USB ->  USB Mass Storage support `

![image-20220227234216771](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227234216771.png)

点击这个`USB Mass Storage support`的配置项，也是提及到要使能`SCSI`协议

![image-20220227234252715](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220227234252715.png)



#### U盘测试

查看加载的驱动

![image-20220313213921771](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220313213921771.png)

准备一个 `FAT32` 格式的U盘，`NTFS` 和 `exFAT` 由于版权问题所以在 `Linux `下支持的不完善，操作的话可能会有问题，比如只能读，不能写或者无法识别等。

![image-20220313214108568](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220313214108568.png)



统检测到 U 盘插入，大小为 `7.94GB`，对应的设备文件为`/dev/sdb` 和`/dev/sdb1`，`/dev/sdb` 是整个 U 盘，`/dev/sdb1` 是 U 盘的第一个分区，我们一般使用` U 盘`的时候都是只有一个分区。要想访问` U 盘`我们需要先对` U 盘`进行挂载，理论上挂载到任意一个目录下都可以，这里我创建一个 `/mnt/usb_disk` 目录，然后将 `U 盘`挂载到`/mnt/usb_disk` 目录下

```shell
mkdir /mnt/usb_disk -p
mount /dev/sdb1 /mnt/usb_disk/ -t vfat -o iocharset=utf8
```

![image-20220313215946374](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220313215946374.png)

对`U盘`内的空文件`readme2.txt`写入`test`，然后`sync`，再执行卸载命令。拔出`U盘`。

![image-20220313220323470](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220313220323470.png)

重新插入`U盘`，重新挂载，查看是否写入。

![image-20220313220440476](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220313220440476.png)





## USB从机实验

### USB声卡实验

板子上板载了音频解码芯片，因此可以把板子当做一个外置 USB 声卡，配置 Linux 内核，路径如下：`Device Drivers -> USB support  -> USB Gadget Support -> USB Gadget Drivers  ->Audio Gadget  ->UAC 1.0 (Legacy) `



![image-20220313221806767](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220313221806767.png)



注这里编译为驱动模块，配置完成以后重新编译内核，得到三个声卡相关驱动

```shell
drivers/usb/gadget/libcomposite.ko 
drivers/usb/gadget/function/usb_f_uac1.ko 
drivers/usb/gadget/legacy/g_audio.ko 
```

升级内核后，依次加载上述三个驱动，但是在加载的时候 ，发现加载第一个`libcomposite.ko`就出现了符号表的错误，最后是找到了`configfs.ko`才成功加载驱动。

![image-20220314000452883](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220314000452883.png)



加载的时候报错

![image-20220314000640313](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220314000640313.png)

根据关键词查询`config_group_init`

![image-20220314000733751](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220314000733751.png)

最后将`fs/configfs`这个路径下的`configfs.ko`加载之后就不会报错了，具体啥原因还不清楚，但是符号表的问题总算是解决了。

 

加载完驱动后，用`USB`连接线，将电脑连接到主机的`USB`接口，这时候打开设备管理器，总是提示识别到**未知的USB设备**。实验到此也无法进行下去，猜测可能是我跟我电脑是`win11`有关？



### 模拟U盘

**模拟 U 盘**实验就是将开发板当做一个 **U 盘**，配置 Linux，路径如下： `Device Drivers -> USB support  -> USB Gadget Support  -> USB Gadget Drivers ->Mass Storage Gadget `

![image-20220313221644152](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220313221644152.png)



配置完成以后重新编译内核，得到三个相关驱动

```shell
drivers/usb/gadget/libcomposite.ko 
drivers/usb/gadget/function/usb_f_mass_storage.ko 
drivers/usb/gadget/legacy/g_mass_storage.ko 
```



升级内核后，输入下列命令，依次加载驱动

```shell
insmod configfs.ko
insmod libcomposite.ko
insmod usb_f_mass_storage.ko
insmod g_mass_storage.ko file=/dev/sdb1 removable=1 
```

加载 `g_mass_storage.ko `的时候使用 `file` 参数指定使用的大容量存储设备，加载成功后可以看到电脑识别到了U盘设备

![image-20220315004211606](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220315004211606.png)

打开内容查看

![image-20220315004219852](https://gitee.com/copyright1999/image-typora-markdown/raw/master/usb_learn1/image-20220315004219852.png)



操作完成后要退出的话执行命令

```shell
rmmod g_mass_storage.ko
```



