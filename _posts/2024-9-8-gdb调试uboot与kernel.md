---
layout:     post   				    
title:      gdb调试uboot与kernel			
subtitle:  
date:       2024-09-08				
author:     婷                               
header-img: img/131.png 	
catalog: true 						
tags:								

- gdb
- jtag
- 调试
- uboot
- linux

---





## 简介

前提是搭建好环境，可以参考之前的[链接](https://copyright1999.github.io/2024/07/27/%E6%A0%91%E8%8E%93%E6%B4%BE%E5%BC%80%E5%8F%91-%E4%BA%8C/)



## uboot

在`uboot`目录下输入`gdb-multiarch`，而不是`gdb-multiarch u-boot`，因为`uboot`会重定位，而且之前这么操作总是有奇奇怪怪的问题

```bash
gdb-multiarch
```



![image-20240903082818053](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240903082818053.png)



连接`openocd`内置的`gdbserver`，端口号是`3333`

```bash
target retmoe :3333
```

![image-20240903083037836](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240903083037836.png)



然后`uboot`下输入`bdinfo`命令，找到重定位后的地址

![image-20240804212309311](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240804212309311.png)



找到地址，输入下列命令，加载符号表，输入`bt`查看函数调用栈，这里连函数的第几行都显示出来了

```bash
add-symbol-file ~/raspi4b-project/uboot/u-boot 0x7f21000
```



![image-20240903083256869](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240903083256869.png)



当然还有另外的方法，不用输入`bdinfo`也可以知道重定位后的地址，因为`bd_info`是在结构体`gd_t`中的，而`gd_t`的地址存放在`x18`寄存器中，所以我们偏移一下就可以算出来了

![image-20240928164616799](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240928164616799.png)

跟`bdinfo`命令中的信息对上了

![image-20240928164733715](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240928164733715.png)





## kernel

如果想要`gdb`调试`kernel`，需要打开如下内核配置

```bash
CONFIG_DEBUG_INFO=y
CONFIG_DEBUG_INFO_DWARF_TOOLCHAIN_DEFAULT=y
CONFIG_FRAME_POINTER=y

CONFIG_GDB_SCRIPTS=y
CONFIG_DEBUG_INFO_REDUCED=n

CONFIG_KGDB=y
CONFIG_RANDOMIZE_BASE=n /*非常重要*/

CONFIG_DEBUG_VM=y
```



以下为内核配置的说明

- `CONFIG_DEBUG_INFO`： 在内核和内核模块中包含调试信息，这个选项在幕后为gcc使用的编译器参数增加了`-g`选项。
- `CONFIG_FRAME_POINTER`：这个选项会将调用帧信息保存在寄存器或堆栈上的不同位置，使`gdb`在调试内核时可以更准确地构造堆栈回溯跟踪（stack back traces）。
- 启用`CONFIG_GDB_SCRIPTS`，但要关闭`CONFIG_DEBUG_INFO_REDUCED`。
- `CONFIG_KGDB`：启用内置的内核调试器，该调试器允许进行远程调试。
- 关闭`CONFIG_RANDOMIZE_BASE`设置：`KASLR`会更改引导时放置内核代码的基地址。如果你在内核配置中启用了`KASLR`（`CONFIG_RANDOMIZE_BASE=y`），则无法从`gdb`设置断点。
- `CONFIG_DEBUG_VM`



配置编译好`kernel`后，输入下列命令

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- scripts_gdb -j16
```

![image-20240906232332681](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240906232332681.png)



执行`gdb`的时候，可能会有类似的报错

```bash
warning: File "/home/sammi/raspi4b-project/raspi_sdk/raspi_linux/scripts/gdb/vmlinux-gdb.py" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
To enable execution of this file add
        add-auto-load-safe-path /home/sammi/raspi4b-project/raspi_sdk/raspi_linux/scripts/gdb/vmlinux-gdb.py
line to your configuration file "/home/sammi/.gdbinit".
```



![image-20240907163910308](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907163910308.png)



那就按照提示语句`add-auto-load-safe-path /home/sammi/raspi4b-project/raspi_sdk/raspi_linux/scripts/gdb/vmlinux-gdb.py`加到文件`/home/sammi/.gdbinit`

![image-20240907163832825](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907163832825.png)





### 未使能MMU的kernel

`kernel`前期在汇编中的代码是没有开启MMU的，而`vmlinux`调试文件中都是虚拟地址，所以我们需要将`.head.text，text ，rodata`这三个段重新映射（为什么是这三个段等以后研究下kernel启动的汇编脚本）



输入下列命令获取三个段的虚拟地址，然后用kernel启动的基地址加上这三个段减去.head.text的地址的偏移量就是我们要重定位的地址了

```bash
readelf -S vmlinux
```



这里可以看出`.head.text`是最开始的第一个段，为什么`.head.text`是第一段呢？

![image-20240904232731909](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240904232731909.png)

其实是由文件`arch/arm64/kernel/vmlinux.lds.S `布局决定的

![image-20240904232617155](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240904232617155.png)



通过虚拟地址跟kernel在ddr中的地址相减获取真实的物理地址，树莓派中kernel的启动地址是0x1000000，如下计算可以得到真实段的地址

```bash
.head.text = 0x1000000 + (ffffffc080000000 - ffffffc080000000) 
.text = 0x1000000 + (ffffffc080010000 - ffffffc080000000)
.rodata =  0x1000000 + (ffffffc080d30000 - ffffffc080000000) 
```



接着我们输入如下命令

```bash
gdb-multiarch vmlinux
target remote :3333
add-symbol-file vmlinux  -s .head.text 0x1000000 -s .text 0x1010000 -s .rodata 0x1d30000
```



![image-20240904085645409](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240904085645409.png)



输入`layout split`即可快乐调试

![image-20240904085726578](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240904085726578.png)





### 使能MMU的kernel

直接输入命令

```bash
gdb-mulitiarch vmlinux
target remote :3333
```



![image-20240907165342192](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907165342192.png)





### 调试ko

调试`ko`的话需要知道`text`段的地址，有两种方法可以查看

```bash
cat /proc/modules
cat /sys/module/general/sections/.text
```



![image-20240907215450410](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907215450410.png)

加载命令如下，注意这里用的是`.o`文件

```bash
add-symbol-file modules/general/general.o 0xffffffc0799a0000
```



![image-20240907215310267](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907215310267.png)

打断点如下

![image-20240907215526792](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907215526792.png)



如果需要调试静态符号，需要加载`.bss`段

![image-20240908135344077](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240908135344077.png)



![image-20240908135500907](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240908135500907.png)

```bash
add-symbol-file modules/general/general.o 0xffffffc0799a0000 -s .bss 0xffffffc0799a2640
```



## 调试技巧

### 调用栈

这个比较常见的用法，通常想知道函数跑到什么位置

```
bt
```



### 断点

具体到某个文件的某一行

![image-20240901204804330](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240901204804330.png)



其他用法

```bash
(gdb) b main         #到某个函数
(gdb) b *0x80483f    #在某个内存
```



### 查看断点

![image-20240908131103890](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240908131103890.png)



### 删除断点

```bash
delete breakpoints N
```



![image-20240908131707819](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240908131707819.png)



### 单指令调试

输入`layout split`命令可以列出`src`跟`asm`两个框

![image-20240903084009373](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240903084009373.png)



![image-20240903084156019](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240903084156019.png)



可以用`step（si）`跳入调用函数并执行，`next（ni）`不跳入调用函数并执行

```bash
si 3 #执行三行
ni 3 #执行三行
```



退出图形界面可以按`Ctrl+X `然后再按`A`





## kernel专属

`GDB`提供了Python接口来扩展功能，内核基于Python接口实现了一系列辅助脚本，简化内核调试，开启`CONFIG_GDB_SCRIPTS`参数就可以使用了。内核中的文档信息如下

```bash
Documentation/translations/zh_CN/dev-tools/gdb-kernel-debugging.rst
Documentation/dev-tools/gdb-kernel-debugging.rst
```



输入`apropos lx`获取帮助信息

![image-20240907220224497](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907220224497.png)



### dmesg

```bash
lx-dmesg
```



![image-20240907221422203](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907221422203.png)



### virt2phy和virt2page

```bash
lx-virt_to_phys
lx-virt_to_page
```



![image-20240907220533928](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907220533928.png)



### mount

```bash
lx-mount
```

就是不知道这些虚拟地址指的是哪个地方的，等以后研究文件系统的时候可以回头看看

![image-20240907013245837](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907013245837.png)



### fdtdump 

```bash
lx-fdtdump
```



![image-20240907221351169](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240907221351169.png)

最后能找到当前目录下生成的`fdtdump.dtb`

![image-20240908140212889](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240908140212889.png)

至于原理怎么实现还不了解





### clk-tree

```
lx-clk
lx-clk-summary
```



![image-20240908125942848](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240908125942848.png)



### cmline

```bash
lx-cmdline
```



![image-20240908130249332](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240908130249332.png)



### 其他

之前自己写的[博客](https://copyright1999.github.io/2023/08/26/gdb%E4%BD%BF%E7%94%A8%E5%B0%8F%E7%BB%93/)



## 参考链接

- [参考链接一](https://github.com/mz1999/blog/blob/master/docs/gdb-kernel-debugging.md)
- [参考链接二](https://blog.csdn.net/www_dong/article/details/117374370)
- [参考链接三](https://github.com/tinyclub/elinux/blob/master/zh/dbg_portal/kernel_dbg/Debugging_The_Linux_Kernel_Using_Gdb/Debugging_The_Linux_Kernel_Using_Gdb.md)
- [参考链接四](https://github.com/tinyclub/elinux/blob/master/zh/dbg_portal/kernel_dbg/Debugging_The_Linux_Kernel_Using_Gdb/Debugging_The_Linux_Kernel_Using_Gdb.md#debugging-a-kernel-module-o-and-ko)
- [参考链接五](https://breezetemple.github.io/2017/08/08/linux-modules-debug/)





## 问题

目前调试使能MMU后使用虚拟地址的内核，无法打断点调试，会提示下列报错，还不清楚是什么问题

![image-20240908140543280](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gdb_uboot_kernel/image-20240908140543280.png)
