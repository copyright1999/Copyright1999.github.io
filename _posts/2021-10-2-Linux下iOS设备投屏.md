---
layout:     post   				  
title:    Linux下iOS设备投屏
subtitle:  给uxplay点赞
date:       2021-10-02				
author:     婷                              
header-img: img/59.jpg 	#这篇文章标题背景图片
catalog: true 						
tags:								

- iOS
- uxplay
- 投屏
---





### 编译uxplay源码

来到`uxplay`的`github`[主页](https://github.com/antimof/UxPlay)，下载源码。（突然发现很少有人发现这个项目）

![1.png](https://i.loli.net/2021/10/02/65DuMcsPfAjtvbC.png)



根据`Readme.md`提示，要编译源码需要依赖的环境为前三部分

![2.png](https://i.loli.net/2021/10/02/z9QlUKVsm8yaPRi.png)

`cmake`很早前我就安装过了，这里就不演示了



来安装第二个跟第三个依赖

```shell
sudo apt-get install libssl-dev libavahi-compat-libdnssd-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-libav
```

![3.png](https://i.loli.net/2021/10/02/yvE36xVGisTpemW.png)



```shell
sudo apt-get install gstreamer1.0-vaapi 
```

![4.png](https://i.loli.net/2021/10/02/JWjCzwMnKmBdGDO.png)



下载好依赖后，来到源码路径下，因为整个工程都是用`cmake`去编译的，所以编译过程跟往常一样，但是如果前面两个依赖没有下载的话，编译的时候链接不到库就会编译失败。

```shell
mkdir build
cd build
cmake ..
make
```

![5.png](https://i.loli.net/2021/10/02/2HwxrekWj9qQAoE.png)



![6.png](https://i.loli.net/2021/10/02/BmEziLpsWK1hlFQ.png)

最后生成的可执行文件`uxplay`就放在这个`build`目录下了

![7.png](https://i.loli.net/2021/10/02/AdhWb4tkvYQ2ujT.png)

如果想要直接命令行输入`uxplay`即可启动，可以将这个可执行文件的路径添加到环境变量中

查明当前`uxplay`的路径，然后编辑`etc/profile`文件

![8.png](https://i.loli.net/2021/10/02/WPAz6MvKlOeSZdD.png)



在文件末尾添加`export PATH=$PATH:/home/copyright/UxPlay-master/build`

![9.png](https://i.loli.net/2021/10/02/aV3AgkdTUFGmu7S.png)



保存退出后终端再运行`source /etc/profile`即可生效





### 运行uxplay

打开终端，输入`uxplay`，此时确保你的苹果手机或者苹果的平板跟你的电脑在同个网络环境下

![10.png](https://i.loli.net/2021/10/02/bpST1Lwa3qGehrs.png)



在平板上拉下控制台 ，选择`屏幕镜像`

![4qE4m9.png](https://z3.ax1x.com/2021/10/02/4qE4m9.png)

点击进去后选择`UxPlay`

![4qEIT1.png](https://z3.ax1x.com/2021/10/02/4qEIT1.png)

连接成功后会弹出一个小框，同时在终端也有提示

![4qEfOJ.png](https://z3.ax1x.com/2021/10/02/4qEfOJ.png)

![4qEWy4.png](https://z3.ax1x.com/2021/10/02/4qEWy4.png)

这是平板的竖屏显示

![4qE5wR.png](https://z3.ax1x.com/2021/10/02/4qE5wR.png)

这是横版显示

![4qE7Y6.png](https://z3.ax1x.com/2021/10/02/4qE7Y6.png)

锁屏也不慌（其实是为了秀老婆）

![4qEHfK.png](https://z3.ax1x.com/2021/10/02/4qEHfK.png)



点开B站，会显示视频在`AirPlay`播放，声音也会投放到电脑上，平板端是这样显示的![4qEXOH.png](https://z3.ax1x.com/2021/10/02/4qEXOH.png)

而电脑这边则是直接全屏显示，相当于在电视投屏一样

![4qELlD.png](https://z3.ax1x.com/2021/10/02/4qELlD.png)



![4qEqSO.png](https://z3.ax1x.com/2021/10/02/4qEqSO.png)



### 其他说明

这个开源项目真的很赞，延迟很小，清晰度也很好，而且用起来很丝滑，不怎么占用电脑资源。唯一的小缺点就是多个设备切换来链接`uxplay`的时候可能图像会发生冲突，这时候直接关闭再运行一次就够了。总之就很ok。



### 参考链接

- [uxplay项目](https://github.com/antimof/UxPlay)
- [修改环境变量](https://blog.csdn.net/White_Idiot/article/details/78253004)











