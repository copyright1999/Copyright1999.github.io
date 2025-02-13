---
layout:     post   				    
title:      emmc的UDA分区enhance功能介绍
subtitle:  
date:       2025-02-13				
author:     婷                               
header-img: img/142.png 	
catalog: true 						
tags:								

- 存储
- emmc

---





## 简介

主要研究下`emmc`中的`UDA`分区的`enhance`的用法，用的`emmc`芯片是三星的，芯片`datasheet`在参考链接有。





## 区域属性

`emmc` 标准中，支持为`UDA` 中一个特定大小的区域设定 `Enhanced`的属性。`Enhanced attribute `的具体作用，由芯片制造商定义。

对于`UDA`分区来说，`Enhanced attribute`就是指`Enhanced storage media`。把`UDA`的某个区域设置为`Enhanced storage media`的

属性后，一般是把该区域的存储介质从` MLC` 改变为 `SLC`，以获得更好的性能和健壮性。





## 未enhance前卡的基本信息

测试的卡的基本信息如下，`uboot`下读取`extCSD`寄存器的值

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737208314078-1.png)

```bash
U-Boot> mmc reg read extcsd all
EXT_CSD:
000:  00 00 00 00 00 00 00 00 00 00
010:  00 00 00 00 00 00 39 00 00 00
020:  00 00 00 00 00 00 00 00 00 00
030:  00 00 00 00 00 00 00 00 00 00
040:  00 00 00 00 00 00 00 00 00 00
050:  00 00 00 00 00 00 00 00 00 00
060:  00 00 00 00 0f 00 00 c8 c8 00
070:  00 00 00 00 00 00 00 00 00 00
080:  00 00 00 00 00 01 00 00 00 00
090:  00 00 00 00 00 00 00 00 00 00
100:  00 05 00 00 00 00 00 00 00 00
110:  00 00 00 00 00 00 00 00 00 00
120:  00 00 00 00 00 00 00 00 00 00
130:  01 00 00 00 00 00 00 00 00 00
140:  00 00 00 00 00 00 00 00 00 00
150:  00 00 00 00 00 00 00 d2 01 00
160:  07 00 01 00 00 00 14 1f 04 00
170:  00 00 00 00 00 00 00 02 00 08
180:  00 00 00 00 01 01 00 00 00 00
190:  00 00 08 00 02 00 57 1f 0a 02
200:  00 00 00 00 00 00 00 00 00 00
210:  00 01 00 00 e9 00 07 11 00 07
220:  07 10 01 01 01 07 20 00 07 11
230:  1b 55 02 00 00 00 00 00 00 00
240:  00 1e 00 00 00 00 00 3c 0a 00
250:  00 01 00 00 06 00 00 00 00 00
260:  00 00 00 00 01 08 00 01 01 01
270:  00 00 00 00 00 00 00 00 00 00
280:  00 00 00 00 00 00 00 00 00 00
290:  00 00 00 00 00 00 00 00 00 00
300:  00 00 00 00 00 00 00 0f 01 00
310:  00 00 00 00 00 00 00 00 00 00
320:  00 00 00 00 00 00 00 00 00 00
330:  00 00 00 00 00 00 00 00 00 00
340:  00 00 00 00 00 00 00 00 00 00
350:  00 00 00 00 00 00 00 00 00 00
360:  00 00 00 00 00 00 00 00 00 00
370:  00 00 00 00 00 00 00 00 00 00
380:  00 00 00 00 00 00 00 00 00 00
390:  00 00 00 00 00 00 00 00 00 00
400:  00 00 00 00 00 00 00 00 00 00
410:  00 00 00 00 00 00 00 00 00 00
420:  00 00 00 00 00 00 00 00 00 00
430:  00 00 00 00 00 00 00 00 00 00
440:  00 00 00 00 00 00 00 00 00 00
450:  00 00 00 00 00 00 00 00 00 00
460:  00 00 00 00 00 00 00 00 00 00
470:  00 00 00 00 00 00 00 00 00 00
480:  00 00 00 00 00 00 00 00 00 81
490:  c7 00 00 03 03 07 05 00 02 01
500:  3f 3f 01 01 01 00 00 00 00 00
510:  00 00
U-Boot>
```



容量信息，总的`UDA`分区为`7634944KB`

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737208512727-7.png)

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737208514770-10.png)





`kernel`下使用`mmc-utils`工具得到的信息

