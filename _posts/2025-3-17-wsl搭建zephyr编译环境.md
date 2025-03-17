---
layout:     post   				    
title:      wsl搭建zephyr编译环境			
subtitle:  
date:       2025-03-17				
author:     婷                               
header-img: img/146.png
catalog: true 						
tags:								

- wsl
- zephyr
- mcxa156
- raspi

---





## 简介

简单的记录下`wsl`下搭建`zephyr`编译环境的过程





## 配置环境

编译`zephyr`需要得到最小版本分别为

| Tool   | Min.Version |
| ------ | ----------- |
| cmake  | 3.20.5      |
| python | 3.10        |
| dtc    | 1.4.6       |



而我们的系统除了`dtc`其他的都需要升级

![image-20250302162359337](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250302162359337.png)



先更新一下

![image-20250302162247908](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250302162247908.png)



### 升级cmake

详情见之前的[文章](https://copyright1999.github.io/2025/03/09/wsl%E5%8D%87%E7%BA%A7cmake/)



### 升级python3

详情见之前的[文章](https://copyright1999.github.io/2025/03/09/wsl%E5%8D%87%E7%BA%A7python3%E7%89%88%E6%9C%AC/)





### 增加kitware源

我的版本是`20.04`，而官方提到如果不是`22.04`版本的还需要单独加`kitware`的`apt`源

![image-20250302201109001](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250302201109001.png)

安装过程如下

```
wget https://apt.kitware.com/kitware-archive.sh
sudo bash kitware-archive.sh
```

![image-20250302201704511](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250302201704511.png)



![image-20250302201720233](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250302201720233.png)



### 安装依赖

```bash
sudo apt install --no-install-recommends git cmake ninja-build gperf \
ccache dfu-util device-tree-compiler wget \
python3-dev python3-pip python3-setuptools python3-tk python3-wheel xz-utils file \
make gcc gcc-multilib g++-multilib libsdl2-dev libmagic1
```



![image-20250302201850490](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250302201850490.png)



```bash
sudo apt install python3-venv
```

![image-20250302202450850](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250302202450850.png)





```bash
sudo apt-get install python3.10-venv
```

![image-20250302203730279](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250302203730279.png)





## 获取zephyr代码

创建`python`的虚拟环境并激活，在虚拟环境中安装`west`工具

```bash
python3 -m venv ~/raspi_zephyr/.venv
source ~/raspi_zephyr/.venv/bin/activate
pip install west
```





![image-20250302203838306](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250302203838306.png)



![image-20250302234432772](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250302234432772.png)



获取`zephyr`的源码，注意路景观

```bash
west init ~/raspi_zephyr
cd ~/raspi_zephyr
west update
```



![image-20250303233704787](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250303233704787.png)



![image-20250304002810584](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250304002810584.png)

`update`结束

![image-20250308120302777](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308120302777.png)



导出`cmake`的包，这样`cmake`就可以自动加载构建`zephyr`应用程序所需的样板代码

```bash
west zephyr-export
```

![image-20250308120331676](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308120331676.png)



更新`python`需要的包

```bash
west packages pip --install
```



![image-20250308120406913](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308120406913.png)



![image-20250308120535983](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308120535983.png)



## 安装zephyr sdk

进入`zephyr`文件夹下，使用`west`安装`sdk`

```bash
cd zephyr
west sdk install
```



![image-20250308120750302](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308120750302.png)

![image-20250308120805612](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308120805612.png)



![image-20250308121950414](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308121950414.png)





## mcxa156 demo

打算用`zephyr`的`mcxa156`的`demo`，不过因为我是`wsl`的环境，所以在烧录`flash`的操作那一步没进行下去，这个等后面有时间回头看看怎么操作，这里先记录过程。



`zephyr`提供了一些类似于`hello world`之类的简单`demo`，可以点开下面的[链接](https://docs.zephyrproject.org/latest/boards/index.html#boards=)，查看哪些支持的板子

```
https://docs.zephyrproject.org/latest/boards/index.html#boards=
```

按照下图的关键词来搜索

![image-20250317222129238](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250317222129238.png)

搜索后可以看到具体的版子都显示出来了，点击[进去](https://docs.zephyrproject.org/latest/boards/nxp/frdm_mcxa156/doc/index.html)，有简单的使用提示

![image-20250308162331074](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308162331074.png)



编译代码，生成镜像

```bash
west build -b frdm_mcxa156 samples/hello_world
```

![image-20250308162744774](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308162744774.png)



![image-20250308162812601](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308162812601.png)



根据`zephyr`官方的提示，我们的`mcxa156`板子用的是`MCU-Link`，所以烧录的话，直接使用下列命令

```bash
west flash
```



不过因为我们是在`wsl`里面开发，所以得指定设备节点，不然是没法使用的

![image-20250308164527684](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250308164527684.png)



参考了一个[esp32](https://lgl88911.github.io/2021/05/22/Zephyr-ESP32%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/)的文档，按道理应该就是用`/dev/ttyACM0`节点的，不过好像`mcxa156`使用都是失败的，反正就先记录到这里，等后面有空回头来解决这个问题

```bash
west flash --frdm_mcxa156-device  /dev/ttyACM0
west flash --mcxa156-device  /dev/ttyACM0
```





## raspi 4b

接下来记录树莓派`4B`板子的`demo`使用过程



找到我们的`boardname`，这里是`rpi_4b`

![image-20250309112747785](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250309112747785.png)





```bash
rm -rf build
west build -b rpi_4b samples/hello_world
```



![image-20250309113148960](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250309113148960.png)



拷贝我们的`bin`文件出来

```bash
cp build/zephyr/zephyr.bin /d/ubuntu_swap/
```



![image-20250309113308219](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250309113308219.png)



放到树莓派的`SD`卡，如下，`zephyr demo`启动成功

![image-20250309113416269](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_zephyr/image-20250309113416269.png)





## 参考链接

- [参考链接一](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)

- [参考链接二](https://www.guoxingyong.net/category/course/)
- [zephyr支持的版型](https://docs.zephyrproject.org/latest/boards/index.html#boards=)
- [mcxa156](https://docs.zephyrproject.org/latest/boards/nxp/frdm_mcxa156/doc/index.html)
- [esp32](https://lgl88911.github.io/2021/05/22/Zephyr-ESP32%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/)





## 待办事项

- 写一个简单的`zephyr demo`

- 配置`zephyr`，使其拥有`shell`界面，如同`linux`的命令行一样

- 上述`zephyr`的下载过程实在是太繁琐了，而且编译还依赖`zephyr`的`SDK`，感觉就跟用`MDK`软件之类的没啥区别，所以看看有没有简单的配置流程








