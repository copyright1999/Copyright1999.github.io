---
layout:     post   				   
title:     linux USB学习(二)			
subtitle:  usb_device_id结构体
date:       2022-07-09				
author:     婷                               
header-img: img/73.jpg 	
catalog: true 						
tags:								

- USB

---



## 简介

这篇主要是讲下`usb_driver`结构体中的`id_table`成员，也就是`usb_device_id`结构体，以及结合具体的`cdc-ncm`驱动的一些分析。



## usb_device_id

在`Linux`内核中，**使用usb_driver结构体描述一个USB设备驱动**，在编写新的`USB`设备驱动时，主要应该完成的工作是`probe（）`和`disconnect（）`函数，即探测和断开函数，它们分别在设备被插入和拔出的时候调用，用于初始化和释放软硬件资源。



那这个`probe`函数跟要讲的`usb_device_id`是什么关系呢？

**当`USB`核心检测到某个设备的属性和某个驱动程序的`usb_device_id`结构体所携带的信息一致时，这个驱动程序的`probe（）`函数就被执行。**



以`cdc-ncm`驱动分析，代码路径`drivers/net/usb/cdc_ncm.c`，找到`usb_driver`，可以看到`id_table`成员变量为`cdc_devs`

![image-20220703220836085](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb2/image-20220703220836085.png)

而且`probe`函数的入参也有`struct usb_device_id`

![image-20220706232144578](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb2/image-20220706232144578.png)



回归正题，接着我们来看`cdc-devs`

![image-20220703221003752](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb2/image-20220703221003752.png)



![image-20220703221136650](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb2/image-20220703221136650.png)





结构体内容很多，我们这里陈列出来

```c
static const struct usb_device_id cdc_devs[] = {
        /* Ericsson MBM devices like F5521gw */
        { .match_flags = USB_DEVICE_ID_MATCH_INT_INFO
                | USB_DEVICE_ID_MATCH_VENDOR,
          .idVendor = 0x0bdb,
          .bInterfaceClass = USB_CLASS_COMM,
          .bInterfaceSubClass = USB_CDC_SUBCLASS_NCM,
          .bInterfaceProtocol = USB_CDC_PROTO_NONE,
          .driver_info = (unsigned long) &wwan_info,
        },

        /* Dell branded MBM devices like DW5550 */
        { .match_flags = USB_DEVICE_ID_MATCH_INT_INFO
                | USB_DEVICE_ID_MATCH_VENDOR,
          .idVendor = 0x413c,
          .bInterfaceClass = USB_CLASS_COMM,
          .bInterfaceSubClass = USB_CDC_SUBCLASS_NCM,
          .bInterfaceProtocol = USB_CDC_PROTO_NONE,
          .driver_info = (unsigned long) &wwan_info,
        },

        /* Toshiba branded MBM devices */
        { .match_flags = USB_DEVICE_ID_MATCH_INT_INFO
                | USB_DEVICE_ID_MATCH_VENDOR,
          .idVendor = 0x0930,
          .bInterfaceClass = USB_CLASS_COMM,
          .bInterfaceSubClass = USB_CDC_SUBCLASS_NCM,
          .bInterfaceProtocol = USB_CDC_PROTO_NONE,
          .driver_info = (unsigned long) &wwan_info,
        },

        /* tag Huawei devices as wwan */
        { USB_VENDOR_AND_INTERFACE_INFO(0x12d1,
                                        USB_CLASS_COMM,
                                        USB_CDC_SUBCLASS_NCM,
                                        USB_CDC_PROTO_NONE),
          .driver_info = (unsigned long)&wwan_info,
        },

        /* Infineon(now Intel) HSPA Modem platform */
        { USB_DEVICE_AND_INTERFACE_INFO(0x1519, 0x0443,
                USB_CLASS_COMM,
                USB_CDC_SUBCLASS_NCM, USB_CDC_PROTO_NONE),
          .driver_info = (unsigned long)&wwan_noarp_info,
        },

        /* Generic CDC-NCM devices */
        { USB_INTERFACE_INFO(USB_CLASS_COMM,
                USB_CDC_SUBCLASS_NCM, USB_CDC_PROTO_NONE),
                .driver_info = (unsigned long)&cdc_ncm_info,
        },
        {
        },
};
MODULE_DEVICE_TABLE(usb, cdc_devs);

```

