---
layout:     post   				    # 使用的布局（不需要改）
title:      	sudo apt-get update报错			# 标题 
subtitle:  Ubuntu18.10下nodejs源无法更新  #副标题
date:       2019-08-23				# 时间
author:     瞎搞了一整个白天的婷                               # 作者
header-img: img/17.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签

- Ubuntu  
- markdown
---





&#160; &#160; &#160; &#160;今早本来想看飞控SDK的代码，因为感觉两个系统切来切去过于麻烦，于是萌生了想要在Ubuntu下玩stm32的想法，突然想起我Ubuntu下没有markdown编辑器，之前在windows用的是typora想着Ubuntu下面也装一个。~~脑回路有点新奇~~

---

&#160; &#160; &#160; &#160;打开[typora官网](https://www.typora.io/)，点击download ，选择[linux的安装包](https://www.typora.io/#linux)，官网直接把命令给你，很简单是不是

```bash
# or run:
# sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
# add Typora's repository
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update
# install typora
sudo apt-get install typora
```

![4.jpeg](https://i.loli.net/2019/08/23/sB3oaFwPUd4V8qj.jpg)

&#160; &#160; 当我输入第二条命令的时候开始出现了这个报错

```bash
$ sudo add-apt-repository 'deb https://typora.io/linux ./'
```

![2.jpeg](https://i.loli.net/2019/08/23/nYcdfVZMrlg9Phs.jpg)



![5.jpg](https://i.loli.net/2019/08/23/VZH5TS3qtPJKk6b.jpg)



嗯？看不懂？~~原谅我是一个刚接触Linux的白痴~~

我就想着跳过直接输入接下来的命令

```bash
$ sudo apt-get update
```

emmm还是报错了

![1.jpeg](https://i.loli.net/2019/08/23/b9TaWHntviQxPVy.jpg)



&#160; &#160; &#160; &#160;`ctrl+c`  `ctrl+v` 这个报错去百度了一下，试了很多乱起八糟的方法，wtf还是不行,一开始以为是typora的问题，大不了就不用typora了。然后换了很多个markdown编辑器，每次下载完了安装包，只要一输入  `sudo apt-get update `  然而还是报同一个错误，初步猜测是nodejs的问题，因为前天有个软件需要安装nodejs才能用，安装之前`sudo apt-get update`的命令还是不会报错的然后又查了很多方法卸载重装卸载重装，然而只要输入这个命令还是会，最后没辙只好向实验室的小伙伴求助。

---



&#160; &#160; &#160; &#160;果然专业的就是专业，小伙伴看了一下我的报错信息，指出是我访问不到nodejs的源，所以无法更新其他的源，把nodejs的源删掉就好了。接下来直接远程，一波操作猛如虎，输入了这几个命令之后就把之前nodejs的源删掉了。

```bash
$ cd /etc/apt/sources.list.d
$ la
$ rm chris* 									 #删掉nodejs的那个源
$la                                                      #确保真的删除干净了

#接下来就是愉快的安装typora啦
$ sudo apt-get update
$ sudo apt-get install typora

```

大功告成！美滋滋～

&#160; &#160; &#160; &#160;不过nodejs我还是需要用到，可能之前跟着百度到的命令瞎搞才导致自己出现nodejs的源无法更新的问题，然后实验室的小伙伴叫我直接从源码安装（ [内附教程](https://github.com/nodesource/distributions/blob/master/README.md#deb)）。

```bash
# Using Ubuntu
$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
$ sudo apt-get install -y nodejs

#至此完毕，输入下列命令，查看你nodejs的版本是多少同时也说明你安装成功啦
$ nodejs -v

#再输入这个命令看看还会不会报错
$ sudo apt-get update                                      #nice~不会报错了
```

在此感谢实验室的小伙伴～

![6.jpeg](https://i.loli.net/2019/08/23/8ZkGQeRFNYPKOCt.jpg)

---

 &#160; &#160; 写在最后，举一反三，下次如果再遇见这种问题就可以根据报错提示做个猜测，是不是软件的源问题啦。
