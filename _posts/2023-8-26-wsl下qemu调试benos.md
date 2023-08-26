---
layout:     post   				   
title:      wsl下qemu调试benos			
subtitle:  
date:       2023-08-26				
author:     婷                              
header-img: img/89.png 
catalog: true 						
tags:								

- wsl
- qemu
- gdb

---



## 简介

主要记录下`gdb `+` qemu`调试`benos`的过程，以及对`benos`基础实验代码的一些小小解析。



## 准备

### 交叉编译工具链



### gdb安装

```bash
sudo apt install gdb-multiarch
```

![image-20230730200810934](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230730200810934.png)





### benos代码编译

代码仓库

```
https://github.com/runninglinuxkernel/arm64_programming_practice/tree/main/chapter_2/lab01_hello_benos/BenOS
```

下载后代码结构如下

![image-20230723104832952](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230723104832952.png)



查看`Makefile`，可知默认的工具链跟板子类型

![image-20230814225319828](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230814225319828.png)



我们模拟`raspi4b`，输入下列命令，编译

```text
 export board=rpi4
 make
```



最后可以看到编译得到的`elf`文件跟`bin`文件

![image-20230723104913632](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230723104913632.png)





## 运行

输入这条命令，就可以运行，看到程序的输出了

```bash
 ./qemu-system-aarch64 -machine raspi4b2g -nographic -kernel ~/benos/benos.bin 
```



![image-20230820183304877](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230820183304877.png)



## 调试

调试命令如下，就是在前面的运行命令加上了`-S -s`

```bash
 ./qemu-system-aarch64 -machine raspi4b2g -nographic -kernel ~/benos/benos.bin -S -s
```



这个时候就会停在这个界面

![image-20230820183625160](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230820183625160.png)



这个时候再开个新的终端

```bash
gdb-multiarch --tui benos/build/benos.elf
```



![image-20230731222005123](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230731222005123.png)

接着进入这个界面

![image-20230820183743816](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230820183743816.png)



敲下回车键

![image-20230820183844849](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230820183844849.png)



接着跟`qemu`的`gdbserver`进行连接，输入`target remote localhost:1234`

![image-20230820184106252](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230820184106252.png)



然后，我们使用`b`命令设置断点，比如`b _start`就是在`_start`处设置端点，再执行`c`，就可以使程序运行到第一个端点处，并在上面的窗口显示代码，如下图所示：

![image-20230810235953042](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230810235953042.png)



![image-20230811000014242](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230811000014242.png)



接着可以使用`s`命令进行单步调试，这里可以看到已经走完了打印`We`的流程了

![image-20230820184541080](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230820184541080.png)



查看另一边开着`qemu`的终端，可以看到`We`的输出

![image-20230820185356853](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230820185356853.png)





## 代码解析

### 编译过程

先从`bin`跟`elf`的怎么生成说起，通过Makefile可知elf文件是通过链接文件`linker.ld`，集合了多个`.o`之后，生成的。而`bin`文件则是`strip`掉一些调试后生成的直接在**裸机上**运行的二进制文件。前面用`qemu`模拟的时候也是用`bin`文件运行，`elf`文件来调试。

![image-20230823232604377](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_benos/image-20230823232604377.png)



生成`elf`文件的命令，`-T`指定链接文件`linker.ld`，`-o`指定输出的产物，`-e`指定程序的入口

```bash
aarch64-linux-gnu-ld -T src/linker.ld -o build/benos.elf  build/pl_uart_c.o build/kernel_c.o build/mm_s.o build/boot_s.o -e _start
```



生成`bin`文件的命令，利用`objcopy`去掉`elf`文件中的调试信息，最终生成可直接在**裸机上**运行的二进制文件（注意是裸机，像`linux`操作系统有加载器，可以直接运行`elf`文件）

```
aarch64-linux-gnu-objcopy build/benos.elf -O binary benos.bin
```



在生成`.o`目标文件的时候，也有几个编译选项

```bash
aarch64-linux-gnu-gcc -DCONFIG_BOARD_PI4B -g -Wall -nostdlib -nostdinc -Iinclude -MMD -c src/pl_uart.c -o build/pl_uart_c.o
aarch64-linux-gnu-gcc -DCONFIG_BOARD_PI4B -g -Wall -nostdlib -nostdinc -Iinclude -MMD -c src/kernel.c -o build/kernel_c.o
aarch64-linux-gnu-gcc -g -Iinclude  -MMD -c src/mm.S -o build/mm_s.o
aarch64-linux-gnu-gcc -g -Iinclude  -MMD -c src/boot.S -o build/boot_s.o
```



- `-g`：表示编译的时候加入调试符号表等信息
- `-Wall`：表示打开所有的警告信息（也就是**warning all**）
- `-nostdlib`：表示不连接系统的标准启动文件和标准库文件，只把指定的文件传递给连接器，这个选项用于编译内核，`bootloader`等程序，它们不需要标准启动文件和标准库文件
- `-nostdinc`：表示不包含C语言的标准库的头文件



### 链接文件

前面提到生成`elf`文件的命令，需要`-T`指定链接文件`linker.ld`，本次实验用的`linker.ld`文件如下

```bash
SECTIONS
{
	. = 0x80000,
	.text.boot : { *(.text.boot) }
	.text : { *(.text) }
	.rodata : { *(.rodata) }
	.data : { *(.data) }
	. = ALIGN(0x8);
	bss_begin = .;
	.bss : { *(.bss*) } 
	bss_end = .;
}
```



- `SECTIONS`：是`LS (Linker Script)`语法中的关键命令，描述输出文件的内存布局

- `. = 0x80000`：程序加载在`DDR`中的`0x80000`处
- `.text.boot`：所有的`.o`文件的`.text.boot`部分组成，启动首先要执行的代码（从下面的分析，就是`-e`指定的`_start`）
- `.text`：所有的`.o`文件的`.text`部分组成，代码段
- `.rodata`：所有的`.o`文件的`.rodata`部分组成，只读数据段

- `.data`：所有的`.o`文件的`.data`部分组成，数据段
- `.bss`：所有的`.o`文件的`.bss`部分组成，包含没有初始化的全局变量跟静态变量





## 其他

如果`qemu`不支持`raspi4g`，可以参考之前我的[链接](https://copyright1999.github.io/2023/08/08/wsl%E7%9A%84qemu%E6%94%AF%E6%8C%81raspi4/)编译一个出来支持`raspi4g`的，或者是用`-machine virt`也可以用来模拟。

```bash
qemu-system-aarch64 -machine virt -serial null -serial mon:stdio -nographic -kernel benos/build/benos.bin -S -s
```