其实会发现，能匹配到`cdc-ncm`这个驱动的，有很多种情况，因为各种匹配情况不同，最终呢不同的可能只是`driver_info`，而目前`driver_info`有三种，其实也就是`description`跟`flags`不同而已

![image-20220706233115881](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb2/image-20220706233115881.png)



在`include/linux/usb/usbnet.h`中可以找到`driver_info`的定义，其中就有对各种`flag`的解释

```c
/* interface from the device/framing level "minidriver" to core */
struct driver_info {
        char            *description;

        int             flags;
/* framing is CDC Ethernet, not writing ZLPs (hw issues), or optionally: */
#define FLAG_FRAMING_NC 0x0001          /* guard against device dropouts */
#define FLAG_FRAMING_GL 0x0002          /* genelink batches packets */
#define FLAG_FRAMING_Z  0x0004          /* zaurus adds a trailer */
#define FLAG_FRAMING_RN 0x0008          /* RNDIS batches, plus huge header */

#define FLAG_NO_SETINT  0x0010          /* device can't set_interface() */
#define FLAG_ETHER      0x0020          /* maybe use "eth%d" names */

#define FLAG_FRAMING_AX 0x0040          /* AX88772/178 packets */
#define FLAG_WLAN       0x0080          /* use "wlan%d" names */
#define FLAG_AVOID_UNLINK_URBS 0x0100   /* don't unlink urbs at usbnet_stop() */
#define FLAG_SEND_ZLP   0x0200          /* hw requires ZLPs are sent */
#define FLAG_WWAN       0x0400          /* use "wwan%d" names */

#define FLAG_LINK_INTR  0x0800          /* updates link (carrier) status */

#define FLAG_POINTTOPOINT 0x1000        /* possibly use "usb%d" names */

/*
 * Indicates to usbnet, that USB driver accumulates multiple IP packets.
 * Affects statistic (counters) and short packet handling.
 */
#define FLAG_MULTI_PACKET       0x2000
#define FLAG_RX_ASSEMBLE        0x4000  /* rx packets may span >1 frames */
#define FLAG_NOARP              0x8000  /* device can't do ARP */

        /* init device ... can sleep, or cause probe() failure */
        int     (*bind)(struct usbnet *, struct usb_interface *);

        /* cleanup device ... can sleep, but can't fail */
        void    (*unbind)(struct usbnet *, struct usb_interface *);

        /* reset device ... can sleep */
        int     (*reset)(struct usbnet *);

        /* stop device ... can sleep */
        int     (*stop)(struct usbnet *);
       /* see if peer is connected ... can sleep */
        int     (*check_connect)(struct usbnet *);

        /* (dis)activate runtime power management */
        int     (*manage_power)(struct usbnet *, int);

        /* for status polling */
        void    (*status)(struct usbnet *, struct urb *);

        /* link reset handling, called from defer_kevent */
        int     (*link_reset)(struct usbnet *);

        /* fixup rx packet (strip framing) */
        int     (*rx_fixup)(struct usbnet *dev, struct sk_buff *skb);

        /* fixup tx packet (add framing) */
        struct sk_buff  *(*tx_fixup)(struct usbnet *dev,
                                struct sk_buff *skb, gfp_t flags);

        /* recover from timeout */
        void    (*recover)(struct usbnet *dev);

        /* early initialization code, can sleep. This is for minidrivers
         * having 'subminidrivers' that need to do extra initialization
         * right after minidriver have initialized hardware. */
        int     (*early_init)(struct usbnet *dev);

        /* called by minidriver when receiving indication */
        void    (*indication)(struct usbnet *dev, void *ind, int indlen);

        /* rx mode change (device changes address list filtering) */
        void    (*set_rx_mode)(struct usbnet *dev);

        /* for new devices, use the descriptor-reading code instead */
        int             in;             /* rx endpoint */
        int             out;            /* tx endpoint */

        unsigned long   data;           /* Misc driver specific data */
};

```





## 内核中与usb_device_id相关的宏

前面提到我们是根据`usb_device_id `结构体来进行`probe`，`usb_device_id`结构体如下

