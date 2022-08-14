# uintr-linux-kernel说明文档

- [toc]

## 项目简介

本项目为支持用户态中断的linux, 使得能够运行使用用户态中断特性的用户程序。

对于操作系统而言, 主要的改变为增加了若干的系统调用, 主要有如下几个系统调用:

```c++
   #define uintr_register_handler(handler, flags)    syscall(__NR_uintr_register_handler, handler, flags)
   #define uintr_unregister_handler(flags)      syscall(__NR_uintr_unregister_handler, flags)
   #define uintr_create_fd(vector, flags)       syscall(__NR_uintr_create_fd, vector, flags)
   #define uintr_register_sender(fd, flags)     syscall(__NR_uintr_register_sender, fd, flags)
   #define uintr_unregister_sender(fd, flags)   syscall(__NR_uintr_unregister_sender, fd, flags)
```

其中系统调用的编号未固定, 编写用户程序时需要对特性操作系统的系统调用编号做适配, 接下来依次介绍系统调用的用途和含义。

#### uintr_register_handler

用于用户态中断的接收方注册中断处理函数, 操作系统会将中断处理函数的地址写入硬件, 从而使得接收到中断后可以跳转到指定的函数入口进行执行。 

#### uintr_unregister_handler

注销中断处理函数, 注销后硬件将不在记录中断处理函数地址, 接受方也无法收到中断。

#### uintr_create_fd

用于接收方创建链接, 接收方向操作系统注册设定的中断向量`vector`, 所有的注册信息将被操作系统封装为一个文件描述符,  用于后续给发送方注册`sender`。注册成功后, 发送方若发送中断, 则收到的中断函数中的参数会携带中断向量`vector`, 从而接收方可以区分不同的发送方。

#### uintr_register_sender

用于发送发注册`sender`, 向操作系统提供的是接收方所注册的文件描述符, 操作系统根据注册的文件描述符来为发送方写入接受方的相关信息, 同时返回接受方信息在`UITT(User Interrupt Target Table)`中的下标, 发送方可以通过下标来给不同的接受方发送用户态中断。

#### uintr_unregister_sender

注销用户态中断的发送方, 注销后对应的下标将无法成功发送中断。


更详细的流程和特性介绍详见[manpage](https://github.com/OS-F-4/uintr-linux-kernel/tree/uintr-next/tools/uintr/manpages)。



## 使用方法

### 编译内核(用于qemu)

注: 在内核编译的过程中可能需要安装支持编译内核的软件包, 通过包管理器安装即可。

```shell
cd uintr-linux-kernel/
make O=build x86_64_defconfig
make O=build menuconfig
```

在选择构建参数时:

在`General setup`目录下, 选择`Initial RAM filesystem and RAM disk (initramfs/initrd) support`。

在`Device Drivers`目录中`Block device`下, 选择`RAM block device support`, 并设置` Default RAM disk size (kbytes)`为65536KB。

在`Processor type and features  ---> `设置`User Interrupts (UINTR) `

随后开始构建:

```shell
make O=build bzImage -j
make O=build  modules -j 4
```

构建完成后, 内核的镜像文件会在`uintr-linux-kernel/build/arch/x86_64/boot/bzImage`, 在后续过程中可以直接调用。



### 创建文件系统(以busybox为例)

接下来应当创建相应的文件系统, 这里以busybox为例, 同时我们也有构建[ubuntu文件系统](https://github.com/OS-F-4/usr-intr/blob/main/ppt/how-to-build-a-ubuntu-rootfs.md)的方法, 并且安装相应的包后可以直接在模拟器中编译用户程序。

下载`busybox`, 执行以下

```shell
mkdir build
make O=build menuconfig
# 在  settings  Build Options 中选择
# [*] Build static binary (no shared libs)
cd build
make -j4
make install
```

```shell
  ./_install//usr/sbin/ubirmvol -> ../../bin/busybox
  ./_install//usr/sbin/ubirsvol -> ../../bin/busybox
  ./_install//usr/sbin/ubiupdatevol -> ../../bin/busybox
  ./_install//usr/sbin/udhcpd -> ../../bin/busybox


--------------------------------------------------
You will probably need to make your busybox binary
setuid root to ensure all configured applets will
work properly.
-------------------------------------------------- # log
```

```shell
mkdir -pv initramfs/x86_64_busybox
cd initramfs/x86_64_busybox/
mkdir -pv {bin,sbin,etc,proc,sys,usr/{bin,sbin}}
cp -av ../../busybox-1.32.0/build/_install/* .
```

文件系统需要初始化脚本，如下：

```shell
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
mknod -m 666 /dev/ttyS0 c 4 64
echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"
setsid cttyhack sh
exec /bin/sh
```

`chmod u+x init`

init放在文件系统根目录下

在`x86_64_busybox`目录下, 执行打包当前文件系统:

```shell
find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../initramfs-busybox-x86_64.cpio.gz
```

### 启动linux的shell脚本如下:

```shell
#!/bin/bash
# 指定文件系统的路径
ubuntu=~/qemu_uintr/ubuntu-x86_64.cpio.gz
box=~/qemu_uintr/initramfs/initramfs-busybox-x86_64.cpio.gz
PORT=2333
# 指定qemu可执行文件路径
QEMU=~/qemu_uintr/qemu/build/x86_64-softmmu/qemu-system-x86_64
# 指定内核路径
KERNEL=~/qemu_uintr/uintr-linux-kernel/build/arch/x86_64/boot/bzImage
$QEMU -smp 2  \
-machine q35,kernel_irqchip=split \
-m 2048M   -nographic -cpu qemu64  \
-kernel $KERNEL \
-initrd $ubuntu \
-append "root=/dev/ram0 rw rootfstype=ext4 console=ttyS0 init=/linuxrc" \
-net user,id=net,hostfwd=tcp::$(PORT)-:22 -net nic,model=e1000e \
-serial mon:stdio
```





### 编译内核(用于物理机)

```shell
cp /boot/configxxx .config # 其中configxxx是当前系统的版本的config文件, 这是为了保持最好的兼容
make menuconfig
```

在`Processor type and features  ---> `设置`User Interrupts (UINTR) `

```shell
# select uintr
make  -j 24
sudo make modules_install
sudo make install
```

此时内核已经安装完毕, 可以通过`grubby --default-kernel` 查看当期默认的启动内核, 确认无误后, 选择重启。

```shell
reboot
```



如果开始编译出来的内核模块太大，导致initramfs也太大，导致夹在时内存溢出， 需要经过压缩或者去除后才能启动。

```shell
cd /lib/modules/<new_kernel>
find . -name *.ko -exec strip --strip-unneeded {} +
```



### 编译用户态中断程序

编译需要用到`gcc-11`, 具体的安装方式见[其他编译帮助](https://github.com/OS-F-4/usr-intr/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/utils.md)。

样例程序位于`uintr-linux-kernel/tools/uintr/sample`, 有对应的Makefile, 若需要自己编写用户程序, 只需要添加`-muintr`, `-mgeneral-regs-only`, ` -minline-all-stringops`三个编译选项即可。

```shell
gcc -Wall -static -muintr -mgeneral-regs-only -minline-all-stringops uipi_sample.c -lpthread
```

