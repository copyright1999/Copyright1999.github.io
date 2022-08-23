---
layout:     post   				    
title:     linux USB学习（四）			
subtitle:  configfs用户层实现gadget驱动
date:       2022-08-23				
author:     婷                             
header-img: img/76.jpg 	
catalog: true 						
tags:								

- USB

---





## 简介

这篇文章主要介绍如何通过`configfs`在应用层实现用户空间的`gadget`驱动



## U盘配置过程

内核打开`configfs`以及`usb`存储设备相关选项

`CONFIG_CONFIGFS_FS ` : 为用户空间提供访问配置内核驱动的`configfs`文件系统

![image-20220820233130622](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220820233130622.png)



`CONFIG_USB_LIBCOMPOSITE`：提供`usb gadget composite`框架

![image-20220822231747996](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822231747996.png)



`CONFIG_USB_CHIPIDEA`和`CONFIG_USB_CHIPIDEA_UDC`：打开`UDC(USB Device Controller)`的配置，配置硬件控制器

![image-20220822231432124](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822231432124.png)



![image-20220822231413515](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822231413515.png)



`CONFIG_USB_CONFIGFS_MASS_STORAGE`和`CONFIG_USB_F_MASS_STORAGE`：对应于`usb_f_mass_storage.ko`驱动



![image-20220820233043190](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220820233043190.png)



![image-20220822232042475](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822232042475.png)



其他相关的选项（不知道是不是需要勾选的，反正都勾选了）

![image-20220820233139709](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220820233139709.png)



![image-20220820233106139](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220820233106139.png)





然后编译内核以及驱动，更新设备的内核，加载相关驱动，挂载`configfs`

```bash
insmod configfs.ko
insmod libcomposite.ko
insmod usb_f_mass_storage.ko
mount -t configfs none /sys/kernel/config/
```



![image-20220821002101262](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821002101262.png)



创建`g1`实例

```bash
cd /sys/kernel/config/
ls
cd usb_gadget/
ls
mkdir g1
cd g1
ls
```



![image-20220821002138940](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821002138940.png)



设置`USB`版本，`PID`，` VID`。（这里的`PID`和` VID`的设置是我读取我的U盘 `lsusb`得到的，不过这个应该可以自己随便设置的）

```bash
echo 0x0200 > bcdUSB
echo "0x0781" > idVendor
echo "0x5151" > idProduct
```



![image-20220821002229621](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821002229621.png)



![image-20220821002353061](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821002353061.png)



创建并设置`strings`目录

```bash
mkdir strings/0x409
ls strings/0x409/
echo "12345678" > strings/0x409/serialnumber
echo "copyright" > strings/0x409/manufacturer
echo "moniupan" > strings/0x409/product
```



![image-20220821002450532](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821002450532.png)



![image-20220821002646057](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821002646057.png)



创建`configuration`和字符串，注意第三行的名字必须按照你加载的驱动后面的`usb_f_*.ko`来填写，比如我们加载的是`usb_f_mass_storage.ko`，那么这里的`*`就是`mass_storage`。如果不按照这样来的话，下一步创建`funcitions`会提示错误。

```bash
mkdir configs/c.1
mkdir configs/c.1/strings/0x409
echo "mass_storage" > configs/c.1/strings/0x409/configuration
```



![image-20220821003144344](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821003144344.png)





创建`funcitions`

```bash
root@copyright:/sys/kernel/config/usb_gadget/g1# mkdir functions/mass_storage.0
[  822.354698] Mass Storage Function, version: 2009/09/11
[  822.361138] LUN: removable file: (no medium)
```



![image-20220821003153563](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821003153563.png)





将`functions`和`configuration`关联起来

```bash
root@copyright:/sys/kernel/config/usb_gadget/g1# ln -s functions/mass_storage.0 configs/c.1
[  986.215888] Number of LUNs=1
root@copyright:/sys/kernel/config/usb_gadget/g1#
```



![image-20220821003415213](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821003415213.png)



绑定到`UDC`，使能`gadget`

```bash
root@copyright:/sys/kernel/config/usb_gadget/g1# ls -l /sys/class/udc/
total 0
lrwxrwxrwx 1 root root 0 Jun 30 03:06 ci_hdrc.0 -> ../../devices/platform/soc/2100000.aips-bus/2184000.usb/ci_hdrc.0/udc/ci_hdrc.0
root@copyright:/sys/kernel/config/usb_gadget/g1# echo ci_hdrc.0 > UDC
```



