# Linux kernel for IP++ protocol stack

[English](README_en.md) | [简体中文](README.md)

IP++协议栈针对linux内核6.18.23开发。为确保协议栈的正常使用，应采用此处的方法定制内核。可根据具体需求选择源码安装或二进制包安装。

## 安装依赖
apt install flex bison libssl-dev libelf-dev libncurses-dev libdw-dev gawk -y
## 源码安装
### 1.配置
由Linux内核官网[`kernel.org`](http:://www.kernel.org)下载内核6.18.23，解压至相应目录。

cp /boot/config-7.0.0-14-generic /内核源码目录/.config

进入linux内核源码目录并运行

make menuconfig

按下面所示修改配置
```
Processor type and features --->
    [ ] Randomize the address of the kernel image (KASLR)

[*] Networking support --> 
        Networking options -->
        	<*>Transport Layer Security support
        	<*>IP:tunneling
        	<*>IP:GRE demultiplexer
        	<*>IP:GRE tunnels over IP
        	<*>Virtual(secure)IP:tunneling
        	<*>IP:AH transformation
        	<*>IP:ESP transformation
            <*> 802.1d Ethernet Bridging
        <*> Plan 9 Resoures Sharing Support (9P2000) -->
            <*> 9P FD Transport
            <*> 9P Virtio Transport

Device Drivers  --->
    [*] Block devices  --->
        <*> RAM block device support
            (65536) Default RAM disk size (kbytes)
    ...
    Network device support --> 
        <*> Universal TUN/TAP device driver support
        <*> Virtual ethernet pair device

File system  --->
    [*] Network File Systems  --->
        <*> Plan 9 Resoures Sharing Support (9P2000)

Cryptographic API  --->
    Certificates for signature checking  --->
        Provide system-wide ring of trusted keys
        ( )  Additional X.509 keys for default system keyring
  
        [*] Provide system-wide ring of revocation certificates
        ( )  X.509 certificates to be preloaded into the system blacklist keyring

Kernel hacking  --->
    Compile-time checks and compiler options  ---> 
        Debug information
            [X] Rely on the toolchain’s implicit default DWARF version
        [*] Provide GDB scripts for kernel debugging
```
此处已经按照上述方法准备好了一个[.config](./.config)文件，直接拷贝至源码根目录即可使用。
### 2.修改源码
自6.15版开始，某些函数、变量，仅在IPv6功能为M（模块）模式时，才编进符号表。因此要对源码进行简单修改。

include/net/ip.h
```
@@ -695,7 +695,7 @@
-#if IS_MODULE(CONFIG_IPV6)
+// #if IS_MODULE(CONFIG_IPV6)
#define EXPORT_IPV6_MOD(X) EXPORT_SYMBOL(X)
#define EXPORT_IPV6_MOD_GPL(X) EXPORT_SYMBOL_GPL(X)
-#else
-#define EXPORT_IPV6_MOD(X)
-#define EXPORT_IPV6_MOD_GPL(X)
-#endif
+// #else
+// #define EXPORT_IPV6_MOD(X)
+// #define EXPORT_IPV6_MOD_GPL(X)
+// #endif
```

net/ipv4/diff inet_hashtables.c
```
@@ -1222,1 +1222,2 @@
}
+EXPORT_SYMBOL_GPL(__inet_hash_connect);
 
@@ -1243,1 +1244,2 @@
}
+EXPORT_SYMBOL_GPL(inet_hash_connect);
```

### 3.编译
`make`

用时较长。根据您的电脑的配置不同，一般几十分钟至几小时不等。

### 4.安装
```
make modules_install
make install
```
## 二进制包安装

如无需对内核进行特别定制，则推荐使用此方法。

因文件较大，另外存于[腾讯微云](https://share.weiyun.com/YjJKx5h7)。下载安装包文件`*.deb`。

运行安装命令

`sudo dpkg -i *.deb`

## 配置
安装完成后，为方便切换使用不同内核，对启动菜单进行相应配置。
修改文件/etc/default/grub
```
# GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=8
GRUB_DEFAULT="1>0"	# 设置默认内核,具体值查看/boot/grub/grub.cfg。
```
为保证内核体系的稳定，建议将内核自动升级功能禁掉。编辑`/etc/apt/apt.conf.d/20auto-upgrades`文件，将其中`Unattended-Upgrade "1"`处的1改为0。

`update-grub`

`reboot`