```bash
# mmc extcsd read /dev/mmcblk0
=============================================
  Extended CSD rev 1.8 (MMC 5.1)
=============================================

Card Supported Command sets [S_CMD_SET: 0x01]
HPI Features [HPI_FEATURE: 0x01]: implementation based on CMD13
Background operations support [BKOPS_SUPPORT: 0x01]
Max Packet Read Cmd [MAX_PACKED_READS: 0x3f]
Max Packet Write Cmd [MAX_PACKED_WRITES: 0x3f]
Data TAG support [DATA_TAG_SUPPORT: 0x01]
Data TAG Unit Size [TAG_UNIT_SIZE: 0x02]
Tag Resources Size [TAG_RES_SIZE: 0x00]
Context Management Capabilities [CONTEXT_CAPABILITIES: 0x05]
Large Unit Size [LARGE_UNIT_SIZE_M1: 0x07]
Extended partition attribute support [EXT_SUPPORT: 0x03]
Generic CMD6 Timer [GENERIC_CMD6_TIME: 0x0a]
Power off notification [POWER_OFF_LONG_TIME: 0x3c]
Cache Size [CACHE_SIZE] is 8192 KiB
Background operations status [BKOPS_STATUS: 0x00]
1st Initialisation Time after programmed sector [INI_TIMEOUT_AP: 0x1e]
Power class for 52MHz, DDR at 3.6V [PWR_CL_DDR_52_360: 0x00]
Power class for 52MHz, DDR at 1.95V [PWR_CL_DDR_52_195: 0x00]
Power class for 200MHz at 3.6V [PWR_CL_200_360: 0x00]
Power class for 200MHz, at 1.95V [PWR_CL_200_195: 0x00]
Minimum Performance for 8bit at 52MHz in DDR mode:
 [MIN_PERF_DDR_W_8_52: 0x00]
 [MIN_PERF_DDR_R_8_52: 0x00]
TRIM Multiplier [TRIM_MULT: 0x02]
Secure Feature support [SEC_FEATURE_SUPPORT: 0x55]
Boot Information [BOOT_INFO: 0x07]
 Device supports alternative boot method
 Device supports dual data rate during boot
 Device supports high speed timing during boot
Boot partition size [BOOT_SIZE_MULTI: 0x20]
Access size [ACC_SIZE: 0x07]
High-capacity erase unit size [HC_ERASE_GRP_SIZE: 0x01]
 i.e. 512 KiB
High-capacity erase timeout [ERASE_TIMEOUT_MULT: 0x01]
Reliable write sector count [REL_WR_SEC_C: 0x01]
High-capacity W protect group size [HC_WP_GRP_SIZE: 0x10]
 i.e. 8192 KiB
Sleep current (VCC) [S_C_VCC: 0x07]
Sleep current (VCCQ) [S_C_VCCQ: 0x07]
Sleep/awake timeout [S_A_TIMEOUT: 0x11]
Sector Count [SEC_COUNT: 0x00e90000]
 Device is block-addressed
Minimum Write Performance for 8bit:
 [MIN_PERF_W_8_52: 0x00]
 [MIN_PERF_R_8_52: 0x00]
 [MIN_PERF_W_8_26_4_52: 0x00]
 [MIN_PERF_R_8_26_4_52: 0x00]
Minimum Write Performance for 4bit:
 [MIN_PERF_W_4_26: 0x00]
 [MIN_PERF_R_4_26: 0x00]
Power classes registers:
 [PWR_CL_26_360: 0x00]
 [PWR_CL_52_360: 0x00]
 [PWR_CL_26_195: 0x00]
 [PWR_CL_52_195: 0x00]
Partition switching timing [PARTITION_SWITCH_TIME: 0x02]
Out-of-interrupt busy timing [OUT_OF_INTERRUPT_TIME: 0x0a]
I/O Driver Strength [DRIVER_STRENGTH: 0x1f]
Card Type [CARD_TYPE: 0x57]
 HS400 Dual Data Rate eMMC @200MHz 1.8VI/O
 HS200 Single Data Rate eMMC @200MHz 1.8VI/O
 HS Dual Data Rate eMMC @52MHz 1.8V or 3VI/O
 HS eMMC @52MHz - at rated device voltage(s)
 HS eMMC @26MHz - at rated device voltage(s)
CSD structure version [CSD_STRUCTURE: 0x02]
Command set [CMD_SET: 0x00]
Command set revision [CMD_SET_REV: 0x00]
Power class [POWER_CLASS: 0x00]
High-speed interface timing [HS_TIMING: 0x01]
Enhanced Strobe mode [STROBE_SUPPORT: 0x01]
Erased memory content [ERASED_MEM_CONT: 0x00]
Boot configuration bytes [PARTITION_CONFIG: 0x08]
 Boot Partition 1 enabled
 No access to boot partition
Boot config protection [BOOT_CONFIG_PROT: 0x00]
Boot bus Conditions [BOOT_BUS_CONDITIONS: 0x02]
High-density erase group definition [ERASE_GROUP_DEF: 0x01]
Boot write protection status registers [BOOT_WP_STATUS]: 0x00
Boot Area Write protection [BOOT_WP]: 0x00
 Power ro locking: possible
 Permanent ro locking: possible
 partition 0 ro lock status: not locked
 partition 1 ro lock status: not locked
User area write protection register [USER_WP]: 0x00
FW configuration [FW_CONFIG]: 0x00
RPMB Size [RPMB_SIZE_MULT]: 0x04
Write reliability setting register [WR_REL_SET]: 0x1f
 user area: the device protects existing data if a power failure occurs during a write operation
 partition 1: the device protects existing data if a power failure occurs during a write operation
 partition 2: the device protects existing data if a power failure occurs during a write operation
 partition 3: the device protects existing data if a power failure occurs during a write operation
 partition 4: the device protects existing data if a power failure occurs during a write operation
Write reliability parameter register [WR_REL_PARAM]: 0x14
 Device supports the enhanced def. of reliable write
Enable background operations handshake [BKOPS_EN]: 0x00
H/W reset function [RST_N_FUNCTION]: 0x01
HPI management [HPI_MGMT]: 0x01
Partitioning Support [PARTITIONING_SUPPORT]: 0x07
 Device support partitioning feature
 Device can have enhanced tech.
Max Enhanced Area Size [MAX_ENH_SIZE_MULT]: 0x0001d2
 i.e. 3817472 KiB
Partitions attribute [PARTITIONS_ATTRIBUTE]: 0x00
Partitioning Setting [PARTITION_SETTING_COMPLETED]: 0x00
 Device partition setting NOT complete
General Purpose Partition Size
 [GP_SIZE_MULT_4]: 0x000000
 [GP_SIZE_MULT_3]: 0x000000
 [GP_SIZE_MULT_2]: 0x000000
 [GP_SIZE_MULT_1]: 0x000000
Enhanced User Data Area Size [ENH_SIZE_MULT]: 0x000000
 i.e. 0 KiB
Enhanced User Data Start Address [ENH_START_ADDR]: 0x00000000
 i.e. 0 bytes offset
Bad Block Management mode [SEC_BAD_BLK_MGMNT]: 0x00
Periodic Wake-up [PERIODIC_WAKEUP]: 0x00
Program CID/CSD in DDR mode support [PROGRAM_CID_CSD_DDR_SUPPORT]: 0x01
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[127]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[126]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[125]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[124]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[123]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[122]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[121]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[120]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[119]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[118]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[117]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[116]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[115]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[114]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[113]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[112]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[111]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[110]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[109]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[108]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[107]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[106]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[105]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[104]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[103]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[102]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[101]]: 0x05
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[100]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[99]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[98]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[97]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[96]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[95]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[94]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[93]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[92]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[91]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[90]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[89]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[88]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[87]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[86]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[85]]: 0x01
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[84]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[83]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[82]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[81]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[80]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[79]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[78]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[77]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[76]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[75]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[74]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[73]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[72]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[71]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[70]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[69]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[68]]: 0xc8
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[67]]: 0xc8
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[66]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[65]]: 0x00
Vendor Specific Fields [VENDOR_SPECIFIC_FIELD[64]]: 0x0f
Native sector size [NATIVE_SECTOR_SIZE]: 0x00
Sector size emulation [USE_NATIVE_SECTOR]: 0x00
Sector size [DATA_SECTOR_SIZE]: 0x00
1st initialization after disabling sector size emulation [INI_TIMEOUT_EMU]: 0x00
Class 6 commands control [CLASS_6_CTRL]: 0x00
Number of addressed group to be Released[DYNCAP_NEEDED]: 0x00
Exception events control [EXCEPTION_EVENTS_CTRL]: 0x0000
Exception events status[EXCEPTION_EVENTS_STATUS]: 0x0000
Extended Partitions Attribute [EXT_PARTITIONS_ATTRIBUTE]: 0x0000
Context configuration [CONTEXT_CONF[51]]: 0x00
Context configuration [CONTEXT_CONF[50]]: 0x00
Context configuration [CONTEXT_CONF[49]]: 0x00
Context configuration [CONTEXT_CONF[48]]: 0x00
Context configuration [CONTEXT_CONF[47]]: 0x00
Context configuration [CONTEXT_CONF[46]]: 0x00
Context configuration [CONTEXT_CONF[45]]: 0x00
Context configuration [CONTEXT_CONF[44]]: 0x00
Context configuration [CONTEXT_CONF[43]]: 0x00
Context configuration [CONTEXT_CONF[42]]: 0x00
Context configuration [CONTEXT_CONF[41]]: 0x00
Context configuration [CONTEXT_CONF[40]]: 0x00
Context configuration [CONTEXT_CONF[39]]: 0x00
Context configuration [CONTEXT_CONF[38]]: 0x00
Context configuration [CONTEXT_CONF[37]]: 0x00
Packed command status [PACKED_COMMAND_STATUS]: 0x00
Packed command failure index [PACKED_FAILURE_INDEX]: 0x00
Power Off Notification [POWER_OFF_NOTIFICATION]: 0x01
Control to turn the Cache ON/OFF [CACHE_CTRL]: 0x01
Control to turn the Cache Barrier ON/OFF [BARRIER_CTRL]: 0x00
eMMC Firmware Version:
eMMC Life Time Estimation A [EXT_CSD_DEVICE_LIFE_TIME_EST_TYP_A]: 0x01
eMMC Life Time Estimation B [EXT_CSD_DEVICE_LIFE_TIME_EST_TYP_B]: 0x01
eMMC Pre EOL information [EXT_CSD_PRE_EOL_INFO]: 0x01
Secure Removal Type [SECURE_REMOVAL_TYPE]: 0x39
 information is configured to be removed using a vendor defined
 Supported Secure Removal Type:
  information removed by an erase of the physical memory
  information removed using a vendor defined
Command Queue Support [CMDQ_SUPPORT]: 0x01
Command Queue Depth [CMDQ_DEPTH]: 16
Command Enabled [CMDQ_MODE_EN]: 0x00
Note: CMDQ_MODE_EN may not indicate the runtime CMDQ ON or OFF.
Please check sysfs node '/sys/devices/.../mmc_host/mmcX/mmcX:XXXX/cmdq_en'
#
```

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737208429604-4.png)



