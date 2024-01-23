---
layout:     post   				    
title:     iptables			
subtitle:  
date:       2024-01-23				
author:     婷                              
header-img: img/108.png 
catalog: true 						
tags:								

- iptables
- netfilter
- 网络
---





### 概述

`netfilter`跟`iptables`组成`linux`平台下的包过滤防火墙，可以完成封包过滤 ，封包重定向 ，网络地址转换等功能。

![image-20231231204136466](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231204136466.png)







这次具体讲`iptables`的用法。



### iptables基础

主要讲讲五链四表的概念

#### 五链

![image-20231230232901934](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231230232901934.png)

- `PREROUTING `链：路由选择前
- `INPUT `链：数据包流入口，路由目的地为本机
- `FORWARD` 链：路由目的地非本机，转发
- `OUTPUT `链：数据包出口，本机发出数据包
- `POSTROUTING` 链：路由选择后



一般到本机的报文，链路是`PREROUTING -> INPUT`；由本地转发的报文，链路是`PREROUTING ->FORWARD -> POSTROUTING` ；本机发送出去的报文，链路是`OUTPUT -> POSTROUTING` 。



`linux`本身也可以看做是一个路由器，如果要实现转发功能，需要打开转发功能

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```



这里提一嘴，五链其实也对应`netfilter `注册的五个`HOOK`函数位置

![image-20231231204322101](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231204322101.png)



#### 四表

这里的表主要讲的是`iptables`的四个表

- `filter`表：负责过滤功能，防火墙，对应的内核模块为`iptable_filter`
- `nat`表：`network address translation`，实现网络地址转换，端口映射，地址映射等功能，对应的内核模块为`iptable_nat`
- `mangle`表：拆解报文，做出修改，并重新封装，例如更改`IP`标头的`TOS / DSCP / ECN`位，对应的内核模块为`iptable_mangle`
- `raw`表：实现数据跟踪，对应的内核模块为`iptable_raw`





`iptables`的`-t`参数可以指定查看具体的表，如果不指定，一般默认查看的`filter`表

![image-20231230175151783](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231230175151783.png)



![image-20231230175213337](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231230175213337.png)



如果当`iptables`无法使用某个表的时候，可以`lsmod`查看是否有安装相关的内核模块

![image-20231230180339809](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231230180339809.png)





#### 表链关系

不一定每个表都会在每个链中使用，有一定的表链关系的。



从链到表的关系

- `PREROUTING`的规则可存在于：`raw`表，`magle`表，`nat`表
- `INPUT`的规则可存在于：`magle`表，`filter`表
- `FORWARD`的规则可存在于：`magle`表，`filter`表
- `OUTPUT`的规则可存在于：`raw`表，`magle`表，`nat`表，`filter`表
- `POSTROUTING`的规则可存在于：`magle`表，`nat`表

![image-20231230233337768](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231230233337768.png)

这里有个优先级需要注意，数据包经过一个链时，会将当前链的所有表都匹配一遍，四表的优先级从高到低为，`raw > mangle  >  nat  > filter`。然后在表内，表内的所有规则都会匹配一遍，匹配的时候按照表内的顺序依次执行。



从表到链的对应关系

- `raw`表：`PREROUTING`，`OUTPUT`
- `magle`表：`PREROUTING`，`INPUT`，`FORWARD`，`OUTPUT`，`POSTROUTING`
- `nat`表：`PREROUTING`，`OUTPUT`，`POSTROUTING`
- `filter`表：`INPUT`，`FORWARD`，`OUTPUT`





#### 规则

`iptables`实现防火墙功能的一个特点是它是按照数据包头来匹配规则的。规则一般的定义为：如果数据包头符合这样的条件，就这样处理数据包。所以我们在五链上编写规则，可以对经过的数据包进行处理。

`iptables`中定义有五条链，每条链中可以定义多条 `Rules`，每当数据包到达一条链时，`iptables`就会从链上第一条规则开始匹配，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据该条规则所定义的规则逻辑（称为 `target`）处理该数据包；否则 `iptables `将继续检查下一条规则，如果该数据包不符合链中任一条规则，`iptables`就会根据预先定义的默认策略来处理数据。

以`nat`表看，现在的默认策略是`ACCEPT`

![image-20231231164256912](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231164256912.png)



默认策略可以通过`-P`参数来修改，不过貌似只能修改`filter`表

![image-20231230204012151](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231230204012151.png)





### iptables配置详解

`iptables`的基本命令格式如下

```bash
iptables  [-t 表名] [-A/I/D/R 规则链名]  [-i/o 网卡名]  [-p 协议名] [-s 源IP/源子网] [-d 目标IP/目标子网]
[--sport 源端口]  [--dport 目标端口] [-j 动作/target]
```

 

-  `-t` ：指定要操纵的表
- `-A/I/D/R`：规则链名 （其中`A`跟`I`是有一定区别的，而且也会影响到规则匹配的优先级）
  - `A`：向指定的链添加条目 ，在链的末尾追加一条规则
  - `D`：向指定的链删除条目
  - `I`：向指定的链中插入条目 ， 在链的开头或者指定序号插入一条规则
  - `R`：替换指定的链中的条目

- `-i/o`：网卡名
  - `-i`： 指定数据包进入本机使用的网络接口 
  - `-o`：指定数据包离开本机使用的网络接口
- `-p` ：协议名，一般只能是传输层跟网络层的协议
- `-s` ：源`IP`/源子网 ，指定要匹配的数据包源`IP`地址
- `-d`： 目标`IP`/目标子网
- `--sport`：源端口
- `--dport`：目标端口
- `-j`：动作
  - `ACCEPT` ：允许数据包通过
  - `DROP` ：直接丢弃数据包，不给任何回应信息
  - `REJECT` ：拒绝数据包通过，必要时会给数据发送端一个相应的信息，客户端刚请求就会收到拒绝的信息
  - `SNAT` ：源地址转换，解决内网用户用同一个公网地址上网的问题
  - `MASQUERADE` ：是`SNAT`的一种特殊形式，适用于动态的，临时会变的`IP`上
  - `DNAT`： 目标地址转换
  - `REDIRECT`： 在本机做端口映射
  - `LOG` ：在`/var/log/messages`文件中记录日志信息，然后将数据包传递给下一条规则，即除了记录外不对数据包做任何其他操作，仍然让下一条规则进行匹配





#### 显示规则

```bash
iptables -nL --line-number
```

- `-n `以数字形式显示规则
- `-L `列出链表里的所有规则
- `--line-number` 打印规则序号

![image-20231230180619813](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231230180619813.png)





#### 清空防火墙设置

```bash
iptables -F
iptables -X
iptables -Z
```

- `F` 清除规则链中已有的条目

- `X `清除用户自定义的规则链

- `Z`清空规则链中的数据包计算器和字节计数器

  

![image-20231231205519826](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231205519826.png)



![image-20231231205556247](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231205556247.png)





#### 修改默认策略

貌似只修改`filter`表

```bash
iptables -P  INPUT DROP
iptables -P  INPUT ACCEPT
```



![image-20231231210102936](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231210102936.png)





#### 拒绝，丢弃与优先级

`PC `是**192.168.1.102**， 设备是**192.168.1.106**

`PC`此时能`ping`通设备

![image-20231231210703932](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231210703932.png)



添加如下规则，可以看到对于`REJECT`，`PC`这边显示的是无法连到端口

```bash
iptables -t filter -I INPUT -p icmp -j REJECT
```



![image-20231231210758187](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231210758187.png)



抓包的时候可以看到其实设备是有回复包的，这是`REJECT`的时候的情况

![image-20231231211032567](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231211032567.png)





![image-20231231211228051](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231211228051.png)



清除刚刚的规则

```bash
iptables -t filter -D INPUT -p icmp -j REJECT
```



现在添加一条`DROP`的规则

```bash
iptables -t filter -A INPUT -p icmp -j DROP
```

`PC`这边显示的是请求超时

![image-20231231211657442](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231211657442.png)

抓包查看，可以看到设备连回复都不回复一个包

![image-20231231211726788](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231211726788.png)





现在我们按照下面的方式先加一条`REJECT`，然后用`-A`追加一条`DROP`规则，看看结果如何

```bash
iptables -F
iptables -t filter -A INPUT -p icmp -j REJECT
iptables -t filter -A INPUT -p icmp -j DROP
iptables -L
```



![image-20231231212152647](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231212152647.png)



从结果看，是执行了`REJECT`这条规则的，从这个结果也可以看出`iptables`是这样的一个特点：如果有一条明确的规则是禁止还是通过，则不再执行下面的规则了。

![image-20231231212651113](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231212651113.png)



再验证一下，我们使用`-I`来插入一条`ACCEPT`的规则，让这条规则排在序号`1`的位置，想让规则成为第几条规则，则序号就是几

```bash
iptables -I  INPUT 1 -p icmp -j ACCEPT
```



![image-20231231230531426](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231230531426.png)



测试结果表明`iptables`的优先级则跟之前的结论一样

![image-20231231230639659](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231230639659.png)



#### 删除某条规则

直接用序号删除会快一点

```bash
iptables -nL --line-number
iptables -D INPUT 2
```



![image-20231231230849517](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231230849517.png)





或者指定具体的规则

```bash
iptables -D INPUT -p icmp -j ACCEPT
```

![image-20231231231007995](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20231231231007995.png)



#### MAC地址

放一个符合`mac`地址的数据包进来

```bash
iptables -A INPUT -m mac --mac-source 70:a6:cc:80:18:4e -j ACCEPT
```



`iptables`配置如下

![image-20240101114058112](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20240101114058112.png)



实验结果

![image-20240101114155864](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20240101114155864.png)



![image-20240101123605022](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20240101123605022.png)





#### 限制主机或者网段

```bash
iptables -I INPUT -s 192.168.1.110 -j REJECT
iptables -I INPUT -s 192.168.1.0/24 -j REJECT

