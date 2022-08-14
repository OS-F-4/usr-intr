# uintr-linux-kernel说明文档

- [toc]

## 项目简介





## 使用方法

### 编译内核(用于qemu)

注: 在内核编译的过程中可能需要安装支持编译内核的软件包, 通过包管理器安装即可。

```shell
cd uintr-linux-kernel/
make O=build x86_64_defconfig
make O=build menuconfig
```



### 编译内核(用于物理机)





### 运行内核(基于qemu)