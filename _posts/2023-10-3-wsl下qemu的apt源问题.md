---
layout:     post   				    
title:     wsl下qemu的apt源问题			
subtitle:  
date:       2023-10-03				
author:     婷                               
header-img: img/92.png 	
catalog: true 						
tags:								

- wsl
- qemu
- apt

---





### 问题

`qemu`模拟的是`arm64`的`debian`系统，在`apt update`的时候，总会报错，导致我想下个`tcpdump`也没有找到包

![image-20231003164734532](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_aptsource/image-20231003164734532.png)



```bash
Get:1 http://mirrors.ustc.edu.cn/debian unstable InRelease [195 kB]
Err:1 http://mirrors.ustc.edu.cn/debian unstable InRelease
  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 0E98404D386FA1D9 NO_PUBKEY 6ED0E7B82643E131
Reading package lists... Done
W: GPG error: http://mirrors.ustc.edu.cn/debian unstable InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 0E98404D386FA1D9 NO_PUBKEY 6ED0E7B82643E131
E: The repository 'http://mirrors.ustc.edu.cn/debian unstable InRelease' is not signed.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```



`debian`的版本

![image-20231003163929077](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_aptsource/image-20231003163929077.png)





我的`qemu`中的`/etc/apt/sources.list`文件内容如下

```bash
deb http://mirrors.ustc.edu.cn/debian/ unstable main non-free contrib
deb-src http://mirrors.ustc.edu.cn/debian/ unstable main non-free contrib
```



### 解决方法

前面的报错提示信息中有两个`key`，分别是`6ED0E7B82643E131`跟`0E98404D386FA1D9`

![image-20231003165103348](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_aptsource/image-20231003165103348.png)



输入这两条命令，然后重新`update`即可

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 6ED0E7B82643E131
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 0E98404D386FA1D9
```



![image-20231003165900948](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_aptsource/image-20231003165900948.png)



