---
layout:     post   				    
title:     wsl升级python3版本			
subtitle:  
date:       2025-03-09				
author:     婷                              
header-img: img/145.png 	
catalog: true 						
tags:								

- wsl
- python3

---





## 简介

介绍`wsl`怎么升级系统原生自动的`python3`版本，从`python3.8.10`到`python3.10`





## 升级python3

原始的系统版本是`3.8.10`

![image-20250302164656253](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302164656253.png)





```bash
sudo add-apt-repository ppa:deadsnakes/ppa 
```

添加`apt`源

![image-20250302164308608](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302164308608.png)



选择敲回车键

![image-20250302164320010](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302164320010.png)



![image-20250302164337757](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302164337757.png)



更新一下

```bash
sudo apt update
```

![image-20250302164600166](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302164600166.png)



查看是否有`python3.10`，这个是我们即将要升级的版本，选择安装

```bash
apt list | grep python3.10
sudo apt install python3.10
```



![image-20250302164737993](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302164737993.png)



安装了之后，因为系统中存在两个版本，所以我们要设置默认的版本为`python3.10`

```bash
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 1
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 2
sudo update-alternatives --config python3
```

![image-20250302165032641](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302165032641.png)





这时候如果用`pip`命令来安装依赖，还是会报错，需要执行以下命令来修复

```bash
sudo apt remove --purge python3-apt
sudo apt autoclean
sudo apt install python3-apt
sudo apt install python3.10-distutils
```



![image-20250302165119425](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302165119425.png)

![image-20250302165152989](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302165152989.png)



![image-20250302165226559](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302165226559.png)



![image-20250302165258094](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302165258094.png)





下载`get-pip.py`并安装，即可解决`pip`的问题

```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
sudo python3.10 get-pip.py
```



![image-20250302165338277](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302165338277-1741522558240-16.png)



![image-20250302201030511](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_python3/image-20250302201030511-1741522546393-10.png)





## 参考链接

- [参考链接一](https://www.gsgundam.com/2023/01/2023-01-18-z08-upgrade-python-3-in-ubuntu-of-wsl-windows/)

