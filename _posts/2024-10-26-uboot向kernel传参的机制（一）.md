---
layout:     post   				    
title:      uboot向kernel传参的机制（一）			
subtitle:  
date:       2024-10-26				
author:     婷                               
header-img: img/130.png 	
catalog: true 						
tags:								

- uboot
- kernel
- bringup

---





## 简介

`uboot`在启动内核时，会向内核传递一些参数，可以通过两种方式传递参数给内核，一种是通过参数链表`(tagged list)`方式，一种是通过设备树方式。文章主要介绍`uboot`向`kernel`传参的这两个机制，其中的一个小点，比如 `uboot`下的`bootargs`，则是`kernel`下的`cmdline`。



## 启动流程

这里简单列举下启动内核的函数流，以`bootm`为例

```c
bootm
    do_bootm
    	bootm_run
        	bootm_run_states
            	boot_selected_os
            		boot_fn <=> do_bootm_linux
              			boot_prep_linux /*准备启动linux之前的一些工作*/
    					boot_jump_linux /*跳转到linux*/
```

两种传参机制都在体现在`boot_prep_linux`函数里面，从代码看出就是三个结果

```c
boot_prep_linux
	debug("using: FDT\n");
	debug("using: ATAGS\n");
	panic("FDT and ATAGS support not compiled in\n");
```



```c
static void boot_prep_linux(struct bootm_headers *images)
{
	char *commandline = env_get("bootargs");

	if (CONFIG_IS_ENABLED(OF_LIBFDT) && IS_ENABLED(CONFIG_LMB) && images->ft_len) {
		debug("using: FDT\n");
		if (image_setup_linux(images)) {
			panic("FDT creation failed!");
		}
	} else if (BOOTM_ENABLE_TAGS) {
		debug("using: ATAGS\n");
		setup_start_tag(gd->bd);
		if (BOOTM_ENABLE_SERIAL_TAG)
			setup_serial_tag(&params);
		if (BOOTM_ENABLE_CMDLINE_TAG)
			setup_commandline_tag(gd->bd, commandline);
		if (BOOTM_ENABLE_REVISION_TAG)
			setup_revision_tag(&params);
		if (BOOTM_ENABLE_MEMORY_TAGS)
			setup_memory_tags(gd->bd);
		if (BOOTM_ENABLE_INITRD_TAG) {
			/*
			 * In boot_ramdisk_high(), it may relocate ramdisk to
			 * a specified location. And set images->initrd_start &
			 * images->initrd_end to relocated ramdisk's start/end
			 * addresses. So use them instead of images->rd_start &
			 * images->rd_end when possible.
			 */
			if (images->initrd_start && images->initrd_end) {
				setup_initrd_tag(gd->bd, images->initrd_start,
						 images->initrd_end);
			} else if (images->rd_start && images->rd_end) {
				setup_initrd_tag(gd->bd, images->rd_start,
						 images->rd_end);
			}
		}
		setup_board_tags(&params);
		setup_end_tag(gd->bd);
		printf("stdebug func:%s line:%d using: ATAGS\n",__func__,__LINE__);
	} else {
		panic("FDT and ATAGS support not compiled in\n");
	}

	board_prep_linux(images);
}
```





## atags

现在很多都不用`atags`的方式传参了，这里介绍一下。`uboot`把要传递给kernel的参数保存在`struct tag`数据结构中，启动内核时，把这个结构体的物理地址传递给`kernel`，`kernel`会通过这个地址分析出`u-boot`传递的参数。

首先看一下`uboot`代码中几个比较重要的数据结构 `struct global_data` ， `struct bd_info`， `struct tag`



### **struct global_data**

定义在`include/asm-generic/global_data.h`这个头文件里面，这里值列举部分成员变量

```c
typedef struct global_data gd_t;
struct global_data {
    struct bd_info *bd;
    unsigned long flags;
    unsigned long env_addr;
    phys_addr_t ram_top; //top address of U-boot in RAM
    unsigned long relocaddr;
    phys_size_t ram_size;
    unsigned long irq_sp;
    unsigned long start_addr_sp;
    unsigned long reloc_off;
    struct global_data *new_gd;
    const void *fdt_blob;//U-Boot own device tree ,NULL if none
    void *new_fdt;
    unsigned long fdt_size;
    enum fdt_source_t fdt_src;
    ...
}
```



`uboot在arch/arm/lib/crt0_64.S`中指定了`gd_t`的地址存放在`x18`寄存器，至于为什么这么做，可以看后面的小节会解释，包括以后分析全流程启动过程也会讲，这里先不深入这个细节。在需要使用`gd`指针的时候,只需要加入`DECLARE_GLOBAL_DATA_PTR`这句话就可以使用。

`arch/arm/include/asm/global_data.h`

```c
#ifdef CONFIG_ARM64
#define DECLARE_GLOBAL_DATA_PTR     register volatile gd_t *gd asm ("x18")
#else
#define DECLARE_GLOBAL_DATA_PTR     register volatile gd_t *gd asm ("r9")
#endif
#endif
```





