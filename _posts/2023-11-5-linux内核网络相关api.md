---
layout:     post   				   
title:     linux内核网络相关api			 
subtitle:  
date:       2023-11-5			
author:     婷                              
header-img: img/97.png	
catalog: true 						
tags:								

- gmac
- linux
- 网络
---



### 前言

最近在看`gmac`驱动代码，发现很多`linux`内核网络的`api`，暂时这里先整理出来，先大概知道什么用法，后续再深入了解



### netif_rx

```c
 void netif_rx(struct sk_buff *skb);
```

调用（包括中断期间）这个函数可以通知内核已经收到一个数据包，并封装入一个套接字缓冲区。



### netif_rx_schedule

```c
void netif_rx_schedule(dev);
```

调用该函数通知内核数据包已经存在，并且在接口上启动轮询机制，它只在`NAPI`驱动程序中使用。



### netif_receive_skb与netif_rx_complete

```c
int netif_receive_skb(struct sk_buff *skb)；
void netif_rx_complete(dev);
```

这两函数只在`NAPI`驱动程序中使用；`NAPI`中的`netif_receive_skb`函数与`netif_rx`等价，它将数据包发送给内核。当`NAPI`驱动程序耗尽了为接收数据包准备的内存时，则它将重新启动中断，然后调用`netif_rx_complete`终止轮询函数。 



### skb_shinfo

通过这个宏来判断数据包是由一个数据片段组成，还是由大量数据片段组成。例如海思的网卡代码

![image-20231105103228717](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gmac/image-20231105103228717.png)





### dev_queue_xmit

网络发送函数，最终调用到`MAC`驱动中的`ndo_start_xmit`



### struct sk_buff

定义在`include/linux/skbuff.h`，只列举几个比较重要的结构体成员

```c
struct sk_buff {
        union {
                struct {
                        /* These two members must be first. */
                        struct sk_buff          *next;
                        struct sk_buff          *prev;

                        union {
                                struct net_device       *dev;
                                /* Some protocols might use this space to store information,
                                 * while device pointer would be NULL.
                                 * UDP receive path is one user.
                                 */
                                unsigned long           dev_scratch;
                        };
                };
                struct rb_node          rbnode; /* used in netem, ip4 defrag, and tcp stack */
                struct list_head        list;
        };

        union {
                struct sock             *sk;
                int                     ip_defrag_offset;
        };

        union {
                ktime_t         tstamp;
                u64             skb_mstamp_ns; /* earliest departure time */
        };
        /*
         * This is the control buffer. It is free to use for every
         * layer. Please put your private variables there. If you
         * want to keep them across layers you have to do a skb_clone()
         * first. This is owned by whoever has the skb queued ATM.
         */
        char                    cb[48] __aligned(8);
        unsigned int            len,
                                data_len;
        __u16                   mac_len,
                                hdr_len;
        __be16                  protocol;
        __u16                   transport_header;
        __u16                   network_header;
        __u16                   mac_header;

        /* private: */
        __u32                   headers_end[0];
    
       /* These elements must be at the end, see alloc_skb() for details.  */
        sk_buff_data_t          tail;
        sk_buff_data_t          end;
        unsigned char           *head,
                                *data;
};
```





`dev`表示当前`sk_buff`从哪个设备接收到或者发出的。

`sk`表示当前`sk_buff`所属的`Socket`。

`cb`为控制缓冲区，不管哪个层都可以自由使用此缓冲区，用于放置私有数据。

`destructor`函数，当释放缓冲区的时候可以在此函数里面完成某些动作。

`len`为实际的数据长度，包括主缓冲区中数据长度和分片中的数据长度。

`data_len`为数据长度，只计算分片中数据的长度。

`protocol`协议。

`transport_header`为传输层头部。

`network_header`为网络层头部。

`mac_header`为链接层头部。

`tail`指向实际数据的尾部，`end`指向缓冲区的尾部。`head`指向缓冲区的头部，`data`指向实际数据的头部。结构如下图所示：

![image-20231111202812248](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gmac/image-20231111202812248.png)



### netif_start_queue

`ndo_open`中，调用这个函数，来激活设备发送队列



### netif_stop_queue

`ndo_stop`中，调用这个函数，停止设备传输包



### eth_type_trans

一般用于网络数据接收中，获取该报文的协议类型。

![image-20231111202755841](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/gmac/image-20231111202755841.png)





### 参考链接

- [参考链接一](https://www.cnblogs.com/zxc2man/p/4105652.html)

- [参考链接二](https://www.cnblogs.com/debruyne/p/9133439.html)
