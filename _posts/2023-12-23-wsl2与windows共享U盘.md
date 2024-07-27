---
layout:     post   				    
title:     wsl2与windows共享U盘			
subtitle:  
date:       2023-12-23		
author:     婷                               
header-img: img/106.png 	
catalog: true 						
tags:								

- win11
- wsl2
- usb

---





## windows准备

点击下面的链接

```bash
https://github.com/dorssel/usbipd-win/releases
```



下载`usbipd-win_3.1.0.msi`文件，一定要选择`3.1.0`版本！！！高版本目前好像是有些使用的问题，比较麻烦。

![image-20231211231710478](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231211231710478.png)



下载后双击

![image-20231211232338743](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231211232338743.png)

点击`Install`

![image-20231211232354919](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231211232354919.png)

一路安装过去之后，这里就完成了

![image-20231211232418537](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231211232418537.png)



用管理员模式打开`Powershell`，输入`usbipd wsl list`查看是否安装成功。

![image-20231211232558664](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231211232558664.png)





## wsl准备

```bash
sudo apt install linux-tools-generic hwdata
```



![image-20231211225539810](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231211225539810.png)



```bash
sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*-generic/usbip 20
```



![image-20231211225621418](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231211225621418.png)

同时还需要`wsl`的内核支持`usbstorage`相关的驱动，如果没有支持，可以参考之前的[文章](https://copyright1999.github.io/2023/12/23/%E7%BC%96%E8%AF%91wsl%E5%86%85%E6%A0%B8/)



## 重启

上述`windows`跟`wsl`准备工作做好后，电脑**一定要**重启，不然可能会有奇奇怪怪的问题





## wsl接入usb设备

`windows`这边用管理员模式打开`Powershell`，输入`usbipd wsl list`，可以看到`BUSID`为`4-1`的`U`盘设备

![image-20231223113620526](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231223113620526.png)



输入`usbipd wsl attach --busid 4-1`将`usb`设备加到`wsl`中，之后输入`usbipd wsl list`，可看到`STATE`一栏已经为`Attached-WSL`的状态

![image-20231211233137869](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231211233137869.png)



在`wsl`这边从`dmesg`跟`lsusb`中都可以看到已经识别到该`usb`设备

![image-20231211233332558](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231211233332558.png)



![image-20231211233151617](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231211233151617.png)



`dmesg`中也有`SCSI`相关的打印

![image-20231223115116269](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231223115116269.png)



输入`lsblk`查看，可以看到我们`/dev/sdd`

![image-20231223115147868](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231223115147868.png)

如果说`dmesg`中有识别到`U盘`，但是`lsblk`没有相关的分区，则可能是因为内核没有打开`SCSI`相关选项，可以参考之前的[文章](https://copyright1999.github.io/2023/12/23/%E7%BC%96%E8%AF%91wsl%E5%86%85%E6%A0%B8/)





## wsl卸载usb设备

在 `WSL `中完成设备使用后，可物理断开` USB `设备，或者在管理员模式下从` PowerShell `运行此命令：

```powershell
usbipd wsl detach --busid <busid>
```







## 其他

`wsl`设备也可以直接访问`u盘`，就是再通过一层转接，跟`wsl`访问`d盘`一样的道理，参考[链接](https://blog.csdn.net/weixin_43224373/article/details/105178324)



如图，有个盘符为`E`的`U盘`

![image-20231212000111484](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231212000111484.png)



`wsl`下创建文件夹

![image-20231212000144873](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231212000144873.png)



执行挂载命令`mount -t drvfs E: /mnt/e`

![image-20231212000308541](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231212000308541.png)

然后就可以正常访问了

![image-20231212000347492](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_misc/image-20231212000347492.png)



如果想在`windows`下正常弹出，需要先`umount`





## 参考链接

- [wsl支持接入usb](https://learn.microsoft.com/zh-cn/windows/wsl/connect-usb#attach-a-usb-device)
- [参考链接一](https://github.com/dorssel/usbipd-win/releases)
- [参考链接二](https://blog.csdn.net/Reasonss/article/details/124376484?spm=1001.2014.3001.5502)