### **struct bd_info**

前面的`struct global_data`中的第一个成员变量`struct bd_info *bd`，定义在`include/asm-generic/u-boot.h`头文件中

```c
struct bd_info {
	unsigned long	bi_flashstart;	/* start of FLASH memory */
	unsigned long	bi_flashsize;	/* size	 of FLASH memory */
	unsigned long	bi_flashoffset; /* reserved area for startup monitor */
	unsigned long	bi_sramstart;	/* start of SRAM memory */
	unsigned long	bi_sramsize;	/* size	 of SRAM memory */
#ifdef CONFIG_ARM
	unsigned long	bi_arm_freq; /* arm frequency */
	unsigned long	bi_dsp_freq; /* dsp core frequency */
	unsigned long	bi_ddr_freq; /* ddr frequency */
#endif
#if defined(CONFIG_MPC8xx) || defined(CONFIG_E500) || defined(CONFIG_MPC86xx)
	unsigned long	bi_immr_base;	/* base of IMMR register */
#endif
#if defined(CONFIG_M68K)
	unsigned long	bi_mbar_base;	/* base of internal registers */
#endif
#if defined(CONFIG_MPC83xx)
	unsigned long	bi_immrbar;
#endif
	unsigned long	bi_bootflags;	/* boot / reboot flag (Unused) */
	unsigned long	bi_ip_addr;	/* IP Address */
	unsigned short	bi_ethspeed;	/* Ethernet speed in Mbps */
	unsigned long	bi_intfreq;	/* Internal Freq, in MHz */
	unsigned long	bi_busfreq;	/* Bus Freq, in MHz */
#if defined(CONFIG_M68K)
	unsigned long	bi_pcifreq;	/* PCI Bus Freq, in MHz */
#endif
#if defined(CONFIG_EXTRA_CLOCK)
	unsigned long bi_inpfreq;	/* input Freq in MHz */
	unsigned long bi_vcofreq;	/* vco Freq in MHz */
	unsigned long bi_flbfreq;	/* Flexbus Freq in MHz */
#endif
	ulong	        bi_arch_number;	/* unique id for this board */
	ulong	        bi_boot_params;	/* where this board expects params */
	struct {			/* RAM configuration */
		phys_addr_t start;
		phys_size_t size;
	} bi_dram[CONFIG_NR_DRAM_BANKS];
};
```

其中的`ulong bi_boot_params`表示传递给内核的参数的位置，`uboot`中命令`bdinfo`也是查看该结构体的值



那`bi_boot_params`是怎么初始化的呢，则是由每个板子各自得到`board_init`函数来做的，比如我们用的树莓派则是`board/raspberrypi/rpi/rpi.c`

![image-20240804214127522](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/uboot_kernel/image-20240804214127522.png)

`bdinfo`中的`bi_boot_params`也对应得上，就是`0x100`，这个值一般都是厂商自定义的

![image-20240804214225137](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/uboot_kernel/image-20240804214225137.png)



### **struct tag**

链表必须以`ATAG_CORE`开始，以`ATAG_NONE`结束，如下为链表得到每个`node`节点的格式，在`arch/arm/include/asm/setup.h`文件中定义，参数结构包括两个部分，一个是`tag_header`, 一个是`u`联合体

```c
struct tag {
	struct tag_header hdr;
	union {
		struct tag_core		core;
		struct tag_mem32	mem;
		struct tag_videotext	videotext;
		struct tag_ramdisk	ramdisk;
		struct tag_initrd	initrd;
		struct tag_serialnr	serialnr;
		struct tag_revision	revision;
		struct tag_videolfb	videolfb;
		struct tag_cmdline	cmdline;

		/*
		 * Acorn specific
		 */
		struct tag_acorn	acorn;

		/*
		 * DC21285 specific
		 */
		struct tag_memclk	memclk;
	} u;
};


struct tag_cmdline {
	char	cmdline[1];	/* this is the minimum size */
};

struct tag_ramdisk {
	u32 flags;	/* bit 0 = load, bit 1 = prompt */
	u32 size;	/* decompressed ramdisk size in _kilo_ bytes */
	u32 start;	/* starting block of floppy-based RAM disk image */
};

```



`tag_header`结构体定义如下，链表必须以`ATAG_CORE`开始，以`ATAG_NONE`结束，就体现在这个`tag`变量

```c
struct tag_header {
    	//表示整个tag结构体的大小，用字的个数表示而不是字节，等于tag_header加上u联合体的大小
        u32 size;  
    	//整个tag结构体的标记，如ATAG_CORE
        u32 tag;        
};
```





### **boot_prep_linux分析**  

现在来分析`boot_prep_linux`函数，**ATAG的分支**大致流程如下，利用`gd_t->bi_boot_params`记录传递给内核参数链表的地址，先是创建`ATAG_CORE`节点，中间再插入各种`ATAG_MEM`，`ATAG_SERIAL`，等等节点，其中的`ATAG_CMDLINE`就是我们要给内核传的`cmdline`，对`uboot`来说就是`bootargs`啦

