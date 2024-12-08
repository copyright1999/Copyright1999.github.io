---
layout:     post   				    
title:      树莓派引导FIT格式的压缩kernel			
subtitle:  
date:       2024-12-8				
author:     婷                               
header-img: img/138.png 	
catalog: true 						
tags:								

- arm64
- fit
- gzip
- raspi4b
- 压缩

---





## 简介

设置树莓派引导FIT镜像格式的`linux`，顺带验证`FIT`打包压缩的`arm64`的`linux`镜像





## uboot配置

`uboot`需要打开`CONFIG_FIT`配置，然后剩下的其他的`FIT`配置会帮你自动勾上，就不操心了





## kernel镜像打包

`arm64 kernel`默认只支持`gzip`压缩算法，产物有`Image`跟`Image.gz`

![image-20241208155127957](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208155127957.png)



要打包`FIT`的镜像，我们需要由一个`its`文件来描述

如果我们打包的是没有压缩的镜像，则`its`文件如下

```bash
/dts-v1/;

/ {
    description = "Raspi4b FIT Image";
    #address-cells = <1>;

    images {
        kernel {
            description = "Kernel";
            data = /incbin/("Image");
            type = "kernel";
            arch = "arm64";
            os = "linux";
            compression = "none";
            load = <0x1000000>;
            entry = <0x1000000>;
        };
        fdt@0 {
            description = "BCM2711-rpi-4b DTB";
            data = /incbin/("bcm2711-rpi-4-b.dtb");
            type = "flat_dt";
            arch = "arm64";
            compression = "none";
            load = <0x5000000>;
            entry = <0x5000000>;
        };
    };

    configurations {
        default = "config@0";
        config@0 {
            description = "kernel no compression";
            kernel = "kernel";
            fdt = "fdt@0";
        };

    };
};
```



如果是打包的`gzip`压缩过的镜像，则`its`文件如下

```bash
/dts-v1/;

/ {
    description = "Raspi4b gzip FIT Image";
    #address-cells = <1>;

    images {
        kernel {
            description = "Kernel";
            data = /incbin/("Image.gz");
            type = "kernel";
            arch = "arm64";
            os = "linux";
            compression = "gzip";
            load = <0x1000000>;
            entry = <0x1000000>;
        };
        fdt@0 {
            description = "BCM2711-rpi-4b DTB";
            data = /incbin/("bcm2711-rpi-4-b.dtb");
            type = "flat_dt";
            arch = "arm64";
            compression = "none";
            load = <0x5000000>;
            entry = <0x5000000>;
        };
    };

    configurations {
        default = "config@0";
        config@0 {
            description = "kernel gzip";
            kernel = "kernel";
            fdt = "fdt@0";
        };

    };
};
```

压缩的重点在于此处

![image-20241208155027790](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208155027790.png)





有了`its`文件后，通过`mkimage`工具可以生成相关的FIT格式的镜像

```bash
mkimage -f kernel-gz.its linux-gz.itb
mkimage -f kernel.its linux.itb
```







## 启动

启动`gzip`压缩的`FIT`镜像，`uboot`设置如下，对于`FIT`镜像格式，`bootm addr`如此便可，代码中会解析到是`FIT`镜像的格式，自行提取跟解压

```bash
U-Boot> setenv itbgz_bootcmd 'tftp 0x4000000 linux-gz.itb ; bootm 0x4000000 '
U-Boot> saveenv
Saving Environment to FAT... OK
U-Boot> run itbgz_bootcmd

```



![image-20241208154225759](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208154225759.png)



如果引导没有压缩的内核，则设置如下

```bash
U-Boot> setenv itb_bootcmd 'tftp 0xffff48 linux.itb ; bootm 0xffff48'
U-Boot> saveenv
Saving Environment to FAT... OK
U-Boot> run itb_bootcmd

```



![image-20241208163347419](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208163347419.png)







## 多镜像打包