iptables -I INPUT -s 192.168.1.110 -j DROP
iptables -I INPUT -s 192.168.1.0/24 -j DROP

# 封IP段即从123.45.0.1到123.45.255.254
iptables -I INPUT -s 123.0.0.0/8 -j DRO
```



#### 限制端口

```bash
#允许外部访问22端口
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
#拒绝外部访问22端口
iptables -A OUTPUT -p tcp --dport 22 -j REJECT


#设置外部tcp允许访问25端口
iptables -A OUTPUT -p tcp -m tcp --dport 1600 -j ACCEPT


#封掉多个端口 从23到8809
iptables -I INPUT -p tcp --dport 23:8809 -j DROP 

#封掉21 23 24 80 3306端口
iptables -I INPUT -p tcp -m multiport --dport 21,23,24,80,3306 -j DROP
```





#### 过滤状态

```bash
#允许已建立的或相关连的通行
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
#设置状态为RELATED跟ESTABLISHED的数据可以从服务器发送到外部
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT


#加上端口的限制
iptables -A INPUT -p tcp --dport 1024 -m state --state RELATED,ESTABLISHED -j ACCEPT
```



#### 标记位匹配

```bash
iptables -I INPUT -i eth0 -p tcp --tcp-flags SYN,ACK,FIN,RST SYN -j DROP
```



![image-20240101134612925](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20240101134612925.png)



这里就是涉及到`TCP`三次握手四次挥手那块了



#### IP范围匹配

```bash
iptables -A INPUT -m iprange --src-range 192.168.1.10-192.168.1.20 -j ACCEPT
```





#### 限制icmp_type类型  

```bash
#允许外部ping服务器
iptables -A INPUT -p icmp -m icmp --icmp-type 8
iptables -A INPUT -p icmp -m icmp --icmp-type 11

