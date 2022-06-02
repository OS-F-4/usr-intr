# 如何构建一个 ubuntu 文件系统

> 提示：执行一些操作可能需要有 root 权限

 ## 依赖

```bash
apt-get install -y debootstrap
```

## 指令

```bash
debootstrap --components=main,universe focal ./rootfs "http://mirrors.tuna.tsinghua.edu.cn/ubuntu"
```

## 操作

```bash
cd rootfs
chroot .
# 在启动的新 shell 中，执行以下指令：
export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
apt-get install -y build-essential manpages-dev software-properties-common
add-apt-repository ppa:ubuntu-toolchain-r/test
apt-get update
apt-get install -y gcc-11 g++-11 cmake pkg-config libzmqpp-dev wget texinfo git
update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 60 --slave /usr/bin/g++ g++ /usr/bin/g++-11
apt-get clean
wget https://mirrors.tuna.tsinghua.edu.cn/gnu/binutils/binutils-2.38.tar.bz2
tar -jxf binutils-2.38.tar.bz2
cd binutils-2.38 && ./configure && make -j && make install && cd ..
rm -rf binutils-2.38 binutils-2.38.tar.bz2
git clone -b linux-rfc-v1 https://github.com/OS-F-4/ipc-bench.git
cd ipc-bench
mkdir build
cd build
cmake ..
make
```

## 用到 qemu 中

记得在根目录下新建一个可执行文件 `init`，内容如下：

```bash
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"
exec /bin/bash
```

## 构建文件系统压缩包

```bash
# 在 rootfs 目录下
find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../ubuntu-x86_64.cpio.gz
```