```c
DECLARE_GLOBAL_DATA_PTR;
    
boot_prep_linux
    char *commandline = env_get("bootargs");

	//通过bd结构体中的参数在内存中的存放位置gd->bd->bi_boot_params来构建初始化tag结构
	//表明参数结构的开始 ATAG_CORE
    setup_start_tag(gd->bd);
	...
        
    //构建命令行参数的tag结构
	setup_commandline_tag(gd->bd, commandline);
	...
        
    //ATAG_NONE作为结尾
	setup_end_tag(gd->bd);
```



`setup_start_tag`函数分析如下

```c
static void setup_start_tag (struct bd_info *bd)
{
    //获取struct tag指针
	params = (struct tag *)bd->bi_boot_params;

     //设置参数链表开始标签
	params->hdr.tag = ATAG_CORE;
	params->hdr.size = tag_size (tag_core);

	params->u.core.flags = 0;
	params->u.core.pagesize = 0;
	params->u.core.rootdev = 0;

	params = tag_next (params);
}

#define tag_next(t)     ((struct tag *)((u32 *)(t) + (t)->hdr.size))
```



`setup_commandline_tag`函数分析如下

```c
static void setup_commandline_tag(struct bd_info *bd, char *commandline)
{
	char *p;

    //前面已经先获取环境变量中是否有bootargs了
	if (!commandline)
		return;

	/* eat leading white space */
	for (p = commandline; *p == ' '; p++);

	/* skip non-existent command lines so the kernel will still
	 * use its default command line.
	 */
	if (*p == '\0')
		return;

	params->hdr.tag = ATAG_CMDLINE;
	params->hdr.size =
		(sizeof (struct tag_header) + strlen (p) + 1 + 4) >> 2;

    //复制bootargs中的cmdline
	strcpy (params->u.cmdline.cmdline, p);

	params = tag_next (params);
}
```





### 小结

从前面的分析，其实可以总结下`atags`的原理，以下为`uboot`内存的分布图，只列出一部分

- `atags`链表的链表头是用`gd_t->bd_info->bi_boot_params`变量存的内存地址去构造的
- 而`atags_cmdline`是链表中的一个节点，就是我们的重点，节点中的内容则是`bootargs`中的值，`bootargs`是从环境变量中读取的

- `gd_t`的地址在`x18`寄存器存着
- `uboot`下输入`bdinfo`可以查看`gd_t->bd_info`，因为`gd_t`第一个参数就是`bd_info`，所以`bd_info`地址也就是`gd_t`地址



![image-20240804210608649](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/uboot_kernel/image-20240804210608649.png)





### global_data的思考

要理解`global data`的意义，需要理解以下的事实：`uboot`是一个`bootloader`，有些情况下他可能位于系统的只读存储器(`rom`或者`flash`)中，并从那里开始执行。因此，这种情况下，在`uboot`执行的前期(在将自己`copy`到可读写的存储器之前)它所在的存储空间是不可写的，这会有两个问题

- 堆栈无法使用，无法执行函数调用，也就是说`C`环境不可用
- 没有`data`段(或正确初始化的`data`段)可用,不同函数或者代码之间，无法通过全局变量的形式共享数据



针对问题一，通常的解决方案是：

`u-boot`运行起来之后，在那些不需要执行任何初始化动作即可使用的、可读写的存储区域开辟一段堆栈`(stack)`空间。一般来说，大部分的平台都有自己的`sram`可用作堆栈空间，如果实在不行，也可借用` CPU`的`data cache`方法

针对问题二，解决方案要稍微复杂一些

首先对于开发者而言，在`u-boot`被拷贝到可读写的`ram`(这个动作被称作`relacation)`之前，永远不要使用全局变量，其次在`relocation`之前不同模块之间确实有通过全局变量传递数据的需求怎么办，这就是 `global data`需要解决的事情。



为了在`relocation`之前通过全局变量的形式传递数据，`u-boot`设计了一个巧妙的方法：在堆栈配置好之后，在堆栈开始的位置，为`struct global_data`预留空间，并将开始地址(就是一个`struct global_data`指针)保存在 一个寄存器中，后续的传递，都是通过保存在寄存器中的指针实现。对于`arm64`平台而言,该指针保存在了`X18`寄存器中。

在 `arch/arm/lib/crt0_64.S` 中有如下实现

```asm
/*board_init_f_alloc_reserve的返回值(x0)就是global data的指针*/
bl board_init_f_alloc_reserve   
mov sp,x0
/* set up gd here, outside any C code */
mv x18,x0
bl board_init_init_reserve
```





## 参考链接

- [参考链接一](http://www.pedestrian.com.cn/u-boot/transfer_param.html)
- [参考链接二](https://www.cnblogs.com/schips/p/special_node_of_device_tree.html)
- [参考链接三](https://blog.csdn.net/weixin_41884251/article/details/120041877)
- [参考链接四](http://blog.chinaunix.net/uid-31408888-id-5756176.html)