写`4G`数据量的速度平均值为`42.74MB/s`

```bash
time dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=4000
```



![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737208558554-13.png)

读`4G`数据量的速度平均值为`45.23MB/s`

```bash
time dd if=/dev/mmcblk0 of=/dev/null bs=1M count=4000
```



![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737208666558-16.png)



## enhance相关寄存器介绍 

主要有四个寄存器

![image-20250212232424397](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/image-20250212232424397.png)







### Max Enhanced Area Size

定义最大能`enhance`的大小，公式为

```bash
Max Enhanced Area = MAX_ENH_SIZE_MULT x HC_WP_GRP_SIZE x HC_ERASE_GRP_SIZE x 512kBytes 
```

这个最大的能`enhance`的大小，指的是`GPP`分区跟`UDA`分区一起能`enhance`的大小

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737211053104-61.png)



这里的`MAX_ENH_SIZE_MULT`为`0x1d2`，带入上面的公式算出来就是`3817472KB`

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737211313476-76.png)







### Enhanced User Data Area Size

这个寄存器是表示已经`enhance`的`UDA`分区的大小，当然如果还没`enhance`的话肯定是`0`，`enhance`后该寄存器的值会反映`enhance`后的`UDA`分区的大小

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737211079639-64.png)





### Enhanced User Data Start Address

