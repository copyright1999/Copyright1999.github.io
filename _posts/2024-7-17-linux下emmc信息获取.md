---
layout:     post   				    
title:      linux下emmc信息获取		
subtitle:  
date:       2024-07-17				
author:     婷                              
header-img: img/124.png 	
catalog: true 						
tags:								

- emmc
- 存储

---





## 简介

主要介绍下`linux`下`emmc`（延伸到块设备）信息的获取





## /proc/partition接口

这是我们的板子显示的内容，这里的`block`的单位是`KB`，一开始我以为是`512byte`

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150030576-16.png)





对比这里的大小信息可以跟上面的信息对应

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150009019-13.png)





如果是`cmdline`传参，给分区设置名字的话，还会显示

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150439105-1.png)



## mount查看分区挂载情况

执行`mount`命令，可以查看分区挂载的情况。

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150473811-4.png)







## fdisk -l

输入`fdisk -l`这个命令

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150511709-7.png)



可以看到能显示具体的`type`

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150524505-10.png)



## lsblk

`lsblk`命令的输出可以看到`boot` ,`rpmb`, `uda`分区都是 `type disk`的，而`uda`里面各自的分区显示的是`part`，这个其实跟内核块设备驱动中的`mmc_add_disk`跟`mmc_blk_alloc_part`相关，后面会分析到的



![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150537891-13.png)







## /sys/class/block 查看eMMC分区情况

在 `/sys/class/block/` 目录下可以查看当前系统的分区情况：

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150550034-16.png)



```Bash
root@copyright:/sys/class/block# ls
loop0  loop4  mmcblk1       mmcblk1p2    ram10  ram14  ram4  ram8
loop1  loop5  mmcblk1boot0  mmcblk1rpmb  ram11  ram15  ram5  ram9
loop2  loop6  mmcblk1boot1  ram0         ram12  ram2   ram6
loop3  loop7  mmcblk1p1     ram1         ram13  ram3   ram7
root@copyright:/sys/class/block# ls -l mmc*
lrwxrwxrwx 1 root root 0 Jan  1  1970 mmcblk1 -> ../../devices/platform/soc/2100000.aips-bus/2194000.usdhc/mmc_host/mmc1/mmc1:0001/block/mmcblk1
lrwxrwxrwx 1 root root 0 Jan  1  1970 mmcblk1boot0 -> ../../devices/platform/soc/2100000.aips-bus/2194000.usdhc/mmc_host/mmc1/mmc1:0001/block/mmcblk1/mmcblk1boot0
lrwxrwxrwx 1 root root 0 Jan  1  1970 mmcblk1boot1 -> ../../devices/platform/soc/2100000.aips-bus/2194000.usdhc/mmc_host/mmc1/mmc1:0001/block/mmcblk1/mmcblk1boot1
lrwxrwxrwx 1 root root 0 Jan  1  1970 mmcblk1p1 -> ../../devices/platform/soc/2100000.aips-bus/2194000.usdhc/mmc_host/mmc1/mmc1:0001/block/mmcblk1/mmcblk1p1
lrwxrwxrwx 1 root root 0 Jan  1  1970 mmcblk1p2 -> ../../devices/platform/soc/2100000.aips-bus/2194000.usdhc/mmc_host/mmc1/mmc1:0001/block/mmcblk1/mmcblk1p2
lrwxrwxrwx 1 root root 0 Jan  1  1970 mmcblk1rpmb -> ../../devices/platform/soc/2100000.aips-bus/2194000.usdhc/mmc_host/mmc1/mmc1:0001/block/mmcblk1/mmcblk1rpmb
root@copyright:/sys/class/block#
```

从上面的信息可以知道：

- 这个`eMMC`上总共共有`5`个分区，`boot0/boot1/rpmb/p1/p2`分区；
- 这个`eMMC`用到的控制器为`2194000.usdhc`；
- 我们可以到软连接显示的`../../devices/platform/soc/2100000.aips-bus/2194000.usdhc/mmc_host/mmc1/mmc1:0001/block/mmcblk1/`目录下查看更具体的信息；



![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150596593-19-1721150616613-22.png)



