---
layout:     post   				    
title:     Windows下Typora增加图床功能			 
subtitle:  
date:       2022-02-22				
author:     婷                               
header-img: img/64.jpg 	
catalog: true 						
tags:								

- windows
- Typora
- 图床

---



## 前言

之前整理下`Linux`下怎么给`Typora`增加图床，详情见这个[链接](https://copyright1999.github.io/2021/10/18/Linux%E4%B8%8BTypora%E5%A2%9E%E5%8A%A0%E5%9B%BE%E5%BA%8A%E5%8A%9F%E8%83%BD/)，这次也想在`windows`下增加这个功能，其实道理都差不多，也是利用`PicGo`加`gitee`。





## 安装PicGo

`Windows`下的`Typora`版本要大于`0.9.84`版本才支持通过`PicGo`上传图片，所以首先要确保自己版本没有问题。



来到[官网](https://molunerfinn.com/PicGo/)，点击**免费下载**

![image-20220221011149091](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220221011149091.png)



点进去后选择红框中的安装包

![image-20220221011440516](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220221011440516.png)



下载后正常安装后，在安装的文件夹中点击`PicGo.exe`

![image-20220221012000397](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220221012000397.png)



然后就会发现任务栏下有这个小小的图标，点击图标就可以看到更详细的设置了



![image-20220221012037380](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220221012037380.png)





## 安装nodejs

因为`windows`下`PicGo`要安装`gitee`图床插件需要`nodejs`环境

来到[官网](https://nodejs.org/zh-cn/)，点击红框中的选项



![image-20220221012142003](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220221012142003.png)

正常安装后，命令行输入，如果出现版本信息，就说明安装成功了

```shell
npm -v
node -v
```



![image-20220221012932267](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220221012932267.png)

## gitee仓库设置

这部分之前的[链接](https://copyright1999.github.io/2021/10/18/Linux%E4%B8%8BTypora%E5%A2%9E%E5%8A%A0%E5%9B%BE%E5%BA%8A%E5%8A%9F%E8%83%BD/)已经有了就先略过了，主要是一个令牌的设置。



## PicGo配置

点击`PicGo`图标，来到**插件设置**，选择红框的插件（注意一定要看清楚是红框中的插件）

![image-20220221012547916](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220221012547916.png)



安装完成后点开**gitee设置**

![image-20220221013402693](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220221013402693.png)



在上一步的**gitee仓库设置**中将我们得到的令牌放到`token`中，然后填写必要的信息，再点击**确定**，**设为默认图床**即可

![image-20220222005623255](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220222005623255.png)





## Typora设置

打开`Typora`，点击**格式**-->**图像**-->**全局图像设置**，按照红框中的设置

![image-20220221013204889](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220221013204889.png)



设置完成后，点击**验证图片上传选项**，成功后，会有如下提示：

![image-20220222005741589](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/windows_typora_picgo/image-20220222005741589.png)



ok，至此完毕