#允许从服务器ping外部
iptables -A OUTPUT -p icmp -m icmp --icmp-type 8
iptables -A OUTPUT -p icmp -m icmp --icmp-type 11

iptables -A INPUT -p icmp -s 192.168.56.0/24 --icmp-type 8 -j ACCEPT
```



`icmp_type`类型见[链接](https://developer.aliyun.com/article/545246)

|      |      |                                                              |       |       |
| ---- | ---- | ------------------------------------------------------------ | ----- | ----- |
| TYPE | CODE | Description                                                  | Query | Error |
| 0    | 0    | Echo Reply——回显应答（Ping应答）                             | x     |       |
| 3    | 0    | Network Unreachable——网络不可达                              |       | x     |
| 3    | 1    | Host Unreachable——主机不可达                                 |       | x     |
| 3    | 2    | Protocol Unreachable——协议不可达                             |       | x     |
| 3    | 3    | Port Unreachable——端口不可达                                 |       | x     |
| 3    | 4    | Fragmentation needed but no frag. bit set——需要进行分片但设置不分片比特 |       | x     |
| 3    | 5    | Source routing failed——源站选路失败                          |       | x     |
| 3    | 6    | Destination network unknown——目的网络未知                    |       | x     |
| 3    | 7    | Destination host unknown——目的主机未知                       |       | x     |
| 3    | 8    | Source host isolated (obsolete)——源主机被隔离（作废不用）    |       | x     |
| 3    | 9    | Destination network administratively prohibited——目的网络被强制禁止 |       | x     |
| 3    | 10   | Destination host administratively prohibited——目的主机被强制禁止 |       | x     |
| 3    | 11   | Network unreachable for TOS——由于服务类型TOS，网络不可达     |       | x     |
| 3    | 12   | Host unreachable for TOS——由于服务类型TOS，主机不可达        |       | x     |
| 3    | 13   | Communication administratively prohibited by filtering——由于过滤，通信被强制禁止 |       | x     |
| 3    | 14   | Host precedence violation——主机越权                          |       | x     |
| 3    | 15   | Precedence cutoff in effect——优先中止生效                    |       | x     |
| 4    | 0    | Source quench——源端被关闭（基本流控制）                      |       |       |
| 5    | 0    | Redirect for network——对网络重定向                           |       |       |
| 5    | 1    | Redirect for host——对主机重定向                              |       |       |
| 5    | 2    | Redirect for TOS and network——对服务类型和网络重定向         |       |       |
| 5    | 3    | Redirect for TOS and host——对服务类型和主机重定向            |       |       |
| 8    | 0    | Echo request——回显请求（Ping请求）                           | x     |       |
| 9    | 0    | Router advertisement——路由器通告                             |       |       |
| 10   | 0    | Route solicitation——路由器请求                               |       |       |
| 11   | 0    | TTL equals 0 during transit——传输期间生存时间为0             |       | x     |
| 11   | 1    | TTL equals 0 during reassembly——在数据报组装期间生存时间为0  |       | x     |
| 12   | 0    | IP header bad (catchall error)——坏的IP首部（包括各种差错）   |       | x     |
| 12   | 1    | Required options missing——缺少必需的选项                     |       | x     |
| 13   | 0    | Timestamp request (obsolete)——时间戳请求（作废不用）         | x     |       |
| 14   |      | Timestamp reply (obsolete)——时间戳应答（作废不用）           | x     |       |
| 15   | 0    | Information request (obsolete)——信息请求（作废不用）         | x     |       |
| 16   | 0    | Information reply (obsolete)——信息应答（作废不用）           | x     |       |
| 17   | 0    | Address mask request——地址掩码请求                           | x     |       |



#### 其他命令  

```bash
#允许服务器使用外部dns解析域名
iptables -A OUTPUT -p udp -m udp --dport 53 -j ACCEPT
#允许本地回环接口
iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT


#允许内部数据环回
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

iptables -A INPUT -i lo -j REJECT
```





![image-20240123224840304](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/iptables/image-20240123224840304.png)





### 小结

布置防火墙的最终思路：

- 尽可能不给服务器配置外网`IP`，可以通过代理转发或者通过防火墙映射
- 并发量很大的时候，开了`iptables`其实是会影响性能的，可以用硬件防火墙去做



在用`iptables`配置相关规则的时候，要抓住一个思路，先想想数据包是要经过五链中的哪个链，从而在该链上的一些相关的表上配置相关的规则。

下次再来分析内核的`netfilter`框架。





### 注意

发送方向：数据包 ->` netfilter` ->` tcpdump`

接收方向：`netfilter `<- `tcpdump` <- 数据包

由此可知发送方向的包`tcpdump`是抓不到被过滤掉的，但是接收方向是可以的



### 后续

后续讲下`iptables`的`log`记录功能，以及`nat`功能的一些介绍



### 参考链接

- [参考链接一](https://blog.yingchi.io/posts/2020/5/linux-iptables.html)
- [参考链接二](https://tinychen.com/20200414-iptables-principle-introduction/)
- [参考链接三，推荐看看，真实的案例](https://mp.weixin.qq.com/s/R-hjcgUxGGghi_F1MkJf2Q)
- [参考链接四](https://tonydeng.github.io/sdn-handbook/linux/iptables.html)
- [参考链接五](https://developer.aliyun.com/article/545246)











