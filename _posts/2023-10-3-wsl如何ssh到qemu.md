---
layout:     post   				    
title:     wsl如何ssh到qemu			
subtitle:  
date:       2023-10-03				
author:     婷                               
header-img: img/91.png 	
catalog: true 						
tags:								

- wsl
- qemu
- ssh
---





### wsl配置

确保`wsl`有安装`ssh`相关软件，这里就不赘述了

```bash
sudo apt-get install ssh
sudo apt-get install openssh-client
```





### qemu启动脚本配置

`qemu`的启动脚本中增加一句

```bash
-net user,hostfwd=::2222-:22 -net nic
```

这句话的意思是，将`qemu`的`22`号端口映射到`hos`t(也就是`wsl`)的`2222`端口，**启动qemu**之后可以在`wsl`这边看到

![image-20231003180504792](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_ssh/image-20231003180504792.png)





完整的启动脚本如下所示

```bash
run_qemu_debian(){
		cmd="$QEMU -m 1024 -cpu max,sve=on,sve256=on -M virt,gic-version=3,its=on,iommu=smmuv3\
			-nographic $SMP -kernel arch/arm64/boot/Image \
			-append \"$kernel_arg $debug_arg $rootfs_arg $crash_arg $dyn_arg\"\
			-drive if=none,file=$rootfs_image,id=hd0\
			-device virtio-blk-device,drive=hd0\
			--fsdev local,id=kmod_dev,path=./kmodules,security_model=none\
			-device virtio-9p-pci,fsdev=kmod_dev,mount_tag=kmod_mount\
			-net user,hostfwd=::2222-:22  -net nic\
			$DBG"
		echo "running:"
		echo $cmd
		eval $cmd

}
```





### qemu配置

启动`qemu`之后，还要在我们`qemu`模拟的`debian`中做好`ssh`相关的配置，在`/etc/ssh/sshd_config`中加上一句

```bash
PermitRootLogin yes
```



![image-20231003122759803](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_ssh/image-20231003122759803.png)

然后重启`ssh`相关服务

```bash
sudo service ssh restart
```



### wsl连接

输入命令，输入密码后即可连接

```bash
ssh -p 2222 root@0.0.0.0
```



![image-20231003181508741](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/wsl_qemu_ssh/image-20231003181508741.png)





这里选择的是`0.0.0.0`这个IP是因为`netstat -anp`中显示的是

```bash
tcp        0      0 0.0.0.0:2222            0.0.0.0:*               LISTEN      44/qemu-system-aarc
```







### 参考链接

- [链接一](http://pwn4.fun/2019/05/31/Ubuntu%E4%B8%8BSSH%E7%99%BB%E5%BD%95Qemu%E8%99%9A%E6%8B%9F%E6%9C%BA/)

  宿主机要通过ssh访问虚拟机有两种网络配置方式，一种是用户模式网络，另一种是网桥网络模式。前面的我们使用的是第一种，这个参考链接还介绍了怎么用网桥桥接的方式。