![image-20220821003527733](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821003527733.png)

切记`echo`的时候不要加上绝对路径，只要把`UDC`的名字写入即可

![image-20220821003700283](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821003700283.png)





## U盘验证过程

这时候板子链接我的电脑，`Ubuntu`系统，`dmesg`下的枚举信息如下，可以看到

![image-20220821004250166](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821004250166.png)

但是这个时候还没有找到分区信息，因为前面的时候还没配置`U盘`要使用的是哪个分区



我的板子弹出的内核打印

![image-20220821010200459](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821010200459.png)

### 添加U盘存储分区配置

不清楚自己板子上哪块分区可以拿来做实验，所以我的想法是，板子外接一个`SD`卡，拿`SD`卡的分区来做为模拟`U盘`的分区，板子插上`SD`卡显示的分区是`/dev/sdb1`

![image-20220821004414322](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821004414322.png)



![image-20220821004423865](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821004423865.png)



![image-20220821004443053](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821004443053.png)



### U盘划分一个分区

这里介绍划分一个分区的做法，划分两个分区的做法可以[参考链接一](https://whycan.com/t_2903.html)，输入如下命令配置

```bash
cd functions/mass_storage.0/
ls
echo /dev/sdb1 > lun.0/file
echo 1 > lun.0/removable
echo 0 > lun.0/nofua
```



![image-20220821005651449](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821005651449.png)



![image-20220821005759442](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821005759442.png)



### 添加分区后验证

添加分区后，我的`Ubuntu`系统，`dmesg`一下可以看到`sdb`分区的打印

![image-20220821010218090](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821010218090.png)



电脑上也识别到这个模拟`U盘`

![image-20220822234429697](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822234429697.png)



然后尝试接到我的`Windows`，貌似这两系统的驱动还不一样，反正`Windows`没识别出来，更新驱动也没用，先不管啦

![image-20220821010754784](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821010754784.png)





![image-20220821010811357](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220821010811357.png)







## CDC-NCM配置过程

相关的内核项打开，过程类似前面的`U盘`配置过程，这里就不赘述了

![image-20220822220742575](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822220742575.png)



![image-20220822220828813](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822220828813.png)



升级内核，加载驱动

```bash
insmod configfs.ko
insmod libcomposite.ko
insmod u_ether.ko
insmod usb_f_ncm.ko
```



![image-20220822221444499](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822221444499.png)

挂载`configfs`，创建`g2`实例

```bash
mount -t configfs none /sys/kernel/config/
cd /sys/kernel/config/
ls
cd usb_gadget/
mkdir g2
cd g2/
ls
```



![image-20220822221553603](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822221553603.png)



设置`USB`版本，`PID`，` VID`。（这里的`PID`和` VID`的设置我是根据内核的`ncm.c`中来的）

```bash
echo 0x0200 > bcdUSB
echo 0x0525 > idVendor
echo 0xa4a1 > idProduct
```



![image-20220822221908730](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822221908730.png)



![image-20220822221730507](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822221730507.png)





创建并设置`strings`目录

```bash
mkdir strings/0x409
echo "123456789" > strings/0x409/serialnumber
echo "copyright_manufacture" > strings/0x409/manufacturer
echo "copyright_product" > strings/0x409/product
```



![image-20220822222226880](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822222226880.png)



创建`configuration`和字符串，注意第三行的名字是`ncm`

```bash
mkdir configs/c.1
mkdir configs/c.1/strings/0x409
echo "ncm" > configs/c.1/strings/0x409/configuration
```



![image-20220822223747292](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822223747292.png)





创建`funcitions`

```bash
root@copyright:/sys/kernel/config/usb_gadget/g2# mkdir functions/ncm.0
[  260.395518] using random self ethernet address
[  260.400003] using random host ethernet address
```



![image-20220822223840325](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822223840325.png)





将`functions`和`configuration`关联起来

```bash
ln -s functions/ncm.0 configs/c.1
```



![image-20220822223928649](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822223928649.png)



绑定到`UDC`，使能`gadget`

```bash
root@copyright:/sys/kernel/config/usb_gadget/g2# ls -l /sys/class/udc/
total 0
lrwxrwxrwx 1 root root 0 Jun 30 03:58 ci_hdrc.0 -> ../../devices/platform/soc/2100000.aips-bus/2184000.usb/ci_hdrc.0/udc/ci_hdrc.0
root@copyright:/sys/kernel/config/usb_gadget/g2# echo ci_hdrc.0 > UDC
[  193.443011] usb0: HOST MAC 56:1f:48:5f:28:9d
[  193.461533] usb0: MAC f6:d4:5f:72:f4:75
root@copyright:/sys/kernel/config/usb_gadget/g2# [  193.621532] IPv6: ADDRCONF(NETDEV_UP): usb0: link is not ready

```



![image-20220822224802905](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822224802905.png)



板子上查看发现`usb0`节点已经有了

```bash
root@copyright:/sys/kernel/config/usb_gadget/g2# ifconfig -a | grep usb
usb0      Link encap:Ethernet  HWaddr f6:d4:5f:72:f4:75
root@copyright:/sys/kernel/config/usb_gadget/g2# ifconfig usb0
usb0      Link encap:Ethernet  HWaddr f6:d4:5f:72:f4:75
          inet addr:192.168.7.2  Bcast:192.168.7.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```



![image-20220822224845518](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822224845518.png)



## CDC-NCM验证过程

配置好后，将板子上的`usb device`接口接到我的`PC`，`Ubuntu`系统弹出的内核热插拔消息

```bash
[5034810.987491] usb 1-2: new high-speed USB device number 84 using xhci_hcd
[5034811.137990] usb 1-2: New USB device found, idVendor=0525, idProduct=a4a1, bcdDevice= 4.01
[5034811.137999] usb 1-2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[5034811.138004] usb 1-2: Product: copyright_product
[5034811.138008] usb 1-2: Manufacturer: copyright_manufacture
[5034811.138013] usb 1-2: SerialNumber: 123456789
[5034811.165335] cdc_ncm 1-2:1.0: MAC-Address: 56:1f:48:5f:28:9d
[5034811.165807] cdc_ncm 1-2:1.0 usb0: register 'cdc_ncm' at usb-0000:00:14.0-2, CDC NCM, 56:1f:48:5f:28:9d
[5034811.213378] cdc_ncm 1-2:1.0 enp0s20f0u2: renamed from usb0
[5034811.254119] cdc_ncm 1-2:1.0 enp0s20f0u2: 425 mbit/s downlink 425 mbit/s uplink
[5034811.318224] IPv6: ADDRCONF(NETDEV_CHANGE): enp0s20f0u2: link becomes ready

```



![image-20220823220131373](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220823220131373.png)

可以看到`enp0s20f0u2: renamed from usb0`和`enp0s20f0u2: link becomes ready`等关键打印，`ifconfig`命令查看，发现`enp0s20f0u2`节点已经有了

![image-20220822225102372](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822225102372.png)



设置`IP`（这里一定要设置子网掩码）

![image-20220822225423896](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822225423896.png)



我的`PC`设置好了后，板子这边的打印`usb0 : link becomes ready`

![image-20220822224951525](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822224951525.png)



板子这边`ping`我的`PC`

![image-20220822225518923](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822225518923.png)



我的`PC`来`ping`我的板子

![image-20220822225559463](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220822225559463.png)





## 参考链接

- [参考链接一，最强教程](https://whycan.com/t_2903.html)
- [参考链接二](https://blog.csdn.net/hktkfly6/article/details/72846218)





## 其他问题

- 参考链接一中有实现两个独立的`U`盘的方法，后续可以自己试试
- 在测试的时候发现，不知道是不是我板子的问题，无论是配置成`U`盘还是`NCM`设备，时不时会自动断联又重联

- `drivers/usb/gadget`目录下`ko`不是有两个目录嘛`function`跟`legacy`，之前不懂有什么差别，这次使用了`configfs`，大概知道了就是，`function`是放可以通过用户态配置的驱动，而`legacy`则是帮你都做好了的

  ![image-20220823221127313](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb4/image-20220823221127313.png)

- `chipidea`不清楚是什么，是一种`usb`设计的规范？

