目录下还有我们`boot`，`rpmb`，`uda`分区的信息

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150619213-24.png)



关于这些参数的意义，可以参考如下两个链接

```bash
https://www.kernel.org/doc/Documentation/ABI/testing/sysfs-block
https://www.kernel.org/doc/Documentation/block/stat.txt
```



这里简单的现实下几个特性，这里的`mmc1:0001`表示`mmc1`控制器，`rca`地址为`0001`的`emmc`卡

下面看了`hwrev`，`name`，`oemid`，`raw_rpmb_size_mult`，`csd`，`cid`，`erase_size`，`enhanced_area_size`，`enhanced_area_offset`，`fwrev`，`type`，`date`

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150621692-27.png)



## 查看各个分区的大小

两种方法，一种是`fdisk/lsblk`，一种是`sysfs`

### fdisk / lsblk

`fdisk`显示了`start`,`end`,`sector`

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150644156-33.png)



![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150641597-30.png)



### sysfs

另一种是`sysfs`的方法

#### uda分区

在`/sys/devices/platform/soc/2100000.aips-bus/2194000.usdhc/mmc_host/mmc1/mmc1:0001/block`目录下，有几个以`mmcblk1`开头的目录名，这里面包含了分区的起始和`block`的块数


![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150662075-36.png)

- 以`mmcblk1p1`为例，这里的`start`单位是`block`，也就是第`2048`块，size是`65536`，也就是`p1`分区大小为`65536*512byte` = `32MB`
- 以`mmcblk1p2`为例，这里的`start`单位是`block`，也就是第`67584`块，`size`是`15202304`，也就是`p2`分区大小为`15202304*512byte `= `7423MB`



跟前面`fdsik /lsblk `显示的信息是一样的，且这里可以看到`p1`开始前面有部分区域是空的，放了个分区表（注意分区管理只针对uda分区）



像如果是`gpt`分区的话，`fdisk`中的`disklabel type`会显示`gpt`

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150669525-39.png)





#### boot分区

没有`start`，只有`size`

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150676795-42.png)



#### rpmb分区

一样

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150683197-45.png)





## 读取emmc控制器

首先执行：`mount -t debugfs none /sys/kernel/debug/`， 在`/sys/kernel/debug/mmc1`目录下，`cat ios`可以查看这个`eMMC`控制器的工作频率和位宽基本信息。

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150693658-48.png)



像这里的`mmc0`是控制器的信息，而不是外接的`emmc`卡信息哦，`emmc`卡信息是`mmc0:0001`这种带地址的，这里也是有个`host`，`device`的思想

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150700147-51.png)







## 读取ext_csd寄存器

这个寄存器属于`csd`扩展出来的，需要将`debugfs`挂载出来才能读取到。注意这里读的是`emmc`卡的信息，相当于`device`了。

在`/sys/kernel/debug/mmc1/mmc1\:0001/`目录下就可以看到`ext_csd`寄存器了。`cat ext_csd`可以得到一个`512`字节大小的数据，如下

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150708378-54.png)


当然这样看很难看，无法很快的确认寄存器中某些位的值。因此，网上就有个小哥写了个`python`脚本来解析这`512`字节的数据，以常人可以理解的格式进行解析。详见此[链接](https://blog.kylemanna.com/linux/parse-emmc-extended-csd-ecsd-registers-with-python/)


脚本这里就先贴出来

```Bash
#!/usr/bin/env python
"""
Author: Kyle Manna <kyle@kylemanna.com>
Blog: https://blog.kylemanna.com
"""
import binascii
import re
import sys

def str2bytearray(s):
    if len(s) % 2:
        s = '0' + s

    reorder = True
    if reorder:
        r = []
        i = 1
        while i <= len(s):
            r.append(s[len(s) - i - 1])
            r.append(s[len(s) - i])
            i += 2
        s = ''.join(r)

    out = binascii.unhexlify(s)

    return out


if __name__ == '__main__':

    ecsd_str = '00000000000000000000000000000000390300c0470700c04707000000000000000101000000000000000000000000000000000000000000000000000a000001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000900e00070100000000151f20000000000000000000000000000000010100090000000008000200571f0a0aeeee8888001e0f460f78140100c0470710140a0a090201320808400007fdfb550100640aeeeeee99011e0200000000320a00100000ee01000000000000000000012020010100000000000000000000000000000000000000000000000000000000000000000000000000001f0100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001ffffffff00000103007f0003013f3f01010100000000000000'
    #ecsd_str = '320100'
    ecsd = str2bytearray(ecsd_str)
    csd_len = len(ecsd)

    line_len = 16
    i = 0
    while i < len(ecsd):
        sys.stdout.write("{0:04d}:\t".format(i))
        for j in range(line_len):
            if (i < csd_len):
                sys.stdout.write("{0:=02x}".format(ord(ecsd[csd_len-i-1])))
                i = i + 1
            else:
                break

            if (j == (line_len - 1)): pass
            elif (i % 4): sys.stdout.write(" ")
            else: sys.stdout.write("   ")

        sys.stdout.write("\n")

    print "BOOT_SIZE_MULTI[226] = 0x{:x}".format(ord(ecsd[csd_len-168-1]))
```



复制然后放到我们板子执行

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150723843-57.png)

