---
layout:     post   				    
title:      	Linux下Typora增加图床功能			# 标题 
subtitle:   
date:       2021-10-18			
author:    婷                               
header-img: img/55.jpg 	
catalog: true 						
tags:								

- Ubuntu  
- Linux
- Typora
- 图床
- markdown
---









这次的内容主要是参照网上的博客，利用`PicGo`在`Ubuntu`下实现`Typora`自动上传图片到图床的功能，这次用到的图床有`gitee`仓库跟`github`仓库

### 设置Gitee仓库

首先新建一个仓库

![image-20211016210631521](https://img-blog.csdnimg.cn/img_convert/4ba8178f113f40d6fe5648290856791c.png)

选择`开源`

![image-20211016210742555](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux%20typora%20markdown/image-20211016210742555.png)

给仓库添加一个`Readme`文件（个人理解这是一种初始化的行为）

![image-20211016210808207](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016210808207.png)

仓库建立完成后点击`设置`

![image-20211016210934935](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016210934935.png)

在`安全设置`里面找到`私人令牌`选项

![image-2021101621000264](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016211000264.png)

点击`生成新令牌`

![image-20211016211018886](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016211018886.png)

对私人令牌补充一些描述，按照红框把所有的选项都勾选上，最后点击`提交`。（因为我`gitee`这个账户只是个工具账户，所以全勾对我来说无所谓）

![image-20211016211124033](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016211124033.png)



如图所示，记得把私人令牌复制下来

![image-20211016211230090](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016211230090.png)



###　安装PicGo

打开这个[下载链接](https://github.com/Molunerfinn/PicGo/releases)，选择`AppImage`后缀的安装包下载

![image-20211016214750764](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016214750764.png)

下载后添加权限运行，第一次运行会跳出用户协议，直接回车跳过即可

```shell
chmod a+x PicGo-2.3.0.AppImage  
./PicGo-2.3.0.AppImage 
```



![image-20211016215537516](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016215537516.png)

运行后会发现侧边任务栏出现了这个小图标

![image-20211016215737788](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016215737788.png)

同时桌面上也会出现这个可以移动的图标

![image-20211016215702090](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016215702090.png)



### 设置PicGo

点击图标，`打开详细窗口`

![2021-10-16 21-58-56 的屏幕截图](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/2021-10-16%2021-58-56%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

或者右键点击桌面上的图标

![image-20211017152400385](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211017152400385.png)

打开之后在`插件设置`中搜索`gitee`，选择`gitee-uploader`安装。（记住不要选错！！！）

![image-20211016220029415](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016220029415.png)

安装后点击`图床设置`，往下拉找到`gitee`

![image-20211016221421418](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016221421418.png)



![image-20211016223149386](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016223149386.png)

填上前面申请的仓库以及令牌的一些信息，`token`则是前面申请的私人令牌，`path`则表示图片要上传到仓库的哪个路径下

![image-20211016224027042](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016224027042.png)

设置后，点击`确定`,然后点击`设为默认图床`



### 设置Typora

搜索`picgo`的存放路径

![image-20211016223622656](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016223622656.png)

打开`Typora`，点击`偏好设置`，按照红框的设置填写

![image-20211016224414238](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016224414238.png)



### 上传图片

首先要确保`PicGo`运行，用`Typora`打开，比如我要上传下面这张阿离的图片，首先先把图片拖进`Typora`

![阿离](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/%E9%98%BF%E7%A6%BB.jpg)



然后右击图片，点击`上传图片`

![2021-10-17 19-38-26 的屏幕截图](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/2021-10-17%2019-38-26%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)



上传中

![image-20211017194001005](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211017194001005.png)

上传成功的提示消息

![2021-10-17 19-40-13 的屏幕截图](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/2021-10-17%2019-40-13%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

来到自己的仓库，确认是已经上传成功了

![image-20211017194249250](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211017194249250.png)



### 设置Github仓库

接下来演示如何将`github`仓库当做图床

首先创建一个仓库，属性设置为`Public`，记得要勾选第二个框的选项，添加一个`README`文件来初始化这个仓库。

![image-20211016211832517](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016211832517.png)



创建完成后，点击`settings`

![image-20211016212026692](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016212026692.png)

点击`Developer settings`



![image-20211016212111528](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016212111528.png)



点击`Personal access tokens`，点击`Generate new token`

![image-20211016212133789](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016212133789.png)

`Note`添加描述，`Expiration`设置令牌生效的时间，我设置的是一年后，接下来只需要勾选`public_repo`选项

![image-20211016212516979](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016212516979.png)



结束后点击`Generate token`

![image-20211016212531913](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016212531913.png)

得到`token`后复制出来，备用

![image-20211016212602616](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211016212602616.png)

打开`PicGo`，在`图床设置`中，点击`Github图床`，然后填好设置后，点击`确定`，然后点击`设为默认图床`

![image-20211017200227125](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211017200227125.png)



跟前面一样在`Typora`里面上传我们的阿离，上传成功后有个提示

![2021-10-18 01-14-59 的屏幕截图](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/2021-10-18%2001-14-59%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

点击查看确实是我们可爱的阿离

![image-20211018011418510](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/%20linux%20typora%20markdown/image-20211018011418510.png)





### 参考链接

- [链接一](https://blog.csdn.net/qq_20549061/article/details/106796119) 



