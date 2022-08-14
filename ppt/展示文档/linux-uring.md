# linux-uring

## 项目简介

本项目基于linux的io_uring子系统, 利用用户态中断的硬件特性作为io完成的通知机制, 实现io的异步提交和io完成的异步返回的双向异步过程, 预期在达到io和计算重叠这样基本的异步框架下更低io延迟。

我们的内核程序位于[仓库](https://github.com/OS-F-4/uintr-linux-kernel/tree/uring)。我们的用户程序以及测试程序位于[仓库](https://github.com/OS-F-4/uring)。

详细的原理说明详见我们的[技术分享报告](https://github.com/OS-F-4/usr-intr/blob/main/%E6%8A%80%E6%9C%AF%E4%BA%A4%E6%B5%81%E6%8A%A5%E5%91%8A.pptx)。以及[最终报告](https://github.com/OS-F-4/usr-intr/blob/main/%E6%9C%80%E7%BB%88%E6%8A%A5%E5%91%8A.md)。

## 使用方法

### 编译内核

编译内核需要`gcc-11`, 查看如何[安装并设置编译器](https://github.com/OS-F-4/usr-intr/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/utils.md)。

内核的编译方法同`uintr-linux`

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