```c
struct usb_device_id {
        /* which fields to match against? */
        __u16           match_flags;//与哪些字段匹配

        /* Used for product specific matches; range is inclusive */
        /* 用于特定产品匹配；范围包括如下 */
        __u16           idVendor;
        __u16           idProduct;
        __u16           bcdDevice_lo;//最小的版本
        __u16           bcdDevice_hi;//最大的版本

        /* Used for device class matches */
    	/* 用于设备类匹配 */
        __u8            bDeviceClass;//设备类型
        __u8            bDeviceSubClass;//设备子类型
        __u8            bDeviceProtocol;//协议

        /* Used for interface class matches */
    	 /* 用于接口类匹配 */
        __u8            bInterfaceClass;
        __u8            bInterfaceSubClass;
        __u8            bInterfaceProtocol;

        /* Used for vendor-specific interface matches */
    	/* 用于供应商特定的接口匹配 */
        __u8            bInterfaceNumber;
    
    
       /* not matched against */
        kernel_ulong_t  driver_info
                __attribute__((aligned(sizeof(kernel_ulong_t))));
};
```



其中的`match_flags`主要是用来指定哪种方式来匹配，其可选择的类型如下

```c
/* Some useful macros to use to create struct usb_device_id */
#define USB_DEVICE_ID_MATCH_VENDOR              0x0001
#define USB_DEVICE_ID_MATCH_PRODUCT             0x0002
#define USB_DEVICE_ID_MATCH_DEV_LO              0x0004
#define USB_DEVICE_ID_MATCH_DEV_HI              0x0008
#define USB_DEVICE_ID_MATCH_DEV_CLASS           0x0010
#define USB_DEVICE_ID_MATCH_DEV_SUBCLASS        0x0020
#define USB_DEVICE_ID_MATCH_DEV_PROTOCOL        0x0040
#define USB_DEVICE_ID_MATCH_INT_CLASS           0x0080
#define USB_DEVICE_ID_MATCH_INT_SUBCLASS        0x0100
#define USB_DEVICE_ID_MATCH_INT_PROTOCOL        0x0200
#define USB_DEVICE_ID_MATCH_INT_NUMBER          0x0400
```



那总的来说其实比如我们某个设备要匹配上这个驱动，也就是我们来填充这个结构体。内核中有一些跟`usb_device_id`相关的宏，这些宏都可以来生成一个`usb_device_id`结构体的实例。下面介绍几个常用的，所有的定义在`include/linux/usb.h`中都有。





- 根据制造商`ID`和产品`ID`生成一个`usb_device_id`结构体的实例，在数组中增加该元素将意味着该驱动可支持与制造商`ID`、产品`ID`匹配的设备。其实也就是匹配特定的设备。

  ```c
  USB_DEVICE(vendor, product) 
      
  /**
   * USB_DEVICE - macro used to describe a specific usb device
   * @vend: the 16 bit USB Vendor ID
   * @prod: the 16 bit USB Product ID
   *
   * This macro is used to create a struct usb_device_id that matches a
   * specific device.
  */
  #define USB_DEVICE(vend, prod)  \
          .match_flags = USB_DEVICE_ID_MATCH_DEVICE,  \
          .idVendor = (vend),  \
          .idProduct = (prod)    
  ```

  



- 根据制造商`ID`、产品`ID`、**产品版本的最小值和最大值**生成一个`usb_device_id`结构体的实例，在数组中增加该元素将意味着该驱动可支持与制造商`ID`、产品`ID`匹配和`lo~hi`范围内版本的设备。

  ```c
  USB_DEVICE_VER(vendor, product, lo, hi) 
  
  /**
   * USB_DEVICE_VER - describe a specific usb device with a version range
   * @vend: the 16 bit USB Vendor ID
   * @prod: the 16 bit USB Product ID
   * @lo: the bcdDevice_lo value
   * @hi: the bcdDevice_hi value
   *
   * This macro is used to create a struct usb_device_id that matches a
   * specific device, with a version range.
   */
  #define USB_DEVICE_VER(vend, prod, lo, hi) \
          .match_flags = USB_DEVICE_ID_MATCH_DEVICE_AND_VERSION, \
          .idVendor = (vend), \
          .idProduct = (prod), \
          .bcdDevice_lo = (lo), \
          .bcdDevice_hi = (hi)
  ```

  





- 用于创建一个匹配设备指定类型的`usb_device_id`结构体实例。 

  ```c
  USB_DEVICE_INFO(class, subclass, protocol) 
       
  /**
   * USB_DEVICE_INFO - macro used to describe a class of usb devices
   * @cl: bDeviceClass value
   * @sc: bDeviceSubClass value
   * @pr: bDeviceProtocol value
   *
   * This macro is used to create a struct usb_device_id that matches a
   * specific class of devices.
   */
  #define USB_DEVICE_INFO(cl, sc, pr) \
          .match_flags = USB_DEVICE_ID_MATCH_DEV_INFO, \
          .bDeviceClass = (cl), \
          .bDeviceSubClass = (sc), \
          .bDeviceProtocol = (pr)
  ```

  



