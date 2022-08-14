# uintr-qemu说明文档

- [toc]

## 项目简介

本项目旨在基于qemu实现intel x86的用户态中断特性, 使得在没有相关硬件的条件下, 我们也能通过模拟器的方式来利用我们的硬件特性, 编写和新的操作系统和用户程序。

我们总体的开发过程如下, 具体的问题解决过程以及解决方式详见[探究过程](https://github.com/OS-F-4/usr-intr/blob/main/ppt/qemu%E5%B7%A5%E4%BD%9C%E6%96%87%E6%A1%A3%E5%88%86%E5%9D%97/%E9%97%AE%E9%A2%98%E4%BB%A5%E5%8F%8A%E6%8E%A2%E7%A9%B6%E8%BF%87%E7%A8%8B.md), 其中有上万字的详细的开发过程中的困难以及解决方式。

1. 环境准备
2. 指令捕捉，向软件反馈硬件特性
3. 内存读写实现发送
4. 修改中断处理实现接收
5. 中断收尾实现完整运行
6. 实现直接发中断，提高性能
7. 多次调试，完善实现细节



## 使用方法

### 编译qemu

```shell
 cd qemu
 mkdir build
 ../configure  --enable-debug  --target-list=x86_64-softmmu --enable-trace-backends=log
 make -j
```

遇到缺少的包, 安装即可：

```shell
ERROR: Cannot find Ninja
sudo apt install ninja-build
```



### 运行linux内核

编译内核详见[如何编译支持用户态中断的内核](https://github.com/OS-F-4/usr-intr/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/linux-kernel.md)。

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