配置要从`UDA`分区的哪个偏移地址开始进行`enhance`，寄存器的值单位是`512byte`，也即当该寄存器的值为`1`时，表示偏移为`512byte`

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737211095918-67.png)





### Partitions atrribute

如果要进行`UDA`分区的`enhance`，则需要设置此寄存器的`bit0`，当然`kernel`下的`mmc-utils`工具会帮你设置好的

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737211105660-70.png)







## enhance测试

### 准备工作

先拷贝一个文件到`UDA`分区的`offset`为`0x2000 blk`处，这样就可以对比`enhance`前后，`UDA`分区中数据的变化，`0x2000 blk`也即`4096KB`

以`python3.11`这个文件为例

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737210056453-31.png)

如图所示，我们的文件已写入`0x2000 blk`偏移处

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737210037983-28.png)



将相关的寄存器值内容打印出来

```bash
mmc extcsd read /dev/mmcblk0 | grep MAX_ENH_SIZE_MULT -A 1
mmc extcsd read /dev/mmcblk0 | grep ENH_SIZE_MULT -A 1
mmc extcsd read /dev/mmcblk0 | grep ENH_START_ADDR -A 1
mmc extcsd read /dev/mmcblk0 | grep PARTITIONS -A 1
```



![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737210445899-34.png)