其实有的芯片引导流程有spl -> atf -> uboot这个过程，spl也可以引导FIT格式的uboot+atf镜像，如下面的its文件所示

```bash
/dts-v1/;
/ {
    description = "Configuration to load ATF before U-Boot";
    
    images {
        uboot@1 {
            description = "U-Boot (64-bit)";
            data = /incbin/("./u-boot-nodtb.bin");
            type = "standalone";
            arch = "arm64";
            os = "U-Boot";
            compression = "none";
            load = <0x48000000>;
            entry = <0x48000000>;
        };
        atf@1 {
            description = "ARM Trusted Firmware";
            data = /incbin/("bl31.bin");
            type = "firmware";
            arch = "arm64";
            os = "arm-trusted-firmware";
            compression = "none";
            load = <0x1000000>;
            entry = <0x1000000>;
        };
        fdt {
            description = "U-Boot dtb";
            data = /incbin/("./u-boot.dtb");
            type = "flat_dt";
            arch = "arm64";
            compression = "none";
        };
    };
    configurations {
        default = "config@1";
        config@1 {
            description = "atf uboot fdt image";
            firmware = "atf@1";
            loadables = "uboot@1";
            fdt = "fdt";
        };
    };
};
```









## 如何确定load的地址

在调试的时候碰到如下问题

```bash
ERROR: new format image overwritten - must RESET the board to recover
```



![image-20241208163035621](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208163035621.png)



最后将`bootm.c`中相关的地址打印出来，如上图的打印，发现当我们将`FIT`下载到`0x1000000`地址的时候，会发现实际的`FIT`的头占了`0xb8`的大小，然后灵光一闪，将`load`的内存地址改为`0x1000000 - 0xb8`后即可。

![image-20241208195838364](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208195838364.png)



从图上的出错日志可以看出这里的`start`，`end`，`load`，`load_end`是对应不上的，所以出错了？那其实是不是只要保证这个`load`的地方能保留好`FIT`的头即可了？

![image-20241208200955729](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208200955729.png)



但是这里留着超过的了，却不行

![image-20241208201238246](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208201238246.png)





最后证明确实就是跟`FIT`的开头占了多少有关系。。。。。而且这个开头多大，还跟你`its`文件的字符有关系呢

![image-20241208201857291](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208201857291.png)

![image-20241208201927364](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208201927364.png)



那为啥压缩的就可以不遵循这个`bootm addr ,addr = load_addr - FIT_header`呢？我们压缩后的镜像是大概**14M多**，而设备树地址是在0x5000000，**刚好距离我们的load地址0x4000000的大小为16M**。我们从0x40000bc开始解压，解压后的数据就放到0x1000000，然后解压后的数据末尾为0x3238a00，刚好没有踩地址0x4000000。所以这个0x4000000这个地址上下都非常的极限。

反正对于压缩的镜像，`bootm addr`中的`addr`，保证要两点

- `addr`放压缩镜像的内存，不踩`FDT`
- `linux`解压后的地址，比如图中的`load end  = 0x3238a00`，不要踩到`addr`

![image-20241208202120825](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208202120825.png)

按照这个约束，其实`bootm 0x3800000`也是可以的

![image-20241208210128272](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208210128272.png)





那能不能就是压缩的镜像跟不压缩的镜像那样，也开头留好`FIT`的头部大小呢？那不行呀，`image_buf`对于压缩的镜像来说就是`src`地址，怎么还能原地解压不是？

![image-20241208203257222](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208203257222.png)

![image-20241208203207747](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208203207747.png)





另外，目前树莓派支持镜像解压能申请到的的内存大小为`0x4000000`，这个是由`CONFIG_SYS_BOOTM_LEN`这个宏来控制的，对于非解压的镜像，不用考虑这个`CONFIG`值

![image-20241208163127114](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/fit_gzip/image-20241208163127114.png)





## 参考链接

- [boot解析](https://www.cnblogs.com/arnoldlu/p/11286841.html)

































