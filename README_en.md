# Linux kernel for IP++ protocol stack

[English](README_en.md) | [简体中文](README.md)

The IP++ protocol stack is developed based on Linux kernel version 6.18.23. To ensure the protocol stack functions correctly, the kernel must be customized using the methods described herein. You may choose either source code installation or binary package installation depending on your specific requirements.

## Install Dependencies
apt install flex bison libssl-dev libelf-dev libncurses-dev libdw-dev gawk -y
## Source Code Installation
### 1.Configuration
Download kernel version 6.18.28 from the official Linux kernel website[`kernel.org`](http:://www.kernel.org)，and extract it to the desired directory。

cp /boot/config-7.0.0-14-generic /path/to/kernel/source/.config

Enter the Linux kernel source code directory and run

make menuconfig

Modify the configuration as shown below
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
A pre-configured [.config](./.config) file is provided here for your convenience; simply copy it to the root directory of the source code.
### 2.Modify Source Code
Starting from version 6.15, certain functions and variables are only included in the symbol table if IPv6 is compiled as a module (M). Therefore, minor source code modifications are required.

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

### 3.Compilation
`make`

This step takes a significant amount of time. Depending on your computer's hardware configuration, it may take anywhere from tens of minutes to several hours.

### 4.Installation
```
make modules_install
make install
```
## Binary Package Installation

If you do not require special customization of the kernel, this method is recommended.

Due to the large file size, the binary package is stored separately on [Tencent Weiyun](https://share.weiyun.com/YjJKx5h7). Please download the installation package files `*.deb`.

Run the installation command:

`sudo dpkg -i *.deb`

## Configuration
After installation, modify the boot menu configuration to facilitate switching between different kernels.
Edit the file`/etc/default/grub`
```
# GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=8
GRUB_DEFAULT="1>0"	# Set the specific value for the default kernel, checking /boot/grub/grub.cfg
```
To ensure kernel stability, it is recommended to disable the kernel's automatic upgrade feature. Edit the file`/etc/apt/apt.conf.d/20auto-upgrades`，and change Unattended-Upgrade "1" to Unattended-Upgrade "0".

`update-grub`

`reboot`