### 预设

因为`enhance`操作是一次性编程的操作，配了一次`UDA`的`enhance`后就不能再配了，所以`mmc-utils`中的`mmc enh_area set`有预先设置命令，相当于一种预告，注意使用参数`-n`

其参数用法为

```bash
mmc enh_area set <-y|-n|-c> <start KiB> <length KiB> <device>
```

- `start KiB`：我们这里假如配置为`4096`，代表开始`enhance`的区域的偏移量，`4096KB`反映到前面提到的`Enhanced User Data Start Address`寄存器，则会是`0x2000`



```bash
# mmc enh_area set -n 4096 81920 /dev/mmcblk0
# mmc extcsd read /dev/mmcblk0 | grep ENH_START_ADDR -A 1
# mmc extcsd read /dev/mmcblk0 | grep PARTITIONS -A 1
# mmc extcsd read /dev/mmcblk0 | grep ENH_SIZE_MULT -A 1
```

这里我们配置了一下，开始的偏移为`4096KB`，`enhance`的大小为`81920KB`，可以看到预设后的`extcsd`寄存器的相关变化

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737210545176-37.png)







### 实操 

上一步中的`-n`参数是一种预设值，只是让你看到设置后得到的预期结果，并没有实际生效，下电后就没有了。而`-y`参数则是真正生效的操作，配置后下电重启即可生效，这一步操作是不可撤销的，注意。

```bash
mmc enh_area set -y 4096 81920 /dev/mmcblk0
```



![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737210583839-40.png)





## enhance后卡的信息 

下电后再上电，可以看到我们的容量变小了 ，也即`enhance`生效了

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737210649768-52.png)

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737210647925-49.png)

不过本来想对比是不是`4096KB`处的数据被清掉了，结果发现好像`UDA`分区全部都被清零了

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737210645835-46.png)

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737210643114-43.png)





现在查看`enhance`相关寄存器的信息

```bash
mmc extcsd read /dev/mmcblk0 | grep ENH_START_ADDR -A 1
mmc extcsd read /dev/mmcblk0 | grep PARTITIONS -A 1
mmc extcsd read /dev/mmcblk0 | grep ENH_SIZE_MULT -A 1
```



![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737210682946-55.png)





速度上，好像没体现出来从`MLC`变成`SLC`之后变快的特性，也有可能是因为我只`enhance`了`80MB`的原因？不够大，体现不出来

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737209157215-19.png)



![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737209169158-22.png)







因为`UDA`的`enhance`的大小是一次性编程的，这时候如果再去设置一次，就会提示已经`enhance`过了

```bash
mmc enh_area set -n 102400 102400 /dev/mmcblk0
```

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/emmc_enhance/1737209892939-25.png)







## 参考链接

- [emmc颗粒datasheet](https://github.com/copyright1999/image-typora-markdown/blob/main/emmc_uda_enhanced/KLMAG1JETD-B041.pdf)

- [参考链接一](https://github.com/copyright1999/image-typora-markdown/blob/main/emmc_uda_enhanced/S32_eMMC%E5%BA%94%E7%94%A8_GP_RPMB_V2_20211013.pdf)

- [参考链接二](https://github.com/copyright1999/image-typora-markdown/blob/main/emmc_uda_enhanced/eMMC_RPMB_Enhance_GP_and_user_protection.pdf)

  




## 后续

下次介绍`GPP`分区的创建，也是跟这个`enhance`有关















