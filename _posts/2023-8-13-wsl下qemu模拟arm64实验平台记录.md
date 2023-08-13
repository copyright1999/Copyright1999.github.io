---
layout:     post   				    
title:     wsl下qemu模拟arm64实验平台记录			
subtitle:  
date:       2023-08-13				
author:     婷                              
header-img: img/88.png	
catalog: true 						
tags:								

- wsl
- qemu

---



### 总结

单纯记录下，`qemu`模拟`arm64`实验平台的过程，用别人的弄好的`kernel`代码跟做好的根文件系统，比较简单。



### 过程

`github`仓库

```bash
https://github.com/runninglinuxkernel/runninglinuxkernel_5.0
```

克隆该仓库

````
git clone git@github.com:runninglinuxkernel/runninglinuxkernel_5.0.git
````



![image-20230812114500562](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_arm64/image-20230812114500562.png)



确保已经安装好了工具链，用命令`aarch64-linux-gnu-gcc -v`查看

![image-20230812114547585](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_arm64/image-20230812114547585.png)



直接运行脚本，编译`kernel`。

```bash
./run_rlk_arm64.sh build_kernel
```



![image-20230812115325443](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_arm64/image-20230812115325443.png)



查看生成的镜像文件

![image-20230812115829969](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_arm64/image-20230812115829969.png)



接着制作根文件系统，大概需要`4GB`的磁盘空间大小，而且需要用`sudo`权限，输入命令

```bash
sudo ./run_rlk_arm64.sh build_rootfs
```

![image-20230812120145958](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_arm64/image-20230812120145958.png)



制作完成

![image-20230812120156770](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_arm64/image-20230812120156770.png)



查看生成的根文件系统`rootfs_debian_arm64.ext4`

![image-20230813120315770](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_arm64/image-20230813120315770.png)



输入命令，利用`qemu`启动

```bash
./run_rlk_arm64.sh run
```



![image-20230812120228916](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_arm64/image-20230812120228916.png)



登录用户为**root**，密码是**123**

![image-20230812120310800](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_arm64/image-20230812120310800.png)



登录成功

![image-20230812120330371](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_arm64/image-20230812120330371.png)
