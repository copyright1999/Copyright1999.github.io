---
layout:     post   				    
title:     wsl升级cmake			
subtitle:  
date:       2025-03-09			
author:     婷                               
header-img: img/144.png 
catalog: true 						
tags:								

- wsl
- cmake

---





## 简介

在`wsl`上搭建`zephyr`环境的时候，需要升级`cmake`，特此记录一下





##  过程

获取`cmake`源码，这里选择的是`3.23`版本 

```bash
sudo wget https://cmake.org/files/v3.23/cmake-3.23.0.tar.gz
sudo tar -zxvf cmake-3.23.0.tar.gz
```



![image-20250302162906706](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/cmake_wsl/image-20250302162906706.png)

![image-20250302163434148](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/cmake_wsl/image-20250302163434148.png)



解压`cmake`源码后，进行配置

```bash
sudo ./configurre
```



![image-20250302163515600](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/cmake_wsl/image-20250302163515600.png)

编译

```bash
sudo make -j8
```



![image-20250302163841261](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/cmake_wsl/image-20250302163841261.png)

安装

```bash
sudo make install
```



![image-20250302164416004](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/cmake_wsl/image-20250302164416004.png)





```bash
cmake --version
```

查看系统的`cmake`版本，如果是我们刚刚编译安装的`3.23`版本，那么到这里就ok了

![image-20250302164459350](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/cmake_wsl/image-20250302164459350.png)



如果还不是的话，因为现在系统有两个`cmake`的版本，所以我们需要建立下软链接

```bash
sudo update-alternatives --install /usr/bin/cmake cmake /usr/local/bin/cmake 1 -force
```





## 参考链接

- [参考链接一](https://blog.csdn.net/loveCC_orange/article/details/136073681)

