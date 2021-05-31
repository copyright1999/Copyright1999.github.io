---
layout:     post   				    # 使用的布局（不需要改）
title:     Windows10下用vscode开发stm32		# 标题 
subtitle:    #副标题
date:       2021-02-13				# 时间
author:     婷                               # 作者
header-img: img/50.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签

- win10
- stm32
- vscode

---



### 前言

本文主要是介绍在`win10`的环境下，利用`cubemx`，`openocd`，`vscode`软件以及烧录工具`stlink`来实现`stm32`的开发。~~（主要是去实习了，公司要求换个环境开发）~~



### 安装清单

- vscode
- MinGw
- 交叉编译工具链
- openocd
- cubemx
- stlink驱动



#### 安装vscode

直接去[官网](https://code.visualstudio.com/)下载安装程序即可，或者可以参考这篇[博客](https://www.jianshu.com/p/51dfbe2c9583)。



#### 安装MinGW

来到[sourceforge网站](https://sourceforge.net/projects/mingw-w64/files/mingw-w64/mingw-w64-release/)下载，选择红框的压缩包，点击下载

![1.png](https://s3.ax1x.com/2021/01/27/sxJ0SA.png)



下载后解压缩运行，最终在桌面可以看到下面MinGW的在线安装器的图标**（如果在桌面看不到就去开始菜单搜索）**。这只是个安装器，需要在线下载真正的安装内容，所以速度不会快。

![2.png](https://s3.ax1x.com/2021/01/27/sxJaJH.png)

点击图标，跳出下面这个界面，点击这些组件旁边的框框，点击`mark for install`。

![3.png](https://s3.ax1x.com/2021/01/27/sxJdWd.png)



选择下面这些需要的组件，有的可能已经先帮你安装好了。

![4.png](https://s3.ax1x.com/2021/01/27/sxJtoD.png)



![5.png](https://s3.ax1x.com/2021/01/27/sxJUFe.png)



![6](https://s3.ax1x.com/2021/01/27/szktbt.png)

![7](https://s3.ax1x.com/2021/01/27/szkYDI.png)

选择完毕后点击`Installation`。

![8](https://s3.ax1x.com/2021/01/27/szk3gH.png)

等待安装成功后，要进行环境变量的设置。右键`我的电脑`属性，点击`高级系统设置`。然后按照下图的操作，记得添加之后要一路点击`确定`。

![9](https://s3.ax1x.com/2021/01/27/szkJKA.md.png)

![10](https://s3.ax1x.com/2021/01/27/szk8vd.png)



点击`Path`

![11](https://s3.ax1x.com/2021/01/27/szkaUf.png)



我的安装路径是 `C:\MinGW`，但是那些可执行文件在`bin`文件夹内，所以记得路径要包括到`bin`。

![12](https://s3.ax1x.com/2021/01/27/szkd58.png)



最后一路点击`确定`。



#### 安装vscode常用插件

- ARM：使的ARM汇编代码获得语法高亮，这里主要用在启动文件上。

- C/C++：使VScode获得对C/C++语言的支持，包括智能提示，调试等。

- Chinese (Simplified) Language Pack for Visual Studio Code：VScode中文语言包。

- C++ Intellisense：提供C++智能感知功能。

- Cortex-Debug：对ARM Cortex-M内核的单片机提供调试支持。

- Cortex-Debug: Device Support Pack - STM32F1(F4,L1)：这三个芯片包分别对Cortex-Debug提供三款芯片的设备支持。

- GBKtoUTF8：因为平时的Keil的STM32工程文件大部分都是GB2312编码的，这个扩展可以自动将GB2312 转换为UTF-8编码。

- LLVM ：代码补全。

  

![13](https://s3.ax1x.com/2021/01/27/szkB8g.md.png)

![14](https://s3.ax1x.com/2021/01/27/szkD2Q.md.png)

![15](https://s3.ax1x.com/2021/01/27/szk0PS.md.png)



![16](https://s3.ax1x.com/2021/01/27/szkrvj.md.png)



![17](https://s3.ax1x.com/2021/01/27/szkyKs.md.png)



![18](https://s3.ax1x.com/2021/01/27/szk6rn.md.png)



![19](https://s3.ax1x.com/2021/01/27/szk2V0.md.png)



![20](https://s3.ax1x.com/2021/01/27/szkcbq.md.png)



![21](https://s3.ax1x.com/2021/01/27/szkRaV.md.png)



#### 安装交叉编译工具链

来到[官网](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads)往下拉就能看到2019的版本，点击这个红框，选择这个版本。

![63](https://s3.ax1x.com/2021/01/27/szcCCR.md.png)

然后下载`zip`压缩包。



![64](https://s3.ax1x.com/2021/01/27/szcp59.md.png)

解压后如下图所示。

![62](https://s3.ax1x.com/2021/01/27/sz6zE4.md.png)

此处可以看到我的路径是`E:\gcc-arm-none-eabi-9-2019-q4-major-win32`，添加环境变量，这次也要将路径拓展到`\bin`。

![61](https://s3.ax1x.com/2021/01/27/szcSUJ.png)

![60](https://s3.ax1x.com/2021/01/27/sz62gP.png)

设置之后，打开命令行，输入`arm-none-eabi-gcc -v`，如果看得到版本信息就说明设置成功啦。

![59](https://s3.ax1x.com/2021/01/27/sz6g3t.md.png)



#### 安装OpenOcd

来到[官网](https://gnutoolchains.com/arm-eabi/openocd/ )，如下图所示，选择的是`windows`的版本。点击红框下载压缩包。

![22](https://s3.ax1x.com/2021/01/27/sz0JhV.md.png)

解压之后如下所示。

![23](https://s3.ax1x.com/2021/01/27/sz08kq.md.png)

同样添加环境变量，也要包括到`bin`。我的路径是`E:\openocd-20201228\OpenOCD-20201228-0.10.0\bin`。

![24](https://s3.ax1x.com/2021/01/27/sz017n.png)

设置之后，打开命令行输入`openocd -v`查看是否设置成功。

![25](https://s3.ax1x.com/2021/01/27/sz0l0s.md.png)





#### 安装stlink驱动

来到[官网](https://www.st.com/zh/development-tools/stsw-link009.html#overview)，然后点击获取软件，之后会让你填个邮箱，在邮箱给出的链接下载软件就行了。

![26](https://s3.ax1x.com/2021/01/27/sz0Gt0.md.png)

下载后解压，选择红框的`exe`。

![27](https://s3.ax1x.com/2021/01/27/sz0N1U.md.png)

安装完成后就是这样的。

![28](https://s3.ax1x.com/2021/01/27/sz0tpT.png)



当电脑连接`stlink`时，如果安装成功，就可以在设备管理器看到下图红框的提示。

![image-20210127155105481](https://s3.ax1x.com/2021/01/27/sz0UcF.png)



### stm32cubemx新建工程

新建一个cube的工程

![67](https://s3.ax1x.com/2021/01/29/yi6f58.png)

选择对应的芯片

![68](https://s3.ax1x.com/2021/01/29/yi658g.md.png)

选择外部晶振

![69](https://s3.ax1x.com/2021/01/29/yi6WUf.md.png)

用的是stlink-sw下载的模式

![80](https://s3.ax1x.com/2021/01/29/yi6Hrn.md.png)

配置时钟树

![75](https://s3.ax1x.com/2021/01/29/yi64PS.md.png)

选择`makefile`

![76](https://s3.ax1x.com/2021/01/29/yi6RVP.md.png)

勾上这两个选项

![77](https://s3.ax1x.com/2021/01/29/yi6ovj.md.png)

然后点击上方的`Generate Code`，即可生成需要的外设配置代码

![78](https://s3.ax1x.com/2021/01/29/yi6I2Q.md.png)



打开文件夹如下所示

![79](https://s3.ax1x.com/2021/01/29/yi67Ks.md.png)

这里有个很方便的小技巧，参考这个[链接](https://www.jianshu.com/p/e8c29211fba9)，可以实现右键选择需要用`vscode`打开的文件。



### 编译

#### 配置c_cpp_properties.json

按下`Ctrl+Shift+P`打开命令行，输入`edit configurations(json)`。他会在工程文件夹下生成一个`.vscode`文件夹并在其中创建一个`c_cpp_properties.json`的配置文件。其中`include`路径和宏定义可以参照`makefile`添加。

![31](https://s3.ax1x.com/2021/01/27/szB5MF.md.png)

这是默认的`vscode`给的模板。

![32](https://s3.ax1x.com/2021/01/27/szBWGV.md.png)

接下来根据我们从`cubemx`生成的`makefile`来进行修改。打开`maikefile`，找到个宏定义处，

将这两个宏定义添加到自己的`c_cpp_properties.json`文件。（记得把开头的`D`去掉）

![33](https://s3.ax1x.com/2021/01/27/szBhxU.md.png)

修改后的内容如下所示。

![34](https://s3.ax1x.com/2021/01/27/szBf2T.md.png)

完整的代码。

```json
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE",
                "USE_HAL_DRIVER" ,
                "STM32F103xB"
            ],
            "compilerPath": "C:\\MinGW\\bin\\gcc.exe",
            "cStandard": "gnu11",
            "cppStandard": "c++17",
            "intelliSenseMode": "gcc-x86"
        }
    ],
    "version": 4
}
```



#### 方法（一）

直接手动在中断输入`make`命令，或者`make -j4`等。如果提示下列错误，找到自己安装的`MinGW`路径，来到`bin`	文件夹。将`mingw32-make`修改为`make`。	

![41](https://s3.ax1x.com/2021/01/27/szr2cT.md.png)



![42](https://s3.ax1x.com/2021/01/27/szr4HJ.png)



![43](https://s3.ax1x.com/2021/01/27/szrhB4.md.png)

如下图所示，编译成功。

![44](https://s3.ax1x.com/2021/01/27/szrfuF.md.png)





#### 方法（二）

按下`Ctrl+Shift+P`打开命令行，输入`task`。点击`配置任务`。

![45](https://s3.ax1x.com/2021/01/27/szrRjU.png)

点击`使用模板创建tasks.json`文件。

![46](https://s3.ax1x.com/2021/01/27/szrIE9.png)

点击`Others`，然后就会生成一个默认模板。

![47](https://s3.ax1x.com/2021/01/27/szroNR.png)

默认模板如下：

![48](https://s3.ax1x.com/2021/01/27/szrT41.md.png)

我们配置这个任务是为了编译，所以将`label`修改成`编译代码`。

![49](https://s3.ax1x.com/2021/01/27/szrH9x.png)

原代码如下：

```json
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "shell",
			"label": "编译代码",
			"command": "make",
			"args": [
				"-j4"
			],
			"problemMatcher":[
				"$gcc"
			],
			"group": "build",
		}
	]
}
```

配置完成后，点击`终端`，点击`运行任务`，就会出现我们配置好的`编译代码`的任务，点击之后其执行内容就相当于之前执行的`make`命令。**执行这个任务的时候会再打开一个终端**。

![50](https://s3.ax1x.com/2021/01/27/szrb36.md.png)



![51](https://s3.ax1x.com/2021/01/27/sz60BD.png)



![52](https://s3.ax1x.com/2021/01/27/sz6wnO.md.png)





### 下载

再次配置`task.json`文件。

![55](https://s3.ax1x.com/2021/01/27/sz6ajK.md.png)

最终的`task.json`文件如下所示。

```json
{
    "version": "2.0.0",
    "tasks": [
     {
      "type": "shell",
      "label": "编译代码",
      "command": "make",
      "args": [
        "-j4"
      ],
      "problemMatcher":[
       "$gcc"
      ],
      "group": "build",
     },

     {
        "type": "shell",
        "label": "下载",
        "command": "openocd",
        "args": [
         "-f",
         "E:/openocd-20201228/OpenOCD-20201228-0.10.0/share/openocd/scripts/interface/stlink.cfg",
         "-f",
         "E:/openocd-20201228/OpenOCD-20201228-0.10.0/share/openocd/scripts/target/stm32f1x.cfg",
         "-c",
         "program build/test.elf verify reset exit"
        ],
        "problemMatcher":[
         "$gcc"
        ],
        "group": "build",
       }
    ]
}
```

配置过程也跟上一步配置编译的任务差不多，但是需要注意的就是`openocd`的路径以及生成的`.elf`目标文件的名字。

下面是`openocd`的路径

![53](https://s3.ax1x.com/2021/01/27/sz6rAH.md.png)



![54](https://s3.ax1x.com/2021/01/27/sz6BHe.md.png)

跟上一步一样，打开`终端`，点击`运行任务`，点击`下载`的任务。

![30](https://s3.ax1x.com/2021/01/27/sz0aX4.md.png)



### 调试

这部分需要做的工作是配置`launch.json`文件以及在`task.json`添加任务。

#### launch.json

点击左边的`Debug`图标，如果第一次设置的话，会让你创建一个`launch.json`文件。

![35](https://s3.ax1x.com/2021/01/27/szBIr4.md.png)

选择红框的选项。

![36](https://s3.ax1x.com/2021/01/27/szBoqJ.png)

接下来选择`默认配置`。

![37](https://s3.ax1x.com/2021/01/27/szB7Z9.png)

生成的默认的配置文件如下。然后红框的部分就根据自己实际工程来修改。

![39](https://s3.ax1x.com/2021/01/27/szBbI1.md.png)



修改后的配置文件：红色部分根据自己`makefile`要生成的`target`来修改，`gdb`路径根据自己的电脑路径来设置。绿色部分是因为我们使用的是`openocd`工具而增加的配置文件。**注意增加部分的`file test.elf`。**

![70](https://s3.ax1x.com/2021/01/29/yiwOpR.png)

最后修改完成的`launch.json`文件的内容：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) 启动",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/test.elf",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "arm-none-eabi-gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {"text": "cd ${workspaceFolder}/build"},
                {"text": "file test.elf"},
                {"text": "target remote localhost:3333"},
                {"text": "monitor reset"},
                {"text": "monitor halt"},
                {"text": "load"},
            ]
        }
    ]
}
```



#### task.json

其实只是增加了一个`启动下载调试器`的任务，文件内容如下

```json
{
    "version": "2.0.0",
    "tasks": [
     {
      "type": "shell",
      "label": "编译代码",
      "command": "make",
      "args": [
        "-j4"
      ],
      "problemMatcher":[
       "$gcc"
      ],
      "group": "build",
     },

     {
        "type": "shell",
        "label": "下载",
        "command": "openocd",
        "args": [
         "-f",
         "E:/openocd-20201228/OpenOCD-20201228-0.10.0/share/openocd/scripts/interface/stlink.cfg",
         "-f",
         "E:/openocd-20201228/OpenOCD-20201228-0.10.0/share/openocd/scripts/target/stm32f1x.cfg",
         "-c",
         "program build/test.elf verify reset exit"
        ],
        "problemMatcher":[
         "$gcc"
        ],
        "group": "build",
       },
        
        
        {
			"label": "启动调试器下载",
			"type": "shell",
			"command":"openocd -f interface/stlink.cfg -f target/stm32f1x.cfg"
		}
        
    ]
}
```





#### 启动

打开终端，执行`启动调试器下载`任务。

![71](https://s3.ax1x.com/2021/01/29/yiynYD.md.png)



![72](https://s3.ax1x.com/2021/01/29/yiymFO.md.png)



当任务开启完成后，点击`Debug`图标，点击`(gdb)启动`

![73](https://s3.ax1x.com/2021/01/29/yiyZTK.png)

当出现下面的界面时说明配置成功，可以开始`debug`了

![74](https://s3.ax1x.com/2021/01/29/yiyufe.md.png)



### 注意

- 有时候需要在cube工程重新配置一些外设，生成需要的代码，在进行编译的时候可能会出现这种错误，这是因为cube会在`makefile`文件的包含路径重复增加，所以直接在`makefile`那里直接把多余的路径删除就可以了。

  ![57](https://s3.ax1x.com/2021/01/27/sz6y4A.png)

![56](https://s3.ax1x.com/2021/01/27/sz6sNd.png)

- 如果要增加自己的一些源文件，记得在`makefile`的`C_SOURCES`增加必要的说明。

  ![65](https://s3.ax1x.com/2021/01/28/y9HvWt.png)

- 在下载交叉编译工具链的时候可能因为下载的是新版本，所以导致了某些路径的冲突，导致环境一直安装不成功。

  

### 总结跟改进

- 用`vscode`来开发的话，比起`MDK`来说还是没有那么方便，需要添加一些配置，不过好处就是编译和下载的速度会稍微快点。但是`debug`功能还是没有`MDK`的香。

- 写新工程重新添加配置的时候，可能会因为生成的目标文件名字不同，所以有时候需要注意修改，后边如果有空可以研究写个脚本，直接把`makefile`要生成的`target`用一个统一的变量表示，以后就不用写新工程的时候去注意修改目标文件的名字。

- 在`windows`环境下，`vscode`有一个非常方便的插件`iotlink`，[使用教程](https://flyfishzy.github.io/iotstudio-doc/zh/)也十分简单。这个插件直接帮你把编译下载调试等功能设置好了，也会帮你把需要的工具链等等自动下载。不过这只限于`windows`。

- 有空也想在`linux`环境下配置一下。

  

### 参考链接

- [安装vscode参考链接](https://www.jianshu.com/p/51dfbe2c9583)

- [安装MinGW参考链接（一）](https://www.jianshu.com/p/d66c2f2e3537)

- [安装MinGW参考链接（二）](https://www.jianshu.com/p/e9ff7b654c4a)

- [vscode配置参考链接](https://blog.csdn.net/u013671216/article/details/106170433?utm_medium=distribute.pc_relevant.none-task-blog-searchFromBaidu-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-searchFromBaidu-2.control)

- [交叉编译工具链配置参考链接](https://blog.csdn.net/u013671216/article/details/106170433?utm_medium=distribute.pc_relevant.none-task-blog-searchFromBaidu-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-searchFromBaidu-2.control)

  



### 模板工程

[Github链接](https://github.com/copyright1999/stm32_vscode_template)