这里解析了很多信息出来，结合`spec`可以更仔细的看

```Bash
copyright:/sys/kernel/debug/mmc1/mmc1:0001#
copyright:/sys/kernel/debug/mmc1/mmc1:0001# vi ~/parse_emmc.py
copyright:/sys/kernel/debug/mmc1/mmc1:0001# chmod +x ~/parse_emmc.py
copyright:/sys/kernel/debug/mmc1/mmc1:0001# python ~/parse_emmc.py
0000:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0016:   39 03 00 c0   47 07 00 c0   47 07 00 00   00 00 00 00
0032:   00 01 01 00   00 00 00 00   00 00 00 00   00 00 00 00
0048:   00 00 00 00   00 00 00 00   00 00 00 00   0a 00 00 01
0064:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0080:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0096:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0112:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0128:   00 00 01 00   00 00 00 00   00 00 00 00   00 00 00 00
0144:   00 00 00 00   00 00 00 00   00 00 00 00   00 90 0e 00
0160:   07 01 00 00   00 00 15 1f   20 00 00 00   00 00 00 00
0176:   00 00 00 00   00 00 00 00   01 01 00 09   00 00 00 00
0192:   08 00 02 00   57 1f 0a 0a   ee ee 88 88   00 1e 0f 46
0208:   0f 78 14 01   00 c0 47 07   10 14 0a 0a   09 02 01 32
0224:   08 08 40 00   07 fd fb 55   01 00 64 0a   ee ee ee 99
0240:   01 1e 02 00   00 00 00 32   0a 00 10 00   00 ee 01 00
0256:   00 00 00 00   00 00 00 00   01 20 20 01   01 00 00 00
0272:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0288:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0304:   00 00 00 1f   01 00 00 00   00 00 00 00   00 00 00 00
0320:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0336:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0352:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0368:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0384:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0400:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0416:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0432:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0448:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0464:   00 00 00 00   00 00 00 00   00 00 00 00   00 00 00 00
0480:   00 00 00 00   00 00 01 ff   ff ff ff 00   00 01 03 00
0496:   7f 00 03 01   3f 3f 01 01   01 00 00 00   00 00 00 00
BOOT_SIZE_MULTI[226] = 0x20
copyright:/sys/kernel/debug/mmc1/mmc1:0001#
```



![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150732647-60.png)



## mmc工具

### 获取extcsd的信息

```Bash
mmc extcsd read /dev/mmcblk0
```

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150782123-72.png)


![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150775069-69.png)



### 检查enhance的最大大小

```Bash
mmc extcsd read /dev/mmcblk0 | grep MAX_ENH_SIZE_MULT -A 1
```


![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150760189-66.png)





## MiB跟MB的区别

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_emmc/1721150750151-63.png)



## 参考链接

- [参考链接一](https://wowothink.com/33268c93/)

- [参考链接二](https://blog.csdn.net/starshine/article/details/8226320)

- [python脚本](https://blog.kylemanna.com/linux/parse-emmc-extended-csd-ecsd-registers-with-python/)



