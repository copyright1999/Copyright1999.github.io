---
layout:     post   				   
title:     windows安装cygwim		
subtitle:  
date:       2022-1-22	
author:     婷                               
header-img: img/62.jpg
catalog: true 						
tags:								

- windows
- cygwin

---





## 前言

本来想在`windows`下利用`git bash`终端来实现一些`linux`下常用的 `tree`， `grep`， `find`等命令，在编译`tree`源码的时候碰到 `windows`下编译 `posix c`的问题，最终通过`cygwin`解决。





## cygwin安装

来到[官网](http://www.cygwin.com/)，选择红框的安装程序

![image-20220116232351880](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116232351880.png)



下载后运行，选择下一步

![image-20220116233053767](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233053767.png)





选择`Install from Internet`

![image-20220116233119335](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233119335.png)

 

选择`Root Directory`，这里可以理解为你要把一个`linux`系统安装在哪里，或者说你想把根文件系统放在哪里，有点类似我们安装`wsl`那样

![image-20220116233228659](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233228659.png)



选择`Local Package Directory`，相当于是把一些程序包存放在本地

![image-20220116233248625](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233248625.png)



网络代理的选择，这里是直连

![image-20220116233327199](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233327199.png)



镜像列表的选择，选择第一个单击即可

![image-20220116233407395](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233407395.png)



然后等待安装列表弹出来

![image-20220116233449601](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233449601.png)



选项出来后，直接点击下一步，一般都是默认安装些常用的命令

![image-20220116233519646](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233519646.png)



![image-20220116233618748](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233618748.png)



等待安装

![image-20220116233647210](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233647210.png)



安装完成后，勾选创建快捷菜单

![image-20220116233725910](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233725910.png)

点开桌面的快捷方式查看，出现了`$`符号，也就是说安装成功了

![image-20220116233809881](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233809881.png)



这个是之前安装的时候选择的“根文件系统”路径

![image-20220116233931398](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233931398.png)



 

接下来安装一些常用的组件，还是打开前面从官网下载的可执行程序**（安装，更新都是通过这个可执行程序来做的）**

 

找到`Devel`展开，从中选择`binutils`，`gcc` ，`mingw` ，`gdb`进行安装，找到以下选项，点击右边的`skip`，使其变为版本号即可

![image-20220116233949957](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116233949957.png)



![image-20220116234017975](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116234017975.png)



![image-20220116234032572](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116234032572.png)

 

![image-20220116234058716](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116234058716.png)





![image-20220116234123494](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116234123494.png)

 

  

选择完毕后点击确认

![image-20220116234145333](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116234145333.png)



等待安装完成

![image-20220116234202074](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220116234202074.png)





 

## 添加cygwin到右键菜单

打开安装程序，按照图中红框所示，搜索`chere`，选择一个版本进行安装



![image-20220117003358722](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220117003358722.png)



![image-20220117003419747](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220117003419747.png)

 

 

安装完成后，右键电脑上的快捷方式，以管理员身份运行，输入

```
chere -i -t mintty -s bash
```

 

![image-20220117003456694](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220117003456694.png)



这时候右击菜单就已经有`Bash Prompt Here`选项

![image-20220117003916674](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220117003916674.png)

 

接下来给他增加个图标，命令行输入`regedit`

![image-20220117003931242](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220117003931242.png)



 

打开注册表，输入

```
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\background\shell\cygwin64_bash
```

来到这个路径下



![image-20220117003952868](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220117003952868.png)







 

 

新建一个字符串值，名为`Icon`

![image-20220117004006886](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220117004006886.png)



![image-20220117004021646](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220117004021646.png)



创建好后双击，填入图标的文件路径`C:\cygwin64\Cygwin.ico`

![image-20220117003631318](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220117003631318.png)





 

## 给cygwin添加管理员权限

右击桌面上的快捷方式，点击`兼容性`，如图勾上红框中的选项

![image-20220117003559263](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows%E5%AE%89%E8%A3%85cygwin/image-20220117003559263.png)



**注意：**一般来说是不用设置管理员身份运行，如果`cygwin`用管理员模式的话，可能有时候无法访问电脑上挂载的网络盘，这时候取消管理员身份运行即可。





## 链接

- [链接一](https://blog.csdn.net/u010356768/article/details/90756742)
- [链接二](https://blog.csdn.net/zuijinhaoma8/article/details/39993435?spm=1001.2014.3001.5501)
- [链接三](https://jingyan.baidu.com/article/6b97984d83dfe51ca2b0bf0e.html)
- [链接四](https://blog.csdn.net/yuisyu/article/details/81293333)
- [链接五](https://www.cnblogs.com/schips/p/10743529.html)











 









