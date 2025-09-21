---
layout:     post   				    
title:      buildroot安装gcc			
subtitle:  
date:       2025-09-21				
author:     婷                               
header-img: img/160.png
catalog: true 						
tags:								

- buildroot
- gcc
- raspi4b

---





## 简介

打算是用`buildroot`把`gcc`安装到树莓派中，然后查了下[发现](https://stackoverflow.com/questions/64610384/installing-gcc-after-flashing-sd-card-buildroot-distro)，`buildroot`从`2012.11`版本就不支持这么做了，如果要实现，需要自己加补丁

<img src="https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rootfs_gcc/202509211737906.png" alt="img" style="zoom: 67%;" />



## 补丁

`commit`链接

```
https://github.com/copyright1999/raspi4b-project/commit/210f3526b47227b7e5f6f59d03dbb8f136afd161
```



### 补丁1

`buildroot/package/gcc/Config.in`文件末尾添加这一段

```SQL
config BR2_PACKAGE_GCC_TARGET
    bool "gcc"
    depends on BR2_TOOLCHAIN_BUILDROOT
    select BR2_PACKAGE_BINUTILS
    select BR2_PACKAGE_BINUTILS_TARGET
    select BR2_PACKAGE_GMP
    select BR2_PACKAGE_MPFR
    select BR2_PACKAGE_MPC
    help
      If you want the target system to be able to run
      binutils/gcc and compile native code, say Y here.

config BR2_EXTRA_TARGET_GCC_CONFIG_OPTIONS
    string "Additional target gcc options"
    default ""
    depends on BR2_PACKAGE_GCC_TARGET
    help
      Any additional target gcc options you may want to include....
      Including, but not limited to --disable-checking etc.
      Refer to */configure in your gcc sources.
```





### 补丁2

增加`buildroot/package/gcc/gcc-target.mk`文件

```makefile
################################################################################
#
# gcc-target
#
################################################################################

GCC_TARGET_VERSION = $(GCC_VERSION)
GCC_TARGET_SITE = $(GCC_SITE)
GCC_TARGET_SOURCE = $(GCC_SOURCE)

# Use the same archive as gcc-initial and gcc-final
GCC_TARGET_DL_SUBDIR = gcc

GCC_TARGET_DEPENDENCIES = gmp mpfr mpc

# First, we use HOST_GCC_COMMON_MAKE_OPTS to get a lot of correct flags (such as
# the arch, abi, float support, etc.) which are based on the config used to
# build the internal toolchain
GCC_TARGET_CONF_OPTS = $(HOST_GCC_COMMON_CONF_OPTS)
# Then, we modify incorrect flags from HOST_GCC_COMMON_CONF_OPTS
GCC_TARGET_CONF_OPTS += \
    --with-sysroot=/ \
    --with-build-sysroot=$(STAGING_DIR) \
    --disable-__cxa_atexit \
    --with-gmp=$(STAGING_DIR) \
    --with-mpc=$(STAGING_DIR) \
    --with-mpfr=$(STAGING_DIR)
# Then, we force certain flags that may appear in HOST_GCC_COMMON_CONF_OPTS
GCC_TARGET_CONF_OPTS += \
    --disable-libquadmath \
    --disable-libsanitizer \
    --disable-plugin \
    --disable-lto
# Finally, we add some of our own flags
GCC_TARGET_CONF_OPTS += \
    --enable-languages=c \
    --disable-boostrap \
    --disable-libgomp \
    --disable-nls \
    --disable-libmpx \
    --disable-gcov \
    $(EXTRA_TARGET_GCC_CONFIG_OPTIONS)

GCC_TARGET_CONF_ENV = $(HOST_GCC_COMMON_CONF_ENV)

GCC_TARGET_MAKE_OPTS += $(HOST_GCC_COMMON_MAKE_OPTS)

# Install standard C headers (from glibc)
define GCC_TARGET_INSTALL_HEADERS
    cp -r $(STAGING_DIR)/usr/include $(TARGET_DIR)/usr
endef
GCC_TARGET_POST_INSTALL_TARGET_HOOKS += GCC_TARGET_INSTALL_HEADERS

GCC_TARGET_GLIBC_LIBS = \
    libBrokenLocale.so libanl.so libbfd.so libc.so libcrypt.so libdl.so \
    libm.so libnss_compat.so libnss_db.so libnss_files.so libnss_hesiod.so \
    libpthread.so libresolv.so librt.so libthread_db.so libutil.so

# Install standard C libraries (from glibc)
define GCC_TARGET_INSTALL_LIBS
    for libpattern in $(GCC_TARGET_GLIBC_LIBS); do \
        $(call copy_toolchain_lib_root,$$libpattern) ; \
    done
    cp -dpf $(STAGING_DIR)/usr/lib/*crt*.o $(TARGET_DIR)/usr/lib/
    cp -dpf $(STAGING_DIR)/usr/lib/*_nonshared.a $(TARGET_DIR)/usr/lib/
endef
GCC_TARGET_POST_INSTALL_TARGET_HOOKS += GCC_TARGET_INSTALL_LIBS

# Remove unnecessary files (extra links to gcc binaries, and libgcc which is
# already in `/lib`)
define GCC_TARGET_RM_FILES
    rm -f $(TARGET_DIR)/usr/bin/$(ARCH)-buildroot-linux-gnu-gcc*
    rm -f $(TARGET_DIR)/usr/lib/libgcc_s*.so*
    rm -f $(TARGET_DIR)/usr/$(ARCH)-buildroot-linux-gnu/lib/ldscripts/elf32*
    rm -f $(TARGET_DIR)/usr/$(ARCH)-buildroot-linux-gnu/lib/ldscripts/elf64b*
endef
GCC_TARGET_POST_INSTALL_TARGET_HOOKS += GCC_TARGET_RM_FILES

$(eval $(autotools-package))
```





### 补丁3

增加`buildroot/package/gcc-target/gcc-target.hash`文件，直接链接`buildroot/package/gcc/gcc.hash`

![image-20250921172406812](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rootfs_gcc/202509211825651.png)



### 补丁4

修改`buildroot/Makefile`文件

![image-20250921172535963](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rootfs_gcc/202509211825247.png)







## buildroot配置

`make menuconfig`，勾选上这三个选项，记住一定要勾选`Enable C++ support`选项

![image-20250921173254709](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rootfs_gcc/202509211825563.png)

配置好后，最好先`make distclean`一下，再去`make`



## 运行结果

编译成功

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rootfs_gcc/202509211826469.png)



安装到树莓派4B的板子上，运行成功

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rootfs_gcc/202509211826560.png)









## 遇到的问题

在编译的过程，碰到了`wsl`的`OOM`问题，可以看到有个`Killed singal`，是进程被`kill`的

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rootfs_gcc/202509211826121.png)

`dmesg`一看不出所料

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rootfs_gcc/202509211826558.png)

一般这种编译工具链，比如`GCC`，`LLVM`都需要机器的内存足够大，而我的`wsl`分配了`4G`，显然不够

![img](https://raw.githubusercontent.com/copyright1999/image-typora-markdown/main/rootfs_gcc/202509211826827.png)

把内存改成了`8G`之后就顺利编译通过了





## 参考链接

- [补丁参考链接](https://luplab.cs.ucdavis.edu/2022/01/06/buildroot-and-compiler-on-target.html)
- [参考链接一](https://www.listera.top/gei/)
- [参考链接二](https://www.linuxquestions.org/questions/linux-software-2/how-to-install-a-native-gcc-toolchain-onto-the-target-platform-4175643535/)











