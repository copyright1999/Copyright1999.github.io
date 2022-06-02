---
layout:     post   				   
title:     tcpdump编译以及交叉编译			
subtitle:  通用脚本一次性搞定
date:       2022-06-03				
author:     婷                               
header-img: img/68.jpg 
catalog: true 						
tags:								

- tcpdump
- 交叉编译

---



## 前言

有时候会遇到用`tcpdump`工具来抓包分析，大多数时候需要交叉编译，而交叉编译总是离不开`libcap`跟`tcpdump`，记得之前自己下载源码编译半天还是失败，后来用`buildroot`相当的方便。不过今天文章里面想分享的方法，是通过一个编译脚本即可解决所有的烦恼，你可以交叉编译，又可以根据你自己当前机器的架构来。



## 准备

### 安装flex 与 bison

> Flex 和 Bison 是两个在编译前期最常实验的工具，分别是用来做 lexical analyse 和 semantic analyse 的

好吧这两个工具我也不熟悉，我只知道当我`configure`出现这个报错的时候，需要用到它们

```shell
configure: error: Neither flex nor lex was found.
```



不管怎样，更新`apt`资源后，安装

```shell
sudo apt-get update
sudo apt-get install flex bison
```

![image-20220602235909160](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220602235909160.png)





### libcap tcpdump源码下载

来到[官网](https://www.tcpdump.org/)，点击红框中的`Latest Releases`

![image-20220602232822396](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220602232822396.png)



然后我这里是选择官网最新的`tcpdump-4.99.1`跟`libcap-1.10.1`，点击红框即可进行下载

![image-20220602232918233](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220602232918233.png)

下载后将这两个压缩包放在同个目录下，比如我就放在`tcpdumpcompile`目录下，然后解压，删除掉压缩包（节省空间）

```shell
tar -zxvf libpcap-1.10.1.tar.gz
tar -zxvf tcpdump-4.99.1.tar.gz
```



![image-20220602233602279](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220602233602279.png)



## 编译脚本

在`tcpdumpcompile`目录下，编写如下脚本，我命名为`compile.sh`

```shell
#!/bin/bash
usage()
{
        echo "***********************************************************"
        echo "eg."
        echo "  ./compile.sh arm"
        echo "***********************************************************"
}

str_to_lower()
{
        echo $1 | tr '[A-Z]' '[a-z]'
}

parse_param()
{
        str=$(str_to_lower $1)
        if [ "$str" == "arm" ] || [ "$str" == "arm32" ]; then
                PLATFORM=$str
                CROSS_COMPILE=arm-linux-gnueabihf-
        elif [ "$str" == "arm64" ]; then
                PLATFORM=$str
                CROSS_COMPILE=aarch64-linux-gnu-
        elif [ "$str" == "ubuntu" ]; then
                PLATFORM=$str
                CROSS_COMPILE=
        elif [ "$str" == "what_you_want" ]; then
                PLATFORM=$str
                CROSS_COMPILE=what_you_want
        else
                usage
                exit 1
        fi
}

PLATFORM=arm
CROSS_COMPILE=arm-linux-gnueabihf-


if [ $# -eq 0 ] ; then
        echo "Default setting:PLATFORM=${PLATFORM}"
        echo "Press Enter key to continue or \"Ctrl+C\" to stop and run \"compile.sh --h\" for help..."
        read -t 10
        parse_param ${PLATFORM}
elif [ $# -eq 1 ] ; then
        parse_param $1
else
        usage
        exit 1
fi

ROOT_DIR=`pwd`
PCAP_INSTALL_DIR=${ROOT_DIR}/libpcap-1.10.1/release
INSTALL_DIR=${ROOT_DIR}/${PLATFORM}/tcpdump
CROSS_STRIP=${CROSS_COMPILE}strip

cd libpcap-1.10.1
make clean
make distclean
if [ "$PLATFORM" != "ubuntu" ];then
        ./configure CC=${CROSS_COMPILE}gcc --host=arm --prefix=${PCAP_INSTALL_DIR}
else
        ./configure CC=${CROSS_COMPILE}gcc  --prefix=${PCAP_INSTALL_DIR}
fi
make
make install

cd ../tcpdump-4.99.1
make clean
make distclean
if [ "$PLATFORM" != "ubuntu" ];then
        ./configure CC=${CROSS_COMPILE}gcc --host=arm --prefix=${INSTALL_DIR} --without-crypto
else
        ./configure CC=${CROSS_COMPILE}gcc  --prefix=${INSTALL_DIR} --without-crypto
fi
make
make install
#${CROSS_STRIP} ${INSTALL_DIR}/sbin/*

```

其实脚本很简单，只要你稍微懂点`shell`的语法以及接触过`automake`，应该都能看懂，这里不赘述了，然后接下来我就从用法，复用性两方面简要说下

### 用法

很简单，输入`./compile.sh xxx`，这个`xxx`是指你的机器架构，目前脚本中支持的有这几个，如果有自己特定的平台跟编译工具链可以自己往上添加

![image-20220603010557144](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220603010557144.png)

### 复用性

这个脚本还是主要针对`tcpdump-4.99.1`跟`libcap-1.10.1`这两个版本的源码，如果版本不是对应的还需要自己修改脚本。（不过其实我也可以让脚本多加几个入参，来确定版本）

![image-20220603010729289](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220603010729289.png)



### 例子

交叉编译适用`ARM 32bit`系统的程序

```shell
./compile.sh arm
```

![image-20220602235146541](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220602235146541.png)

最后生成的成果物，用`file`查看其属性

![image-20220603000555956](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220603000555956.png)



```shell
copyright@copyright-Vostro-3559:~/tcpdumpcompile$ file arm32/tcpdump/bin/tcpdump
arm32/tcpdump/bin/tcpdump: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 2.6.31, BuildID[sha1]=8d2f92c65c6881d010692ca105cc9519a60bd0b9, with debug_info, not stripped

```



放到我们上`ARM 32bit`系统的板子，执行成功

![image-20220603001221616](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220603001221616.png)



编译适合于`Ubuntu x86-64`的系统的程序

```shell
./compile.sh ubuntu
```

用`file`查看成果物的属性

![image-20220603004254977](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220603004254977.png)

再我们自己的`Ubuntu`系统上运行成功

![image-20220603004316854](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/tcpdumpcompile/image-20220603004316854.png)



## 后话

后续可以写下怎么用`tcpdump`抓到的包放到`wiresharks`上分析



## 参考链接

- [configure错误](https://blog.csdn.net/ai2000ai/article/details/71082245)
- [flex bison](https://www.cnblogs.com/wAther/p/10662978.html )