- 用于创建一个匹配接口指定类型的`usb_device_id`结构体实例

  ```c
  USB_INTERFACE_INFO(class, subclass, protocol) 
  
  /**
   * USB_INTERFACE_INFO - macro used to describe a class of usb interfaces
   * @cl: bInterfaceClass value
   * @sc: bInterfaceSubClass value
   * @pr: bInterfaceProtocol value
   *
   * This macro is used to create a struct usb_device_id that matches a
   * specific class of interfaces.
   */
  #define USB_INTERFACE_INFO(cl, sc, pr) \
          .match_flags = USB_DEVICE_ID_MATCH_INT_INFO, \
          .bInterfaceClass = (cl), \
          .bInterfaceSubClass = (sc), \
          .bInterfaceProtocol = (pr)
  ```

  



那我们以最通用的`/* Generic CDC-NCM devices */`来讲，可以看到他是用了`USB_INTERFACE_INFO`这个宏，他的`match_flags`是`USB_DEVICE_ID_MATCH_INT_INFO`，也就是接口还有设备类型相关，然后还有相关的`USB_CLASS_COMM`   `USB_CDC_SUBCLASS_NCM`  ` USB_CDC_PROTO_NONE`这三个参数匹配的。

![image-20220709154134171](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/linux_usb2/image-20220709154134171.png)

这三个参数宏定义为

```c
#define USB_CLASS_COMM   2
#define USB_CDC_SUBCLASS_NCM 0x0d
#define USB_CDC_PROTO_NONE  0
```



现在设备上接入了一个`CDC-NCM`设备，我们挂载`debugfs`查看

```bash
T:  Bus=01 Lev=01 Prnt=01 Port=00 Cnt=01 Dev#=  5 Spd=480  MxCh= 0
D:  Ver= 2.00 Cls=02(comm.) Sub=00 Prot=00 MxPS=64 #Cfgs=  1
P:  Vendor=0525 ProdID=a4a1 Rev= 4.09
S:  Manufacturer=Linux 4.9.37 with 2b000000.usb
S:  Product=NCM Gadget
C:* #Ifs= 2 Cfg#= 1 Atr=c0 MxPwr=  2mA
A:  FirstIf#= 0 IfCount= 2 Cls=02(comm.) Sub=0d Prot=00
I:* If#= 0 Alt= 0 #EPs= 1 Cls=02(comm.) Sub=0d Prot=00 Driver=cdc_ncm
E:  Ad=82(I) Atr=03(Int.) MxPS=  16 Ivl=32ms
I:  If#= 1 Alt= 0 #EPs= 0 Cls=0a(data ) Sub=00 Prot=01 Driver=cdc_ncm
I:* If#= 1 Alt= 1 #EPs= 2 Cls=0a(data ) Sub=00 Prot=01 Driver=cdc_ncm
E:  Ad=81(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
E:  Ad=01(O) Atr=02(Bulk) MxPS= 512 Ivl=0ms
```



通过这两行查看跟前面的三个参数对应上了`（FirstIF == first interface）`

```bash
A:  FirstIf#= 0 IfCount= 2 Cls=02(comm.) Sub=0d Prot=00
I:* If#= 0 Alt= 0 #EPs= 1 Cls=02(comm.) Sub=0d Prot=00 Driver=cdc_ncm
```





## usb_device_id总结用法

总的来说，可以自己一个成员变量的一个成员变量的去填充`usb_device_id`这个结构体，也可以借助内核提供的宏，更加的直观。

```c
/* 本驱动支持的USB设备列表 */

/* 实例1  借助内核的宏*/
static struct usb_device_id id_table [] = {
 { USB_DEVICE(VENDOR_ID, PRODUCT_ID) },
 { },
};
MODULE_DEVICE_TABLE (usb, id_table);


/* 实例2 手动填充成员变量 */
static struct usb_device_id id_table [] = {
 { .idVendor = 0x10D2, .match_flags = USB_DEVICE_ID_MATCH_VENDOR, },
 { },
};
MODULE_DEVICE_TABLE (usb, id_table);
```







 
